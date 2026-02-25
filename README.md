<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: MULTIVERSE BREACH - ULTIMATE</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');

        :root {
            --ut-red: #ff0000;
            --ut-white: #ffffff;
            --ut-gold: #ffcc00;
            --ut-bg: #000000;
            --ut-blue: #008cff;
        }

        body {
            background: #050505;
            color: white;
            font-family: 'Courier Prime', monospace;
            margin: 0;
            height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            align-items: center;
        }

        /* OVERLAYS */
        #menu-overlay {
            position: fixed;
            inset: 0;
            background: black;
            z-index: 2000;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            border: 8px double white;
            margin: 20px;
        }

        /* MAIN LAYOUT GRID */
        #main-container {
            display: grid;
            grid-template-columns: 850px 250px; /* Board | Shop */
            grid-template-rows: auto 150px;    /* Board / Bottom Bar */
            gap: 10px;
            padding: 10px;
            border: 4px solid white;
            margin-top: 10px;
            background: black;
        }

        /* CANVAS AREA */
        #canvas-wrapper {
            position: relative;
            grid-column: 1;
            grid-row: 1;
            border: 2px solid #333;
            overflow: hidden;
        }

        /* SHOP AREA (Right Side) */
        #shop-panel {
            grid-column: 2;
            grid-row: 1;
            border: 2px solid white;
            background: #0a0a0a;
            display: flex;
            flex-direction: column;
            padding: 10px;
            gap: 8px;
        }

        .shop-item {
            border: 2px solid white;
            padding: 8px;
            cursor: pointer;
            font-size: 0.9rem;
            text-align: center;
            transition: 0.1s;
        }
        .shop-item:hover { background: white; color: black; }
        .shop-item.active { border-color: var(--ut-gold); color: var(--ut-gold); background: #222; }

        /* BOTTOM BAR (Upgrades & Dialogue) */
        #bottom-bar {
            grid-column: 1 / 3;
            grid-row: 2;
            border: 2px solid white;
            display: flex;
            padding: 15px;
            gap: 20px;
        }

        #dialogue-box {
            flex: 1;
            font-size: 1.2rem;
            border-left: 2px solid white;
            padding-left: 20px;
        }

        #upgrade-details {
            width: 300px;
            display: none;
        }

        .ut-btn {
            background: black;
            border: 2px solid white;
            color: white;
            padding: 8px;
            cursor: pointer;
            font-family: inherit;
            text-transform: uppercase;
        }
        .ut-btn:hover { color: var(--ut-gold); border-color: var(--ut-gold); }

        .stat-val { color: var(--ut-gold); }
        .heart { color: var(--ut-red); animation: pulse 0.6s infinite alternate; }
        @keyframes pulse { from { transform: scale(1); } to { transform: scale(1.1); } }
        
        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="menu-overlay">
        <h1 style="font-size: 3rem; color: var(--ut-gold);">UNDERTALE TD</h1>
        <p>MULTIVERSE BREACH: INITIALIZING...</p>
        <div id="select-grid" style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin: 20px;"></div>
        <button id="btn-start" class="ut-btn" style="width: 200px;" disabled>START BATTLE</button>
    </div>

    <div style="width: 1110px; display: flex; justify-content: space-between; padding: 10px 0;">
        <div>HP: <span id="hp-val" class="stat-val">20</span></div>
        <div>GOLD: <span id="money-val" class="stat-val">600</span></div>
        <div>LV: <span id="wave-val" class="stat-val">1</span></div>
        <button id="btn-wave" class="ut-btn">SPARE / FIGHT</button>
    </div>

    <div id="main-container">
        <div id="canvas-wrapper">
            <canvas id="gameCanvas"></canvas>
        </div>

        <div id="shop-panel">
            <h4 style="margin: 0; text-align: center; color: var(--ut-gold);">SUMMONS</h4>
            <div id="shop-list"></div>
        </div>

        <div id="bottom-bar">
            <div id="upgrade-details">
                <div id="up-name" style="color: var(--ut-gold); font-size: 1.2rem;">SANS</div>
                <div id="up-stats" style="font-size: 0.8rem; margin: 5px 0;">DMG: 5 | RNG: 300</div>
                <button id="btn-up" class="ut-btn" style="width: 100%;">LEVEL UP</button>
            </div>
            <div id="dialogue-box">
                <span class="heart">❤</span> <span id="dialogue-text">Stay Determined. Pick your guardians on the right.</span>
            </div>
        </div>
    </div>

