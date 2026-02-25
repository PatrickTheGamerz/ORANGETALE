<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Defender Core: Advanced Edition</title>
    <style>
        :root {
            --bg: #020617;
            --panel: #1e293b;
            --accent: #38bdf8;
            --text: #f8fafc;
            --danger: #ef4444;
        }

        body {
            background-color: var(--bg);
            color: var(--text);
            font-family: 'Segoe UI', system-ui, sans-serif;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow: hidden;
            user-select: none;
        }

        #ui-layer {
            position: absolute;
            inset: 0;
            pointer-events: none;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }

        .menu-panel {
            background: var(--panel);
            padding: 2.5rem;
            border-radius: 16px;
            box-shadow: 0 0 40px rgba(0,0,0,0.8);
            text-align: center;
            pointer-events: auto;
            border: 2px solid var(--accent);
            max-width: 700px;
        }

        .tower-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
            margin: 25px 0;
        }

        .tower-card {
            background: #334155;
            padding: 12px;
            border-radius: 10px;
            cursor: pointer;
            border: 2px solid transparent;
            transition: all 0.2s;
        }

        .tower-card.selected {
            border-color: var(--accent);
            background: #0ea5e9;
            transform: scale(1.05);
        }

        canvas {
            background: #020617;
            cursor: crosshair;
            display: block;
        }

        #hud {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(15, 23, 42, 0.9);
            padding: 12px 30px;
            border-radius: 100px;
            display: flex;
            gap: 30px;
            pointer-events: auto;
            border: 1px solid var(--accent);
            backdrop-filter: blur(5px);
            align-items: center;
        }

        .stat { font-weight: 800; color: var(--accent); font-size: 1.2rem; }

        #tower-shop {
            position: absolute;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 15px;
            pointer-events: auto;
            background: rgba(15, 23, 42, 0.8);
            padding: 15px;
            border-radius: 20px;
        }

        .shop-btn {
            width: 90px;
            height: 90px;
            background: var(--panel);
            border: 2px solid #475569;
            border-radius: 12px;
            color: white;
            cursor: pointer;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            transition: 0.2s;
        }

        .shop-btn.active { 
            border-color: var(--accent); 
            background: #0f172a; 
            box-shadow: 0 0 15px var(--accent);
        }

        #upgrade-menu {
            position: absolute;
            right: 20px;
            top: 50%;
            transform: translateY(-50%);
            background: var(--panel);
            padding: 20px;
            border-radius: 15px;
            display: none;
            pointer-events: auto;
            width: 180px;
            border-left: 4px solid var(--accent);
        }

        button {
            background: var(--accent);
            border: none;
            padding: 12px 24px;
            color: #000;
            font-weight: 900;
            border-radius: 8px;
            cursor: pointer;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        button:disabled { background: #475569; cursor: not-allowed; }

        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="ui-layer">
        <div id="selection-menu" class="menu-panel">
            <h1 style="margin-top:0; color:var(--accent)">DEFENDER CORE</h1>
            <p>Select 4 strategic assets for the upcoming wave.</p>
            <div id="tower-options" class="tower-grid"></div>
            <p id="selection-count" style="font-size: 1.2rem">Selected: 0/4</p>
            <button id="start-btn" disabled>ENGAGE SYSTEM</button>
        </div>

        <div id="game-over" class="menu-panel hidden">
            <h1 id="result-text" style="color:var(--danger)">CORE BREACHED</h1>
            <button onclick="location.reload()">REBOOT SYSTEM</button>
        </div>
    </div>

    <div id="hud" class="hidden">
        <div>CORE: <span id="hp-val" class="stat">20</span></div>
        <div>CREDITS: <span id="money-val" class="stat">500</span></div>
        <div>WAVE: <span id="wave-val" class="stat">0</span></div>
        <button id="next-wave" style="padding: 5px 15px;">START WAVE</button>
    </div>

    <div id="tower-shop" class="hidden"></div>

    <div id="upgrade-menu">
        <h3 id="up-name" style="margin:0 0 10px 0">Tower</h3>
        <p id="up-stats" style="font-size:0.9rem; line-height:1.4"></p>
        <button id="up-btn" style="width:100%; margin-bottom:10px;">Upgrade</button>
        <button id="sell-btn" style="width:100%; background:var(--danger); color:white;">Decommission</button>
    </div>

    <canvas id="gameCanvas"></canvas>

<script>
/**
 * DATA DEFINITIONS
 */
const TILE_SIZE = 60;
const COLS = 16;
const ROWS = 10;
const WIDTH = COLS * TILE_SIZE;
const HEIGHT = ROWS * TILE_SIZE;

const TOWER_TYPES = [
    { id: 'gun', name: 'Sentry', cost: 100, range: 150, dmg: 10, rate: 25, color: '#94a3b8', desc: 'Basic rapid fire.' },
    { id: 'sniper', name: 'Railgun', cost: 250, range: 350, dmg: 80, rate: 100, color: '#fbbf24', desc: 'Extreme range/damage.' },
    { id: 'radar', name: 'Oracle', cost: 150, range: 200, dmg: 0, rate: 0, color: '#c084fc', detect: true, desc: 'Reveals cloaked units.' },
    { id: 'slow', name: 'Cryo', cost: 200, range: 130, dmg: 2, rate: 15, color: '#38bdf8', desc: 'Slows enemies by 50%.' },
    { id: 'tesla', name: 'Tesla', cost: 350, range: 120, dmg: 15, rate: 6, color: '#4ade80', detect: true, desc: 'Fast chain lightning.' },
    { id: 'drone', name: 'Hive', cost: 400, range: 400, dmg: 40, rate: 150, color: '#f472b6', desc: 'Global seeking drones.' },
    { id: 'plasma', name: 'Plasma', cost: 300, range: 180, dmg: 1, rate: 1, color: '#facc15', desc: 'Ramps dmg over time.' },
    { id: 'mine', name: 'Mine Layer', cost: 225, range: 100, dmg: 100, rate: 200, color: '#ef4444', desc: 'Drops mines on path.' }
];

const ENEMY_TYPES = [
    { name: 'Scout', hp: 40, speed: 1.8, color: '#fff', money: 20, stealth: false },
    { name: 'Tank', hp: 250, speed: 0.8, color: '#64748b', money: 45, stealth: false },
    { name: 'Phantom', hp: 60, speed: 1.4, color: '#818cf8', money: 40, stealth: true },
    { name: 'Swarm', hp: 20, speed: 3.0, color: '#fbbf24', money: 15, stealth: false }
];

/**
 * CORE STATE
 */
let canvas = document.getElementById('gameCanvas');
let ctx = canvas.getContext('2d');
canvas.width = WIDTH;
canvas.height = HEIGHT;

let state = 'SELECTING';
let selectedTowerIds = [];
let map = [], path = [], enemies = [], towers = [], mines = [];
let wave = 0, money = 600, lives = 20;
let activeTowerToPlace = null;
let selectedTowerInstance = null;
let waveInProgress = false;
let mouse = { x: 0, y: 0, tx: 0, ty: 0 };

/**
 * INITIALIZATION & MENU
 */
const towerOptions = document.getElementById('tower-options');
TOWER_TYPES.forEach(t => {
    const card = document.createElement('div');
    card.className = 'tower-card';
    card.innerHTML = `<strong>${t.name}</strong><br><small>${t.desc}</small><br><b>$${t.cost}</b>`;
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
    towerOptions.appendChild(card);
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
        btn.innerHTML = `<b>${type.name}</b><span>$${type.cost}</span>`;
        btn.onclick = (e) => {
            e.stopPropagation();
            activeTowerToPlace = type;
            selectedTowerInstance = null;
            document.querySelectorAll('.shop-btn').forEach(b => b.classList.remove('active'));
            btn.classList.add('active');
        };
        shop.appendChild(btn);
    });
}

function generateMap() {
    map = Array(ROWS).fill().map(() => Array(COLS).fill(1));
    let x = 0;
    let y = Math.floor(Math.random() * (ROWS - 4)) + 2;
    path = [];
    while (x < COLS) {
        path.push({x, y});
        map[y][x] = 0;
        let r = Math.random();
        if (r < 0.3 && y > 1 && map[y-1][x] !== 0) y--;
        else if (r < 0.6 && y < ROWS - 2 && map[y+1][x] !== 0) y++;
        else x++;
    }
}

/**
 * GAME LOGIC
 */
class Tower {
    constructor(type, tx, ty) {
        this.type = type;
        this.tx = tx; this.ty = ty;
        this.x = tx * TILE_SIZE + TILE_SIZE/2;
        this.y = ty * TILE_SIZE + TILE_SIZE/2;
        this.level = 1;
        this.range = type.range;
        this.dmg = type.dmg;
        this.rate = type.rate;
        this.cooldown = 0;
        this.plasmaMult = 1;
        this.target = null;
    }

    update() {
        if (this.cooldown > 0) this.cooldown--;
        
        // Stealth revealing
        if (this.type.detect) {
            enemies.forEach(e => {
                if (Math.hypot(e.x - this.x, e.y - this.y) < this.range) e.revealed = true;
            });
        }

        // Logic for Mine Layer
        if (this.type.id === 'mine' && this.cooldown <= 0) {
            mines.push({ x: this.x, y: this.y, dmg: this.dmg });
            this.cooldown = this.rate;
        }

        // Targeting
        this.target = enemies.find(e => Math.hypot(e.x - this.x, e.y - this.y) < this.range && e.revealed);
        
        if (this.target && this.cooldown <= 0) {
            this.shoot();
        } else if (!this.target) {
            this.plasmaMult = 1;
        }
    }

    shoot() {
        const t = this.target;
        if (this.type.id === 'plasma') {
            t.hp -= this.dmg * this.plasmaMult;
            this.plasmaMult += 0.05;
            this.drawBeam(this.type.color, 4);
        } else if (this.type.id === 'slow') {
            t.hp -= this.dmg;
            t.slowTimer = 30;
            this.drawBeam(this.type.color, 2);
        } else {
            t.hp -= this.dmg;
            this.cooldown = this.rate;
            this.drawBeam(this.type.color, 3);
        }
    }

    drawBeam(color, width) {
        ctx.strokeStyle = color;
        ctx.lineWidth = width;
        ctx.beginPath();
        ctx.moveTo(this.x, this.y);
        ctx.lineTo(this.target.x, this.target.y);
        ctx.stroke();
    }

    draw() {
        ctx.fillStyle = this.type.color;
        ctx.beginPath();
        ctx.roundRect(this.tx*TILE_SIZE+5, this.ty*TILE_SIZE+5, TILE_SIZE-10, TILE_SIZE-10, 8);
        ctx.fill();
        ctx.fillStyle = "#000";
        ctx.font = "bold 12px Arial";
        ctx.fillText("L" + this.level, this.x - 7, this.y + 5);
    }
}

class Enemy {
    constructor(type, waveIdx) {
        this.type = type;
        this.pathIndex = 0;
        this.x = path[0].x * TILE_SIZE + TILE_SIZE/2;
        this.y = path[0].y * TILE_SIZE + TILE_SIZE/2;
        this.hp = type.hp * (1 + waveIdx * 0.3);
        this.maxHp = this.hp;
        this.speed = type.speed;
        this.revealed = !type.stealth;
        this.slowTimer = 0;
    }

    update() {
        if (this.slowTimer > 0) this.slowTimer--;
        let curSpeed = this.slowTimer > 0 ? this.speed * 0.5 : this.speed;
        
        const target = path[this.pathIndex];
        const tx = target.x * TILE_SIZE + TILE_SIZE/2;
        const ty = target.y * TILE_SIZE + TILE_SIZE/2;
        const dist = Math.hypot(tx - this.x, ty - this.y);

        if (dist < 5) {
            this.pathIndex++;
            if (this.pathIndex >= path.length) return true;
        }

        this.x += ((tx - this.x) / dist) * curSpeed;
        this.y += ((ty - this.y) / dist) * curSpeed;
        return false;
    }

    draw() {
        ctx.globalAlpha = (this.type.stealth && !this.revealed) ? 0.15 : 1.0;
        ctx.fillStyle = this.type.color;
        ctx.beginPath();
        ctx.arc(this.x, this.y, 12, 0, Math.PI*2);
        ctx.fill();
        
        ctx.fillStyle = 'red';
        ctx.fillRect(this.x - 12, this.y - 18, 24, 4);
        ctx.fillStyle = 'lime';
        ctx.fillRect(this.x - 12, this.y - 18, 24 * (this.hp/this.maxHp), 4);
        ctx.globalAlpha = 1.0;
    }
}

/**
 * ENGINE
 */
function gameLoop() {
    if (lives <= 0) {
        document.getElementById('game-over').classList.remove('hidden');
        return;
    }

    ctx.clearRect(0, 0, WIDTH, HEIGHT);

    // Grid & Path
    for (let r = 0; r < ROWS; r++) {
        for (let c = 0; c < COLS; c++) {
            ctx.fillStyle = map[r][c] === 0 ? '#0f172a' : '#020617';
            ctx.fillRect(c * TILE_SIZE, r * TILE_SIZE, TILE_SIZE, TILE_SIZE);
        }
    }

    // Mines
    mines.forEach((m, i) => {
        ctx.fillStyle = 'red';
        ctx.beginPath(); ctx.arc(m.x, m.y, 5, 0, Math.PI*2); ctx.fill();
        enemies.forEach(e => {
            if (Math.hypot(e.x - m.x, e.y - m.y) < 20) {
                e.hp -= m.dmg;
                mines.splice(i, 1);
            }
        });
    });

    // Towers
    towers.forEach(t => {
        t.update();
        t.draw();
    });

    // Enemies
    if (waveInProgress) {
        for (let i = enemies.length - 1; i >= 0; i--) {
            const e = enemies[i];
            e.revealed = !e.type.stealth;
            const reached = e.update();
            e.draw();
            if (reached) { lives--; enemies.splice(i, 1); }
            else if (e.hp <= 0) { money += e.type.money; enemies.splice(i, 1); }
        }
        if (enemies.length === 0) {
            waveInProgress = false;
            document.getElementById('next-wave').disabled = false;
        }
    }

    // UI Overlays (Ranges)
    drawOverlays();

    updateHUD();
    requestAnimationFrame(gameLoop);
}

function drawOverlays() {
    // Range for selected placed tower
    if (selectedTowerInstance) {
        ctx.beginPath();
        ctx.arc(selectedTowerInstance.x, selectedTowerInstance.y, selectedTowerInstance.range, 0, Math.PI*2);
        ctx.fillStyle = "rgba(56, 189, 248, 0.1)";
        ctx.fill();
        ctx.strokeStyle = "var(--accent)";
        ctx.stroke();
    }

    // Ghost tower and range
    if (activeTowerToPlace) {
        const canPlace = map[mouse.ty][mouse.tx] === 1 && !towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty);
        ctx.globalAlpha = 0.5;
        ctx.fillStyle = canPlace ? activeTowerToPlace.color : "red";
        ctx.beginPath();
        ctx.roundRect(mouse.tx*TILE_SIZE+5, mouse.ty*TILE_SIZE+5, TILE_SIZE-10, TILE_SIZE-10, 8);
        ctx.fill();
        
        ctx.beginPath();
        ctx.arc(mouse.tx*TILE_SIZE + TILE_SIZE/2, mouse.ty*TILE_SIZE + TILE_SIZE/2, activeTowerToPlace.range, 0, Math.PI*2);
        ctx.strokeStyle = canPlace ? "white" : "red";
        ctx.stroke();
        ctx.globalAlpha = 1.0;
    }
}

