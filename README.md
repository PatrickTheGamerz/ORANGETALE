<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: TRUE MULTIVERSE OVERHAUL</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { 
            --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --orange: #ffa500; 
            --blue: #0000ff; --green: #00ff00; --yellow: #ffff00; --purple: #a020f0; 
        }
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        
        #game-ui { display: grid; grid-template-columns: 850px 300px; grid-template-rows: 60px 450px 180px; gap: 10px; padding: 10px; border: 4px double white; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; font-size: 1.2rem; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #333; }
        
        #shop { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 15px; display: flex; flex-direction: column; gap: 12px; background: #0a0a0a; }
        .tower-btn { border: 2px solid white; padding: 10px; text-align: center; cursor: pointer; transition: 0.2s; position: relative; }
        .tower-btn:hover { background: #fff; color: #000; }
        .tower-btn.active { border-color: var(--gold); box-shadow: 0 0 10px var(--gold); color: var(--gold); }

        #footer { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; }
        #upgrade-box { width: 350px; border-right: 2px solid white; padding-right: 15px; }
        #console { flex: 1; font-size: 1.1rem; color: #fff; overflow-y: auto; }

        .btn { background: #000; border: 2px solid #fff; color: #fff; padding: 8px; cursor: pointer; font-family: inherit; width: 100%; margin-top: 5px; font-weight: bold; }
        .btn:hover:not(:disabled) { background: #fff; color: #000; }
        
        .soul-icon { width: 20px; height: 20px; display: inline-block; clip-path: path('M10,3.2C8.3,0.8,4,0.8,2.2,3.2c-1.1,1.5-1.1,3.6,0.1,5.1l7.7,8.5l7.7-8.5c1.2-1.5,1.2-3.6,0.1-5.1C16,0.8,11.7,0.8,10,3.2z'); }

        #intro { position: fixed; inset: 0; background: #000; z-index: 9999; display: flex; flex-direction: column; align-items: center; justify-content: center; border: 15px double white; margin: 10px; }
        .glitch { animation: glitch 0.1s infinite; filter: invert(1); }
        @keyframes glitch { 0% { transform: translate(2px); } 50% { transform: translate(-2px); } }
    </style>
</head>
<body id="b">

    <div id="intro">
        <h1 style="font-size: 5rem; color: var(--red); margin: 0;">UTD: REVAMPED</h1>
        <p>SELECT YOUR 4 GUARDIANS</p>
        <div id="setup-grid" style="display:grid; grid-template-columns: repeat(3, 1fr); gap:15px; margin: 30px;"></div>
        <button id="start-btn" class="btn" style="width: 250px; font-size: 1.5rem;" disabled>START</button>
    </div>

    <div id="game-ui">
        <header>
            <div>DT: <span id="hp" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold" style="color:var(--gold)">600</span></div>
            <div>WAVE: <span id="wave">0</span></div>
            <button id="wave-start" class="btn" style="width:100px; margin:0">FIGHT</button>
        </header>

        <canvas id="canvas"></canvas>

        <div id="shop">
            <div id="shop-items"></div>
        </div>

        <div id="footer">
            <div id="upgrade-box" style="display:none">
                <b id="up-name">TOWER</b> <span id="up-lv" style="float:right">LV 1</span>
                <p id="up-desc" style="font-size:0.8rem; height:40px; margin:5px 0;"></p>
                <div id="evo-btns"></div>
                <button id="up-go" class="btn">UPGRADE ($100)</button>
            </div>
            <div id="console">
                <span style="color:red">❤</span> <span id="log">The timeline is resetting...</span>
            </div>
        </div>
    </div>

<script>
/** * TOWER & WEAPON DATA 
 */
const T_DATA = {
    frisk: { name: "Frisk", cost: 200, color: "#ff00ff", weapons: [
        { lv: 1,  n: "Stick", dmg: 350, rng: 70,  type: "stun" },
        { lv: 5,  n: "Toy Knife", dmg: 600, rng: 90, type: "melee" },
        { lv: 10, n: "Notebook", dmg: 100, rng: 150, type: "inv" },
        { lv: 15, n: "Empty Gun", dmg: 1200, rng: 300, type: "ranged" },
        { lv: 20, n: "True Knife", dmg: 9999, rng: 120, type: "god" }
    ]},
    sans: { name: "Sans", cost: 500, color: "#008cff", desc: "Applies KR (Poison). Blasters pierce line." },
    undyne: { name: "Undyne", cost: 300, color: "#00ffff", desc: "Spears target highest HP enemy." },
    papy: { name: "Papyrus", cost: 150, color: "#ffffff", desc: "Blue Bones slow enemies by 60%." },
    asgore: { name: "Asgore", cost: 450, color: "#ffa500", desc: "Orange/Blue Trident AOE." },
    void: { name: "Void", cost: 400, color: "#fff", desc: "LV 1 choice: Error or Ink Sans." }
};

const SOULS = [
    { n: 'Patience',  hp: 100, spd: 0.8, c: 'var(--cyan)',   p: 'cyan' },
    { n: 'Bravery',   hp: 80,  spd: 2.0, c: 'var(--orange)', p: 'orange' },
    { n: 'Integrity', hp: 120, spd: 1.2, c: 'var(--blue)',   p: 'blue' },
    { n: 'Kindness',  hp: 250, spd: 0.7, c: 'var(--green)',  p: 'green' },
    { n: 'Justice',   hp: 150, spd: 1.4, c: 'var(--yellow)', p: 'yellow' },
    { n: 'Sans Soul', hp: 1,   spd: 2.5, c: 'var(--blue)',   p: 'dodge' },
    { n: 'GASTER',    hp: 15000,spd: 0.5, c: '#000', p: 'glitch' }
];

/** ENGINE **/
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
canvas.width = 850; canvas.height = 450;

let gold = 600, hp = 999, wave = 0;
let towers = [], enemies = [], path = [], bullets = [], effects = [];
let loadout = [], active = null, selection = null, waveOn = false;
let mouse = { tx: 0, ty: 0 };

// Setup Menu
const setup = document.getElementById('setup-grid');
Object.keys(T_DATA).forEach(k => {
    const div = document.createElement('div');
    div.className = 'tower-btn';
    div.innerHTML = `<b>${T_DATA[k].name}</b><br>$${T_DATA[k].cost}`;
    div.onclick = () => {
        if(loadout.includes(k)) loadout = loadout.filter(i => i!==k);
        else if(loadout.length < 4) loadout.push(k);
        div.classList.toggle('active', loadout.includes(k));
        document.getElementById('start-btn').disabled = loadout.length < 4;
    };
    setup.appendChild(div);
});

document.getElementById('start-btn').onclick = () => {
    document.getElementById('intro').style.display = 'none';
    const shopList = document.getElementById('shop-items');
    loadout.forEach(k => {
        const btn = document.createElement('div');
        btn.className = 'tower-btn';
        btn.innerHTML = `<b>${T_DATA[k].name}</b><br>$${T_DATA[k].cost}`;
        btn.onclick = () => {
            active = k;
            document.querySelectorAll('.tower-btn').forEach(b => b.classList.remove('active'));
            btn.classList.add('active');
        };
        shopList.appendChild(btn);
    });
    genMap();
    requestAnimationFrame(loop);
};

function genMap() {
    path = []; let x = 0, y = 4;
    while(x < 17) {
        path.push({x, y});
        if(Math.random() < 0.3 && y > 1) y--;
        else if(Math.random() < 0.3 && y < 7) y++;
        else x++;
    }
}

/** TOWER CLASS **/
class Tower {
    constructor(k, tx, ty) {
        this.k = k; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = 1; this.cd = 0; this.stun = 0;
        this.variant = 'base';
        this.refresh();
    }
    refresh() {
        const d = T_DATA[this.k];
        if(this.k === 'frisk') {
            const w = [...d.weapons].reverse().find(wi => this.lv >= wi.lv);
            this.dmg = w.dmg; this.rng = w.rng; this.rate = w.type === 'rapid' ? 10 : 40;
            this.wName = w.n;
        } else {
            this.dmg = 50 + (this.lv * 20);
            this.rng = (this.k === 'sans' || this.k === 'undyne') ? 300 : 150;
            this.rate = (this.k === 'sans') ? 8 : 45;
            this.wName = "Magic";
        }
        if(this.variant === 'chara') this.dmg *= 5;
    }
    update() {
        if(this.stun > 0) { this.stun--; return; }
        if(this.cd > 0) this.cd--;
        let target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < this.rng);
        if(target && this.cd <= 0) {
            this.attack(target);
            this.cd = this.rate;
        }
    }
    attack(t) {
        if(t.p === 'dodge' && Math.random() < 0.7) {
            effects.push({x:t.x, y:t.y-20, txt:'MISS', life:30, c:'#fff'});
            return;
        }
        // Logic Revamp: Cyan patience
        if(t.p === 'cyan' && !t.waiting) return; 

        let c = this.k === 'sans' ? '#0cf' : (this.k==='frisk' && this.variant==='chara'?'#f00':'#fff');
        bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, c:c, w: (this.variant==='chara'?8:2)});
        
        t.hp -= this.dmg;
        if(this.k === 'sans') t.kr = 100; // Karma
        if(this.k === 'papy') t.blue = 60; // Blue slow

        if(t.p === 'yellow' && Math.random() < 0.1) this.stun = 100; // Justice stun
    }
    draw() {
        ctx.fillStyle = this.stun > 0 ? '#333' : (this.variant==='chara'?'#f00':T_DATA[this.k].color);
        ctx.beginPath();
        ctx.roundRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10, 5);
        ctx.fill();
        ctx.fillStyle = '#000'; ctx.font = 'bold 10px Courier';
        ctx.fillText("LV"+this.lv, this.x-12, this.y+5);
    }
}

