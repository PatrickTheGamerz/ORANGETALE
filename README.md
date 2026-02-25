<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: MULTIVERSE BREACH - ADVANCED</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');

        :root {
            --ut-red: #ff0000;
            --ut-white: #ffffff;
            --ut-gold: #ffcc00;
            --ut-bg: #000000;
            --ut-border: #ffffff;
        }

        body {
            background: var(--ut-bg);
            color: var(--ut-white);
            font-family: 'Courier Prime', monospace;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            user-select: none;
        }

        #game-wrapper {
            display: grid;
            grid-template-columns: 800px 300px;
            grid-template-rows: auto 180px;
            gap: 15px;
            padding: 20px;
            border: 4px double white;
            background: black;
            position: relative;
        }

        canvas {
            grid-column: 1;
            grid-row: 1;
            border: 2px solid #333;
            background: #050505;
        }

        /* SIDEBAR / SHOP */
        #sidebar {
            grid-column: 2;
            grid-row: 1 / 3;
            border: 4px double white;
            padding: 15px;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .shop-btn {
            border: 2px solid white;
            padding: 10px;
            text-align: center;
            cursor: pointer;
            background: black;
            color: white;
            font-size: 0.9rem;
        }
        .shop-btn:hover { background: white; color: black; }
        .shop-btn.active { border-color: var(--ut-gold); color: var(--ut-gold); }

        /* BOTTOM BOX */
        #bottom-panel {
            grid-column: 1;
            grid-row: 2;
            border: 4px double white;
            padding: 20px;
            display: flex;
            gap: 30px;
            position: relative;
        }

        #upgrade-view { width: 350px; border-right: 2px solid white; padding-right: 20px; display: none; }
        #dialogue-box { flex: 1; font-size: 1.2rem; }

        .choice-btn {
            background: black;
            border: 2px solid var(--ut-gold);
            color: var(--ut-gold);
            padding: 5px 10px;
            margin: 5px;
            cursor: pointer;
            font-family: inherit;
        }

        /* SCREEN EFFECTS */
        .glitch { animation: glitchAnim 0.1s infinite; filter: hue-rotate(90deg) invert(1); }
        @keyframes glitchAnim {
            0% { transform: translate(0); }
            20% { transform: translate(-5px, 5px); }
            40% { transform: translate(-5px, -5px); }
            60% { transform: translate(5px, 5px); }
            80% { transform: translate(5px, -5px); }
            100% { transform: translate(0); }
        }

        #menu-overlay {
            position: fixed; inset: 0; background: black; z-index: 1000;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }

        .hud-val { color: var(--ut-gold); }
    </style>
</head>
<body id="body-tag">

    <div id="menu-overlay">
        <h1 style="font-size: 3rem; letter-spacing: 5px;">UTD: MULTIVERSE</h1>
        <p>SELECT 4 GUARDIANS</p>
        <div id="starter-grid" style="display:grid; grid-template-columns: repeat(3, 1fr); gap:10px; margin:20px;"></div>
        <button id="btn-init" class="shop-btn" style="width:200px" disabled>INITIALIZE</button>
    </div>

    <div id="game-wrapper">
        <canvas id="gameCanvas"></canvas>

        <div id="sidebar">
            <div style="text-align:center; border-bottom:2px solid white; padding-bottom:5px; margin-bottom:10px;">
                HP: <span id="hp-val" class="hud-val">20</span> | G: <span id="g-val" class="hud-val">600</span><br>
                WAVE: <span id="wave-val" class="hud-val">0</span>
            </div>
            <div id="shop-list"></div>
            <button id="btn-wave" class="shop-btn" style="margin-top:auto; border-color:var(--ut-red)">FIGHT</button>
        </div>

        <div id="bottom-panel">
            <div id="upgrade-view">
                <b id="up-name">Name</b> <span id="up-item" style="font-size:0.7rem; color:#888;">Stick</span>
                <div id="up-stats" style="font-size:0.8rem; margin:5px 0;">LV: 1 | DMG: 10</div>
                <div id="branch-options" style="margin-bottom:10px;"></div>
                <button id="btn-upgrade" class="shop-btn" style="width:100%">LEVEL UP</button>
            </div>
            <div id="dialogue-box">
                <span style="color:red">❤</span> <span id="diag-text">Pick a tower to view its soul path.</span>
            </div>
        </div>
    </div>

<script>
/** * DATA & CONFIGURATION 
 */