/**
 * INTERACTION
 */
canvas.addEventListener('mousemove', (e) => {
    const rect = canvas.getBoundingClientRect();
    mouse.x = e.clientX - rect.left;
    mouse.y = e.clientY - rect.top;
    mouse.tx = Math.floor(mouse.x / TILE_SIZE);
    mouse.ty = Math.floor(mouse.y / TILE_SIZE);
});

canvas.addEventListener('mousedown', () => {
    const existing = towers.find(t => t.tx === mouse.tx && t.ty === mouse.ty);
    
    if (activeTowerToPlace) {
        const occupied = towers.find(t => t.tx === mouse.tx && t.ty === mouse.ty);
        if (!occupied && map[mouse.ty][mouse.tx] === 1 && money >= activeTowerToPlace.cost) {
            money -= activeTowerToPlace.cost;
            towers.push(new Tower(activeTowerToPlace, mouse.tx, mouse.ty));
            activeTowerToPlace = null;
            document.querySelectorAll('.shop-btn').forEach(b => b.classList.remove('active'));
        } else {
            activeTowerToPlace = null;
            document.querySelectorAll('.shop-btn').forEach(b => b.classList.remove('active'));
        }
    } else if (existing) {
        selectedTowerInstance = existing;
        showUpgradeMenu(existing);
    } else {
        selectedTowerInstance = null;
        document.getElementById('upgrade-menu').style.display = 'none';
    }
});

