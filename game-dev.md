---
name: game-dev
description: Build browser games and interactive game components with HTML5 Canvas, WebGL, and React. Use this skill whenever the user asks to create a game, game mechanic, interactive playable, arcade-style app, puzzle, simulation, or any project involving a game loop, animation frame, collision detection, scoring, or player input. Trigger even for casual mentions like "make it a game", "add a score", "bouncing ball", "snake", "tetris clone", "dodge the obstacles", or "make it interactive and fun". Also use for React game components with hooks-based game loops, canvas rendering inside React, or game UI (health bars, scoreboards, menus, leaderboards). If it involves player input + state + feedback loop, this skill applies.
---

# Game Development Skill

Covers browser/HTML5 games (Canvas, WebGL) and React interactive game components — from project setup through game loop, physics, collision, UI, menus, and scoring.

---

## Quick Decision Tree

1. **React artifact?** → Use React + hooks game loop (see React section)
2. **Standalone HTML file?** → Use HTML5 Canvas (see Canvas section)
3. **WebGL / 3D?** → See WebGL section; consider Three.js
4. **Just a UI widget (scoreboard, menu)?** → Use frontend-design skill first, then bolt on game logic

---

## Universal Game Architecture

Every game needs these layers — implement all of them:

```
Input → State Update → Collision/Physics → Render → Repeat
```

### State Shape (always define upfront)

```js
const state = {
  phase: 'menu' | 'playing' | 'paused' | 'gameover',
  score: 0,
  highScore: 0,
  player: { x, y, vx, vy, width, height, health },
  entities: [],   // enemies, bullets, collectibles
  frame: 0,
  dt: 0,          // delta time in seconds
};
```

### Delta-Time Game Loop (always use dt, never raw frame count for physics)

```js
let lastTime = 0;
function loop(timestamp) {
  const dt = Math.min((timestamp - lastTime) / 1000, 0.05); // cap at 50ms
  lastTime = timestamp;
  update(dt);
  render();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
```

---

## HTML5 Canvas Games

### Boilerplate

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #0a0a0f; display: flex; align-items: center; justify-content: center; height: 100vh; }
    canvas { display: block; image-rendering: pixelated; }
  </style>
</head>
<body>
<canvas id="game"></canvas>
<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

// Responsive sizing
function resize() {
  const size = Math.min(window.innerWidth, window.innerHeight, 600);
  canvas.width = size;
  canvas.height = size;
}
resize();
window.addEventListener('resize', resize);
</script>
</body>
</html>
```

### Input Handling

```js
const keys = {};
const mouse = { x: 0, y: 0, down: false };

window.addEventListener('keydown', e => { keys[e.key] = true; e.preventDefault(); });
window.addEventListener('keyup',   e => { keys[e.key] = false; });
canvas.addEventListener('mousemove', e => {
  const r = canvas.getBoundingClientRect();
  mouse.x = (e.clientX - r.left) * (canvas.width / r.width);
  mouse.y = (e.clientY - r.top)  * (canvas.height / r.height);
});
canvas.addEventListener('mousedown', () => mouse.down = true);
canvas.addEventListener('mouseup',   () => mouse.down = false);

// Touch support
canvas.addEventListener('touchmove', e => {
  e.preventDefault();
  const t = e.touches[0], r = canvas.getBoundingClientRect();
  mouse.x = (t.clientX - r.left) * (canvas.width / r.width);
  mouse.y = (t.clientY - r.top)  * (canvas.height / r.height);
}, { passive: false });
```

### Common Canvas Drawing Helpers

```js
// Rounded rectangle
function roundRect(ctx, x, y, w, h, r) {
  ctx.beginPath();
  ctx.roundRect(x, y, w, h, r);
  ctx.fill();
}

// Text with shadow
function drawText(ctx, text, x, y, { size = 24, color = '#fff', align = 'center', shadow = true } = {}) {
  ctx.font = `bold ${size}px monospace`;
  ctx.textAlign = align;
  ctx.textBaseline = 'middle';
  if (shadow) { ctx.shadowBlur = 10; ctx.shadowColor = color; }
  ctx.fillStyle = color;
  ctx.fillText(text, x, y);
  ctx.shadowBlur = 0;
}