/** ENEMY CLASS **/
class Enemy {
    constructor(d, wave) {
        this.d = d; this.pIdx = 0;
        this.x = path[0].x*TILE+TILE/2; this.y = path[0].y*TILE+TILE/2;
        this.hp = d.hp * (1 + wave * 0.6); this.max = this.hp;
        this.spd = d.spd; this.p = d.p;
        this.timer = 0; this.kr = 0; this.blue = 0; this.shield = (d.p === 'green' ? 5 : 0);
    }
    update() {
        this.timer++;
        if(this.kr > 0) { this.hp -= 0.5; this.kr--; }
        let s = this.blue > 0 ? this.spd * 0.4 : this.spd;
        if(this.blue > 0) this.blue--;

        // 1:1 Lore Ability: Patience
        if(this.p === 'cyan') {
            this.waiting = (this.timer % 150 > 100);
            if(this.waiting) s = 0;
        }
        // Bravery: Fast if not slowed
        if(this.p === 'orange' && this.blue === 0) s *= 1.8;

        // Integrity: Jump
        if(this.p === 'blue' && this.timer % 120 === 0) {
            this.pIdx = Math.min(path.length-1, this.pIdx + 2);
            this.x = path[this.pIdx].x*TILE+TILE/2; this.y = path[pIdx].y*TILE+TILE/2;
        }

        let target = path[this.pIdx];
        if(!target) return 'leak';
        let tx = target.x*TILE+TILE/2, ty = target.y*TILE+TILE/2;
        let dist = Math.hypot(tx-this.x, ty-this.y);
        if(dist < s) this.pIdx++;
        else { this.x += ((tx-this.x)/dist)*s; this.y += ((ty-this.y)/dist)*s; }

        return this.hp <= 0 ? 'die' : null;
    }
    draw() {
        ctx.fillStyle = this.d.c;
        ctx.beginPath();
        // Soul Shape
        ctx.moveTo(this.x, this.y + 10);
        ctx.bezierCurveTo(this.x - 15, this.y - 15, this.x - 20, this.y + 5, this.x, this.y + 15);
        ctx.bezierCurveTo(this.x + 20, this.y + 5, this.x + 15, this.y - 15, this.x, this.y + 10);
        ctx.fill();
        ctx.strokeStyle = '#fff'; ctx.lineWidth = 1; ctx.stroke();
        
        // HP
        ctx.fillStyle = 'red'; ctx.fillRect(this.x-15, this.y-25, 30, 4);
        ctx.fillStyle = '#0f0'; ctx.fillRect(this.x-15, this.y-25, 30*(this.hp/this.max), 4);
    }
}

/** LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    path.forEach(p => { ctx.fillStyle = '#111'; ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE); });

    towers.forEach(t => { t.update(); t.draw(); });

    for(let i=enemies.length-1; i>=0; i--) {
        let res = enemies[i].update();
        if(res === 'leak') { alert("TIMELINE ERASED."); location.reload(); return; }
        if(res === 'die') { gold += 50; enemies.splice(i,1); if(enemies.length === 0 && waveOn) waveOn = false; }
        else enemies[i].draw();
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.c; ctx.lineWidth = b.w;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    effects.forEach((e,i) => {
        ctx.fillStyle = e.c; ctx.font = 'bold 16px Courier'; ctx.fillText(e.txt, e.x-10, e.y);
        e.y--; e.life--; if(e.life <= 0) effects.splice(i,1);
    });

    if(active) {
        ctx.globalAlpha = 0.3; ctx.fillStyle = T_DATA[active].color;
        ctx.fillRect(mouse.tx*TILE, mouse.ty*TILE, TILE, TILE);
        ctx.globalAlpha = 1;
    }

    document.getElementById('hp').innerText = hp;
    document.getElementById('gold').innerText = gold;
    document.getElementById('wave').innerText = wave;
    requestAnimationFrame(loop);
}

/** UI **/
canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

