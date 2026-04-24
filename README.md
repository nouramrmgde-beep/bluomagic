<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <title>Cyber Runner - Modern JS Edition</title>
    <style>
        :root {
            --bg-color: #050505;
            --player-color: #00f2ff;
            --enemy-color: #ff0055;
            --road-line: #1a1a1a;
        }
        body {
            margin: 0;
            background-color: var(--bg-color);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            color: white;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
        }
        #game-container {
            position: relative;
            box-shadow: 0 0 50px rgba(0, 242, 255, 0.1);
            border-radius: 15px;
            overflow: hidden;
            border: 2px solid #222;
        }
        canvas {
            display: block;
            background: linear-gradient(to bottom, #000 0%, #0a0a0a 100%);
        }
        .ui-layer {
            position: absolute;
            top: 20px;
            left: 20px;
            pointer-events: none;
        }
        .score-box {
            font-size: 28px;
            font-weight: bold;
            letter-spacing: 2px;
            text-transform: uppercase;
            background: rgba(255, 255, 255, 0.05);
            padding: 10px 20px;
            border-radius: 8px;
            backdrop-filter: blur(5px);
        }
    </style>
</head>
<body>

    <div id="game-container">
        <div class="ui-layer">
            <div class="score-box">SCORE: <span id="score">0</span></div>
        </div>
        <canvas id="gameCanvas"></canvas>
    </div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreElement = document.getElementById('score');

    // إعدادات القياسات
    canvas.width = 400;
    canvas.height = 700;

    const LANES = [66, 200, 334]; // أماكن المسارات الثلاثة
    let currentLane = 1;
    let score = 0;
    let gameSpeed = 7;
    let isGameOver = false;

    // كائن اللاعب (الدائرة)
    const player = {
        x: LANES[currentLane],
        y: canvas.height - 100,
        radius: 22,
        targetX: LANES[currentLane],
        ease: 0.15 // سرعة الانزلاق بين المسارات
    };

    // مصفوفة العقبات (المربعات)
    const obstacles = [];

    // دالة إنشاء عقبة جديدة
    function spawnObstacle() {
        const lane = Math.floor(Math.random() * 3);
        obstacles.push({
            x: LANES[lane] - 30,
            y: -100,
            width: 60,
            height: 60,
            color: '#ff0055',
            glow: '#ff0055'
        });
    }

    // التحديث البرمجي (Logic)
    function update() {
        if (isGameOver) return;

        // تحريك اللاعب بسلاسة للمسار المطلوب
        player.x += (player.targetX - player.x) * player.ease;

        // تحريك العقبات
        if (Math.random() < 0.02) spawnObstacle();

        obstacles.forEach((obs, index) => {
            obs.y += gameSpeed;

            // كشف التصادم (دائرة مع مربع)
            let closestX = Math.max(obs.x, Math.min(player.x, obs.x + obs.width));
            let closestY = Math.max(obs.y, Math.min(player.y, obs.y + obs.height));
            let distance = Math.sqrt((player.x - closestX)**2 + (player.y - closestY)**2);

            if (distance < player.radius) {
                endGame();
            }

            // حذف العقبات خارج الشاشة
            if (obs.y > canvas.height) {
                obstacles.splice(index, 1);
                score += 10;
                scoreElement.innerText = score;
                gameSpeed += 0.05; // زيادة الصعوبة
            }
        });
    }

    // الرسم (Graphics)
    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // رسم خطوط الطريق العصرية
        ctx.strokeStyle = '#1a1a1a';
        ctx.setLineDash([20, 30]);
        ctx.lineWidth = 2;
        [133, 266].forEach(lineX => {
            ctx.beginPath();
            ctx.moveTo(lineX, 0);
            ctx.lineTo(lineX, canvas.height);
            ctx.stroke();
        });

        // رسم اللاعب (دائرة نيون)
        ctx.save();
        ctx.shadowBlur = 20;
        ctx.shadowColor = '#00f2ff';
        ctx.fillStyle = '#00f2ff';
        ctx.beginPath();
        ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
        ctx.fill();
        // إضافة لمعة داخلية للدائرة
        ctx.fillStyle = 'rgba(255,255,255,0.5)';
        ctx.beginPath();
        ctx.arc(player.x - 5, player.y - 5, 5, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();

        // رسم العقبات (مربعات متوهجة)
        obstacles.forEach(obs => {
            ctx.save();
            ctx.shadowBlur = 15;
            ctx.shadowColor = obs.glow;
            
            // تدرج لوني للمربع
            let grad = ctx.createLinearGradient(obs.x, obs.y, obs.x, obs.y + obs.height);
            grad.addColorStop(0, '#ff0055');
            grad.addColorStop(1, '#80002b');
            
            ctx.fillStyle = grad;
            ctx.beginPath();
            ctx.roundRect(obs.x, obs.y, obs.width, obs.height, 10); // مربعات بحواف ناعمة
            ctx.fill();
            ctx.restore();
        });

        requestAnimationFrame(() => {
            update();
            draw();
        });
    }

    function endGame() {
        isGameOver = true;
        alert("Game Over! Score: " + score);
        location.reload();
    }

    // التحكم
    window.addEventListener('keydown', (e) => {
        if (e.key === 'ArrowLeft' && currentLane > 0) {
            currentLane--;
        } else if (e.key === 'ArrowRight' && currentLane < 2) {
            currentLane++;
        }
        player.targetX = LANES[currentLane];
    });

    draw(); // بدء اللعبة
</script>
</body>
</html>
