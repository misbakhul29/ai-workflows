# Game Genre Patterns

## Endless Runner

```js
// Scrolling background speed ramps over time
state.speed = 200 + state.score * 0.5;

// Spawn obstacles at right edge
function spawnObstacle() {
  obstacles.push({ x: W, y: H - 60, w: 30, h: 60 + Math.random()*80 });
}
// Spawn timer
spawnTimer -= dt;
if (spawnTimer <= 0) { spawnObstacle(); spawnTimer = 0.8 + Math.random()*0.6 - state.score*0.0002; }

// Move all obstacles left
obstacles.forEach(o => o.x -= state.speed * dt);
obstacles = obstacles.filter(o => o.x + o.w > 0);

// Player jump
if (keys[' '] && player.grounded) { player.vy = -600; player.grounded = false; }
```

## Top-Down Shooter

```js
// 8-directional movement
const dx = (keys['ArrowRight'] ? 1 : 0) - (keys['ArrowLeft'] ? 1 : 0);
const dy = (keys['ArrowDown']  ? 1 : 0) - (keys['ArrowUp']   ? 1 : 0);
const len = Math.hypot(dx, dy) || 1;
player.x += (dx/len) * player.speed * dt;
player.y += (dy/len) * player.speed * dt;

// Shoot toward mouse
if (mouse.down && shootCooldown <= 0) {
  const angle = Math.atan2(mouse.y - player.y, mouse.x - player.x);
  bullets.push({ x: player.x, y: player.y,
                 vx: Math.cos(angle)*500, vy: Math.sin(angle)*500, r: 4 });
  shootCooldown = 0.15;
}
shootCooldown -= dt;

// Enemy AI: move toward player
enemies.forEach(e => {
  const angle = Math.atan2(player.y - e.y, player.x - e.x);
  e.x += Math.cos(angle) * e.speed * dt;
  e.y += Math.sin(angle) * e.speed * dt;
});
```

## Platformer

```js
const GRAVITY = 900;
const JUMP_VEL = -500;

function updatePlayer(dt) {
  // Horizontal
  player.vx = 0;
  if (keys['ArrowLeft'])  player.vx = -220;
  if (keys['ArrowRight']) player.vx =  220;

  // Jump
  if ((keys['ArrowUp'] || keys[' ']) && player.grounded) {
    player.vy = JUMP_VEL;
    player.grounded = false;
  }

  // Gravity
  player.vy += GRAVITY * dt;
  player.x += player.vx * dt;
  player.y += player.vy * dt;

  // Platform collisions
  player.grounded = false;
  for (const plat of platforms) {
    if (aabb(player, plat) && player.vy > 0 &&
        player.y + player.h - player.vy*dt <= plat.y + 2) {
      player.y = plat.y - player.h;
      player.vy = 0;
      player.grounded = true;
    }
  }

  // Floor
  if (player.y + player.h > H) { player.y = H - player.h; player.vy = 0; player.grounded = true; }
}
```

## Match-3 / Puzzle Grid

```js
const COLS = 8, ROWS = 8, CELL = 60;
const COLORS = ['#e74', '#4af', '#8f4', '#f80', '#a4f'];

// Initialize grid
const grid = Array.from({ length: ROWS }, () =>
  Array.from({ length: COLS }, () => Math.floor(Math.random() * COLORS.length))
);

// Check matches
function findMatches() {
  const matched = new Set();
  // Horizontal
  for (let r = 0; r < ROWS; r++)
    for (let c = 0; c < COLS - 2; c++)
      if (grid[r][c] === grid[r][c+1] && grid[r][c] === grid[r][c+2])
        [c, c+1, c+2].forEach(i => matched.add(`${r},${i}`));
  // Vertical
  for (let c = 0; c < COLS; c++)
    for (let r = 0; r < ROWS - 2; r++)
      if (grid[r][c] === grid[r+1][c] && grid[r][c] === grid[r+2][c])
        [r, r+1, r+2].forEach(i => matched.add(`${i},${c}`));
  return matched;
}
```

## Snake

```js
let snake = [{ x: 10, y: 10 }];
let dir = { x: 1, y: 0 };
let food = spawnFood();
let moveTimer = 0;
const MOVE_INTERVAL = 0.12;

function updateSnake(dt) {
  if (keys['ArrowUp']    && dir.y === 0) dir = { x: 0, y: -1 };
  if (keys['ArrowDown']  && dir.y === 0) dir = { x: 0, y:  1 };
  if (keys['ArrowLeft']  && dir.x === 0) dir = { x:-1, y:  0 };
  if (keys['ArrowRight'] && dir.x === 0) dir = { x: 1, y:  0 };

  moveTimer += dt;
  if (moveTimer >= MOVE_INTERVAL) {
    moveTimer -= MOVE_INTERVAL;
    const head = { x: snake[0].x + dir.x, y: snake[0].y + dir.y };
    // Wall wrap
    head.x = (head.x + COLS) % COLS;
    head.y = (head.y + ROWS) % ROWS;
    // Self collision
    if (snake.some(s => s.x === head.x && s.y === head.y)) return gameOver();
    snake.unshift(head);
    if (head.x === food.x && head.y === food.y) {
      state.score += 10;
      food = spawnFood();
    } else snake.pop();
  }
}
```
