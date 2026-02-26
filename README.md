<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: THE COUNCIL TERMINAL</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --green: #22c55e; --blue: #3b82f6; --white: #ffffff; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; height: 100vh; display: flex; align-items: center; justify-content: center; }
        
        #game-interface { width: 1200px; height: 850px; display: grid; grid-template-columns: 850px 350px; grid-template-rows: 80px 1fr 250px; gap: 10px; border: 5px double #fff; padding: 10px; background: #000; position: relative; }

        /* THE BOARD */
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 2px solid #333; cursor: crosshair; }

        /* THE CHAT BOX (META) */
        #meta-chat { grid-column: 2; grid-row: 2; border: 2px solid #fff; background: #0a0a0a; display: flex; flex-direction: column; padding: 10px; overflow: hidden; }
        #chat-log { flex: 1; overflow-y: auto; font-size: 0.85rem; padding-right: 5px; border-bottom: 1px solid #333; margin-bottom: 5px; }
        #chat-log div { margin-bottom: 8px; line-height: 1.2; }
        .ch-s { color: var(--blue); } .ch-p { color: var(--white); } .ch-c { color: var(--red); font-weight: bold; } .ch-f { color: var(--purple); } .ch-sys { color: #555; font-style: italic; }

        /* THE SOUL COUNCIL (DASHBOARD) */
        #council-dashboard { grid-column: 1 / 3; grid-row: 3; border: 2px solid #fff; display: flex; padding: 15px; gap: 20px; }
        .soul-card { flex: 1; border-right: 1px solid #444; padding: 0 10px; }
        .influence-bg { width: 100%; height: 8px; background: #111; border: 1px solid #666; margin: 5px 0; overflow: hidden; }
        .influence-fill { height: 100%; width: 25%; transition: width 0.5s ease; }

        /* BUTTONS */
        .cmd-btn { background: #000; border: 2px solid #fff; color: #fff; padding: 8px; cursor: pointer; font-family: inherit; width: 100%; font-size: 0.8rem; margin-top: 5px; }
        .cmd-btn:hover { background: #fff; color: #000; }
        .btn-red { border-color: var(--red); color: var(--red); }

        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; font-size: 1.2rem; }
        
        /* OVERLAYS */
        #glitch-overlay { position: fixed; inset: 0; pointer-events: none; background: rgba(255,0,0,0); z-index: 1000; transition: background 0.1s; }
        .glitch-active { background: rgba(255,0,0,0.1) !important; animation: flicker 0.1s infinite; }
        @keyframes flicker { 0% { opacity: 0; } 50% { opacity: 1; } 100% { opacity: 0; } }
    </style>
</head>
<body>

    <div id="glitch-overlay"></div>

    <div id="game-interface">
        <header>
            <div>DT: <span id="hp-val" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold-val" style="color:var(--gold)">600</span></div>
            <div>LV: <span id="lv-val">1</span></div>
            <button id="main-fight-btn" class="cmd-btn" style="width:120px;">READY?</button>
        </header>

        <canvas id="gameCanvas"></canvas>

        <aside id="meta-chat">
            <div id="chat-log"></div>
            <div id="whisper-box" style="font-size:0.7rem; color:var(--purple);"><i>[Whispering to Frisk...]</i></div>
        </aside>

        <footer id="council-dashboard">
            <div class="soul-card">
                <b style="color:var(--cyan)">PLAYER/FRISK</b>
                <div class="influence-bg"><div id="bar-p" class="influence-fill" style="background:var(--cyan)"></div></div>
                <button class="cmd-btn" onclick="playerMove('ACT')">ACT</button>
                <button class="cmd-btn" onclick="playerMove('FIGHT')">FIGHT</button>
            </div>
            <div class="soul-card">
                <b style="color:var(--blue)">SANS</b>
                <div class="influence-bg"><div id="bar-s" class="influence-fill" style="background:var(--blue)"></div></div>
                <button class="cmd-btn" id="sans-special" onclick="sansMove()">SHORTCUT</button>
            </div>
            <div class="soul-card">
                <b style="color:var(--white)">PAPYRUS</b>
                <div class="influence-bg"><div id="bar-papy" class="influence-fill" style="background:var(--white)"></div></div>
                <button class="cmd-btn" id="papy-special" onclick="papyMove()">GIFT</button>
            </div>
            <div class="soul-card">
                <b style="color:var(--red)">CHARA</b>
                <div class="influence-bg"><div id="bar-c" class="influence-fill" style="background:var(--red)"></div></div>
                <button class="cmd-btn btn-red" id="chara-special" onclick="charaMove()">ERASE</button>
            </div>
        </footer>
    </div>

<script>
/** 💾 MEMORY & PERSISTENCE **/
let state = {
    gold: 600, lv: 1, hp: 999, wave: 0,
    p: 25, s: 25, papy: 25, c: 25,
    history: [],
    mouse: { tx: 0, ty: 0 }
};

const TOWERS = {
    frisk: { name: "Protagonist", cost: 200, color: "#ff00ff" },
    sans:  { name: "Sans", cost: 600, color: "#008cff" },
    papy:  { name: "Papyrus", cost: 150, color: "#fff" }
};

/** 🎨 ENGINE SETUP **/
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
canvas.width = 850; canvas.height = 450;

let units = [], enemies = [], path = [], bullets = [];
let activePlacement = null;

function init() {
    // Load Memory
    const memory = localStorage.getItem('council_memory_v2');
    if(memory) {
        const d = JSON.parse(memory);
        state.p = d.p; state.s = d.s; state.papy = d.papy; state.c = d.c;
        log("SYS", "Memory loaded. The souls remember you.");
    }

    // Auto-Save
    setInterval(() => {
        localStorage.setItem('council_memory_v2', JSON.stringify({
            p: state.p, s: state.s, papy: state.papy, c: state.c
        }));
        log("SYS", "Timeline progress auto-saved.");
    }, 60000);

    genPath();
    loop();
    
    // Initial Banter
    setTimeout(() => log("SANS", "hey. nice terminal you got here."), 1000);
    setTimeout(() => log("PAPYRUS", "SANS!! STOP SLACKING!! WE HAVE SOULS TO PROTECT!!"), 2500);
}

function genPath() {
    let x = 0, y = 4;
    while(x < 17) {
        path.push({x, y});
        if(Math.random() < 0.2 && y > 1) y--; else if(Math.random() < 0.2 && y < 7) y++; else x++;
    }
}

/** 🤖 AI BEHAVIOR & TALKING **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    path.forEach(p => { ctx.fillStyle = '#111'; ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE); });

    // META-BEHAVIOR: Random Council Actions
    if(Math.random() > 0.997) {
        const roll = Math.random();
        if(roll < 0.3) sansAI();
        else if(roll < 0.6) papyAI();
        else charaAI();
    }

    units.forEach(u => {
        u.cd = u.cd > 0 ? u.cd - 1 : 0;
        let target = enemies.find(e => Math.hypot(e.x - (u.tx*50+25), e.y - (u.ty*50+25)) < 150);
        if(target && u.cd <= 0) {
            target.hp -= (u.id === 'frisk' ? state.lv * 100 : 50);
            bullets.push({x1: u.tx*50+25, y1: u.ty*50+25, x2: target.x, y2: target.y, life: 5, col: u.id === 'sans' ? 'cyan' : 'white'});
            u.cd = 40;
        }
        ctx.fillStyle = TOWERS[u.id].color;
        ctx.fillRect(u.tx*TILE+5, u.ty*TILE+5, TILE-10, TILE-10);
    });

    for(let i=enemies.length-1; i>=0; i--) {
        let e = enemies[i];
        let target = path[e.pIdx];
        if(!target) { 
            log("CHARA", "Too slow."); 
            location.reload(); 
            return; 
        }
        let d = Math.hypot(target.x*TILE+25-e.x, target.y*TILE+25-e.y);
        if(d < 2) e.pIdx++; else { e.x += ((target.x*TILE+25-e.x)/d)*1.5; e.y += ((target.y*TILE+25-e.y)/d)*1.5; }
        if(e.hp <= 0) { state.gold += 30; enemies.splice(i,1); }
        else { ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(e.x, e.y, 10, 0, Math.PI*2); ctx.fill(); }
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.col; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    updateUI();
    requestAnimationFrame(loop);
}

/** 🗣️ DIALOGUE ENGINE **/
function log(who, msg) {
    const log = document.getElementById('chat-log');
    let cls = "ch-sys";
    if(who === "SANS") cls = "ch-s";
    if(who === "PAPYRUS") cls = "ch-p";
    if(who === "CHARA") cls = "ch-c";
    if(who === "FRISK") cls = "ch-f";
    
    log.innerHTML += `<div><b class="${cls}">${who}:</b> ${msg}</div>`;
    log.scrollTop = log.scrollHeight;

    // Logic for characters responding to each other
    if(who === "CHARA" && Math.random() > 0.5) setTimeout(() => log("SANS", "hey, kid. chill."), 1000);
    if(who === "SYS" && msg.includes("auto-saved")) setTimeout(() => log("PAPYRUS", "THE TIMELINE IS SECURE!!"), 800);
}

/** 🕹️ CHARACTER MOVES **/
function playerMove(type) {
    if(type === 'FIGHT') {
        state.c += 2; state.p -= 2;
        log("CHARA", "Good choice.");
    } else {
        state.papy += 1; state.c -= 1;
        log("FRISK", "* You reached out. Chara sighs.");
    }
}

function sansMove() {
    if(state.gold < 100) return;
    state.gold -= 100;
    enemies.forEach(e => e.pIdx = Math.max(0, e.pIdx - 2));
    log("SANS", "shortcut. don't mention it.");
}

function papyMove() {
    state.gold += 100;
    log("PAPYRUS", "I MADE SPAGHETTI!! HAVE SOME EXTRA GOLD!!");
}

function charaMove() {
    if(state.c < 50) {
        log("CHARA", "Not enough power... yet.");
        return;
    }
    document.getElementById('glitch-overlay').classList.add('glitch-active');
    setTimeout(() => {
        enemies = [];
        document.getElementById('glitch-overlay').classList.remove('glitch-active');
        log("CHARA", "E R A S E D.");
    }, 500);
}

/** 🤖 AUTOMATED AI EVENTS **/
function sansAI() {
    if(state.s > 60) {
        log("SANS", "i'm bored. let's skip ahead.");
        document.getElementById('main-fight-btn').click();
    }
}

function papyAI() {
    if(state.gold < 150) {
        state.gold += 50;
        log("PAPYRUS", "YOU LOOKED BROKE! I HELPED!");
    }
}

function charaAI() {
    if(state.lv > 5 && Math.random() > 0.8) {
        log("CHARA", "Why place towers when I can just... delete?");
        if(enemies.length > 0) enemies[0].hp = 0;
    }
}

/** 🖱️ INPUTS **/
canvas.onmousedown = (e) => {
    let rect = canvas.getBoundingClientRect();
    let tx = Math.floor((e.clientX - rect.left)/TILE);
    let ty = Math.floor((e.clientY - rect.top)/TILE);
    
    if(state.gold >= 200) {
        state.gold -= 200;
        units.push({ id: 'frisk', tx, ty, cd: 0 });
    }
};

document.getElementById('main-fight-btn').onclick = () => {
    state.wave++;
    log("SYS", `Wave ${state.wave} incoming.`);
    for(let i=0; i<5+state.wave; i++) {
        setTimeout(() => {
            enemies.push({ x: path[0].x*TILE+25, y: path[0].y*TILE+25, pIdx: 0, hp: 300 + (state.wave*100) });
        }, i * 1200);
    }
};

function updateUI() {
    document.getElementById('gold-val').innerText = Math.floor(state.gold);
    document.getElementById('hp-val').innerText = state.hp;
    document.getElementById('lv-val').innerText = state.lv;
    document.getElementById('bar-p').style.width = state.p + "%";
    document.getElementById('bar-s').style.width = state.s + "%";
    document.getElementById('bar-papy').style.width = state.papy + "%";
    document.getElementById('bar-c').style.width = state.c + "%";
}

init();
</script>
</body>
</html>