const WEAPONS = [
    { lv: 1, name: "Stick" }, { lv: 3, name: "Toy Knife" }, { lv: 7, name: "Tough Glove" },
    { lv: 10, name: "Ballet Shoes" }, { lv: 12, name: "Torn Notebook" }, { lv: 15, name: "Empty Gun" },
    { lv: 19, name: "True Knife" }
];

const TOWERS_BASE = [
    { id: 'frisk', name: 'Frisk', cost: 200, range: 140, dmg: 15, rate: 40, color: '#ff00ff' },
    { id: 'sans', name: 'Sans', cost: 500, range: 300, dmg: 5, rate: 2, color: '#008cff' },
    { id: 'void', name: 'Void', cost: 400, range: 250, dmg: 20, rate: 100, color: '#fff' },
    { id: 'undyne', name: 'Undyne', cost: 300, range: 200, dmg: 40, rate: 50, color: '#00ffff' },
    { id: 'papyrus', name: 'Papyrus', cost: 150, range: 130, dmg: 20, rate: 45, color: '#fff' },
    { id: 'toriel', name: 'Toriel', cost: 250, range: 180, dmg: 55, rate: 90, color: '#a020f0' }
];

const LOST_SOULS = [
    { name: 'Soul: Toriel', hp: 200, speed: 0.8, color: '#a020f0' },
    { name: 'Soul: Papyrus', hp: 120, speed: 1.6, color: '#fff' },
    { name: 'Soul: Sans', hp: 50, speed: 2.5, color: '#008cff' },
    { name: 'GASTER', hp: 5000, speed: 0.4, color: '#000', boss: true }
];

/** ENGINE **/
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const TILE = 50, COLS = 16, ROWS = 9;
canvas.width = 800; canvas.height = 450;

let money = 600, hp = 20, wave = 0;
let towers = [], enemies = [], path = [], projectiles = [], puppets = [];
let selectedStarterIds = [], activeShopTower = null, selectedInst = null, waveActive = false;
let mouse = { x:0, y:0, tx:0, ty:0 };

// Menu Setup
const starterGrid = document.getElementById('starter-grid');
TOWERS_BASE.forEach(t => {
    const d = document.createElement('div');
    d.className = 'shop-btn';
    d.innerHTML = `<b>${t.name}</b><br>$${t.cost}`;
    d.onclick = () => {
        if(selectedStarterIds.includes(t.id)) {
            selectedStarterIds = selectedStarterIds.filter(i => i !== t.id);
            d.style.borderColor = 'white';
        } else if(selectedStarterIds.length < 4) {
            selectedStarterIds.push(t.id);
            d.style.borderColor = 'var(--ut-gold)';
        }
        document.getElementById('btn-init').disabled = selectedStarterIds.length !== 4;
    };
    starterGrid.appendChild(d);
});

document.getElementById('btn-init').onclick = () => {
    document.getElementById('menu-overlay').style.display = 'none';
    const list = document.getElementById('shop-list');
    selectedStarterIds.forEach(id => {
        const t = TOWERS_BASE.find(x => x.id === id);
        const b = document.createElement('div');
        b.className = 'shop-btn';
        b.innerHTML = `<b>${t.name}</b> $${t.cost}`;
        b.onclick = () => {
            activeShopTower = t;
            document.querySelectorAll('.shop-btn').forEach(x => x.classList.remove('active'));
            b.classList.add('active');
        };
        list.appendChild(b);
    });
    generateMap();
    requestAnimationFrame(loop);
};

function generateMap() {
    path = []; let x = 0, y = 4;
    while(x < COLS) {
        path.push({x, y});
        let r = Math.random();
        if(r < 0.2 && y > 1) y--; else if(r < 0.4 && y < ROWS-2) y++; else x++;
    }
}

