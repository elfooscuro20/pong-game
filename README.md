const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const playerScoreEl = document.getElementById("playerScore");
const computerScoreEl = document.getElementById("computerScore");

const paddleWidth = 14;
const paddleHeight = 120;
const paddleSpeed = 7;

const leftPaddle = {
  x: 20,
  y: canvas.height / 2 - paddleHeight / 2,
  width: paddleWidth,
  height: paddleHeight,
  speed: paddleSpeed
};

const rightPaddle = {
  x: canvas.width - 20 - paddleWidth,
  y: canvas.height / 2 - paddleHeight / 2,
  width: paddleWidth,
  height: paddleHeight,
  speed: 5.2
};

const ball = {
  x: canvas.width / 2,
  y: canvas.height / 2,
  radius: 10,
  speed: 6,
  vx: 5,
  vy: 3
};

const trail = [];
let screenFlash = 0;

let playerScore = 0;
let computerScore = 0;

let mouseY = canvas.height / 2;
let keys = {
  ArrowUp: false,
  ArrowDown: false
};

function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max);
}

function addTrail() {
  trail.push({ x: ball.x, y: ball.y, life: 1 });
  if (trail.length > 18) trail.shift();
}

function resetBall(direction) {
  ball.x = canvas.width / 2;
  ball.y = canvas.height / 2;
  const angle = (Math.random() - 0.5) * 1.2;
  const speed = ball.speed;

  ball.vx = direction * speed * (0.8 + Math.random() * 0.5);
  ball.vy = Math.sin(angle) * speed * 1.2;
}

function updateScores() {
  playerScoreEl.textContent = playerScore;
  computerScoreEl.textContent = computerScore;
}

function updatePaddles() {
  if (keys.ArrowUp) {
    leftPaddle.y -= leftPaddle.speed;
  }
  if (keys.ArrowDown) {
    leftPaddle.y += leftPaddle.speed;
  }

  const mouseTarget = mouseY - leftPaddle.height / 2;
  if (!keys.ArrowUp && !keys.ArrowDown) {
    leftPaddle.y += (mouseTarget - leftPaddle.y) * 0.22;
  }

  leftPaddle.y = clamp(leftPaddle.y, 0, canvas.height - leftPaddle.height);

  const targetY = ball.y - rightPaddle.height / 2;
  rightPaddle.y += (targetY - rightPaddle.y) * 0.12;
  rightPaddle.y = clamp(rightPaddle.y, 0, canvas.height - rightPaddle.height);
}

function updateBall() {
  ball.x += ball.vx;
  ball.y += ball.vy;
  addTrail();

  if (ball.y - ball.radius <= 0) {
    ball.y = ball.radius;
    ball.vy *= -1;
    screenFlash = 0.14;
  } else if (ball.y + ball.radius >= canvas.height) {
    ball.y = canvas.height - ball.radius;
    ball.vy *= -1;
    screenFlash = 0.14;
  }

  if (
    ball.x - ball.radius <= leftPaddle.x + leftPaddle.width &&
    ball.y >= leftPaddle.y &&
    ball.y <= leftPaddle.y + leftPaddle.height &&
    ball.x > leftPaddle.x &&
    ball.vx < 0
  ) {
    ball.x = leftPaddle.x + leftPaddle.width + ball.radius;
    const offset = (ball.y - (leftPaddle.y + leftPaddle.height / 2)) / (leftPaddle.height / 2);
    ball.vx = Math.abs(ball.vx) + 0.4;
    ball.vy = offset * 8;
    screenFlash = 0.18;
  }

  if (
    ball.x + ball.radius >= rightPaddle.x &&
    ball.y >= rightPaddle.y &&
    ball.y <= rightPaddle.y + rightPaddle.height &&
    ball.x < rightPaddle.x + rightPaddle.width &&
    ball.vx > 0
  ) {
    ball.x = rightPaddle.x - ball.radius;
    const offset = (ball.y - (rightPaddle.y + rightPaddle.height / 2)) / (rightPaddle.height / 2);
    ball.vx = -Math.abs(ball.vx) - 0.4;
    ball.vy = offset * 8;
    screenFlash = 0.18;
  }

  if (ball.x - ball.radius < 0) {
    computerScore++;
    updateScores();
    resetBall(1);
    screenFlash = 0.25;
  } else if (ball.x + ball.radius > canvas.width) {
    playerScore++;
    updateScores();
    resetBall(-1);
    screenFlash = 0.25;
  }

  if (screenFlash > 0) screenFlash -= 0.02;
}

function drawRect(x, y, w, h, color) {
  ctx.fillStyle = color;
  ctx.fillRect(x, y, w, h);
}

function drawBall(x, y, radius, color) {
  ctx.fillStyle = color;
  ctx.beginPath();
  ctx.arc(x, y, radius, 0, Math.PI * 2);
  ctx.fill();
}

function drawTrail() {
  for (let i = 0; i < trail.length; i++) {
    const p = trail[i];
    const alpha = p.life * 0.55;
    ctx.fillStyle = `rgba(126, 231, 135, ${alpha})`;
    ctx.beginPath();
    ctx.arc(p.x, p.y, ball.radius * (0.4 + i / trail.length), 0, Math.PI * 2);
    ctx.fill();
    p.life -= 0.08;
  }

  for (let i = trail.length - 1; i >= 0; i--) {
    if (trail[i].life <= 0) trail.splice(i, 1);
  }
}

function drawBoard() {
  ctx.fillStyle = "#000";
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.shadowColor = "rgba(94, 234, 212, 0.8)";
  ctx.shadowBlur = 10;
  ctx.strokeStyle = "rgba(218, 240, 255, 0.8)";
  ctx.setLineDash([15, 15]);
  ctx.beginPath();
  ctx.moveTo(canvas.width / 2, 0);
  ctx.lineTo(canvas.width / 2, canvas.height);
  ctx.stroke();
  ctx.setLineDash([]);
  ctx.shadowBlur = 0;

  drawTrail();

  ctx.shadowColor = "rgba(126, 231, 135, 0.9)";
  ctx.shadowBlur = 18;
  drawRect(leftPaddle.x, leftPaddle.y, leftPaddle.width, leftPaddle.height, "#f0f6fc");
  drawRect(rightPaddle.x, rightPaddle.y, rightPaddle.width, rightPaddle.height, "#f0f6fc");
  drawBall(ball.x, ball.y, ball.radius, "#7ee787");
  ctx.shadowBlur = 0;

  if (screenFlash > 0) {
    ctx.fillStyle = `rgba(126, 231, 135, ${screenFlash * 0.3})`;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  }
}

function gameLoop() {
  updatePaddles();
  updateBall();
  drawBoard();
  requestAnimationFrame(gameLoop);
}

document.addEventListener("keydown", (event) => {
  if (event.key === "ArrowUp") keys.ArrowUp = true;
  if (event.key === "ArrowDown") keys.ArrowDown = true;
});

document.addEventListener("keyup", (event) => {
  if (event.key === "ArrowUp") keys.ArrowUp = false;
  if (event.key === "ArrowDown") keys.ArrowDown = false;
});

canvas.addEventListener("mousemove", (event) => {
  const rect = canvas.getBoundingClientRect();
  const scaleY = canvas.height / rect.height;
  mouseY = (event.clientY - rect.top) * scaleY;
});

updateScores();
resetBall(1);
requestAnimationFrame(gameLoop);
