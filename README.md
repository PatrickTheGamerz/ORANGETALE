<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: ABSOLUTE RESOLVE</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --white: #ffffff; --box-bg: #000; }
        body { background: #111; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        
        #game-ui { display: grid; grid-template-columns: 850px 300px; grid-template-rows: 60px 450px 200px; gap: 10px; padding: 10px; border: 4px solid white; background: #000; }
        
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        canvas { grid-column: 1; grid-row: 2; background: #000; border: 1px solid #333; }
        
        #sidebar { grid-column: 2; grid-row: 2 / 4; border: 2px solid white; padding: 15px; display: flex; flex-direction: column; gap: 10px; }
        .tower-card { border: 2px solid white; padding: 10px; text-align: center; cursor: pointer; font-size: 0.8rem; }
        .tower-card.active { border-color: var(--gold); color: var(--gold); }

        #bottom-panel { grid-column: 1; grid-row: 3; border: 2px solid white; display: flex; padding: 15px; gap: 20px; }
        #frisk-controls { width: 400px; border-right: 2px solid white; padding-right: 15px; display: none; }
        #dialogue { flex: 1; font-size: 1.2rem; position: relative; }

        .act-btn { background: #000; border: 2px solid #ffcc00; color: #ffcc00; padding: 5px; margin: 2px; cursor: pointer; width: 45%; font-family: inherit; font-weight: bold; }
        .act-btn:hover { background: #ffcc00; color: #000; }
        
        #menu-overlay { position: fixed; inset: 0; background: #000; z-index: 9999; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .btn { background: #000; border: 2px solid #fff; color: #fff; padding: 10px 20px; cursor: pointer; font-family: inherit; font-size: 1.2rem; }
        .btn:hover { background: #fff; color: #000; }
    </style>
</head>
<body>

    <div id="menu-overlay">
        <h1 style="color:var(--red); font-size: 4rem;">UTD</h1>
        <p>ONLY ONE RESOLVE REMAINS.</p>
        <div id="selection-grid" style="display:grid; grid-template-columns: repeat(3, 1fr); gap:10px; margin: 20px;"></div>
        <button id="start-game" class="btn" disabled>INITIATE</button>
    </div>

    <div id="game-ui">
        <header>
            <div>DT: <span id="hp" style="color:var(--red)">999</span></div>
            <div>GOLD: <span id="gold" style="color:var(--gold)">400</span></div>
            <div>LV: <span id="lv">1</span></div>
            <button id="wave-btn" class="btn" style="font-size: 0.8rem;">FIGHT</button>
        </header>

        <canvas id="canvas"></canvas>

        <div id="sidebar">
            <h4 style="text-align:center; margin:0;">SUMMON</h4>
            <div id="shop"></div>
        </div>

        <div id="bottom-panel">
            <div id="frisk-controls">
                <b id="frisk-name">FRISK</b> <span id="frisk-weapon" style="font-size:0.7rem; color: #888;">Stick</span>
                <div style="display:flex; flex-wrap:wrap; margin-top:10px;">
                    <button class="act-btn" onclick="friskCommand('FIGHT')">FIGHT</button>
                    <button class="act-btn" onclick="friskCommand('TALK')">ACT: TALK</button>
                    <button class="act-btn" onclick="friskCommand('FLIRT')">ACT: FLIRT</button>
                    <button class="act-btn" onclick="friskCommand('SPARE')">MERCY</button>
                </div>
                <button id="up-frisk" class="btn" style="width:100%; margin-top:5px; font-size:0.7rem;">LEVEL UP ($100)</button>
            </div>
            <div id="dialogue">
                <span style="color:red">❤</span> <span id="log">A Sans Lost Soul has 1 HP, but can kill you in a millisecond.</span>
            </div>
        </div>
    </div>

<script>
/** * LORE-ACCURATE DATA 
 */
const TOWERS = {
    frisk: { name: "Frisk", cost: 200, color: "#ff00ff", desc: "The only one who can FIGHT effectively." },
    sans:  { name: "Sans",  cost: 600, color: "#008cff", desc: "1 DMG. Inflicts KR (Karma Poison)." },
    papy:  { name: "Papyrus", cost: 150, color: "#fff", desc: "0 DMG. Turns souls Blue (Massive Slow)." },
    undyne:{ name: "Undyne", cost: 400, color: "#00ffff", desc: "10 DMG. Spears 'Mark' enemies for Frisk." }
};

const SOULS = [
    { n: 'Patience',  hp: 100, spd: 0.5, c: '#0ff', perk: 'wait' },
    { n: 'Justice',   hp: 150, spd: 1.2, c: '#ff0', perk: 'stun' },
    { n: 'SANS BOSS', hp: 1,   spd: 3.5, c: '#008cff', perk: 'god' }
];

const WEAPONS = [
    { lv: 1,  n: "Stick", dmg: 50 },
    { lv: 10, n: "Tough Glove", dmg: 300 },
    { lv: 19, n: "True Knife", dmg: 9999 }
];

/** ENGINE **/
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
canvas.width = 850; canvas.height = 450;

let gold = 400, playerLv = 1, wave = 0;
let guardians = [], enemies = [], path = [], bullets = [], vfx = [];
let loadout = [], activePlacement = null, selectedFrisk = null;
let mouse = { tx: 0, ty: 0 };

// Init Menu
const grid = document.getElementById('selection-grid');
Object.keys(TOWERS).forEach(k => {
    const card = document.createElement('div');
    card.className = 'tower-card';
    card.innerHTML = `<b>${TOWERS[k].name}</b><br>$${TOWERS[k].cost}`;
    card.onclick = () => {
        if(loadout.includes(k)) loadout = loadout.filter(i => i!==k);
        else if(loadout.length < 4) loadout.push(k);
        card.classList.toggle('active', loadout.includes(k));
        document.getElementById('start-game').disabled = loadout.length < 4;
    };
    grid.appendChild(card);
});

document.getElementById('start-game').onclick = () => {
    document.getElementById('menu-overlay').style.display = 'none';
    const shopList = document.getElementById('shop');
    loadout.forEach(k => {
        const btn = document.createElement('div');
        btn.className = 'tower-card';
        btn.innerHTML = `<b>${TOWERS[k].name}</b><br>$${TOWERS[k].cost}`;
        btn.onclick = () => {
            activePlacement = TOWERS[k];
            activePlacement.id = k;
            document.querySelectorAll('.tower-card').forEach(c => c.classList.remove('active'));
            btn.classList.add('active');
        };
        shopList.appendChild(btn);
    });
    genPath();
    requestAnimationFrame(loop);
};

function genPath() {
    let x = 0, y = 4;
    while(x < 17) {
        path.push({x, y});
        if(Math.random() < 0.2 && y > 1) y--;
        else if(Math.random() < 0.2 && y < 7) y++;
        else x++;
    }
}

/** TOWER CLASS **/
class Guardian {
    constructor(id, tx, ty) {
        this.id = id; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = (id === 'frisk' ? playerLv : 1);
        this.cd = 0; this.stun = 0;
        this.target = null;
    }
    update() {
        if(this.stun > 0) { this.stun--; return; }
        if(this.cd > 0) this.cd--;
        
        let range = (this.id === 'frisk') ? 120 : 300;
        this.target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < range);

        if(this.target && this.cd <= 0 && this.id !== 'frisk') {
            this.autoAttack(this.target);
            this.cd = (this.id === 'sans') ? 2 : 50;
        }
    }
    autoAttack(t) {
        if(this.id === 'sans') { t.hp -= 0.1; t.kr += 5; } // Debuffed: 0.1 DMG
        if(this.id === 'papy') { t.blue = 60; } // 0 DMG
        if(this.id === 'undyne') { t.hp -= 10; t.marked = true; } // Low DMG
        
        bullets.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, life:5, c:TOWERS[this.id].color});
    }
    draw() {
        ctx.fillStyle = (selectedFrisk === this) ? varColor('--gold') : TOWERS[this.id].color;
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        if(this.id === 'frisk') { ctx.strokeStyle='red'; ctx.strokeRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10); }
    }
}

/** ENEMY CLASS **/
class Enemy {
    constructor(d, wave) {
        this.d = d; this.pIdx = 0;
        this.x = path[0].x*TILE+TILE/2; this.y = path[0].y*TILE+TILE/2;
        this.hp = d.hp * (1 + wave); this.max = this.hp;
        this.spd = d.spd; this.kr = 0; this.blue = 0;
        this.vuln = false; this.marked = false;
    }
    update() {
        if(this.kr > 0) { this.hp -= 0.5; this.kr--; }
        let s = this.blue > 0 ? this.spd * 0.3 : this.spd;
        if(this.blue > 0) this.blue--;

        if(this.d.perk === 'god' && Math.random() < 0.95) this.dodging = true; // Sans boss dodge
        else this.dodging = false;

        let target = path[this.pIdx];
        if(!target) return 'leak';
        let tx = target.x*TILE+TILE/2, ty = target.y*TILE+TILE/2;
        let dist = Math.hypot(tx-this.x, ty-this.y);
        if(dist < s) this.pIdx++;
        else { this.x += ((tx-this.x)/dist)*s; this.y += ((ty-this.y)/dist)*s; }

        return this.hp <= 0 ? 'die' : null;
    }
    draw() {
        ctx.fillStyle = this.vuln ? 'yellow' : this.d.c;
        ctx.beginPath(); ctx.arc(this.x, this.y, 12, 0, Math.PI*2); ctx.fill();
        if(this.marked) { ctx.strokeStyle='cyan'; ctx.stroke(); }
        ctx.fillStyle = 'red'; ctx.fillRect(this.x-15, this.y-22, 30, 4);
        ctx.fillStyle = '#0f0'; ctx.fillRect(this.x-15, this.y-22, 30*(this.hp/this.max), 4);
    }
}

/** LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    path.forEach(p => { ctx.fillStyle = '#111'; ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE); });

    guardians.forEach(g => { g.update(); g.draw(); });

    for(let i=enemies.length-1; i>=0; i--) {
        let res = enemies[i].update();
        if(res === 'leak') { alert("TIMELINE ERASED IN A MILLISECOND."); location.reload(); return; }
        if(res === 'die') { gold += 30; enemies.splice(i,1); }
        else enemies[i].draw();
    }

    bullets.forEach((b,i) => {
        ctx.strokeStyle = b.c; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.life--; if(b.life <= 0) bullets.splice(i,1);
    });

    if(activePlacement) {
        ctx.globalAlpha = 0.3; ctx.fillStyle = activePlacement.color;
        ctx.fillRect(mouse.tx*TILE, mouse.ty*TILE, TILE, TILE);
        ctx.globalAlpha = 1;
    }

    document.getElementById('gold').innerText = gold;
    document.getElementById('lv').innerText = playerLv;
    requestAnimationFrame(loop);
}

/** FRISK COMPLEX ACTIONS **/
function friskCommand(cmd) {
    if(!selectedFrisk || !selectedFrisk.target) { 
        logIt("* But nobody was in range."); return; 
    }
    let t = selectedFrisk.target;
    let weapon = WEAPONS.reverse().find(w => playerLv >= w.lv);

    if(cmd === 'FIGHT') {
        if(t.dodging) { logIt("* MISS"); return; }
        let dmg = weapon.dmg;
        if(t.vuln) dmg *= 10;
        if(t.marked) dmg *= 2;
        t.hp -= dmg;
        logIt(`* You dealt ${Math.floor(dmg)} damage.`);
        playerLv++; // FIGHTing gives EXP
    } else if(cmd === 'TALK') {
        t.blue += 120;
        logIt("* You told the soul a joke. It stopped to listen.");
    } else if(cmd === 'FLIRT') {
        t.vuln = true;
        logIt("* The soul is confused and vulnerable.");
    } else if(cmd === 'SPARE') {
        if(t.hp < t.max * 0.3) {
            gold += 200; // SPAREing gives massive GOLD
            enemies = enemies.filter(e => e !== t);
            logIt("* You Spared the soul. You gained 200G.");
        } else {
            logIt("* The soul is not ready to be spared.");
        }
    }
}

/** UI **/
canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    mouse.tx = Math.floor((e.clientX - r.left)/TILE);
    mouse.ty = Math.floor((e.clientY - r.top)/TILE);
};

