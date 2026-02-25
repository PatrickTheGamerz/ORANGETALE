<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: MULTIVERSE BREACH - ABSOLUTE EDITION</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --blue: #008cff; --white: #ffffff; }
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        
        #layout { display: grid; grid-template-columns: 850px 250px; grid-template-rows: 60px 450px 180px; gap: 10px; padding: 10px; border: 4px double white; background: #000; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; border: 1px solid #333; background: #050505; cursor: crosshair; }
        
        #shop { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; gap: 10px; }
        .shop-card { border: 2px solid white; padding: 8px; text-align: center; cursor: pointer; transition: 0.2s; font-size: 0.9rem; }
        .shop-card:hover { background: #222; color: var(--gold); }
        .shop-card.active { border-color: var(--gold); background: #111; color: var(--gold); }

        #bottom { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; }
        #upgrade-panel { width: 320px; border-right: 2px solid white; padding-right: 15px; display: none; }
        #dialogue { flex: 1; font-size: 1.1rem; line-height: 1.4; }

        .btn { background: #000; border: 2px solid #fff; color: #fff; padding: 8px; cursor: pointer; font-family: inherit; width: 100%; text-transform: uppercase; }
        .btn:hover:not(:disabled) { color: var(--gold); border-color: var(--gold); }
        .btn:disabled { opacity: 0.3; }
        
        .choice-row { display: flex; gap: 5px; margin: 5px 0; }
        .choice-btn { font-size: 0.7rem; padding: 4px; border: 1px solid var(--gold); color: var(--gold); background: #000; cursor: pointer; flex: 1; }

        .glitch { animation: gAnim 0.15s infinite; filter: invert(1) hue-rotate(180deg); }
        @keyframes gAnim { 0% { transform: translate(0); } 25% { transform: translate(-4px, 2px); } 75% { transform: translate(4px, -2px); } }
        
        #menu { position: fixed; inset: 0; background: #000; z-index: 1000; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .stat-val { color: var(--gold); font-weight: bold; }
    </style>
</head>
<body id="main-body">

    <div id="menu">
        <h1 style="font-size: 3rem; color: var(--gold);">UTD: MULTIVERSE</h1>
        <p>SELECT 4 GUARDIANS TO PROTECT THE TIMELINE</p>
        <div id="select-grid" style="display:grid; grid-template-columns: repeat(3,1fr); gap:10px; margin:20px;"></div>
        <button id="start-init" class="btn" style="width:200px" disabled>START BATTLE</button>
    </div>

    <div id="layout">
        <header>
            <div>HP: <span id="hp-val" class="stat-val">20</span></div>
            <div>GOLD: <span id="g-val" class="stat-val">600</span></div>
            <div>WAVE: <span id="w-val" class="stat-val">0</span></div>
            <button id="wave-btn" class="btn" style="width:120px; border-color:var(--red)">BATTLE</button>
        </header>

        <canvas id="gameCanvas"></canvas>

        <div id="shop">
            <h4 style="margin:0; text-align:center;">GUARDIANS</h4>
            <div id="shop-list"></div>
        </div>

        <div id="bottom">
            <div id="upgrade-panel">
                <div style="display:flex; justify-content:space-between">
                    <b id="up-name">NAME</b>
                    <span id="up-item" style="color:var(--gold); font-size:0.7rem">Stick</span>
                </div>
                <div id="up-stats" style="font-size:0.8rem; margin:5px 0; color:#aaa;"></div>
                <div id="evolution-box"></div>
                <button id="up-btn" class="btn">LEVEL UP</button>
            </div>
            <div id="dialogue">
                <span style="color:red">❤</span> <span id="diag-text">Stay determined. Select a guardian to view their path.</span>
            </div>
        </div>
    </div>

<script>
/** * WEAPON & ENEMY DATA
 */
const WEAPONS = [
    { lv: 1, name: "Stick", type: "melee", range: 60, dmg: 25 },
    { lv: 4, name: "Toy Knife", type: "melee", range: 70, dmg: 45 },
    { lv: 7, name: "Tough Glove", type: "rapid", range: 100, dmg: 15 },
    { lv: 10, name: "Notebook", type: "utility", range: 150, dmg: 10 },
    { lv: 13, name: "Burnt Pan", type: "splash", range: 120, dmg: 60 },
    { lv: 16, name: "Empty Gun", type: "ranged", range: 250, dmg: 100 },
    { lv: 19, name: "True Knife", type: "melee", range: 90, dmg: 999 }
];

const TOWERS_CFG = [
    { id:'frisk', name:'Frisk', cost:200, color:'#ff00ff' },
    { id:'sans', name:'Sans', cost:500, color:'#008cff' },
    { id:'void', name:'Void', cost:400, color:'#ffffff' },
    { id:'undyne', name:'Undyne', cost:300, color:'#00ffff' },
    { id:'papyrus', name:'Papyrus', cost:150, color:'#ffffff' },
    { id:'asgore', name:'Asgore', cost:450, color:'#ffa500' }
];

const ENEMY_TYPES = [
    { id:'patience', name:'Patience Soul', hp:60, speed:0.8, color:'#00ffff', perk:'slow' },
    { id:'bravery', name:'Bravery Soul', hp:40, speed:2.5, color:'#ffa500', perk:'fast' },
    { id:'kindness', name:'Kindness Soul', hp:150, speed:1.0, color:'#00ff00', perk:'tank' },
    { id:'justice', name:'Justice Soul', hp:80, speed:1.4, color:'#ffff00', perk:'stun' },
    { id:'det', name:'Determination Soul', hp:100, speed:1.2, color:'#ff0000', perk:'revive' },
    { id:'sans_ls', name:'Sans Lost Soul', hp:1, speed:2.0, color:'#008cff', perk:'dodge' },
    { id:'undyne_ls', name:'Undyne Lost Soul', hp:400, speed:1.1, color:'#00ffff', perk:'phase2' },
    { id:'gaster', name:'GASTER', hp:8000, speed:0.5, color:'#000', perk:'glitch' }
];

/** ENGINE SETUP **/
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
const COLS = 17, ROWS = 9;
canvas.width = 850; canvas.height = 450;

let money = 600, hp = 20, wave = 0;
let towers = [], enemies = [], path = [], bullets = [], puppets = [], vfx = [];
let selectedGuardians = [], activeSummon = null, selectedInst = null, waveActive = false;
let mouse = { tx:0, ty:0 };

// Character Select
const grid = document.getElementById('select-grid');
TOWERS_CFG.forEach(t => {
    const d = document.createElement('div');
    d.className = 'shop-card';
    d.innerHTML = `<b>${t.name}</b><br>$${t.cost}`;
    d.onclick = () => {
        if(selectedGuardians.includes(t.id)) {
            selectedGuardians = selectedGuardians.filter(i=>i!==t.id);
            d.style.borderColor = 'white';
        } else if(selectedGuardians.length < 4) {
            selectedGuardians.push(t.id);
            d.style.borderColor = 'var(--gold)';
        }
        document.getElementById('start-init').disabled = selectedGuardians.length !== 4;
    };
    grid.appendChild(d);
});

document.getElementById('start-init').onclick = () => {
    document.getElementById('menu').style.display = 'none';
    const list = document.getElementById('shop-list');
    selectedGuardians.forEach(id => {
        const t = TOWERS_CFG.find(x=>x.id===id);
        const b = document.createElement('div');
        b.className = 'shop-card';
        b.innerHTML = `<b>${t.name}</b><br>$${t.cost}`;
        b.onclick = () => {
            activeSummon = t;
            document.querySelectorAll('.shop-card').forEach(x=>x.classList.remove('active'));
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
        this.lv = 1; this.cd = 0; this.stun = 0;
        this.variant = 'base';
        this.updateStats();
    }
    updateStats() {
        const w = [...WEAPONS].reverse().find(wi => this.lv >= wi.lv) || WEAPONS[0];
        this.weapon = w;
        this.dmg = w.dmg; this.range = w.range; this.rate = 40;
        
        if(this.id === 'sans') { this.range = 300; this.rate = 5; }
        if(this.variant === 'bad_time') this.rate = 2;
        if(this.variant === 'chara') this.dmg = 999;
    }
    update() {
        if(this.stun > 0) { this.stun--; return; }
        if(this.cd > 0) this.cd--;

        if(this.id === 'void' && this.variant !== 'base' && this.cd <= 0) {
            this.spawnPuppet(); this.cd = 120;
        }

        let target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < this.range);
        if(target && this.cd <= 0 && this.id !== 'void') {
            this.attack(target);
            this.cd = this.rate;
        }
    }
    spawnPuppet() {
        puppets.push({
            pIdx: path.length-1, x: path[path.length-1].x*TILE+TILE/2, y: path[path.length-1].y*TILE+TILE/2,
            hp: 10 + this.lv, color: this.variant === 'error' ? '#00f' : '#fff'
        });
    }
    attack(t) {
        if(t.perk === 'dodge' && t.stamina > 0 && Math.random() < 0.5) {
            t.stamina--; 
            vfx.push({x:t.x, y:t.y-20, txt:'MISS', life:30, color:'#fff'});
            return;
        }
        
        if(this.weapon.type === 'melee') {
            bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, color:'#fff', wide:4});
        } else {
            bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:10, color:this.id==='sans'?'#0cf':'#f0f', wide:2});
        }
        
        t.hp -= this.dmg;
        if(t.perk === 'stun' && Math.random() < 0.1) this.stun = 60;
    }
    draw() {
        ctx.fillStyle = this.stun > 0 ? '#555' : (this.variant==='chara'?'#f00':'#fff');
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.fillStyle = 'black'; ctx.font = '10px Courier';
        ctx.fillText("LV"+this.lv, this.x-12, this.y+5);
    }
}

/** ENEMY CLASS **/
class Enemy {
    constructor(data, wave) {
        this.data = data; this.pIdx = 0;
        this.x = path[0].x*TILE+TILE/2; this.y = path[0].y*TILE+TILE/2;
        this.maxHp = data.hp * (1 + wave*0.4); this.hp = this.maxHp;
        this.speed = data.speed; this.perk = data.perk;
        this.lives = (data.perk === 'revive') ? 2 : 1;
        this.stamina = (data.perk === 'dodge') ? 5 : 0;
    }
    update() {
        let t = path[this.pIdx];
        if(!t) return 'leak';
        let tx = t.x*TILE+TILE/2, ty = t.y*TILE+TILE/2;
        let d = Math.hypot(tx-this.x, ty-this.y);
        
        let curSpeed = this.speed;
        if(this.perk === 'dodge' && this.stamina <= 0) curSpeed *= 0.5;

        if(d < curSpeed) this.pIdx++;
        else { this.x += ((tx-this.x)/d)*curSpeed; this.y += ((ty-this.y)/d)*curSpeed; }

        if(this.hp <= 0) {
            this.lives--;
            if(this.lives > 0) { this.hp = this.maxHp; return null; }
            if(this.perk === 'phase2') {
                this.perk = 'none'; this.data.name = "Undying"; this.hp = this.maxHp*2; this.speed *= 1.5;
                vfx.push({x:this.x, y:this.y, txt:'UNDYING', life:60, color:'#0ff'});
                return null;
            }
            return 'die';
        }
        return null;
    }
    draw() {
        ctx.fillStyle = this.data.color;
        ctx.beginPath(); ctx.arc(this.x, this.y, 12, 0, Math.PI*2); ctx.fill();
        ctx.strokeStyle = '#fff'; ctx.lineWidth = 2; ctx.stroke();
        
        ctx.fillStyle = 'red'; ctx.fillRect(this.x-15, this.y-22, 30, 4);
        ctx.fillStyle = '#0f0'; ctx.fillRect(this.x-15, this.y-22, 30*(this.hp/this.maxHp), 4);
    }
}

/** LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    
    // Road
    ctx.fillStyle = '#111';
    path.forEach(p => ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE));

    towers.forEach(t => { t.update(); t.draw(); });

    // Puppets
    for(let i=puppets.length-1; i>=0; i--) {
        let p = puppets[i];
        let target = path[p.pIdx];
        if(!target) { puppets.splice(i,1); continue; }
        let tx = target.x*TILE+TILE/2, ty = target.y*TILE+TILE/2;
        let d = Math.hypot(tx-p.x, ty-p.y);
        if(d < 3) p.pIdx--; else { p.x += ((tx-p.x)/d)*3; p.y += ((ty-p.y)/d)*3; }
        ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, 6, 0, Math.PI*2); ctx.fill();
        enemies.forEach(e => { if(Math.hypot(e.x-p.x, e.y-p.y)<20) { e.hp -= 2; p.hp--; } });
        if(p.hp <= 0) puppets.splice(i,1);
    }

    // Enemies
    for(let i=enemies.length-1; i>=0; i--) {
        let res = enemies[i].update();
        if(res === 'leak') { hp--; enemies.splice(i,1); }
        else if(res === 'die') { money += 40; 
            if(enemies[i].perk === 'glitch') document.getElementById('main-body').classList.remove('glitch');
            enemies.splice(i,1); 
        }
        else enemies[i].draw();
    }

    // Bullets & VFX
    ctx.lineCap = 'round';
    for(let i=bullets.length-1; i>=0; i--) {
        let b = bullets[i]; ctx.strokeStyle = b.color; ctx.lineWidth = b.wide;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    }

    vfx.forEach((v,i) => {
        ctx.fillStyle = v.color; ctx.font = 'bold 14px Courier'; ctx.fillText(v.txt, v.x-20, v.y);
        v.y--; v.life--; if(v.life <= 0) vfx.splice(i,1);
    });

    // Placement
    if(activeSummon) {
        let occ = towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty) || path.find(p=>p.x===mouse.tx && p.y===mouse.ty);
        ctx.globalAlpha = 0.4; ctx.fillStyle = occ ? 'red' : activeSummon.color;
        ctx.fillRect(mouse.tx*TILE, mouse.ty*TILE, TILE, TILE);
        ctx.globalAlpha = 1;
    }

    updateUI();
    if(hp > 0) requestAnimationFrame(loop);
    else alert("GAME OVER. THE TIMELINE WAS ERASED.");
}

/** INTERACTION & UPGRADES **/
canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

canvas.onmousedown = () => {
    if(activeSummon) {
        let occ = towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty) || path.find(p=>p.x===mouse.tx && p.y===mouse.ty);
        if(!occ && money >= activeSummon.cost) {
            money -= activeSummon.cost; towers.push(new Tower(activeSummon, mouse.tx, mouse.ty));
            activeSummon = null; document.querySelectorAll('.shop-card').forEach(x=>x.classList.remove('active'));
        }
    } else {
        let t = towers.find(x=>x.tx===mouse.tx && x.ty===mouse.ty);
        if(t) { selectedInst = t; showUpgrade(t); }
    }
};

