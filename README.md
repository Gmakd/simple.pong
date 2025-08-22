# simple.pong
Pong
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Pong Game</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div id="pong-container">
        <canvas id="pong" width="800" height="500"></canvas>
        <div id="score">
            <span id="player-score">0</span> : <span id="ai-score">0</span>
        </div>
    </div>
    <script src="game.js"></script>
</body>
</html>
body {
    background: #222;
    color: #fff;
    font-family: 'Segoe UI', Arial, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
}

#pong-container {
    display: flex;
    flex-direction: column;
    align-items: center;
}

#pong {
    background: #000;
    border: 4px solid #fff;
    display: block;
}

#score {
    margin-top: 16px;
    font-size: 1.5em;
    letter-spacing: 0.2em;
    font-weight: bold;
}
const canvas = document.getElementById('pong');
const ctx = canvas.getContext('2d');

// Game settings
const PADDLE_WIDTH = 12;
const PADDLE_HEIGHT = 90;
const BALL_SIZE = 16;
const PLAYER_X = 20;
const AI_X = canvas.width - PADDLE_WIDTH - 20;
const PADDLE_SPEED = 8;
const AI_SPEED = 5;

// Game state
let playerY = (canvas.height - PADDLE_HEIGHT) / 2;
let aiY = (canvas.height - PADDLE_HEIGHT) / 2;
let ballX = canvas.width / 2 - BALL_SIZE / 2;
let ballY = canvas.height / 2 - BALL_SIZE / 2;
let ballSpeedX = 6 * (Math.random() > 0.5 ? 1 : -1);
let ballSpeedY = (Math.random() * 4 + 2) * (Math.random() > 0.5 ? 1 : -1);

let playerScore = 0;
let aiScore = 0;

// Mouse control for player paddle
canvas.addEventListener('mousemove', (e) => {
    const rect = canvas.getBoundingClientRect();
    const mouseY = e.clientY - rect.top;
    playerY = mouseY - PADDLE_HEIGHT / 2;

    // Clamp paddle within canvas
    if (playerY < 0) playerY = 0;
    if (playerY + PADDLE_HEIGHT > canvas.height) playerY = canvas.height - PADDLE_HEIGHT;
});

function resetBall(direction = 1) {
    ballX = canvas.width / 2 - BALL_SIZE / 2;
    ballY = canvas.height / 2 - BALL_SIZE / 2;
    ballSpeedX = 6 * direction;
    ballSpeedY = (Math.random() * 4 + 2) * (Math.random() > 0.5 ? 1 : -1);
}

function drawRect(x, y, w, h, color = '#fff') {
    ctx.fillStyle = color;
    ctx.fillRect(x, y, w, h);
}

function drawCircle(x, y, r, color = '#fff') {
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.arc(x, y, r, 0, Math.PI * 2);
    ctx.closePath();
    ctx.fill();
}

function drawNet() {
    ctx.strokeStyle = '#fff';
    ctx.beginPath();
    for (let i = 0; i < canvas.height; i += 30) {
        ctx.moveTo(canvas.width / 2, i);
        ctx.lineTo(canvas.width / 2, i + 18);
    }
    ctx.stroke();
}

function drawEverything() {
    // Clear canvas
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    drawNet();

    // Draw paddles
    drawRect(PLAYER_X, playerY, PADDLE_WIDTH, PADDLE_HEIGHT, '#0ff');
    drawRect(AI_X, aiY, PADDLE_WIDTH, PADDLE_HEIGHT, '#f00');

    // Draw ball
    drawCircle(ballX + BALL_SIZE / 2, ballY + BALL_SIZE / 2, BALL_SIZE / 2, '#fff');
}

function updateAI() {
    // AI tries to follow the ball's center
    const aiCenter = aiY + PADDLE_HEIGHT / 2;
    const ballCenter = ballY + BALL_SIZE / 2;
    if (aiCenter < ballCenter - 12)
        aiY += AI_SPEED;
    else if (aiCenter > ballCenter + 12)
        aiY -= AI_SPEED;

    // Clamp AI paddle
    if (aiY < 0) aiY = 0;
    if (aiY + PADDLE_HEIGHT > canvas.height) aiY = canvas.height - PADDLE_HEIGHT;
}

function updateBall() {
    ballX += ballSpeedX;
    ballY += ballSpeedY;

    // Top and bottom wall collision
    if (ballY < 0) {
        ballY = 0;
        ballSpeedY *= -1;
    }
    if (ballY + BALL_SIZE > canvas.height) {
        ballY = canvas.height - BALL_SIZE;
        ballSpeedY *= -1;
    }

    // Player paddle collision
    if (
        ballX < PLAYER_X + PADDLE_WIDTH &&
        ballY + BALL_SIZE > playerY &&
        ballY < playerY + PADDLE_HEIGHT
    ) {
        ballX = PLAYER_X + PADDLE_WIDTH;
        ballSpeedX *= -1;

        // Add some "spin" based on where it hit the paddle
        let hitPos = (ballY + BALL_SIZE / 2) - (playerY + PADDLE_HEIGHT / 2);
        ballSpeedY = hitPos * 0.25;
    }

    // AI paddle collision
    if (
        ballX + BALL_SIZE > AI_X &&
        ballY + BALL_SIZE > aiY &&
        ballY < aiY + PADDLE_HEIGHT
    ) {
        ballX = AI_X - BALL_SIZE;
        ballSpeedX *= -1;

        let hitPos = (ballY + BALL_SIZE / 2) - (aiY + PADDLE_HEIGHT / 2);
        ballSpeedY = hitPos * 0.25;
    }

    // Left & right wall: score
    if (ballX < 0) {
        aiScore++;
        document.getElementById('ai-score').textContent = aiScore;
        resetBall(1);
    }
    if (ballX + BALL_SIZE > canvas.width) {
        playerScore++;
        document.getElementById('player-score').textContent = playerScore;
        resetBall(-1);
    }
}

function gameLoop() {
    updateAI();
    updateBall();
    drawEverything();
    requestAnimationFrame(gameLoop);
}

// Initialize scores
document.getElementById('player-score').textContent = playerScore;
document.getElementById('ai-score').textContent = aiScore;

gameLoop();
