# WebGL & Three.js Game Patterns

## When to Use WebGL vs Canvas

- **Canvas 2D** — default for most games (platformers, top-down, puzzle, snake)
- **WebGL raw** — only for custom shaders or maximum performance 2D
- **Three.js** — 3D games, 3D UI, particle-heavy scenes; always prefer over raw WebGL

---

## Three.js Setup (HTML file)

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

const scene  = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(0, 5, 10);
camera.lookAt(0, 0, 0);

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

// Game loop
let lastTime = 0;
function loop(timestamp) {
  const dt = Math.min((timestamp - lastTime) / 1000, 0.05);
  lastTime = timestamp;
  update(dt);
  renderer.render(scene, camera);
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
</script>
```

---

## Three.js Setup (React artifact)

```jsx
import { useEffect, useRef } from 'react';
import * as THREE from 'three'; // available via CDN; import map not needed in artifacts

function Game3D() {
  const mountRef = useRef(null);

  useEffect(() => {
    const mount = mountRef.current;
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(mount.clientWidth, mount.clientHeight);
    mount.appendChild(renderer.domElement);

    const scene  = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(60, mount.clientWidth / mount.clientHeight, 0.1, 500);
    camera.position.set(0, 5, 10);
    camera.lookAt(0, 0, 0);

    let animId;
    let lastTime = 0;
    function loop(ts) {
      const dt = Math.min((ts - lastTime) / 1000, 0.05);
      lastTime = ts;
      // update(dt, scene, camera)
      renderer.render(scene, camera);
      animId = requestAnimationFrame(loop);
    }
    animId = requestAnimationFrame(loop);

    return () => {
      cancelAnimationFrame(animId);
      renderer.dispose();
      mount.removeChild(renderer.domElement);
    };
  }, []);

  return <div ref={mountRef} style={{ width: '100%', height: '100vh' }} />;
}
```

---

## Common 3D Game Objects

```js
// Box / cube
const box = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1, 1),
  new THREE.MeshStandardMaterial({ color: 0x4488ff })
);
scene.add(box);

// Sphere
const sphere = new THREE.Mesh(
  new THREE.SphereGeometry(0.5, 32, 32),
  new THREE.MeshStandardMaterial({ color: 0xff4444, roughness: 0.4 })
);
scene.add(sphere);

// Ground plane
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(50, 50),
  new THREE.MeshStandardMaterial({ color: 0x228822 })
);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

// Lighting (always add both ambient + directional for visible 3D)
scene.add(new THREE.AmbientLight(0xffffff, 0.4));
const sun = new THREE.DirectionalLight(0xffffff, 0.8);
sun.position.set(5, 10, 5);
sun.castShadow = true;
scene.add(sun);
```

---

## 3D Movement & Camera Follow

```js
// WASD movement for a player mesh
function updatePlayer3D(player, keys, dt) {
  const speed = 6;
  const dir = new THREE.Vector3();
  if (keys['w'] || keys['ArrowUp'])    dir.z -= 1;
  if (keys['s'] || keys['ArrowDown'])  dir.z += 1;
  if (keys['a'] || keys['ArrowLeft'])  dir.x -= 1;
  if (keys['d'] || keys['ArrowRight']) dir.x += 1;
  if (dir.length() > 0) dir.normalize().multiplyScalar(speed * dt);
  player.position.add(dir);
}

// Smooth camera follow (top-down / third-person)
function followCamera(camera, target, offset = new THREE.Vector3(0, 8, 10)) {
  const goal = target.position.clone().add(offset);
  camera.position.lerp(goal, 0.1);
  camera.lookAt(target.position);
}
```

---

## Raw WebGL: Minimal Shader Setup

Use only when Three.js is too heavy or you need a custom full-screen shader effect.

```html
<canvas id="c"></canvas>
<script>
const canvas = document.getElementById('c');
const gl = canvas.getContext('webgl');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
gl.viewport(0, 0, canvas.width, canvas.height);

