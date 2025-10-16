
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Escape the Devil's Door</title>
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
            max-width: 100vw;
            max-height: 100vh;
        }
        #ui { 
            position: absolute; 
            top: 10px; 
            left: 10px; 
            font-size: 14px; 
            display: none; /* Hidden during cutscene */
        }
        #cutscene-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            font-size: 24px; /* Bigger for readability */
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
        <div>Health: <span id="health">150</span></div>
        <div>Controls: WASD or Arrows</div>
    </div>
    <div id="cutscene-text"></div>
    <script>
        const canvas = document.getElementById('game');
        const ctx = canvas.getContext('2d');
        ctx.imageSmoothingEnabled = false;

        const TILE = 32; // Larger tile for zoom-in feel (fewer tiles visible, world feels huge)
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
        const MAP_WIDTH = map[0].length; // 38
        const MAP_HEIGHT = map.length; // 35

        // Load textures
        let wallTexture = new Image();
        wallTexture.src = 'https://files.idyllic.app/files/static/2839122'; // Dark brick wall texture

        let floorTexture = new Image();
        floorTexture.src = 'https://opengameart.org/sites/default/files/preview_253.png'; // Stone floor texture, replace with a tileable one if needed

        // Player - start at open space
        let player = { px: 1 * TILE + TILE / 2, py: 1 * TILE + TILE / 2 };

        // Load player sprite
        let playerSprite = new Image();
        playerSprite.src = 'https://i.ibb.co/ym74XxwP/warrior-pixlated-Photoroom.png';

        // Load monster sprite (devil)
        let monsterSprite = new Image();
        monsterSprite.src = 'https://i.ibb.co/M57m3y52/pixlated-devil-Photoroom.png';

        // Items (tasks) - now as glowing cubes
        let items = [];
        for (let y = 0; y < MAP_HEIGHT; y++) {
            for (let x = 0; x < MAP_WIDTH; x++) {
                if (map[y][x] === 2) {
                    items.push({ x, y, collected: false });
                    map[y][x] = 0; // Set back to floor after collecting positions
                }
            }
        }
        let collected = 0;

        // Door
        let door = { x: 36, y: 34 };

        // Camera for scrolling/zoom feel
        let camera = { x: 0, y: 0 };

        // Multiple monsters (3 devils)
        let monsters = [];
        const devilPositions = [
            {x: 20, y: 1},
            {x: 5, y: 10},
            {x: 30, y: 15}
        ];
        for (let pos of devilPositions) {
            monsters.push({
                x: pos.x,
                y: pos.y,
                updateTimer: Math.random() * 60,
                path: null,
                pathTimer: 0
            });
        }

        let health = 150;
        let keys = {};
        let gameState = 'cutscene'; // Start with cutscene
        let cutsceneFrame = 0;
        const INTRO_CUTSCENE_FRAMES = 600;
        const COLLECT_CUTSCENE_FRAMES = 300;
        const WIN_CUTSCENE_FRAMES = 600;
        const GAMEOVER_CUTSCENE_FRAMES = 600;

        // Sound effects
        const bgMusic = new Audio('https://tabletopaudio.com/download.php?down=mp3&file=Undercroft.mp3'); // Example dark dungeon music, replace with actual URL
        bgMusic.loop = true;

        const collectSound = new Audio('https://www.orangefreesounds.com/wp-content/uploads/2020/02/Collect-item-sound-effect.mp3');

        const hurtSound = new Audio('https://freesound.org/data/previews/264/264982_4520574-lq.mp3');

        const winSound = new Audio('https://www.orangefreesounds.com/wp-content/uploads/2014/10/Winning-sound-effect.mp3');

        const doorSound = new Audio('https://freesound.org/data/previews/276/276537_5123857-lq.mp3');

        const cutsceneText = document.getElementById('cutscene-text');
        const ui = document.getElementById('ui');
        ui.style.display = 'none'; // Hide UI during cutscene

        document.addEventListener('keydown', e => {
            const key = e.key.toLowerCase();
            keys[key] = true;
            if (gameState === 'cutscene' && e.key === ' ') {
                gameState = 'game';
                ui.style.display = 'block';
                cutsceneText.style.display = 'none';
                bgMusic.play();
            } else if (gameState === 'collectCutscene' && e.key === ' ') {
                gameState = 'game';
                cutsceneText.style.display = 'none';
            } else if (gameState === 'winCutscene' && e.key === ' ') {
                alert('Congratulations! You escaped!');
                location.reload();
            } else if (gameState === 'gameOverCutscene' && e.key === ' ') {
                alert('Game Over! Try again.');
                location.reload();
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
                if (currentKey === null) break;
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

        function drawGlowingCube(wx, wy, glow = true) {
            const cx = wx + TILE / 2 - 8, cy = wy + TILE / 2 - 8;
            // Cube base (scaled)
            ctx.fillStyle = '#ff0';
            ctx.fillRect(cx, cy, 16, 16);
            // Edges
            ctx.fillStyle = '#fff';
            drawPixel(cx, cy, '#fff', 4);
            drawPixel(cx + 16, cy, '#fff', 2);
            drawPixel(cx, cy + 16, '#fff', 2);
            if (glow) {
                ctx.fillStyle = '#ffff00';
                for (let i = 1; i <= 6; i++) {
                    drawPixel(cx - i, cy - i, '#ffff00', 1);
                    drawPixel(cx + 16 + i - 1, cy - i, '#ffff00', 1);
                    drawPixel(cx - i, cy + 16 + i - 1, '#ffff00', 1);
                }
            }
        }

        function drawFogAndLight() {
            // Fog: dark overlay with hole around player (far from player)
            const fogGradient = ctx.createRadialGradient(
                player.px - camera.x, player.py - camera.y, 100, // inner clear radius (close to player)
                player.px - camera.x, player.py - camera.y, 300  // outer foggy radius (far away)
            );
            fogGradient.addColorStop(0, 'rgba(0,0,0,0)');
            fogGradient.addColorStop(0.7, 'rgba(0,0,0,0.3)'); // semi-transparent far away
            fogGradient.addColorStop(1, 'rgba(0,0,0,0.8)'); // full dark fog

            ctx.fillStyle = fogGradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Light around player (close/mid range)
            const lightGradient = ctx.createRadialGradient(
                player.px - camera.x, player.py - camera.y, 0,
                player.px - camera.x, player.py - camera.y, 80  // light radius close/mid
            );
            lightGradient.addColorStop(0, 'rgba(255, 215, 0, 0.8)');
            lightGradient.addColorStop(0.5, 'rgba(255, 215, 0, 0.4)');
            lightGradient.addColorStop(1, 'rgba(255, 215, 0, 0)');

            ctx.fillStyle = lightGradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        function drawDirectionLine() {
            if (collected < 3 || gameState !== 'game') return;

            const doorCenterX = door.x * TILE + TILE / 2;
            const doorCenterY = door.y * TILE + TILE / 2;
            const dx = doorCenterX - player.px;
            const dy = doorCenterY - player.py;
            const distance = Math.sqrt(dx * dx + dy * dy);

            if (distance < TILE) return; // Close enough, no line

            const playerScreenX = player.px - camera.x;
            const playerScreenY = player.py - camera.y;
            const doorScreenX = doorCenterX - camera.x;
            const doorScreenY = doorCenterY - camera.y;

            ctx.strokeStyle = '#ffd700';
            ctx.lineWidth = 2;
            ctx.setLineDash([5, 5]); // Dashed line for guide
            ctx.beginPath();
            ctx.moveTo(playerScreenX, playerScreenY);
            ctx.lineTo(doorScreenX, doorScreenY);
            ctx.stroke();
            ctx.setLineDash([]); // Reset to solid
        }

        function drawCutscene() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Phase 1: Wandering in forest (0-200)
            if (cutsceneFrame < 200) {
                ctx.fillStyle = '#006400';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Moving trees
                for (let i = 0; i < 15; i++) {
                    const tx = (i * 60 + cutsceneFrame * 1.5) % (canvas.width + 60) - 30;
                    const ty = 400 + Math.sin((cutsceneFrame * 0.03) + i * 0.5) * 20;
                    // Trunk
                    ctx.fillStyle = '#8B4513';
                    ctx.fillRect(tx, ty, 10, 30);
                    // Leaves
                    ctx.fillStyle = '#228B22';
                    ctx.beginPath();
                    ctx.arc(tx + 5, ty - 10, 20, 0, Math.PI * 2);
                    ctx.fill();
                }
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('Lost in the enchanted forest,', canvas.width / 2, canvas.height / 2 - 50);
                ctx.fillText('you seek a way out.', canvas.width / 2, canvas.height / 2 + 20);
            } 
            // Phase 2: Discovering the door (200-400)
            else if (cutsceneFrame < 400) {
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#ffd700';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('A glowing door appears in the shadows...', canvas.width / 2, canvas.height / 2 - 50);
                // Draw door
                ctx.fillStyle = '#8B0000';
                ctx.fillRect(canvas.width / 2 - 20, canvas.height / 2 + 20, 40, 80);
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(canvas.width / 2 - 10, canvas.height / 2 + 20, 20, 10); // handle
            } 
            // Phase 3: Devil appears (400-600)
            else if (cutsceneFrame < 600) {
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Creepy red overlay
                const fadeStart = 400;
                const fadeDuration = 200;
                let alpha = Math.min(1, (cutsceneFrame - fadeStart) / fadeDuration);
                ctx.fillStyle = `rgba(139, 0, 0, ${alpha * 0.3})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#f00';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('But the Devil stands guard!', canvas.width / 2, canvas.height / 2 - 100);
                // Door
                ctx.fillStyle = '#8B0000';
                ctx.fillRect(canvas.width / 2 - 25, canvas.height / 2 - 20, 50, 100);
                // Handle
                ctx.fillStyle = '#ff0';
                ctx.fillRect(canvas.width / 2 - 5, canvas.height / 2 + 20, 10, 10);
                // Red eyes fade in
                const eyeAlpha = Math.min(1, (cutsceneFrame - fadeStart) / fadeDuration);
                const eyeX = canvas.width / 2;
                const eyeY = canvas.height / 2 + 10;
                ctx.save();
                ctx.fillStyle = `rgba(255, 0, 0, ${eyeAlpha})`;
                // Left eye
                ctx.beginPath();
                ctx.arc(eyeX - 10, eyeY, 8, 0, Math.PI * 2);
                ctx.fill();
                // Right eye
                ctx.beginPath();
                ctx.arc(eyeX + 10, eyeY, 8, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            }
            ctx.textAlign = 'left';
        }

        function drawCollectCutscene() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#111';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Three cubes merging animation
            const progress = cutsceneFrame / COLLECT_CUTSCENE_FRAMES;
            const cubeSize = 32 * (1 - progress * 0.5);
            const glowIntensity = Math.sin(cutsceneFrame * 0.2) * 0.5 + 0.5;

            // Draw three cubes converging to center
            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;
            const positions = [
                {x: centerX - 60, y: centerY - 30},
                {x: centerX, y: centerY - 30},
                {x: centerX + 60, y: centerY - 30}
            ];

            for (let i = 0; i < 3; i++) {
                let px = positions[i].x + (centerX - positions[i].x) * progress;
                let py = positions[i].y + (centerY - positions[i].y) * progress;
                ctx.save();
                ctx.translate(px, py);
                ctx.scale(cubeSize / 32, cubeSize / 32);
                drawGlowingCube(0, 0, false);
                ctx.restore();
                // Glow
                ctx.fillStyle = `rgba(255, 255, 0, ${glowIntensity * 0.5})`;
                ctx.beginPath();
                ctx.arc(px, py, 20 * glowIntensity, 0, Math.PI * 2);
                ctx.fill();
            }

            // Merged key or something at center
            if (progress > 0.8) {
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(centerX - 10, centerY - 10, 20, 20);
                ctx.fillStyle = '#fff';
                ctx.font = '16px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('Key Formed!', centerX, centerY + 40);
            }

            ctx.fillStyle = '#fff';
            ctx.font = '24px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('All cubes collected! The key is formed.', canvas.width / 2, 100);
            ctx.fillText('Now head to the door!', canvas.width / 2, 130);

            ctx.textAlign = 'left';
        }

        function drawWinCutscene() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Play win sound and door sound
            if (cutsceneFrame === 1) {
                doorSound.play();
                winSound.play();
            }

            // Phase 1: Entering the door (0-200)
            if (cutsceneFrame < 200) {
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Glowing door
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(canvas.width / 2 - 25, canvas.height / 2 - 50, 50, 100);
                // Player sprite entering (simple animation)
                const playerX = canvas.width / 2 - 25 + (cutsceneFrame * 0.5);
                if (playerSprite.complete) {
                    ctx.drawImage(playerSprite, playerX, canvas.height / 2 - 16, 32, 32);
                } else {
                    ctx.fillStyle = '#00f';
                    ctx.fillRect(playerX, canvas.height / 2 - 8, 16, 16);
                }
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('You dash through the door!', canvas.width / 2, canvas.height / 2 - 100);
            } 
            // Phase 2: Devil chases (200-400)
            else if (cutsceneFrame < 400) {
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Door closing
                const doorAlpha = Math.min(1, (cutsceneFrame - 200) / 200);
                ctx.save();
                ctx.globalAlpha = 1 - doorAlpha;
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(canvas.width / 2 - 25, canvas.height / 2 - 50, 50, 100);
                ctx.restore();
                // Devil appearing behind
                const devilX = canvas.width / 2 + 50 + Math.sin((cutsceneFrame - 200) * 0.1) * 10;
                if (monsterSprite.complete) {
                    ctx.drawImage(monsterSprite, devilX, canvas.height / 2, 32, 32);
                } else {
                    ctx.fillStyle = '#f00';
                    ctx.fillRect(devilX, canvas.height / 2, 16, 16);
                }
                // Red eyes
                ctx.fillStyle = '#f00';
                ctx.beginPath();
                ctx.arc(devilX + 5, canvas.height / 2 + 5, 3, 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(devilX + 11, canvas.height / 2 + 5, 3, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('The Devil lunges, but too late!', canvas.width / 2, canvas.height / 2 - 100);
            } 
            // Phase 3: Escaping into light (400-600)
            else if (cutsceneFrame < 600) {
                // Bright light overlay
                const lightAlpha = Math.min(1, (cutsceneFrame - 400) / 200);
                ctx.fillStyle = `rgba(255, 255, 255, ${lightAlpha * 0.8})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Player running forward (pixelated animation)
                const playerRunX = (cutsceneFrame - 400) * 2;
                const playerRunY = canvas.height / 2 + Math.sin((cutsceneFrame - 400) * 0.2) * 5;
                if (playerSprite.complete) {
                    ctx.globalAlpha = 1 - lightAlpha * 0.5;
                    ctx.drawImage(playerSprite, playerRunX, playerRunY - 16, 32, 32);
                } else {
                    ctx.fillStyle = '#00f';
                    ctx.globalAlpha = 1 - lightAlpha * 0.5;
                    ctx.fillRect(playerRunX, playerRunY - 8, 16, 16);
                }
                ctx.globalAlpha = 1;
                // Forest fading in
                ctx.fillStyle = `rgba(0, 100, 0, ${lightAlpha})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Trees appearing
                for (let i = 0; i < 10; i++) {
                    const tx = playerRunX + i * 60 + (cutsceneFrame - 400) * 1;
                    const ty = canvas.height - 100 + Math.sin((cutsceneFrame * 0.03) + i * 0.5) * 10;
                    ctx.fillStyle = `rgba(139, 69, 19, ${lightAlpha})`;
                    ctx.fillRect(tx, ty, 10, 30);
                    ctx.fillStyle = `rgba(34, 139, 34, ${lightAlpha})`;
                    ctx.beginPath();
                    ctx.arc(tx + 5, ty - 10, 20, 0, Math.PI * 2);
                    ctx.fill();
                }
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('You escape the Devil\'s domain!', canvas.width / 2, 100);
                ctx.fillText('Freedom awaits...', canvas.width / 2, 130);
            }
            ctx.textAlign = 'left';
        }

        function drawGameOverCutscene() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#111';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Devil grabbing player animation
            const progress = cutsceneFrame / GAMEOVER_CUTSCENE_FRAMES;
            const shake = Math.sin(cutsceneFrame * 0.3) * 5;

            // Player in center
            const playerX = canvas.width / 2 + shake;
            const playerY = canvas.height / 2 + shake;
            if (playerSprite.complete) {
                ctx.drawImage(playerSprite, playerX - 16, playerY - 16, 32, 32);
            } else {
                ctx.fillStyle = '#00f';
                ctx.fillRect(playerX - 8, playerY - 8, 16, 16);
            }

            // Devil closing in
            const devilScale = 1 + progress * 0.5;
            const devilX = canvas.width / 2;
            const devilY = canvas.height / 2;
            ctx.save();
            ctx.translate(devilX, devilY);
            ctx.scale(devilScale, devilScale);
            if (monsterSprite.complete) {
                ctx.drawImage(monsterSprite, -16, -16, 32, 32);
            } else {
                ctx.fillStyle = '#f00';
                ctx.fillRect(-8, -8, 16, 16);
            }
            ctx.restore();

            // Red flash
            if (progress > 0.5) {
                const flashAlpha = Math.sin(cutsceneFrame * 0.1) * 0.3;
                ctx.fillStyle = `rgba(255, 0, 0, ${flashAlpha})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }

            ctx.fillStyle = '#f00';
            ctx.font = '24px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('The Devil catches you!', canvas.width / 2, 100);
            ctx.fillText('Your soul is claimed...', canvas.width / 2, 130);

            ctx.textAlign = 'left';
        }

        function update() {
            const speed = 3; // pixels per frame
            let vx = 0, vy = 0;
            if (keys['a'] || keys['arrowleft']) vx = -speed;
            if (keys['d'] || keys['arrowright']) vx = speed;
            if (keys['w'] || keys['arrowup']) vy = -speed;
            if (keys['s'] || keys['arrowdown']) vy = speed;
            // Normalize diagonal movement
            if (vx !== 0 && vy !== 0) {
                vx /= Math.sqrt(2);
                vy /= Math.sqrt(2);
            }

            // Move X
            let newPx = player.px + vx;
            let tx = Math.floor(newPx / TILE);
            let ty = Math.floor(player.py / TILE);
            if (tx >= 0 && tx < MAP_WIDTH && map[ty][tx] !== 1) {
                player.px = newPx;
            }

            // Move Y
            let newPy = player.py + vy;
            tx = Math.floor(player.px / TILE);
            ty = Math.floor(newPy / TILE);
            if (ty >= 0 && ty < MAP_HEIGHT && map[ty][tx] !== 1) {
                player.py = newPy;
            }

            const playerTileX = Math.floor(player.px / TILE);
            const playerTileY = Math.floor(player.py / TILE);

            // Collect items
            let justCollected = false;
            for (let item of items) {
                if (!item.collected && playerTileX === item.x && playerTileY === item.y) {
                    item.collected = true;
                    collected++;
                    justCollected = true;
                    document.getElementById('tasks').textContent = collected;
                    collectSound.play();
                }
            }

            // Trigger collect cutscene if third item collected
            if (justCollected && collected >= 3 && gameState === 'game') {
                gameState = 'collectCutscene';
                ui.style.display = 'none';
                cutsceneText.style.display = 'block';
                cutsceneFrame = 0;
            }

            // Check win condition
            if (playerTileX === door.x && playerTileY === door.y && collected >= 3) {
                gameState = 'winCutscene';
                ui.style.display = 'none';
                cutsceneFrame = 0;
            }

            // Update each monster
            const chaseDist = collected >= 3 ? 20 : 10;
            for (let monster of monsters) {
                const distToPlayer = Math.abs(monster.x - playerTileX) + Math.abs(monster.y - playerTileY);
                let isChasing = distToPlayer < chaseDist;
                if (isChasing) {
                    monster.pathTimer++;
                    if (monster.pathTimer > (collected >= 3 ? 15 : 30)) { // Faster path update when chasing with key
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
                        // Random move or patrol
                        const neighbors = getNeighbors({x: monster.x, y: monster.y});
                        if (neighbors.length > 0) {
                            nextPos = neighbors[Math.floor(Math.random() * neighbors.length)];
                        }
                    }
                    if (nextPos) {
                        monster.x = nextPos.x;
                        monster.y = nextPos.y;
                    }
                    // Check collision with player
                    if (playerTileX === monster.x && playerTileY === monster.y) {
                        health -= 30;
                        document.getElementById('health').textContent = health;
                        hurtSound.play();
                        if (health <= 0) {
                            gameState = 'gameOverCutscene';
                            ui.style.display = 'none';
                            cutsceneFrame = 0;
                        }
                    }
                }
            }
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Update camera to follow player
            const camX = player.px - canvas.width / 2;
            const camY = player.py - canvas.height / 2;
            camera.x = Math.max(0, Math.min(camX, MAP_WIDTH * TILE - canvas.width));
            camera.y = Math.max(0, Math.min(camY, MAP_HEIGHT * TILE - canvas.height));

            // Draw map tiles with textures
            const startX = Math.max(0, Math.floor(camera.x / TILE));
            const endX = Math.min(MAP_WIDTH, startX + Math.ceil(canvas.width / TILE) + 1);
            const startY = Math.max(0, Math.floor(camera.y / TILE));
            const endY = Math.min(MAP_HEIGHT, startY + Math.ceil(canvas.height / TILE) + 1);

            for (let y = startY; y < endY; y++) {
                for (let x = startX; x < endX; x++) {
                    const tileX = x * TILE - camera.x;
                    const tileY = y * TILE - camera.y;
                    if (map[y][x] === 1) {
                        if (wallTexture.complete) {
                            ctx.drawImage(wallTexture, tileX, tileY, TILE, TILE);
                        } else {
                            ctx.fillStyle = '#555';
                            ctx.fillRect(tileX, tileY, TILE, TILE);
                        }
                    } else if (map[y][x] === 0) {
                        if (floorTexture.complete) {
                            ctx.drawImage(floorTexture, tileX, tileY, TILE, TILE);
                        } else {
                            ctx.fillStyle = '#222';
                            ctx.fillRect(tileX, tileY, TILE, TILE);
                        }
                    }
                }
            }

            // Draw items
            for (let item of items) {
                if (!item.collected) {
                    const ix = item.x * TILE - camera.x;
                    const iy = item.y * TILE - camera.y;
                    drawGlowingCube(ix, iy);
                }
            }

            // Draw door - red golden door
            const dx = door.x * TILE - camera.x;
            const dy = door.y * TILE - camera.y;
            // Door background red
            ctx.fillStyle = '#8B0000';
            ctx.fillRect(dx, dy, TILE, TILE);
            // Golden frame
            ctx.strokeStyle = '#ffd700';
            ctx.lineWidth = 2;
            ctx.strokeRect(dx, dy, TILE, TILE);
            // Golden handle
            ctx.fillStyle = '#ffd700';
            ctx.fillRect(dx + TILE - 8, dy + TILE / 2 - 4, 6, 8);

            // Draw player
            if (playerSprite.complete) {
                ctx.drawImage(playerSprite, player.px - camera.x - 16, player.py - camera.y - 16, 32, 32);
            } else {
                ctx.fillStyle = '#00f';
                ctx.fillRect(player.px - camera.x - 8, player.py - camera.y - 8, 16, 16);
            }

            // Draw monsters (simple, no body to avoid duplication glitch)
            if (monsterSprite.complete) {
                for (let monster of monsters) {
                    ctx.drawImage(monsterSprite, monster.x * TILE - camera.x, monster.y * TILE - camera.y, TILE, TILE);
                }
            } else {
                // Placeholder
                for (let monster of monsters) {
                    ctx.fillStyle = '#f00';
                    ctx.fillRect(monster.x * TILE - camera.x - 8, monster.y * TILE - camera.y - 8, 16, 16);
                }
            }

            // Draw fog and light effects
            drawFogAndLight();

            // Draw direction line to door if key collected
            drawDirectionLine();
        }

        function gameLoop() {
            if (gameState === 'cutscene') {
                cutsceneFrame++;
                drawCutscene();
                if (cutsceneFrame >= INTRO_CUTSCENE_FRAMES) {
                    gameState = 'game';
                    ui.style.display = 'block';
                    bgMusic.play();
                }
            } else if (gameState === 'collectCutscene') {
                cutsceneFrame++;
                drawCollectCutscene();
                if (cutsceneFrame >= COLLECT_CUTSCENE_FRAMES) {
                    gameState = 'game';
                    ui.style.display = 'block';
                }
            } else if (gameState === 'winCutscene') {
                cutsceneFrame++;
                drawWinCutscene();
                if (cutsceneFrame >= WIN_CUTSCENE_FRAMES) {
                    // Keep showing last frame until space pressed
                }
            } else if (gameState === 'gameOverCutscene') {
                cutsceneFrame++;
                drawGameOverCutscene();
                if (cutsceneFrame >= GAMEOVER_CUTSCENE_FRAMES) {
                    // Keep showing last frame until space pressed
                }
            } else {
                update();
                draw();
            }
            requestAnimationFrame(gameLoop);
        }

        // Start the game loop
        gameLoop();

        // Update UI initially
        document.getElementById('tasks').textContent = collected;
        document.getElementById('health').textContent = health;
    </script>
</body>
</html>         
