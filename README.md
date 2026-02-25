<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: ABSOLUTE CORRUPTION</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --corrupt: #4ade80; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        
        #layout { display: grid; grid-template-columns: 850px 320px; grid-template-rows: 70px 450px 240px; gap: 10px; padding: 10px; border: 4px double white; position: relative; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #222; }
        
        #sidebar { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; gap: 8px; background: #0a0a0a; position: relative; }
        .tower-card { border: 2px solid white; padding: 10px; text-align: center; cursor: pointer; font-size: 0.8rem; transition: 0.1s; }
        .tower-card.active { border-color: var(--gold); box-shadow: 0 0 15px var(--gold); }

        #dashboard { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; background: #000; overflow: hidden; }
        
        /* CORRUPTION UI */
        #frisk-core { width: 440px; border-right: 2px solid white; padding-right: 15px; position: relative; }
        .stat-line { display: flex; justify-content: space-between; font-size: 0.75rem; margin-bottom: 2px; }
        .meter-bg { width: 100%; height: 6px; background: #222; border: 1px solid #444; margin-bottom: 5px; }
        .meter-fill { height: 100%; transition: width 0.4s ease-out; }

        .btn-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 5px; margin-top: 10px; }
        .cmd-btn { background: #000; border: 2px solid #fff; color: #fff; padding: 10px; cursor: pointer; font-family: inherit; font-weight: bold; font-size: 1rem; position: relative; }
        .cmd-btn:hover:not(:disabled) { background: #fff; color: #000; }
        
        #log-box { flex: 1; font-size: 1.1rem; line-height: 1.2; position: relative; }

        .glitch-heavy { animation: glitch 0.1s infinite; color: var(--red); filter: contrast(200%); }
        @keyframes glitch { 0% { transform: translate(3px, -2px); opacity: 0.8; } 50% { transform: translate(-3px, 2px); opacity: 1; } 100% { transform: translate(0); } }
        
        #soulless-overlay { position: fixed; inset: 0; background: red; z-index: 10000; display: none; align-items: center; justify-content: center; color: black; font-size: 3rem; font-weight: bold; text-align: center; }
    </style>
</head>
<body id="b">

    <div id="soulless-overlay">ERASING...</div>

    <div id="layout">
        <header id="main-header">
            <div>DT: <span id="hp" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold" style="color:var(--gold)">600</span></div>
            <div>LV: <span id="lv">1</span></div>
            <button id="btn-fight" class="cmd-btn" style="width:120px; padding:2px">FIGHT</button>
        </header>

        <canvas id="canvas"></canvas>

        <div id="sidebar">
            <h4 style="text-align:center; margin:0;">THE VOID</h4>
            <div id="shop"></div>
            <div id="corruption-warning" style="color:var(--red); font-size:0.6rem; text-align:center; position:absolute; bottom:5px;"></div>
        </div>

        <div id="dashboard">
            <div id="frisk-core">
                <div class="stat-line"><b>FRISK</b> <span id="status-text">PLAYER CONTROL: 95%</span></div>
                <div class="meter-bg"><div id="player-bar" class="meter-fill" style="background:var(--cyan); width:95%"></div></div>
                <div class="stat-line"><span style="color:var(--red)">CHARA INFLUENCE</span> <span id="chara-val">5%</span></div>
                <div class="meter-bg"><div id="chara-bar" class="meter-fill" style="background:var(--red); width:5%"></div></div>
                
                <div class="btn-grid" id="controls">
                    <button class="cmd-btn" id="btn-f" onclick="cmd('FIGHT')">FIGHT</button>
                    <button class="cmd-btn" id="btn-a" onclick="cmd('ACT')">ACT</button>
                    <button class="cmd-btn" id="btn-i" onclick="cmd('ITEM')">ITEM</button>
                    <button class="cmd-btn" id="btn-m" onclick="cmd('MERCY')">SPARE</button>
                </div>
                <button id="reset-btn" class="cmd-btn" style="width:100%; margin-top:10px; border-color:var(--red); color:var(--red); display:none;">ERASE TIMELINE (100G)</button>
            </div>
            <div id="log-box">
                <span style="color:red">❤</span> <span id="log">* Chara is watching you.</span>
            </div>
        </div>
    </div>

<script>
/** ⚙️ THE CORRUPTION ENGINE **/
let state = {
    gold: 600, hp: 999, lv: 1, 
    playerInfluence: 95, charaInfluence: 5, friskPower: 0,
    soulless: false, wave: 0,
    mouse: { tx: 0, ty: 0 }
};

const TOWERS = {
    frisk: { n: "Frisk", c: 200, col: "#ff00ff" },
    sans:  { n: "Sans",  c: 600, col: "#008cff" },
    papy:  { n: "Papyrus", c: 150, col: "#fff" },
    undy:  { n: "Undyne", c: 400, col: "#00ffff" },
    void:  { n: "Void",   c: 500, col: "#555" }
};

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
canvas.width = 850; canvas.height = 450;

let units = [], enemies = [], path = [], bullets = [], effects = [];
let placement = null, selected = null;

/** ⚔️ CORE LOGIC **/
function init() {
    const shop = document.getElementById('shop');
    Object.keys(TOWERS).forEach(k => {
        const div = document.createElement('div');
        div.className = 'tower-card';
        div.innerHTML = `<b>${TOWERS[k].n}</b><br>$${TOWERS[k].c}`;
        div.onclick = () => {
            placement = { ...TOWERS[k], id: k };
            document.querySelectorAll('.tower-card').forEach(c => c.classList.remove('active'));
            div.classList.add('active');
        };
        shop.appendChild(div);
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

/** 🛡️ GUARDIAN REFACTOR **/
class Guardian {
    constructor(id, tx, ty) {
        this.id = id; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = (id==='frisk')?state.lv:1;
        this.stamina = 100; this.dodge = 100;
        this.cd = 0;
    }
    update() {
        if(this.cd > 0) this.cd--;
        
        let range = (this.id === 'frisk') ? 140 : 350;
        this.target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < range);

        // AUTO-CORRUPTION (Chara taking control)
        if(this.id === 'frisk') {
            this.lv = state.lv;
            if(state.charaInfluence > 30 && this.target && this.cd <= 0) {
                if(Math.random()*100 < state.charaInfluence) {
                    this.strike(this.target, true);
                    this.cd = 30;
                }
            }
        } else if(this.target && this.cd <= 0) {
            this.strike(this.target);
            this.cd = (this.id==='sans')?5:60;
        }
    }
    strike(t, charaControlled = false) {
        let dmg = (this.id==='frisk') ? this.lv*80 : 50;
        if(this.id === 'sans') { t.kr = 100; dmg = 1; }
        if(charaControlled) dmg *= 2;

        t.hp -= dmg;
        bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, col:charaControlled?'#f00':'#fff'});
    }
    draw() {
        let c = TOWERS[this.id].col;
        if(this.id === 'frisk' && state.charaInfluence > 50) c = "#f00";
        if(selected === this) { ctx.shadowBlur = 10; ctx.shadowColor = 'gold'; }
        ctx.fillStyle = c;
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.shadowBlur = 0;
        ctx.fillStyle='#000'; ctx.font='10px Arial'; ctx.fillText("LV"+this.lv, this.x-10, this.y+5);
    }
}

/** 🕹️ COMMAND ENGINE **/
function cmd(type) {
    if(!selected || selected.id !== 'frisk') return;
    
    // Meta-Corruption Check
    if(Math.random()*100 < state.charaInfluence) {
        type = "FIGHT";
        logIt("* CHARA: Do not hesitate.");
    }

    if(!selected.target) return logIt("* But nobody was there.");
    let t = selected.target;

    if(type === 'FIGHT') {
        t.hp -= state.lv * 150;
        state.lv++; 
        logIt(`* You dealt ${state.lv*150} damage.`);
    } else if(type === 'ACT') {
        t.spd *= 0.5;
        logIt("* You calmed the soul.");
    } else if(type === 'MERCY') {
        if(t.hp < 100) {
            state.gold += 300;
            enemies = enemies.filter(e => e !== t);
            logIt("* SPARED. Gained 300G.");
        } else logIt("* Not weak enough.");
    }
    
    updateSchism();
}

function updateSchism() {
    if(!state.soulless) {
        state.charaInfluence = Math.min(90, (state.lv * 4.5));
        state.playerInfluence = 100 - state.charaInfluence;
    } else {
        // Reincarnated State: 60 Player, 30 Chara base
        state.charaInfluence = 30 + (state.lv * 3);
        state.playerInfluence = 100 - state.charaInfluence;
    }

    if(state.lv >= 20) {
        document.getElementById('reset-btn').style.display = 'block';
    }

    // Visual Glitches based on Corruption
    if(state.charaInfluence > 70) {
        document.getElementById('dashboard').classList.add('glitch-heavy');
        document.getElementById('corruption-warning').innerText = "SYSTEM FAILURE: CHARA DETECTED";
    }
}

document.getElementById('reset-btn').onclick = () => {
    if(state.gold >= 100) {
        state.gold -= 100;
        state.soulless = true;
        document.getElementById('soulless-overlay').style.display = 'flex';
        setTimeout(() => {
            document.getElementById('soulless-overlay').style.display = 'none';
            state.lv = 1;
            state.playerInfluence = 60;
            state.charaInfluence = 30;
            state.friskPower = 10;
            units = units.filter(u => u.id === 'frisk'); // Keep only Frisk
            logIt("* You brought her back. Why?");
            document.getElementById('reset-btn').style.display = 'none';
            updateSchism();
        }, 1500);
    }
};

/** 🔁 LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    path.forEach(p => { ctx.fillStyle = '#111'; ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE); });

    units.forEach(u => { u.update(); u.draw(); });

    for(let i=enemies.length-1; i>=0; i--) {
        let e = enemies[i];
        let s = (e.spd || 1);
        let target = path[e.pIdx];
        if(!target) { alert("TIMELINE ERASED."); location.reload(); return; }
        
        let tx = target.x*TILE+TILE/2, ty = target.y*TILE+TILE/2;
        let d = Math.hypot(tx-e.x, ty-e.y);
        if(d < s) e.pIdx++; else { e.x += ((tx-e.x)/d)*s; e.y += ((ty-e.y)/d)*s; }

        if(e.hp <= 0) { state.gold += 40; enemies.splice(i,1); }
        else {
            ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(e.x, e.y, 10, 0, Math.PI*2); ctx.fill();
        }
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.col; ctx.lineWidth = 3;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    // Dashboard HUD
    document.getElementById('hp').innerText = state.hp;
    document.getElementById('gold').innerText = state.gold;
    document.getElementById('lv').innerText = state.lv;
    document.getElementById('player-bar').style.width = state.playerInfluence + "%";
    document.getElementById('chara-bar').style.width = state.charaInfluence + "%";
    document.getElementById('status-text').innerText = `PLAYER CONTROL: ${Math.floor(state.playerInfluence)}%`;
    document.getElementById('chara-val').innerText = `${Math.floor(state.charaInfluence)}%`;

    requestAnimationFrame(loop);
}

/** 🖱️ INPUT **/
canvas.onmousedown = () => {
    let tx = state.mouse.tx, ty = state.mouse.ty;
    let u = units.find(x => x.tx === tx && x.ty === ty);
    
    if(u) {
        selected = u;
        document.getElementById('frisk-core').style.opacity = (u.id === 'frisk') ? "1" : "0.3";
    } else if(placement) {
        if(state.gold >= placement.c) {
            state.gold -= placement.c;
            units.push(new Guardian(placement.id, tx, ty));
            placement = null;
            document.querySelectorAll('.tower-card').forEach(c => c.classList.remove('active'));
        }
    }
};

canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    state.mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    state.mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

document.getElementById('btn-fight').onclick = () => {
    state.wave++;
    for(let i=0; i<3+state.wave; i++) {
        setTimeout(() => {
            enemies.push({ x: path[0].x*TILE+TILE/2, y: path[0].y*TILE+TILE/2, pIdx: 0, hp: 200*(1+state.wave*0.5), spd: 1.2 });
        }, i * 1500);
    }
};

function logIt(t) { document.getElementById('log').innerText = t; }
init();
</script>
</body>
</html>
