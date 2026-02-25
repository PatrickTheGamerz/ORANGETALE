<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: MIMICRY OF THE FALLEN</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --green: #22c55e; --blue: #3b82f6; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        
        #layout { display: grid; grid-template-columns: 850px 320px; grid-template-rows: 70px 450px 260px; gap: 10px; padding: 10px; border: 4px double white; position: relative; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #333; cursor: crosshair; }
        
        #sidebar { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; gap: 8px; background: #0a0a0a; }
        .tower-card { border: 2px solid white; padding: 10px; text-align: center; cursor: pointer; font-size: 0.8rem; }
        .tower-card.active { border-color: var(--gold); background: #1a1a1a; }

        #dashboard { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; background: #000; }
        
        /* TRINITY & MIMICRY UI */
        #soul-nexus { width: 450px; border-right: 2px solid white; padding-right: 15px; }
        .bar-container { width: 100%; height: 8px; background: #111; border: 1px solid #444; margin: 4px 0; overflow: hidden; }
        .bar-fill { height: 100%; transition: width 0.5s; }

        .btn-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 5px; margin-top: 10px; }
        .cmd-btn { background: #000; border: 2px solid #fff; color: #fff; padding: 10px; cursor: pointer; font-family: inherit; font-weight: bold; position: relative; }
        .cmd-btn:disabled { opacity: 0.2; border-color: #444; color: #444; cursor: not-allowed; }
        
        #dialogue { flex: 1; font-size: 1rem; position: relative; }
        
        /* CHARA'S GHOST CURSOR */
        #chara-cursor { position: fixed; width: 24px; height: 24px; border: 2px solid var(--red); pointer-events: none; z-index: 10000; display: none; background: rgba(255,0,0,0.2); }
        #chara-cursor::after { content: 'LV 20'; position: absolute; top: -15px; left: 0; font-size: 8px; color: var(--red); font-weight: bold; }

        .glitch-anim { animation: glitch 0.1s infinite; }
        @keyframes glitch { 0%{transform: translate(2px)} 50%{transform: translate(-2px)} }
    </style>
</head>
<body id="body-main">

    <div id="chara-cursor"></div>

    <div id="layout">
        <header>
            <div>DT: <span id="hp" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold" style="color:var(--gold)">600</span></div>
            <div>LV: <span id="lv-val">1</span></div>
            <button id="wave-btn" class="cmd-btn" style="width:100px; padding:2px">FIGHT</button>
        </header>

        <canvas id="canvas"></canvas>

        <div id="sidebar">
            <h4 style="text-align:center; margin:0; color:var(--red);">SOULLESS SHOP</h4>
            <div id="shop"></div>
            <div id="mimicry-status" style="font-size:0.6rem; color:var(--gold); text-align:center; margin-top:10px;">MIMICRY: RECORDING...</div>
        </div>

        <div id="dashboard">
            <div id="soul-nexus">
                <div style="display:flex; justify-content:space-between; font-size: 0.65rem;">
                    <span>PLAYER/FRISK</span><span id="p-inf">99%</span>
                </div>
                <div class="bar-container"><div id="p-bar" class="bar-fill" style="background:var(--blue); width:99%"></div></div>
                
                <div style="display:flex; justify-content:space-between; font-size: 0.65rem;">
                    <span style="color:var(--red)">CHARA AI</span><span id="c-inf">1%</span>
                </div>
                <div class="bar-container"><div id="c-bar" class="bar-fill" style="background:var(--red); width:1%"></div></div>

                <div class="btn-grid" id="button-field">
                    <button class="cmd-btn" id="b-fight" onclick="executeCmd('FIGHT')">FIGHT</button>
                    <button class="cmd-btn" id="b-act" onclick="executeCmd('ACT')">ACT</button>
                    <button class="cmd-btn" id="b-item" onclick="executeCmd('ITEM')">ITEM</button>
                    <button class="cmd-btn" id="b-spare" onclick="executeCmd('SPARE')">MERCY</button>
                </div>
            </div>
            <div id="dialogue">
                <span style="color:red">❤</span> <span id="log-text">* I'm watching how you handle things, partner.</span>
            </div>
        </div>
    </div>

<script>
/** ⚙️ SYSTEM STATE **/
let sys = {
    gold: 600, lv: 1, resetCount: 0,
    player: 99, chara: 1, 
    isSoulless: false,
    history: [], // Stores {tx, ty, id}
    charaThinking: false,
    charaTargetX: 0, charaTargetY: 0,
    mouse: { tx: 0, ty: 0 }
};

const TOWERS_DB = {
    frisk: { name: "Protagonist", cost: 200, col: "#ff00ff" },
    sans:  { name: "Sans", cost: 600, col: "#008cff" },
    papy:  { name: "Papyrus", cost: 150, col: "#fff" },
    undy:  { name: "Undyne", cost: 350, col: "#00ffff" },
    void:  { name: "Void", cost: 500, col: "#555" }
};

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
canvas.width = 850; canvas.height = 450;

let units = [], enemies = [], path = [], bullets = [];
let placement = null, activeUnit = null;

/** ⚔️ CORE LOGIC **/
function init() {
    const shop = document.getElementById('shop');
    Object.keys(TOWERS_DB).forEach(k => {
        const d = document.createElement('div');
        d.className = 'tower-card';
        d.innerHTML = `<b>${TOWERS_DB[k].name}</b><br>$${TOWERS_DB[k].cost}`;
        d.onclick = () => {
            if(sys.chara > 85) return; // Disable shop for player
            placement = { ...TOWERS_DB[k], id: k };
        };
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

/** 🛡️ GUARDIAN **/
class Guardian {
    constructor(id, tx, ty) {
        this.id = id; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = (id==='frisk') ? sys.lv : 1;
        this.cd = 0;
    }
    update() {
        if(this.cd > 0) this.cd--;
        let range = 150;
        this.target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < range);

        // Chara Auto-Attacking
        if(this.id === 'frisk' && sys.chara > 10 && this.target && this.cd <= 0) {
            if(Math.random()*100 < sys.chara) {
                this.attack(this.target, true);
                this.cd = 20;
            }
        } else if(this.target && this.cd <= 0) {
            this.attack(this.target);
            this.cd = 60;
        }
    }
    attack(t, isChara = false) {
        t.hp -= isChara ? (sys.lv * 150) : 50;
        bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, col:isChara?'#f00':'#fff'});
    }
    draw() {
        ctx.fillStyle = (this.id === 'frisk' && sys.chara > 50) ? "#f00" : TOWERS_DB[this.id].col;
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.fillStyle='#000'; ctx.font='bold 10px Arial'; ctx.fillText("LV"+this.lv, this.x-13, this.y+5);
    }
}

/** 🕹️ COMMANDS & TRINITY **/
function executeCmd(type) {
    if(!activeUnit || activeUnit.id !== 'frisk' || sys.chara > 90) return;

    let roll = Math.random() * 100;
    if(roll < sys.chara) {
        type = "FIGHT";
        logIt("* Chara: Why SPARE when you can LOVE?");
    }

    if(!activeUnit.target && type !== "ITEM") return;
    let t = activeUnit.target;

    if(type === 'FIGHT') {
        t.hp -= sys.lv * 100;
        sys.lv++;
        updateTrinity();
    } else if(type === 'SPARE') {
        if(t.hp < 100) { sys.gold += 300; enemies = enemies.filter(e => e !== t); }
    }
}

function updateTrinity() {
    if(!sys.isSoulless) {
        if(sys.lv >= 19) { sys.chara = 95; sys.player = 5; }
        else { sys.chara = Math.min(90, sys.lv * 5); sys.player = 100 - sys.chara; }
    } else {
        sys.chara = Math.min(99, 90 + (sys.lv * 0.5));
        sys.player = 100 - sys.chara;
    }

    if(sys.chara > 90 && !sys.isSoulless) {
        document.getElementById('dialogue').innerHTML = `<button class="cmd-btn glitch-anim" style="color:red; border-color:red; width:100%" onclick="soullessReset()">ERASE THE TIMELINE</button>`;
    }
}

function soullessReset() {
    sys.isSoulless = true;
    sys.lv = 1; sys.gold = 500;
    sys.chara = 90; sys.player = 10;
    units = [];
    document.getElementById('body-main').style.background = "#300";
    setTimeout(() => { document.getElementById('body-main').style.background = "#000"; }, 1000);
    logIt("* You reset. But I remember everything.");
}

/** 🤖 CHARA AI MIMICRY **/
function charaThink() {
    if(sys.chara < 40 || sys.charaThinking) return;
    
    // Check if she can afford anything
    let affordable = Object.keys(TOWERS_DB).filter(k => TOWERS_DB[k].cost <= sys.gold);
    if(affordable.length === 0) return;

    sys.charaThinking = true;
    document.getElementById('chara-cursor').style.display = 'block';

    // Learning Logic: Look at player history
    let targetMove;
    if(sys.history.length > 0 && Math.random() > (1 - sys.chara/100)) {
        // High level Chara copies player
        targetMove = sys.history[Math.floor(Math.random()*sys.history.length)];
    } else {
        // Low level Chara places randomly
        targetMove = { tx: Math.floor(Math.random()*17), ty: Math.floor(Math.random()*9), id: affordable[0] };
    }

    // Move red cursor
    let rect = canvas.getBoundingClientRect();
    sys.charaTargetX = rect.left + targetMove.tx * TILE + 25;
    sys.charaTargetY = rect.top + targetMove.ty * TILE + 25;

    setTimeout(() => {
        const cur = document.getElementById('chara-cursor');
        cur.style.left = sys.charaTargetX + 'px';
        cur.style.top = sys.charaTargetY + 'px';
        
        setTimeout(() => {
            // Place tower
            let occ = units.find(u => u.tx === targetMove.tx && u.ty === targetMove.ty);
            if(!occ && sys.gold >= TOWERS_DB[targetMove.id].cost) {
                sys.gold -= TOWERS_DB[targetMove.id].cost;
                units.push(new Guardian(targetMove.id, targetMove.tx, targetMove.ty));
                logIt(`* Chara placed a ${TOWERS_DB[targetMove.id].name}.`);
            }
            sys.charaThinking = false;
        }, 1000);
    }, 500);
}

/** 🔁 LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    path.forEach(p => { ctx.fillStyle = '#111'; ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE); });

    units.forEach(u => u.update());
    units.forEach(u => u.draw());

    for(let i=enemies.length-1; i>=0; i--) {
        let e = enemies[i];
        let target = path[e.pIdx];
        if(!target) { alert("CORE BREACHED. RESETTING..."); location.reload(); return; }
        let d = Math.hypot(target.x*TILE+25-e.x, target.y*TILE+25-e.y);
        if(d < 2) e.pIdx++; else { e.x += ((target.x*TILE+25-e.x)/d)*1.5; e.y += ((target.y*TILE+25-e.y)/d)*1.5; }
        if(e.hp <= 0) { sys.gold += 30; enemies.splice(i,1); }
        else { ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(e.x, e.y, 10, 0, Math.PI*2); ctx.fill(); }
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.col; ctx.lineWidth = 3;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    if(sys.chara > 10) charaThink();

    document.getElementById('gold').innerText = Math.floor(sys.gold);
    document.getElementById('lv-val').innerText = sys.lv;
    document.getElementById('p-inf').innerText = Math.floor(sys.player) + "%";
    document.getElementById('c-inf').innerText = Math.floor(sys.chara) + "%";
    document.getElementById('p-bar').style.width = sys.player + "%";
    document.getElementById('c-bar').style.width = sys.chara + "%";

    requestAnimationFrame(loop);
}

/** 🖱️ INPUT **/
canvas.onmousedown = () => {
    let u = units.find(x => x.tx === sys.mouse.tx && x.ty === sys.mouse.ty);
    if(u) {
        activeUnit = u;
        document.getElementById('soul-nexus').style.opacity = (u.id === 'frisk') ? "1" : "0.3";
    } else if(placement && sys.chara < 90) {
        if(sys.gold >= placement.cost) {
            sys.gold -= placement.cost;
            units.push(new Guardian(placement.id, sys.mouse.tx, sys.mouse.ty));
            // Record player history for Chara to mimic
            sys.history.push({ tx: sys.mouse.tx, ty: sys.mouse.ty, id: placement.id });
            placement = null;
        }
    }
};

canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    sys.mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    sys.mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

document.getElementById('wave-btn').onclick = () => {
    sys.wave++;
    for(let i=0; i<3+sys.wave; i++) {
        setTimeout(() => {
            enemies.push({ x: path[0].x*TILE+25, y: path[0].y*TILE+25, pIdx: 0, hp: 300*(1+sys.wave*0.7) });
        }, i * 1500);
    }
};

function logIt(t) { document.getElementById('log-text').innerText = t; }
init();
</script>
</body>
</html>