/** TOWER CLASS **/
class Tower {
    constructor(cfg, tx, ty) {
        this.id = cfg.id; this.name = cfg.name; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = 1; this.cd = 0; this.dmg = cfg.dmg; this.range = cfg.range; this.rate = cfg.rate;
        this.variant = 'base';
        this.weapon = "Stick";
    }
    update() {
        if(this.cd > 0) this.cd--;
        
        // Weapon Logic (Frisk/Chara/Sans)
        const w = [...WEAPONS].reverse().find(wi => this.lv >= wi.lv);
        this.weapon = w ? w.name : "Stick";

        // Puppet Logic
        if(this.id === 'void' && this.variant !== 'base' && this.cd <= 0) {
            this.spawnPuppet();
            this.cd = this.rate;
        }

        let target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < this.range);
        if(target && this.cd <= 0 && this.id !== 'void') {
            this.fire(target);
            this.cd = this.rate;
        }
    }
    spawnPuppet() {
        puppets.push({
            pIdx: path.length - 1,
            x: path[path.length-1].x*TILE+TILE/2,
            y: path[path.length-1].y*TILE+TILE/2,
            color: this.variant === 'error' ? '#00f' : '#fff',
            hp: 5 + this.lv,
            dmg: this.dmg
        });
    }
    fire(t) {
        projectiles.push({x:this.x, y:this.y, tx:t.x, ty:t.y, color:this.variant === 'dust' ? '#f0f' : '#fff', life:5});
        t.hp -= this.dmg;
    }
    draw() {
        ctx.fillStyle = (this.variant === 'chara' || this.variant === 'something_new') ? 'red' : (this.variant === 'base' ? '#555' : 'white');
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.fillStyle = 'white'; ctx.font = '10px Courier';
        ctx.fillText("LV"+this.lv, this.x-10, this.y+5);
    }
}

/** ENEMY CLASS **/
class Enemy {
    constructor(data, wave) {
        this.data = data;
        this.pIdx = 0;
        this.x = path[0].x*TILE+TILE/2; this.y = path[0].y*TILE+TILE/2;
        this.maxHp = data.hp * (1 + wave*0.5); this.hp = this.maxHp;
        this.speed = data.speed;
        if(data.boss) document.getElementById('body-tag').classList.add('glitch');
    }
    update() {
        let t = path[this.pIdx];
        if(!t) return 'leak';
        let tx = t.x*TILE+TILE/2, ty = t.y*TILE+TILE/2;
        let d = Math.hypot(tx-this.x, ty-this.y);
        if(d < this.speed) this.pIdx++;
        else { this.x += ((tx-this.x)/d)*this.speed; this.y += ((ty-this.y)/d)*this.speed; }
        return this.hp <= 0 ? 'die' : null;
    }
    draw() {
        ctx.fillStyle = this.data.color;
        ctx.beginPath(); ctx.arc(this.x, this.y, 12, 0, Math.PI*2); ctx.fill();
        ctx.strokeStyle = 'white'; ctx.stroke();
        ctx.fillStyle = 'red'; ctx.fillRect(this.x-15, this.y-20, 30, 4);
        ctx.fillStyle = 'lime'; ctx.fillRect(this.x-15, this.y-20, 30*(this.hp/this.maxHp), 4);
    }
}

/** MAIN LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    
    // Draw Path
    ctx.fillStyle = '#111';
    path.forEach(p => ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE));

    towers.forEach(t => { t.update(); t.draw(); });

    // Puppet Logic
    for(let i=puppets.length-1; i>=0; i--) {
        let p = puppets[i];
        let target = path[p.pIdx];
        if(!target) { puppets.splice(i,1); continue; }
        let tx = target.x*TILE+TILE/2, ty = target.y*TILE+TILE/2;
        let d = Math.hypot(tx-p.x, ty-p.y);
        if(d < 3) p.pIdx--;
        else { p.x += ((tx-p.x)/d)*3; p.y += ((ty-p.y)/d)*3; }
        
        ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, 6, 0, Math.PI*2); ctx.fill();
        
        enemies.forEach(e => {
            if(Math.hypot(e.x-p.x, e.y-p.y) < 20) { e.hp -= p.dmg; p.hp--; }
        });
        if(p.hp <= 0) puppets.splice(i,1);
    }

    for(let i=enemies.length-1; i>=0; i--) {
        let res = enemies[i].update();
        if(res === 'leak') { hp--; enemies.splice(i,1); }
        else if(res === 'die') { 
            money += 40; 
            if(enemies[i].data.boss) document.getElementById('body-tag').classList.remove('glitch');
            enemies.splice(i,1); 
        }
        else enemies[i].draw();
    }

    // Beams
    for(let i=projectiles.length-1; i>=0; i--) {
        let p = projectiles[i];
        ctx.strokeStyle = p.color; ctx.lineWidth = p.life;
        ctx.beginPath(); ctx.moveTo(p.x, p.y); ctx.lineTo(p.tx, p.ty); ctx.stroke();
        p.life--; if(p.life <= 0) projectiles.splice(i,1);
    }

    updateUI();
    if(hp > 0) requestAnimationFrame(loop);
    else alert("THE TIMELINE COLLAPSED.");
}

/** INTERACTION & EVOLUTION **/
canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

