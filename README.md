<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: MULTIVERSE BREACH - FINAL OVERHAUL</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --orange: #ffa500; --blue: #0000ff; --green: #00ff00; --yellow: #ffff00; --purple: #a020f0; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        
        #game-layout { display: grid; grid-template-columns: 850px 250px; grid-template-rows: 60px 450px 160px; gap: 10px; padding: 15px; border: 5px double white; background: #000; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #333; }
        
        #shop { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; gap: 8px; }
        .shop-card { border: 2px solid white; padding: 10px; text-align: center; cursor: pointer; transition: 0.1s; font-size: 0.8rem; }
        .shop-card:hover { background: #fff; color: #000; }
        .shop-card.active { border-color: var(--gold); color: var(--gold); background: #111; }

        #bottom-ui { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; }
        #up-panel { width: 350px; border-right: 2px solid white; padding-right: 15px; display: none; }
        #dialogue { flex: 1; font-size: 1.1rem; color: #eee; position: relative; }

        .btn { background: #000; border: 2px solid #fff; color: #fff; padding: 8px; cursor: pointer; font-family: inherit; width: 100%; text-transform: uppercase; margin-top: 5px; }
        .btn:hover:not(:disabled) { color: var(--gold); border-color: var(--gold); }
        .btn:disabled { opacity: 0.3; }

        .glitch { animation: glitchEffect 0.2s infinite; filter: invert(1) hue-rotate(90deg); }
        @keyframes glitchEffect { 0% { transform: translate(2px, -2px); } 50% { transform: translate(-2px, 2px); } 100% { transform: translate(0); } }
        
        #intro { position: fixed; inset: 0; background: #000; z-index: 9999; display: flex; flex-direction: column; align-items: center; justify-content: center; border: 10px double #fff; margin: 10px; }
    </style>
</head>
<body id="main-body">

    <div id="intro">
        <h1 style="font-size: 4rem; color: var(--red); margin: 0;">UTD</h1>
        <p>MULTIVERSE BREACH: INITIALIZE GUARDIANS</p>
        <div id="select-grid" style="display:grid; grid-template-columns: repeat(3, 1fr); gap:10px; margin: 20px;"></div>
        <button id="btn-start" class="btn" style="width: 250px;" disabled>ENTER THE VOID</button>
    </div>

    <div id="game-layout">
        <header>
            <div>CORE HP: <span id="hp-val" style="color:var(--red); font-weight:bold;">99</span></div>
            <div>GOLD: <span id="g-val" style="color:var(--gold);">600</span></div>
            <div>LV: <span id="lv-val">1</span></div>
            <button id="btn-wave" class="btn" style="width:120px; margin-top:0;">FIGHT</button>
        </header>

        <canvas id="gameCanvas"></canvas>

        <div id="shop">
            <h3 style="margin:0; text-align:center; color:var(--gold);">SUMMON</h3>
            <div id="shop-list"></div>
        </div>

        <div id="bottom-ui">
            <div id="up-panel">
                <div style="display:flex; justify-content:space-between; align-items:center;">
                    <b id="up-name">NAME</b>
                    <span id="up-weapon" style="color:var(--cyan); font-size:0.7rem;">Stick</span>
                </div>
                <div id="up-stats" style="font-size:0.8rem; margin:5px 0; color:#888;">LV: 1 | DMG: 10</div>
                <div id="evo-choices" style="display:flex; gap:5px; margin-bottom:5px;"></div>
                <button id="btn-upgrade" class="btn">UPGRADE / EXP</button>
            </div>
            <div id="dialogue">
                <span style="color:red">❤</span> <span id="diag-text">Choose your guardians wisely. The timeline is fragile.</span>
            </div>
        </div>
    </div>

<script>
/** ⚔ WEAPON & ENEMY LOGIC DATA
 */
const FRISK_ARSENAL = [
    { lv: 1,  name: "Stick",        type: "melee",  range: 60,  dmg: 35,  rate: 40 },
    { lv: 4,  name: "Toy Knife",    type: "melee",  range: 80,  dmg: 60,  rate: 35 },
    { lv: 7,  name: "Tough Glove",  type: "burst",  range: 120, dmg: 20,  rate: 10 },
    { lv: 10, name: "Notebook",     type: "utility",range: 160, dmg: 15,  rate: 20 },
    { lv: 13, name: "Burnt Pan",    type: "splash", range: 140, dmg: 80,  rate: 60 },
    { lv: 16, name: "Empty Gun",    type: "ranged", range: 350, dmg: 150, rate: 80 },
    { lv: 19, name: "True Knife",   type: "melee",  range: 100, dmg: 999, rate: 100 }
];

const GUARDIAN_DATA = [
    { id: 'frisk', name: 'Frisk', cost: 200, color: '#ff00ff' },
    { id: 'sans',  name: 'Sans',  cost: 500, color: '#008cff' },
    { id: 'void',  name: 'Void',  cost: 400, color: '#ffffff' },
    { id: 'undyne',name: 'Undyne',cost: 300, color: '#00ffff' },
    { id: 'asgore',name: 'Asgore',cost: 450, color: '#ffa500' },
    { id: 'papy',  name: 'Papyrus',cost: 150, color: '#ffffff' }
];

const SOUL_TYPES = [
    { id: 'pat', name: 'Patience Soul', hp: 80,  speed: 0.7, color: 'var(--cyan)',   perk: 'cyan' },
    { id: 'bra', name: 'Bravery Soul',  hp: 60,  speed: 1.5, color: 'var(--orange)', perk: 'orange' },
    { id: 'int', name: 'Integrity Soul',hp: 90,  speed: 1.1, color: 'var(--blue)',   perk: 'blue' },
    { id: 'per', name: 'Perseverance Soul', hp: 110, speed: 0.9, color: 'var(--purple)', perk: 'purple' },
    { id: 'kin', name: 'Kindness Soul', hp: 180, speed: 0.8, color: 'var(--green)',  perk: 'green' },
    { id: 'jus', name: 'Justice Soul',  hp: 100, speed: 1.2, color: 'var(--yellow)', perk: 'yellow' },
    { id: 'det', name: 'Determination', hp: 200, speed: 1.3, color: 'var(--red)',    perk: 'red' },
    { id: 'sans_ls', name: 'Sans Soul', hp: 1,   speed: 2.2, color: 'var(--blue)',   perk: 'dodge' },
    { id: 'undy_ls', name: 'Undyne Soul', hp: 500, speed: 1.0, color: 'var(--cyan)',   perk: 'undying' },
    { id: 'gaster', name: 'GASTER',     hp: 10000, speed: 0.5, color: '#000', perk: 'glitch' }
];

/** ENGINE SETUP **/
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
const COLS = 17, ROWS = 9;
canvas.width = 850; canvas.height = 450;

let money = 600, coreHP = 99, wave = 0;
let towers = [], enemies = [], path = [], bullets = [], puppets = [], vfx = [];
let loadout = [], activeSummon = null, selectedInst = null, waveActive = false;
let mouse = { tx: 0, ty: 0 };

// Menu Select Logic
const selGrid = document.getElementById('select-grid');
GUARDIAN_DATA.forEach(g => {
    const d = document.createElement('div');
    d.className = 'shop-card';
    d.innerHTML = `<b>${g.name}</b><br>$${g.cost}`;
    d.onclick = () => {
        if(loadout.includes(g.id)) {
            loadout = loadout.filter(i => i !== g.id);
            d.style.borderColor = 'white';
        } else if(loadout.length < 4) {
            loadout.push(g.id);
            d.style.borderColor = 'var(--gold)';
        }
        document.getElementById('btn-start').disabled = loadout.length !== 4;
    };
    selGrid.appendChild(d);
});

document.getElementById('btn-start').onclick = () => {
    document.getElementById('intro').style.display = 'none';
    const list = document.getElementById('shop-list');
    loadout.forEach(id => {
        const g = GUARDIAN_DATA.find(x => x.id === id);
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
        if(r < 0.25 && y > 1) y--; else if(r < 0.5 && y < ROWS-2) y++; else x++;
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
            const w = [...FRISK_ARSENAL].reverse().find(wi => this.lv >= wi.lv);
            this.weapon = w;
            this.dmg = w.dmg; this.range = w.range; this.rate = w.rate;
        } else {
            this.weapon = { name: "Magic", type: "magic" };
            this.dmg = 30 + (this.lv * 15);
            this.range = (this.id === 'sans') ? 350 : 200;
            this.rate = (this.id === 'sans') ? 5 : 50;
        }
    }
    update() {
        if(this.stun > 0) { this.stun--; return; }
        if(this.cd > 0) this.cd--;

        if(this.id === 'void' && this.variant !== 'base' && this.cd <= 0) {
            this.spawnPuppet(); this.cd = 100;
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
            hp: 20 + this.lv, color: (this.variant==='error')?'#00f':'#fff', dmg: 2 + this.lv
        });
    }
    attack(t) {
        // Dodge logic for Sans Lost Soul
        if(t.perk === 'dodge' && t.stamina > 0 && Math.random() < 0.6) {
            t.stamina--; 
            vfx.push({x:t.x, y:t.y-20, txt:'MISS', life:30, color:'#fff'});
            return;
        }

        if(this.weapon.type === 'splash') {
            enemies.forEach(e => {
                if(Math.hypot(e.x-t.x, e.y-t.y) < 60) e.hp -= this.dmg;
            });
            bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, color:'#f00', wide:8});
        } else {
            bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, color:this.id==='sans'?'#0cf':'#fff', wide:2});
            t.hp -= this.dmg;
        }
    }
    draw() {
        ctx.fillStyle = this.stun > 0 ? '#444' : (this.variant==='chara'?'#f00':'#fff');
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.fillStyle = '#000'; ctx.font = 'bold 10px Courier';
        ctx.fillText("LV"+this.lv, this.x-12, this.y+5);
    }
}

