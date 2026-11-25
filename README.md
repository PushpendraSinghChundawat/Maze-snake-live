<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Live Maze Snake</title>
<style>
  body {
    margin: 0;
    background-color: #050505;
    color: #fff;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    min-height: 100vh;
    user-select: none;
    overflow-x: hidden;
  }
  .header {
    width: 95%;
    max-width: 400px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 12px;
  }
  .level-badge {
    border: 2px solid #d4af37;
    color: #ffd700;
    padding: 4px 14px;
    border-radius: 20px;
    font-weight: 900;
    font-size: 16px;
  }
  .likes-bar {
    background: #1a1a1a;
    border: 1px solid #444;
    padding: 5px 12px;
    border-radius: 15px;
    font-size: 12px;
    font-weight: bold;
    color: #ffeb3b;
  }
  .gifts-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 6px;
    width: 95%;
    max-width: 400px;
    margin: 10px 0;
  }
  .gift-btn {
    background: #161616;
    border: 1px solid #333;
    border-radius: 8px;
    padding: 6px 4px;
    text-align: center;
    cursor: pointer;
    transition: 0.1s;
  }
  .gift-btn:active {
    transform: scale(0.95);
    border-color: #ffd700;
  }
  .gift-icon { font-size: 20px; }
  .gift-title { font-size: 10px; font-weight: 800; color: #fff; margin-top: 2px; }
  .gift-sub { font-size: 9px; color: #ff5252; font-weight: bold; }

  #game-container {
    position: relative;
    border: 4px solid #3a2a0d;
    border-radius: 10px;
    background: #0a0804;
    box-shadow: 0 0 20px rgba(212, 175, 55, 0.2);
  }
  canvas {
    display: block;
  }
  .status-msg {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(0, 0, 0, 0.85);
    border: 2px solid #ffd700;
    padding: 12px 20px;
    border-radius: 10px;
    font-size: 20px;
    font-weight: bold;
    color: #ffd700;
    display: none;
    text-align: center;
  }
</style>
</head>
<body>

  <div class="header">
    <div class="level-badge" id="levelDisplay">LEVEL 1</div>
    <div class="likes-bar" id="appleDisplay">🍎 APPLES LEFT: 5</div>
  </div>

  <div class="gifts-grid">
    <div class="gift-btn" onclick="changeSnakeColor()">
      <div class="gift-icon">⭐</div>
      <div class="gift-title">CHANGE COLOR</div>
      <div class="gift-sub">SNAKE</div>
    </div>
    <div class="gift-btn" onclick="changeAppleColor()">
      <div class="gift-icon">🦴</div>
      <div class="gift-title">CHANGE COLOR</div>
      <div class="gift-sub">APPLES</div>
    </div>
    <div class="gift-btn" onclick="spawnApples(5)">
      <div class="gift-icon">🪙</div>
      <div class="gift-title">SEND +5</div>
      <div class="gift-sub">APPLES 🍎</div>
    </div>
    <div class="gift-btn" onclick="triggerFart()">
      <div class="gift-icon">💨</div>
      <div class="gift-title">FART</div>
      <div class="gift-sub">SMOKE</div>
    </div>
    <div class="gift-btn" onclick="triggerBomb()">
      <div class="gift-icon">💣</div>
      <div class="gift-title">x1 BOMB</div>
      <div class="gift-sub">SHRINK</div>
    </div>
    <div class="gift-btn" onclick="triggerTurbo()">
      <div class="gift-icon">⚡</div>
      <div class="gift-title">TURBO</div>
      <div class="gift-sub">SPEED</div>
    </div>
    <div class="gift-btn" onclick="triggerGameOver()">
      <div class="gift-icon">💀</div>
      <div class="gift-title">GAME OVER</div>
      <div class="gift-sub">RESET</div>
    </div>
    <div class="gift-btn" onclick="nextLevel()">
      <div class="gift-icon">🚀</div>
      <div class="gift-title">NEXT LEVEL</div>
      <div class="gift-sub">SKIP</div>
    </div>
  </div>

  <div id="game-container">
    <canvas id="mazeCanvas" width="360" height="360"></canvas>
    <div class="status-msg" id="statusMsg">LEVEL UP!</div>
  </div>

<script>
const canvas = document.getElementById("mazeCanvas");
const ctx = canvas.getContext("2d");

const cols = 15;
const rows = 15;
const cellSize = canvas.width / cols;

let grid = [];
let snake = [];
let apples = [];
let level = 1;
let snakeColors = ["#ff9800", "#00e676", "#00b0ff", "#ff4081", "#e040fb"];
let appleColors = ["#ff1744", "#00e5ff", "#76ff03", "#ffea00"];
let sColorIdx = 0;
let aColorIdx = 0;
let speed = 130;
let turboTimer = null;
let fartClouds = [];

class Cell {
  constructor(c, r) {
    this.c = c;
    this.r = r;
    this.walls = [true, true, true, true];
    this.visited = false;
  }
  show() {
    let x = this.c * cellSize;
    let y = this.r * cellSize;
    ctx.strokeStyle = "#4d3811";
    ctx.lineWidth = 4;
    ctx.lineCap = "round";

    if (this.walls[0]) { ctx.beginPath(); ctx.moveTo(x, y); ctx.lineTo(x + cellSize, y); ctx.stroke(); }
    if (this.walls[1]) { ctx.beginPath(); ctx.moveTo(x + cellSize, y); ctx.lineTo(x + cellSize, y + cellSize); ctx.stroke(); }
    if (this.walls[2]) { ctx.beginPath(); ctx.moveTo(x + cellSize, y + cellSize); ctx.lineTo(x, y + cellSize); ctx.stroke(); }
    if (this.walls[3]) { ctx.beginPath(); ctx.moveTo(x, y + cellSize); ctx.lineTo(x, y); ctx.stroke(); }
  }
}

function index(c, r) {
  if (c < 0 || r < 0 || c > cols - 1 || r > rows - 1) return -1;
  return c + r * cols;
}

function generateMaze() {
  grid = [];
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      grid.push(new Cell(c, r));
    }
  }
  let current = grid[0];
  let stack = [];
  current.visited = true;

  while (true) {
    let neighbors = [];
    let top = grid[index(current.c, current.r - 1)];
    let right = grid[index(current.c + 1, current.r)];
    let bottom = grid[index(current.c, current.r + 1)];
    let left = grid[index(current.c - 1, current.r)];

    if (top && !top.visited) neighbors.push(top);
    if (right && !right.visited) neighbors.push(right);
    if (bottom && !bottom.visited) neighbors.push(bottom);
    if (left && !left.visited) neighbors.push(left);

    if (neighbors.length > 0) {
      let next = neighbors[Math.floor(Math.random() * neighbors.length)];
      next.visited = true;
      stack.push(current);
      removeWalls(current, next);
      current = next;
    } else if (stack.length > 0) {
      current = stack.pop();
    } else {
      break;
    }
  }
}

