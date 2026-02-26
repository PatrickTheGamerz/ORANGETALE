<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>UTD: COUNCIL NEXUS PROTOCOL</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
    :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --green: #22c55e; --blue: #3b82f6; }
    
    body { background: #050505; color: white; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
    
    #nexus { display: grid; grid-template-columns: 850px 350px; grid-template-rows: 70px 480px 250px; gap: 10px; padding: 15px; border: 4px solid white; background: black; box-shadow: 0 0 50px rgba(0,0,0,1); }

    /* BOARD & SOUL BOX */
    #board { grid-column: 1; grid-row: 2; position: relative; border: 2px solid #333; background: radial-gradient(circle at top, #1a0f2e 0, #000 70%); overflow: hidden; }
    #soul-box { position: absolute; bottom: 50px; left: 50%; transform: translateX(-50%); width: 220px; height: 160px; border: 3px solid white; background: rgba(0,0,0,0.9); display: none; }
    #soul { width: 16px; height: 16px; background: red; position: absolute; box-shadow: 0 0 10px red; z-index: 10; }

    /* SIDEBAR CHAT & STATS */
    #sidebar { grid-column: 2; grid-row: 2 / 4; display: flex; flex-direction: column; gap: 10px; border-left: 2px solid white; padding-left: 15px; }
    #chat-container { flex: 1; display: flex; flex-direction: column; background: #0a0a0a; border: 1px solid #444; overflow: hidden; }
    #chat-log { flex: 1; overflow-y: auto; padding: 10px; font-size: 13px; color: #ccc; scroll-behavior: smooth; }
    .msg { margin-bottom: 8px; border-left: 2px solid transparent; padding-left: 5px; }
    
    /* COUNCIL BARS */
    .council-panel { background: #111; padding: 8px; border: 1px solid #333; }
    .bar-wrap { width: 100%; height: 6px; background: #000; margin-top: 4px; border: 1px solid #444; }
    .fill { height: 100%; width: 25%; transition: width 0.3s; }

    /* DASHBOARD */
    #dashboard { grid-column: 1; grid-row: 3; border-top: 2px solid white; display: flex; padding: 15px; gap: 20px; }
    #dialogue-box { flex: 1; background: black; border: 3px solid white; padding: 15px; font-size: 18px; position: relative; }
    #ui-menu { display: flex; flex-direction: column; width: 150px; gap: 8px; }

    /* BUTTONS */
    .cmd-btn { background: #000; border: 2px solid #fff; color: #fff; padding: 10px; cursor: pointer; font-family: inherit; font-size: 18px; text-align: center; }
    .cmd-btn.sel { color: yellow; border-color: yellow; }
    .cmd-btn:disabled { opacity: 0.2; }

    /* ATTACK ELEMENTS */
    .bone { position: absolute; background: white; }
    .gb-head { position: absolute; width: 40px; height: 40px; border: 2px solid white; background: black; border-radius: 5px; }
    .gb-beam { position: absolute; background: var(--cyan); box-shadow: 0 0 20px var(--cyan); opacity: 0; transition: opacity 0.1s; }

    header { grid-column: 1 / 3; display: flex; justify-content: space-around; align-items: center; border-bottom: 2px solid white; }
  </style>
</head>
<body>

  <div id="nexus">
    <header>
      <div>DT: <span id="dt-val" style="color:var(--red)">999</span></div>
      <div>GOLD: <span id="g-val" style="color:var(--gold)">600</span></div>
      <div>LV: <span id="lv-val">1</span></div>
      <div>MODE: <span id="mode-val" style="color:var(--purple)">PLAYER</span></div>
    </header>

    <div id="board">
      <div id="enemy-sprite" style="text-align:center; margin-top:50px; font-size:40px;">S A N S</div>
      <div id="soul-box"><div id="soul"></div></div>
    </div>

    <div id="sidebar">
      <div id="chat-container">
        <div id="chat-log"></div>
      </div>
      
      <div class="council-panel">
        <div style="color:var(--cyan); font-size:0.7rem">PLAYER/FRISK: <span id="v-p">25</span>%</div>
        <div class="bar-wrap"><div id="f-p" class="fill" style="background:var(--cyan)"></div></div>
        
        <div style="color:var(--red); font-size:0.7rem; margin-top:5px">CHARA (AUTO-LEARN): <span id="v-c">25</span>%</div>
        <div class="bar-wrap"><div id="f-c" class="fill" style="background:var(--red)"></div></div>
        
        <div style="color:var(--blue); font-size:0.7rem; margin-top:5px">SANS (AWARENESS): <span id="v-s">25</span>%</div>
        <div class="bar-wrap"><div id="f-s" class="fill" style="background:var(--blue)"></div></div>
      </div>
    </div>

    <div id="dashboard">
      <div id="ui-menu">
        <div class="cmd-btn sel" id="m0">FIGHT</div>
        <div class="cmd-btn" id="m1">ACT</div>
        <div class="cmd-btn" id="m2">ITEM</div>
        <div class="cmd-btn" id="m3">MERCY</div>
      </div>
      <div id="dialogue-box">* sans is judging your every move.</div>
    </div>
  </div>

<script>
/** ⚙️ NEXUS STATE ENGINE **/
let state = {
    gold: 600, lv: 1, dt: 999,
    p: 30, c: 20, s: 25, papy: 25,
    controlMode: 'PLAYER', // PLAYER or FRISK
    wave: 0, turn: 0,
    isAttack: false,
    menuIdx: 0,
    soul: { x: 102, y: 72, vx: 0, vy: 0 },
    keys: {}
};

const chatLog = document.getElementById('chat-log');
const soulBox = document.getElementById('soul-box');
const soul = document.getElementById('soul');

/** 💾 PERSISTENCE **/
function saveProgress() {
    localStorage.setItem('nexus_council_v3', JSON.stringify({p:state.p, c:state.c, s:state.s, lv:state.lv}));
    addMsg("SYS", "Memory alignment saved to local storage.");
}
setInterval(saveProgress, 60000);

function loadProgress() {
    const saved = localStorage.getItem('nexus_council_v3');
    if(saved) {
        const d = JSON.parse(saved);
        state.p = d.p; state.c = d.c; state.s = d.s; state.lv = d.lv;
        updateBars();
        addMsg("SYS", "The souls remember your previous actions.");
    }
}

/** 🗣️ VIRTUAL CHAT **/
function addMsg(who, txt) {
    const div = document.createElement('div');
    div.className = 'msg';
    let color = "#aaa";
    if(who === "SANS") color = "var(--blue)";
    if(who === "CHARA") color = "var(--red)";
    if(who === "FRISK") color = "var(--purple)";
    div.innerHTML = `<b style="color:${color}">${who}:</b> ${txt}`;
    chatLog.appendChild(div);
    chatLog.scrollTop = chatLog.scrollHeight;
}

/** 🕹️ INPUT & SOUL ENGINE **/
window.onkeydown = (e) => {
    state.keys[e.key] = true;
    if(e.key === 's' || e.key === 'S') toggleMode();
    if(!state.isAttack) handleMenu(e.key);
    if(e.key === 'z' || e.key === 'Z') confirmMenu();
};
window.onkeyup = (e) => state.keys[e.key] = false;

function toggleMode() {
    state.controlMode = state.controlMode === 'PLAYER' ? 'FRISK' : 'PLAYER';
    document.getElementById('mode-val').innerText = state.controlMode;
    addMsg("FRISK", state.controlMode === 'FRISK' ? "I'll handle the movement." : "You take over.");
}

function handleMenu(k) {
    if(k === 'ArrowUp') state.menuIdx = (state.menuIdx + 3) % 4;
    if(k === 'ArrowDown') state.menuIdx = (state.menuIdx + 1) % 4;
    document.querySelectorAll('.cmd-btn').forEach((b,i) => b.classList.toggle('sel', i === state.menuIdx));
}

function confirmMenu() {
    if(state.isAttack) return;
    if(state.menuIdx === 0) { // FIGHT
        state.c += 5; state.p -= 2;
        addMsg("CHARA", "Strike. Don't think about it.");
        startSansAttack();
    } else if(state.menuIdx === 1) { // ACT
        state.p += 5; state.c -= 3;
        addMsg("PAPYRUS", "NYEH HEH HEH! FRIENDSHIP!");
        startSansAttack();
    }
    updateBars();
}

/** 💀 SANS ATTACK ENGINE **/
function startSansAttack() {
    state.isAttack = true;
    state.turn++;
    soulBox.style.display = 'block';
    
    // Attack Variant Selection
    const r = Math.random();
    if(r < 0.3) spawnBoneWave();
    else if(r < 0.6) spawnBlasterStorm();
    else spawnMixedChaos();

    setTimeout(() => {
        state.isAttack = false;
        soulBox.style.display = 'none';
        soulBox.innerHTML = '<div id="soul"></div>'; // Clear all attacks
        document.getElementById('dialogue-box').innerText = "* Sans is looking bored.";
    }, 5000);
}

function spawnBoneWave() {
    for(let i=0; i<15; i++) {
        setTimeout(() => {
            if(!state.isAttack) return;
            createBone(220, 110, 15, 50, -3);
        }, i * 300);
    }
}

function spawnBlasterStorm() {
    for(let i=0; i<4; i++) {
        setTimeout(() => {
            if(!state.isAttack) return;
            createBlaster(Math.random()*180, -40, "V");
        }, i * 1000);
    }
}

function spawnMixedChaos() {
    spawnBoneWave();
    setTimeout(spawnBlasterStorm, 1000);
}

function createBone(x, y, w, h, vx) {
    const b = document.createElement('div');
    b.className = 'bone';
    b.style.width = w+'px'; b.style.height = h+'px';
    b.style.left = x+'px'; b.style.top = y+'px';
    soulBox.appendChild(b);
    let curX = x;
    let int = setInterval(() => {
        curX += vx;
        b.style.left = curX+'px';
        if(curX < -50) { clearInterval(int); b.remove(); }
        checkHit(b);
    }, 16);
}

function createBlaster(x, y, type) {
    const head = document.createElement('div');
    head.className = 'gb-head';
    head.style.left = x+'px'; head.style.top = y+'px';
    soulBox.appendChild(head);
    
    setTimeout(() => {
        const beam = document.createElement('div');
        beam.className = 'gb-beam';
        beam.style.width = '20px'; beam.style.height = '160px';
        beam.style.left = (x+10)+'px'; beam.style.top = '0';
        soulBox.appendChild(beam);
        beam.style.opacity = '1';
        
        let blastInt = setInterval(() => checkHit(beam), 16);
        setTimeout(() => { 
            clearInterval(blastInt); 
            beam.remove(); 
            head.remove(); 
        }, 500);
    }, 800);
}

function checkHit(el) {
    const sRect = document.getElementById('soul').getBoundingClientRect();
    const eRect = el.getBoundingClientRect();
    if(!(sRect.right < eRect.left || sRect.left > eRect.right || sRect.bottom < eRect.top || sRect.top > eRect.bottom)) {
        state.dt -= 1.5;
        document.getElementById('dt-val').innerText = Math.floor(state.dt);
    }
}

/** 💓 SOUL MOVEMENT LOOP **/
function updateSoul() {
    if(!state.isAttack) {
        requestAnimationFrame(updateSoul);
        return;
    }

    let spd = 3.5;
    
    // CHARA AUTO-DODGE/SABOTAGE (90% Influence)
    if(state.c >= 90) {
        // Chara is "Learning"—she actually helps dodge better than the player!
        const bones = document.querySelectorAll('.bone, .gb-beam');
        bones.forEach(b => {
            let br = b.getBoundingClientRect();
            let sr = soul.getBoundingClientRect();
            if(Math.hypot(br.left-sr.left, br.top-sr.top) < 40) {
                state.soul.x += (sr.left < br.left ? -2 : 2);
            }
        });
    }

    // FRISK PASSIVE MODE (Smoother movement)
    if(state.controlMode === 'FRISK') spd = 2.5;

    if(state.keys['ArrowUp']) state.soul.y -= spd;
    if(state.keys['ArrowDown']) state.soul.y += spd;
    if(state.keys['ArrowLeft']) state.soul.x -= spd;
    if(state.keys['ArrowRight']) state.soul.x += spd;

    // Bounds
    state.soul.x = Math.max(0, Math.min(204, state.soul.x));
    state.soul.y = Math.max(0, Math.min(144, state.soul.y));

    soul.style.left = state.soul.x + 'px';
    soul.style.top = state.soul.y + 'px';

    requestAnimationFrame(updateSoul);
}

function updateBars() {
    document.getElementById('f-p').style.width = state.p + "%";
    document.getElementById('v-p').innerText = Math.floor(state.p);
    document.getElementById('f-c').style.width = state.c + "%";
    document.getElementById('v-c').innerText = Math.floor(state.c);
    document.getElementById('f-s').style.width = state.s + "%";
    document.getElementById('v-s').innerText = Math.floor(state.s);
    document.getElementById('lv-val').innerText = state.lv;
}

function updateHUD() {
    document.getElementById('g-val').innerText = state.gold;
    document.getElementById('dt-val').innerText = Math.floor(state.dt);
}

loadProgress();
updateSoul();
init();
</script>
</body>
</html>