class Enemy {
    constructor(data, wave) {
        this.data = data; this.pIdx = 0;
        this.x = path[0].x*TILE+TILE/2; this.y = path[0].y*TILE+TILE/2;
        this.maxHp = data.hp * (1 + wave * 0.45); this.hp = this.maxHp;
        this.speed = data.speed; this.perk = data.perk;
        this.stamina = (data.perk === 'dodge') ? 8 : 0;
        this.revived = false;
        this.timer = 0;
    }
    update() {
        this.timer++;
        let curSpeed = this.speed;

        // Perk Logic
        if(this.perk === 'cyan' && this.timer % 150 === 0) this.wait = 40;
        if(this.wait > 0) { this.wait--; curSpeed = 0; }
        
        if(this.perk === 'orange' && this.timer % 100 > 80) curSpeed *= 2.5;

        if(this.perk === 'blue' && this.timer % 120 === 0) {
            this.x += (path[this.pIdx+1]?.x - path[this.pIdx]?.x) * TILE;
            this.y += (path[this.pIdx+1]?.y - path[this.pIdx]?.y) * TILE;
            this.pIdx++;
        }

        if(this.perk === 'green') {
            enemies.forEach(e => {
                if(e !== this && Math.hypot(e.x-this.x, e.y-this.y) < 80) e.hp = Math.min(e.maxHp, e.hp + 0.15);
            });
        }

        if(this.perk === 'yellow' && this.timer % 200 === 0) {
            let t = towers[Math.floor(Math.random()*towers.length)];
            if(t) t.stun = 80;
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
            if(this.perk === 'undying' && !this.revived) {
                this.hp = this.maxHp*2; this.revived = true; this.speed *= 1.4;
                vfx.push({x:this.x, y:this.y, txt:'UNDYING', life:60, color:'#0ff'});
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
        if(d < 4) p.pIdx--; else { p.x += ((tx-p.x)/d)*4; p.y += ((ty-p.y)/d)*4; }
        ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, 6, 0, Math.PI*2); ctx.fill();
        enemies.forEach(e => { if(Math.hypot(e.x-p.x, e.y-p.y)<25) { e.hp -= p.dmg; p.hp -= 0.5; } });
        if(p.hp <= 0) puppets.splice(i,1);
    }

    // Enemies & Core Damage Logic
    for(let i=enemies.length-1; i>=0; i--) {
        let res = enemies[i].update();
        if(res === 'leak') { 
            let dmg = Math.ceil(enemies[i].hp);
            coreHP -= dmg;
            vfx.push({x:enemies[i].x, y:enemies[i].y, txt:`-${dmg}`, life:40, color:'#f00'});
            enemies.splice(i,1); 
        }
        else if(res === 'die') { money += 50; 
            if(enemies[i].perk === 'glitch') document.getElementById('main-body').classList.remove('glitch');
            enemies.splice(i,1); 
        }
        else enemies[i].draw();
    }

    // VFX
    for(let i=bullets.length-1; i>=0; i--) {
        let b = bullets[i]; ctx.strokeStyle = b.color; ctx.lineWidth = b.wide;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    }
    vfx.forEach((v,i) => {
        ctx.fillStyle = v.color; ctx.font = 'bold 16px Courier'; ctx.fillText(v.txt, v.x-20, v.y);
        v.y--; v.life--; if(v.life <= 0) vfx.splice(i,1);
    });

    updateUI();
    if(coreHP > 0) requestAnimationFrame(loop);
    else { alert("BUT NOBODY CAME. (Timeline Erased)"); location.reload(); }
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
            activeSummon = null; document.querySelectorAll('.shop-card').forEach(x=>x.classList.remove('active'));
        }
    } else {
        let t = towers.find(x=>x.tx===mouse.tx && x.ty===mouse.ty);
        if(t) { selectedInst = t; showUpgrade(t); }
    }
};

