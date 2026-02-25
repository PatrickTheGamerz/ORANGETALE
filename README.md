<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: PARTNERSHIP PROTOCOL</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --green: #22c55e; --blue: #3b82f6; --white: #ffffff; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        #layout { display: grid; grid-template-columns: 850px 320px; grid-template-rows: 70px 450px 280px; gap: 10px; padding: 10px; border: 4px double white; position: relative; box-shadow: inset 0 0 100px rgba(255,0,0,0.1); }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #333; cursor: none; }
        
        #sidebar { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; gap: 8px; background: #0a0a0a; }
        .tower-card { border: 2px solid white; padding: 8px; text-align: center; cursor: pointer; font-size: 0.8rem; position: relative; }
        .tower-card.locked { opacity: 0.2; cursor: not-allowed; border-color: var(--red); }

        #dashboard { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; background: #000; }
        
        /* PARTNERSHIP METER */
        #morality-core { width: 450px; border-right: 2px solid white; padding-right: 15px; }
        .dispo-bar { width: 100%; height: 12px; background: #111; border: 1px solid #fff; margin: 5px 0; overflow: hidden; display: flex; }
        #pacifist-fill { height: 100%; background: var(--green); transition: width 0.5s; }
        #genocide-fill { height: 100%; background: var(--red); transition: width 0.5s; }

        .btn-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 5px; margin-top: 10px; }
        .cmd-btn { background: #000; border: 2px solid #fff; color: #fff; padding: 10px; cursor: pointer; font-family: inherit; font-weight: bold; font-size: 1rem; }
        .cmd-btn:hover:not(:disabled) { background: #fff; color: #000; }
        
        #dialogue { flex: 1; font-size: 1.1rem; border-left: 2px solid white; padding-left: 15px; position: relative; }
        
        /* META ELEMENTS */
        #soul-cursor { position: fixed; width: 20px; height: 20px; pointer-events: none; z-index: 10000; }
        .soul-heart { width: 100%; height: 100%; background: var(--red); clip-path: path('M10,30 A5,5 0,0,1 20,30 A5,5 0,0,1 30,30 L20,40 Z'); transform: scale(2) translateY(-15px); transition: background 0.3s; }

        .glitch { animation: gl 0.1s infinite; }
        @keyframes gl { 0%{transform:translate(2px)} 50%{transform:translate(-2px)} }
    </style>
</head>
<body id="b">

    <div id="soul-cursor"><div class="soul-heart" id="heart"></div></div>

    <div id="layout">
        <header>
            <div>DT: <span id="hp" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold" style="color:var(--gold)">600</span></div>
            <div>STATUS: <span id="status-tag">NEUTRAL</span></div>
            <button id="ready-btn" class="cmd-btn" style="width:120px; font-size:0.7rem;">READY?</button>
        </header>

        <canvas id="canvas"></canvas>

        <div id="sidebar">
            <h4 style="text-align:center; margin:0;">GUARDIANS</h4>
            <div id="shop"></div>
            <div id="chara-hint" style="font-size:0.6rem; color:var(--gold); text-align:center; margin-top:10px;">CHARA: "I'm watching."</div>
        </div>

        <div id="dashboard">
            <div id="morality-core">
                <div style="display:flex; justify-content:space-between; font-size:0.6rem;">
                    <span style="color:var(--green)">PACIFIST (FRISK)</span>
                    <span style="color:var(--red)">GENOCIDE (CHARA)</span>
                </div>
                <div class="dispo-bar">
                    <div id="pacifist-fill" style="width:50%"></div>
                    <div id="genocide-fill" style="width:50%"></div>
                </div>
                
                <div class="btn-grid">
                    <button class="cmd-btn" id="b-fight" onclick="action('FIGHT')">FIGHT</button>
                    <button class="cmd-btn" id="b-act" onclick="action('ACT')">ACT</button>
                    <button class="cmd-btn" id="b-item" onclick="action('ITEM')">ITEM</button>
                    <button class="cmd-btn" id="b-mercy" onclick="action('SPARE')">MERCY</button>
                </div>
                <div id="control-stats" style="font-size:0.6rem; margin-top:5px; text-align:center; color:#888;">PLAYER CONTROL: 95%</div>
            </div>
            <div id="dialogue">
                <span style="color:red">❤</span> <span id="log">* You feel a presence beside you.</span>
            </div>
        </div>
    </div>

<script>
/** ⚙️ SYSTEM STATE **/
let sys = {
    gold: 600, lv: 1, 
    pacifist: 50, genocide: 50,
    playerControl: 95,
    wave: 0,
    isCharaPartner: false,
    cursor: { x: 0, y: 0, tx: 0, ty: 0 }
};

const TOWERS_DB = {
    frisk: { name: "Frisk", cost: 200, col: "#ff00ff" },
    sans:  { name: "Sans", cost: 600, col: "#008cff" },
    papy:  { name: "Papyrus", cost: 150, col: "#fff" },
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
        d.id = 'card-' + k;
        d.innerHTML = `<b>${TOWERS_DB[k].name}</b><br>$${TOWERS_DB[k].cost}`;
        d.onclick = () => {
            if(sys.playerControl < 10) return;
            placement = { ...TOWERS_DB[k], id: k };
            document.querySelectorAll('.tower-card').forEach(c => c.style.borderColor = 'white');
            d.style.borderColor = 'var(--gold)';
        };
        shop.appendChild(d);
    });
    genPath();
    loop();
}

