<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UNDERTALE TOWER DEFENSE: MULTIVERSE BREACH</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');

        :root {
            --ut-red: #ff0000;
            --ut-white: #ffffff;
            --ut-gold: #ffcc00;
            --ut-bg: #000000;
            --ut-border: #ffffff;
            --ut-blue: #008cff;
        }

        body {
            background: var(--ut-bg);
            color: var(--ut-white);
            font-family: 'Courier Prime', monospace;
            margin: 0;
            display: grid;
            grid-template-columns: 1fr 300px;
            grid-template-rows: 80px 1fr 150px;
            height: 100vh;
            overflow: hidden;
            user-select: none;
        }

        /* SCREEN OVERLAYS */
        #menu-overlay {
            position: fixed;
            inset: 0;
            background: black;
            z-index: 1000;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            border: 10px double white;
            margin: 20px;
        }

        .hidden { display: none !important; }

        /* SELECTION GRID */
        .selection-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            margin: 20px;
        }

        .char-card {
            border: 2px solid white;
            padding: 10px;
            text-align: center;
            cursor: pointer;
            transition: 0.2s;
            width: 150px;
        }

        .char-card:hover { background: #333; color: var(--ut-gold); }
        .char-card.selected { border-color: var(--ut-gold); color: var(--ut-gold); box-shadow: 0 0 10px var(--ut-gold); }

        /* UI SECTIONS */
        header {
            grid-column: 1 / 3;
            border-bottom: 4px solid white;
            display: flex;
            align-items: center;
            justify-content: space-around;
            padding: 0 20px;
        }

        .stat-val { color: var(--ut-gold); font-weight: bold; font-size: 1.5rem; }

        #game-container {
            grid-column: 1;
            grid-row: 2;
            display: flex;
            justify-content: center;
            align-items: center;
            background: #050505;
            position: relative;
        }

        aside {
            grid-column: 2;
            grid-row: 2;
            border-left: 4px solid white;
            background: #000;
            padding: 20px;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        footer {
            grid-column: 1 / 3;
            grid-row: 3;
            border-top: 4px solid white;
            padding: 20px;
            display: flex;
            gap: 20px;
        }

        /* BUTTONS */
        .ut-btn {
            background: black;
            border: 2px solid white;
            color: white;
            padding: 10px;
            cursor: pointer;
            font-family: inherit;
            text-transform: uppercase;
        }

        .ut-btn:hover { background: white; color: black; }
        .ut-btn:disabled { opacity: 0.3; cursor: not-allowed; }

        .shop-item {
            border: 2px solid white;
            padding: 10px;
            text-align: center;
            cursor: pointer;
        }
        .shop-item.active { border-color: var(--ut-gold); color: var(--ut-gold); }

        #dialogue-box {
            flex: 1;
            border: 2px solid white;
            padding: 15px;
            font-size: 1.2rem;
            position: relative;
        }

        canvas { image-rendering: pixelated; }

        .heart { color: var(--ut-red); display: inline-block; margin-right: 10px; }
    </style>
</head>
<body>

    <div id="menu-overlay">
        <h1 style="font-size: 4rem; letter-spacing: 10px;">UTD</h1>
        <p style="color: #888;">MULTIVERSE BREACH: Select 4 Guardians</p>
        <div class="selection-grid" id="select-grid"></div>
        <button class="ut-btn" id="start-btn" style="width: 200px; height: 50px;" disabled>INITIATE</button>
    </div>

    <header>
        <div>CORE HP: <span id="hp-val" class="stat-val">20</span></div>
        <div style="color: var(--ut-red); font-weight: bold;">LOVE (LV): <span id="lv-val">1</span></div>
        <div>GOLD (G): <span id="money-val" class="stat-val">500</span></div>
        <button class="ut-btn" id="next-wave-btn">SPARE/FIGHT</button>
    </header>

    <div id="game-container">
        <canvas id="gameCanvas"></canvas>
    </div>

    <aside>
        <h3 style="text-align: center; border-bottom: 2px solid white;">SUMMON</h3>
        <div id="shop-container" style="display: flex; flex-direction: column; gap: 10px;"></div>
    </aside>

    <footer>
        <div id="upgrade-panel" style="width: 250px; border-right: 2px solid white; padding-right: 20px; display: none;">
            <div id="up-name" style="color: var(--ut-gold); font-weight: bold;">Character</div>
            <div id="up-stats" style="font-size: 0.8rem; margin: 10px 0;"></div>
            <button class="ut-btn" id="up-btn" style="width: 100%;">LEVEL UP</button>
        </div>
        <div id="dialogue-box">
            <span class="heart">❤</span> <span id="dialogue-text">Welcome to the Multiverse. Select a guardian to begin.</span>
        </div>
    </footer>

