<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: JUDGMENT EDITION</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --orange: #ffa500; --blue: #0000ff; --green: #00ff00; --yellow: #ffff00; --purple: #a020f0; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        
        #game-container { display: grid; grid-template-columns: 850px 250px; grid-template-rows: 60px 450px 180px; gap: 10px; padding: 10px; border: 5px double white; background: #000; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #222; image-rendering: pixelated; }
        
        #shop { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; gap: 10px; }
        .shop-card { border: 2px solid white; padding: 12px; text-align: center; cursor: pointer; transition: 0.1s; font-size: 0.9rem; }
        .shop-card:hover { background: #fff; color: #000; }
        .shop-card.active { border-color: var(--gold); background: #1a1a1a; color: var(--gold); }

        #bottom-ui { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; }
        #upgrade-panel { width: 350px; border-right: 2px solid white; padding-right: 20px; display: none; }
        #dialogue { flex: 1; font-size: 1.2rem; line-height: 1.4; color: #fff; }

        .btn { background: #000; border: 2px solid #fff; color: #fff; padding: 10px; cursor: pointer; font-family: inherit; width: 100%; text-transform: uppercase; font-weight: bold; }
        .btn:hover:not(:disabled) { color: var(--gold); border-color: var(--gold); }
        .btn:disabled { opacity: 0.3; cursor: not-allowed; }

        .glitch-fx { animation: glitch 0.1s infinite; filter: invert(1) hue-rotate(180deg); }
        @keyframes glitch { 0% { transform: translate(2px, -2px); } 50% { transform: translate(-2px, 2px); } 100% { transform: translate(0); } }
        
        #overlay { position: fixed; inset: 0; background: #000; z-index: 9999; display: flex; flex-direction: column; align-items: center; justify-content: center; border: 10px double white; margin: 15px; }
        .stat-val { color: var(--gold); font-weight: bold; }
    </style>
</head>
<body id="main-body">

    <div id="overlay">
        <h1 style="font-size: 4rem; color: var(--red); margin: 0; letter-spacing: 10px;">UTD</h1>
        <p>CORE DEFENSE: THE TIMELINE HANGS BY A THREAD</p>
        <div id="select-grid" style="display:grid; grid-template-columns: repeat(3, 1fr); gap:12px; margin: 30px;"></div>
        <button id="btn-start" class="btn" style="width: 280px;" disabled>INITIATE DEFENSE</button>
    </div>

    <div id="game-container">
        <header>
            <div>DETERMINATION: <span id="hp-val" style="color:var(--red);">999</span></div>
            <div>GOLD: <span id="g-val" class="stat-val">600</span></div>
            <div>WAVE: <span id="w-val" class="stat-val">0</span></div>
            <button id="btn-next-wave" class="btn" style="width:140px; margin:0; border-color:var(--red);">FIGHT</button>
        </header>

        <canvas id="gameCanvas"></canvas>

        <div id="shop">
            <h3 style="margin:0; text-align:center; color:var(--gold);">SUMMON</h3>
            <div id="shop-list"></div>
        </div>

        <div id="bottom-ui">
            <div id="upgrade-panel">
                <div style="display:flex; justify-content:space-between; align-items:center;">
                    <b id="up-name">NAME</b>
                    <span id="up-weapon" style="color:var(--cyan); font-size:0.8rem;">Stick</span>
                </div>
                <div id="up-stats" style="font-size:0.8rem; margin:8px 0; color:#aaa;">LV: 1 | DMG: 150</div>
                <div id="evo-container"></div>
                <button id="btn-upgrade" class="btn">EXP + LEVEL UP</button>
            </div>
            <div id="dialogue">
                <span style="color:red">❤</span> <span id="diag-text">If a single Soul reaches the end, the world ends. Stay focused.</span>
            </div>
        </div>
    </div>

<script>
/** ⚔ DATA ARRAYS
 */
const FRISK_ITEMS = [
    { lv: 1,  name: "Stick",        dmg: 250, range: 70,  type: "melee" },
    { lv: 4,  name: "Toy Knife",    dmg: 450, range: 90,  type: "melee" },
    { lv: 7,  name: "Tough Glove",  dmg: 80,  range: 130, type: "rapid" },
    { lv: 10, name: "Notebook",     dmg: 40,  range: 160, type: "utility" },
    { lv: 13, name: "Burnt Pan",    dmg: 200, range: 140, type: "splash" },
    { lv: 16, name: "Empty Gun",    dmg: 500, range: 350, type: "sniper" },
    { lv: 19, name: "True Knife",   dmg: 9999,range: 110, type: "god" }
];

const GUARDIANS = [
    { id: 'frisk', name: 'Frisk', cost: 200, color: '#ff00ff' },
    { id: 'sans',  name: 'Sans',  cost: 500, color: '#008cff' },
    { id: 'void',  name: 'Void',  cost: 400, color: '#ffffff' },
    { id: 'undyne',name: 'Undyne',cost: 350, color: '#00ffff' },
    { id: 'asgore',name: 'Asgore',cost: 450, color: '#ffa500' },
    { id: 'papy',  name: 'Papyrus',cost: 150, color: '#ffffff' }
];

const SOULS = [
    { name: 'Patience', hp: 80,  speed: 0.6, color: 'var(--cyan)',   perk: 'cyan' },
    { name: 'Bravery',  hp: 60,  speed: 1.8, color: 'var(--orange)', perk: 'orange' },
    { name: 'Integrity',hp: 90,  speed: 1.2, color: 'var(--blue)',   perk: 'blue' },
    { name: 'Kindness', hp: 200, speed: 0.7, color: 'var(--green)',  perk: 'green' },
    { name: 'Justice',  hp: 120, speed: 1.3, color: 'var(--yellow)', perk: 'yellow' },
    { name: 'Determination', hp: 250, speed: 1.4, color: 'var(--red)', perk: 'red' },
    { name: 'GASTER',   hp: 12000,speed: 0.5, color: '#000', perk: 'glitch' }
];

/** ENGINE SETUP **/
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const TILE = 50, COLS = 17, ROWS = 9;
canvas.width = 850; canvas.height = 450;

let money = 600, coreHP = 999, wave = 0;
let towers = [], enemies = [], path = [], bullets = [], puppets = [], vfx = [];
let myLoadout = [], activeSummon = null, selectedInst = null, waveActive = false;
let mouse = { tx: 0, ty: 0 };

// Character Selection Logic
const grid = document.getElementById('select-grid');
GUARDIANS.forEach(g => {
    const d = document.createElement('div');
    d.className = 'shop-card';
    d.innerHTML = `<b>${g.name}</b><br>$${g.cost}`;
    d.onclick = () => {
        if(myLoadout.includes(g.id)) {
            myLoadout = myLoadout.filter(i => i !== g.id);
            d.style.borderColor = 'white';
        } else if(myLoadout.length < 4) {
            myLoadout.push(g.id);
            d.style.borderColor = 'var(--gold)';
        }
        document.getElementById('btn-start').disabled = myLoadout.length !== 4;
    };
    grid.appendChild(d);
});

document.getElementById('btn-start').onclick = () => {
    document.getElementById('overlay').style.display = 'none';
    const list = document.getElementById('shop-list');
    myLoadout.forEach(id => {
        const g = GUARDIANS.find(x => x.id === id);
        const b = document.createElement('div');
        b.className = 'shop-card';
        b.innerHTML = `<b>${g.name}</b><br>$${g.cost}`;
        b.onclick = () => {
            activeSummon = g;
            document.querySelectorAll('.shop-card').forEach(x => x.classList.remove('active'));
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

/** CLASSES **/
class Tower {
    constructor(cfg, tx, ty) {
        this.id = cfg.id; this.name = cfg.name; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = 1; this.cd = 0; this.stun = 0;
        this.variant = 'base';
        this.updateStats();
    }
    updateStats() {
        if(this.id === 'frisk') {
            const w = [...FRISK_ITEMS].reverse().find(wi => this.lv >= wi.lv);
            this.weapon = w;
            this.dmg = w.dmg; this.range = w.range; this.rate = (w.type === 'rapid') ? 8 : 45;
        } else {
            this.weapon = { name: "Magic", type: "magic" };
            this.dmg = 60 + (this.lv * 30);
            this.range = (this.id === 'sans') ? 350 : 200;
            this.rate = (this.id === 'sans') ? 3 : 50;
        }
        if(this.variant === 'chara') this.dmg *= 5;
    }
    update() {
        if(this.stun > 0) { this.stun--; return; }
        if(this.cd > 0) this.cd--;

        if(this.id === 'void' && this.variant !== 'base' && this.cd <= 0) {
            this.spawnPuppet(); this.cd = 90;
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
            hp: 25 + this.lv * 10, color: (this.variant==='error')?'#00f':'#fff', dmg: 5 + this.lv
        });
    }
    attack(t) {
        let beamColor = this.id === 'sans' ? '#0cf' : (this.id === 'undyne' ? '#0ff' : '#fff');
        if(this.id === 'frisk' && this.variant === 'chara') beamColor = '#f00';

        bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:6, color:beamColor, wide: (this.weapon.type==='god'?10:3)});
        t.hp -= this.dmg;
        
        // Justice Soul Stun
        if(t.perk === 'yellow' && Math.random() < 0.15) this.stun = 90;
    }
    draw() {
        ctx.fillStyle = this.stun > 0 ? '#333' : (this.variant==='chara'?'#f00':'#fff');
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.fillStyle = '#000'; ctx.font = 'bold 12px Courier';
        ctx.fillText("LV"+this.lv, this.x-14, this.y+5);
    }
}

class Enemy {
    constructor(data, wave) {
        this.data = data; this.pIdx = 0;
        this.x = path[0].x*TILE+TILE/2; this.y = path[0].y*TILE+TILE/2;
        this.maxHp = data.hp * (1 + wave * 0.5); this.hp = this.maxHp;
        this.speed = data.speed; this.perk = data.perk;
        this.timer = 0; this.revived = false;
    }
    update() {
        this.timer++;
        let curSpeed = this.speed;

        // Soul Perks
        if(this.perk === 'cyan' && this.timer % 140 === 0) this.wait = 30;
        if(this.wait > 0) { this.wait--; curSpeed = 0; }
        
        if(this.perk === 'orange' && this.timer % 100 > 85) curSpeed *= 2.5;

        if(this.perk === 'blue' && this.timer % 130 === 0) {
            this.x += (path[this.pIdx+1]?.x - path[this.pIdx]?.x) * TILE;
            this.y += (path[this.pIdx+1]?.y - path[this.pIdx]?.y) * TILE;
            this.pIdx++;
        }

        if(this.perk === 'green') {
            enemies.forEach(e => {
                if(e !== this && Math.hypot(e.x-this.x, e.y-this.y) < 70) e.hp = Math.min(e.maxHp, e.hp + 0.2);
            });
        }

        let target = path[this.pIdx];
        if(!target) return 'leak';
        let tx = target.x*TILE+TILE/2, ty = target.y*TILE+TILE/2;
        let d = Math.hypot(tx-this.x, ty-this.y);
        if(d < curSpeed) this.pIdx++;
        else { this.x += ((tx-this.x)/d)*curSpeed; this.y += ((ty-this.y)/d)*curSpeed; }

        if(this.hp <= 0) {
            if(this.perk === 'red' && !this.revived) {
                this.hp = this.maxHp; this.revived = true;
                vfx.push({x:this.x, y:this.y, txt:'REFUSE', life:40, color:'#f00'});
                return null;
            }
            return 'die';
        }
        return null;
    }
    draw() {
        ctx.fillStyle = this.data.color;
        ctx.beginPath(); ctx.arc(this.x, this.y, 14, 0, Math.PI*2); ctx.fill();
        ctx.strokeStyle = '#fff'; ctx.lineWidth = 2; ctx.stroke();
        ctx.fillStyle = 'red'; ctx.fillRect(this.x-15, this.y-24, 30, 4);
        ctx.fillStyle = '#0f0'; ctx.fillRect(this.x-15, this.y-24, 30*(this.hp/this.maxHp), 4);
    }
}

/** LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    
    // Draw Path
    ctx.fillStyle = '#111';
    path.forEach(p => ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE));

    towers.forEach(t => { t.update(); t.draw(); });

    // Puppets (Backwards Pathing)
    for(let i=puppets.length-1; i>=0; i--) {
        let p = puppets[i];
        let target = path[p.pIdx];
        if(!target) { puppets.splice(i,1); continue; }
        let tx = target.x*TILE+TILE/2, ty = target.y*TILE+TILE/2;
        let d = Math.hypot(tx-p.x, ty-p.y);
        if(d < 4) p.pIdx--; else { p.x += ((tx-p.x)/d)*4; p.y += ((ty-p.y)/d)*4; }
        ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, 6, 0, Math.PI*2); ctx.fill();
        enemies.forEach(e => { if(Math.hypot(e.x-p.x, e.y-p.y)<30) { e.hp -= p.dmg; p.hp -= 0.3; } });
        if(p.hp <= 0) puppets.splice(i,1);
    }

    // Enemies
    for(let i=enemies.length-1; i>=0; i--) {
        let res = enemies[i].update();
        if(res === 'leak') { 
            coreHP = 0; // INSTANT DEATH
            document.getElementById('hp-val').innerText = 0;
            alert("SUDDEN DEATH: A Soul breached the Core. Timeline Erased.");
            location.reload();
            return;
        }
        else if(res === 'die') { money += 60; 
            if(enemies[i].perk === 'glitch') document.getElementById('main-body').classList.remove('glitch-fx');
            enemies.splice(i,1); 
        }
        else enemies[i].draw();
    }

    // Visuals
    for(let i=bullets.length-1; i>=0; i--) {
        let b = bullets[i]; ctx.strokeStyle = b.color; ctx.lineWidth = b.wide;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    }
    vfx.forEach((v,i) => {
        ctx.fillStyle = v.color; ctx.font = 'bold 16px Courier'; ctx.fillText(v.txt, v.x-25, v.y);
        v.y--; v.life--; if(v.life <= 0) vfx.splice(i,1);
    });

    updateUI();
    if(coreHP > 0) requestAnimationFrame(loop);
}

/** INTERACTION **/
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
            activeSummon = null; document.querySelectorAll('.shop-card').forEach(x => x.classList.remove('active'));
        }
    } else {
        let t = towers.find(x => x.tx === mouse.tx && x.ty === mouse.ty);
        if(t) { selectedInst = t; showUpgrade(t); }
    }
};

function showUpgrade(t) {
    const p = document.getElementById('upgrade-panel'); p.style.display = 'block';
    t.updateStats();
    document.getElementById('up-name').innerText = t.name;
    document.getElementById('up-weapon').innerText = t.weapon.name;
    document.getElementById('up-stats').innerText = `LV: ${t.lv} | DMG: ${t.dmg} | RNG: ${t.range}`;
    
    const ev = document.getElementById('evo-container'); ev.innerHTML = "";
    if(t.lv >= 5 && t.variant === 'base') {
        const d = document.createElement('div'); d.style.display='flex'; d.style.gap='5px';
        if(t.id === 'sans') {
            d.appendChild(createEvoBtn("BAD TIME", () => t.variant='bad_time'));
            d.appendChild(createEvoBtn("LAST BREATH", () => { t.variant='lb'; t.dmg*=4; t.rate=15; }));
        }
        if(t.id === 'void') {
            d.appendChild(createEvoBtn("ERROR", () => t.variant='error'));
            d.appendChild(createEvoBtn("INK", () => t.variant='ink'));
        }
        ev.appendChild(d);
    }
    if(t.id === 'frisk' && t.lv >= 19 && t.variant === 'base') {
        const d = document.createElement('div'); d.style.display='flex'; d.style.gap='5px';
        d.appendChild(createEvoBtn("CHARA", () => { t.variant='chara'; t.updateStats(); }));
        ev.appendChild(d);
    }

    const upBtn = document.getElementById('btn-upgrade');
    const cost = 120 * t.lv;
    upBtn.innerText = `LEVEL UP ($${cost})`;
    upBtn.disabled = money < cost || t.lv >= 20;
    upBtn.onclick = () => { if(money>=cost) { money -= cost; t.lv++; showUpgrade(t); } };
}

function createEvoBtn(txt, cb) {
    const b = document.createElement('button'); b.className = 'btn'; 
    b.style = "font-size:0.7rem; padding:4px; margin-top:5px; border-color:var(--gold); color:var(--gold);";
    b.innerText = txt; b.onclick = () => { cb(); showUpgrade(selectedInst); }; return b;
}

document.getElementById('btn-next-wave').onclick = () => {
    if(waveActive) return;
    wave++; waveActive = true;
    let count = 4 + wave * 2;
    document.getElementById('diag-text').innerText = `* Wave ${wave} - One breach and it's over.`;
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            let pool = SOULS.slice(0, Math.min(wave, 6));
            if(wave % 10 === 0 && i === 0) pool = [SOULS[6]];
            let type = pool[Math.floor(Math.random()*pool.length)];
            if(type.perk === 'glitch') document.getElementById('main-body').classList.add('glitch-fx');
            enemies.push(new Enemy(type, wave));
            if(i === count-1) {
                let c = setInterval(() => {
                    if(enemies.length === 0) { waveActive = false; clearInterval(c); }
                }, 500);
            }
        }, i * 1100);
    }
};

function updateUI() {
    document.getElementById('hp-val').innerText = coreHP;
    document.getElementById('g-val').innerText = money;
    document.getElementById('w-val').innerText = wave;
}
</script>
</body>
</html>