const vert = `
  attribute vec2 a_pos;
  void main() { gl_Position = vec4(a_pos, 0.0, 1.0); }
`;
const frag = `
  precision mediump float;
  uniform float u_time;
  uniform vec2  u_res;
  void main() {
    vec2 uv = gl_FragCoord.xy / u_res;
    gl_FragColor = vec4(uv, 0.5 + 0.5*sin(u_time), 1.0);
  }
`;

function makeShader(type, src) {
  const s = gl.createShader(type);
  gl.shaderSource(s, src); gl.compileShader(s);
  return s;
}
const prog = gl.createProgram();
gl.attachShader(prog, makeShader(gl.VERTEX_SHADER, vert));
gl.attachShader(prog, makeShader(gl.FRAGMENT_SHADER, frag));
gl.linkProgram(prog);
gl.useProgram(prog);

// Full-screen quad
const buf = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buf);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1,-1, 1,-1, -1,1, 1,1]), gl.STATIC_DRAW);
const loc = gl.getAttribLocation(prog, 'a_pos');
gl.enableVertexAttribArray(loc);
gl.vertexAttribPointer(loc, 2, gl.FLOAT, false, 0, 0);

const uTime = gl.getUniformLocation(prog, 'u_time');
const uRes  = gl.getUniformLocation(prog, 'u_res');
gl.uniform2f(uRes, canvas.width, canvas.height);

function loop(t) {
  gl.uniform1f(uTime, t / 1000);
  gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
</script>
```

---

## 3D Collision (simple sphere vs sphere, sphere vs AABB)

```js
// Sphere vs sphere (use .position + bounding radius)
function sphereCollide(a, ra, b, rb) {
  return a.position.distanceTo(b.position) < ra + rb;
}

// Point inside axis-aligned box
function pointInBox(point, boxMin, boxMax) {
  return point.x > boxMin.x && point.x < boxMax.x &&
         point.y > boxMin.y && point.y < boxMax.y &&
         point.z > boxMin.z && point.z < boxMax.z;
}

// Three.js built-in (preferred for complex scenes)
const box3 = new THREE.Box3().setFromObject(mesh);
const sphere3 = new THREE.Sphere(center, radius);
const hit = box3.intersectsSphere(sphere3);
```

---

## 3D Particles (Three.js Points)

```js
function createParticleSystem(count = 500) {
  const positions = new Float32Array(count * 3);
  for (let i = 0; i < count * 3; i++) positions[i] = (Math.random() - 0.5) * 20;

  const geo = new THREE.BufferGeometry();
  geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));

  const mat = new THREE.PointsMaterial({ color: 0xffffff, size: 0.1 });
  const points = new THREE.Points(geo, mat);
  scene.add(points);
  return { points, positions, geo };
}

// Animate particles (update positions each frame)
function updateParticles3D({ positions, geo }, dt) {
  for (let i = 1; i < positions.length; i += 3) {
    positions[i] -= 2 * dt; // fall down
    if (positions[i] < -10) positions[i] = 10;
  }
  geo.attributes.position.needsUpdate = true;
}
```

---

## Three.js Version Notes (r128 — available on cdnjs)

- `THREE.OrbitControls` — **not bundled**; skip or implement manual orbit
- `THREE.CapsuleGeometry` — **not available** (added r142); use `CylinderGeometry` + `SphereGeometry` instead
- `GLTFLoader` — available as separate script: `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/examples/js/loaders/GLTFLoader.min.js`
- Prefer `MeshStandardMaterial` over `MeshBasicMaterial` for lit scenes

```js
// Capsule substitute (cylinder body + two sphere caps)
function makeCapsule(radiusTop, radiusBottom, height, color) {
  const group = new THREE.Group();
  group.add(new THREE.Mesh(new THREE.CylinderGeometry(radiusTop, radiusBottom, height, 16),
                            new THREE.MeshStandardMaterial({ color })));
  const cap = new THREE.SphereGeometry(radiusTop, 16, 8);
  const mat = new THREE.MeshStandardMaterial({ color });
  const top = new THREE.Mesh(cap, mat); top.position.y =  height / 2; group.add(top);
  const bot = new THREE.Mesh(cap, mat); bot.position.y = -height / 2; group.add(bot);
  return group;
}
```
