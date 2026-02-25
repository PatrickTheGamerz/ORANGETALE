<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Defender Core: Ultra Edition</title>
    <style>
        :root { --bg: #050a14; --panel: #111827; --accent: #00f2ff; --text: #e2e8f0; --gold: #fbbf24; }
        body { background: var(--bg); color: var(--text); font-family: 'Courier New', monospace; margin: 0; overflow: hidden; display: flex; flex-direction: column; align-items: center; user-select: none; }
        #ui-layer { position: absolute; inset: 0; pointer-events: none; display: flex; justify-content: center; align-items: center; z-index: 100; }
        .menu-panel { background: var(--panel); padding: 30px; border-radius: 4px; border: 2px solid var(--accent); text-align: center; pointer-events: auto; box-shadow: 0 0 50px rgba(0, 242, 255, 0.2); }
        .tower-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin: 20px 0; }
        .tower-card { background: #1f2937; padding: 15px; border: 1px solid #374151; cursor: pointer; transition: 0.2s; }
        .tower-card.selected { border-color: var(--accent); background: #0e7490; transform: translateY(-5px); }
        canvas { background: #050a14; cursor: crosshair; image-rendering: pixelated; }
        #hud { position: absolute; top: 10px; left: 10px; right: 10px; display: flex; justify-content: space-between; pointer-events: auto; padding: 10px; background: rgba(0,0,0,0.5); border-bottom: 1px solid var(--accent); }
        .stat-box { display: flex; flex-direction: column; align-items: center; }
        .val { font-size: 1.5rem; color: var(--accent); font-weight: bold; }
        #tower-shop { position: absolute; bottom: 20px; display: flex; gap: 10px; pointer-events: auto; }
        .shop-btn { width: 100px; height: 110px; background: var(--panel); border: 1px solid #374151; color: white; cursor: pointer; font-family: inherit; }
        .shop-btn.active { border-color: var(--accent); box-shadow: 0 0 10px var(--accent); }
        #upgrade-menu { position: absolute; right: 20px; top: 100px; background: var(--panel); padding: 20px; border: 1px solid var(--accent); display: none; pointer-events: auto; width: 200px; }
        .btn-action { background: var(--accent); color: black; border: none; padding: 10px; width: 100%; font-weight: bold; cursor: pointer; margin-top: 5px; }
        .btn-speed { position: absolute; top: 80px; left: 10px; pointer-events: auto; background: #374151; color: white; border: none; padding: 10px; cursor: pointer; }
    </style>
</head>
<body>

    <div id="ui-layer">
        <div id="selection-menu" class="menu-panel">
            <h1 style="color:var(--accent)">LOADOUT SELECTION</h1>
            <p>Choose 4 towers to initialize the defense grid</p>
            <div id="tower-options" class="tower-grid"></div>
            <button id="start-btn" class="btn-action" style="width:200px" disabled>START SESSION</button>
        </div>

        <div id="game-over" class="menu-panel hidden">
            <h1 style="color:#ef4444">CORE TERMINATED</h1>
            <button class="btn-action" onclick="location.reload()">REBOOT</button>
        </div>
    </div>

    <div id="hud" class="hidden">
        <div class="stat-box"><span>HEALTH</span><span id="hp-val" class="val">20</span></div>
        <div class="stat-box"><span>CREDITS</span><span id="money-val" class="val" style="color:var(--gold)">$500</span></div>
        <div class="stat-box"><span>WAVE</span><span id="wave-val" class="val">0</span></div>
        <button id="next-wave" class="btn-action" style="width:150px">START WAVE</button>
    </div>

    <button id="speed-btn" class="btn-speed hidden">SPEED: 1X</button>

    <div id="tower-shop" class="hidden"></div>

    <div id="upgrade-menu">
        <h3 id="up-name" style="margin:0">Sentry</h3>
        <p id="up-stats" style="font-size:0.8rem; color:#9ca3af"></p>
        <button id="up-btn" class="btn-action">UPGRADE</button>
        <button id="sell-btn" class="btn-action" style="background:#ef4444; color:white">SCRAP</button>
    </div>

    <canvas id="gameCanvas"></canvas>

<script>
const TILE = 64;
const COLS = 16, ROWS = 10;
const W = COLS * TILE, H = ROWS * TILE;

const TOWER_DATA = [
    { id:'gun', name:'Sentry', cost:100, range:180, dmg:12, rate:30, color:'#94a3b8', type:'proj', desc:'Fast ballistic fire' },
    { id:'rail', name:'Railgun', cost:250, range:400, dmg:100, rate:120, color:'#fbbf24', type:'proj', desc:'Heavy armor piercing' },
    { id:'cryo', name:'Cryo', cost:150, range:140, dmg:2, rate:10, color:'#38bdf8', type:'beam', desc:'Slows enemy units' },
    { id:'radar', name:'Radar', cost:125, range:220, dmg:0, rate:0, color:'#c084fc', type:'aura', detect:true, desc:'Detection pulse' },
    { id:'tesla', name:'Tesla', cost:300, range:120, dmg:20, rate:8, color:'#4ade80', type:'beam', detect:true, desc:'Chain lightning' },
    { id:'hive', name:'Swarm', cost:400, range:500, dmg:45, rate:80, color:'#f472b6', type:'proj', desc:'Long-range missiles' },
    { id:'plasma', name:'Plasma', cost:350, range:200, dmg:2, rate:1, color:'#facc15', type:'beam', desc:'Heat ramp damage' },
    { id:'mine', name:'Mine', cost:200, range:80, dmg:150, rate:200, color:'#ef4444', type:'mine', desc:'Deploys path bombs' },
    { id:'mortar', name:'Mortar', cost:275, range:300, dmg:60, rate:150, color:'#ea580c', type:'proj', splash:80, desc:'Area explosion' }
];

const ENEMY_DATA = [
    { name:'Drone', hp:50, speed:2, color:'#f8fafc', money:20 },
    { name:'Phalanx', hp:300, speed:0.8, color:'#475569', money:50 },
    { name:'Wraith', hp:80, speed:1.6, color:'#818cf8', money:40, stealth:true },
    { name:'Goliath', hp:2500, speed:0.5, color:'#dc2626', money:250, boss:true }
];

/** ENGINE STATE **/
let canvas = document.getElementById('gameCanvas'), ctx = canvas.getContext('2d');
canvas.width = W; canvas.height = H;
let towers = [], enemies = [], projectiles = [], particles = [], mines = [], floatingTexts = [];
let selectedIds = [], path = [], map = [], wave = 0, money = 600, lives = 20, speed = 1;
let activePlace = null, selectedInst = null, waveActive = false;
let mouse = { x:0, y:0, tx:0, ty:0 };

/** INITIALIZE SELECTION **/
const optionsDiv = document.getElementById('tower-options');
TOWER_DATA.forEach(t => {
    const card = document.createElement('div');
    card.className = 'tower-card';
    card.innerHTML = `<b>${t.name}</b><br><span style="font-size:0.7rem">${t.desc}</span><br><b style="color:var(--gold)">$${t.cost}</b>`;
    card.onclick = () => {
        if(selectedIds.includes(t.id)) {
            selectedIds = selectedIds.filter(i => i !== t.id);
            card.classList.remove('selected');
        } else if(selectedIds.length < 4) {
            selectedIds.push(t.id);
            card.classList.add('selected');
        }
        document.getElementById('start-btn').disabled = selectedIds.length !== 4;
    };
    optionsDiv.appendChild(card);
});

document.getElementById('start-btn').onclick = () => {
    document.getElementById('selection-menu').classList.add('hidden');
    document.getElementById('hud').classList.remove('hidden');
    document.getElementById('speed-btn').classList.remove('hidden');
    document.getElementById('tower-shop').classList.remove('hidden');
    initShop(); generateMap(); requestAnimationFrame(loop);
};

function initShop() {
    const shop = document.getElementById('tower-shop');
    selectedIds.forEach(id => {
        const t = TOWER_DATA.find(x => x.id === id);
        const btn = document.createElement('button');
        btn.className = 'shop-btn';
        btn.innerHTML = `<b style="font-size:1rem">${t.name}</b><br>$${t.cost}`;
        btn.onclick = (e) => { 
            e.stopPropagation(); activePlace = t; 
            document.querySelectorAll('.shop-btn').forEach(b=>b.classList.remove('active'));
            btn.classList.add('active');
        };
        shop.appendChild(btn);
    });
}

function generateMap() {
    map = Array(ROWS).fill().map(() => Array(COLS).fill(1));
    let x = 0, y = Math.floor(Math.random()*(ROWS-4))+2;
    path = [];
    while(x < COLS) {
        path.push({x, y}); map[y][x] = 0;
        let r = Math.random();
        if(r < 0.2 && y > 1 && map[y-1][x] !== 0) y--;
        else if(r < 0.4 && y < ROWS-2 && map[y+1][x] !== 0) y++;
        else x++;
    }
}

/** GAME CLASSES **/
class Projectile {
    constructor(x, y, target, tower) {
        this.x = x; this.y = y; this.target = target; this.tower = tower;
        this.speed = 8;
    }
    update() {
        let dx = this.target.x - this.x, dy = this.target.y - this.y;
        let dist = Math.hypot(dx, dy);
        if(dist < 10) {
            this.hit(); return true;
        }
        this.x += (dx/dist) * this.speed;
        this.y += (dy/dist) * this.speed;
        return false;
    }
    hit() {
        if(this.tower.type.splash) {
            enemies.forEach(e => {
                if(Math.hypot(e.x - this.x, e.y - this.y) < this.tower.type.splash) e.hp -= this.tower.dmg;
            });
            createParticles(this.x, this.y, this.tower.type.color);
        } else {
            this.target.hp -= this.tower.dmg;
        }
    }
    draw() {
        ctx.fillStyle = this.tower.type.color;
        ctx.beginPath(); ctx.arc(this.x, this.y, 4, 0, Math.PI*2); ctx.fill();
    }
}

class Enemy {
    constructor(data, wave) {
        this.data = data;
        this.pIdx = 0;
        this.x = path[0].x * TILE + TILE/2;
        this.y = path[0].y * TILE + TILE/2;
        this.hp = data.hp * (1 + wave*0.4);
        this.maxHp = this.hp;
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
            if(this.pIdx >= path.length) return "leak";
        } else {
            this.x += ((tx-this.x)/d)*s; this.y += ((ty-this.y)/d)*s;
        }
        if(this.hp <= 0) return "die";
        return null;
    }
    draw() {
        ctx.globalAlpha = (this.data.stealth && !this.revealed) ? 0.1 : 1;
        ctx.fillStyle = this.data.color;
        ctx.fillRect(this.x-12, this.y-12, 24, 24);
        ctx.fillStyle = 'red'; ctx.fillRect(this.x-15, this.y-20, 30, 4);
        ctx.fillStyle = '#00ff00'; ctx.fillRect(this.x-15, this.y-20, 30*(this.hp/this.maxHp), 4);
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
        if(this.type.detect) {
            enemies.forEach(e => { if(Math.hypot(e.x-this.x, e.y-this.y) < this.range) e.revealed = true; });
        }
        if(this.type.id === 'mine' && this.cd <= 0) {
            mines.push({x:this.x, y:this.y, dmg:this.dmg, color:this.type.color});
            this.cd = this.type.rate;
        }
        let target = enemies.find(e => e.revealed && Math.hypot(e.x-this.x, e.y-this.y) < this.range);
        if(target && this.cd <= 0) {
            if(this.type.type === 'proj') {
                projectiles.push(new Projectile(this.x, this.y, target, this));
                this.cd = this.type.rate;
            } else if(this.type.type === 'beam') {
                if(this.type.id === 'cryo') { target.hp -= this.dmg; target.slow = 30; }
                else if(this.type.id === 'plasma') { target.hp -= this.dmg * this.plasma; this.plasma += 0.02; }
                else { target.hp -= this.dmg; }
                ctx.strokeStyle = this.type.color; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.moveTo(this.x, this.y); ctx.lineTo(target.x, target.y); ctx.stroke();
            }
        } else { this.plasma = 1; }
    }
    draw() {
        ctx.fillStyle = this.type.color;
        ctx.beginPath(); ctx.roundRect(this.tx*TILE+8, this.ty*TILE+8, TILE-16, TILE-16, 5); ctx.fill();
        ctx.fillStyle = "black"; ctx.font = "bold 12px Arial";
        ctx.fillText("V"+this.level, this.x-8, this.y+5);
    }
}

/** LOOP **/
function loop() {
    for(let i=0; i<speed; i++) update();
    draw();
    requestAnimationFrame(loop);
}

function update() {
    if(lives <= 0) return;
    
    // Process Enemies
    for(let i=enemies.length-1; i>=0; i--) {
        enemies[i].revealed = !enemies[i].data.stealth;
        let status = enemies[i].update();
        if(status === "leak") { lives--; enemies.splice(i,1); }
        else if(status === "die") {
            let e = enemies[i];
            money += e.data.money;
            floatingTexts.push({x:e.x, y:e.y, txt:`+$${e.data.money}`, life:50});
            createParticles(e.x, e.y, e.data.color);
            enemies.splice(i,1);
        }
    }

    towers.forEach(t => t.update());
    
    projectiles = projectiles.filter(p => !p.update());
    mines.forEach((m, i) => {
        let hit = enemies.find(e => Math.hypot(e.x-m.x, e.y-m.y) < 30);
        if(hit) { hit.hp -= m.dmg; createParticles(m.x, m.y, m.color); mines.splice(i,1); }
    });

    particles.forEach((p,i) => { 
        p.x += p.vx; p.y += p.vy; p.life--; 
        if(p.life <= 0) particles.splice(i,1); 
    });
    
    floatingTexts.forEach((f,i) => { f.y -= 0.5; f.life--; if(f.life <= 0) floatingTexts.splice(i,1); });

    if(waveActive && enemies.length === 0) { waveActive = false; document.getElementById('next-wave').disabled = false; }
}

function draw() {
    ctx.clearRect(0,0,W,H);
    // Road
    ctx.fillStyle = "#0f172a";
    path.forEach(p => ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE));
    
    mines.forEach(m => { ctx.fillStyle = m.color; ctx.beginPath(); ctx.arc(m.x, m.y, 6, 0, Math.PI*2); ctx.fill(); });
    towers.forEach(t => t.draw());
    enemies.forEach(e => e.draw());
    projectiles.forEach(p => p.draw());
    
    particles.forEach(p => { ctx.fillStyle = p.c; ctx.fillRect(p.x, p.y, 2, 2); });
    floatingTexts.forEach(f => { ctx.fillStyle = "#4ade80"; ctx.fillText(f.txt, f.x, f.y); });

    // Placement Ghost
    if(activePlace) {
        let can = map[mouse.ty][mouse.tx] === 1 && !towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty);
        ctx.globalAlpha = 0.4;
        ctx.fillStyle = can ? activePlace.color : "red";
        ctx.fillRect(mouse.tx*TILE, mouse.ty*TILE, TILE, TILE);
        ctx.beginPath(); ctx.arc(mouse.tx*TILE+TILE/2, mouse.ty*TILE+TILE/2, activePlace.range, 0, Math.PI*2);
        ctx.strokeStyle = "white"; ctx.stroke();
        ctx.globalAlpha = 1;
    }
    
    if(selectedInst) {
        ctx.strokeStyle = varColor('--accent');
        ctx.beginPath(); ctx.arc(selectedInst.x, selectedInst.y, selectedInst.range, 0, Math.PI*2); ctx.stroke();
    }

    document.getElementById('hp-val').innerText = lives;
    document.getElementById('money-val').innerText = `$${Math.floor(money)}`;
    document.getElementById('wave-val').innerText = wave;
}

/** HELPERS **/
function createParticles(x, y, c) {
    for(let i=0; i<8; i++) particles.push({x, y, vx:Math.random()*4-2, vy:Math.random()*4-2, life:20, c});
}

function varColor(name) { return getComputedStyle(document.documentElement).getPropertyValue(name); }

canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    mouse.x = e.clientX - r.left; mouse.y = e.clientY - r.top;
    mouse.tx = Math.floor(mouse.x/TILE); mouse.ty = Math.floor(mouse.y/TILE);
};

canvas.onmousedown = () => {
    let t = towers.find(t => t.tx === mouse.tx && t.ty === mouse.ty);
    if(activePlace) {
        if(!t && map[mouse.ty][mouse.tx] === 1 && money >= activePlace.cost) {
            money -= activePlace.cost;
            towers.push(new Tower(activePlace, mouse.tx, mouse.ty));
        }
        activePlace = null;
        document.querySelectorAll('.shop-btn').forEach(b=>b.classList.remove('active'));
    } else if(t) {
        selectedInst = t; showUpgrade(t);
    } else {
        selectedInst = null; document.getElementById('upgrade-menu').style.display='none';
    }
};

function showUpgrade(t) {
    const m = document.getElementById('upgrade-menu'); m.style.display='block';
    const cost = Math.floor(t.type.cost * 0.8 * t.level);
    document.getElementById('up-name').innerText = t.type.name;
    document.getElementById('up-stats').innerHTML = `LVL: ${t.level}<br>DMG: ${t.dmg}<br>RANGE: ${t.range}`;
    document.getElementById('up-btn').innerText = `UPGRADE ($${cost})`;
    document.getElementById('up-btn').onclick = () => {
        if(money >= cost) {
            money -= cost; t.level++; t.dmg = Math.ceil(t.dmg*1.4); t.range += 20; showUpgrade(t);
        }
    };
    document.getElementById('sell-btn').onclick = () => {
        money += Math.floor(t.type.cost * 0.5); towers = towers.filter(x=>x!==t);
        m.style.display='none'; selectedInst = null;
    };
}

document.getElementById('next-wave').onclick = () => {
    wave++; waveActive = true; document.getElementById('next-wave').disabled = true;
    let count = 5 + wave * 2;
    let isBossWave = (wave % 5 === 0);
    for(let i=0; i < (isBossWave ? 1 : count); i++) {
        setTimeout(() => {
            let type = isBossWave ? ENEMY_DATA[3] : ENEMY_DATA[Math.floor(Math.random()*3)];
            enemies.push(new Enemy(type, wave));
        }, i * 600);
    }
};

document.getElementById('speed-btn').onclick = () => {
    speed = (speed === 1) ? 3 : 1;
    document.getElementById('speed-btn').innerText = `SPEED: ${speed}X`;
};
</script>
</body>
</html>