canvas.onmousedown = () => {
    if(activePlacement) {
        let occ = guardians.find(g=>g.tx===mouse.tx && g.ty===mouse.ty) || !path.every(p => p.x!==mouse.tx || p.y!==mouse.ty);
        if(!occ && gold >= activePlacement.cost) {
            gold -= activePlacement.cost; guardians.push(new Guardian(activePlacement.id, mouse.tx, mouse.ty));
            activePlacement = null; document.querySelectorAll('.tower-card').forEach(b => b.classList.remove('active'));
        }
    } else {
        let g = guardians.find(x => x.tx === mouse.tx && x.ty === mouse.ty);
        if(g && g.id === 'frisk') {
            selectedFrisk = g;
            document.getElementById('frisk-controls').style.display='block';
            let weapon = WEAPONS.find(w => playerLv >= w.lv);
            document.getElementById('frisk-weapon').innerText = weapon.n;
        } else {
            selectedFrisk = null;
            document.getElementById('frisk-controls').style.display='none';
        }
    }
};

document.getElementById('wave-btn').onclick = () => {
    wave++;
    let count = 3 + wave;
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            let type = SOULS[Math.floor(Math.random()*2)];
            if(wave % 5 === 0 && i === 0) type = SOULS[2]; // Sans boss
            enemies.push(new Enemy(type, wave));
        }, i * 1500);
    }
};

function logIt(txt) { document.getElementById('log').innerText = txt; }
function varColor(n) { return getComputedStyle(document.documentElement).getPropertyValue(n); }
</script>
</body>
</html>
