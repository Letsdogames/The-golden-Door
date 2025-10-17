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
        <div>Controls: WASD or Arrows (R to restart)</div>
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

        // Clear any 2 or 3 to 0 for randomization
        for (let y = 0; y < MAP_HEIGHT; y++) {
            for (let x = 0; x < MAP_WIDTH; x++) {
                if (map[y][x] === 2 || map[y][x] === 3) {
                    map[y][x] = 0;
                }
            }
        }

        // Find open positions
        let openPositions = [];
        for (let y = 0; y < MAP_HEIGHT; y++) {
            for (let x = 0; x < MAP_WIDTH; x++) {
                if (map[y][x] === 0 && !(x === 1 && y === 1)) { // Avoid player start
                    openPositions.push({x, y});
                }
            }
        }

        // Shuffle function
        function shuffle(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
        }
        shuffle(openPositions);

        // Pick 3 for items, 1 for door
        let itemPos = openPositions.slice(0, 3);
        let doorPos = openPositions[3] || {x: 36, y: 34}; // Fallback

        // Load textures
        let wallTexture = new Image();
        wallTexture.src = 'https://files.idyllic.app/files/static/2839122'; // Dark brick wall texture

        let floorTexture = new Image();
        floorTexture.src = 'https://i.ibb.co/G6DRQdr/floor.jpg'; // New floor texture

        // Player - start at open space
        let player = { px: 1 * TILE + TILE / 2, py: 1 * TILE + TILE / 2 };

        // Load player sprite
        let playerSprite = new Image();
        playerSprite.src = 'https://i.ibb.co/ym74XxwP/warrior-pixlated-Photoroom.png';

        // Load monster sprite (devil)
        let monsterSprite = new Image();
        monsterSprite.src = 'https://i.ibb.co/M57m3y52/pixlated-devil-Photoroom.png';

        // Items (tasks) - now as glowing cubes
        let items = itemPos.map(pos => ({ x: pos.x, y: pos.y, collected: false }));
        let collected = 0;

        // Door
        let door = { x: doorPos.x, y: doorPos.y };

        // Camera for scrolling/zoom feel
        let camera = { x: 0, y: 0 };

        // Multiple monsters (3 devils)
        let monsters = [];
        const devilPositions = [
            {x: 20, y: 1},
            {x: 5, y: 11},
            {x: 30, y: 16}
        ];
        for (let index = 0; index < devilPositions.length; index++) {
            let pos = devilPositions[index];
            let patrolPoints = [];
            if (index === 0) {
                patrolPoints = [{x: 18, y: 1}, {x: 22, y: 1}, {x: 20, y: 3}];
            } else if (index === 1) {
                patrolPoints = [{x: 3, y: 11}, {x: 7, y: 11}, {x: 5, y: 14}];
            } else {
                patrolPoints = [{x: 28, y: 16}, {x: 32, y: 16}, {x: 30, y: 19}];
            }
            monsters.push({
                x: pos.x,
                y: pos.y,
                updateTimer: Math.random() * 60,
                path: null,
                pathTimer: 0,
                freezeTimer: 0,
                patrolPoints: patrolPoints,
                patrolIndex: 0
            });
        }

        let health = 150;
        let keys = {};
        let gameState = 'cutscene'; // Start with cutscene
        let cutsceneFrame = 0;
        const INTRO_CUTSCENE_FRAMES = 900; // Extended for better animation
        const COLLECT_CUTSCENE_FRAMES = 450; // Extended
        const WIN_CUTSCENE_FRAMES = 900; // Extended
        const GAMEOVER_CUTSCENE_FRAMES = 750; // Extended
        let warningTimer = 0;

        // Particles for collection effects
        let particles = [];

        function createParticles(x, y) {
            for (let i = 0; i < 10; i++) {
                particles.push({
                    x: x * TILE + TILE / 2,
                    y: y * TILE + TILE / 2,
                    vx: (Math.random() - 0.5) * 4,
                    vy: (Math.random() - 0.5) * 4,
                    life: 30,
                    color: '#ff0'
                });
            }
        }

        function updateParticles() {
            for (let i = particles.length - 1; i >= 0; i--) {
                let p = particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.vy += 0.1; // gravity
                p.life--;
                if (p.life <= 0) {
                    particles.splice(i, 1);
                }
            }
        }

        // Easing function for smoother animations
        function easeInOutCubic(t) {
            return t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2;
        }

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

        function resetGame() {
            // Re-randomize positions
            shuffle(openPositions);
            itemPos = openPositions.slice(0, 3);
            doorPos = openPositions[3] || {x: 36, y: 34};
            items = itemPos.map(pos => ({ x: pos.x, y: pos.y, collected: false }));
            door = { x: doorPos.x, y: doorPos.y };
            player = { px: 1 * TILE + TILE / 2, py: 1 * TILE + TILE / 2 };
            collected = 0;
            health = 150;
            for (let item of items) item.collected = false;
            for (let index = 0; index < monsters.length; index++) {
                let m = monsters[index];
                let pos = devilPositions[index];
                m.x = pos.x;
                m.y = pos.y;
                m.path = null;
                m.pathTimer = 0;
                m.updateTimer = Math.random() * 60;
                m.freezeTimer = 0;
                m.patrolIndex = 0;
            }
            particles = [];
            warningTimer = 0;
            gameState = 'game';
            ui.style.display = 'block';
            cutsceneFrame = 0;
            document.getElementById('tasks').textContent = collected;
            document.getElementById('health').textContent = health;
            bgMusic.play();
        }

        document.addEventListener('keydown', e => {
            const key = e.key.toLowerCase();
            keys[key] = true;
            if (key === 'r') {
                if (gameState === 'game') {
                    resetGame();
                }
                return;
            }
            if (gameState === 'cutscene' && key === ' ') {
                gameState = 'game';
                ui.style.display = 'block';
                cutsceneText.style.display = 'none';
                bgMusic.play();
            } else if (gameState === 'collectCutscene' && key === ' ') {
                gameState = 'game';
                cutsceneText.style.display = 'none';
            } else if (gameState === 'winCutscene' && key === ' ') {
                resetGame();
            } else if (gameState === 'gameOverCutscene' && key === ' ') {
                resetGame();
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
            const lightRadius = collected >= 3 ? 60 : 80;
            const fogColor = collected >= 3 ? 'rgba(139,0,0,0.9)' : 'rgba(0,0,0,0.8)';

            // Fog: dark/red overlay with hole around player (far from player)
            const fogGradient = ctx.createRadialGradient(
                player.px - camera.x, player.py - camera.y, 100, // inner clear radius (close to player)
                player.px - camera.x, player.py - camera.y, 300  // outer foggy radius (far away)
            );
            fogGradient.addColorStop(0, 'rgba(0,0,0,0)');
            fogGradient.addColorStop(0.7, 'rgba(0,0,0,0.3)'); // semi-transparent far away
            fogGradient.addColorStop(1, fogColor); // full dark/red fog

            ctx.fillStyle = fogGradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Light around player (close/mid range)
            const lightGradient = ctx.createRadialGradient(
                player.px - camera.x, player.py - camera.y, 0,
                player.px - camera.x, player.py - camera.y, lightRadius  // light radius close/mid
            );
            lightGradient.addColorStop(0, 'rgba(255, 215, 0, 0.8)');
            lightGradient.addColorStop(0.5, 'rgba(255, 215, 0, 0.4)');
            lightGradient.addColorStop(1, 'rgba(255, 215, 0, 0)');

            ctx.fillStyle = lightGradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        function drawWarning() {
            if (warningTimer <= 0) return;
            warningTimer--;
            ctx.save();
            ctx.shadowColor = 'darkred';
            ctx.shadowBlur = 10;
            ctx.fillStyle = '#8B0000';
            ctx.font = 'bold 28px monospace';
            ctx.textAlign = 'center';
            // Bloody effect: multiple offsets
            const text = 'RUN! The monsters awaken!';
            const x = canvas.width / 2;
            const y = 40;
            for (let dx = -2; dx <= 2; dx += 2) {
                for (let dy = -1; dy <= 1; dy += 2) {
                    ctx.fillText(text, x + dx, y + dy);
                }
            }
            // Main text
            ctx.fillStyle = '#FF0000';
            ctx.shadowBlur = 5;
            ctx.fillText(text, x, y);
            ctx.restore();
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

            const progress = cutsceneFrame / INTRO_CUTSCENE_FRAMES;

            // Phase 1: Wandering in forest (0-300 frames, extended)
            if (cutsceneFrame < 300) {
                const phaseProgress = cutsceneFrame / 300;
                const eased = easeInOutCubic(phaseProgress);
                ctx.fillStyle = '#006400';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Moving trees with more sway
                for (let i = 0; i < 20; i++) { // More trees
                    const tx = (i * 40 + cutsceneFrame * 2) % (canvas.width + 80) - 40;
                    const ty = 350 + Math.sin((cutsceneFrame * 0.05) + i * 0.3) * 30 * eased; // More sway
                    const trunkHeight = 40 * eased;
                    // Trunk
                    ctx.fillStyle = '#8B4513';
                    ctx.fillRect(tx, ty, 12, trunkHeight);
                    // Leaves with bob
                    ctx.fillStyle = '#228B22';
                    ctx.save();
                    ctx.translate(tx + 6, ty - 10);
                    ctx.rotate(Math.sin(cutsceneFrame * 0.02 + i) * 0.1 * eased);
                    ctx.beginPath();
                    ctx.arc(0, 0, 25 * eased, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.restore();
                }
                // Simple player walking animation
                const playerWalkX = (canvas.width / 2 - 50) + Math.sin(cutsceneFrame * 0.1) * 20 * eased;
                const playerWalkY = canvas.height / 2 + 50;
                if (playerSprite.complete) {
                    ctx.save();
                    ctx.translate(playerWalkX, playerWalkY);
                    ctx.rotate(Math.sin(cutsceneFrame * 0.2) * 0.2);
                    ctx.drawImage(playerSprite, -16, -16, 32, 32);
                    ctx.restore();
                } else {
                    ctx.fillStyle = '#00f';
                    ctx.fillRect(playerWalkX - 8, playerWalkY - 8, 16, 16);
                }
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('Lost in the enchanted forest,', canvas.width / 2, canvas.height / 2 - 50);
                ctx.fillText('you seek a way out.', canvas.width / 2, canvas.height / 2 + 20);
            } 
            // Phase 2: Discovering the door (300-600 frames)
            else if (cutsceneFrame < 600) {
                const phaseProgress = (cutsceneFrame - 300) / 300;
                const eased = easeInOutCubic(phaseProgress);
                // Fade from green to dark
                const greenAlpha = 1 - eased;
                ctx.fillStyle = `rgba(0, 100, 0, ${greenAlpha})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#ffd700';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('A glowing door appears in the shadows...', canvas.width / 2, canvas.height / 2 - 50);
                // Draw door with scale in
                const doorScale = eased;
                const doorX = canvas.width / 2 - 20 * doorScale;
                const doorY = canvas.height / 2 + 20;
                const doorW = 40 * doorScale;
                const doorH = 80 * doorScale;
                ctx.save();
                ctx.translate(canvas.width / 2, canvas.height / 2 + 20);
                ctx.scale(doorScale, doorScale);
                ctx.fillStyle = '#8B0000';
                ctx.fillRect(-20, 0, 40, 80);
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(-10, 0, 20, 10); // handle
                ctx.restore();
                // Player approaching door
                const playerApproachX = canvas.width / 2 - 100 + 50 * eased;
                if (playerSprite.complete) {
                    ctx.drawImage(playerSprite, playerApproachX, canvas.height / 2 - 16, 32, 32);
                } else {
                    ctx.fillStyle = '#00f';
                    ctx.fillRect(playerApproachX, canvas.height / 2 - 8, 16, 16);
                }
            } 
            // Phase 3: Devil appears (600-900 frames)
            else {
                const phaseProgress = (cutsceneFrame - 600) / 300;
                const eased = easeInOutCubic(phaseProgress);
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Creepy red overlay with pulse
                const alpha = Math.min(1, eased) * (0.3 + Math.sin(cutsceneFrame * 0.05) * 0.1);
                ctx.fillStyle = `rgba(139, 0, 0, ${alpha})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#f00';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('But the Devil stands guard!', canvas.width / 2, canvas.height / 2 - 100);
                // Door with glow
                ctx.save();
                ctx.shadowColor = '#ffd700';
                ctx.shadowBlur = 20 * eased;
                ctx.fillStyle = '#8B0000';
                ctx.fillRect(canvas.width / 2 - 25, canvas.height / 2 - 20, 50, 100);
                ctx.fillStyle = '#ff0';
                ctx.fillRect(canvas.width / 2 - 5, canvas.height / 2 + 20, 10, 10);
                ctx.restore();
                // Red eyes fade in with scale
                const eyeScale = 1 + eased * 0.5;
                const eyeX = canvas.width / 2;
                const eyeY = canvas.height / 2 + 10;
                ctx.save();
                ctx.scale(eyeScale, eyeScale);
                ctx.fillStyle = '#f00';
                // Left eye
                ctx.beginPath();
                ctx.arc((eyeX - 10)/eyeScale, eyeY/eyeScale, 8, 0, Math.PI * 2);
                ctx.fill();
                // Right eye
                ctx.beginPath();
                ctx.arc((eyeX + 10)/eyeScale, eyeY/eyeScale, 8, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
                // Devil silhouette emerging
                const devilY = canvas.height / 2 + 50 - eased * 100;
                if (monsterSprite.complete) {
                    ctx.globalAlpha = eased;
                    ctx.drawImage(monsterSprite, canvas.width / 2 - 16, devilY - 16, 32, 32);
                    ctx.globalAlpha = 1;
                } else {
                    ctx.fillStyle = 'rgba(255, 0, 0, ' + eased + ')';
                    ctx.fillRect(canvas.width / 2 - 8, devilY - 8, 16, 16);
                }
            }
            // Prompt text with fade in
            const promptAlpha = Math.min(1, cutsceneFrame / 100);
            ctx.save();
            ctx.globalAlpha = promptAlpha;
            ctx.fillStyle = '#ffd700';
            ctx.font = '18px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('Press SPACE to begin', canvas.width / 2, canvas.height - 50);
            ctx.restore();
            ctx.textAlign = 'left';
        }

        function drawCollectCutscene() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#111';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            const progress = cutsceneFrame / COLLECT_CUTSCENE_FRAMES;
            const eased = easeInOutCubic(progress);
            const cubeSize = 32 * (1 - eased * 0.5);
            const glowIntensity = Math.sin(cutsceneFrame * 0.2) * 0.5 + 0.5;
            const rotation = cutsceneFrame * 0.05;

            // Draw three cubes converging to center with rotation and scale
            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;
            const positions = [
                {x: centerX - 60, y: centerY - 30},
                {x: centerX, y: centerY - 30},
                {x: centerX + 60, y: centerY - 30}
            ];

            for (let i = 0; i < 3; i++) {
                let px = positions[i].x + (centerX - positions[i].x) * eased;
                let py = positions[i].y + (centerY - positions[i].y) * eased;
                ctx.save();
                ctx.translate(px, py);
                ctx.rotate(rotation * (i + 1));
                ctx.scale(cubeSize / 32, cubeSize / 32);
                drawGlowingCube(0, 0, false);
                ctx.restore();
                // Enhanced glow with pulse
                const glowAlpha = glowIntensity * (0.5 + Math.sin(cutsceneFrame * 0.1) * 0.2);
                ctx.fillStyle = `rgba(255, 255, 0, ${glowAlpha})`;
                ctx.beginPath();
                ctx.arc(px, py, 20 * glowIntensity * eased, 0, Math.PI * 2);
                ctx.fill();
                // Trail effect
                ctx.fillStyle = `rgba(255, 255, 0, ${0.3 * (1 - eased)})`;
                ctx.beginPath();
                ctx.arc(positions[i].x, positions[i].y, 10, 0, Math.PI * 2);
                ctx.fill();
            }

            // Merged key with shine animation
            if (eased > 0.7) {
                const keyScale = 1 + Math.sin(cutsceneFrame * 0.3) * 0.2;
                ctx.save();
                ctx.translate(centerX, centerY);
                ctx.scale(keyScale, keyScale);
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(-10, -10, 20, 20);
                ctx.fillStyle = '#fff';
                ctx.font = '16px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('Key Formed!', 0, 40);
                ctx.restore();
            }

            ctx.fillStyle = '#fff';
            ctx.font = '24px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('All cubes collected! The key is formed.', canvas.width / 2, 100);
            ctx.fillText('Now head to the door!', canvas.width / 2, 130);

            // Prompt text
            ctx.fillStyle = '#ffd700';
            ctx.font = '18px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('Press SPACE to continue', canvas.width / 2, canvas.height - 50);
            ctx.textAlign = 'left';
        }

        function drawWinCutscene() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Play win sound and door sound
            if (cutsceneFrame === 1) {
                doorSound.play();
                winSound.play();
            }

            const progress = cutsceneFrame / WIN_CUTSCENE_FRAMES;
            const phase = Math.floor(progress * 3);

            // Phase 1: Entering the door (0-300 frames)
            if (cutsceneFrame < 300) {
                const phaseProgress = cutsceneFrame / 300;
                const eased = easeInOutCubic(phaseProgress);
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Glowing door with pulse
                const glow = Math.sin(cutsceneFrame * 0.1) * 0.3 + 0.7;
                ctx.save();
                ctx.shadowColor = '#ffd700';
                ctx.shadowBlur = 30 * glow;
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(canvas.width / 2 - 25, canvas.height / 2 - 50, 50, 100);
                ctx.restore();
                // Player sprite entering with acceleration
                const playerX = canvas.width / 2 - 25 + (cutsceneFrame * 1.5) * eased; // Faster with ease
                const playerY = canvas.height / 2 - 16 + Math.sin(cutsceneFrame * 0.3) * 3; // Slight bob
                if (playerSprite.complete) {
                    ctx.save();
                    ctx.translate(playerX, playerY);
                    ctx.rotate(Math.sin(cutsceneFrame * 0.4) * 0.1);
                    ctx.drawImage(playerSprite, -16, -16, 32, 32);
                    ctx.restore();
                } else {
                    ctx.fillStyle = '#00f';
                    ctx.fillRect(playerX - 8, playerY - 8, 16, 16);
                }
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('You dash through the door!', canvas.width / 2, canvas.height / 2 - 100);
            } 
            // Phase 2: Devil chases (300-600 frames)
            else if (cutsceneFrame < 600) {
                const phaseProgress = (cutsceneFrame - 300) / 300;
                const eased = easeInOutCubic(phaseProgress);
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Door closing with slide
                const doorOffset = eased * 50;
                ctx.save();
                ctx.globalAlpha = 1 - eased;
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(canvas.width / 2 - 25 + doorOffset, canvas.height / 2 - 50, 50, 100);
                ctx.restore();
                // Devil appearing behind with lunge
                const devilX = canvas.width / 2 + 50 + Math.sin((cutsceneFrame - 300) * 0.2) * 20 * eased;
                const devilY = canvas.height / 2 + Math.cos((cutsceneFrame - 300) * 0.15) * 10;
                if (monsterSprite.complete) {
                    ctx.save();
                    ctx.translate(devilX, devilY);
                    ctx.scale(1 + eased * 0.3, 1 + eased * 0.3); // Grow slightly
                    ctx.rotate(Math.sin((cutsceneFrame - 300) * 0.3) * 0.3);
                    ctx.drawImage(monsterSprite, -16, -16, 32, 32);
                    ctx.restore();
                } else {
                    ctx.fillStyle = '#f00';
                    ctx.fillRect(devilX - 8, devilY - 8, 16, 16);
                }
                // Red eyes with flare
                ctx.save();
                ctx.shadowColor = '#f00';
                ctx.shadowBlur = 10 * eased;
                ctx.fillStyle = '#f00';
                ctx.beginPath();
                ctx.arc(devilX - 5, devilY + 5, 3 * (1 + eased), 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(devilX + 5, devilY + 5, 3 * (1 + eased), 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('The Devil lunges, but too late!', canvas.width / 2, canvas.height / 2 - 100);
            } 
            // Phase 3: Escaping into light (600-900 frames)
            else {
                const phaseProgress = (cutsceneFrame - 600) / 300;
                const eased = easeInOutCubic(phaseProgress);
                // Bright light overlay with bloom
                const lightAlpha = eased * 0.8;
                const lightGradient = ctx.createRadialGradient(canvas.width / 2, canvas.height / 2, 0, canvas.width / 2, canvas.height / 2, canvas.width);
                lightGradient.addColorStop(0, `rgba(255, 255, 255, ${lightAlpha})`);
                lightGradient.addColorStop(1, 'rgba(255, 255, 255, 0)');
                ctx.fillStyle = lightGradient;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Player running forward with trail
                const playerRunX = (cutsceneFrame - 600) * 3 * eased;
                const playerRunY = canvas.height / 2 + Math.sin((cutsceneFrame - 600) * 0.3) * 8 * eased;
                if (playerSprite.complete) {
                    ctx.globalAlpha = 1 - lightAlpha * 0.6;
                    ctx.save();
                    ctx.translate(playerRunX, playerRunY);
                    ctx.rotate(Math.sin((cutsceneFrame - 600) * 0.4) * 0.2);
                    ctx.drawImage(playerSprite, -16, -16, 32, 32);
                    ctx.restore();
                    // Trail
                    ctx.globalAlpha = 0.5 * (1 - eased);
                    ctx.fillStyle = '#00f';
                    ctx.fillRect(playerRunX - 32, playerRunY - 16, 32, 32);
                } else {
                    ctx.fillStyle = '#00f';
                    ctx.globalAlpha = 1 - lightAlpha * 0.6;
                    ctx.fillRect(playerRunX - 8, playerRunY - 8, 16, 16);
                }
                ctx.globalAlpha = 1;
                // Forest fading in with parallax trees
                const forestAlpha = eased;
                ctx.fillStyle = `rgba(0, 100, 0, ${forestAlpha * 0.5})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                // Trees appearing with depth
                for (let i = 0; i < 15; i++) {
                    const depth = i / 15;
                    const tx = playerRunX + i * 80 + (cutsceneFrame - 600) * (1 + depth) * 1.5;
                    const ty = canvas.height - 120 + Math.sin((cutsceneFrame * 0.04) + i * 0.5) * 15 * forestAlpha;
                    ctx.save();
                    ctx.globalAlpha = forestAlpha * (1 - depth * 0.5);
                    ctx.fillStyle = `rgba(139, 69, 19, ${forestAlpha})`;
                    ctx.fillRect(tx, ty, 10 * (1 + depth), 40 * (1 + depth));
                    ctx.fillStyle = `rgba(34, 139, 34, ${forestAlpha})`;
                    ctx.beginPath();
                    ctx.arc(tx + 5 * (1 + depth), ty - 15 * (1 + depth), 25 * (1 + depth), 0, Math.PI * 2);
                    ctx.fill();
                    ctx.restore();
                }
                ctx.fillStyle = '#fff';
                ctx.font = '24px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('You escape the Devil\'s domain!', canvas.width / 2, 100);
                ctx.fillText('Freedom awaits...', canvas.width / 2, 130);
            }
            // Prompt text
            ctx.fillStyle = '#ffd700';
            ctx.font = '18px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('Press SPACE to play again', canvas.width / 2, canvas.height - 50);
            ctx.textAlign = 'left';
        }

        function drawGameOverCutscene() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#111';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            const progress = cutsceneFrame / GAMEOVER_CUTSCENE_FRAMES;
            const eased = easeInOutCubic(progress);
            const shakeIntensity = Math.sin(cutsceneFrame * 0.4) * 10 * eased;

            // Player in center with shake and fade
            const playerX = canvas.width / 2 + shakeIntensity;
            const playerY = canvas.height / 2 + shakeIntensity;
            ctx.save();
            ctx.translate(shakeIntensity, shakeIntensity);
            if (playerSprite.complete) {
                ctx.globalAlpha = 1 - eased * 0.5;
                ctx.drawImage(playerSprite, playerX - 16, playerY - 16, 32, 32);
            } else {
                ctx.fillStyle = '#00f';
                ctx.globalAlpha = 1 - eased * 0.5;
                ctx.fillRect(playerX - 8, playerY - 8, 16, 16);
            }
            ctx.restore();

            // Devil closing in with scale and rotation
            const devilScale = 1 + eased * 1.5;
            const devilX = canvas.width / 2 + Math.sin(cutsceneFrame * 0.2) * 5;
            const devilY = canvas.height / 2 + Math.cos(cutsceneFrame * 0.25) * 3;
            ctx.save();
            ctx.translate(devilX, devilY);
            ctx.scale(devilScale, devilScale);
            ctx.rotate(Math.sin(cutsceneFrame * 0.1) * 0.5 * eased);
            if (monsterSprite.complete) {
                ctx.drawImage(monsterSprite, -16, -16, 32, 32);
            } else {
                ctx.fillStyle = '#f00';
                ctx.fillRect(-8, -8, 16, 16);
            }
            ctx.restore();

            // Red flash with increasing frequency
            if (eased > 0.3) {
                const flashAlpha = Math.sin(cutsceneFrame * 0.15) * 0.4 * eased;
                ctx.fillStyle = `rgba(255, 0, 0, ${flashAlpha})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }

            // Darken screen gradually
            ctx.fillStyle = `rgba(0, 0, 0, ${eased * 0.7})`;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = '#f00';
            ctx.font = '24px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('The Devil catches you!', canvas.width / 2, 100);
            ctx.fillText('Your soul is claimed...', canvas.width / 2, 130);

            // Prompt text
            ctx.fillStyle = '#ffd700';
            ctx.font = '18px monospace';
            ctx.textAlign = 'center';
            ctx.fillText('Press SPACE to try again', canvas.width / 2, canvas.height - 50);
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
                    createParticles(item.x, item.y);
                    document.getElementById('tasks').textContent = collected;
                    collectSound.play();
                }
            }

            // Trigger collect cutscene if third item collected
            if (justCollected && collected >= 3 && gameState === 'game') {
                warningTimer = 180;
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

            // Update particles
            updateParticles();

            // Update each monster
            const detectionDist = collected >= 3 ? 100 : 10;
            for (let monster of monsters) {
                if (monster.freezeTimer > 0) {
                    monster.freezeTimer--;
                    continue; // Skip movement if frozen
                }

                const distToPlayer = Math.abs(monster.x - playerTileX) + Math.abs(monster.y - playerTileY);
                let isChasing = distToPlayer < detectionDist;

                let target = isChasing ? {x: playerTileX, y: playerTileY} : monster.patrolPoints[monster.patrolIndex];

                if (isChasing) {
                    monster.pathTimer++;
                    if (monster.pathTimer > (collected >= 3 ? 3 : 8)) {
                        monster.pathTimer = 0;
                        monster.path = astar({x: monster.x, y: monster.y}, target);
                    }
                } else {
                    if (!monster.path || monster.path.length <= 1) {
                        monster.path = astar({x: monster.x, y: monster.y}, target);
                        if (monster.path && monster.path.length <= 1) {
                            monster.patrolIndex = (monster.patrolIndex + 1) % monster.patrolPoints.length;
                            target = monster.patrolPoints[monster.patrolIndex];
                            monster.path = astar({x: monster.x, y: monster.y}, target);
                        }
                    }
                }

                monster.updateTimer++;
                if (monster.updateTimer > 15) { // Lower speed: move less frequently
                    monster.updateTimer = 0;
                    let nextPos = null;
                    if (monster.path && monster.path.length > 1) {
                        nextPos = monster.path[1];
                        monster.path.shift();
                    } else {
                        // Random move if no path
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
                        health -= 20;
                        document.getElementById('health').textContent = health;
                        monster.freezeTimer = 120; // Freeze for 2 seconds (assuming 60 FPS)
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
                    } else {
                        // Floor for 0 or anything else
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

            // Draw particles
            for (let p of particles) {
                ctx.save();
                ctx.globalAlpha = p.life / 30;
                ctx.fillStyle = p.color;
                ctx.fillRect(p.x - camera.x - 2, p.y - camera.y - 2, 4, 4);
                ctx.restore();
            }

            // Draw fog and light effects
            drawFogAndLight();

            // Draw warning
            drawWarning();

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
