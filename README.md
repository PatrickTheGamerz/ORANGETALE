<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: DUST PARADOX (1:1 MECHANICS)</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --blue: #3b82f6; --purple: #d946ef; --dust: #8a2be2; }
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; height: 100vh; display: flex; align-items: center; justify-content: center; }
        
        #game-frame { width: 1000px; height: 750px; border: 4px solid #fff; position: relative; display: grid; grid-template-columns: 650px 1fr; grid-template-rows: 80px 1fr 220px; gap: 10px; padding: 10px; background: #000; }

        /* CHARACTER BOX & ENEMY */
        #arena { grid-column: 1; grid-row: 2; position: relative; border: 2px solid #555; overflow: hidden; background: #000; }
        #dust-sans { position: absolute; top: 20px; left: 50%; transform: translateX(-50%); text-align: center; }
        #enemy-sprite { font-size: 80px; filter: drop-shadow(0 0 10px var(--dust)); }
        
        /* THE SOUL BOX (1:1 UNDER TALE SCALE) */
        #battle-box { position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%); width: 300px; height: 160px; border: 5px solid #fff; background: #000; transition: width 0.3s, height 0.3s; }
        #soul { width: 16px; height: 16px; position: absolute; z-index: 100; pointer-events: none; }
        #soul svg { width: 100%; height: 100%; }

        /* DASHBOARD */
        #dashboard { grid-column: 1 / 3; grid-row: 3; display: flex; flex-direction: column; gap: 10px; border-top: 2px solid #fff; padding-top: 10px; }
        #stat-bar { display: flex; gap: 30px; font-size: 24px; font-weight: bold; }
        #menu-buttons { display: flex; justify-content: space-between; padding: 0 20px; }
        .menu-btn { width: 110px; height: 42px; border: 3px solid #ffaa00; background: #000; color: #ffaa00; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 20px; cursor: pointer; }
        .menu-btn.active { background: #ffaa00; color: #000; box-shadow: 0 0 10px #ffaa00; }

        /* SIDEBAR (THE COUNCIL) */
        #sidebar { grid-column: 2; grid-row: 2; border-left: 2px solid #fff; padding: 10px; display: flex; flex-direction: column; gap: 10px; }
        #chat { flex: 1; border: 1px solid #444; background: #050505; font-size: 14px; overflow-y: auto; padding: 5px; }
        .influence-row { font-size: 12px; margin-bottom: 5px; }
        .bar-bg { width: 100%; height: 6px; background: #222; border: 1px solid #444; }
        .bar-fill { height: 100%; transition: width 0.3s; }

        /* VFX */
        .blaster-beam { position: absolute; background: #fff; box-shadow: 0 0 15px #fff; z-index: 50; }
        .bone { position: absolute; background: #fff; z-index: 40; }
    </style>
</head>
<body>

<div id="game-frame">
    <header style="grid-column: 1 / 3; display: flex; justify-content: space-around; font-size: 20px;">
        <div id="current-controller">OWNER: PLAYER</div>
        <div>LV <span id="lv-disp">19</span></div>
        <div>HP <span id="hp-disp">20 / 20</span></div>
        <div id="kr-disp" style="color:var(--purple); display:none;">KR</div>
    </header>

    <div id="arena">
        <div id="dust-sans">
            <div id="enemy-sprite">💀</div>
            <div id="enemy-hp-bar" style="width:200px; height:10px; border:2px solid #fff; margin-top:5px;">
                <div id="enemy-hp-fill" style="width:100%; height:100%; background:var(--dust);"></div>
            </div>
        </div>
        <div id="battle-box">
            <div id="soul">
                <svg viewBox="0 0 32 32"><path id="soul-path" d="M16,28.3L3.7,16c-2.3-2.3-2.3-6.1,0-8.5c2.3-2.3,6.1-2.3,8.5,0l3.8,3.8l3.8-3.8c2.3-2.3,6.1-2.3,8.5,0c2.3,2.3,2.3,6.1,0,8.5L16,28.3z" fill="red"/></svg>
            </div>
        </div>
    </div>

    <div id="sidebar">
        <div id="chat"></div>
        <div id="council-data"></div>
    </div>

    <div id="dashboard">
        <div id="dialogue-box" style="border: 4px solid #fff; height: 100px; padding: 15px; font-size: 24px; background: #000;">
            * Dust Sans stands in your way.
        </div>
        <div id="menu-buttons">
            <div class="menu-btn active" id="btn-fight">FIGHT</div>
            <div class="menu-btn" id="btn-act">ACT</div>
            <div class="menu-btn" id="btn-item">ITEM</div>
            <div class="menu-btn" id="btn-mercy">MERCY</div>
        </div>
    </div>
</div>

<script>
/** ⚙️ THE RESOLUTE ENGINE (1:1 MECHANICS) **/
const STATE = {
    hp: 20, maxHp: 20, kr: 0,
    lv: 19, gold: 600,
    owner: 'PLAYER', // PLAYER, FRISK, CHARA, SANS, PAPYRUS
    menuIdx: 0, phase: 'MENU',
    influence: { player: 25, frisk: 25, sans: 25, papy: 25, chara: 0 },
    soul: { x: 142, y: 72, color: 'red', gravity: false, vY: 0 },
    keys: {},
    enemy: { hp: 5000, max: 5000, name: 'Dust Sans' }
};

const SOULS_ATTR = {
    PLAYER: { col: '#ff0000', hp: 20, upside: false },
    FRISK:  { col: '#ff0000', hp: 20, upside: false },
    CHARA:  { col: '#000000', stroke: '#ff0000', hp: 99, upside: false },
    SANS:   { col: '#ffffff', hp: 1, upside: true },
    PAPYRUS:{ col: '#ffffff', hp: 680, upside: true }
};

const box = document.getElementById('battle-box');
const soul = document.getElementById('soul');
const chat = document.getElementById('chat');

/** 🛠️ UTILS **/
function log(who, msg) {
    const color = who === 'SANS' ? 'var(--blue)' : who === 'CHARA' ? 'var(--red)' : who === 'PAPYRUS' ? '#fff' : 'var(--cyan)';
    chat.innerHTML += `<div style="margin-bottom:5px;"><b style="color:${color}">${who}:</b> ${msg}</div>`;
    chat.scrollTop = chat.scrollHeight;
}

function updateBars() {
    const area = document.getElementById('council-data');
    area.innerHTML = '';
    Object.keys(STATE.influence).forEach(key => {
        area.innerHTML += `
            <div class="influence-row">
                ${key.toUpperCase()}: ${Math.floor(STATE.influence[key])}%
                <div class="bar-bg"><div class="bar-fill" style="width:${STATE.influence[key]}%; background:${key==='chara'?'red':key==='sans'?'blue':'cyan'}"></div></div>
            </div>`;
    });
}

/** 💓 SOUL ENGINE **/
function moveSoul() {
    if (STATE.phase !== 'ATTACK') return;
    
    // AI Control Logic
    if (STATE.owner !== 'PLAYER') {
        autoDodge(); // Logic for Frisk/Sans/Papy to move themselves
    } else {
        const speed = 3.5;
        if (!STATE.soul.gravity) {
            if (STATE.keys['ArrowUp']) STATE.soul.y -= speed;
            if (STATE.keys['ArrowDown']) STATE.soul.y += speed;
        } else {
            // 1:1 Gravity Mechanics
            if (STATE.keys['z'] || STATE.keys['Z']) {
                if (STATE.soul.y >= box.clientHeight - 21) STATE.soul.vY = -4.5;
            }
            STATE.soul.vY += 0.2; // Gravity constant
            STATE.soul.y += STATE.soul.vY;
        }
        
        if (STATE.keys['ArrowLeft']) STATE.soul.x -= speed;
        if (STATE.keys['ArrowRight']) STATE.soul.x += speed;
    }

    // Bounds
    STATE.soul.x = Math.max(0, Math.min(box.clientWidth - 21, STATE.soul.x));
    STATE.soul.y = Math.max(0, Math.min(box.clientHeight - 21, STATE.soul.y));

    soul.style.left = STATE.soul.x + 'px';
    soul.style.top = STATE.soul.y + 'px';
    
    requestAnimationFrame(moveSoul);
}

function autoDodge() {
    // Frisk/Sans/Papy/Chara move automatically based on personality
    // We simulate a basic dodge towards center or away from objects
    const targets = document.querySelectorAll('.bone, .blaster-beam');
    if (targets.length > 0) {
        let t = targets[0].getBoundingClientRect();
        let s = soul.getBoundingClientRect();
        if (t.left < s.left + 50 && t.right > s.left - 50) {
            STATE.soul.y -= (STATE.owner === 'SANS' ? 5 : 2); // Sans teleports/dodges faster
        }
    }
}

/** ⚔️ ATTACK SEQUENCES (1:1 DUSTTALE) **/
function startAttack() {
    STATE.phase = 'ATTACK';
    document.getElementById('dialogue-box').style.display = 'none';
    box.style.display = 'block';
    moveSoul();

    const pattern = Math.random();
    if (pattern < 0.3) patternGasterRain();
    else if (pattern < 0.6) patternBoneJumps();
    else patternXYCross();

    setTimeout(endAttack, 6000);
}

function patternGasterRain() {
    for (let i = 0; i < 6; i++) {
        setTimeout(() => {
            spawnBlaster(Math.random() * 260, 0, 'down');
        }, i * 800);
    }
}

function patternBoneJumps() {
    STATE.soul.gravity = true;
    soul.style.fill = 'blue';
    for (let i = 0; i < 15; i++) {
        setTimeout(() => {
            spawnBone(300, 130, 20, 30, -4);
        }, i * 400);
    }
}

function patternXYCross() {
    spawnBlaster(150, 0, 'down');
    setTimeout(() => spawnBlaster(0, 80, 'right'), 1000);
}

function spawnBlaster(x, y, dir) {
    const flash = document.createElement('div');
    flash.className = 'blaster-beam';
    flash.style.left = (dir === 'down' ? x : 0) + 'px';
    flash.style.top = (dir === 'down' ? 0 : y) + 'px';
    flash.style.width = (dir === 'down' ? '30px' : '300px') + 'px';
    flash.style.height = (dir === 'down' ? '160px' : '30px') + 'px';
    flash.style.opacity = '0.2'; // Telegraph
    box.appendChild(flash);

    setTimeout(() => {
        flash.style.opacity = '1';
        flash.style.background = '#00ffff';
        const blastInterval = setInterval(() => checkHit(flash, 5), 16);
        setTimeout(() => {
            clearInterval(blastInterval);
            flash.remove();
        }, 400);
    }, 600);
}

function spawnBone(x, y, w, h, vx) {
    const b = document.createElement('div');
    b.className = 'bone';
    b.style.width = w+'px'; b.style.height = h+'px';
    b.style.left = x+'px'; b.style.top = y+'px';
    box.appendChild(b);
    
    let curX = x;
    const move = setInterval(() => {
        curX += vx;
        b.style.left = curX + 'px';
        checkHit(b, 2);
        if (curX < -50) { clearInterval(move); b.remove(); }
    }, 16);
}

function checkHit(el, dmg) {
    const sR = soul.getBoundingClientRect();
    const eR = el.getBoundingClientRect();
    if (!(sR.right < eR.left || sR.left > eR.right || sR.bottom < eR.top || sR.top > eR.bottom)) {
        if (STATE.owner === 'SANS' && Math.random() > 0.1) return; // Sans Dodge
        STATE.hp -= (dmg / 10); // Karma-style drain
        updateHUD();
        if (STATE.hp <= 0) alert("STAY DETERMINED...");
    }
}

function endAttack() {
    STATE.phase = 'MENU';
    STATE.soul.gravity = false;
    STATE.soul.vY = 0;
    document.getElementById('soul-path').setAttribute('fill', SOULS_ATTR[STATE.owner].col);
    box.style.display = 'none';
    document.getElementById('dialogue-box').style.display = 'block';
}

/** 🕹️ MENU SYSTEM **/
function updateMenu() {
    const btns = ['fight', 'act', 'item', 'mercy'];
    btns.forEach((b, i) => {
        document.getElementById('btn-'+b).classList.toggle('active', i === STATE.menuIdx);
    });
}

function confirmMenu() {
    if (STATE.phase !== 'MENU') return;
    
    if (STATE.menuIdx === 0) { // FIGHT
        STATE.enemy.hp -= (STATE.owner === 'CHARA' ? 999 : 100);
        STATE.influence.chara = Math.min(100, STATE.influence.chara + 10);
        STATE.influence.player = Math.max(0, STATE.influence.player - 5);
        log("CHARA", "Strike. Again.");
    } else if (STATE.menuIdx === 1) { // ACT
        STATE.influence.papy = Math.min(100, STATE.influence.papy + 5);
        log("PAPYRUS", "I BELIEVE IN YOU, HUMAN!");
    }
    
    updateBars();
    startAttack();
}

/** 🧠 SOUL SOVEREIGNTY SWITCHING **/
function switchOwner(target) {
    if (STATE.owner === 'CHARA') return; // Chara never lets go
    STATE.owner = target;
    const attr = SOULS_ATTR[target];
    STATE.maxHp = attr.hp;
    STATE.hp = attr.hp;
    document.getElementById('soul-path').setAttribute('fill', attr.col);
    document.getElementById('soul-path').setAttribute('stroke', attr.stroke || 'none');
    document.getElementById('current-controller').innerText = `OWNER: ${target}`;
    updateHUD();
}

/** 🖱️ EVENT LISTENERS **/
window.onkeydown = (e) => {
    STATE.keys[e.key] = true;
    if (STATE.phase === 'MENU') {
        if (e.key === 'ArrowLeft') { STATE.menuIdx = (STATE.menuIdx + 3) % 4; updateMenu(); }
        if (e.key === 'ArrowRight') { STATE.menuIdx = (STATE.menuIdx + 1) % 4; updateMenu(); }
        if (e.key === 'z' || e.key === 'Z') confirmMenu();
    }
    if (e.key === 's' || e.key === 'S') {
        // Toggle between Player and Council
        if (STATE.owner === 'PLAYER') switchOwner('FRISK');
        else if (STATE.owner === 'FRISK') switchOwner('PLAYER');
    }
};
window.onkeyup = (e) => STATE.keys[e.key] = false;

function updateHUD() {
    document.getElementById('hp-disp').innerText = `${Math.ceil(STATE.hp)} / ${STATE.maxHp}`;
    document.getElementById('enemy-hp-fill').style.width = (STATE.enemy.hp / STATE.enemy.max * 100) + '%';
}

// Minute Auto-save
setInterval(() => {
    localStorage.setItem('dust_nexus_memory', JSON.stringify(STATE.influence));
    log("SYS", "Timeline metadata synchronized.");
}, 60000);

// Init
log("SANS", "it's a beautiful day outside.");
log("PAPYRUS", "NYEH HEH HEH!");
updateBars();
updateHUD();

</script>
</body>
</html>
