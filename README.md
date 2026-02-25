<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DEFENDER CORE: ABSOLUTE</title>
    <style>
        :root {
            --bg: #020617;
            --panel: #0f172a;
            --accent: #22d3ee;
            --gold: #f59e0b;
            --danger: #ef4444;
            --glass: rgba(15, 23, 42, 0.8);
        }

        body {
            background: var(--bg);
            color: #f8fafc;
            font-family: 'Segoe UI', system-ui, sans-serif;
            margin: 0;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            user-select: none;
        }

        /* UI LAYERS */
        #screen-menu, #screen-select, #screen-game {
            position: absolute;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            transition: opacity 0.5s ease;
        }

        .hidden { opacity: 0; pointer-events: none; display: none !important; }

        /* MENU & SELECTION */
        .panel {
            background: var(--panel);
            padding: 40px;
            border-radius: 20px;
            border: 2px solid var(--accent);
            box-shadow: 0 0 50px rgba(34, 211, 238, 0.2);
            text-align: center;
            max-width: 800px;
        }

        h1 { font-size: 3rem; margin: 0; color: var(--accent); letter-spacing: 4px; text-shadow: 0 0 15px var(--accent); }

        .tower-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            margin: 30px 0;
        }

        .tower-card {
            background: #1e293b;
            padding: 15px;
            border-radius: 12px;
            border: 1px solid #334155;
            cursor: pointer;
            transition: 0.2s;
        }

        .tower-card:hover { border-color: var(--accent); transform: translateY(-3px); }
        .tower-card.selected { background: #0891b2; border-color: white; box-shadow: 0 0 15px var(--accent); }

        /* GAME HUD */
        #hud {
            position: absolute;
            top: 20px;
            width: 90%;
            display: flex;
            justify-content: space-between;
            pointer-events: none;
        }

        .hud-item {
            background: var(--glass);
            padding: 10px 25px;
            border-radius: 50px;
            border: 1px solid var(--accent);
            font-weight: bold;
            backdrop-filter: blur(10px);
            pointer-events: auto;
        }

        #shop {
            position: absolute;
            bottom: 20px;
            display: flex;
            gap: 15px;
            background: var(--glass);
            padding: 15px;
            border-radius: 20px;
            border: 1px solid #334155;
        }

        .shop-btn {
            width: 90px;
            height: 100px;
            background: #1e293b;
            border: 1px solid #475569;
            border-radius: 10px;
            color: white;
            cursor: pointer;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            font-size: 0.8rem;
        }

        .shop-btn.active { border-color: var(--accent); background: #0c4a6e; }

        #upgrade-menu {
            position: absolute;
            right: 20px;
            top: 50%;
            transform: translateY(-50%);
            background: var(--panel);
            border: 1px solid var(--accent);
            padding: 20px;
            border-radius: 15px;
            width: 200px;
            display: none;
        }

        button.primary {
            background: var(--accent);
            color: black;
            border: none;
            padding: 12px 30px;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            text-transform: uppercase;
        }

        button.primary:disabled { background: #475569; cursor: not-allowed; }

        canvas { background: #020617; border-radius: 4px; }
    </style>
</head>
<body>

    <div id="screen-menu">
        <div class="panel">
            <h1>DEFENDER CORE</h1>
            <p style="color: #94a3b8">The final frontier of procedural defense.</p>
            <button class="primary" onclick="toSelect()">Begin Mission</button>
        </div>
    </div>

    <div id="screen-select" class="hidden">
        <div class="panel">
            <h2>TACTICAL LOADOUT</h2>
            <p>Select 4 towers to bring into the field.</p>
            <div id="select-grid" class="tower-grid"></div>
            <div id="select-count">0/4 SELECTED</div>
            <button id="btn-deploy" class="primary" style="margin-top:20px" disabled onclick="startGame()">Initialize Grid</button>
        </div>
    </div>

    <div id="screen-game" class="hidden">
        <div id="hud">
            <div class="hud-item">CORE: <span id="hp-val" style="color:var(--danger)">20</span></div>
            <div class="hud-item">CREDITS: <span id="money-val" style="color:var(--gold)">$600</span></div>
            <div class="hud-item">WAVE: <span id="wave-val">0</span></div>
            <button class="primary" id="btn-next-wave" style="padding:8px 20px; pointer-events: auto;">Start Wave</button>
        </div>

        <div id="upgrade-menu">
            <h3 id="up-name" style="margin:0">Tower</h3>
            <p id="up-stats" style="font-size:0.8rem; color:#94a3b8"></p>
            <button class="primary" id="btn-up" style="width:100%; margin-bottom:10px;">Upgrade</button>
            <button id="btn-sell" style="width:100%; background:var(--danger); border:none; color:white; padding:8px; border-radius:5px; cursor:pointer;">Scrap</button>
        </div>

        <div id="shop"></div>
        <canvas id="gameCanvas"></canvas>
    </div>

<script>
/** * DATA CONFIG 
 */
const TOWER_DB = [
    { id: 'sentry', name: 'Sentry', cost: 100, range: 180, dmg: 12, rate: 30, color: '#94a3b8', type: 'bullet' },
    { id: 'rail', name: 'Railgun', cost: 250, range: 400, dmg: 120, rate: 140, color: '#fbbf24', type: 'bullet' },
    { id: 'cryo', name: 'Cryo', cost: 175, range: 150, dmg: 2, rate: 10, color: '#38bdf8', type: 'beam' },
    { id: 'radar', name: 'Oracle', cost: 150, range: 250, dmg: 0, rate: 0, color: '#c084fc', type: 'aura' },
    { id: 'tesla', name: 'Tesla', cost: 350, range: 140, dmg: 25, rate: 8, color: '#4ade80', type: 'beam' },
    { id: 'mortar', name: 'Mortar', cost: 300, range: 300, dmg: 80, rate: 150, color: '#ea580c', type: 'bullet', splash: 80 },
    { id: 'plasma', name: 'Plasma', cost: 400, range: 220, dmg: 1.5, rate: 1, color: '#facc15', type: 'beam' },
    { id: 'hive', name: 'Hive', cost: 450, range: 600, dmg: 40, rate: 100, color: '#f472b6', type: 'bullet' },
    { id: 'mine', name: 'Mine', cost: 200, range: 80, dmg: 200, rate: 300, color: '#ef4444', type: 'mine' }
];

const ENEMY_DB = [
    { id: 'scout', hp: 50, speed: 2.2, color: '#f8fafc', money: 20 },
    { id: 'tank', hp: 350, speed: 0.9, color: '#64748b', money: 55 },
    { id: 'phantom', hp: 80, speed: 1.6, color: '#a855f7', money: 45, stealth: true },
    { id: 'boss', hp: 3500, speed: 0.5, color: '#ef4444', money: 500, boss: true }
];

/** ENGINE CORE **/
const TILE = 64;
const COLS = 16, ROWS = 10;
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
canvas.width = COLS * TILE;
canvas.height = ROWS * TILE;

let state = 'MENU';
let money = 600, lives = 20, wave = 0;
let selectedIds = [];
let path = [], map = [], enemies = [], towers = [], projectiles = [], particles = [], mines = [], floatingTexts = [];
let activePlaceTower = null, selectedTowerInst = null, waveActive = false;
let mouse = { x: 0, y: 0, tx: 0, ty: 0 };

/** SCREEN NAVIGATION **/
function toSelect() {
    document.getElementById('screen-menu').classList.add('hidden');
    document.getElementById('screen-select').classList.remove('hidden');
    const grid = document.getElementById('select-grid');
    TOWER_DB.forEach(t => {
        const card = document.createElement('div');
        card.className = 'tower-card';
        card.innerHTML = `<b>${t.name}</b><br><small>$${t.cost}</small>`;
        card.onclick = () => {
            if(selectedIds.includes(t.id)) {
                selectedIds = selectedIds.filter(id => id !== t.id);
                card.classList.remove('selected');
            } else if(selectedIds.length < 4) {
                selectedIds.push(t.id);
                card.classList.add('selected');
            }
            document.getElementById('select-count').innerText = `${selectedIds.length}/4 SELECTED`;
            document.getElementById('btn-deploy').disabled = selectedIds.length !== 4;
        };
        grid.appendChild(card);
    });
}

function startGame() {
    document.getElementById('screen-select').classList.add('hidden');
    document.getElementById('screen-game').classList.remove('hidden');
    
    // Init Shop
    const shop = document.getElementById('shop');
    selectedIds.forEach(id => {
        const t = TOWER_DB.find(x => x.id === id);
        const btn = document.createElement('button');
        btn.className = 'shop-btn';
        btn.innerHTML = `<b>${t.name}</b>$${t.cost}`;
        btn.onclick = (e) => {
            e.stopPropagation();
            activePlaceTower = t;
            document.querySelectorAll('.shop-btn').forEach(b => b.classList.remove('active'));
            btn.classList.add('active');
        };
        shop.appendChild(btn);
    });

    generateMap();
    requestAnimationFrame(loop);
}

function generateMap() {
    map = Array(ROWS).fill().map(() => Array(COLS).fill(1));
    let x = 0, y = Math.floor(Math.random() * (ROWS - 4)) + 2;
    path = [];
    while(x < COLS) {
        path.push({x, y});
        map[y][x] = 0;
        let r = Math.random();
        if(r < 0.2 && y > 1 && map[y-1][x] !== 0) y--;
        else if(r < 0.4 && y < ROWS - 2 && map[y+1][x] !== 0) y++;
        else x++;
    }
}

/** GAME OBJECTS **/
class Projectile {
    constructor(x, y, target, tower) {
        this.x = x; this.y = y; this.target = target; this.tower = tower;
        this.speed = 10;
    }
    update() {
        let dx = this.target.x - this.x, dy = this.target.y - this.y;
        let d = Math.hypot(dx, dy);
        if(d < 10) { this.hit(); return true; }
        this.x += (dx/d) * this.speed;
        this.y += (dy/d) * this.speed;
        return false;
    }
    hit() {
        if(this.tower.splash) {
            enemies.forEach(e => {
                if(Math.hypot(e.x - this.x, e.y - this.y) < this.tower.splash) e.hp -= this.tower.dmg;
            });
            createExplosion(this.x, this.y, this.tower.color);
        } else {
            this.target.hp -= this.tower.dmg;
        }
    }
    draw() {
        ctx.fillStyle = this.tower.color;
        ctx.shadowBlur = 10; ctx.shadowColor = this.tower.color;
        ctx.beginPath(); ctx.arc(this.x, this.y, 4, 0, Math.PI*2); ctx.fill();
        ctx.shadowBlur = 0;
    }
}

class Enemy {
    constructor(data, wave) {
        this.data = data;
        this.pIdx = 0;
        this.x = path[0].x * TILE + TILE/2;
        this.y = path[0].y * TILE + TILE/2;
        this.maxHp = data.hp * (1 + wave * 0.35);
        this.hp = this.maxHp;
        this.speed = data.speed;
        this.slow = 0;
        this.revealed = !data.stealth;
    }
    update() {
        if(this.slow > 0) this.slow--;
        let s = (this.slow > 0) ? this.speed * 0.5 : this.speed;
        let target = path[this.pIdx];
        let tx = target.x * TILE + TILE/2, ty = target.y * TILE + TILE/2;
        let d = Math.hypot(tx-this.x, ty-this.y);
        
        if(d < s) {
            this.pIdx++;
            if(this.pIdx >= path.length) return 'leak';
        } else {
            this.x += ((tx-this.x)/d)*s; this.y += ((ty-this.y)/d)*s;
        }
        return this.hp <= 0 ? 'die' : null;
    }
    draw() {
        ctx.globalAlpha = (this.data.stealth && !this.revealed) ? 0.1 : 1;
        ctx.fillStyle = this.data.color;
        ctx.beginPath(); ctx.roundRect(this.x-15, this.y-15, 30, 30, 5); ctx.fill();
        
        // HP bar
        ctx.fillStyle = '#444'; ctx.fillRect(this.x-15, this.y-25, 30, 4);
        ctx.fillStyle = '#22c55e'; ctx.fillRect(this.x-15, this.y-25, 30 * (this.hp/this.maxHp), 4);
        ctx.globalAlpha = 1;
    }
}

class Tower {
    constructor(type, tx, ty) {
        this.type = type; this.tx = tx; this.ty = ty;
        this.x = tx*TILE + TILE/2; this.y = ty*TILE + TILE/2;
        this.level = 1; this.range = type.range; this.dmg = type.dmg;
        this.cd = 0; this.plasma = 1;
    }
    update() {
        if(this.cd > 0) this.cd--;
        
        // Radar/Tesla Reveal
        if(this.type.id === 'radar' || this.type.id === 'tesla') {
            enemies.forEach(e => {
                if(Math.hypot(e.x-this.x, e.y-this.y) < this.range) e.revealed = true;
            });
        }

        if(this.type.id === 'mine' && this.cd <= 0) {
            mines.push({x:this.x, y:this.y, dmg:this.dmg, color:this.type.color});
            this.cd = this.type.rate;
        }

        let target = enemies.find(e => e.revealed && Math.hypot(e.x-this.x, e.y-this.y) < this.range);
        
        if(target && this.cd <= 0) {
            if(this.type.type === 'bullet') {
                projectiles.push(new Projectile(this.x, this.y, target, this));
                this.cd = this.type.rate;
            } else if(this.type.type === 'beam') {
                this.fireBeam(target);
            }
        } else { this.plasma = 1; }
    }
    fireBeam(target) {
        if(this.type.id === 'cryo') { target.hp -= this.dmg; target.slow = 30; }
        else if(this.type.id === 'plasma') { target.hp -= this.dmg * this.plasma; this.plasma += 0.02; }
        else { target.hp -= this.dmg; }

        ctx.strokeStyle = this.type.color; ctx.lineWidth = 4;
        ctx.shadowBlur = 10; ctx.shadowColor = this.type.color;
        ctx.beginPath(); ctx.moveTo(this.x, this.y); ctx.lineTo(target.x, target.y); ctx.stroke();
        ctx.shadowBlur = 0;
    }
    draw() {
        ctx.fillStyle = this.type.color;
        ctx.beginPath(); ctx.roundRect(this.tx*TILE+10, this.ty*TILE+10, TILE-20, TILE-20, 8); ctx.fill();
        ctx.fillStyle = 'black'; ctx.font = 'bold 12px sans-serif'; ctx.fillText('V'+this.level, this.x-8, this.y+5);
    }
}

/** MAIN LOOP **/
function loop() {
    if(lives <= 0) { alert("Core Overrun! Wave: " + wave); location.reload(); return; }

    ctx.clearRect(0,0,canvas.width,canvas.height);

    // Draw Map Tiles
    for(let r=0; r<ROWS; r++) {
        for(let c=0; c<COLS; c++) {
            ctx.fillStyle = map[r][c] === 0 ? '#0f172a' : '#020617';
            ctx.fillRect(c*TILE, r*TILE, TILE, TILE);
            ctx.strokeStyle = '#1e293b'; ctx.strokeRect(c*TILE, r*TILE, TILE, TILE);
        }
    }

    // Process Objects
    mines.forEach((m,i) => {
        ctx.fillStyle = m.color; ctx.beginPath(); ctx.arc(m.x, m.y, 8, 0, Math.PI*2); ctx.fill();
        let hit = enemies.find(e => Math.hypot(e.x-m.x, e.y-m.y) < 30);
        if(hit) { hit.hp -= m.dmg; createExplosion(m.x,m.y,m.color); mines.splice(i,1); }
    });

    towers.forEach(t => t.update());
    towers.forEach(t => t.draw());

    for(let i=enemies.length-1; i>=0; i--) {
        let e = enemies[i]; e.revealed = !e.data.stealth;
        let res = e.update();
        if(res === 'leak') { lives--; enemies.splice(i,1); }
        else if(res === 'die') {
            money += e.data.money;
            floatingTexts.push({x:e.x, y:e.y, val:'+$'+e.data.money, life:40});
            createExplosion(e.x, e.y, e.data.color);
            enemies.splice(i,1);
        } else { e.draw(); }
    }

    projectiles = projectiles.filter(p => !p.update());
    projectiles.forEach(p => p.draw());

    // VFX
    particles.forEach((p,i) => {
        p.x += p.vx; p.y += p.vy; p.life--;
        ctx.fillStyle = p.c; ctx.fillRect(p.x, p.y, 2, 2);
        if(p.life <= 0) particles.splice(i,1);
    });

    floatingTexts.forEach((f,i) => {
        ctx.fillStyle = '#22d3ee'; ctx.fillText(f.val, f.x, f.y);
        f.y -= 1; f.life--; if(f.life <= 0) floatingTexts.splice(i,1);
    });

    // Placement Overlays
    if(activePlaceTower) {
        let valid = map[mouse.ty][mouse.tx] === 1 && !towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty);
        ctx.globalAlpha = 0.5;
        ctx.fillStyle = valid ? activePlaceTower.color : '#ef4444';
        ctx.fillRect(mouse.tx*TILE, mouse.ty*TILE, TILE, TILE);
        ctx.beginPath(); ctx.arc(mouse.tx*TILE+TILE/2, mouse.ty*TILE+TILE/2, activePlaceTower.range, 0, Math.PI*2);
        ctx.strokeStyle = 'white'; ctx.stroke();
        ctx.globalAlpha = 1;
    }

    if(selectedTowerInst) {
        ctx.strokeStyle = '#22d3ee'; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.arc(selectedTowerInst.x, selectedTowerInst.y, selectedTowerInst.range, 0, Math.PI*2); ctx.stroke();
    }

    updateHUD();
    requestAnimationFrame(loop);
}

/** HELPERS **/
function createExplosion(x, y, c) {
    for(let i=0; i<10; i++) particles.push({x, y, vx:Math.random()*4-2, vy:Math.random()*4-2, life:20, c});
}

function updateHUD() {
    document.getElementById('hp-val').innerText = lives;
    document.getElementById('money-val').innerText = '$' + Math.floor(money);
    document.getElementById('wave-val').innerText = wave;
}

canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    mouse.x = e.clientX - r.left; mouse.y = e.clientY - r.top;
    mouse.tx = Math.floor(mouse.x / TILE); mouse.ty = Math.floor(mouse.y / TILE);
};