canvas.onmousedown = () => {
    if(active) {
        let occ = towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty) || !path.every(p => p.x!==mouse.tx || p.y!==mouse.ty);
        if(!occ && gold >= T_DATA[active].cost) {
            gold -= T_DATA[active].cost; towers.push(new Tower(active, mouse.tx, mouse.ty));
            active = null; document.querySelectorAll('.tower-btn').forEach(b => b.classList.remove('active'));
        }
    } else {
        let t = towers.find(x => x.tx === mouse.tx && x.ty === mouse.ty);
        if(t) { selection = t; showUp(t); }
        else { selection = null; document.getElementById('upgrade-box').style.display='none'; }
    }
};

function showUp(t) {
    document.getElementById('upgrade-box').style.display='block';
    document.getElementById('up-name').innerText = T_DATA[t.k].name;
    document.getElementById('up-lv').innerText = "LV " + t.lv;
    document.getElementById('up-desc').innerText = `Weapon: ${t.wName}\nDMG: ${Math.floor(t.dmg)} | RNG: ${t.rng}`;
    
    const evo = document.getElementById('evo-btns'); evo.innerHTML = "";
    if(t.k === 'frisk' && t.lv >= 19 && t.variant === 'base') {
        const b = document.createElement('button'); b.className='btn'; b.innerText="GENOCIDE (CHARA)";
        b.onclick = () => { t.variant = 'chara'; t.refresh(); showUp(t); };
        evo.appendChild(b);
    }

    const go = document.getElementById('up-go');
    const cost = 100 * t.lv;
    go.innerText = `UPGRADE ($${cost})`;
    go.disabled = gold < cost || t.lv >= 20;
    go.onclick = () => { if(gold>=cost) { gold -= cost; t.lv++; t.refresh(); showUp(t); } };
}

document.getElementById('wave-start').onclick = () => {
    if(waveOn) return; wave++; waveOn = true;
    let count = 5 + wave * 2;
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            let p = SOULS.slice(0, Math.min(wave, 6));
            if(wave % 10 === 0 && i === 0) p = [SOULS[6]];
            enemies.push(new Enemy(p[Math.floor(Math.random()*p.length)], wave));
        }, i * 800);
    }
};
</script>
</body>
</html>