<script>
/** * DATA & CONFIGURATION
 */
const CHARS = [
    { id: 'sans', name: 'Sans', cost: 500, range: 300, dmg: 5, rate: 2, color: '#008cff', desc: 'Fires Gaster Blasters. Pure Karma.', type: 'blaster' },
    { id: 'undyne', name: 'Undyne', cost: 300, range: 200, dmg: 40, rate: 45, color: '#00ffff', desc: 'Rapid spear throws.', type: 'spear' },
    { id: 'flowey', name: 'Flowey', cost: 100, range: 150, dmg: 15, rate: 25, color: '#ffff00', desc: 'Friendly pellets.', type: 'pellet' },
    { id: 'toriel', name: 'Toriel', cost: 250, range: 160, dmg: 50, rate: 80, color: '#a020f0', desc: 'Fire magic AOE.', type: 'fire' },
    { id: 'mettaton', name: 'Mettaton', cost: 350, range: 180, dmg: 10, rate: 5, color: '#ff69b4', desc: 'Disco Laser frenzy.', type: 'laser' },
    { id: 'asgore', name: 'Asgore', cost: 450, range: 140, dmg: 120, rate: 100, color: '#ffa500', desc: 'Trident swing.', type: 'melee' },
    { id: 'ink', name: 'Ink Sans', cost: 600, range: 400, dmg: 30, rate: 120, color: '#ffffff', desc: 'Paints AU clones.', type: 'paint' },
    { id: 'papyrus', name: 'Papyrus', cost: 200, range: 150, dmg: 20, rate: 40, color: '#ffffff', desc: 'Blue bones slow enemies.', type: 'bone' },
    { id: 'frisk', name: 'Frisk', cost: 300, range: 100, dmg: 25, rate: 30, color: '#ff0000', desc: 'Has a 10% chance to heal HP.', type: 'save' }
];

const BOARD_W = 850, BOARD_H = 450;
const TILE = 50;
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
canvas.width = BOARD_W; canvas.height = BOARD_H;

/** STATE **/
let money = 600, lives = 20, wave = 0;
let towers = [], enemies = [], path = [], particles = [], beams = [], floatingText = [];
let selectedTowerIds = [], waveActive = false, activePlacement = null, selectedInst = null;
let shake = 0;
let mouse = { x:0, y:0, tx:0, ty:0 };

/** INITIALIZATION **/
const selectGrid = document.getElementById('select-grid');
CHARS.forEach(c => {
    const card = document.createElement('div');
    card.style = "border:1px solid white; padding:10px; cursor:pointer; text-align:center;";
    card.innerHTML = `<b>${c.name}</b><br>$${c.cost}`;
    card.onclick = () => {
        if(selectedTowerIds.includes(c.id)) {
            selectedTowerIds = selectedTowerIds.filter(i => i !== c.id);
            card.style.borderColor = 'white';
        } else if(selectedTowerIds.length < 4) {
            selectedTowerIds.push(c.id);
            card.style.borderColor = 'var(--ut-gold)';
        }
        document.getElementById('btn-start').disabled = selectedTowerIds.length !== 4;
    };
    selectGrid.appendChild(card);
});

document.getElementById('btn-start').onclick = () => {
    document.getElementById('menu-overlay').classList.add('hidden');
    initGame();
};

function initGame() {
    const shopList = document.getElementById('shop-list');
    selectedTowerIds.forEach(id => {
        const c = CHARS.find(x => x.id === id);
        const btn = document.createElement('div');
        btn.className = 'shop-item';
        btn.innerHTML = `<b>${c.name}</b><br>$${c.cost}`;
        btn.onclick = () => {
            activePlacement = c;
            document.querySelectorAll('.shop-item').forEach(i => i.classList.remove('active'));
            btn.classList.add('active');
        };
        shopList.appendChild(btn);
    });
    generateMap();
    requestAnimationFrame(loop);
}

function generateMap() {
    let x = 0, y = 4;
    path = [];
    while(x * TILE < BOARD_W) {
        path.push({x, y});
        let r = Math.random();
        if(r < 0.2 && y > 1) y--;
        else if(r < 0.4 && y < 7) y++;
        else x++;
    }
}

