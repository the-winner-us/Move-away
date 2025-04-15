# Move-away
A game that you have to get away from the obstacles 
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>人物避障遊戲 - 我的遊戲網站</title>
    <style>
        /* 網站通用樣式 */
        body {
            font-family: 'Arial', sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 0;
            color: #333;
        }
        
        .container {
            width: 90%;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        header {
            background-color: #5D5CDE;
            color: white;
            padding: 1rem 0;
            text-align: center;
        }
        
        nav {
            display: flex;
            justify-content: center;
            background-color: #444;
            padding: 1rem 0;
        }
        
        nav a {
            color: white;
            margin: 0 15px;
            text-decoration: none;
            font-weight: bold;
        }
        
        nav a:hover {
            text-decoration: underline;
        }
        
        footer {
            background-color: #333;
            color: white;
            text-align: center;
            padding: 2rem 0;
            margin-top: 2rem;
        }
        
        /* 遊戲樣式（此處保留原始遊戲樣式） */
        :root {
            --primary-color: #5D5CDE;
            --light-bg: #FFFFFF;
            --dark-bg: #181818;
            --light-text: #333333;
            --dark-text: #EEEEEE;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            touch-action: manipulation;
        }

        html, body {
            height: 100%;
            overflow-x: hidden;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            transition: background-color 0.3s, color 0.3s;
        }

        html.dark {
            background-color: var(--dark-bg);
            color: var(--dark-text);
        }
        
        html.dark nav, html.dark footer {
            background-color: #222;
        }
        
        html.dark header {
            background-color: #4c4bb3;
        }

        .game-container {
            display: flex;
            flex-direction: column;
            height: auto;
            min-height: 80vh;
            align-items: center;
            justify-content: center;
            padding: 1rem;
        }

        canvas {
            background-color: #f0f0f0;
            max-width: 100%;
            max-height: 70vh;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
            transition: background-color 0.3s;
        }

        html.dark canvas {
            background-color: #303030;
        }

        .controls {
            margin-top: 1rem;
            text-align: center;
            width: 100%;
            max-width: 500px;
        }

        .score-display {
            font-size: 1.2rem;
            font-weight: bold;
            margin-bottom: 0.5rem;
        }

        button {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 0.75rem 1.5rem;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 4px;
            cursor: pointer;
            margin: 0.5rem;
            transition: transform 0.1s, background-color 0.3s;
        }

        button:active {
            transform: scale(0.98);
        }

        .game-over {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(255, 255, 255, 0.9);
            padding: 2rem;
            border-radius: 8px;
            text-align: center;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
            transition: background-color 0.3s;
        }

        html.dark .game-over {
            background-color: rgba(40, 40, 40, 0.9);
        }

        .instructions {
            font-size: 0.9rem;
            margin-top: 1rem;
            opacity: 0.8;
        }

        @media (max-width: 480px) {
            canvas {
                height: 60vh;
            }
        }
    </style>
</head>
<body>
    <header>
        <div class="container">
            <h1>人物避障遊戲</h1>
        </div>
    </header>
    
    <nav>
        <a href="index.html">首頁</a>
        <a href="game.html">玩遊戲</a>
        <a href="index.html#about">關於</a>
    </nav>
    
    <div class="game-container">
        <div class="score-display">分數: <span id="score">0</span></div>
        <canvas id="gameCanvas"></canvas>
        <div class="controls">
            <button id="startBtn">開始遊戲</button>
            <button id="restartBtn" style="display: none;">重新開始</button>
        </div>
        <div class="instructions">
            左右滑動螢幕來移動角色，避開從上方掉落的障礙物。
        </div>
    </div>

    <div id="gameOverScreen" class="game-over">
        <h2>遊戲結束</h2>
        <p>最終分數: <span id="finalScore">0</span></p>
        <button id="playAgainBtn">再玩一次</button>
    </div>
    
    <footer>
        <div class="container">
            <p>&copy; 2023 我的遊戲網站. 保留所有權利。</p>
        </div>
    </footer>

    <script>
        // 深色模式檢測與設置
        if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
            document.documentElement.classList.add('dark');
        }
        window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', event => {
            if (event.matches) {
                document.documentElement.classList.add('dark');
            } else {
                document.documentElement.classList.remove('dark');
            }
        });

        // 遊戲基本設置
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startBtn = document.getElementById('startBtn');
        const restartBtn = document.getElementById('restartBtn');
        const playAgainBtn = document.getElementById('playAgainBtn');
        const scoreDisplay = document.getElementById('score');
        const finalScoreDisplay = document.getElementById('finalScore');
        const gameOverScreen = document.getElementById('gameOverScreen');

        // 設置畫布大小
        function resizeCanvas() {
            const containerWidth = Math.min(window.innerWidth - 40, 500);
            const containerHeight = Math.min(window.innerHeight * 0.6, 600);
            
            canvas.width = containerWidth;
            canvas.height = containerHeight;
        }

        // 初始化時調整畫布大小
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);

        // 遊戲狀態
        let gameStarted = false;
        let gameOver = false;
        let score = 0;

        // 角色設置
        const character = {
            x: 0,
            y: 0,
            width: 30,
            height: 50,
            speed: 10,
            color: '#5D5CDE'
        };

        // 障礙物數組
        let obstacles = [];
        const obstacleWidth = 40;
        const obstacleMinGap = 15; // 障礙物最小間距
        const maxObstacles = 3;  // 屏幕上最多同時存在的障礙物數量

        // 重置遊戲
        function resetGame() {
            character.x = canvas.width / 2 - character.width / 2;
            character.y = canvas.height - character.height - 20;
            obstacles = [];
            score = 0;
            gameOver = false;
            scoreDisplay.textContent = score;
            gameOverScreen.style.display = 'none';
        }

        // 更新角色位置（限制在畫布內）
        function updateCharacterPosition(newX) {
            character.x = Math.max(0, Math.min(newX, canvas.width - character.width));
        }

        // 創建新障礙物
        function createObstacle() {
            if (obstacles.length >= maxObstacles) return;
            
            // 隨機障礙物寬度和位置
            const width = obstacleWidth + Math.random() * 30;
            let x;
            let attempts = 0;
            let validPosition = false;
            
            // 確保新障礙物與現有障礙物不重疊
            while (!validPosition && attempts < 10) {
                attempts++;
                x = Math.random() * (canvas.width - width);
                validPosition = true;
                
                // 檢查是否與其他頂部附近的障礙物重疊
                for (const obstacle of obstacles) {
                    if (obstacle.y < 100 && 
                        x < obstacle.x + obstacle.width + obstacleMinGap && 
                        x + width + obstacleMinGap > obstacle.x) {
                        validPosition = false;
                        break;
                    }
                }
            }
            
            // 如果找到有效位置，添加新障礙物
            if (validPosition) {
                obstacles.push({
                    x,
                    y: -20, // 在畫布上方開始
                    width,
                    height: 20,
                    speed: 2 + Math.random() * 3 + score/500, // 隨著分數增加而加速
                    color: `hsl(${Math.random() * 360}, 70%, 50%)`
                });
            }
        }

        // 更新障礙物位置
        function updateObstacles() {
            for (let i = obstacles.length - 1; i >= 0; i--) {
                obstacles[i].y += obstacles[i].speed;
                
                // 檢查障礙物是否已離開畫布
                if (obstacles[i].y > canvas.height) {
                    obstacles.splice(i, 1);
                    score += 10;
                    scoreDisplay.textContent = score;
                }
            }
            
            // 根據遊戲進度調整障礙物生成頻率
            if (Math.random() < 0.02 + score/10000) {
                createObstacle();
            }
        }

        // 檢查碰撞
        function checkCollision() {
            for (const obstacle of obstacles) {
                if (character.x < obstacle.x + obstacle.width &&
                    character.x + character.width > obstacle.x &&
                    character.y < obstacle.y + obstacle.height &&
                    character.y + character.height > obstacle.y) {
                    return true;
                }
            }
            return false;
        }

        // 繪製角色
        function drawCharacter() {
            ctx.save();
            
            // 繪製身體
            ctx.fillStyle = character.color;
            ctx.fillRect(character.x, character.y, character.width, character.height);
            
            // 繪製頭部
            const headRadius = character.width * 0.4;
            ctx.beginPath();
            ctx.arc(
                character.x + character.width / 2,
                character.y - headRadius * 0.5,
                headRadius,
                0, Math.PI * 2
            );
            ctx.fill();
            
            // 繪製手臂
            ctx.fillRect(
                character.x - character.width * 0.2,
                character.y + character.height * 0.15,
                character.width * 0.2,
                character.height * 0.3
            );
            ctx.fillRect(
                character.x + character.width,
                character.y + character.height * 0.15,
                character.width * 0.2,
                character.height * 0.3
            );
            
            // 繪製腿部
            ctx.fillRect(
                character.x + character.width * 0.15,
                character.y + character.height,
                character.width * 0.3,
                character.height * 0.3
            );
            ctx.fillRect(
                character.x + character.width * 0.55,
                character.y + character.height,
                character.width * 0.3,
                character.height * 0.3
            );
            
            ctx.restore();
        }

        // 繪製障礙物
        function drawObstacles() {
            obstacles.forEach(obstacle => {
                ctx.fillStyle = obstacle.color;
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
            });
        }

        // 遊戲主循環
        function gameLoop() {
            if (!gameStarted || gameOver) return;
            
            // 清除畫布
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 更新障礙物位置
            updateObstacles();
            
            // 檢查碰撞
            if (checkCollision()) {
                gameOver = true;
                finalScoreDisplay.textContent = score;
                gameOverScreen.style.display = 'block';
                restartBtn.style.display = 'inline-block';
                startBtn.style.display = 'none';
                return;
            }
            
            // 繪製角色和障礙物
            drawCharacter();
            drawObstacles();
            
            // 繼續循環
            requestAnimationFrame(gameLoop);
        }

        // 滑動處理
        let touchStartX = 0;
        let touchEndX = 0;
        
        // 滑動事件處理器
        function handleTouchStart(e) {
            if (!gameStarted || gameOver) return;
            touchStartX = e.touches[0].clientX;
        }

        function handleTouchMove(e) {
            if (!gameStarted || gameOver) return;
            touchEndX = e.touches[0].clientX;
            const diffX = touchEndX - touchStartX;
            touchStartX = touchEndX;
            
            // 根據滑動更新角色位置
            updateCharacterPosition(character.x + diffX);
        }
        
        // 滑動控制
        canvas.addEventListener('touchstart', handleTouchStart);
        canvas.addEventListener('touchmove', handleTouchMove);
        
        // 鼠標控制（用於桌面測試）
        let isMouseDown = false;
        let lastMouseX = 0;
        
        canvas.addEventListener('mousedown', (e) => {
            if (!gameStarted || gameOver) return;
            isMouseDown = true;
            lastMouseX = e.clientX;
        });
        
        canvas.addEventListener('mousemove', (e) => {
            if (!gameStarted || gameOver || !isMouseDown) return;
            const diffX = e.clientX - lastMouseX;
            lastMouseX = e.clientX;
            updateCharacterPosition(character.x + diffX);
        });
        
        canvas.addEventListener('mouseup', () => {
            isMouseDown = false;
        });
        
        canvas.addEventListener('mouseleave', () => {
            isMouseDown = false;
        });

        // 按鈕事件
        startBtn.addEventListener('click', () => {
            gameStarted = true;
            resetGame();
            startBtn.style.display = 'none';
            restartBtn.style.display = 'inline-block';
            gameLoop();
        });

        restartBtn.addEventListener('click', () => {
            resetGame();
            gameLoop();
        });

        playAgainBtn.addEventListener('click', () => {
            resetGame();
            gameLoop();
        });

        // 初始顯示
        resetGame();
        drawCharacter();
    </script>
</body>
</html>