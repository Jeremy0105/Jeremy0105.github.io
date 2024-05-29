<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>俄羅斯方塊遊戲網站</title>
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
            margin: 0 auto;
        }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
</head>
<body>
    <header>
        <h1>俄羅斯方塊</h1>
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
            俄羅斯方塊是一款經典的益智遊戲，玩家需要將不同形狀的方塊排列成完整的橫行，以獲得分數。
        </p>
    </section>
    <section id="rules">
        <h2>遊戲規則</h2>
        <p class="retro">
            玩家需要將隨機出現的方塊移動、旋轉，並排列成完整的一行以消除該行並得分。如果方塊堆到頂部，遊戲結束。
        </p>
    </section>
    <section id="history">
        <h2>遊戲歷史</h2>
        <p class="retro">
            俄羅斯方塊由蘇聯程式設計師阿列克謝·帕基特諾夫於1984年發明，並迅速成為全球最受歡迎的遊戲之一。
        </p>
    </section>
    <section id="game">
        <h2>遊戲</h2>
        <canvas id="tetris" width="320" height="640"></canvas>
        <script>
            document.addEventListener('DOMContentLoaded', () => {
                const canvas = document.getElementById('tetris');
                const context = canvas.getContext('2d');
                context.scale(20, 20);

                function arenaSweep() {
                    let rowCount = 1;
                    outer: for (let y = arena.length - 1; y > 0; --y) {
                        for (let x = 0; x < arena[y].length; ++x) {
                            if (arena[y][x] === 0) {
                                continue outer;
                            }
                        }
                        const row = arena.splice(y, 1)[0].fill(0);
                        arena.unshift(row);
                        ++y;

                        player.score += rowCount * 10;
                        rowCount *= 2;
                    }
                }

                function collide(arena, player) {
                    const [m, o] = [player.matrix, player.pos];
                    for (let y = 0; y < m.length; ++y) {
                        for (let x = 0; x < m[y].length; ++x) {
                            if (m[y][x] !== 0 &&
                               (arena[y + o.y] &&
                                arena[y + o.y][x + o.x]) !== 0) {
                                return true;
                            }
                        }
                    }
                    return false;
                }

                function createMatrix(w, h) {
                    const matrix = [];
                    while (h--) {
                        matrix.push(new Array(w).fill(0));
                    }
                    return matrix;
                }

                function createPiece(type) {
                    if (type === 'T') {
                        return [
                            [0, 0, 0],
                            [1, 1, 1],
                            [0, 1, 0],
                        ];
                    } else if (type === 'O') {
                        return [
                            [2, 2],
                            [2, 2],
                        ];
                    } else if (type === 'L') {
                        return [
                            [0, 3, 0],
                            [0, 3, 0],
                            [0, 3, 3],
                        ];
                    } else if (type === 'J') {
                        return [
                            [0, 4, 0],
                            [0, 4, 0],
                            [4, 4, 0],
                        ];
                    } else if (type === 'I') {
                        return [
                            [0, 5, 0, 0],
                            [0, 5, 0, 0],
                            [0, 5, 0, 0],
                            [0, 5, 0, 0],
                        ];
                    } else if (type === 'S') {
                        return [
                            [0, 6, 6],
                            [6, 6, 0],
                            [0, 0, 0],
                        ];
                    } else if (type === 'Z') {
                        return [
                            [7, 7, 0],
                            [0, 7, 7],
                            [0, 0, 0],
                        ];
                    }
                }

                function drawMatrix(matrix, offset) {
                    matrix.forEach((row, y) => {
                        row.forEach((value, x) => {
                            if (value !== 0) {
                                context.fillStyle = colors[value];
                                context.fillRect(x + offset.x,
                                                 y + offset.y,
                                                 1, 1);
                            }
                        });
                    });
                }

                function draw() {
                    context.fillStyle = '#000';
                    context.fillRect(0, 0, canvas.width, canvas.height);

                    drawMatrix(arena, {x: 0, y: 0});
                    drawMatrix(player.matrix, player.pos);
                }

                function merge(arena, player) {
                    player.matrix.forEach((row, y) => {
                        row.forEach((value, x) => {
                            if (value !== 0) {
                                arena[y + player.pos.y][x + player.pos.x] = value;
                            }
                        });
                    });
                }

                function rotate(matrix, dir) {
                    for (let y = 0; y < matrix.length; ++y) {
                        for (let x = 0; x < y; ++x) {
                            [
                                matrix[x][y],
                                matrix[y][x],
                            ] = [
                                matrix[y][x],
                                matrix[x][y],
                            ];
                        }
                    }

                    if (dir > 0) {
                        matrix.forEach(row => row.reverse());
                    } else {
                        matrix.reverse();
                    }
                }

                function playerDrop() {
                    player.pos.y++;
                    if (collide(arena, player)) {
                        player.pos.y--;
                        merge(arena, player);
                        playerReset();
                        arenaSweep();
                        updateScore();
                    }
                    dropCounter = 0;
                }

                function playerMove(dir) {
                    player.pos.x += dir;
                    if (collide(arena, player)) {
                        player.pos.x -= dir;
                    }
                }

                function playerReset() {
                    const pieces = 'ILJOTSZ';
                    player.matrix = createPiece(pieces[pieces.length * Math.random() | 0]);
                    player.pos.y = 0;
                    player.pos.x = (arena[0].length / 2 | 0) -
                                   (player.matrix[0].length / 2 | 0);
                    if (collide(arena, player)) {
                        arena.forEach(row => row.fill(0));
                        player.score = 0;
                        updateScore();
                    }
                }

                function playerRotate(dir) {
                    const pos = player.pos.x;
                    let offset = 1;
                    rotate(player.matrix, dir);
                    while (collide(arena, player)) {
                        player.pos.x += offset;
                        offset = -(offset + (offset > 0 ? 1 : -1));
                        if (offset > player.matrix[0].length) {
                            rotate(player.matrix, -dir);
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>俄羅斯方塊遊戲網站</title>
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
        <h1>俄羅斯方塊</h1>
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
            俄羅斯方塊是一款經典的益智遊戲，玩家需要將不同形狀的方塊排列成完整的橫行，以獲得分數。
        </p>
    </section>
    <section id="rules">
        <h2>遊戲規則</h2>
        <p class="retro">
            玩家需要將隨機出現的方塊移動、旋轉，並排列成完整的一行以消除該行並得分。如果方塊堆到頂部，遊戲結束。
        </p>
    </section>
    <section id="history">
        <h2>遊戲歷史</h2>
        <p class="retro">
            俄羅斯方塊由蘇聯程式設計師阿列克謝·帕基特諾夫於1984年發明，並迅速成為全球最受歡迎的遊戲之一。
        </p>
    </section>
    <section id="game">
        <h2>遊戲</h2>
        <button id="startButton">開始遊戲</button>
        <canvas id="tetris" width="320" height="640" style="display:none;"></canvas>
        <script>
            document.getElementById('startButton').addEventListener('click', () => {
                document.getElementById('startButton').style.display = 'none';
                document.getElementById('tetris').style.display = 'block';
                startGame();
            });

            function startGame() {
                const canvas = document.getElementById('tetris');
                const context = canvas.getContext('2d');
                context.scale(20, 20);

                let arena = createMatrix(12, 20);
                let player = {
                    pos: {x: 0, y: 0},
                    matrix: null,
                    score: 0
                };

                const colors = [
                    null,
                    '#FF0D72',
                    '#0DC2FF',
                    '#0DFF72',
                    '#F538FF',
                    '#FF8E0D',
                    '#FFE138',
                    '#3877FF',
                ];

                function arenaSweep() {
                    let rowCount = 1;
                    outer: for (let y = arena.length - 1; y > 0; --y) {
                        for (let x = 0; x < arena[y].length; ++x) {
                            if (arena[y][x] === 0) {
                                continue outer;
                            }
                        }
                        const row = arena.splice(y, 1)[0].fill(0);
                        arena.unshift(row);
                        ++y;

                        player.score += rowCount * 10;
                        rowCount *= 2;
                    }
                }

                function collide(arena, player) {
                    const [m, o] = [player.matrix, player.pos];
                    for (let y = 0; y < m.length; ++y) {
                        for (let x = 0; x < m[y].length; ++x) {
                            if (m[y][x] !== 0 &&
                               (arena[y + o.y] &&
                                arena[y + o.y][x + o.x]) !== 0) {
                                return true;
                            }
                        }
                    }
                    return false;
                }

                function createMatrix(w, h) {
                    const matrix = [];
                    while (h--) {
                        matrix.push(new Array(w).fill(0));
                    }
                    return matrix;
                }

                function createPiece(type) {
                    if (type === 'T') {
                        return [
                            [0, 0, 0],
                            [1, 1, 1],
                            [0, 1, 0],
                        ];
                    } else if (type === 'O') {
                        return [
                            [2, 2],
                            [2, 2],
                        ];
                    } else if (type === 'L') {
                        return [
                            [0, 3, 0],
                            [0, 3, 0],
                            [0, 3, 3],
                        ];
                    } else if (type === 'J') {
                        return [
                            [0, 4, 0],
                            [0, 4, 0],
                            [4, 4, 0],
                        ];
                    } else if (type === 'I') {
                        return [
                            [0, 5, 0, 0],
                            [0, 5, 0, 0],
                            [0, 5, 0, 0],
                            [0, 5, 0, 0],
                        ];
                    } else if (type === 'S') {
                        return [
                            [0, 6, 6],
                            [6, 6, 0],
                            [0, 0, 0],
                        ];
                    } else if (type === 'Z') {
                        return [
                            [7, 7, 0],
                            [0, 7, 7],
                            [0, 0, 0],
                        ];
                    }
                }

                function drawMatrix(matrix, offset) {
                    matrix.forEach((row, y) => {
                        row.forEach((value, x) => {
                            if (value !== 0) {
                                context.fillStyle = colors[value];
                                context.fillRect(x + offset.x,
                                                 y + offset.y,
                                                 1, 1);
                            }
                        });
                    });
                }

                function draw() {
                    context.fillStyle = '#000';
                    context.fillRect(0, 0, canvas.width, canvas.height);

                    drawMatrix(arena, {x: 0, y: 0});
                    drawMatrix(player.matrix, player.pos);
                }

                function merge(arena, player) {
                    player.matrix.forEach((row, y) => {
                        row.forEach((value, x) => {
                            if (value !== 0) {
                                arena[y + player.pos.y][x + player.pos.x] = value;
                            }
                        });
                    });
                }

                function rotate(matrix, dir) {
                    for (let y = 0; y < matrix.length; ++y) {
                        for (let x = 0; x < y; ++x) {
                            [
                                matrix[x][y],
                                matrix[y][x],
                            ] = [
                                matrix[y][x],
                                matrix[x][y],
                            ];
                        }
                    }

                    if (dir > 0) {
                        matrix.forEach(row => row.reverse());
                    } else {
                        matrix.reverse();
                    }
                }

                function playerDrop() {
                    player.pos.y++;
                    if (collide(arena, player)) {
                        player.pos.y--;
                        merge(arena, player);
                        playerReset();
                        arenaSweep();
                        updateScore();
                    }
                    dropCounter = 0;
                }

                function playerMove(dir) {
                    player.pos.x += dir;
                    if (collide(arena, player)) {
                        player.pos.x -= dir;
                    }
                }

                function playerReset() {
                    const pieces = 'ILJOTSZ';
                    player.matrix = createPiece(pieces[pieces.length * Math.random() | 0]);
                    player.pos.y = 0;
                    player.pos.x = (arena[0].length / 2 | 0) -
                                   (player.matrix[0].length / 2 | 0);
                    if (collide(arena, player)) {
                        arena.forEach(row => row.fill(0));
                        player.score = 0;
                        updateScore();
                    }
                }

                function playerRotate(dir) {
                    const pos = player.pos.x;
                    let offset = 1;
                    rotate(player.matrix, dir);
                    while (collide(arena, player)) {
                        player.pos.x += offset;
                        offset = -(offset + (offset > 0 ? 1 : -1));
                        if (offset > player.matrix[0].length) {
                            rotate(player.matrix, -dir);
                            player.pos.x = pos;
                            return;
                        }
                    }
                }

                let dropCounter = 0;
                let dropInterval = 1000;

                let lastTime = 0;
                function update(time = 0) {
                    const deltaTime = time - lastTime;
                    lastTime = time;

                    dropCounter += deltaTime;
                    if (dropCounter > dropInterval) {
                        playerDrop();
                    }

                    draw();
                    requestAnimationFrame(update);
                }

                function updateScore() {
                    document.getElementById('score').innerText = player.score;
                }

                document.addEventListener('keydown', event => {
                    if (event.keyCode === 37) {
                        playerMove(-1);
                    } else if (event.keyCode === 39) {
                        playerMove(1);
                    } else if (event.keyCode === 40) {
                        playerDrop();
                    } else if (event.keyCode === 81) {
                        playerRotate(-1);
                    } else if (event.keyCode === 87) {
                        playerRotate(1);
                    }
                });

                playerReset();
                updateScore();
                update();
            }
        </script>
    </section>
</body>
</html>