function showUpgrade(t) {
    const p = document.getElementById('upgrade-panel'); p.style.display = 'block';
    t.updateStats();
    document.getElementById('up-name').innerText = t.name;
    document.getElementById('up-item').innerText = t.weapon.name;
    document.getElementById('up-stats').innerText = `LV: ${t.lv} | DMG: ${t.dmg} | RNG: ${t.range}`;
    
    const ev = document.getElementById('evolution-box'); ev.innerHTML = "";
    if(t.lv >= 5 && t.variant === 'base') {
        const row = document.createElement('div'); row.className = 'choice-row';
        if(t.id === 'sans') {
            row.appendChild(createEvoBtn("Bad Time", () => { t.variant='bad_time'; t.rate=2; }));
            row.appendChild(createEvoBtn("Dust", () => { t.variant='dust'; t.dmg*=3; }));
            row.appendChild(createEvoBtn("Last Breath", () => { t.variant='lb'; t.rate=15; t.dmg*=10; }));
        }
        if(t.id === 'frisk' && t.lv >= 19) {
            row.appendChild(createEvoBtn("Pacifist", () => { t.variant='pacifist'; t.dmg=0; t.range=500; }));
            row.appendChild(createEvoBtn("Chara", () => { t.variant='chara'; }));
        }
        if(t.id === 'void') {
            row.appendChild(createEvoBtn("Error", () => { t.variant='error'; }));
            row.appendChild(createEvoBtn("Ink", () => { t.variant='ink'; }));
        }
        ev.appendChild(row);
    }

    const upBtn = document.getElementById('up-btn');
    const cost = 80 * t.lv;
    upBtn.innerText = `LEVEL UP ($${cost})`;
    upBtn.disabled = money < cost || t.lv >= 20;
    upBtn.onclick = () => { if(money>=cost) { money -= cost; t.lv++; showUpgrade(t); } };
}

function createEvoBtn(txt, cb) {
    const b = document.createElement('button'); b.className = 'choice-btn'; b.innerText = txt;
    b.onclick = () => { cb(); showUpgrade(selectedInst); }; return b;
}

document.getElementById('wave-btn').onclick = () => {
    if(waveActive) return;
    wave++; waveActive = true;
    let count = 6 + wave * 2;
    document.getElementById('diag-text').innerText = `* Wave ${wave} - The Lost Souls have noticed you.`;
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            let pool = ENEMY_TYPES.slice(0, Math.min(wave, 6));
            if(wave % 10 === 0 && i === 0) pool = [ENEMY_TYPES[7]];
            let type = pool[Math.floor(Math.random()*pool.length)];
            if(type.perk === 'glitch') document.getElementById('main-body').classList.add('glitch');
            enemies.push(new Enemy(type, wave));
            if(i === count-1) {
                let c = setInterval(() => {
                    if(enemies.length === 0) { waveActive = false; clearInterval(c); }
                }, 500);
            }
        }, i * 900);
    }
};

function updateUI() {
    document.getElementById('hp-val').innerText = hp;
    document.getElementById('g-val').innerText = money;
    document.getElementById('w-val').innerText = wave;
}
</script>
</body>
</html>