function showUpgradeMenu(tower) {
    const menu = document.getElementById('upgrade-menu');
    menu.style.display = 'block';
    document.getElementById('up-name').innerText = tower.type.name;
    const upCost = Math.floor(tower.type.cost * 0.7 * tower.level);
    document.getElementById('up-stats').innerHTML = `LEVEL: ${tower.level}<br>DMG: ${tower.dmg}<br>RANGE: ${tower.range}`;
    
    const upBtn = document.getElementById('up-btn');
    upBtn.innerText = `UPGRADE ($${upCost})`;
    upBtn.disabled = money < upCost;
    upBtn.onclick = () => {
        if (money >= upCost) {
            money -= upCost;
            tower.level++;
            tower.dmg = Math.ceil(tower.dmg * 1.4);
            tower.range += 15;
            tower.rate = Math.max(1, tower.rate - 1);
            showUpgradeMenu(tower);
        }
    };

    document.getElementById('sell-btn').onclick = () => {
        money += Math.floor(tower.type.cost * 0.5);
        towers = towers.filter(t => t !== tower);
        selectedTowerInstance = null;
        menu.style.display = 'none';
    };
}

document.getElementById('next-wave').onclick = () => {
    if (waveInProgress) return;
    wave++;
    waveInProgress = true;
    document.getElementById('next-wave').disabled = true;
    let count = 8 + wave * 3;
    for (let i = 0; i < count; i++) {
        setTimeout(() => {
            const pool = ENEMY_TYPES.slice(0, Math.min(wave, ENEMY_TYPES.length));
            const type = pool[Math.floor(Math.random() * pool.length)];
            enemies.push(new Enemy(type, wave));
        }, i * 500);
    }
};

function updateHUD() {
    document.getElementById('hp-val').innerText = lives;
    document.getElementById('money-val').innerText = Math.floor(money);
    document.getElementById('wave-val').innerText = wave;
}

</script>
</body>
</html>