/** CLASSES **/
class Tower {
    constructor(type, tx, ty) {
        this.type = type; this.tx = tx; this.ty = ty;
        this.x = tx*TILE+TILE/2; this.y = ty*TILE+TILE/2;
        this.lv = 1; this.cd = 0; this.dmg = type.dmg; this.range = type.range;
        this.frame = 0;
    }
    update() {
        this.frame += 0.05;
        if(this.cd > 0) this.cd--;
        let target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < this.range);
        if(target && this.cd <= 0) {
            this.fire(target);
            this.cd = Math.max(2, this.type.rate - (this.lv * 2));
        }
    }
    fire(t) {
        shake = 3;
        if(this.type.type === 'blaster') {
            beams.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, alpha: 1, color: '#fff', wide: 15});
        }
        if(this.type.type === 'spear') {
            beams.push({x1:this.x, y1:this.y, x2:t.x, y2:t.y, alpha: 1, color: '#0ff', wide: 3});
        }
        if(this.type.id === 'frisk' && Math.random() < 0.1) {
            lives = Math.min(20, lives + 1);
            floatingText.push({x:this.x, y:this.y, txt: '❤SAVE', alpha:1});
        }
        t.hp -= this.dmg;
    }
    draw() {
        let bob = Math.sin(this.frame) * 5;
        ctx.fillStyle = this.lv >= 5 ? varColor('--ut-gold') : this.type.color;
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5 + bob, TILE-10, TILE-10);
        ctx.fillStyle = 'white'; ctx.font = '10px Courier';
        ctx.fillText("LV"+this.lv, this.x-10, this.y + bob + 5);
    }
}

class Enemy {
    constructor(wave) {
        this.pIdx = 0;
        this.x = path[0].x*TILE+TILE/2; this.y = path[0].y*TILE+TILE/2;
        this.maxHp = 50 + (wave * 40); this.hp = this.maxHp;
        this.speed = 1.5 + (wave * 0.1);
    }
    update() {
        let t = path[this.pIdx];
        if(!t) return 'leak';
        let tx = t.x*TILE+TILE/2, ty = t.y*TILE+TILE/2;
        let d = Math.hypot(tx-this.x, ty-this.y);
        if(d < this.speed) {
            this.pIdx++;
        } else {
            this.x += ((tx-this.x)/d)*this.speed; this.y += ((ty-this.y)/d)*this.speed;
        }
        return this.hp <= 0 ? 'die' : null;
    }
    draw() {
        ctx.fillStyle = 'white';
        ctx.beginPath(); ctx.arc(this.x, this.y, 10, 0, Math.PI*2); ctx.fill();
        ctx.fillStyle = 'red'; ctx.fillRect(this.x-15, this.y-20, 30, 4);
        ctx.fillStyle = 'lime'; ctx.fillRect(this.x-15, this.y-20, 30*(this.hp/this.maxHp), 4);
    }
}

