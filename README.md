<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: COUNCIL OF SOULS</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --green: #22c55e; --blue: #3b82f6; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        #layout { display: grid; grid-template-columns: 850px 350px; grid-template-rows: 80px 450px 280px; gap: 10px; padding: 10px; border: 5px double white; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #333; cursor: crosshair; }
        
        #sidebar { grid-column: 2; grid-row: 2; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; background: #0a0a0a; }
        
        /* CHAT SYSTEM */
        #chat-area { flex: 1; border: 1px solid #444; margin-bottom: 10px; overflow-y: auto; padding: 5px; font-size: 0.8rem; background: #000; }
        #whisper-area { height: 60px; border: 1px solid var(--purple); margin-bottom: 10px; padding: 5px; font-size: 0.75rem; color: var(--purple); background: #050005; }
        
        /* MULTI-SOUL BARS */
        #council-dashboard { grid-column: 1 / 3; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 15px; }
        .soul-container { flex: 1; border-right: 1px solid #333; padding-right: 10px; }
        .influence-bar { width: 100%; height: 10px; background: #222; margin-top: 5px; border: 1px solid #fff; }
        .fill { height: 100%; transition: width 0.5s ease-in-out; }

        .cmd-btn { background: #000; border: 2px solid #fff; color: #fff; padding: 8px; cursor: pointer; font-family: inherit; font-weight: bold; width: 100%; margin-top: 5px; }
        .cmd-btn:hover { background: #fff; color: #000; }

        .glitch { animation: gl 0.1s infinite; }
        @keyframes gl { 0%{transform:translate(1px)} 50%{transform:translate(-1px)} }
    </style>
</head>
<body>

    <div id="layout">
        <header>
            <div>CORE: <span id="hp-val" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold-val" style="color:var(--gold)">600</span></div>
            <div>LV: <span id="lv-val">1</span></div>
            <button id="wave-btn" class="cmd-btn" style="width:120px;">READY</button>
        </header>

        <canvas id="canvas"></canvas>

        <div id="sidebar">
            <h4 style="text-align:center; margin:0; color:var(--gold);">COUNCIL CHAT</h4>
            <div id="chat-area"></div>
            <div id="whisper-area"><i>[Whisper to Frisk]: Stay determined.</i></div>
            <div id="shop">
                <button class="cmd-btn" onclick="setPlacement('frisk')">FRISK ($200)</button>
                <button class="cmd-btn" onclick="setPlacement('sans')">SANS ($600)</button>
                <button class="cmd-btn" onclick="setPlacement('papy')">PAPYRUS ($150)</button>
            </div>
        </div>

        <div id="council-dashboard">
            <div class="soul-container" style="color:var(--cyan)">
                <b>PLAYER/FRISK</b>
                <div class="influence-bar"><div id="bar-player" class="fill" style="background:var(--cyan)"></div></div>
                <div class="btn-grid">
                    <button class="cmd-btn" style="font-size:0.7rem" onclick="playerAction('ACT')">ACT</button>
                    <button class="cmd-btn" style="font-size:0.7rem" onclick="playerAction('FIGHT')">FIGHT</button>
                </div>
            </div>
            <div class="soul-container" style="color:var(--blue)">
                <b>SANS</b>
                <div class="influence-bar"><div id="bar-sans" class="fill" style="background:var(--blue)"></div></div>
                <div style="font-size:0.6rem; margin-top:5px;">STATUS: <span id="sans-status">LAZING</span></div>
            </div>
            <div class="soul-container" style="color:#fff">
                <b>PAPYRUS</b>
                <div class="influence-bar"><div id="bar-papy" class="fill" style="background:#fff"></div></div>
                <div style="font-size:0.6rem; margin-top:5px;">PUZZLES: <span id="papy-count">0</span></div>
            </div>
            <div class="soul-container" style="color:var(--red)">
                <b>CHARA</b>
                <div class="influence-bar"><div id="bar-chara" class="fill" style="background:var(--red)"></div></div>
                <div style="font-size:0.6rem; margin-top:5px;">MIMICRY: <span id="chara-mode">LEARNING</span></div>
            </div>
        </div>
    </div>

<script>
/** ⚙️ COUNCIL DATA & PERSONAS **/
let council = {
    player: 25, frisk: 25, sans: 25, papy: 25, chara: 25,
    gold: 600, lv: 1, hp: 999, wave: 0,
    history: [],
    mouse: { tx:0, ty:0 }
};

const TOWERS = {
    frisk: { name: "Protagonist", cost: 200, color: "#ff00ff" },
    sans:  { name: "Sans", cost: 600, color: "#008cff" },
    papy:  { name: "Papyrus", cost: 150, color: "#fff" }
};

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
canvas.width = 850; canvas.height = 450;

let units = [], enemies = [], path = [], bullets = [];
let placement = null, selected = null;

/** ⚔️ CORE ENGINE **/
function init() {
    // Load Saved Influence
    const saved = localStorage.getItem('utd_council_progress');
    if(saved) {
        let data = JSON.parse(saved);
        council.player = data.player; council.sans = data.sans;
        council.papy = data.papy; council.chara = data.chara;
    }
    
    // Auto-Save Loop (Every Minute)
    setInterval(() => {
        localStorage.setItem('utd_council_progress', JSON.stringify({
            player: council.player, sans: council.sans, papy: council.papy, chara: council.chara
        }));
        chat("SYSTEM", "Influence progress auto-saved.");
    }, 60000);

    genPath();
    loop();
    chat("SANS", "hey. looks like we're all here.");
}

function genPath() {
    path = []; let x = 0, y = 4;
    while(x < 17) {
        path.push({x, y});
        if(Math.random() < 0.2 && y > 1) y--; else if(Math.random() < 0.2 && y < 7) y++; else x++;
    }
}

class Guardian {
    constructor(id, tx, ty, owner) {
        this.id = id; this.tx = tx; this.ty = ty; this.owner = owner;
        this.x = tx*TILE+25; this.y = ty*TILE+25;
        this.cd = 0;
    }
    update() {
        if(this.cd > 0) this.cd--;
        let target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < 150);
        if(target && this.cd <= 0) {
            target.hp -= (this.id==='frisk') ? (council.lv * 100) : 50;
            bullets.push({x1:this.x, y1:this.y, x2:target.x, y2:target.y, life:5, col:this.owner==='chara'?'red':'white'});
            this.cd = (this.id==='sans') ? 5 : 40;
        }
    }
    draw() {
        ctx.fillStyle = (this.owner==='chara')?'#f00':TOWERS[this.id].color;
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.fillStyle='#000'; ctx.font='9px Arial'; ctx.fillText(this.owner.toUpperCase(), this.x-20, this.y+5);
    }
}

/** 🤖 AI BEHAVIOR LOGIC **/
function aiTick() {
    // SANS: Random & Lazy (25%+ Placing)
    if(council.sans >= 25 && Math.random() > 0.995 && council.gold >= 600) {
        let tx = Math.floor(Math.random()*17), ty = Math.floor(Math.random()*9);
        units.push(new Guardian('sans', tx, ty, 'sans'));
        council.gold -= 600;
        chat("SANS", "put a sentry station over there. or something. i'm going to grillby's.");
    }

    // PAPYRUS: Placing Strategically (20%+ Placing)
    if(council.papy >= 25 && Math.random() > 0.997 && council.gold >= 150) {
        let p = path[Math.floor(path.length/2)]; // Place near middle
        units.push(new Guardian('papy', p.x, p.y+1, 'papy'));
        council.gold -= 150;
        chat("PAPYRUS", "NYEH HEH HEH! I HAVE PLACED A MAGNIFICENT PUZZLE!");
    }

    // CHARA: Aggressive & Complex (70%+ UI Hijack)
    if(council.chara >= 70 && Math.random() > 0.999) {
        chat("CHARA", "This timeline is boring. Let's speed it up.");
        document.getElementById('wave-btn').click(); // Hijacks Wave button
    }
}

/** 💬 CHAT SYSTEM **/
function chat(who, msg) {
    const area = document.getElementById('chat-area');
    let color = who === 'SANS' ? 'var(--blue)' : (who === 'PAPYRUS' ? '#fff' : (who === 'CHARA' ? 'var(--red)' : 'var(--cyan)'));
    area.innerHTML += `<div><b style="color:${color}">${who}:</b> ${msg}</div>`;
    area.scrollTop = area.scrollHeight;
}

function whisper(msg) {
    document.getElementById('whisper-area').innerHTML = `<i>[Whisper to Frisk]: ${msg}</i>`;
}

/** 🔁 LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    path.forEach(p => { ctx.fillStyle = '#111'; ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE); });

    aiTick();
    units.forEach(u => { u.update(); u.draw(); });

    for(let i=enemies.length-1; i>=0; i--) {
        let e = enemies[i];
        let target = path[e.pIdx];
        if(!target) { 
            chat("CHARA", "How pathetic."); 
            alert("TIMELINE RESET."); 
            location.reload(); 
            return; 
        }
        let d = Math.hypot(target.x*TILE+25-e.x, target.y*TILE+25-e.y);
        if(d < 2) e.pIdx++; else { e.x += ((target.x*TILE+25-e.x)/d)*1.2; e.y += ((target.y*TILE+25-e.y)/d)*1.2; }
        if(e.hp <= 0) { council.gold += 30; enemies.splice(i,1); }
        else { ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(e.x, e.y, 10, 0, Math.PI*2); ctx.fill(); }
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.col; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    updateHUD();
    requestAnimationFrame(loop);
}

function playerAction(type) {
    if(type === 'FIGHT') {
        council.chara += 2; council.player -= 2;
        whisper("Chara is gaining control...");
    } else {
        council.papy += 1; council.chara -= 1;
        whisper("Papyrus likes your mercy.");
    }
}

function setPlacement(id) {
    placement = { ...TOWERS[id], id: id };
}

canvas.onmousedown = () => {
    if(placement && council.gold >= placement.cost) {
        council.gold -= placement.cost;
        units.push(new Guardian(placement.id, council.mouse.tx, council.mouse.ty, 'player'));
        placement = null;
    }
};

canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    council.mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    council.mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

document.getElementById('wave-btn').onclick = () => {
    council.wave++;
    for(let i=0; i<3+council.wave; i++) {
        setTimeout(() => {
            enemies.push({ x: path[0].x*TILE+25, y: path[0].y*TILE+25, pIdx: 0, hp: 400 });
        }, i * 1500);
    }
};

function updateHUD() {
    document.getElementById('gold-val').innerText = Math.floor(council.gold);
    document.getElementById('hp-val').innerText = council.hp;
    document.getElementById('bar-player').style.width = council.player + "%";
    document.getElementById('bar-sans').style.width = council.sans + "%";
    document.getElementById('bar-papy').style.width = council.papy + "%";
    document.getElementById('bar-chara').style.width = council.chara + "%";
}

init();
</script>
</body>
</html>