function showUpgrade(t) {
    const p = document.getElementById('up-panel'); p.style.display = 'block';
    t.updateStats();
    document.getElementById('up-name').innerText = t.name;
    document.getElementById('up-weapon').innerText = t.weapon.name;
    document.getElementById('up-stats').innerText = `LV: ${t.lv} | DMG: ${Math.floor(t.dmg)} | RNG: ${t.range}`;
    
    const ev = document.getElementById('evo-choices'); ev.innerHTML = "";
    if(t.lv >= 5 && t.variant === 'base') {
        if(t.id === 'sans') {
            ev.appendChild(createEvoBtn("BAD TIME", () => t.variant='bad_time'));
            ev.appendChild(createEvoBtn("DUST", () => { t.variant='dust'; t.dmg*=3; }));
        }
        if(t.id === 'void') {
            ev.appendChild(createEvoBtn("ERROR", () => t.variant='error'));
            ev.appendChild(createEvoBtn("INK", () => t.variant='ink'));
        }
    }
    if(t.id === 'frisk' && t.lv >= 20 && t.variant === 'base') {
        ev.appendChild(createEvoBtn("CHARA", () => { t.variant='chara'; t.dmg=999; }));
        ev.appendChild(createEvoBtn("PACIFIST", () => { t.variant='pacifist'; t.dmg=0; t.range=600; }));
    }

    const upBtn = document.getElementById('btn-upgrade');
    const cost = 100 * t.lv;
    upBtn.innerText = `LEVEL UP ($${cost})`;
    upBtn.disabled = money < cost || t.lv >= 20;
    upBtn.onclick = () => { if(money>=cost) { money -= cost; t.lv++; showUpgrade(t); } };
}