<script>
/** * CHARACTERS & DATA
 */
const CHARS = [
    { id: 'flowey', name: 'Flowey', cost: 100, range: 150, dmg: 15, rate: 30, color: '#ffff00', desc: 'Cheap, fast friendly pellets.', evo: 'Omega Flowey' },
    { id: 'toriel', name: 'Toriel', cost: 200, range: 180, dmg: 40, rate: 80, color: '#a020f0', desc: 'Fire magic AOE damage.', evo: 'Queen of Fire' },
    { id: 'papyrus', name: 'Papyrus', cost: 150, range: 120, dmg: 10, rate: 50, color: '#ffffff', desc: 'Blue Bones: Slows enemies.', evo: 'Cool Skeleton' },
    { id: 'undyne', name: 'Undyne', cost: 300, range: 250, dmg: 60, rate: 60, color: '#00ffff', desc: 'Pierce spears through multiple souls.', evo: 'The Undying' },
    { id: 'mettaton', name: 'Mettaton', cost: 350, range: 200, dmg: 10, rate: 5, color: '#ff69b4', desc: 'Rapid disco lasers.', evo: 'Mettaton NEO' },
    { id: 'sans', name: 'Sans', cost: 500, range: 300, dmg: 5, rate: 1, color: '#008cff', desc: 'Gaster Blasters & Karma (DOT).', evo: 'Last Breath' },
    { id: 'asgore', name: 'Asgore', cost: 450, range: 150, dmg: 150, rate: 120, color: '#ffa500', desc: 'Huge Trident swing AOE.', evo: 'Hyperdeath' },
    { id: 'ink', name: 'Ink Sans', cost: 600, range: 350, dmg: 50, rate: 150, color: '#ffffff', desc: 'Summons AU Sans variants on path.', evo: 'Creative God' },
    { id: 'frisk', name: 'Frisk', cost: 250, range: 100, dmg: 20, rate: 40, color: '#ff0000', desc: 'Determined: Has a chance to recover HP.', evo: 'Chara' }
];

/** ENGINE SETUP **/
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const TILE = 50;
const COLS = 16, ROWS = 10;
canvas.width = COLS * TILE;
canvas.height = ROWS * TILE;

let money = 600, lives = 20, lv = 1, wave = 0;
let selectedIds = [], towers = [], enemies = [], path = [], map = [], projectiles = [], auEchoes = [];
let waveActive = false, activeShopSelection = null, selectedInstance = null;
let mouse = { x: 0, y: 0, tx: 0, ty: 0 };

/** MENU LOGIC **/
const selectGrid = document.getElementById('select-grid');
CHARS.forEach(c => {
    const card = document.createElement('div');
    card.className = 'char-card';
    card.innerHTML = `<b>${c.name}</b><br><small>$${c.cost}</small>`;
    card.onclick = () => {
        if(selectedIds.includes(c.id)) {
            selectedIds = selectedIds.filter(i => i !== c.id);
            card.classList.remove('selected');
        } else if(selectedIds.length < 4) {
            selectedIds.push(c.id);
            card.classList.add('selected');
        }
        document.getElementById('start-btn').disabled = selectedIds.length !== 4;
    };
    selectGrid.appendChild(card);
});

document.getElementById('start-btn').onclick = () => {
    document.getElementById('menu-overlay').classList.add('hidden');
    initGame();
};

function initGame() {
    // Populate Shop
    const shop = document.getElementById('shop-container');
    selectedIds.forEach(id => {
        const c = CHARS.find(x => x.id === id);
        const item = document.createElement('div');
        item.className = 'shop-item';
        item.innerHTML = `<b>${c.name}</b><br>$${c.cost}`;
        item.onclick = (e) => {
            e.stopPropagation();
            activeShopSelection = c;
            document.querySelectorAll('.shop-item').forEach(i => i.classList.remove('active'));
            item.classList.add('active');
            setDialogue(`${c.name}: ${c.desc}`);
        };
        shop.appendChild(item);
    });

    generateMap();
    requestAnimationFrame(loop);
}

