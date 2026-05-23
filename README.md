<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pong Game</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            width: 100%;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            font-family: 'Arial', sans-serif;
            overflow: hidden;
        }

        .game-container {
            position: relative;
            width: 800px;
            height: 600px;
            background: #000;
            border: 3px solid #0ff;
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
        }

        /* Center line */
        .center-line {
            position: absolute;
            left: 50%;
            top: 0;
            width: 2px;
            height: 100%;
            background: repeating-linear-gradient(
                to bottom,
                #0ff 0px,
                #0ff 10px,
                transparent 10px,
                transparent 20px
            );
            transform: translateX(-50%);
        }

        /* Paddles */
        .paddle {
            position: absolute;
            width: 15px;
            height: 100px;
            background: #0ff;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 255, 255, 0.7);
        }

        .paddle-left {
            left: 20px;
        }

        .paddle-right {
            right: 20px;
        }

        /* Ball */
        .ball {
            position: absolute;
            width: 15px;
            height: 15px;
            background: #0ff;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(0, 255, 255, 0.9);
        }

        /* Scoreboard */
        .scoreboard {
            position: absolute;
            top: 20px;
            left: 0;
            right: 0;
            display: flex;
            justify-content: space-around;
            padding: 0 100px;
            font-size: 36px;
            font-weight: bold;
            color: #0ff;
            text-shadow: 0 0 10px rgba(0, 255, 255, 0.8);
            z-index: 10;
        }

        .score {
            min-width: 60px;
            text-align: center;
        }

        /* Game Over Overlay */
        .game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.9);
            padding: 40px;
            border-radius: 10px;
            text-align: center;
            display: none;
            z-index: 100;
            border: 2px solid #0ff;
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.8);
        }

        .game-over h2 {
            color: #0ff;
            font-size: 32px;
            margin-bottom: 20px;
            text-shadow: 0 0 10px rgba(0, 255, 255, 0.8);
        }

        .game-over p {
            color: #fff;
            font-size: 18px;
            margin-bottom: 30px;
        }

        .game-over button {
            background: #0ff;
            color: #000;
            border: none;
            padding: 12px 30px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 5px;
            cursor: pointer;
            transition: all 0.3s;
            box-shadow: 0 0 10px rgba(0, 255, 255, 0.5);
        }

        .game-over button:hover {
            background: #fff;
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.8);
        }

        /* Instructions */
        .instructions {
            position: absolute;
            bottom: 10px;
            left: 0;
            right: 0;
            text-align: center;
            color: #0ff;
            font-size: 12px;
            opacity: 0.7;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <div class="center-line"></div>
        <div class="scoreboard">
            <div class="score" id="player-score">0</div>
            <div class="score" id="computer-score">0</div>
        </div>
        <div class="paddle paddle-left" id="paddle-left"></div>
        <div class="paddle paddle-right" id="paddle-right"></div>
        <div class="ball" id="ball"></div>
        <div class="game-over" id="game-over">
            <h2 id="winner-text"></h2>
            <p>First to 5 wins!</p>
            <button onclick="location.reload()">Play Again</button>
        </div>
        <div class="instructions">
            ↑↓ or MOUSE to move left paddle | Computer controls right paddle
        </div>
    </div>

    <script>
        // Game Constants
        const GAME_WIDTH = 800;
        const GAME_HEIGHT = 600;
        const PADDLE_HEIGHT = 100;
        const PADDLE_WIDTH = 15;
        const BALL_SIZE = 15;
        const PADDLE_SPEED = 6;
        const COMPUTER_SPEED = 5;
        const BALL_SPEED = 5;
        const WIN_SCORE = 5;

        // Game Variables
        let playerScore = 0;
        let computerScore = 0;
        let gameActive = true;

        // Ball object
        const ball = {
            x: GAME_WIDTH / 2 - BALL_SIZE / 2,
            y: GAME_HEIGHT / 2 - BALL_SIZE / 2,
            vx: BALL_SPEED,
            vy: BALL_SPEED,
            size: BALL_SIZE
        };

        // Paddles object
        const paddles = {
            left: {
                x: 20,
                y: GAME_HEIGHT / 2 - PADDLE_HEIGHT / 2,
                width: PADDLE_WIDTH,
                height: PADDLE_HEIGHT,
                speed: PADDLE_SPEED,
                dy: 0
            },
            right: {
                x: GAME_WIDTH - 20 - PADDLE_WIDTH,
                y: GAME_HEIGHT / 2 - PADDLE_HEIGHT / 2,
                width: PADDLE_WIDTH,
                height: PADDLE_HEIGHT,
                speed: COMPUTER_SPEED
            }
        };

        // Input handling
        const keys = {};
        let mouseY = GAME_HEIGHT / 2;

        window.addEventListener('keydown', (e) => {
            keys[e.key] = true;
        });

        window.addEventListener('keyup', (e) => {
            keys[e.key] = false;
        });

        window.addEventListener('mousemove', (e) => {
            const gameContainer = document.querySelector('.game-container');
            const rect = gameContainer.getBoundingClientRect();
            mouseY = e.clientY - rect.top;
        });

        // Update player paddle position
        function updatePlayerPaddle() {
            const paddle = paddles.left;

            // Arrow keys
            if (keys['ArrowUp']) {
                paddle.dy = -paddle.speed;
            } else if (keys['ArrowDown']) {
                paddle.dy = paddle.speed;
            } else {
                paddle.dy = 0;
            }

            // Mouse control (smooth following)
            const targetY = mouseY - paddle.height / 2;
            paddle.y = Math.max(0, Math.min(GAME_HEIGHT - paddle.height, targetY));

            // Apply keyboard movement if used
            if (keys['ArrowUp'] || keys['ArrowDown']) {
                paddle.y = Math.max(0, Math.min(GAME_HEIGHT - paddle.height, paddle.y + paddle.dy));
            }
        }

        // Update computer paddle position (AI)
        function updateComputerPaddle() {
            const paddle = paddles.right;
            const paddleCenter = paddle.y + paddle.height / 2;
            const ballCenter = ball.y + ball.size / 2;

            // Simple AI: follow the ball
            if (ballCenter < paddleCenter - 15) {
                paddle.y = Math.max(0, paddle.y - paddle.speed);
            } else if (ballCenter > paddleCenter + 15) {
                paddle.y = Math.min(GAME_HEIGHT - paddle.height, paddle.y + paddle.speed);
            }
        }

        // Update ball position
        function updateBall() {
            ball.x += ball.vx;
            ball.y += ball.vy;

            // Top and bottom collision
            if (ball.y <= 0 || ball.y + ball.size >= GAME_HEIGHT) {
                ball.vy = -ball.vy;
                ball.y = Math.max(0, Math.min(GAME_HEIGHT - ball.size, ball.y));
            }

            // Paddle collision (left)
            if (
                ball.x < paddles.left.x + paddles.left.width &&
                ball.y + ball.size > paddles.left.y &&
                ball.y < paddles.left.y + paddles.left.height &&
                ball.vx < 0
            ) {
                ball.vx = -ball.vx;
                ball.x = paddles.left.x + paddles.left.width;
                // Add spin based on where ball hits paddle
                const hitPos = (ball.y + ball.size / 2 - (paddles.left.y + paddles.left.height / 2)) / (paddles.left.height / 2);
                ball.vy += hitPos * 2;
            }

            // Paddle collision (right)
            if (
                ball.x + ball.size > paddles.right.x &&
                ball.y + ball.size > paddles.right.y &&
                ball.y < paddles.right.y + paddles.right.height &&
                ball.vx > 0
            ) {
                ball.vx = -ball.vx;
                ball.x = paddles.right.x - ball.size;
                // Add spin based on where ball hits paddle
                const hitPos = (ball.y + ball.size / 2 - (paddles.right.y + paddles.right.height / 2)) / (paddles.right.height / 2);
                ball.vy += hitPos * 2;
            }

            // Score points
            if (ball.x < 0) {
                computerScore++;
                resetBall();
                updateScore();
            } else if (ball.x > GAME_WIDTH) {
                playerScore++;
                resetBall();
                updateScore();
            }
        }

        // Reset ball to center
        function resetBall() {
            ball.x = GAME_WIDTH / 2 - BALL_SIZE / 2;
            ball.y = GAME_HEIGHT / 2 - BALL_SIZE / 2;
            ball.vx = (Math.random() > 0.5 ? 1 : -1) * BALL_SPEED;
            ball.vy = (Math.random() - 0.5) * BALL_SPEED * 2;
        }

        // Update score display
        function updateScore() {
            document.getElementById('player-score').textContent = playerScore;
            document.getElementById('computer-score').textContent = computerScore;

            // Check for win
            if (playerScore >= WIN_SCORE || computerScore >= WIN_SCORE) {
                endGame();
            }
        }

        // End game
        function endGame() {
            gameActive = false;
            const gameOverDiv = document.getElementById('game-over');
            const winnerText = document.getElementById('winner-text');

            if (playerScore >= WIN_SCORE) {
                winnerText.textContent = '🎉 YOU WIN! 🎉';
            } else {
                winnerText.textContent = '💻 COMPUTER WINS! 💻';
            }

            gameOverDiv.style.display = 'block';
        }

        // Render game
        function render() {
            const ballElement = document.getElementById('ball');
            const paddleLeft = document.getElementById('paddle-left');
            const paddleRight = document.getElementById('paddle-right');

            ballElement.style.left = ball.x + 'px';
            ballElement.style.top = ball.y + 'px';

            paddleLeft.style.top = paddles.left.y + 'px';
            paddleRight.style.top = paddles.right.y + 'px';
        }

        // Game loop
        function gameLoop() {
            if (gameActive) {
                updatePlayerPaddle();
                updateComputerPaddle();
                updateBall();
            }
            render();
            requestAnimationFrame(gameLoop);
        }

        // Start game
        gameLoop();
    </script>
</body>
</html>
