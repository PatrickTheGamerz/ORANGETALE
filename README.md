<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: PARADOX PROTOCOL</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --green: #22c55e; --blue: #3b82f6; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        #layout { display: grid; grid-template-columns: 850px 320px; grid-template-rows: 70px 450px 240px; gap: 10px; padding: 10px; border: 4px double white; position: relative; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #333; cursor: crosshair; }
        
        #sidebar { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; gap: 8px; background: #0a0a0a; }
        .tower-card { border: 2px solid white; padding: 10px; text-align: center; cursor: pointer; font-size: 0.8rem; }
        .tower-card.active { border-color: var(--gold); background: #1a1a1a; }

        #dashboard { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; background: #000; }
        #control-panel { width: 450px; border-right: 2px solid white; padding-right: 15px; }
        
        .btn-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 5px; margin-top: 10px; }
        .cmd-btn { background: #000; border: 2px solid #fff; color: #fff; padding: 10px; cursor: pointer; font-family: inherit; font-weight: bold; }
        .cmd-btn:hover { background: #fff; color: #000; }
        .cmd-btn:disabled { opacity: 0.2; cursor: not-allowed; }

        #log-box { flex: 1; font-size: 1.1rem; line-height: 1.3; overflow: hidden; }
        
        /* HP BARS */
        .bar-wrap { width: 100%; height: 8px; background: #222; border: 1px solid #fff; margin-top: 5px; }
        .bar-fill { height: 100%; transition: width 0.3s; }
        
        .glitch { animation: gl 0.1s infinite; }
        @keyframes gl { 0%{transform:translate(1px)} 50%{transform:translate(-1px)} }
    </style>
</head>
<body id="b">

    <div id="layout">
        <header>
            <div>DT: <span id="hp-val" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold-val" style="color:var(--gold)">600</span></div>
            <div>LV: <span id="lv-val">1</span></div>
            <button id="wave-btn" class="cmd-btn" style="width:100px; padding:2px">READY</button>
        </header>

        <canvas id="canvas"></canvas>

        <div id="sidebar">
            <h4 style="text-align:center; margin:0;">TIMELINE SHOP</h4>
            <div id="shop"></div>
        </div>

        <div id="dashboard">
            <div id="control-panel">
                <div style="display:flex; justify-content:space-between">
                    <b id="char-title">SELECT A UNIT</b>
                    <span id="influence-text" style="font-size:0.6rem">PLAYER: 100%</span>
                </div>
                <div class="bar-wrap"><div id="p-bar" class="bar-fill" style="background:var(--blue); width:100%"></div></div>
                <div class="bar-wrap"><div id="c-bar" class="bar-fill" style="background:var(--red); width:0%"></div></div>

                <div id="btn-container" class="btn-grid">
                    </div>
            </div>
            <div id="log-box">
                <span style="color:red">❤</span> <span id="log-text">* You feel the weight of your choices.</span>
            </div>
        </div>
    </div>

<script>
/** ⚙️ SYSTEM STATE **/
let world = {
    gold: 600, lv: 1, dt: 999,
    player: 100, frisk: 0, chara: 0,
    soulless: false, waveActive: false,
    selected: null, mouse: { tx:0, ty:0 }
};

const TOWERS_DATA = {
    frisk: { name: "Frisk", cost: 200, color: "#ff00ff" },
    sans:  { name: "Sans", cost: 600, color: "#008cff" },
    papy:  { name: "Papyrus", cost: 150, color: "#fff" },
    asgore:{ name: "Asgore", cost: 500, color: "#ffa500" }
};

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
canvas.width = 850; canvas.height = 450;

let units = [], enemies = [], path = [], bullets = [];
let placement = null;

/** ⚔️ CORE LOOP **/
function init() {
    const shop = document.getElementById('shop');
    Object.keys(TOWERS_DATA).forEach(k => {
        const d = document.createElement('div');
        d.className = 'tower-card';
        d.innerHTML = `<b>${TOWERS_DATA[k].name}</b><br>$${TOWERS_DATA[k].cost}`;
        d.onclick = () => { if(world.chara < 90) placement = { ...TOWERS_DATA[k], id: k }; };
        shop.appendChild(d);
    });
    genPath();
    loop();
}

function genPath() {
    path = []; let x = 0, y = 4;
    while(x < 17) {
        path.push({x, y});
        if(Math.random() < 0.2 && y > 1) y--; else if(Math.random() < 0.2 && y < 7) y++; else x++;
    }
}

class Guardian {
    constructor(id, tx, ty) {
        this.id = id; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = (id==='frisk')?world.lv:1;
        this.cd = 0; this.stamina = 100;
    }
    update() {
        if(this.cd > 0) this.cd--;
        let range = (this.id==='frisk')?140:350;
        this.target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < range);

        if(this.target && this.cd <= 0) {
            this.attack(this.target);
            this.cd = (this.id==='sans')?5:50;
        }
    }
    attack(t) {
        let isChara = (this.id==='frisk' && world.chara > 60);
        let dmg = (this.id==='frisk')?(this.lv*100):50;
        if(this.id==='sans') { t.kr = 100; dmg = 1; }
        if(isChara) dmg *= 5;
        t.hp -= dmg;
        bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, col:isChara?'#f00':'#fff'});
    }
    draw() {
        ctx.fillStyle = (this.id==='frisk' && world.chara > 50)?'#f00':TOWERS_DATA[this.id].color;
        if(world.selected === this) { ctx.shadowBlur = 10; ctx.shadowColor = 'gold'; }
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.shadowBlur = 0;
    }
}