function generateMap() {
    map = Array(ROWS).fill().map(() => Array(COLS).fill(1));
    let x = 0, y = 5;
    path = [];
    while(x < COLS) {
        path.push({x, y}); map[y][x] = 0;
        let r = Math.random();
        if(r < 0.3 && y > 1 && map[y-1][x] !== 0) y--;
        else if(r < 0.6 && y < ROWS - 2 && map[y+1][x] !== 0) y++;
        else x++;
    }
}

/** CLASSES **/
class Tower {
    constructor(type, tx, ty) {
        this.type = type; this.tx = tx; this.ty = ty;
        this.x = tx*TILE + TILE/2; this.y = ty*TILE + TILE/2;
        this.lv = 1; this.dmg = type.dmg; this.range = type.range;
        this.cd = 0;
    }
    update() {
        if(this.cd > 0) this.cd--;
        
        // Ink Sans Special Spawner
        if(this.type.id === 'ink' && this.cd <= 0) {
            this.spawnAUSans();
            this.cd = this.type.rate;
        }

        let target = enemies.find(e => Math.hypot(e.x-this.x, e.y-this.y) < this.range);
        if(target && this.cd <= 0) {
            this.fire(target);
            this.cd = this.type.rate;
        }
    }
    spawnAUSans() {
        const variants = ['#f00', '#0ff', '#f0f'];
        auEchoes.push({
            pIdx: path.length - 1,
            x: path[path.length-1].x*TILE+TILE/2,
            y: path[path.length-1].y*TILE+TILE/2,
            color: variants[Math.floor(Math.random()*3)],
            dmg: this.dmg
        });
    }
    fire(target) {
        if(this.type.id === 'papyrus') target.slow = 60;
        if(this.type.id === 'frisk' && Math.random() < 0.05) {
            lives = Math.min(20, lives + 1);
            setDialogue("* Determination restores 1 HP.");
        }
        
        ctx.strokeStyle = this.type.color; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(this.x, this.y); ctx.lineTo(target.x, target.y); ctx.stroke();
        target.hp -= this.dmg;
    }
    draw() {
        ctx.fillStyle = this.type.color;
        ctx.fillRect(this.tx*TILE+5, this.ty*TILE+5, TILE-10, TILE-10);
        ctx.fillStyle = 'white'; ctx.font = '10px Courier';
        ctx.fillText("LV"+this.lv, this.x-10, this.y+5);
    }
}

class Enemy {
    constructor(wave) {
        this.pIdx = 0;
        this.x = path[0].x*TILE+TILE/2; this.y = path[0].y*TILE+TILE/2;
        this.maxHp = 40 + (wave * 25);
        this.hp = this.maxHp;
        this.speed = 1.5 + (wave * 0.1);
        this.slow = 0;
    }
    update() {
        if(this.slow > 0) this.slow--;
        let s = this.slow > 0 ? this.speed * 0.5 : this.speed;
        let t = path[this.pIdx];
        let tx = t.x*TILE+TILE/2, ty = t.y*TILE+TILE/2;
        let d = Math.hypot(tx-this.x, ty-this.y);
        if(d < s) {
            this.pIdx++;
            if(this.pIdx >= path.length) return 'leak';
        } else {
            this.x += ((tx-this.x)/d)*s; this.y += ((ty-this.y)/d)*s;
        }
        return this.hp <= 0 ? 'die' : null;
    }
    draw() {
        ctx.fillStyle = '#fff';
        ctx.beginPath(); ctx.moveTo(this.x, this.y-10); ctx.lineTo(this.x+10, this.y+10); ctx.lineTo(this.x-10, this.y+10); ctx.closePath(); ctx.fill();
        ctx.fillStyle = 'red'; ctx.fillRect(this.x-15, this.y-20, 30, 4);
        ctx.fillStyle = 'lime'; ctx.fillRect(this.x-15, this.y-20, 30*(this.hp/this.maxHp), 4);
    }
}

