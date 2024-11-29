<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Racing Game with Pit Stop</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
        }
        canvas {
            display: block;
            background: #e0e0e0;
        }
        #gameInfo {
            position: fixed;
            top: 10px;
            left: 10px;
            background: white;
            padding: 10px;
            border: 1px solid black;
        }
    </style>
</head>
<body>
    <div id="gameInfo">
        <p id="laps">Laps: 0 / 3</p>
        <p id="hp">HP: 444</p>
    </div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        // Canvas size
        canvas.width = 800;
        canvas.height = 600;

        // Game variables
        const player = { x: 375, y: 500, width: 50, height: 100, speed: 5, hp: 444, laps: 0 };
        const cpuCars = [];
        const numCPUCars = 99;
        const cpuSpeeds = [];
        const totalLaps = 3;
        const playerStartPosition = 1;
        const obstacle = { x: Math.random() * 750, y: -50, width: 50, height: 50, speed: 7 };
        const pitStop = { x: 650, y: 400, width: 100, height: 150 };
        let inPit = false;
        let pitTimer = 0;

        // Generate CPU cars
        for (let i = 0; i < numCPUCars; i++) {
            cpuCars.push({ x: Math.random() * 750, y: Math.random() * -600, width: 50, height: 100, laps: 0 });
            cpuSpeeds.push(3 + Math.random() * 2); // Random speed between 3 and 5
        }

        // Draw car
        function drawCar(car, color) {
            ctx.fillStyle = color;
            ctx.fillRect(car.x, car.y, car.width, car.height);
        }

        // Draw pit stop
        function drawPitStop() {
            ctx.fillStyle = "blue";
            ctx.fillRect(pitStop.x, pitStop.y, pitStop.width, pitStop.height);
        }

        // Draw obstacle
        function drawObstacle() {
            ctx.fillStyle = "red";
            ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
        }

        // Draw finish line
        function drawFinishLine() {
            ctx.fillStyle = "black";
            ctx.fillRect(0, canvas.height - 50, canvas.width, 10);
        }

        // Update positions
        function updatePositions() {
            // Player controls
            if (keys["ArrowLeft"] && player.x > 0) player.x -= player.speed;
            if (keys["ArrowRight"] && player.x < canvas.width - player.width) player.x += player.speed;

            // Move CPU cars
            cpuCars.forEach((car, index) => {
                car.y += cpuSpeeds[index];
                if (car.y > canvas.height) {
                    car.y = -100;
                    car.x = Math.random() * 750;
                }
            });

            // Move obstacle
            obstacle.y += obstacle.speed;
            if (obstacle.y > canvas.height) {
                obstacle.y = -50;
                obstacle.x = Math.random() * 750;
            }

            // Check collisions
            if (
                player.x < obstacle.x + obstacle.width &&
                player.x + player.width > obstacle.x &&
                player.y < obstacle.y + obstacle.height &&
                player.y + player.height > obstacle.y
            ) {
                const damage = Math.floor(Math.random() * 144) + 1;
                player.hp -= damage;
                obstacle.y = -50; // Reset obstacle position
            }

            // Check if player is in pit stop
            if (
                player.x < pitStop.x + pitStop.width &&
                player.x + player.width > pitStop.x &&
                player.y < pitStop.y + pitStop.height &&
                player.y + player.height > pitStop.y
            ) {
                inPit = true;
            } else {
                inPit = false;
                pitTimer = 0;
            }

            if (inPit) {
                pitTimer++;
                if (pitTimer >= 240) {
                    player.hp = Math.min(player.hp + 100, 444);
                    pitTimer = 0;
                }
            }

            // Check lap completion
            if (player.y + player.height >= canvas.height - 50) {
                player.laps++;
                player.y = 500; // Reset position
            }

            cpuCars.forEach((car) => {
                if (car.y + car.height >= canvas.height - 50) {
                    car.laps++;
                    car.y = Math.random() * -600; // Reset position
                }
            });

            // End game conditions
            if (player.hp <= 0) {
                alert("Game Over! Your car is destroyed.");
                document.location.reload();
            }

            if (player.laps >= totalLaps) {
                alert("You Win! Congratulations on finishing the race!");
                document.location.reload();
            }
        }

        // Render game
        function render() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawPitStop();
            drawObstacle();
            drawFinishLine();
            drawCar(player, "green");
            cpuCars.forEach((car) => drawCar(car, "gray"));
            updateGameInfo();
        }

        // Update game info
        function updateGameInfo() {
            document.getElementById("laps").innerText = `Laps: ${player.laps} / ${totalLaps}`;
            document.getElementById("hp").innerText = `HP: ${player.hp}`;
        }

        // Keys
        const keys = {};
        window.addEventListener("keydown", (e) => (keys[e.key] = true));
        window.addEventListener("keyup", (e) => (keys[e.key] = false));

        // Game loop
        function gameLoop() {
            updatePositions();
            render();
            requestAnimationFrame(gameLoop);
        }

        gameLoop();
    </script>
</body>
</html>