function genPath() {
    let x = 0, y = 4;
    while(x < 17) {
        path.push({x, y});
        if(Math.random() < 0.2 && y > 1) y--; else if(Math.random() < 0.2 && y < 7) y++; else x++;
    }
}

class Guardian {
    constructor(id, tx, ty) {
        this.id = id; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = (id==='frisk')?sys.lv:1;
        this.cd = 0;
    }
    update() {
        if(this.cd > 0) this.cd--;
        let range = (this.id === 'frisk') ? 140 : 350;
        this.target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < range);

        // AWARENESS: SANS JUDGMENT
        if(this.id === 'sans' && sys.genocide > 80) {
            // Sans refuses to attack for a killer
            this.target = null;
            if(Math.random() > 0.99) logIt("* Sans is staring at you with a blank expression.");
        }

        if(this.target && this.cd <= 0) {
            this.fire(this.target);
            this.cd = (this.id==='sans')?5:60;
        }
    }
    fire(t) {
        let dmg = (this.id==='frisk')?(this.lv*100):50;
        if(sys.isCharaPartner) dmg *= 2; // Chara helps optimize hits
        t.hp -= dmg;
        bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, col:sys.genocide > 70?'#f00':'#fff'});
    }
    draw() {
        ctx.fillStyle = (this.id==='frisk' && sys.genocide > 50)?'#f00':TOWERS_DB[this.id].col;
        if(activeUnit === this) { ctx.shadowBlur = 10; ctx.shadowColor = 'gold'; }
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.shadowBlur = 0;
    }
}

/** 🎭 PERSONALITY ACTIONS **/
function action(type) {
    if(!activeUnit || activeUnit.id !== 'frisk') return;
    
    let roll = Math.random() * 100;
    // Chara's Override based on persona
    if(sys.genocide > 80 && type === 'SPARE' && roll < 70) {
        logIt("* Chara: No. We are not done.");
        type = "FIGHT";
    }
    // Frisk's Override
    if(sys.pacifist > 80 && type === 'FIGHT' && roll < 70) {
        logIt("* Frisk: There is always another way.");
        type = "ACT";
    }

    if(!activeUnit.target && type !== 'ITEM') return;
    let t = activeUnit.target;

    if(type === 'FIGHT') {
        t.hp -= sys.lv * 200;
        sys.genocide = Math.min(100, sys.genocide + 5);
        sys.pacifist = 100 - sys.genocide;
        sys.lv++;
    } else if(type === 'SPARE') {
        if(t.hp < 100) {
            sys.gold += 400;
            enemies = enemies.filter(e => e !== t);
            sys.pacifist = Math.min(100, sys.pacifist + 5);
            sys.genocide = 100 - sys.pacifist;
            logIt("* SPARED. You feel Chara's confusion.");
        }
    } else if(type === 'ACT') {
        t.spd *= 0.5;
        logIt("* You Talked. Chara is observing the dialogue.");
        sys.pacifist = Math.min(100, sys.pacifist + 2);
        sys.genocide = 100 - sys.pacifist;
    }

    updatePersona();
}