// Sprite sheet frame
function drawSprite(ctx, img, frameX, frameY, frameW, frameH, dx, dy, dw, dh) {
  ctx.drawImage(img, frameX * frameW, frameY * frameH, frameW, frameH, dx, dy, dw, dh);
}
```

---

## React Game Components

### Hooks-Based Game Loop

```jsx
import { useEffect, useRef, useCallback } from 'react';

function useGameLoop(update, render, deps = []) {
  const rafRef = useRef();
  const lastTimeRef = useRef(0);

  useEffect(() => {
    function loop(timestamp) {
      const dt = Math.min((timestamp - lastTimeRef.current) / 1000, 0.05);
      lastTimeRef.current = timestamp;
      update(dt);
      render();
      rafRef.current = requestAnimationFrame(loop);
    }
    rafRef.current = requestAnimationFrame(loop);
    return () => cancelAnimationFrame(rafRef.current);
  }, deps);
}
```

### Canvas in React

```jsx
function GameCanvas({ width = 600, height = 600 }) {
  const canvasRef = useRef(null);
  const stateRef = useRef({ /* initial state */ });

  useGameLoop(
    (dt) => { /* update stateRef.current using dt */ },
    () => {
      const ctx = canvasRef.current?.getContext('2d');
      if (!ctx) return;
      ctx.clearRect(0, 0, width, height);
      // draw from stateRef.current
    }
  );

  return <canvas ref={canvasRef} width={width} height={height} style={{ display: 'block' }} />;
}
```

**Key rule**: game state lives in `useRef` (not `useState`) so updates don't trigger re-renders. Only use `useState` for UI outside the canvas (score display, menu phase).

### Input in React

```jsx
useEffect(() => {
  const keys = {};
  const onDown = e => { keys[e.key] = true; };
  const onUp   = e => { keys[e.key] = false; };
  window.addEventListener('keydown', onDown);
  window.addEventListener('keyup', onUp);
  stateRef.current.keys = keys;
  return () => {
    window.removeEventListener('keydown', onDown);
    window.removeEventListener('keyup', onUp);
  };
}, []);
```

---

## Physics & Collision

### AABB Collision (axis-aligned bounding box)

```js
function aabb(a, b) {
  return a.x < b.x + b.w && a.x + a.w > b.x &&
         a.y < b.y + b.h && a.y + a.h > b.y;
}
```

### Circle Collision

```js
function circles(a, b) {
  const dx = a.x - b.x, dy = a.y - b.y;
  return dx*dx + dy*dy < (a.r + b.r) ** 2;
}
```

### Basic Gravity + Friction

```js
function applyPhysics(entity, dt) {
  entity.vy += 800 * dt;           // gravity (px/s²)
  entity.x  += entity.vx * dt;
  entity.y  += entity.vy * dt;
  entity.vx *= Math.pow(0.85, dt * 60); // friction
}
```

### Screen Wrapping / Clamping

```js
// Wrap
if (entity.x > canvas.width)  entity.x = -entity.w;
if (entity.x < -entity.w)     entity.x = canvas.width;

// Clamp
entity.x = Math.max(0, Math.min(canvas.width - entity.w, entity.x));
entity.y = Math.max(0, Math.min(canvas.height - entity.h, entity.y));
```

---

## Game Phases & Screens

Always implement all three: menu → playing → game over.

```js
function render() {
  ctx.clearRect(0, 0, W, H);
  if (state.phase === 'menu')     renderMenu();
  else if (state.phase === 'playing')  renderGame();
  else if (state.phase === 'gameover') renderGameOver();
}

function renderMenu() {
  drawText(ctx, 'GAME TITLE', W/2, H*0.35, { size: 48, color: '#f0c' });
  drawText(ctx, 'Click or Press Space to Start', W/2, H*0.55, { size: 18, color: '#aaa' });
}