/** 🕹️ COMMAND SYSTEM **/
function drawControls() {
    const container = document.getElementById('btn-container');
    container.innerHTML = "";
    if(!world.selected) return;

    const unit = world.selected;
    document.getElementById('char-title').innerText = unit.id.toUpperCase();

    let btns = [];
    if(unit.id === 'frisk') {
        btns = [
            { n: "FIGHT", f: () => cmdFrisk('FIGHT') },
            { n: "ACT", f: () => cmdFrisk('ACT') },
            { n: "SPARE", f: () => cmdFrisk('SPARE') },
            { n: "ITEM", f: () => cmdFrisk('ITEM') }
        ];
    } else if(unit.id === 'sans') {
        btns = [
            { n: "BLASTER", f: () => { if(unit.target) unit.target.hp -= 500; logIt("* Sans fired a blaster."); } },
            { n: "NAP", f: () => { unit.cd = 200; logIt("* Sans is taking a nap."); } }
        ];
    } else if(unit.id === 'papy') {
        btns = [
            { n: "BLUE ATTACK", f: () => { enemies.forEach(e => e.blue = 100); logIt("* The Great Papyrus made everyone BLUE!"); } },
            { n: "SPAGHETTI", f: () => { world.gold += 50; logIt("* Nyeh Heh Heh! Have some pasta."); } }
        ];
    }

    btns.forEach(b => {
        const btn = document.createElement('button');
        btn.className = 'cmd-btn';
        btn.innerText = b.n;
        btn.onclick = b.f;
        container.appendChild(btn);
    });
}

function cmdFrisk(type) {
    if(!world.selected.target && type !== 'ITEM') return;
    let t = world.selected.target;
    let roll = Math.random()*100;

    if(roll < world.chara) type = "FIGHT";

    if(type === 'FIGHT') {
        t.hp -= world.lv * 150;
        world.lv++;
        updateMeta();
    } else if(type === 'SPARE') {
        if(t.hp < 100) { world.gold += 400; enemies = enemies.filter(e => e !== t); }
    }
}

function updateMeta() {
    world.chara = Math.min(95, world.lv * 5);
    world.player = 100 - world.chara;
    document.getElementById('p-bar').style.width = world.player + "%";
    document.getElementById('c-bar').style.width = world.chara + "%";
    document.getElementById('influence-text').innerText = `PLAYER: ${Math.floor(world.player)}%`;
}

/** 🔁 LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    path.forEach(p => { ctx.fillStyle = '#111'; ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE); });

    units.forEach(u => u.update());
    units.forEach(u => u.draw());

    for(let i=enemies.length-1; i>=0; i--) {
        let e = enemies[i];
        let s = (e.blue > 0) ? 0.4 : 1.2;
        if(e.blue > 0) e.blue--;
        if(e.kr > 0) { e.hp -= 0.5; e.kr--; }

        let target = path[e.pIdx];
        if(!target) { location.reload(); return; }
        
        let d = Math.hypot(target.x*TILE+25-e.x, target.y*TILE+25-e.y);
        if(d < s) e.pIdx++; else { e.x += ((target.x*TILE+25-e.x)/d)*s; e.y += ((target.y*TILE+25-e.y)/d)*s; }

        if(e.hp <= 0) { world.gold += 30; enemies.splice(i,1); }
        else { ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(e.x, e.y, 10, 0, Math.PI*2); ctx.fill(); }
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.col; ctx.lineWidth = 3;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    // CHARA AUTO-START
    if(world.chara > 90 && !world.waveActive && Math.random() > 0.998) {
        document.getElementById('wave-btn').click();
    }

    updateHUD();
    requestAnimationFrame(loop);
}

/** 🖱️ INPUT **/
canvas.onmousedown = () => {
    let u = units.find(x => x.tx === world.mouse.tx && x.ty === world.mouse.ty);
    if(u) {
        world.selected = u;
        drawControls();
    } else if(placement) {
        if(world.gold >= placement.cost) {
            world.gold -= placement.cost;
            units.push(new Guardian(placement.id, world.mouse.tx, world.mouse.ty));
            placement = null;
        }
    } else {
        world.selected = null;
        drawControls();
    }
};

canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    world.mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    world.mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

document.getElementById('wave-btn').onclick = () => {
    world.waveActive = true;
    for(let i=0; i<5; i++) {
        setTimeout(() => {
            enemies.push({ x: path[0].x*TILE+25, y: path[0].y*TILE+25, pIdx: 0, hp: 400, kr: 0, blue: 0 });
        }, i * 1500);
    }
};

function logIt(t) { document.getElementById('log-text').innerText = t; }
function updateHUD() {
    document.getElementById('gold-val').innerText = Math.floor(world.gold);
    document.getElementById('lv-val').innerText = world.lv;
}
init();
</script>
</body>
</html>