function updatePersona() {
    const status = document.getElementById('status-tag');
    const hint = document.getElementById('chara-hint');
    const heart = document.getElementById('heart');

    if(sys.genocide > 80) {
        status.innerText = "GENOCIDE";
        status.style.color = "var(--red)";
        hint.innerText = 'CHARA: "A true partner. Let us continue."';
        sys.playerControl = Math.max(5, 100 - (sys.lv * 5));
        heart.style.background = "var(--red)";
    } else if(sys.pacifist > 80) {
        status.innerText = "TRUE PACIFIST";
        status.style.color = "var(--green)";
        hint.innerText = 'CHARA: "You are... strange. But I will help."';
        sys.isCharaPartner = true; // Cooperating Chara buffs damage without stealing control
        sys.playerControl = 95;
        heart.style.background = "var(--gold)";
    } else {
        status.innerText = "NEUTRAL";
        status.style.color = "var(--white)";
        hint.innerText = 'CHARA: "I am watching your choices."';
        heart.style.background = "var(--red)";
    }

    document.getElementById('pacifist-fill').style.width = sys.pacifist + "%";
    document.getElementById('genocide-fill').style.width = sys.genocide + "%";
    document.getElementById('control-stats').innerText = `PLAYER CONTROL: ${Math.floor(sys.playerControl)}%`;
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
        if(!target) { location.reload(); return; }
        let d = Math.hypot(target.x*TILE+25-e.x, target.y*TILE+25-e.y);
        if(d < 2) e.pIdx++; else { e.x += ((target.x*TILE+25-e.x)/d)*1.5; e.y += ((target.y*TILE+25-e.y)/d)*1.5; }
        if(e.hp <= 0) { sys.gold += 30; enemies.splice(i,1); }
        else { ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(e.x, e.y, 8, 0, Math.PI*2); ctx.fill(); }
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.col; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    // CHARA META CONTROL: HIJACKING THE READY BUTTON
    if(sys.genocide > 90 && Math.random() > 0.998) {
        logIt("* Chara is impatient.");
        document.getElementById('ready-btn').click();
    }

    updateHUD();
    requestAnimationFrame(loop);
}

/** 🖱️ INPUT **/
window.onmousemove = (e) => {
    let jitter = (sys.genocide / 25);
    sys.cursor.x = e.clientX + (Math.random()*jitter - jitter/2);
    sys.cursor.y = e.clientY + (Math.random()*jitter - jitter/2);
    document.getElementById('soul-cursor').style.left = sys.cursor.x + 'px';
    document.getElementById('soul-cursor').style.top = sys.cursor.y + 'px';

    let r = canvas.getBoundingClientRect();
    sys.cursor.tx = Math.floor((sys.cursor.x - r.left)/TILE);
    sys.cursor.ty = Math.floor((sys.cursor.y - r.top)/TILE);
};

canvas.onmousedown = () => {
    let u = units.find(x => x.tx === sys.cursor.tx && x.ty === sys.cursor.ty);
    if(u) {
        activeUnit = u;
        document.getElementById('morality-core').style.opacity = (u.id==='frisk')?"1":"0.3";
    } else if(placement) {
        if(sys.gold >= placement.cost) {
            sys.gold -= placement.cost;
            units.push(new Guardian(placement.id, sys.cursor.tx, sys.cursor.ty));
            placement = null;
            document.querySelectorAll('.tower-card').forEach(c => c.style.borderColor = 'white');
        }
    }
};

document.getElementById('ready-btn').onclick = () => {
    sys.wave++;
    for(let i=0; i<3+sys.wave; i++) {
        setTimeout(() => {
            enemies.push({ x: path[0].x*TILE+25, y: path[0].y*TILE+25, pIdx: 0, hp: 400*(1+sys.wave*0.6), spd: 1.2 });
        }, i * 1500);
    }
};

function logIt(t) { document.getElementById('log-text').innerText = t; }
function updateHUD() {
    document.getElementById('gold').innerText = Math.floor(sys.gold);
    document.getElementById('lv').innerText = sys.lv;
}
init();
</script>
</body>
</html>