function renderGameOver() {
  ctx.fillStyle = 'rgba(0,0,0,0.6)';
  ctx.fillRect(0, 0, W, H);
  drawText(ctx, 'GAME OVER', W/2, H*0.4, { size: 42, color: '#f44' });
  drawText(ctx, `Score: ${state.score}`, W/2, H*0.55, { size: 28 });
  drawText(ctx, 'Press Space to Restart', W/2, H*0.7, { size: 16, color: '#aaa' });
}
```

---

## Scoring & High Score

```js
// Persist high score
function saveHighScore(score) {
  try { localStorage.setItem('hs', Math.max(score, getHighScore())); } catch {}
}
function getHighScore() {
  try { return parseInt(localStorage.getItem('hs')) || 0; } catch { return 0; }
}

// Animated score pop
function scorePopup(x, y, value) {
  particles.push({ x, y, text: `+${value}`, life: 1, vy: -80 });
}
```

---

## Particles & Juice

Game feel comes from particles, screen shake, and sound. Always add at least one:

```js
// Particle system
const particles = [];
function spawnParticles(x, y, color, count = 12) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    const speed = 80 + Math.random() * 160;
    particles.push({ x, y, vx: Math.cos(angle)*speed, vy: Math.sin(angle)*speed,
                     life: 1, color, r: 3 + Math.random() * 4 });
  }
}
function updateParticles(dt) {
  for (let p of particles) { p.x += p.vx*dt; p.y += p.vy*dt; p.life -= dt*2; }
  particles.splice(0, particles.length, ...particles.filter(p => p.life > 0));
}
function drawParticles() {
  for (const p of particles) {
    ctx.globalAlpha = p.life;
    ctx.fillStyle = p.color;
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.r * p.life, 0, Math.PI*2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;
}

// Screen shake
let shakeTime = 0, shakeIntensity = 0;
function shake(intensity = 8, duration = 0.2) { shakeIntensity = intensity; shakeTime = duration; }
function applyShake(dt) {
  if (shakeTime <= 0) return;
  shakeTime -= dt;
  const s = shakeIntensity * (shakeTime / 0.2);
  ctx.translate((Math.random()-0.5)*s, (Math.random()-0.5)*s);
}
```

---

## Entity Pooling (performance for many objects)

```js
class Pool {
  constructor(create, reset) { this.create = create; this.reset = reset; this.pool = []; }
  get()    { return this.pool.length ? this.reset(this.pool.pop()) : this.create(); }
  release(obj) { this.pool.push(obj); }
}

const bulletPool = new Pool(
  () => ({ x:0, y:0, vx:0, vy:0, active:false }),
  (b) => { b.active = true; return b; }
);
```

---

## Audio (Web Audio API)

```js
const AC = new (window.AudioContext || window.webkitAudioContext)();

function playTone(freq = 440, type = 'square', duration = 0.1, vol = 0.3) {
  const osc  = AC.createOscillator();
  const gain = AC.createGain();
  osc.connect(gain); gain.connect(AC.destination);
  osc.type = type; osc.frequency.value = freq;
  gain.gain.setValueAtTime(vol, AC.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, AC.currentTime + duration);
  osc.start(); osc.stop(AC.currentTime + duration);
}

// Resume on first interaction (required by browsers)
document.addEventListener('click', () => AC.resume(), { once: true });
```

---

## Polish Checklist

Before calling any game done, verify:

- [ ] Delta-time used for all movement (frame-rate independent)
- [ ] Menu screen with title and start instruction
- [ ] Game-over screen with score and restart option
- [ ] High score persisted to localStorage
- [ ] At least one particle effect or screen shake on impact/death
- [ ] Touch / mobile input handled
- [ ] Canvas scales responsively
- [ ] Consistent visual style (colors, fonts, effects cohesive)

---

## Reference Files

- `references/webgl.md` — WebGL & Three.js setup, shader basics, 3D game patterns
- `references/patterns.md` — Common game genres: platformer, top-down shooter, puzzle, endless runner