
<!DOCTYPE html>
<html lang="fi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Retro Pong</title>
<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  background: #080d08;
  color: #39ff72;
  font-family: monospace;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

.game {
  width: 95%;
  max-width: 760px;
  text-align: center;
  padding: 15px;
}

h1 {
  font-size: clamp(30px, 7vw, 52px);
  letter-spacing: 8px;
  text-shadow: 0 0 12px #39ff72;
  margin: 5px 0 15px;
}

.score {
  display: flex;
  justify-content: space-around;
  font-size: 25px;
  margin: 12px 0;
}

canvas {
  width: 100%;
  aspect-ratio: 16 / 9;
  display: block;
  background: #000;
  border: 2px solid #39ff72;
  box-shadow: 0 0 15px #39ff7255;
  touch-action: none;
}

button {
  background: #39ff72;
  color: #001b08;
  border: none;
  padding: 12px 25px;
  margin: 8px;
  font-family: monospace;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
}

button:hover {
  background: white;
}

.info {
  color: #9bffaF;
  font-size: 13px;
  line-height: 1.7;
}

.overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #000e;
  padding: 15px;
}

.overlay h2 {
  font-size: clamp(22px, 5vw, 36px);
  text-shadow: 0 0 12px #39ff72;
}

.arena {
  position: relative;
}

.hidden { display: none; }

</style>
</head>
<body>

<div class="game">
  <h1>RETRO PONG</h1>

  <div class="score">
    <span>YOU: <b id="playerScore">0</b></span>
    <span>CPU: <b id="cpuScore">0</b></span>
  </div>

  <div class="arena">
    <canvas id="canvas" width="800" height="450"></canvas>

    <div class="overlay" id="menu">
      <h2>RETRO PONG</h2>
      <p>FIRST TO 5 WINS</p>
      <button id="startBtn">START GAME</button>
    </div>

    <div class="overlay hidden" id="endMenu">
      <h2 id="result">GAME OVER</h2>
      <button id="restartBtn">PLAY AGAIN</button>
    </div>
  </div>

  <p class="info">
    MOVE: W / S OR ↑ / ↓<br>
    YOU CAN ALSO USE YOUR MOUSE OR FINGER
  </p>
  <button id="resetBtn">RESTART</button>
</div>

<script>
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

const W = canvas.width;
const H = canvas.height;

const player = { x: 25, y: 180, w: 13, h: 90, score: 0 };
const cpu = { x: W - 38, y: 180, w: 13, h: 90, score: 0 };

const ball = {
  x: W / 2,
  y: H / 2,
  r: 9,
  vx: 5,
  vy: 3
};

let running = false;
let animation = null;
let keys = {};

const menu = document.getElementById("menu");
const endMenu = document.getElementById("endMenu");

function resetBall(direction = 1) {
  ball.x = W / 2;
  ball.y = H / 2;
  ball.vx = 5 * direction;
  ball.vy = (Math.random() * 4 - 2) || 2;
}

function resetGame() {
  player.score = 0;
  cpu.score = 0;
  player.y = H / 2 - player.h / 2;
  cpu.y = H / 2 - cpu.h / 2;

  updateScore();
  resetBall();
}

function updateScore() {
  document.getElementById("playerScore").textContent = player.score;
  document.getElementById("cpuScore").textContent = cpu.score;
}

function draw() {
  ctx.fillStyle = "#000";
  ctx.fillRect(0, 0, W, H);

  // Center line
  ctx.setLineDash([10, 12]);
  ctx.strokeStyle = "#39ff7255";
  ctx.beginPath();
  ctx.moveTo(W / 2, 0);
  ctx.lineTo(W / 2, H);
  ctx.stroke();
  ctx.setLineDash([]);

  // Paddles
  ctx.fillStyle = "#39ff72";
  ctx.shadowColor = "#39ff72";
  ctx.shadowBlur = 12;

  ctx.fillRect(player.x, player.y, player.w, player.h);
  ctx.fillRect(cpu.x, cpu.y, cpu.w, cpu.h);

  // Ball
  ctx.fillRect(ball.x - ball.r, ball.y - ball.r,
               ball.r * 2, ball.r * 2);

  ctx.shadowBlur = 0;
}

function update() {
  if (!running) return;

  // Player movement
  if (keys["ArrowUp"] || keys["w"]) player.y -= 7;
  if (keys["ArrowDown"] || keys["s"]) player.y += 7;

  player.y = Math.max(0, Math.min(H - player.h, player.y));

  // AI opponent
  const target = ball.y - cpu.h / 2;
  const difficulty = 4.2 + Math.min(cpu.score, 2) * 0.2;

  if (cpu.y + cpu.h / 2 < target - 8) cpu.y += difficulty;
  if (cpu.y + cpu.h / 2 > target + 8) cpu.y -= difficulty;

  cpu.y = Math.max(0, Math.min(H - cpu.h, cpu.y));

  // Ball movement
  ball.x += ball.vx;
  ball.y += ball.vy;

  // Top and bottom collision
  if (ball.y - ball.r < 0 || ball.y + ball.r > H) {
    ball.vy *= -1;
  }

  // Player collision
  if (
    ball.vx < 0 &&
    ball.x - ball.r < player.x + player.w &&
    ball.x > player.x &&
    ball.y > player.y &&
    ball.y < player.y + player.h
  ) {
    ball.vx = Math.abs(ball.vx) * 1.04;
    ball.vy += ((ball.y - (player.y + player.h / 2)) / 10);
  }

  // CPU collision
  if (
    ball.vx > 0 &&
    ball.x + ball.r > cpu.x &&
    ball.x < cpu.x + cpu.w &&
    ball.y > cpu.y &&
    ball.y < cpu.y + cpu.h
  ) {
    ball.vx = -Math.abs(ball.vx) * 1.04;
    ball.vy += ((ball.y - (cpu.y + cpu.h / 2)) / 10);
  }

  // Scoring
  if (ball.x < 0) {
    cpu.score++;
    pointScored();
    resetBall(-1);
  }

  if (ball.x > W) {
    player.score++;
    pointScored();
    resetBall(1);
  }
}

function pointScored() {
  updateScore();

  if (player.score >= 5 || cpu.score >= 5) {
    running = false;
    document.getElementById("result").textContent =
      player.score >= 5 ? "YOU WIN!" : "CPU WINS!";
    endMenu.classList.remove("hidden");
  }
}

function loop() {
  update();
  draw();

  if (running) {
    animation = requestAnimationFrame(loop);
  }
}

function startGame() {
  cancelAnimationFrame(animation);
  resetGame();
  menu.classList.add("hidden");
  endMenu.classList.add("hidden");
  running = true;
  loop();
}

document.getElementById("startBtn").onclick = startGame;
document.getElementById("restartBtn").onclick = startGame;
document.getElementById("resetBtn").onclick = startGame;

document.addEventListener("keydown", e => {
  const key = e.key.toLowerCase();
  if (["arrowup", "arrowdown", " "].includes(key)) {
    e.preventDefault();
  }
  keys[key] = true;
});

document.addEventListener("keyup", e => {
  keys[e.key.toLowerCase()] = false;
});

function movePlayer(e) {
  if (!running) return;

  const rect = canvas.getBoundingClientRect();
  const scaleY = H / rect.height;
  const y = (e.clientY - rect.top) * scaleY;

  player.y = Math.max(0, Math.min(H - player.h, y - player.h / 2));
}

canvas.addEventListener("pointermove", movePlayer);
canvas.addEventListener("pointerdown", movePlayer);

resetGame();
draw();
</script>

</body>
</html>