/** LOOP **/
function loop() {
    ctx.clearRect(0,0,canvas.width,canvas.height);
    
    // Road
    ctx.fillStyle = '#111';
    path.forEach(p => ctx.fillRect(p.x*TILE, p.y*TILE, TILE, TILE));

    towers.forEach(t => { t.update(); t.draw(); });

    // AU Echoes logic (Ink Sans)
    for(let i=auEchoes.length-1; i>=0; i--) {
        let e = auEchoes[i];
        let t = path[e.pIdx];
        let tx = t.x*TILE+TILE/2, ty = t.y*TILE+TILE/2;
        let d = Math.hypot(tx-e.x, ty-e.y);
        if(d < 5) {
            e.pIdx--;
            if(e.pIdx < 0) { auEchoes.splice(i,1); continue; }
        }
        e.x += ((tx-e.x)/d)*4; e.y += ((ty-e.y)/d)*4;
        ctx.fillStyle = e.color; ctx.beginPath(); ctx.arc(e.x, e.y, 8, 0, Math.PI*2); ctx.fill();
        
        enemies.forEach(en => {
            if(Math.hypot(en.x-e.x, en.y-e.y) < 30) en.hp -= 1;
        });
    }

    for(let i=enemies.length-1; i>=0; i--) {
        let res = enemies[i].update();
        if(res === 'leak') { lives--; enemies.splice(i,1); setDialogue("* A soul escaped your grasp."); }
        else if(res === 'die') { money += 25; enemies.splice(i,1); }
        else enemies[i].draw();
    }

    // Overlays
    if(activeShopSelection) {
        ctx.globalAlpha = 0.3; ctx.fillStyle = activeShopSelection.color;
        ctx.fillRect(mouse.tx*TILE, mouse.ty*TILE, TILE, TILE);
        ctx.beginPath(); ctx.arc(mouse.tx*TILE+TILE/2, mouse.ty*TILE+TILE/2, activeShopSelection.range, 0, Math.PI*2);
        ctx.strokeStyle = 'white'; ctx.stroke(); ctx.globalAlpha = 1;
    }

    if(selectedInstance) {
        ctx.strokeStyle = 'gold'; ctx.beginPath();
        ctx.arc(selectedInstance.x, selectedInstance.y, selectedInstance.range, 0, Math.PI*2); ctx.stroke();
    }

    updateUI();
    if(lives > 0) requestAnimationFrame(loop);
    else alert("GAME OVER - YOU WERE SPARED NO LONGER.");
}

/** INTERACTION **/
canvas.onmousemove = (e) => {
    let r = canvas.getBoundingClientRect();
    mouse.x = e.clientX - r.left; mouse.y = e.clientY - r.top;
    mouse.tx = Math.floor(mouse.x/TILE); mouse.ty = Math.floor(mouse.y/TILE);
};

canvas.onmousedown = () => {
    if(activeShopSelection) {
        if(map[mouse.ty][mouse.tx] === 1 && money >= activeShopSelection.cost) {
            money -= activeShopSelection.cost;
            towers.push(new Tower(activeShopSelection, mouse.tx, mouse.ty));
            activeShopSelection = null;
            document.querySelectorAll('.shop-item').forEach(i => i.classList.remove('active'));
        }
    } else {
        let t = towers.find(t => t.tx === mouse.tx && t.ty === mouse.ty);
        if(t) { selectedInstance = t; showUpgrade(t); }
        else { selectedInstance = null; document.getElementById('upgrade-panel').style.display='none'; }
    }
};

function showUpgrade(t) {
    const p = document.getElementById('upgrade-panel'); p.style.display='block';
    const cost = 100 * t.lv;
    document.getElementById('up-name').innerText = `${t.type.name} (LV ${t.lv})`;
    document.getElementById('up-stats').innerText = `DMG: ${t.dmg} | RNG: ${t.range}`;
    const btn = document.getElementById('up-btn');
    btn.innerText = `INCREASE LV ($${cost})`;
    btn.onclick = () => {
        if(money >= cost) {
            money -= cost; t.lv++; t.dmg += 15; t.range += 10;
            if(t.lv === 5) setDialogue(`* ${t.type.name} is evolving!`);
            showUpgrade(t);
        }
    };
}

document.getElementById('next-wave-btn').onclick = () => {
    if(waveActive) return;
    wave++; waveActive = true;
    setDialogue(`* Wave ${wave} is approaching. Stay determined.`);
    let count = 5 + wave * 2;
    for(let i=0; i<count; i++) {
        setTimeout(() => {
            enemies.push(new Enemy(wave));
            if(i === count - 1) {
                let check = setInterval(() => {
                    if(enemies.length === 0) { waveActive = false; clearInterval(check); }
                }, 500);
            }
        }, i * 800);
    }
};

function setDialogue(txt) { document.getElementById('dialogue-text').innerText = txt; }
function updateUI() {
    document.getElementById('hp-val').innerText = lives;
    document.getElementById('money-val').innerText = money;
    document.getElementById('lv-val').innerText = wave;
}

</script>
</body>
</html>
