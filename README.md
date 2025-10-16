<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Escape the Devil's Door - Fixed</title>
    <style>
        body { 
            display: flex; 
            justify-content: center; 
            align-items: center; 
            height: 100vh; 
            margin: 0; 
            background: #111; 
            color: #fff; 
            font-family: monospace; 
        }
        canvas { 
            border: 2px solid #ffd700; 
            background: #000; 
            image-rendering: pixelated; 
            image-rendering: -moz-crisp-edges; 
            image-rendering: crisp-edges;
            width: 100vw;
            height: 100vh;
            max-width: 600px;
            max-height: 600px;
            object-fit: contain;
        }
        #ui { 
            position: absolute; 
            top: 10px; 
            left: 10px; 
            font-size: 14px; 
            display: none;
        }
        #cutscene-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            font-size: 24px;
            color: #ffd700;
            text-shadow: 2px 2px 4px #000;
            display: none;
            line-height: 1.4;
            max-width: 600px;
        }
    </style>
</head>
<body>
    <canvas id="game" width="600" height="600"></canvas>
    <div id="ui">
        <div>Tasks: <span id="tasks">0</span>/3</div>
        <div>Health: <span id="health">100</span></div>
        <div>Controls: WASD or Arrows</div>
    </div>
    <div id="cutscene-text"></div>
    <script>
        const canvas = document.getElementById('game');
        const ctx = canvas.getContext('2d');
        if (!ctx) {
            console.error('Failed to get 2D context');
            document.body.innerHTML = '<h1 style="color: red;">Canvas not supported in this browser</h1>';
            throw new Error('No canvas context');
        }
        ctx.imageSmoothingEnabled = false;

        const TILE = 32;
        let map = [
            [1, 1, 1, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1, 1],
            [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1],
            [1, 0, 1, 0, 1, 2, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 1, 1, 1, 0, 0, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 0, 1],
            [0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1],
            [1, 0, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 0, 0, 1, 1, 1, 0, 1, 1, 0, 0, 1, 0, 1, 0, 1, 0, 1, 1],
            [0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 1],
            [0, 0, 1, 1, 1, 1, 1, 0, 1, 0, 0, 0, 1, 1, 1, 2, 1, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 1, 0, 1, 1],
            [1, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 1],
            [1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 0, 0, 1, 1, 1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 1, 1, 1],
            [1, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 0, 1, 1, 0, 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 0, 0, 1, 0, 1, 1, 1, 0, 0, 0, 0, 1],
            [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 1],
            [0, 0, 1, 0, 1, 0, 1, 0, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 0, 0, 1, 1, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 0, 1],
            [1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1],
            [0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 2, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 0, 1, 1, 0, 1, 0, 0, 1, 1, 1, 1, 0, 1, 1, 1, 0, 0, 0, 1, 1],
            [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 1],
            [1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 0, 0, 1, 1],
            [1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 1],
            [1, 0, 1, 1, 1, 0, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 0, 0, 0, 1, 0, 0, 1, 1, 0, 1, 1],
            [0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 1],
            [1, 0, 0, 0, 0, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1],
            [1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 1, 1],
            [1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1],
            [1, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 1],
            [1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1],
            [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1],
            [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 3, 1]
        ];
        const MAP_WIDTH = map[0].length;
        const MAP_HEIGHT = map.length;

        // Assets with fallbacks
        let wallTexture = new Image();
        wallTexture.src = 'https://opengameart.org/sites/default/files/brick_0.png';
        wallTexture.onerror = () => console.warn('Wall texture failed to load');
        
        let floorTexture = new Image();
        floorTexture.src = 'https://opengameart.org/sites/default/files/floor_0.png';
        floorTexture.onerror = () => console.warn('Floor texture failed to load');
        
        let playerSprite = new Image();
        playerSprite.src = 'https://i.ibb.co/ym74Xxw/warrior-pixlated-Photoroom.png';
        playerSprite.onerror = () => console.warn('Player sprite failed to load');
        
        let monsterSprite = new Image();
        monsterSprite.src = 'https://i.ibb.co/M57m3y52/pixlated-devil-Photoroom.png';
        monsterSprite.onerror = () => console.warn('Monster sprite failed to load');

        let player = { px: 1 * TILE + TILE / 2, py: 1 * TILE + TILE / 2 };
        let items = [];
        for (let y = 0; y < MAP_HEIGHT; y++) {
            for (let x = 0; x < MAP_WIDTH; x++) {
                if (map[y][x] === 2) {
                    items.push({ x, y, collected: false });
                    map[y][x] = 0;
                }
            }
        }
        let collected = 0;
        let door = { x: 36, y: 34 };
        let camera = { x: 0, y: 0 };
        let monsters = [
            { x: 20, y: 1, updateTimer: Math.random() * 60, path: null, pathTimer: 0 },
            { x: 5, y: 10, updateTimer: Math.random() * 60, path: null, pathTimer: 0 },
            { x: 30, y: 15, updateTimer: Math.random() * 60, path: null, pathTimer: 0 }
        ];
        let health = 100;
        let keys = {};
        let gameState = 'cutscene';
        let cutsceneFrame = 0;
        const INTRO_CUTSCENE_FRAMES = 600;

        // Minimal audio
        const bgMusic = new Audio('https://tabletopaudio.com/download.php?down=mp3&file=Undercroft.mp3');
        bgMusic.loop = true;
        bgMusic.volume = 0.5;
        bgMusic.onerror = () => console.warn('Background music failed to load');

        const ui = document.getElementById('ui');
        const cutsceneText = document.getElementById('cutscene-text');

        document.addEventListener('keydown', e => {
            keys[e.key.toLowerCase()] = true;
            if (gameState === 'cutscene' && e.key === ' ') {
                gameState = 'game';
                ui.style.display = 'block';
                cutsceneText.style.display = 'none';
                bgMusic.play().catch(e => console.error('Audio play failed:', e));
            }
        });
        document.addEventListener('keyup', e => {
            keys[e.key.toLowerCase()] = false;
        });

        function heuristic(a, b) {
            return Math.abs(a.x - b.x) + Math.abs(a.y - b.y);
        }

        function getNeighbors(pos) {
            const dirs = [{dx: 0, dy: -1}, {dx: 0, dy: 1}, {dx: -1, dy: 0}, {dx: 1, dy: 0}];
            let neigh = [];
            for (let d of dirs) {
                let nx = pos.x + d.dx;
                let ny = pos.y + d.dy;
                if (nx >= 0 && nx < MAP_WIDTH && ny >= 0 && ny < MAP_HEIGHT && map[ny][nx] !== 1) {
                    neigh.push({x: nx, y: ny});
                }
            }
            return neigh;
        }

        function astar(start, goal) {
            if (start.x === goal.x && start.y === goal.y) return [{x: start.x, y: start.y}];
            let openSet = new Set();
            let cameFrom = new Map();
            let gScore = new Map();
            let fScore = new Map();
            const startKey = `${start.x},${start.y}`;
            openSet.add(startKey);
            gScore.set(startKey, 0);
            fScore.set(startKey, heuristic(start, goal));
            while (openSet.size > 0) {
                let currentKey = null;
                let lowestF = Infinity;
                for (let key of openSet) {
                    if (fScore.get(key) < lowestF) {
                        lowestF = fScore.get(key);
                        currentKey = key;
                    }
                }
                if (!currentKey) break;
                const [cx, cy] = currentKey.split(',').map(Number);
                const current = {x: cx, y: cy};
                if (current.x === goal.x && current.y === goal.y) {
                    let path = [];
                    let temp = current;
                    while (temp) {
                        path.push({x: temp.x, y: temp.y});
                        const key = `${temp.x},${temp.y}`;
                        temp = cameFrom.has(key) ? cameFrom.get(key) : null;
                        if (temp) {
                            const [tx, ty] = temp.split(',').map(Number);
                            temp = {x: tx, y: ty};
                        }
                    }
                    return path.reverse();
                }
                openSet.delete(currentKey);
                const neighbors = getNeighbors(current);
                for (let neighbor of neighbors) {
                    const nKey = `${neighbor.x},${neighbor.y}`;
                    const tentG = gScore.get(currentKey) + 1;
                    if (!gScore.has(nKey) || tentG < gScore.get(nKey)) {
                        cameFrom.set(nKey, currentKey);
                        gScore.set(nKey, tentG);
                        fScore.set(nKey, tentG + heuristic(neighbor, goal));
                        if (!openSet.has(nKey)) {
                            openSet.add(nKey);
                        }
                    }
                }
            }
            return null;
        }

        function drawPixel(x, y, color, size = 1) {
            ctx.fillStyle = color;
            ctx.fillRect(x, y, size, size);
        }

        function drawGlowingCube(wx, wy) {
            const cx = wx + TILE / 2 - 8, cy = wy + TILE / 2 - 8;
            ctx.fillStyle = '#ff0';
            ctx.fillRect(cx, cy, 16, 16);
            ctx.fillStyle = '#fff';
            drawPixel(cx, cy, '#fff', 4);
            drawPixel(cx + 16, cy, '#fff', 2);
            drawPixel(cx, cy + 16, '#fff', 2);
        }

        function drawCutscene() {
            ctx.fillStyle = '#111';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#fff';
            ctx.font = '24px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('Lost in the enchanted forest,', canvas.width / 2, canvas.height / 2 - 50);
            ctx.fillText('Press SPACE to start', canvas.width / 2, canvas.height / 2 + 20);
            cutsceneText.style.display = 'block';
            cutsceneText.textContent = 'Lost in the enchanted forest,\nPress SPACE to start';
            ctx.textAlign = 'left';
        }

        function update() {
            const speed = 3;
            let vx = 0, vy = 0;
            if (keys['a'] || keys['arrowleft']) vx = -speed;
            if (keys['d'] || keys['arrowright']) vx = speed;
            if (keys['w'] || keys['arrowup']) vy = -speed;
            if (keys['s'] || keys['arrowdown']) vy = speed;
            if (vx !== 0 && vy !== 0) {
                vx /= Math.sqrt(2);
                vy /= Math.sqrt(2);
            }

            let newPx = player.px + vx;
            let tx = Math.floor(newPx / TILE);
            let ty = Math.floor(player.py / TILE);
            if (tx >= 0 && tx < MAP_WIDTH && map[ty][tx] !== 1) {
                player.px = newPx;
            }

            let newPy = player.py + vy;
            tx = Math.floor(player.px / TILE);
            ty = Math.floor(newPy / TILE);
            if (ty >= 0 && ty < MAP_HEIGHT && map[ty][tx] !== 1) {
                player.py = newPy;
            }

            const playerTileX = Math.floor(player.px / TILE);
            const playerTileY = Math.floor(player.py / TILE);

            for (let item of items) {
                if (!item.collected && playerTileX === item.x && playerTileY === item.y) {
                    item.collected = true;
                    collected++;
                    document.getElementById('tasks').textContent = collected;
                }
            }

            if (playerTileX === door.x && playerTileY === door.y && collected >= 3) {
                alert('You Win!');
                location.reload();
            }

            const chaseDist = 10;
            for (let monster of monsters) {
                const distToPlayer = Math.abs(monster.x - playerTileX) + Math.abs(monster.y - playerTileY);
                let isChasing = distToPlayer < chaseDist;
                if (isChasing) {
                    monster.pathTimer++;
                    if (monster.pathTimer > 30) {
                        monster.pathTimer = 0;
                        const playerTile = {x: playerTileX, y: playerTileY};
                        monster.path = astar({x: monster.x, y: monster.y}, playerTile);
                    }
                } else {
                    monster.path = null;
                    monster.pathTimer = 0;
                }

                monster.updateTimer++;
                if (monster.updateTimer > 15) {
                    monster.updateTimer = 0;
                    let nextPos = null;
                    if (isChasing && monster.path && monster.path.length > 1) {
                        nextPos = monster.path[1];
                        monster.path.shift();
                    } else {
                        const neighbors = getNeighbors({x: monster.x, y: monster.y});
                        if (neighbors.length > 0) {
                            nextPos = neighbors[Math.floor(Math.random() * neighbors.length)];
                        }
                    }
                    if (nextPos) {
                        monster.x = nextPos.x;
                        monster.y = nextPos.y;
                    }
                    if (playerTileX === monster.x && playerTileY === monster.y) {
                        health -= 30;
                        document.getElementById('health').textContent = health;
                        if (health <= 0) {
                            alert('Game Over!');
                            location.reload();
                        }
                    }
                }
            }
        }

        function draw() {
            try {
                ctx.clearRect(0, 0, canvas.width, canvas.height);

                const camX = player.px - canvas.width / 2;
                const camY = player.py - canvas.height / 2;
                camera.x = Math.max(0, Math.min(camX, MAP_WIDTH * TILE - canvas.width));
                camera.y = Math.max(0, Math.min(camY, MAP_HEIGHT * TILE - canvas.height));

                const startX = Math.max(0, Math.floor(camera.x / TILE));
                const endX = Math.min(MAP_WIDTH, startX + Math.ceil(canvas.width / TILE) + 1);
                const startY = Math.max(0, Math.floor(camera.y / TILE));
                const endY = Math.min(MAP_HEIGHT, startY + Math.ceil(canvas.height / TILE) + 1);

                for (let y = startY; y < endY; y++) {
                    for (let x = startX; x < endX; x++) {
                        const tileX = x * TILE - camera.x;
                        const tileY = y * TILE - camera.y;
                        if (map[y][x] === 1) {
                            ctx.fillStyle = wallTexture.complete ? '#555' : '#555';
                            if (wallTexture.complete) {
                                ctx.drawImage(wallTexture, tileX, tileY, TILE, TILE);
                            } else {
                                ctx.fillRect(tileX, tileY, TILE, TILE);
                            }
                        } else {
                            ctx.fillStyle = floorTexture.complete ? '#222' : '#222';
                            if (floorTexture.complete) {
                                ctx.drawImage(floorTexture, tileX, tileY, TILE, TILE);
                            } else {
                                ctx.fillRect(tileX, tileY, TILE, TILE);
                            }
                        }
                    }
                }

                for (let item of items) {
                    if (!item.collected) {
                        drawGlowingCube(item.x * TILE - camera.x, item.y * TILE - camera.y);
                    }
                }

                const dx = door.x * TILE - camera.x;
                const dy = door.y * TILE - camera.y;
                ctx.fillStyle = '#8B0000';
                ctx.fillRect(dx, dy, TILE, TILE);
                ctx.strokeStyle = '#ffd700';
                ctx.lineWidth = 2;
                ctx.strokeRect(dx, dy, TILE, TILE);
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(dx + TILE - 8, dy + TILE / 2 - 4, 6, 8);

                ctx.fillStyle = playerSprite.complete ? '#00f' : '#00f';
                if (playerSprite.complete) {
                    ctx.drawImage(playerSprite, player.px - camera.x - 16, player.py - camera.y - 16, 32, 32);
                } else {
                    ctx.fillRect(player.px - camera.x - 8, player.py - camera.y - 8, 16, 16);
                }

                for (let monster of monsters) {
                    ctx.fillStyle = monsterSprite.complete ? '#f00' : '#f00';
                    if (monsterSprite.complete) {
                        ctx.drawImage(monsterSprite, monster.x * TILE - camera.x, monster.y * TILE - camera.y, TILE, TILE);
                    } else {
                        ctx.fillRect(monster.x * TILE - camera.x - 8, monster.y * TILE - camera.y - 8, 16, 16);
                    }
                }
            } catch (e) {
                console.error('Draw error:', e);
                ctx.fillStyle = '#f00';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.fillText('Render Error', 50, 50);
            }
        }

        function gameLoop() {
            try {
                if (gameState === 'cutscene') {
                    cutsceneFrame++;
                    drawCutscene();
                    if (cutsceneFrame >= INTRO_CUTSCENE_FRAMES) {
                        gameState = 'game';
                        ui.style.display = 'block';
                        cutsceneText.style.display = 'none';
                        bgMusic.play().catch(e => console.error('Audio play failed:', e));
                    }
                } else {
                    update();
                    draw();
                }
                requestAnimationFrame(gameLoop);
            } catch (e) {
                console.error('Game loop error:', e);
                ctx.fillStyle = '#f00';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.fillText('Game Loop Error', 50, 50);
            }
        }

        console.log('Starting game loop');
        document.getElementById('tasks').textContent = collected;
        document.getElementById('health').textContent = health;
        gameLoop();
    </script>
</body>
</html>
