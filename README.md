<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>UTD: Council Paradox</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
    body { margin: 0; background: #050008; color: white; font-family: "Courier Prime", monospace; display: flex; justify-content: center; align-items: center; height: 100vh; overflow: hidden; }
    
    #container { display: flex; gap: 10px; border: 4px solid white; padding: 10px; background: black; }
    
    #game { width: 650px; height: 600px; border: 2px solid #444; position: relative; overflow: hidden; background: radial-gradient(circle at top, #1a0f2e 0, #000 70%); }

    /* COUNCIL SIDEBAR */
    #sidebar { width: 280px; display: flex; flex-direction: column; gap: 10px; border-left: 2px solid white; padding-left: 10px; }
    #chat-log { flex: 1; border: 1px solid #444; padding: 10px; font-size: 14px; overflow-y: auto; background: #000; color: #aaa; }
    .influence-box { border: 1px solid #666; padding: 5px; font-size: 12px; }
    .bar-bg { width: 100%; height: 8px; background: #222; margin-top: 4px; border: 1px solid #555; }
    .bar-fill { height: 100%; width: 25%; transition: width 0.4s; }

    /* ORIGINAL ASSETS */
    #enemy-area { position: absolute; top: 20px; width: 100%; text-align: center; }
    #enemy-sprite { font-size: 40px; margin: 10px 0; color: white; text-shadow: 0 0 10px #fff; }
    #enemy-hp-container { width: 200px; height: 10px; border: 2px solid white; margin: 0 auto; }
    #enemy-hp-bar { height: 100%; background: cyan; width: 100%; }
    
    #soul-box { position: absolute; bottom: 180px; left: 50%; transform: translateX(-50%); width: 200px; height: 150px; border: 4px solid white; background: rgba(0,0,0,0.8); display: none; }
    #soul { width: 16px; height: 16px; background: red; position: absolute; box-shadow: 0 0 8px red; }
    
    #dialogue-box { position: absolute; bottom: 180px; left: 20px; right: 20px; height: 150px; border: 4px solid white; padding: 15px; font-size: 20px; background: black; box-sizing: border-box; display: block; }
    
    #ui-bar { position: absolute; bottom: 20px; width: 100%; display: flex; justify-content: center; gap: 15px; }
    .menu-item { border: 2px solid white; padding: 10px 20px; cursor: pointer; font-size: 20px; }
    .menu-item.selected { color: yellow; border-color: yellow; }

    .bone { position: absolute; background: white; }
    .glitch { animation: gl 0.1s infinite; }
    @keyframes gl { 0%{transform:translate(2px)} 50%{transform:translate(-2px)} }
  </style>
</head>
<body>

<div id="container">
  <div id="game">
    <div id="enemy-area">
      <div>sans</div>
      <div id="enemy-sprite">S A N S</div>
      <div id="enemy-hp-container"><div id="enemy-hp-bar"></div></div>
    </div>

    <div id="dialogue-box"></div>
    <div id="soul-box"><div id="soul"></div></div>

    <div id="ui-bar">
      <div class="menu-item selected" id="m0">FIGHT</div>
      <div class="menu-item" id="m1">ACT</div>
      <div class="menu-item" id="m2">ITEM</div>
      <div class="menu-item" id="m3">MERCY</div>
    </div>
  </div>

  <div id="sidebar">
    <div class="influence-box">
        <div style="color:var(--cyan)">PLAYER/FRISK: <span id="v-p">25</span>%</div>
        <div class="bar-bg"><div id="b-p" class="bar-fill" style="background:cyan"></div></div>
    </div>
    <div class="influence-box">
        <div style="color:red">CHARA: <span id="v-c">25</span>%</div>
        <div class="bar-bg"><div id="b-c" class="bar-fill" style="background:red"></div></div>
    </div>
    <div class="influence-box">
        <div style="color:blue">SANS: <span id="v-s">25</span>%</div>
        <div class="bar-bg"><div id="b-s" class="bar-fill" style="background:blue"></div></div>
    </div>
    <div class="influence-box">
        <div style="color:white">PAPYRUS: <span id="v-papy">25</span>%</div>
        <div class="bar-bg"><div id="b-papy" class="bar-fill" style="background:white"></div></div>
    </div>
    <div id="chat-log"></div>
  </div>
</div>

<script>
  // ========= COUNCIL STATE =========
  let influence = { p: 25, c: 25, s: 25, papy: 25 };
  let playerHP = 20, enemyHP = 1, menuIdx = 0, isSoulMode = false;
  let phase = "TURN";
  
  const chatLog = document.getElementById("chat-log");
  const dialogueBox = document.getElementById("dialogue-box");
  const soul = document.getElementById("soul");
  const soulBox = document.getElementById("soul-box");
  
  // ========= AUTO-SAVE SYSTEM =========
  function saveState() {
    localStorage.setItem('utd_paradox_save', JSON.stringify(influence));
    addChat("SYS", "Influence auto-saved.");
  }
  setInterval(saveState, 60000);

  function loadState() {
    const saved = localStorage.getItem('utd_paradox_save');
    if(saved) {
        influence = JSON.parse(saved);
        updateBars();
        addChat("SYS", "Timeline memory loaded.");
    }
  }

  // ========= CHAT SYSTEM =========
  function addChat(who, msg) {
    let color = "#aaa";
    if(who === "SANS") color = "blue";
    if(who === "CHARA") color = "red";
    if(who === "PAPYRUS") color = "white";
    if(who === "FRISK") color = "cyan";
    
    chatLog.innerHTML += `<div><b style="color:${color}">${who}:</b> ${msg}</div>`;
    chatLog.scrollTop = chatLog.scrollHeight;
  }

  // ========= GAME LOGIC =========
  function updateBars() {
    document.getElementById('b-p').style.width = influence.p + "%";
    document.getElementById('v-p').innerText = Math.floor(influence.p);
    document.getElementById('b-c').style.width = influence.c + "%";
    document.getElementById('v-c').innerText = Math.floor(influence.c);
    document.getElementById('b-s').style.width = influence.s + "%";
    document.getElementById('v-s').innerText = Math.floor(influence.s);
    document.getElementById('b-papy').style.width = influence.papy + "%";
    document.getElementById('v-papy').innerText = Math.floor(influence.papy);
  }

  function handleFight() {
    // Chara Gains Influence on Attack
    influence.c = Math.min(100, influence.c + 5);
    influence.p = Math.max(0, influence.p - 2);
    influence.s = Math.min(100, influence.s + 1); // Sans gets "Ready"
    updateBars();
    
    addChat("CHARA", "Good choice, partner. Keep swinging.");
    if(influence.c > 75) addChat("SANS", "you're really going through with this, huh?");
    
    startEnemyAttack();
  }

  function handleAct() {
    influence.p = Math.min(100, influence.p + 5);
    influence.papy = Math.min(100, influence.papy + 5);
    influence.c = Math.max(0, influence.c - 3);
    updateBars();
    
    addChat("PAPYRUS", "NYEH HEH HEH! A FRIENDLY GESTURE!");
    addChat("FRISK", "* You reached out. It feels warm.");
    startEnemyAttack();
  }

  let soulX = 90, soulY = 70;
  function soulLoop() {
    if(isSoulMode) {
        let speed = 3;
        
        // CHARA HIJACK (90% Influence)
        if(influence.c >= 90) {
            // Chara drifts the soul towards any active "bones"
            const bones = document.querySelectorAll('.bone');
            if(bones.length > 0) {
                let b = bones[0].getBoundingClientRect();
                let s = soul.getBoundingClientRect();
                if(b.left > s.left) soulX += 0.5; else soulX -= 0.5;
                if(b.top > s.top) soulY += 0.5; else soulY -= 0.5;
            }
        }

        if(keys['ArrowUp']) soulY -= speed;
        if(keys['ArrowDown']) soulY += speed;
        if(keys['ArrowLeft']) soulX -= speed;
        if(keys['ArrowRight']) soulX += speed;

        // Bounds
        soulX = Math.max(0, Math.min(184, soulX));
        soulY = Math.max(0, Math.min(134, soulY));
        
        soul.style.left = soulX + "px";
        soul.style.top = soulY + "px";
        
        checkCollision();
    }
    requestAnimationFrame(soulLoop);
  }

  function checkCollision() {
    const sRect = soul.getBoundingClientRect();
    document.querySelectorAll('.bone').forEach(b => {
        const bRect = b.getBoundingClientRect();
        if(!(sRect.right < bRect.left || sRect.left > bRect.right || sRect.bottom < bRect.top || sRect.top > bRect.bottom)) {
            playerHP -= 0.1; // Sans KR damage
            if(playerHP <= 0) alert("GAME OVER");
        }
    });
  }

  function startEnemyAttack() {
    phase = "ATTACK";
    dialogueBox.style.display = "none";
    soulBox.style.display = "block";
    isSoulMode = true;
    
    // Spawn simple bones
    for(let i=0; i<3; i++) {
        const b = document.createElement('div');
        b.className = 'bone';
        b.style.width = "10px"; b.style.height = "40px";
        b.style.left = "200px"; b.style.bottom = "0";
        soulBox.appendChild(b);
        
        let bx = 200;
        let bInt = setInterval(() => {
            bx -= 2;
            b.style.left = bx + "px";
            if(bx < -20) { b.remove(); clearInterval(bInt); }
        }, 10);
    }

    setTimeout(() => {
        isSoulMode = false;
        soulBox.style.display = "none";
        dialogueBox.style.display = "block";
        phase = "TURN";
        dialogueBox.innerText = "* Sans is sweating.";
    }, 4000);
  }

  // ========= INPUTS =========
  const keys = {};
  window.addEventListener('keydown', e => {
    keys[e.key] = true;
    if(phase === "TURN") {
        if(e.key === 'ArrowRight') { menuIdx = (menuIdx+1)%4; updateMenu(); }
        if(e.key === 'ArrowLeft') { menuIdx = (menuIdx+3)%4; updateMenu(); }
        if(e.key === 'z') {
            if(menuIdx === 0) handleFight();
            if(menuIdx === 1) handleAct();
        }
    }
  });
  window.addEventListener('keyup', e => keys[e.key] = false);

  function updateMenu() {
    for(let i=0; i<4; i++) document.getElementById('m'+i).classList.remove('selected');
    document.getElementById('m'+menuIdx).classList.add('selected');
  }

  function typeWriter(txt) {
    dialogueBox.innerText = "";
    let i = 0;
    let int = setInterval(() => {
        dialogueBox.innerText += txt[i];
        i++;
        if(i >= txt.length) clearInterval(int);
    }, 50);
  }

  loadState();
  typeWriter("* You feel your sins crawling on your back.");
  requestAnimationFrame(soulLoop);

</script>
</body>
</html>
