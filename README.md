<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>貪食蛇遊戲網站</title>
    <style>
        body {
            background-color: #f0f0f0;
            font-family: 'Press Start 2P', cursive;
            color: #333;
            margin: 0;
            padding: 0;
            text-align: center;
        }
        header {
            background-color: #333;
            color: #fff;
            padding: 20px;
        }
        nav {
            margin: 20px;
        }
        nav a {
            margin: 0 15px;
            text-decoration: none;
            color: #333;
        }
        section {
            margin: 20px;
            padding: 20px;
            background-color: #fff;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .retro {
            font-size: 16px;
            line-height: 1.6;
        }
        canvas {
            border: 1px solid #333;
            background-color: #000;
            display: block;
            margin: 20px auto;
        }
        button {
            font-family: 'Press Start 2P', cursive;
            font-size: 16px;
            padding: 10px 20px;
            margin: 20px;
            border: none;
            border-radius: 5px;
            background-color: #333;
            color: #fff;
            cursor: pointer;
        }
        button:hover {
            background-color: #555;
        }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
</head>
<body>
    <header>
        <h1>貪食蛇</h1>
        <nav>
            <a href="#introduction">遊戲介紹</a>
            <a href="#rules">遊戲規則</a>
            <a href="#history">遊戲歷史</a>
            <a href="#game">遊戲</a>
        </nav>
    </header>
    <section id="introduction">
        <h2>遊戲介紹</h2>
        <p class="retro">
            貪食蛇是一款經典的街機遊戲，玩家控制蛇的移動，讓蛇吃到食物並使其變長，同時避免撞到牆壁或自己的身體。
        </p>
    </section>
    <section id="rules">
        <h2>遊戲規則</h2>
        <p class="retro">
            玩家通過方向鍵控制蛇的移動，目標是吃掉出現的食物使蛇變長。遊戲結束的條件是蛇撞到牆壁或自己的身體。
        </p>
    </section>
    <section id="history">
        <h2>遊戲歷史</h2>
        <p class="retro">
            貪食蛇最早出現在1976年的街機遊戲中，並在1997年隨著 Nokia 手機內建遊戲而風靡全球。
        </p>
    </section>
    <section id="game">
        <h2>遊戲</h2>
        <button id="startButton">開始遊戲</button>
        <canvas id="snake" width="400" height="400" style="display:none;"></canvas>
        <script>
            document.getElementById('startButton').addEventListener('click', () => {
                document.getElementById('startButton').style.display = 'none';
                document.getElementById('snake').style.display = 'block';
                startGame();
            });

            function startGame() {
                const canvas = document.getElementById('snake');
                const context = canvas.getContext('2d');
                const box = 20;
                let snake = [];
                snake[0] = { x: 9 * box, y: 10 * box };

                let food = {
                    x: Math.floor(Math.random() * 19 + 1) * box,
                    y: Math.floor(Math.random() * 19 + 1) * box
                }

                let score = 0;
                let d;

                document.addEventListener("keydown", direction);

                function direction(event) {
                    if (event.keyCode == 37 && d != "RIGHT") {
                        d = "LEFT";
                    } else if (event.keyCode == 38 && d != "DOWN") {
                        d = "UP";
                    } else if (event.keyCode == 39 && d != "LEFT") {
                        d = "RIGHT";
                    } else if (event.keyCode == 40 && d != "UP") {
                        d = "DOWN";
                    }
                }

                function collision(head, array) {
                    for (let i = 0; i < array.length; i++) {
                        if (head.x == array[i].x && head.y == array[i].y) {
                            return true;
                        }
                    }
                    return false;
                }

                function draw() {
                    context.fillStyle = "#000";
                    context.fillRect(0, 0, canvas.width, canvas.height);

                    for (let i = 0; i < snake.length; i++) {
                        context.fillStyle = (i == 0) ? "#0FF" : "#FFF";
                        context.fillRect(snake[i].x, snake[i].y, box, box);

                        context.strokeStyle = "#000";
                        context.strokeRect(snake[i].x, snake[i].y, box, box);
                    }

                    context.fillStyle = "#FF0000";
                    context.fillRect(food.x, food.y, box, box);

                    let snakeX = snake[0].x;
                    let snakeY = snake[0].y;

                    if (d == "LEFT") snakeX -= box;
                    if (d == "UP") snakeY -= box;
                    if (d == "RIGHT") snakeX += box;
                    if (d == "DOWN") snakeY += box;

                    if (snakeX == food.x && snakeY == food.y) {
                        score++;
                        food = {
                            x: Math.floor(Math.random() * 19 + 1) * box,
                            y: Math.floor(Math.random() * 19 + 1) * box
                        }
                    } else {
                        snake.pop();
                    }

                    let newHead = {
                        x: snakeX,
                        y: snakeY
                    }

                    if (snakeX < 0 || snakeX >= canvas.width || snakeY < 0 || snakeY >= canvas.height || collision(newHead, snake)) {
                        clearInterval(game);
                        alert("遊戲結束。得分: " + score);
                    }

                    snake.unshift(newHead);

                    context.fillStyle = "#FFF";
                    context.font = "20px 'Press Start 2P'";
                    context.fillText("分數: " + score, 2 * box, 1.5 * box);
                }

                let game = setInterval(draw, 100);
            }
        </script>
    </section>
</body>
</html>