canvas.onmousedown = () => {
    if(activePlaceTower) {
        let valid = map[mouse.ty][mouse.tx] === 1 && !towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty);
        if(valid && money >= activePlaceTower.cost) {
            money -= activePlaceTower.cost;
            towers.push(new Tower(activePlaceTower, mouse.tx, mouse.ty));
            activePlaceTower = null;
            document.querySelectorAll('.shop-btn').forEach(b => b.classList.remove('active'));
        } else { activePlaceTower = null; document.querySelectorAll('.shop-btn').forEach(b => b.classList.remove('active')); }
    } else {
        let t = towers.find(t => t.tx === mouse.tx && t.ty === mouse.ty);
        if(t) { selectedTowerInst = t; showUpgrade(t); }
        else { selectedTowerInst = null; document.getElementById('upgrade-menu').style.display = 'none'; }
    }
};

function showUpgrade(t) {
    const menu = document.getElementById('upgrade-menu'); menu.style.display = 'block';
    const cost = Math.floor(t.type.cost * 0.8 * t.level);
    document.getElementById('up-name').innerText = t.type.name;
    document.getElementById('up-stats').innerHTML = `LEVEL: ${t.level}<br>DMG: ${Math.floor(t.dmg)}<br>RANGE: ${t.range}`;
    const btn = document.getElementById('btn-up');
    btn.innerText = `Upgrade ($${cost})`;
    btn.onclick = () => {
        if(money >= cost) {
            money -= cost; t.level++; t.dmg *= 1.4; t.range += 20; showUpgrade(t);
        }
    };
    document.getElementById('btn-sell').onclick = () => {
        money += Math.floor(t.type.cost * 0.5); towers = towers.filter(x => x !== t);
        menu.style.display = 'none'; selectedTowerInst = null;
    };
}

document.getElementById('btn-next-wave').onclick = () => {
    if(waveActive) return;
    wave++; waveActive = true;
    document.getElementById('btn-next-wave').disabled = true;
    
    let isBoss = (wave % 5 === 0);
    let count = isBoss ? 1 : 5 + wave * 2;
    
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            let pool = ENEMY_DB.filter((e, idx) => idx <= Math.floor(wave / 3));
            if(isBoss) pool = [ENEMY_DB[3]];
            let type = pool[Math.floor(Math.random() * pool.length)];
            enemies.push(new Enemy(type, wave));
            if(i === count - 1) {
                const checkFinished = setInterval(() => {
                    if(enemies.length === 0) { 
                        waveActive = false; 
                        document.getElementById('btn-next-wave').disabled = false;
                        clearInterval(checkFinished);
                    }
                }, 500);
            }
        }, i * (isBoss ? 0 : 800));
    }
};
</script>
</body>
</html>
