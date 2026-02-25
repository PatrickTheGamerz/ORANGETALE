<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Procedural Tower Defense</title>
    <style>
        :root {
            --bg: #0f172a;
            --panel: #1e293b;
            --accent: #38bdf8;
            --text: #f8fafc;
        }

        body {
            background-color: var(--bg);
            color: var(--text);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow: hidden;
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .menu-panel {
            background: var(--panel);
            padding: 2rem;
            border-radius: 12px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
            text-align: center;
            pointer-events: auto;
            border: 2px solid var(--accent);
            max-width: 600px;
        }

        .tower-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin: 20px 0;
        }

        .tower-card {
            background: #334155;
            padding: 10px;
            border-radius: 8px;
            cursor: pointer;
            border: 2px solid transparent;
            transition: 0.2s;
        }

        .tower-card.selected {
            border-color: var(--accent);
            background: #0ea5e9;
        }

        .tower-card:hover {
            transform: translateY(-2px);
        }

        canvas {
            background: #020617;
            cursor: crosshair;
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
        }

        #hud {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(30, 41, 59, 0.9);
            padding: 10px 20px;
            border-radius: 50px;
            display: flex;
            gap: 20px;
            pointer-events: auto;
            border: 1px solid var(--accent);
        }

        .stat { font-weight: bold; color: var(--accent); }

        #tower-shop {
            position: absolute;
            right: 20px;
            top: 50%;
            transform: translateY(-50%);
            display: flex;
            flex-direction: column;
            gap: 10px;
            pointer-events: auto;
        }

        .shop-btn {
            width: 80px;
            height: 80px;
            background: var(--panel);
            border: 2px solid #475569;
            border-radius: 10px;
            color: white;
            cursor: pointer;
            font-size: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        .shop-btn.active { border-color: var(--accent); background: #0f172a; }

        #upgrade-menu {
            position: absolute;
            left: 20px;
            top: 50%;
            transform: translateY(-50%);
            background: var(--panel);
            padding: 15px;
            border-radius: 10px;
            display: none;
            pointer-events: auto;
            width: 150px;
        }

        button {
            background: var(--accent);
            border: none;
            padding: 10px 20px;
            color: var(--bg);
            font-weight: bold;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 10px;
        }

        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="ui-layer">
        <div id="selection-menu" class="menu-panel">
            <h1>DEFENDER CORE</h1>
            <p>Pick exactly 4 towers to bring into battle</p>
            <div id="tower-options" class="tower-grid"></div>
            <p id="selection-count">Selected: 0/4</p>
            <button id="start-btn" disabled>START MISSION</button>
        </div>

        <div id="game-over" class="menu-panel hidden">
            <h1 id="result-text">BASE DESTROYED</h1>
            <button onclick="location.reload()">RETRY</button>
        </div>
    </div>

    <div id="hud" class="hidden">
        <div>Health: <span id="hp-val" class="stat">20</span></div>
        <div>Credits: <span id="money-val" class="stat">500</span></div>
        <div>Wave: <span id="wave-val" class="stat">1</span></div>
        <button id="next-wave">START WAVE</button>
    </div>

    <div id="tower-shop" class="hidden"></div>

    <div id="upgrade-menu">
        <h4 id="up-name">Tower</h4>
        <p id="up-stats">Lvl: 1</p>
        <button id="up-btn">Upgrade ($100)</button>
        <button id="sell-btn" style="background:#ef4444">Sell</button>
    </div>

    <canvas id="gameCanvas"></canvas>

<script>
/** * GAME CONFIGURATION 
 */
const TILE_SIZE = 50;
const COLS = 16;
const ROWS = 12;
const WIDTH = COLS * TILE_SIZE;
const HEIGHT = ROWS * TILE_SIZE;

const TOWER_TYPES = [
    { id: 'gun', name: 'Sentry', cost: 100, range: 120, dmg: 10, rate: 20, color: '#94a3b8', detect: false, desc: 'Basic rapid fire' },
    { id: 'sniper', name: 'Railgun', cost: 200, range: 250, dmg: 50, rate: 80, color: '#fbbf24', detect: false, desc: 'High dmg, long range' },
    { id: 'radar', name: 'Oracle', cost: 150, range: 150, dmg: 5, rate: 30, color: '#c084fc', detect: true, desc: 'Reveals cloaked enemies' },
    { id: 'slow', name: 'Cryo', cost: 175, range: 100, dmg: 2, rate: 10, color: '#38bdf8', detect: false, desc: 'Slows enemies down' },
    { id: 'splash', name: 'Mortar', cost: 250, range: 180, dmg: 30, rate: 100, color: '#f87171', detect: false, splash: 60, desc: 'Area of effect damage' },
    { id: 'tesla', name: 'Tesla', cost: 300, range: 90, dmg: 15, rate: 5, color: '#4ade80', detect: true, desc: 'Ultra fast, detects cloaked' }
];

const ENEMY_TYPES = [
    { name: 'Scout', hp: 30, speed: 1.5, color: '#fff', money: 15, stealth: false },
    { name: 'Tank', hp: 150, speed: 0.6, color: '#475569', money: 30, stealth: false },
    { name: 'Phantom', hp: 40, speed: 1.2, color: '#6366f1', money: 25, stealth: true },
    { name: 'Speedster', hp: 20, speed: 2.8, color: '#fbbf24', money: 20, stealth: false }
];

/**
 * GAME STATE
 */
let canvas = document.getElementById('gameCanvas');
let ctx = canvas.getContext('2d');
canvas.width = WIDTH;
canvas.height = HEIGHT;

let state = 'SELECTING';
let selectedTowerIds = [];
let map = [];
let path = [];
let enemies = [];
let towers = [];
let projectiles = [];
let wave = 0;
let money = 500;
let lives = 20;
let selectedMapTile = null;
let activeTowerToPlace = null;
let waveInProgress = false;

/**
 * MAP GENERATION (Random Walk / Pathfinding)
 */
function generateMap() {
    // 0 = path, 1 = wall/buildable
    map = Array(ROWS).fill().map(() => Array(COLS).fill(1));
    
    let x = 0;
    let y = Math.floor(Math.random() * (ROWS - 2)) + 1;
    path = [];

    // Simple random walk to right side
    while (x < COLS) {
        path.push({x, y});
        map[y][x] = 0;
        
        let rand = Math.random();
        if (rand < 0.4 && y > 1 && map[y-1][x] !== 0) y--;
        else if (rand < 0.8 && y < ROWS - 2 && map[y+1][x] !== 0) y++;
        else x++;
    }
}

/**
 * TOWER SELECTION LOGIC
 */
const selectionMenu = document.getElementById('tower-options');
TOWER_TYPES.forEach(t => {
    const card = document.createElement('div');
    card.className = 'tower-card';
    card.innerHTML = `<strong>${t.name}</strong><br><small>${t.desc}</small><br>Cost: $${t.cost}`;
    card.onclick = () => {
        if (selectedTowerIds.includes(t.id)) {
            selectedTowerIds = selectedTowerIds.filter(id => id !== t.id);
            card.classList.remove('selected');
        } else if (selectedTowerIds.length < 4) {
            selectedTowerIds.push(t.id);
            card.classList.add('selected');
        }
        document.getElementById('selection-count').innerText = `Selected: ${selectedTowerIds.length}/4`;
        document.getElementById('start-btn').disabled = selectedTowerIds.length !== 4;
    };
    selectionMenu.appendChild(card);
});

document.getElementById('start-btn').onclick = () => {
    document.getElementById('selection-menu').classList.add('hidden');
    document.getElementById('hud').classList.remove('hidden');
    document.getElementById('tower-shop').classList.remove('hidden');
    initShop();
    generateMap();
    state = 'PLAYING';
    requestAnimationFrame(gameLoop);
};

function initShop() {
    const shop = document.getElementById('tower-shop');
    selectedTowerIds.forEach(id => {
        const type = TOWER_TYPES.find(t => t.id === id);
        const btn = document.createElement('button');
        btn.className = 'shop-btn';
        btn.innerHTML = `<b>${type.name}</b><br>$${type.cost}`;
        btn.onclick = () => {
            activeTowerToPlace = type;
            document.querySelectorAll('.shop-btn').forEach(b => b.classList.remove('active'));
            btn.classList.add('active');
        };
        shop.appendChild(btn);
    });
}

/**
 * GAME OBJECTS
 */
class Enemy {
    constructor(type, waveMult) {
        this.type = type;
        this.pathIndex = 0;
        this.x = path[0].x * TILE_SIZE + TILE_SIZE/2;
        this.y = path[0].y * TILE_SIZE + TILE_SIZE/2;
        this.hp = type.hp * (1 + waveMult * 0.2);
        this.maxHp = this.hp;
        this.speed = type.speed;
        this.money = type.money;
        this.stealth = type.stealth;
        this.revealed = !type.stealth;
        this.reachedEnd = false;
        this.slowTimer = 0;
    }

    update() {
        if (this.slowTimer > 0) {
            this.slowTimer--;
            this.x += (path[this.pathIndex].x * TILE_SIZE + TILE_SIZE/2 - this.x) * (this.speed * 0.5) * 0.1;
            this.y += (path[this.pathIndex].y * TILE_SIZE + TILE_SIZE/2 - this.y) * (this.speed * 0.5) * 0.1;
        } else {
            const targetX = path[this.pathIndex].x * TILE_SIZE + TILE_SIZE/2;
            const targetY = path[this.pathIndex].y * TILE_SIZE + TILE_SIZE/2;
            const dx = targetX - this.x;
            const dy = targetY - this.y;
            const dist = Math.sqrt(dx*dx + dy*dy);

            if (dist < 2) {
                this.pathIndex++;
                if (this.pathIndex >= path.length) {
                    this.reachedEnd = true;
                    return;
                }
            }
            this.x += (dx/dist) * this.speed;
            this.y += (dy/dist) * this.speed;
        }
    }

    draw() {
        if (this.stealth && !this.revealed) {
            ctx.globalAlpha = 0.1;
        }
        ctx.fillStyle = this.type.color;
        ctx.beginPath();
        ctx.arc(this.x, this.y, 10, 0, Math.PI*2);
        ctx.fill();
        
        // HP Bar
        ctx.fillStyle = 'red';
        ctx.fillRect(this.x - 10, this.y - 15, 20, 3);
        ctx.fillStyle = 'lime';
        ctx.fillRect(this.x - 10, this.y - 15, 20 * (this.hp/this.maxHp), 3);
        ctx.globalAlpha = 1.0;
    }
}

class Tower {
    constructor(type, tx, ty) {
        this.type = type;
        this.tx = tx;
        this.ty = ty;
        this.x = tx * TILE_SIZE + TILE_SIZE/2;
        this.y = ty * TILE_SIZE + TILE_SIZE/2;
        this.level = 1;
        this.range = type.range;
        this.dmg = type.dmg;
        this.rate = type.rate;
        this.cooldown = 0;
    }

    update() {
        if (this.cooldown > 0) this.cooldown--;

        // Stealth Detection logic
        if (this.type.detect) {
            enemies.forEach(e => {
                let d = Math.sqrt((e.x - this.x)**2 + (e.y - this.y)**2);
                if (d < this.range) e.revealed = true;
            });
        }

        if (this.cooldown <= 0) {
            const target = enemies.find(e => {
                let d = Math.sqrt((e.x - this.x)**2 + (e.y - this.y)**2);
                return d < this.range && e.revealed;
            });

            if (target) {
                this.shoot(target);
                this.cooldown = this.rate;
            }
        }
    }

    shoot(target) {
        if (this.type.id === 'slow') {
            target.hp -= this.dmg;
            target.slowTimer = 60;
        } else if (this.type.splash) {
            enemies.forEach(e => {
                let d = Math.sqrt((e.x - target.x)**2 + (e.y - target.y)**2);
                if (d < this.type.splash) e.hp -= this.dmg;
            });
        } else {
            target.hp -= this.dmg;
        }
        
        // Visual Bolt
        ctx.strokeStyle = this.type.color;
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(this.x, this.y);
        ctx.lineTo(target.x, target.y);
        ctx.stroke();
    }

    draw() {
        ctx.fillStyle = this.type.color;
        ctx.fillRect(this.tx * TILE_SIZE + 5, this.ty * TILE_SIZE + 5, TILE_SIZE - 10, TILE_SIZE - 10);
        ctx.fillStyle = 'white';
        ctx.fillText("L" + this.level, this.x - 5, this.y + 5);
    }
}

/**
 * GAME LOOP & INTERACTION
 */
function gameLoop() {
    if (lives <= 0) {
        document.getElementById('game-over').classList.remove('hidden');
        return;
    }

    ctx.clearRect(0, 0, WIDTH, HEIGHT);

    // Draw Map
    for (let r = 0; r < ROWS; r++) {
        for (let c = 0; c < COLS; c++) {
            ctx.fillStyle = map[r][c] === 0 ? '#1e293b' : '#020617';
            ctx.fillRect(c * TILE_SIZE, r * TILE_SIZE, TILE_SIZE, TILE_SIZE);
            ctx.strokeStyle = '#0f172a';
            ctx.strokeRect(c * TILE_SIZE, r * TILE_SIZE, TILE_SIZE, TILE_SIZE);
        }
    }

    // Update & Draw Towers
    towers.forEach(t => {
        t.update();
        t.draw();
    });

    // Handle Wave Logic
    if (waveInProgress) {
        enemies.forEach((e, i) => {
            e.revealed = !e.stealth; // Reset revelation state each frame
            e.update();
            e.draw();
            if (e.reachedEnd) {
                lives--;
                enemies.splice(i, 1);
            } else if (e.hp <= 0) {
                money += e.money;
                enemies.splice(i, 1);
            }
        });

        if (enemies.length === 0) waveInProgress = false;
    }

    updateHUD();
    requestAnimationFrame(gameLoop);
}

function updateHUD() {
    document.getElementById('hp-val').innerText = lives;
    document.getElementById('money-val').innerText = money;
    document.getElementById('wave-val').innerText = wave;
}

// Interaction
canvas.addEventListener('mousedown', (e) => {
    const rect = canvas.getBoundingClientRect();
    const mx = e.clientX - rect.left;
    const my = e.clientY - rect.top;
    const tx = Math.floor(mx / TILE_SIZE);
    const ty = Math.floor(my / TILE_SIZE);

    const existingTower = towers.find(t => t.tx === tx && t.ty === ty);

    if (existingTower) {
        showUpgradeMenu(existingTower);
    } else if (activeTowerToPlace && map[ty][tx] === 1 && money >= activeTowerToPlace.cost) {
        money -= activeTowerToPlace.cost;
        towers.push(new Tower(activeTowerToPlace, tx, ty));
        document.getElementById('upgrade-menu').style.display = 'none';
    } else {
        document.getElementById('upgrade-menu').style.display = 'none';
    }
});

function showUpgradeMenu(tower) {
    const menu = document.getElementById('upgrade-menu');
    menu.style.display = 'block';
    document.getElementById('up-name').innerText = tower.type.name;
    const upCost = Math.floor(tower.type.cost * 0.8 * tower.level);
    document.getElementById('up-stats').innerText = `Lvl: ${tower.level}\nDmg: ${tower.dmg}\nRange: ${tower.range}`;
    document.getElementById('up-btn').innerText = `Upgrade ($${upCost})`;
    
    document.getElementById('up-btn').onclick = () => {
        if (money >= upCost) {
            money -= upCost;
            tower.level++;
            tower.dmg = Math.floor(tower.dmg * 1.5);
            tower.range += 10;
            tower.rate = Math.max(2, tower.rate - 2);
            showUpgradeMenu(tower);
        }
    };

    document.getElementById('sell-btn').onclick = () => {
        money += Math.floor(tower.type.cost * 0.5);
        towers = towers.filter(t => t !== tower);
        menu.style.display = 'none';
    };
}

document.getElementById('next-wave').onclick = () => {
    if (waveInProgress) return;
    wave++;
    waveInProgress = true;
    const count = 5 + wave * 2;
    for (let i = 0; i < count; i++) {
        setTimeout(() => {
            const type = ENEMY_TYPES[Math.floor(Math.random() * Math.min(wave, ENEMY_TYPES.length))];
            enemies.push(new Enemy(type, wave));
        }, i * 600);
    }
};

</script>
</body>
</html>