function removeWalls(a, b) {
  let x = a.c - b.c;
  if (x === 1) { a.walls[3] = false; b.walls[1] = false; }
  else if (x === -1) { a.walls[1] = false; b.walls[3] = false; }
  let y = a.r - b.r;
  if (y === 1) { a.walls[0] = false; b.walls[2] = false; }
  else if (y === -1) { a.walls[2] = false; b.walls[0] = false; }
}

function initLevel() {
  generateMaze();
  snake = [{ c: 0, r: 0 }, { c: 0, r: 0 }, { c: 0, r: 0 }];
  apples = [];
  spawnApples(5 + Math.floor(level / 2));
  document.getElementById("levelDisplay").innerText = "LEVEL " + level;
}

function spawnApples(count) {
  for (let i = 0; i < count; i++) {
    let c = Math.floor(Math.random() * cols);
    let r = Math.floor(Math.random() * rows);
    apples.push({ c, r });
  }
  updateAppleCount();
}

function updateAppleCount() {
  document.getElementById("appleDisplay").innerText = "🍎 APPLES LEFT: " + apples.length;
}

function getNextStep() {
  if (apples.length === 0) return snake[0];
  let start = snake[0];
  let target = apples[0];

  let queue = [[start]];
  let visited = new Set([start.c + "," + start.r]);

  while (queue.length > 0) {
    let path = queue.shift();
    let curr = path[path.length - 1];

    if (curr.c === target.c && curr.r === target.r) {
      return path[1] || curr;
    }

    let cell = grid[index(curr.c, curr.r)];
    let moves = [
      { dc: 0, dr: -1, wall: 0 },
      { dc: 1, dr: 0, wall: 1 },
      { dc: 0, dr: 1, wall: 2 },
      { dc: -1, dr: 0, wall: 3 }
    ];

    for (let m of moves) {
      if (!cell.walls[m.wall]) {
        let nc = curr.c + m.dc;
        let nr = curr.r + m.dr;
        let key = nc + "," + nr;
        if (!visited.has(key)) {
          visited.add(key);
          queue.push([...path, { c: nc, r: nr }]);
        }
      }
    }
  }
  return start;
}

function update() {
  if (apples.length === 0) {
    nextLevel();
    return;
  }

  let next = getNextStep();
  snake.unshift(next);

  let ate = false;
  for (let i = 0; i < apples.length; i++) {
    if (apples[i].c === next.c && apples[i].r === next.r) {
      apples.splice(i, 1);
      ate = true;
      updateAppleCount();
      break;
    }
  }

  if (!ate) {
    snake.pop();
  }
}

function draw() {
  ctx.fillStyle = "#140f05";
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  for (let cell of grid) cell.show();

  for (let a of apples) {
    let x = a.c * cellSize + cellSize / 2;
    let y = a.r * cellSize + cellSize / 2;
    ctx.fillStyle = appleColors[aColorIdx];
    ctx.beginPath();
    ctx.arc(x, y, cellSize * 0.28, 0, Math.PI * 2);
    ctx.fill();
    ctx.fillStyle = "#4caf50";
    ctx.fillRect(x - 1, y - cellSize * 0.38, 3, 4);
  }

  for (let i = fartClouds.length - 1; i >= 0; i--) {
    let f = fartClouds[i];
    ctx.fillStyle = `rgba(118, 255, 3, ${f.alpha})`;
    ctx.beginPath();
    ctx.arc(f.x, f.y, f.r, 0, Math.PI * 2);
    ctx.fill();
    f.alpha -= 0.03;
    f.r += 0.5;
    if (f.alpha <= 0) fartClouds.splice(i, 1);
  }

  ctx.strokeStyle = snakeColors[sColorIdx];
  ctx.lineWidth = cellSize * 0.55;
  ctx.lineCap = "round";
  ctx.lineJoin = "round";
  ctx.beginPath();
  for (let i = 0; i < snake.length; i++) {
    let x = snake[i].c * cellSize + cellSize / 2;
    let y = snake[i].r * cellSize + cellSize / 2;
    if (i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  }
  ctx.stroke();

  let headX = snake[0].c * cellSize + cellSize / 2;
  let headY = snake[0].r * cellSize + cellSize / 2;
  ctx.fillStyle = "#fff";
  ctx.beginPath();
  ctx.arc(headX - 3, headY - 2, 3, 0, Math.PI * 2);
  