function createEvoBtn(txt, cb) {
    const b = document.createElement('button'); b.className = 'choice-btn'; 
    b.style = "background:black; color:var(--gold); border:1px solid var(--gold); padding:4px; font-family:inherit; cursor:pointer;";
    b.innerText = txt; b.onclick = () => { cb(); showUpgrade(selectedInst); }; return b;
}

document.getElementById('btn-wave').onclick = () => {
    if(waveActive) return;
    wave++; waveActive = true;
    let count = 4 + wave * 2;
    document.getElementById('diag-text').innerText = `* Wave ${wave} - Don't let their Souls escape.`;
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            let pool = SOUL_TYPES.slice(0, Math.min(wave, 7));
            if(wave % 10 === 0 && i === 0) pool = [SOUL_TYPES[9]];
            let type = pool[Math.floor(Math.random()*pool.length)];
            if(type.perk === 'glitch') document.getElementById('main-body').classList.add('glitch');
            enemies.push(new Enemy(type, wave));
            if(i === count-1) {
                let c = setInterval(() => {
                    if(enemies.length === 0) { waveActive = false; clearInterval(c); }
                }, 500);
            }
        }, i * 1000);
    }
};

function updateUI() {
    document.getElementById('hp-val').innerText = coreHP;
    document.getElementById('g-val').innerText = money;
    document.getElementById('lv-val').innerText = wave;
}
</script>
</body>
</html>