canvas.onmousedown = () => {
    if(activeShopTower) {
        if(money >= activeShopTower.cost) {
            money -= activeShopTower.cost;
            towers.push(new Tower(activeShopTower, mouse.tx, mouse.ty));
            activeShopTower = null;
            document.querySelectorAll('.shop-btn').forEach(x => x.classList.remove('active'));
        }
    } else {
        let t = towers.find(x => x.tx === mouse.tx && x.ty === mouse.ty);
        if(t) { selectedInst = t; showUpgrade(t); }
    }
};

function showUpgrade(t) {
    const view = document.getElementById('upgrade-view');
    view.style.display = 'block';
    document.getElementById('up-name').innerText = t.name;
    document.getElementById('up-item').innerText = `[${t.weapon}]`;
    document.getElementById('up-stats').innerText = `LV: ${t.lv} | DMG: ${Math.floor(t.dmg)} | RNG: ${t.range}`;
    
    const branch = document.getElementById('branch-options');
    branch.innerHTML = "";
    
    // Evolution Choices
    if(t.id === 'sans' && t.lv === 5 && t.variant === 'base') {
        createBranchBtn("Bad Time", () => { t.variant = 'bad_time'; t.rate = 1; t.range += 100; });
        createBranchBtn("Dust", () => { t.variant = 'dust'; t.dmg *= 3; });
        createBranchBtn("Last Breath", () => { t.variant = 'last_breath'; t.rate = 5; t.dmg *= 5; });
        if(towers.find(x => x.variant === 'chara')) {
            createBranchBtn("Killer", () => { 
                t.variant = 'something_new'; t.dmg *= 10; 
                let chara = towers.find(x => x.variant === 'chara');
                towers = towers.filter(x => x !== chara);
            });
        }
    }
    
    if(t.id === 'frisk' && t.lv === 20 && t.variant === 'base') {
        createBranchBtn("True Pacifist", () => { t.variant = 'pacifist'; t.dmg = 0; t.range = 500; });
        createBranchBtn("Chara", () => { t.variant = 'chara'; t.dmg = 999; t.rate = 100; });
    }

    if(t.id === 'void' && t.lv === 1 && t.variant === 'base') {
        createBranchBtn("Error", () => { t.variant = 'error'; t.rate = 80; });
        createBranchBtn("Ink", () => { t.variant = 'ink'; t.rate = 120; });
    }

    const upBtn = document.getElementById('btn-upgrade');
    const upCost = 50 * t.lv;
    upBtn.innerText = `LEVEL UP ($${upCost})`;
    
    // Max Level Check
    let isMax = false;
    if(t.variant === 'bad_time' && t.lv >= 10) isMax = true;
    if(t.variant === 'last_breath' && t.lv >= 15) isMax = true;
    if(t.lv >= 20) isMax = true;

    upBtn.disabled = isMax || money < upCost;
    upBtn.onclick = () => {
        if(money >= upCost) {
            money -= upCost; t.lv++; t.dmg *= 1.25; t.range += 5;
            showUpgrade(t);
        }
    };
}

function createBranchBtn(label, cb) {
    const b = document.createElement('button');
    b.className = 'choice-btn'; b.innerText = label;
    b.onclick = () => { cb(); showUpgrade(selectedInst); };
    document.getElementById('branch-options').appendChild(b);
}

document.getElementById('btn-wave').onclick = () => {
    if(waveActive) return;
    wave++; waveActive = true;
    let count = 5 + wave * 2;
    document.getElementById('diag-text').innerText = `* Wave ${wave} - Lost Souls are emerging.`;
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            let type = LOST_SOULS[Math.floor(Math.random()*3)];
            if(wave % 10 === 0 && i === 0) type = LOST_SOULS[3];
            enemies.push(new Enemy(type, wave));
            if(i === count-1) {
                let c = setInterval(() => {
                    if(enemies.length === 0) { waveActive = false; clearInterval(c); }
                }, 500);
            }
        }, i * 800);
    }
};

function updateUI() {
    document.getElementById('hp-val').innerText = hp;
    document.getElementById('g-val').innerText = money;
    document.getElementById('wave-val').innerText = wave;
}

</script>
</body>
</html>
