<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Neon Dodge</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            background: #0a0a0a;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Courier New', monospace;
            overflow: hidden;
        }
        
        #gameContainer {
            position: relative;
            box-shadow: 0 0 50px rgba(0, 255, 255, 0.3);
        }
        
        canvas {
            display: block;
            background: linear-gradient(180deg, #0a0a0a 0%, #1a0a2e 100%);
        }
        
        #ui {
            position: absolute;
            top: 20px;
            left: 20px;
            color: #0ff;
            font-size: 18px;
            text-shadow: 0 0 10px #0ff;
            pointer-events: none;
        }
        
        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            color: #fff;
            display: none;
        }
        
        #gameOver h1 {
            font-size: 48px;
            color: #ff0066;
            text-shadow: 0 0 20px #ff0066;
            margin-bottom: 20px;
            animation: pulse 1s infinite;
        }
        
        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.1); }
        }
        
        #gameOver p {
            font-size: 24px;
            color: #0ff;
            margin-bottom: 30px;
        }
        
        #restartBtn {
            background: transparent;
            border: 2px solid #0ff;
            color: #0ff;
            padding: 15px 40px;
            font-size: 20px;
            font-family: 'Courier New', monospace;
            cursor: pointer;
            text-transform: uppercase;
            transition: all 0.3s;
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.3);
        }
        
        #restartBtn:hover {
            background: #0ff;
            color: #000;
            box-shadow: 0 0 40px rgba(0, 255, 255, 0.6);
        }
        
        #startScreen {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            color: #fff;
        }
        
        #startScreen h1 {
            font-size: 56px;
            background: linear-gradient(90deg, #0ff, #ff0066, #0ff);
            background-size: 200% auto;
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            animation: shine 3s linear infinite;
            margin-bottom: 20px;
        }
        
        @keyframes shine {
            to { background-position: 200% center; }
        }
        
        #startScreen p {
            color: #888;
            margin-bottom: 30px;
            font-size: 16px;
        }
        
        #startBtn {
            background: linear-gradient(90deg, #0ff, #ff0066);
            border: none;
            color: #000;
            padding: 15px 50px;
            font-size: 24px;
            font-weight: bold;
            font-family: 'Courier New', monospace;
            cursor: pointer;
            text-transform: uppercase;
            transition: transform 0.2s;
        }
        
        #startBtn:hover {
            transform: scale(1.1);
        }
        
        .controls {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            color: #666;
            font-size: 14px;
            text-align: center;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="gameCanvas" width="800" height="600"></canvas>
        
        <div id="ui">
            <div>SCORE: <span id="score">0</span></div>
            <div>LEVEL: <span id="level">1</span></div>
            <div>LIVES: <span id="lives">3</span></div>
        </div>
        
        <div id="startScreen">
            <h1>NEON DODGE</h1>
            <p>Dodge the red obstacles • Collect blue orbs</p>
            <button id="startBtn">START GAME</button>
        </div>
        
        <div id="gameOver">
            <h1>GAME OVER</h1>
            <p>Final Score: <span id="finalScore">0</span></p>
            <button id="restartBtn">PLAY AGAIN</button>
        </div>
        
        <div class="controls">Use ARROW KEYS or WASD to move</div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOver');
        const scoreElement = document.getElementById('score');
        const levelElement = document.getElementById('level');
        const livesElement = document.getElementById('lives');
        const finalScoreElement = document.getElementById('finalScore');

        // Game state
        let gameRunning = false;
        let score = 0;
        let level = 1;
        let lives = 3;
        let frameCount = 0;

        // Player
        const player = {
            x: 400,
            y: 500,
            width: 30,
            height: 30,
            speed: 5,
            dx: 0,
            dy: 0,
            trail: []
        };

        // Game objects
        let obstacles = [];
        let powerUps = [];
        let particles = [];

        // Input handling
        const keys = {};
        document.addEventListener('keydown', (e) => {
            keys[e.key.toLowerCase()] = true;
        });
        document.addEventListener('keyup', (e) => {
            keys[e.key.toLowerCase()] = false;
        });

        // Particle class
        class Particle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                this.vx = (Math.random() - 0.5) * 8;
                this.vy = (Math.random() - 0.5) * 8;
                this.life = 1;
                this.color = color;
            }
            
            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.life -= 0.02;
                this.vx *= 0.98;
                this.vy *= 0.98;
            }
            
            draw() {
                ctx.save();
                ctx.globalAlpha = this.life;
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x, this.y, 4, 4);
                ctx.restore();
            }
        }

        // Create explosion effect
        function createExplosion(x, y, color, count = 15) {
            for (let i = 0; i < count; i++) {
                particles.push(new Particle(x, y, color));
            }
        }

        // Spawn obstacle
        function spawnObstacle() {
            const size = Math.random() * 30 + 20;
            obstacles.push({
                x: Math.random() * (canvas.width - size),
                y: -size,
                width: size,
                height: size,
                speed: Math.random() * 3 + 2 + (level * 0.5),
                rotation: 0,
                rotSpeed: (Math.random() - 0.5) * 0.1
            });
        }

        // Spawn power-up
        function spawnPowerUp() {
            powerUps.push({
                x: Math.random() * (canvas.width - 20),
                y: -20,
                radius: 10,
                speed: 2
            });
        }

        // Update game
        function update() {
            if (!gameRunning) return;

            frameCount++;

            // Player movement
            player.dx = 0;
            player.dy = 0;
            
            if (keys['arrowleft'] || keys['a']) player.dx = -player.speed;
            if (keys['arrowright'] || keys['d']) player.dx = player.speed;
            if (keys['arrowup'] || keys['w']) player.dy = -player.speed;
            if (keys['arrowdown'] || keys['s']) player.dy = player.speed;

            player.x += player.dx;
            player.y += player.dy;

            // Keep player in bounds
            player.x = Math.max(0, Math.min(canvas.width - player.width, player.x));
            player.y = Math.max(0, Math.min(canvas.height - player.height, player.y));

            // Player trail
            player.trail.push({x: player.x, y: player.y, alpha: 1});
            player.trail.forEach(t => t.alpha -= 0.1);
            player.trail = player.trail.filter(t => t.alpha > 0);

            // Spawn obstacles
            if (frameCount % Math.max(30, 60 - level * 5) === 0) {
                spawnObstacle();
            }

            // Spawn power-ups
            if (frameCount % 150 === 0) {
                spawnPowerUp();
            }

            // Update obstacles
            obstacles.forEach((obs, index) => {
                obs.y += obs.speed;
                obs.rotation += obs.rotSpeed;

                // Collision with player
                if (player.x < obs.x + obs.width &&
                    player.x + player.width > obs.x &&
                    player.y < obs.y + obs.height &&
                    player.y + player.height > obs.y) {
                    
                    lives--;
                    createExplosion(player.x + player.width/2, player.y + player.height/2, '#ff0066', 30);
                    obstacles.splice(index, 1);
                    
                    if (lives <= 0) {
                        gameOver();
                    }
                }

                // Remove off-screen
                if (obs.y > canvas.height) {
                    obstacles.splice(index, 1);
                    score += 10;
                }
            });

            // Update power-ups
            powerUps.forEach((pu, index) => {
                pu.y += pu.speed;

                // Collision with player
                const dx = (player.x + player.width/2) - pu.x;
                const dy = (player.y + player.height/2) - pu.y;
                const distance = Math.sqrt(dx*dx + dy*dy);

                if (distance < pu.radius + player.width/2) {
                    score += 50;
                    lives = Math.min(lives + 1, 5);
                    createExplosion(pu.x, pu.y, '#00ffff', 20);
                    powerUps.splice(index, 1);
                }

                // Remove off-screen
                if (pu.y > canvas.height) {
                    powerUps.splice(index, 1);
                }
            });

            // Update particles
            particles.forEach((p, index) => {
                p.update();
                if (p.life <= 0) particles.splice(index, 1);
            });

            // Level up
            if (score > level * 200) {
                level++;
                createExplosion(canvas.width/2, canvas.height/2, '#ffff00', 50);
            }

            // Update UI
            scoreElement.textContent = score;
            levelElement.textContent = level;
            livesElement.textContent = lives;
        }

        // Draw game
        function draw() {
            // Clear with fade effect
            ctx.fillStyle = 'rgba(10, 10, 10, 0.3)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw grid
            ctx.strokeStyle = 'rgba(0, 255, 255, 0.1)';
            ctx.lineWidth = 1;
            for (let i = 0; i < canvas.width; i += 50) {
                ctx.beginPath();
                ctx.moveTo(i, 0);
                ctx.lineTo(i, canvas.height);
                ctx.stroke();
            }
            for (let i = 0; i < canvas.height; i += 50) {
                ctx.beginPath();
                ctx.moveTo(0, i);
                ctx.lineTo(canvas.width, i);
                ctx.stroke();
            }

            // Draw player trail
            player.trail.forEach(t => {
                ctx.fillStyle = `rgba(0, 255, 255, ${t.alpha * 0.5})`;
                ctx.fillRect(t.x, t.y, player.width, player.height);
            });

            // Draw player
            ctx.save();
            ctx.shadowBlur = 20;
            ctx.shadowColor = '#0ff';
            ctx.fillStyle = '#0ff';
            ctx.fillRect(player.x, player.y, player.width, player.height);
            
            // Player inner glow
            ctx.fillStyle = '#fff';
            ctx.fillRect(player.x + 5, player.y + 5, player.width - 10, player.height - 10);
            ctx.restore();

            // Draw obstacles
            obstacles.forEach(obs => {
                ctx.save();
                ctx.translate(obs.x + obs.width/2, obs.y + obs.height/2);
                ctx.rotate(obs.rotation);
                ctx.shadowBlur = 15;
                ctx.shadowColor = '#ff0066';
                ctx.fillStyle = '#ff0066';
                ctx.fillRect(-obs.width/2, -obs.height/2, obs.width, obs.height);
                
                // Inner detail
                ctx.strokeStyle = '#ff99cc';
                ctx.lineWidth = 2;
                ctx.strokeRect(-obs.width/2 + 5, -obs.height/2 + 5, obs.width - 10, obs.height - 10);
                ctx.restore();
            });

            // Draw power-ups
            powerUps.forEach(pu => {
                ctx.save();
                ctx.shadowBlur = 20;
                ctx.shadowColor = '#00ffff';
                ctx.fillStyle = '#00ffff';
                ctx.beginPath();
                ctx.arc(pu.x, pu.y, pu.radius, 0, Math.PI * 2);
                ctx.fill();
                
                // Pulse effect
                ctx.strokeStyle = '#fff';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(pu.x, pu.y, pu.radius + Math.sin(frameCount * 0.1) * 5, 0, Math.PI * 2);
                ctx.stroke();
                ctx.restore();
            });

            // Draw particles
            particles.forEach(p => p.draw());
        }

        // Game loop
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Start game
        function startGame() {
            gameRunning = true;
            score = 0;
            level = 1;
            lives = 3;
            frameCount = 0;
            obstacles = [];
            powerUps = [];
            particles = [];
            player.x = 400;
            player.y = 500;
            player.trail = [];
            
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
        }

        // Game over
        function gameOver() {
            gameRunning = false;
            finalScoreElement.textContent = score;
            gameOverScreen.style.display = 'block';
        }

        // Event listeners
        document.getElementById('startBtn').addEventListener('click', startGame);
        document.getElementById('restartBtn').addEventListener('click', startGame);

        // Start loop
        gameLoop();
    </script>
</body>
</html>
