<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: THE DUST PARADOX</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --blue: #3b82f6; --dust: #8a2be2; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; height: 100vh; display: flex; align-items: center; justify-content: center; }
        
        #game-container { display: grid; grid-template-columns: 800px 350px; grid-template-rows: 60px 450px 250px; gap: 10px; border: 4px solid #fff; padding: 10px; background: #000; position: relative; }

        /* HEADER */
        header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid #fff; }
        .stat-box { font-size: 1.2rem; }

        /* BATTLEFIELD */
        #battlefield { grid-column: 1; grid-row: 2; position: relative; border: 2px solid #333; background: radial-gradient(circle at center, #1a002e 0%, #000 80%); overflow: hidden; }
        #dust-sans { width: 100px; height: 120px; position: absolute; top: 40px; left: 350px; text-align: center; font-size: 0.8rem; }
        #dust-hp-bar { width: 200px; height: 10px; border: 2px solid #fff; margin: 5px auto; }
        #dust-hp-fill { height: 100%; background: var(--dust); width: 100%; transition: width 0.3s; }

        /* SOUL BOX */
        #soul-box { position: absolute; bottom: 40px; left: 275px; width: 250px; height: 160px; border: 4px solid #fff; background: rgba(0,0,0,0.9); display: none; z-index: 10; }
        #soul { width: 20px; height: 20px; position: absolute; transition: transform 0.1s; display: flex; align-items: center; justify-content: center; }
        .heart { width: 100%; height: 100%; clip-path: path('M10,4 C10,4 8,0 4,0 C1,0 0,3 0,5 C0,9 10,14 10,14 C10,14 20,9 20,5 C20,3 19,0 16,0 C12,0 10,4 10,4'); transform: scale(1.5); }

        /* DASHBOARD */
        #dashboard { grid-column: 1; grid-row: 3; border-top: 2px solid #fff; display: flex; padding: 10px; gap: 15px; }
        #dialogue-box { flex: 1; border: 3px solid #fff; padding: 15px; font-size: 1.1rem; min-height: 100px; background: #000; }
        #menu-container { width: 180px; display: flex; flex-direction: column; gap: 5px; }

        /* SIDEBAR / CHAT */
        #sidebar { grid-column: 2; grid-row: 2 / 4; border-left: 2px solid #fff; padding-left: 10px; display: flex; flex-direction: column; gap: 10px; }
        #chat-log { flex: 1; background: #0a0a0a; border: 1px solid #444; overflow-y: auto; padding: 8px; font-size: 0.8rem; }
        .msg { margin-bottom: 5px; }

        /* COUNCIL BARS */
        .council-bar { padding: 5px; background: #111; border: 1px solid #333; font-size: 0.7rem; }
        .fill-wrap { height: 6px; background: #000; width: 100%; border: 1px solid #444; margin-top: 2px; }
        .fill { height: 100%; transition: width 0.3s; }

        /* BUTTONS */
        .btn { background: #000; border: 2px solid #fff; color: #fff; padding: 10px; cursor: pointer; font-family: inherit; font-size: 1.1rem; text-align: left; }
        .btn:hover { background: #fff; color: #000; }
        .btn.active { color: #ffff00; border-color: #ffff00; }
        
        .glitch { animation: gl 0.1s infinite; }
        @keyframes gl { 0%{transform:translate(2px)} 50%{transform:translate(-2px)} }
    </style>
</head>
<body>

<div id="game-container">
    <header>
        <div class="stat-box">OWNER: <span id="owner-name" style="color:var(--cyan)">PLAYER</span></div>
        <div class="stat-box">HP: <span id="hp-val">20/20</span></div>
        <div class="stat-box">STAMINA: <span id="stamina-val">100</span></div>
        <button id="ready-btn" class="btn" style="padding: 2px 10px; font-size: 0.8rem;">ENGAGE</button>
    </header>

    <div id="battlefield">
        <div id="dust-sans">
            <div style="color:var(--dust); font-weight:bold;">DUST SANS</div>
            <div id="dust-hp-bar"><div id="dust-hp-fill"></div></div>
            <div style="font-size:3rem; margin-top:10px;">💀</div>
        </div>
        <div id="soul-box"><div id="soul"><div class="heart" id="heart-inner"></div></div></div>
    </div>

    <div id="sidebar">
        <div id="chat-log"></div>
        <div id="council-area"></div>
    </div>

    <div id="dashboard">
        <div id="menu-container">
            <button class="btn active" id="btn-0" onclick="setMenu(0)">FIGHT</button>
            <button class="btn" id="btn-1" onclick="setMenu(1)">ACT</button>
            <button class="btn" id="btn-2" onclick="setMenu(2)">ITEM</button>
            <button class="btn" id="btn-3" onclick="setMenu(3)">MERCY</button>
        </div>
        <div id="dialogue-box">
            * You feel your sins crawling on your back.
        </div>
    </div>
</div>

<script>
/** ⚙️ CORE SYSTEM ENGINE **/
const STATE = {
    gold: 600, lv: 1, dt: 20, maxDt: 20, stamina: 100,
    owner: 'PLAYER', // PLAYER, FRISK, CHARA, SANS, PAPYRUS
    charaRage: 0,
    isAttackPhase: false,
    menuIdx: 0,
    bossHP: 5000,
    council: { 
        player: 25, frisk: 25, sans: 25, papy: 25, chara: 0 
    },
    keys: {},
    soul: { x: 115, y: 70 }
};

const SOULS = {
    PLAYER: { col: 'red', hp: 20, acts: ['CHECK', 'INSULT'], shape: 'normal' },
    FRISK:  { col: 'red', hp: 20, acts: ['TALK', 'HOPE'], shape: 'normal' },
    CHARA:  { col: 'black', hp: 99, acts: ['ERASE', 'THREAT'], shape: 'fallen', stroke: 'red' },
    SANS:   { col: 'white', hp: 1, acts: ['JOKE', 'REST'], shape: 'upside' },
    PAPYRUS:{ col: 'white', hp: 680, acts: ['COOK', 'BELIEVE'], shape: 'upside' }
};

/** 📝 CHAT SYSTEM **/
function log(who, msg) {
    const chat = document.getElementById('chat-log');
    const colors = { SANS: 'var(--blue)', CHARA: 'var(--red)', PAPYRUS: 'white', FRISK: 'var(--purple)', SYS: '#555' };
    chat.innerHTML += `<div class="msg"><b style="color:${colors[who]}">${who}:</b> ${msg}</div>`;
    chat.scrollTop = chat.scrollHeight;
}

/** 📊 COUNCIL UI **/
function updateCouncil() {
    const area = document.getElementById('council-area');
    area.innerHTML = '';
    Object.keys(STATE.council).forEach(key => {
        const val = STATE.council[key];
        area.innerHTML += `
            <div class="council-bar">
                ${key.toUpperCase()}: ${Math.floor(val)}%
                <div class="fill-wrap"><div class="fill" style="width:${val}%; background:${key==='player'?'var(--cyan)':key==='chara'?'red':key==='sans'?'blue':'white'}"></div></div>
            </div>`;
    });
    
    // Auto-switch to Chara if Rage/Influence is 100
    if(STATE.council.chara >= 100 && STATE.owner !== 'CHARA') {
        switchOwner('CHARA');
        log("SYS", "The Demon has taken full control.");
    }
}

/** 🕹️ CONTROL LOGIC **/
function switchOwner(target) {
    if(STATE.owner === 'FRISK' && target === 'PLAYER') { /* Allowed */ }
    else if(STATE.owner === 'PLAYER' && target === 'FRISK') { /* Allowed */ }
    else if(STATE.owner === 'CHARA') return; // Cannot switch back easily

    STATE.owner = target;
    const s = SOULS[target];
    STATE.dt = s.hp;
    STATE.maxDt = s.hp;
    
    document.getElementById('owner-name').innerText = target;
    document.getElementById('owner-name').style.color = target === 'CHARA' ? 'red' : 'var(--cyan)';
    
    const heart = document.getElementById('heart-inner');
    heart.style.background = s.col;
    heart.style.border = s.stroke ? `2px solid ${s.stroke}` : 'none';
    heart.style.transform = s.shape === 'upside' ? 'scale(1.5) rotate(180deg)' : 'scale(1.5)';
    
    updateHUD();
    log("SYS", `Ownership shifted to ${target}.`);
}

function setMenu(idx) {
    if(STATE.isAttackPhase) return;
    STATE.menuIdx = idx;
    for(let i=0; i<4; i++) document.getElementById(`btn-${i}`).classList.toggle('active', i === idx);
}

function handleConfirm() {
    if(STATE.isAttackPhase) return;
    
    if(STATE.menuIdx === 0) { // FIGHT
        const dmg = STATE.owner === 'CHARA' ? 999 : 100;
        STATE.bossHP -= dmg;
        STATE.council.chara += 10;
        STATE.council.player -= 5;
        log(STATE.owner, "Take this!");
        triggerAttackPhase();
    } else if(STATE.menuIdx === 1) { // ACT
        const acts = SOULS[STATE.owner].acts;
        document.getElementById('dialogue-box').innerHTML = acts.map(a => `<button class="btn" style="font-size:0.8rem" onclick="useAct('${a}')">${a}</button>`).join(' ');
    }
    updateCouncil();
}

function useAct(act) {
    if(act === 'HOPE') { STATE.dt = STATE.maxDt; log("FRISK", "I won't give up!"); }
    if(act === 'REST') { STATE.stamina = 100; log("SANS", "zzz... much better."); }
    if(act === 'INSULT') { log("PLAYER", "You're just a pile of dust."); }
    
    updateHUD();
    triggerAttackPhase();
}

/** 💀 BOSS ENGINE **/
function triggerAttackPhase() {
    STATE.isAttackPhase = true;
    document.getElementById('soul-box').style.display = 'block';
    document.getElementById('dialogue-box').innerText = "* Dust Sans is preparing a special hell for you.";
    
    // Spawn Dust Bones
    for(let i=0; i<8; i++) {
        setTimeout(() => spawnBone(), i * 500);
    }
    
    // End Phase
    setTimeout(() => {
        STATE.isAttackPhase = false;
        document.getElementById('soul-box').style.display = 'none';
        document.getElementById('soul-box').innerHTML = '<div id="soul"><div class="heart" id="heart-inner"></div></div>';
        // Reset soul element ref
        switchOwner(STATE.owner); 
    }, 5000);
}

function spawnBone() {
    if(!STATE.isAttackPhase) return;
    const b = document.createElement('div');
    b.className = 'bone';
    b.style.width = '15px'; b.style.height = '60px';
    b.style.left = '250px'; b.style.bottom = Math.random() * 100 + 'px';
    b.style.background = 'var(--dust)';
    document.getElementById('soul-box').appendChild(b);
    
    let bx = 250;
    const bInt = setInterval(() => {
        bx -= 3;
        b.style.left = bx + 'px';
        if(bx < -20) { clearInterval(bInt); b.remove(); }
        checkCollision(b);
    }, 16);
}

function checkCollision(el) {
    const sRect = document.getElementById('soul').getBoundingClientRect();
    const eRect = el.getBoundingClientRect();
    if(!(sRect.right < eRect.left || sRect.left > eRect.right || sRect.bottom < eRect.top || sRect.top > eRect.bottom)) {
        // Sans Dodge Logic
        if(STATE.owner === 'SANS' && STATE.stamina > 20) {
            STATE.stamina -= 5;
            // Ghost effect for dodge
            return;
        }
        STATE.dt -= 0.5;
        if(STATE.dt <= 0) {
            alert("DETERMINATION LOST.");
            location.reload();
        }
        updateHUD();
    }
}

/** 💓 CORE LOOP **/
function update() {
    // Movement
    if(STATE.isAttackPhase) {
        let spd = (STATE.owner === 'SANS') ? 2 : 4;
        if(STATE.keys['ArrowUp']) STATE.soul.y -= spd;
        if(STATE.keys['ArrowDown']) STATE.soul.y += spd;
        if(STATE.keys['ArrowLeft']) STATE.soul.x -= spd;
        if(STATE.keys['ArrowRight']) STATE.soul.x += spd;
        
        // Bounds
        STATE.soul.x = Math.max(0, Math.min(230, STATE.soul.x));
        STATE.soul.y = Math.max(0, Math.min(140, STATE.soul.y));
        
        const soulEl = document.getElementById('soul');
        soulEl.style.left = STATE.soul.x + 'px';
        soulEl.style.top = STATE.soul.y + 'px';
    }

    // Chara Control Decay
    if(STATE.owner === 'CHARA') {
        STATE.council.chara -= 0.05;
        if(STATE.council.chara <= 50) switchOwner('PLAYER');
    }

    document.getElementById('dust-hp-fill').style.width = (STATE.bossHP / 5000 * 100) + '%';
    requestAnimationFrame(update);
}

/** 🖱️ EVENT LISTENERS **/
window.onkeydown = (e) => {
    STATE.keys[e.key] = true;
    if(e.key === 's' || e.key === 'S') {
        if(STATE.owner === 'PLAYER') switchOwner('FRISK');
        else if(STATE.owner === 'FRISK') switchOwner('PLAYER');
    }
    if(e.key === 'z' || e.key === 'Z') handleConfirm();
    if(e.key === 'ArrowUp') setMenu((STATE.menuIdx + 3) % 4);
    if(e.key === 'ArrowDown') setMenu((STATE.menuIdx + 1) % 4);
};
window.onkeyup = (e) => STATE.keys[e.key] = false;

document.getElementById('ready-btn').onclick = () => {
    if(!STATE.isAttackPhase) triggerAttackPhase();
};

function updateHUD() {
    document.getElementById('hp-val').innerText = `${Math.ceil(STATE.dt)}/${STATE.maxDt}`;
    document.getElementById('stamina-val').innerText = Math.floor(STATE.stamina);
    document.getElementById('gold-val').innerText = STATE.gold;
    document.getElementById('lv-val').innerText = STATE.lv;
}

// Minute Auto-save
setInterval(() => {
    localStorage.setItem('utd_dust_paradox_memory', JSON.stringify(STATE.council));
    log("SYS", "Timeline metadata synchronized.");
}, 60000);

// Init
log("SANS", "it's a beautiful day outside.");
log("CHARA", "Greetings.");
updateCouncil();
update();
</script>
</body>
</html>
