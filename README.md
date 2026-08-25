# d.r-driving<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
    <title>Dr. Driving 3D - Supercar Edition</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #000; font-family: Arial, sans-serif; }
        #game-container { position: relative; width: 100%; height: 100%; overflow: hidden; }
        canvas { display: block; width: 100%; height: 100%; }

        #hud {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.8);
            color: #fff;
            padding: 10px 30px;
            border-radius: 20px;
            font-size: 18px;
            font-weight: bold;
            border: 2px solid rgba(255,255,255,0.2);
            z-index: 10;
            display: flex;
            gap: 20px;
            text-align: center;
        }
        #hud span { color: #3498db; }

        .pedals {
            position: absolute;
            bottom: 30px;
            left: 30px;
            display: flex;
            flex-direction: column;
            gap: 15px;
            z-index: 10;
        }
        .pedal-btn {
            width: 70px;
            height: 90px;
            background: linear-gradient(135deg, #444, #222);
            border: 3px solid #666;
            border-radius: 12px;
            color: white;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 8px 0 #111;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .pedal-btn:active {
            transform: translateY(6px);
            box-shadow: 0 2px 0 #111;
        }
        #gasPedal { background: linear-gradient(135deg, #27ae60, #1e8449); border-color: #2ecc71; }
        #brakePedal { background: linear-gradient(135deg, #c0392b, #962d22); border-color: #e74c3c; }

        #steering-wheel-container {
            position: absolute;
            bottom: 20px;
            right: 30px;
            width: 180px;
            height: 180px;
            z-index: 10;
            touch-action: none;
        }
        #steeringWheel {
            width: 100%;
            height: 100%;
            background: radial-gradient(circle, #333 40%, #111 70%);
            border: 12px solid #222;
            border-radius: 50%;
            position: relative;
            box-shadow: 0 10px 25px rgba(0,0,0,0.8);
            cursor: grab;
            transition: transform 0.05s ease-out;
        }
        #steeringWheel::before {
            content: '';
            position: absolute;
            top: 50%; left: 10px; right: 10px;
            height: 10px;
            background: #444;
            transform: translateY(-50%);
            border-radius: 5px;
        }
        #steeringWheel::after {
            content: '';
            position: absolute;
            top: 10px; bottom: 10px; left: 50%;
            width: 10px;
            background: #444;
            transform: translateX(-50%);
            border-radius: 5px;
        }
        .center-cap {
            position: absolute;
            top: 50%; left: 50%;
            width: 55px; height: 55px;
            background: #1a1a1a;
            border: 4px solid #555;
            border-radius: 50%;
            transform: translate(-50%, -50%);
            z-index: 2;
        }

        #screen {
            position: absolute;
            inset: 0;
            background: rgba(0,0,0,0.85);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            z-index: 100;
            text-align: center;
            padding: 20px;
        }
        #screen h1 { font-size: 42px; color: #3498db; margin-bottom: 15px; }
        #screen p { font-size: 18px; color: #ccc; margin-bottom: 25px; line-height: 1.6; }
        #startBtn {
            padding: 15px 40px;
            font-size: 22px;
            font-weight: bold;
            background: #3498db;
            color: #fff;
            border: none;
            border-radius: 12px;
            cursor: pointer;
            box-shadow: 0 6px #21618c;
        }
        #startBtn:active { transform: translateY(4px); box-shadow: 0 2px #21618c; }
        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="game-container">
        <div id="hud">
            <div>السرعة: <span id="speedVal">0</span> كم/س</div>
            <div>النقاط: <span id="scoreVal">0</span></div>
        </div>
        <canvas id="gameCanvas"></canvas>

        <div class="pedals">
            <button class="pedal-btn" id="gasPedal">بنزين</button>
            <button class="pedal-btn" id="brakePedal">فرامل</button>
        </div>

        <div id="steering-wheel-container">
            <div id="steeringWheel">
                <div class="center-cap"></div>
            </div>
        </div>

        <div id="screen">
            <h1 id="screenTitle">Supercar Driving 3D</h1>
            <p id="screenDesc">قُد سيارتك الرياضية الزرقاء الفاخرة، تفادى حركة المرور وتجنب الاصطدام بالمباني!</p>
            <button id="startBtn">انطلق الآن 🏎️</button>
        </div>
    </div>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    function resizeCanvas() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    }
    window.addEventListener("resize", resizeCanvas);
    resizeCanvas();

    let gameRunning = false;
    let score = 0;
    let player = {
        x: 0,
        speed: 0,
        maxSpeed: 1.5, // سرعة أعلى للسيارة الرياضية الفاخرة
        accel: 0.02,
        brake: 0.04,
        steerAngle: 0
    };

    let input = { gas: false, brake: false };

    const gasBtn = document.getElementById("gasPedal");
    const brakeBtn = document.getElementById("brakePedal");
    setupButton(gasBtn, "gas");
    setupButton(brakeBtn, "brake");

    function setupButton(btn, key) {
        btn.addEventListener("pointerdown", () => input[key] = true);
        btn.addEventListener("pointerup", () => input[key] = false);
        btn.addEventListener("pointerleave", () => input[key] = false);
    }

    const wheel = document.getElementById("steeringWheel");
    let wheelDragging = false;
    let currentWheelAngle = 0;
    let startTouchAngle = 0;

    wheel.addEventListener("pointerdown", (e) => {
        wheelDragging = true;
        let rect = wheel.getBoundingClientRect();
        let cx = rect.left + rect.width / 2;
        let cy = rect.top + rect.height / 2;
        startTouchAngle = Math.atan2(e.clientY - cy, e.clientX - cx) - currentWheelAngle;
        wheel.setPointerCapture(e.pointerId);
    });

    wheel.addEventListener("pointermove", (e) => {
        if (!wheelDragging) return;
        let rect = wheel.getBoundingClientRect();
        let cx = rect.left + rect.width / 2;
        let cy = rect.top + rect.height / 2;
        let angle = Math.atan2(e.clientY - cy, e.clientX - cx) - startTouchAngle;
        let maxDeg = Math.PI / 2;
        if (angle > maxDeg) angle = maxDeg;
        if (angle < -maxDeg) angle = -maxDeg;
        currentWheelAngle = angle;
        wheel.style.transform = `rotate(${currentWheelAngle * (180 / Math.PI)}deg)`;
        player.steerAngle = currentWheelAngle / maxDeg;
    });

    function releaseWheel() {
        wheelDragging = false;
        currentWheelAngle = 0;
        wheel.style.transform = `rotate(0deg)`;
        player.steerAngle = 0;
    }
    wheel.addEventListener("pointerup", releaseWheel);
    wheel.addEventListener("pointercancel", releaseWheel);

    let keys = {};
    window.addEventListener("keydown", e => keys[e.key] = true);
    window.addEventListener("keyup", e => keys[e.key] = false);

    let trafficCars = [];
    function spawnTrafficCar() {
        if (!gameRunning) return;
        let lanes = [-200, 0, 200];
        let laneX = lanes[Math.floor(Math.random() * lanes.length)];
        trafficCars.push({
            x: laneX,
            z: 1500,
            color: ['#e74c3c', '#f1c40f', '#2ecc71', '#ffffff'][Math.floor(Math.random() * 4)]
        });
    }
    setInterval(spawnTrafficCar, 1800);

    function resetGame() {
        player.x = 0;
        player.speed = 0;
        player.steerAngle = 0;
        score = 0;
        trafficCars = [];
        gameRunning = true;
        document.getElementById("screen").classList.add("hidden");
    }

    document.getElementById("startBtn").addEventListener("click", resetGame);

    function gameOver(reason) {
        gameRunning = false;
        document.getElementById("screenTitle").innerText = "انتهت الرحلة!";
        document.getElementById("screenDesc").innerText = reason + `\nالنقاط النهائية: ${score}`;
        document.getElementById("startBtn").innerText = "قيادة جديدة 🔄";
        document.getElementById("screen").classList.remove("hidden");
    }

    function update() {
        if (!gameRunning) return;

        if (keys["ArrowLeft"] || keys["a"]) player.steerAngle = -0.8;
        else if (keys["ArrowRight"] || keys["d"]) player.steerAngle = 0.8;
        else if (!wheelDragging) { player.steerAngle *= 0.8; }

        if (input.gas || keys["ArrowUp"] || keys["w"]) {
            player.speed += player.accel;
            if (player.speed > player.maxSpeed) player.speed = player.maxSpeed;
        } else if (input.brake || keys["ArrowDown"] || keys["s"]) {
            player.speed -= player.brake;
            if (player.speed < 0) player.speed = 0;
        } else {
            player.speed -= 0.006;
            if (player.speed < 0) player.speed = 0;
        }

        player.x += player.steerAngle * player.speed * 9;

        if (Math.abs(player.x) > 350) {
            gameOver("اصطدمت بالرصيف والمباني!");
            return;
        }

        for (let i = trafficCars.length - 1; i >= 0; i--) {
            let car = trafficCars[i];
            car.z -= (player.speed * 35 + 12);

            if (car.z < 100 && car.z > 20) {
                if (Math.abs(player.x - car.x) < 80) {
                    gameOver("اصطدمت بسيارة أخرى!");
                    return;
                }
            }

            if (car.z < -50) {
                trafficCars.splice(i, 1);
                score += 15;
            }
        }

        document.getElementById("speedVal").innerText = Math.round(player.speed * 60);
        document.getElementById("scoreVal").innerText = score;
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        let w = canvas.width;
        let h = canvas.height;
        let horizonY = h * 0.45;

        // السماء
        let skyGradient = ctx.createLinearGradient(0, 0, 0, horizonY);
        skyGradient.addColorStop(0, "#0b192c");
        skyGradient.addColorStop(1, "#1e3e62");
        ctx.fillStyle = skyGradient;
        ctx.fillRect(0, 0, w, horizonY);

        // المباني الجانبية
        ctx.fillStyle = "#161a30";
        ctx.beginPath();
        ctx.moveTo(0, horizonY);
        ctx.lineTo(w * 0.25 + player.x * 0.2, horizonY);
        ctx.lineTo(0, h);
        ctx.fill();

        ctx.beginPath();
        ctx.moveTo(w, horizonY);
        ctx.lineTo(w * 0.75 + player.x * 0.2, horizonY);
        ctx.lineTo(w, h);
        ctx.fill();

        // الطريق الإسفلتي
        ctx.fillStyle = "#222831";
        ctx.beginPath();
        ctx.moveTo(w * 0.25 + player.x * 0.2, horizonY);
        ctx.lineTo(w * 0.75 + player.x * 0.2, horizonY);
        ctx.lineTo(w + player.x * 1.5, h);
        ctx.lineTo(0 + player.x * 1.5, h);
        ctx.fill();

        // السيارات المعاكسة
        trafficCars.forEach(car => {
            let scale = 100 / car.z;
            if (scale > 0 && scale < 5) {
                let carW = 140 * scale;
                let carH = 80 * scale;
                let carScreenX = (w / 2) + ((car.x - player.x) * scale) - (carW / 2);
                let carScreenY = horizonY + (h - horizonY) * (1 - (car.z / 1500));

                ctx.fillStyle = car.color;
                ctx.fillRect(carScreenX, carScreenY - carH, carW, carH);
                ctx.fillStyle = "#111";
                ctx.fillRect(carScreenX + carW*0.15, carScreenY - carH*0.8, carW*0.7, carH*0.3);
            }
        });

        // رسم سيارة اللاعب الرياضية الفاخرة من الخلف (تصميم مشابه للصورة الزرقاء)
        let carWidth = 200;
        let carHeight = 110;
        let carX = (w / 2) - (carWidth / 2) - (player.x * 0.5);
        let carY = h - carHeight - 35;

        // ظل تحتي
        ctx.fillStyle = "rgba(0,0,0,0.6)";
        ctx.fillRect(carX + 5, carY + carHeight - 8, carWidth - 10, 15);

        // هيكل السيارة الرياضية الأزرق المعدني
        let carGrad = ctx.createLinearGradient(carX, carY, carX, carY + carHeight);
        carGrad.addColorStop(0, "#2980b9");
        carGrad.addColorStop(0.5, "#1f618d");
        carGrad.addColorStop(1, "#154360");
        ctx.fillStyle = carGrad;
        
        ctx.beginPath();
        ctx.roundRect(carX, carY, carWidth, carHeight, [35, 35, 12, 12]);
        ctx.fill();

        // الزجاج الخلفي المائل
        ctx.fillStyle = "#111827";
        ctx.beginPath();
        ctx.moveTo(carX + 35, carY + 15);
        ctx.lineTo(carX + carWidth - 35, carY + 15);
        ctx.lineTo(carX + carWidth - 20, carY + 50);
        ctx.lineTo(carX + 20, carY + 50);
        ctx.fill();

        // المصابيح الخلفية LED الرياضية (مضاءة باللون الأحمر/الأزرق الهادئ)
        ctx.fillStyle = "#e74c3c";
        ctx.shadowColor = "#e74c3c";
        ctx.shadowBlur = 15;
        ctx.fillRect(carX + 15, carY + 45, 35, 12);
        ctx.fillRect(carX + carWidth - 50, carY + 45, 35, 12);
        ctx.shadowBlur = 0; // إلغاء الوهج للبقية

        // فتحات التهوية والصدام السفلي
        ctx.fillStyle = "#0f172a";
        ctx.fillRect(carX + 50, carY + 75, carWidth - 100, 22);

        // شعار أو لوحة رياضية وسطية
        ctx.fillStyle = "#f1c40f";
        ctx.fillRect(carX + (carWidth/2) - 25, carY + 68, 50, 10);
    }

    function gameLoop() {
        update();
        draw();
        requestAnimationFrame(gameLoop);
    }

    gameLoop();
</script>

</body>
</html>
