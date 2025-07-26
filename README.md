```markdown
# Pong Game (HTML, CSS, JavaScript)

Below are the complete files for a simple Pong game. The left paddle is controlled by your mouse, and the right paddle is controlled by a basic AI. The ball bounces off paddles and walls, and the game includes collision detection.

---

## 1. `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Pong Game</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Pong Game</h1>
    <canvas id="pong" width="800" height="500"></canvas>
    <script src="pong.js"></script>
</body>
</html>
```

---

## 2. `style.css`

```css
body {
    background: #181818;
    color: #fff;
    font-family: 'Segoe UI', Arial, sans-serif;
    text-align: center;
    margin: 0;
    padding: 0;
}

h1 {
    margin-top: 30px;
    margin-bottom: 10px;
    font-size: 2.5em;
    letter-spacing: 2px;
}

#pong {
    background: #222;
    display: block;
    margin: 0 auto;
    border: 4px solid #fff;
    border-radius: 10px;
    box-shadow: 0 0 40px #000a;
}
```

---

## 3. `pong.js`

```javascript
const canvas = document.getElementById('pong');
const ctx = canvas.getContext('2d');

// Game constants
const PADDLE_WIDTH = 16;
const PADDLE_HEIGHT = 100;
const BALL_RADIUS = 12;
const PLAYER_X = 30;
const AI_X = canvas.width - 30 - PADDLE_WIDTH;
const PADDLE_SPEED = 7;
const AI_REACTION = 0.09; // Lower is harder

// Game state
let playerY = (canvas.height - PADDLE_HEIGHT) / 2;
let aiY = (canvas.height - PADDLE_HEIGHT) / 2;
let ball = {
    x: canvas.width / 2,
    y: canvas.height / 2,
    vx: Math.random() > 0.5 ? 6 : -6,
    vy: (Math.random() - 0.5) * 8,
    speed: 7
};
let playerScore = 0;
let aiScore = 0;

// Mouse control
canvas.addEventListener('mousemove', function(e) {
    const rect = canvas.getBoundingClientRect();
    let mouseY = e.clientY - rect.top;
    playerY = mouseY - PADDLE_HEIGHT / 2;
    // Clamp paddle within canvas
    playerY = Math.max(0, Math.min(canvas.height - PADDLE_HEIGHT, playerY));
});

// Draw functions
function drawRect(x, y, w, h, color) {
    ctx.fillStyle = color;
    ctx.fillRect(x, y, w, h);
}

function drawCircle(x, y, r, color) {
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.arc(x, y, r, 0, Math.PI * 2, false);
    ctx.closePath();
    ctx.fill();
}

function drawText(text, x, y, size = 48) {
    ctx.fillStyle = "#fff";
    ctx.font = `${size}px Segoe UI, Arial, sans-serif`;
    ctx.textAlign = "center";
    ctx.fillText(text, x, y);
}

function drawNet() {
    ctx.strokeStyle = "#fff5";
    ctx.lineWidth = 4;
    for (let i = 0; i < canvas.height; i += 30) {
        ctx.beginPath();
        ctx.moveTo(canvas.width / 2, i);
        ctx.lineTo(canvas.width / 2, i + 16);
        ctx.stroke();
    }
}

// Collision detection
function collide(ball, paddleX, paddleY) {
    return (
        ball.x + BALL_RADIUS > paddleX &&
        ball.x - BALL_RADIUS < paddleX + PADDLE_WIDTH &&
        ball.y + BALL_RADIUS > paddleY &&
        ball.y - BALL_RADIUS < paddleY + PADDLE_HEIGHT
    );
}

// Reset ball after score
function resetBall(direction) {
    ball.x = canvas.width / 2;
    ball.y = canvas.height / 2;
    ball.vx = direction * (6 + Math.random() * 2);
    ball.vy = (Math.random() - 0.5) * 8;
    ball.speed = 7;
}

// Main game loop
function update() {
    // Move ball
    ball.x += ball.vx;
    ball.y += ball.vy;

    // Top and bottom wall collision
    if (ball.y - BALL_RADIUS < 0) {
        ball.y = BALL_RADIUS;
        ball.vy *= -1;
    }
    if (ball.y + BALL_RADIUS > canvas.height) {
        ball.y = canvas.height - BALL_RADIUS;
        ball.vy *= -1;
    }

    // Player paddle collision
    if (collide(ball, PLAYER_X, playerY)) {
        ball.x = PLAYER_X + PADDLE_WIDTH + BALL_RADIUS;
        let collidePoint = (ball.y - (playerY + PADDLE_HEIGHT / 2)) / (PADDLE_HEIGHT / 2);
        let angle = collidePoint * (Math.PI / 4); // Max 45deg
        let direction = 1;
        ball.vx = direction * ball.speed * Math.cos(angle);
        ball.vy = ball.speed * Math.sin(angle);
        ball.speed += 0.5;
    }

    // AI paddle collision
    if (collide(ball, AI_X, aiY)) {
        ball.x = AI_X - BALL_RADIUS;
        let collidePoint = (ball.y - (aiY + PADDLE_HEIGHT / 2)) / (PADDLE_HEIGHT / 2);
        let angle = collidePoint * (Math.PI / 4);
        let direction = -1;
        ball.vx = direction * ball.speed * Math.cos(angle);
        ball.vy = ball.speed * Math.sin(angle);
        ball.speed += 0.5;
    }

    // Score
    if (ball.x - BALL_RADIUS < 0) {
        aiScore++;
        resetBall(1);
    }
    if (ball.x + BALL_RADIUS > canvas.width) {
        playerScore++;
        resetBall(-1);
    }

    // AI movement (tracks ball with some delay)
    let aiCenter = aiY + PADDLE_HEIGHT / 2;
    if (aiCenter < ball.y - 10) {
        aiY += PADDLE_SPEED * AI_REACTION * Math.abs(ball.vx);
    } else if (aiCenter > ball.y + 10) {
        aiY -= PADDLE_SPEED * AI_REACTION * Math.abs(ball.vx);
    }
    // Clamp AI paddle
    aiY = Math.max(0, Math.min(canvas.height - PADDLE_HEIGHT, aiY));
}

function render() {
    // Clear
    drawRect(0, 0, canvas.width, canvas.height, "#222");
    drawNet();

    // Draw paddles
    drawRect(PLAYER_X, playerY, PADDLE_WIDTH, PADDLE_HEIGHT, "#0ff");
    drawRect(AI_X, aiY, PADDLE_WIDTH, PADDLE_HEIGHT, "#f0f");

    // Draw ball
    drawCircle(ball.x, ball.y, BALL_RADIUS, "#fff");

    // Draw scores
    drawText(playerScore, canvas.width / 4, 60, 48);
    drawText(aiScore, 3 * canvas.width / 4, 60, 48);
}

function gameLoop() {
    update();
    render();
    requestAnimationFrame(gameLoop);
}

gameLoop();
```
```
