<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: ABSOLUTE PARADOX (FRISK SOVEREIGNTY)</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --blue: #3b82f6; --purple: #d946ef; --dust: #8a2be2; }
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; height: 100vh; display: flex; align-items: center; justify-content: center; }
        
        #nexus { width: 1000px; height: 750px; border: 4px solid #fff; display: grid; grid-template-columns: 650px 1fr; grid-template-rows: 80px 450px 220px; gap: 10px; padding: 10px; background: #000; }

        /* ARENA */
        #arena { grid-column: 1; grid-row: 2; position: relative; border: 2px solid #555; background: #000; overflow: hidden; }
        #dust-sans { position: absolute; top: 20px; left: 50%; transform: translateX(-50%); text-align: center; }
        #enemy-sprite { font-size: 80px; text-shadow: 0 0 20px var(--dust); }
        
        /* BOX (1:1 SCALE) */
        #battle-box { position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%); width: 300px; height: 160px; border: 5px solid #fff; background: #000; }
        #soul { width: 16px; height: 16px; position: absolute; z-index: 100; transition: transform 0.1s; }

        /* DASHBOARD */
        #dashboard { grid-column: 1 / 3; grid-row: 3; border-top: 2px solid #fff; padding: 10px; }
        #menu-btns { display: flex; justify-content: space-around; margin-top: 15px; }
        .menu-btn { width: 130px; border: 3px solid #ffaa00; padding: 10px; color: #ffaa00; font-size: 24px; font-weight: bold; text-align: center; cursor: pointer; }
        .menu-btn.active { background: #ffaa00; color: #000; box-shadow: 0 0 15px #ffaa00; }

        /* SIDEBAR */
        #sidebar { grid-column: 2; grid-row: 2; border-left: 2px solid #fff; padding: 10px; display: flex; flex-direction: column; }
        #chat { flex: 1; background: #050505; border: 1px solid #444; font-size: 14px; overflow-y: auto; padding: 8px; margin-bottom: 10px; }
        .bar-wrap { height: 8px; background: #222; border: 1px solid #444; margin: 4px 0; }
        .fill { height: 100%; transition: width 0.3s; }
        
        /* ATTACKS */
        .bone { position: absolute; background: #fff; }
        .blaster-beam { position: absolute; background: #fff; box-shadow: 0 0 20px #fff; z-index: 50; opacity: 0; }
    </style>
</head>
<body>

<div id="nexus">
    <header style="grid-column: 1/3; display: flex; justify-content: space-around; font-size: 22px; align-items: center;">
        <div id="status-owner" style="color:var(--cyan)">OWNER: PLAYER</div>
        <div>LV <span id="lv">19</span></div>
        <div style="display:flex; align-items:center;">HP <div style="width:100px; height:15px; background:red; margin:0 10px; border:1px solid #fff;"><div id="hp-fill" style="width:100%; height:100%; background:yellow;"></div></div> <span id="hp-text">20/20</span></div>
    </header>

    <div id="arena">
        <div id="dust-sans">
            <div id="enemy-sprite">💀</div>
            <div style="width:200px; height:10px; border:2px solid #fff; margin-top:5px;"><div id="e-hp" style="width:100%; height:100%; background:var(--dust);"></div></div>
        </div>
        <div id="battle-box">
            <div id="soul"><svg viewBox="0 0 32 32"><path id="soul-svg" d="M16,28.3L3.7,16c-2.3-2.3-2.3-6.1,0-8.5c2.3-2.3,6.1-2.3,8.5,0l3.8,3.8l3.8-3.8c2.3-2.3,6.1-2.3,8.5,0c2.3,2.3,2.3,6.1,0,8.5L16,28.3z" fill="red"/></svg></div>
        </div>
    </div>

    <div id="sidebar">
        <div id="chat"></div>
        <div style="font-size:12px;">
            PLAYER: <div class="bar-wrap"><div id="bar-p" class="fill" style="background:var(--cyan); width:25%"></div></div>
            FRISK (AI LV: <span id="frisk-skill">1</span>): <div class="bar-wrap"><div id="bar-f" class="fill" style="background:var(--purple); width:25%"></div></div>
            CHARA: <div class="bar-wrap"><div id="bar-c" class="fill" style="background:var(--red); width:0%"></div></div>
        </div>
    </div>

    <div id="dashboard">
        <div id="dialogue" style="border:4px solid #fff; height:80px; padding:15px; font-size:24px;">* Dust Sans stands in your way.</div>
        <div id="menu-btns">
            <div class="menu-btn active" id="btn0">FIGHT</div>
            <div class="menu-btn" id="btn1">ACT</div>
            <div class="menu-btn" id="btn2">ITEM</div>
            <div class="menu-btn" id="btn3">MERCY</div>
        </div>
    </div>
</div>

<script>
/** ⚙️ PHYSICS & STATE ENGINE **/
const STATE = {
    hp: 20, maxHp: 20, kr: 0, lv: 19,
    owner: 'PLAYER', // PLAYER, FRISK, CHARA
    friskSkill: 1, 
    phase: 'MENU', menuIdx: 0,
    soul: { x: 142, y: 72, vY: 0, gravity: false, invuln: 0 },
    keys: {}, influence: { p: 25, f: 25, c: 0 },
    enemyHP: 5000
};

const chat = document.getElementById('chat');
const soul = document.getElementById('soul');
const box = document.getElementById('battle-box');

function log(who, msg) {
    const col = who === 'SANS' ? 'var(--blue)' : who === 'FRISK' ? 'var(--purple)' : 'var(--cyan)';
    chat.innerHTML += `<div><b style="color:${col}">${who}:</b> ${msg}</div>`;
    chat.scrollTop = chat.scrollHeight;
}

/** 💓 CORE PHYSICS LOOP **/
function update() {
    if (STATE.phase === 'ATTACK') {
        if (STATE.soul.invuln > 0) STATE.soul.invuln--;

        if (STATE.owner === 'PLAYER') {
            const speed = 3.5;
            if (STATE.soul.gravity) {
                // 1:1 Jump Arc
                if ((STATE.keys['z'] || STATE.keys['Z']) && STATE.soul.y >= 144) STATE.soul.vY = -5;
                STATE.soul.vY += 0.25; // Gravity
                STATE.soul.y += STATE.soul.vY;
            } else {
                if (STATE.keys['ArrowUp']) STATE.soul.y -= speed;
                if (STATE.keys['ArrowDown']) STATE.soul.y += speed;
            }
            if (STATE.keys['ArrowLeft']) STATE.soul.x -= speed;
            if (STATE.keys['ArrowRight']) STATE.soul.x += speed;
        } else if (STATE.owner === 'FRISK') {
            friskAIDodge();
        }

        // 1:1 Box Collision
        STATE.soul.x = Math.max(0, Math.min(284, STATE.soul.x));
        STATE.soul.y = Math.max(0, Math.min(144, STATE.soul.y));
        if (STATE.soul.y >= 144) STATE.soul.vY = 0;

        soul.style.left = STATE.soul.x + 'px';
        soul.style.top = STATE.soul.y + 'px';
    }
    
    // Karma Drain
    if (STATE.kr > 0) {
        STATE.kr -= 0.05;
        STATE.hp -= 0.02;
    }
    
    updateHUD();
    requestAnimationFrame(update);
}

/** 🤖 FRISK AI DODGING **/
function friskAIDodge() {
    // Frisk learns to look for bones/beams
    const bones = document.querySelectorAll('.bone, .blaster-beam');
    let safestX = STATE.soul.x;
    let safestY = STATE.soul.y;

    bones.forEach(b => {
        const r = b.getBoundingClientRect();
        const s = soul.getBoundingClientRect();
        // If dangerous, move away. Skill factor increases "predictive" speed.
        if (Math.hypot(r.left - s.left, r.top - s.top) < 60) {
            safestX += (s.left < r.left ? -STATE.friskSkill : STATE.friskSkill);
            if (!STATE.soul.gravity) safestY += (s.top < r.top ? -STATE.friskSkill : STATE.friskSkill);
            else if (r.top > s.top + 20) STATE.soul.vY = -5; // Jump!
        }
    });

    STATE.soul.x += (safestX - STATE.soul.x) * 0.1;
    if (!STATE.soul.gravity) STATE.soul.y += (safestY - STATE.soul.y) * 0.1;
    else {
        STATE.soul.vY += 0.25;
        STATE.soul.y += STATE.soul.vY;
    }
}

/** ⚔️ ATTACK ENGINE (1:1 DUSTTALE) **/
function startEnemyTurn() {
    STATE.phase = 'ATTACK';
    box.style.display = 'block';
    document.getElementById('dialogue').style.display = 'none';

    const rand = Math.random();
    if (rand < 0.4) spawnGasterCircle();
    else if (rand < 0.7) spawnBoneStalks();
    else spawnBlueJumpPattern();

    setTimeout(() => {
        STATE.phase = 'MENU';
        box.style.display = 'none';
        document.getElementById('dialogue').style.display = 'block';
        if (STATE.owner === 'FRISK') friskThinkMenu();
    }, 6000);
}

function spawnGasterCircle() {
    for (let i = 0; i < 8; i++) {
        setTimeout(() => {
            if (STATE.phase !== 'ATTACK') return;
            createBlaster(Math.random() * 250, 0, 'V');
        }, i * 700);
    }
}

function spawnBoneStalks() {
    for (let i = 0; i < 20; i++) {
        setTimeout(() => {
            if (STATE.phase !== 'ATTACK') return;
            createBone(300, 100, 15, 60, -4);
        }, i * 300);
    }
}

function spawnBlueJumpPattern() {
    STATE.soul.gravity = true;
    document.getElementById('soul-svg').setAttribute('fill', 'blue');
    for (let i = 0; i < 10; i++) {
        setTimeout(() => createBone(300, 140, 20, 20, -3), i * 600);
    }
}

function createBlaster(x, y, dir) {
    const beam = document.createElement('div');
    beam.className = 'blaster-beam';
    beam.style.left = x + 'px'; beam.style.top = '0px';
    beam.style.width = '30px'; beam.style.height = '160px';
    box.appendChild(beam);
    
    // Telegraph
    setTimeout(() => beam.style.opacity = '0.3', 100);
    setTimeout(() => {
        beam.style.opacity = '1';
        beam.style.background = '#00ffff';
        const hit = setInterval(() => checkCollision(beam, 4), 16);
        setTimeout(() => { clearInterval(hit); beam.remove(); }, 400);
    }, 800);
}

function createBone(x, y, w, h, vx) {
    const b = document.createElement('div');
    b.className = 'bone';
    b.style.left = x+'px'; b.style.top = y+'px';
    b.style.width = w+'px'; b.style.height = h+'px';
    box.appendChild(b);
    const move = setInterval(() => {
        x += vx; b.style.left = x+'px';
        checkCollision(b, 2);
        if (x < -50) { clearInterval(move); b.remove(); }
    }, 16);
}

function checkCollision(el, dmg) {
    if (STATE.soul.invuln > 0) return;
    const sR = soul.getBoundingClientRect();
    const eR = el.getBoundingClientRect();
    if (!(sR.right < eR.left || sR.left > eR.right || sR.bottom < eR.top || sR.top > eR.bottom)) {
        STATE.hp -= 1;
        STATE.kr += dmg;
        STATE.soul.invuln = 30; // 0.5s i-frames
        if (STATE.hp <= 0) { alert("YOU DIED. FRISK FAILED."); location.reload(); }
    }
}

/** 🕹️ MENU & SOVEREIGNTY **/
function toggleSovereignty() {
    if (STATE.owner === 'FRISK') return; // Frisk won't let go
    STATE.owner = 'FRISK';
    document.getElementById('status-owner').innerText = "OWNER: FRISK";
    document.getElementById('status-owner').style.color = "var(--purple)";
    log("SYS", "Control transferred to Frisk. You are now a spectator.");
    if (STATE.phase === 'MENU') friskThinkMenu();
}

function friskThinkMenu() {
    // Frisk simulates thinking and choosing
    log("FRISK", "... I'll handle this.");
    setTimeout(() => {
        STATE.menuIdx = Math.random() > 0.5 ? 0 : 1; // Fight or Act
        updateMenu();
        setTimeout(() => confirmSelection(), 1000);
    }, 1500);
}

function confirmSelection() {
    if (STATE.menuIdx === 0) {
        STATE.enemyHP -= 100;
        STATE.friskSkill += 0.2; // Learning!
        log("SYS", "Frisk attacked! Skill increased.");
    } else {
        log("FRISK", "* You called for help.");
    }
    startEnemyTurn();
}

function updateMenu() {
    for(let i=0; i<4; i++) document.getElementById('btn'+i).classList.toggle('active', i === STATE.menuIdx);
}

window.onkeydown = (e) => {
    STATE.keys[e.key] = true;
    if (e.key === 's' || e.key === 'S') toggleSovereignty();
    
    if (STATE.owner === 'PLAYER') {
        if (STATE.phase === 'MENU') {
            if (e.key === 'ArrowLeft') { STATE.menuIdx = (STATE.menuIdx + 3) % 4; updateMenu(); }
            if (e.key === 'ArrowRight') { STATE.menuIdx = (STATE.menuIdx + 1) % 4; updateMenu(); }
            if (e.key === 'z' || e.key === 'Z') confirmSelection();
        }
    }
};
window.onkeyup = (e) => STATE.keys[e.key] = false;

function updateHUD() {
    document.getElementById('hp-text').innerText = `${Math.ceil(STATE.hp)} / ${STATE.maxHp}`;
    document.getElementById('hp-fill').style.width = (STATE.hp / STATE.maxHp * 100) + '%';
    document.getElementById('frisk-skill').innerText = Math.floor(STATE.friskSkill);
}

update();
log("SANS", "it's a beautiful day outside.");
</script>
</body>
</html>
