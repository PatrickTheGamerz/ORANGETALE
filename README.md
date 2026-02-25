<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: ABSOLUTE TIMELINE</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --green: #22c55e; --blue: #3b82f6; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        
        #layout { display: grid; grid-template-columns: 850px 320px; grid-template-rows: 70px 450px 280px; gap: 10px; padding: 10px; border: 4px double white; position: relative; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #050505; border: 1px solid #333; cursor: crosshair; }
        
        #sidebar { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 10px; display: flex; flex-direction: column; gap: 8px; background: #0a0a0a; overflow-y: auto; }
        .tower-card { border: 2px solid white; padding: 10px; text-align: center; cursor: pointer; font-size: 0.8rem; }
        .tower-card.active { border-color: var(--gold); background: #1a1a1a; }

        #dashboard { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; background: #000; }
        
        /* THE COMMAND WHEEL UI */
        #command-center { width: 450px; border-right: 2px solid white; padding-right: 15px; }
        .tab-row { display: flex; gap: 5px; margin-bottom: 10px; }
        .tab-btn { flex: 1; background: #000; border: 2px solid #fff; color: #fff; padding: 5px; cursor: pointer; font-family: inherit; font-size: 0.7rem; }
        .tab-btn.active { background: #fff; color: #000; }
        
        .action-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
        .action-btn { background: #000; border: 2px solid var(--gold); color: var(--gold); padding: 10px; cursor: pointer; font-family: inherit; font-weight: bold; font-size: 0.9rem; position: relative; }
        .action-btn:disabled { opacity: 0.2; filter: grayscale(1); }

        #dialogue { flex: 1; font-size: 1.1rem; border-left: 2px solid white; padding-left: 15px; }
        
        /* GHOST CURSOR */
        #chara-hand { position: fixed; width: 30px; height: 30px; border: 3px solid var(--red); pointer-events: none; z-index: 10000; display: none; transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1); }

        .stat-val { font-weight: bold; color: var(--gold); }
        .prep-overlay { position: absolute; top: 10px; left: 10px; background: rgba(0,0,0,0.8); border: 2px solid var(--red); padding: 10px; display: none; z-index: 100; }
    </style>
</head>
<body id="body">

    <div id="chara-hand"></div>
    <div id="prep-box" class="prep-overlay">
        <b style="color:var(--red)">PREPARATION PHASE</b><br>
        <span id="wave-info">Souls Detected...</span><br>
        <button onclick="engageWave()" class="action-btn" style="width:100%; margin-top:10px;">ENGAGE SOULS</button>
    </div>

    <div id="layout">
        <header>
            <div>DT: <span id="hp" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold" class="stat-val">600</span></div>
            <div>LV: <span id="lv" class="stat-val">1</span></div>
            <button id="prep-btn" class="action-btn" style="width:120px; padding:2px">PREPARE</button>
        </header>

        <canvas id="canvas"></canvas>

        <div id="sidebar">
            <h4 style="text-align:center; margin:0; color:var(--red);">LOADOUT</h4>
            <div id="shop"></div>
            <div id="ai-status" style="font-size:0.6rem; color:#666; text-align:center;">AI: OBSERVING...</div>
        </div>

        <div id="dashboard">
            <div id="command-center" style="display:none;">
                <div style="display:flex; justify-content:space-between; margin-bottom:5px;">
                    <b id="char-name">FRISK</b>
                    <span id="control-tag" style="font-size:0.6rem;">PLAYER: 95%</span>
                </div>
                
                <div class="tab-row">
                    <button class="tab-btn active" onclick="switchTab('ACT')">ACT</button>
                    <button class="tab-btn" onclick="switchTab('ITEM')">ITEM</button>
                    <button class="tab-btn" onclick="switchTab('FIGHT')">FIGHT</button>
                    <button class="tab-btn" onclick="switchTab('MERCY')">MERCY</button>
                </div>

                <div id="action-area" class="action-grid">
                    </div>
                
                <button id="up-hero" class="tab-btn" style="width:100%; margin-top:10px; border-color:var(--gold);">LEVEL UP ($150)</button>
            </div>
            <div id="dialogue">
                <span style="color:red">❤</span> <span id="log-text">* Determination is the key to survival.</span>
            </div>
        </div>
    </div>

<script>
/** ⚙️ THE ABSOLUTE STATE **/
let game = {
    gold: 600, lv: 1, hp: 999, wave: 0,
    playerInf: 95, friskInf: 4, charaInf: 1,
    isPrep: false, waveActive: false,
    history: [], activeTab: 'ACT',
    selectedUnit: null,
    mouse: { tx:0, ty:0 }
};

const TOWERS = {
    frisk: { n: "Frisk", c: 200, col: "#ff00ff", d: "The Protagonist. Essential for ACTing." },
    sans:  { n: "Sans",  c: 600, col: "#008cff", d: "1 DMG. KR Poison stalls souls." },
    papy:  { n: "Papyrus", c: 150, col: "#fff", d: "0 DMG. Turns souls BLUE (Slow)." },
    void:  { n: "Void",   c: 500, col: "#555", d: "Spawns Backwards Puppets." }
};

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
canvas.width = 850; canvas.height = 450;

let units = [], enemies = [], path = [], bullets = [], effects = [];
let placement = null;

/** ⚔️ CORE SYSTEMS **/
function init() {
    const shop = document.getElementById('shop');
    Object.keys(TOWERS).forEach(k => {
        const d = document.createElement('div');
        d.className = 'tower-card';
        d.innerHTML = `<b>${TOWERS[k].n}</b><br>$${TOWERS[k].c}`;
        d.onclick = () => {
            if(game.charaInf > 90) return;
            placement = { ...TOWERS[k], id: k };
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

class Guardian {
    constructor(id, tx, ty) {
        this.id = id; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = (id==='frisk')?game.lv:1;
        this.cd = 0; this.itemBuff = 0;
    }
    update() {
        if(this.cd > 0) this.cd--;
        if(this.itemBuff > 0) this.itemBuff--;
        
        let range = (this.id === 'frisk') ? 140 : 350;
        this.target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < range);

        if(this.id === 'frisk') {
            this.lv = game.lv;
            // Chara Sabotage Attack
            if(game.charaInf > 20 && this.target && this.cd <= 0) {
                if(Math.random()*100 < game.charaInf) {
                    this.strike(this.target, true);
                    this.cd = 30;
                }
            }
        } else if(this.target && this.cd <= 0) {
            this.strike(this.target);
            this.cd = (this.id==='sans')?4:60;
        }
    }
    strike(t, isChara = false) {
        let dmg = (this.id==='frisk') ? (this.lv * 80) : 50;
        if(this.id==='sans') { t.kr = 100; dmg = 1; }
        if(this.itemBuff > 0) dmg *= 3;
        if(isChara) dmg *= 5;

        t.hp -= dmg;
        bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, col:isChara?'#f00':'#fff'});
    }
    draw() {
        ctx.fillStyle = (this.id==='frisk' && game.charaInf > 50) ? "#f00" : TOWERS[this.id].col;
        if(game.selectedUnit === this) { ctx.shadowBlur = 10; ctx.shadowColor = 'gold'; }
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.shadowBlur = 0;
        ctx.fillStyle='#000'; ctx.font='9px Arial'; ctx.fillText("LV"+this.lv, this.x-10, this.y+5);
    }
}

/** 🕹️ COMMAND WHEEL LOGIC **/
function switchTab(t) {
    game.activeTab = t;
    updateCommandUI();
}

function updateCommandUI() {
    const area = document.getElementById('action-area');
    area.innerHTML = "";
    
    const cmds = {
        'ACT': ['Check', 'Talk', 'Flirt', 'Threaten'],
        'ITEM': ['B-Pie', 'Snowman', 'Steak'],
        'FIGHT': ['Slash'],
        'MERCY': ['Spare']
    };

    cmds[game.activeTab].forEach(c => {
        const btn = document.createElement('button');
        btn.className = 'action-btn';
        btn.innerText = c;
        btn.onclick = () => doAction(c);
        
        // Chara locks buttons
        if(game.charaInf > 90 && (game.activeTab !== 'FIGHT')) btn.disabled = true;
        
        area.appendChild(btn);
    });
}

function doAction(act) {
    if(!game.selectedUnit || !game.selectedUnit.target && act !== 'B-Pie') return;
    let t = game.selectedUnit.target;
    let roll = Math.random()*100;

    // Control Check
    if(roll < game.charaInf && game.lv < 19) {
        logIt("* Chara: Don't hold back.");
        act = 'Slash';
    }

    if(act === 'Slash') {
        t.hp -= game.lv * 200;
        game.lv++;
        updateTrinity();
    } else if(act === 'Talk') {
        t.stun = 60; logIt("* You talked. The soul paused.");
    } else if(act === 'Flirt') {
        t.spd *= 0.7; logIt("* The soul is flustered.");
    } else if(act === 'Threaten') {
        t.pIdx = Math.max(0, t.pIdx - 2); logIt("* You terrified them.");
    } else if(act === 'B-Pie') {
        game.hp = 999; game.gold -= 100; logIt("* You ate the pie. Determination restored.");
    } else if(act === 'Steak') {
        game.selectedUnit.itemBuff = 300; logIt("* Damage boosted!");
    } else if(act === 'Spare') {
        if(t.hp < 100) {
            game.gold += 400; enemies = enemies.filter(e => e !== t);
            logIt("* SPARED.");
        } else logIt("* Too strong to spare.");
    }
    updateCommandUI();
}

function updateTrinity() {
    if(game.lv < 7) {
        game.playerInf = 95 - (game.lv * 5);
        game.charaInf = 5 + (game.lv * 1);
    } else if(game.lv >= 19) {
        game.charaInf = 95; game.playerInf = 4; game.friskInf = 1;
    } else {
        game.charaInf = Math.min(90, game.lv * 5);
        game.playerInf = 100 - game.charaInf;
    }
    document.getElementById('control-tag').innerText = `PLAYER: ${Math.floor(game.playerInf)}%`;
}

/** 🔁 LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    path.forEach(p => { ctx.fillStyle = '#111'; ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE); });

    units.forEach(u => u.update());
    units.forEach(u => u.draw());

    if(game.waveActive) {
        for(let i=enemies.length-1; i>=0; i--) {
            let e = enemies[i];
            let target = path[e.pIdx];
            if(!target) { alert("TIMELINE RESET."); location.reload(); return; }
            
            if(e.stun > 0) { e.stun--; continue; }
            let d = Math.hypot(target.x*TILE+25-e.x, target.y*TILE+25-e.y);
            if(d < (e.spd||1)) e.pIdx++; else { e.x += ((target.x*TILE+25-e.x)/d)*e.spd; e.y += ((target.y*TILE+25-e.y)/d)*e.spd; }
            
            if(e.hp <= 0) { game.gold += 30; enemies.splice(i,1); }
            else { ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(e.x, e.y, 10, 0, Math.PI*2); ctx.fill(); }
        }
        if(enemies.length === 0) game.waveActive = false;
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.col; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    updateHUD();
    requestAnimationFrame(loop);
}

/** 🚪 PHASE CONTROL **/
document.getElementById('prep-btn').onclick = () => {
    if(game.waveActive) return;
    game.wave++;
    document.getElementById('prep-box').style.display = 'block';
    document.getElementById('wave-info').innerText = `Detected: ${3+game.wave} Souls. Prepare!`;
};

function engageWave() {
    document.getElementById('prep-box').style.display = 'none';
    game.waveActive = true;
    let count = 3 + game.wave;
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            enemies.push({ x: path[0].x*TILE+25, y: path[0].y*TILE+25, pIdx: 0, hp: 400*(1+game.wave*0.6), spd: 1.3, stun: 0 });
        }, i * 1500);
    }
}

function logIt(t) { document.getElementById('log-text').innerText = t; }

canvas.onmousedown = () => {
    let u = units.find(x => x.tx === game.mouse.tx && x.ty === game.mouse.ty);
    if(u && u.id === 'frisk') {
        game.selectedUnit = u;
        document.getElementById('command-center').style.display = 'block';
        updateCommandUI();
    } else if(placement) {
        if(game.gold >= placement.c) {
            game.gold -= placement.c;
            units.push(new Guardian(placement.id, game.mouse.tx, game.mouse.ty));
            game.history.push({ tx: game.mouse.tx, ty: game.mouse.ty, id: placement.id });
            placement = null;
        }
    }
};

canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    game.mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    game.mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

function updateHUD() {
    document.getElementById('gold').innerText = Math.floor(game.gold);
    document.getElementById('lv').innerText = game.lv;
    document.getElementById('hp').innerText = game.hp;
}

init();
</script>
</body>
</html>