/** MAIN LOOP **/
function loop() {
    // Screen Shake logic
    ctx.save();
    if(shake > 0) {
        ctx.translate(Math.random()*shake - shake/2, Math.random()*shake - shake/2);
        shake *= 0.9;
    }

    ctx.fillStyle = '#000'; ctx.fillRect(0,0,BOARD_W,BOARD_H);
    
    // Draw Path
    ctx.fillStyle = '#111';
    path.forEach(p => ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE));

    towers.forEach(t => { t.update(); t.draw(); });

    for(let i=enemies.length-1; i>=0; i--) {
        let res = enemies[i].update();
        if(res === 'leak') { lives--; enemies.splice(i,1); }
        else if(res === 'die') {
            money += 30;
            spawnParticles(enemies[i].x, enemies[i].y);
            enemies.splice(i,1);
        } else { enemies[i].draw(); }
    }

    // Beams (Animations)
    for(let i=beams.length-1; i>=0; i--) {
        let b = beams[i];
        ctx.strokeStyle = b.color;
        ctx.lineWidth = b.wide * b.alpha;
        ctx.globalAlpha = b.alpha;
        ctx.beginPath(); ctx.moveTo(b.x1, b.y1); ctx.lineTo(b.x2, b.y2); ctx.stroke();
        b.alpha -= 0.1;
        if(b.alpha <= 0) beams.splice(i,1);
    }
    ctx.globalAlpha = 1;

    // Particles & Text
    particles.forEach((p,i) => {
        p.x += p.vx; p.y += p.vy; p.a -= 0.02;
        ctx.fillStyle = `rgba(255,255,255,${p.a})`; ctx.fillRect(p.x, p.y, 2, 2);
        if(p.a <= 0) particles.splice(i,1);
    });

    floatingText.forEach((f,i) => {
        ctx.fillStyle = `rgba(255, 204, 0, ${f.alpha})`; ctx.fillText(f.txt, f.x, f.y);
        f.y -= 1; f.alpha -= 0.02; if(f.alpha <= 0) floatingText.splice(i,1);
    });

    // Placement UI
    if(activePlacement) {
        let valid = !towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty) && !path.find(p=>p.x===mouse.tx && p.y===mouse.ty);
        ctx.globalAlpha = 0.4;
        ctx.fillStyle = valid ? activePlacement.color : 'red';
        ctx.fillRect(mouse.tx*TILE, mouse.ty*TILE, TILE, TILE);
        ctx.beginPath(); ctx.arc(mouse.tx*TILE+TILE/2, mouse.ty*TILE+TILE/2, activePlacement.range, 0, Math.PI*2);
        ctx.strokeStyle = 'white'; ctx.stroke();
        ctx.globalAlpha = 1;
    }

    ctx.restore();
    updateUI();
    if(lives > 0) requestAnimationFrame(loop);
}

/** INTERACTION **/
canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    mouse.x = e.clientX - r.left; mouse.y = e.clientY - r.top;
    mouse.tx = Math.floor(mouse.x/TILE); mouse.ty = Math.floor(mouse.y/TILE);
};

canvas.onmousedown = () => {
    if(activePlacement) {
        let valid = !towers.find(t=>t.tx===mouse.tx && t.ty===mouse.ty) && !path.find(p=>p.x===mouse.tx && p.y===mouse.ty);
        if(valid && money >= activePlacement.cost) {
            money -= activePlacement.cost;
            towers.push(new Tower(activePlacement, mouse.tx, mouse.ty));
            activePlacement = null;
            document.querySelectorAll('.shop-item').forEach(i => i.classList.remove('active'));
        }
    } else {
        let t = towers.find(t => t.tx === mouse.tx && t.ty === mouse.ty);
        if(t) { selectedInst = t; showUpgrade(t); }
        else { selectedInst = null; document.getElementById('upgrade-details').style.display='none'; }
    }
};

function showUpgrade(t) {
    const box = document.getElementById('upgrade-details');
    box.style.display = 'block';
    const cost = 100 * t.lv;
    document.getElementById('up-name').innerText = `${t.type.name} [LV ${t.lv}]`;
    document.getElementById('up-stats').innerText = `DMG: ${t.dmg} | RNG: ${t.range}`;
    const btn = document.getElementById('btn-up');
    btn.innerText = `LEVEL UP ($${cost})`;
    btn.onclick = () => {
        if(money >= cost) {
            money -= cost; t.lv++; t.dmg += 20; t.range += 15;
            if(t.lv === 5) setDialogue(`* ${t.type.name} has ascended to their true form.`);
            showUpgrade(t);
        }
    };
}

document.getElementById('btn-wave').onclick = () => {
    if(waveActive) return;
    wave++; waveActive = true;
    let count = 5 + wave * 2;
    setDialogue(`* Wave ${wave} begins. Stay Determined.`);
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            enemies.push(new Enemy(wave));
            if(i === count-1) {
                let check = setInterval(() => {
                    if(enemies.length === 0) { waveActive = false; clearInterval(check); }
                }, 500);
            }
        }, i * 700);
    }
};

function spawnParticles(x, y) {
    for(let i=0; i<8; i++) particles.push({x, y, vx:Math.random()*4-2, vy:Math.random()*4-2, a:1});
}

function setDialogue(txt) { document.getElementById('dialogue-text').innerText = txt; }
function varColor(n) { return getComputedStyle(document.documentElement).getPropertyValue(n); }
function updateUI() {
    document.getElementById('hp-val').innerText = lives;
    document.getElementById('money-val').innerText = money;
    document.getElementById('wave-val').innerText = wave;
}
</script>
</body>
</html>
