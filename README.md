<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>A Test of Skill - Sans Promised (Snowdin)</title>
  <style>
    body { margin: 0; background: black; color: white; font-family: "Courier New", monospace; display: flex; justify-content: center; align-items: center; height: 100vh; overflow: hidden; font-weight: bold;}
    #game { width: 800px; height: 600px; border: 4px solid white; box-sizing: border-box; position: relative; background: black; overflow: hidden; }
    
    /* Sans Sprite via inline SVG to avoid broken links */
    #enemy-area { position: absolute; top: 20px; left: 0; width: 100%; height: 180px; display: flex; flex-direction: column; align-items: center; justify-content: flex-start; pointer-events: none; }
    #enemy-sprite { width: 100px; height: 100px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 30"><rect x="11" y="2" width="8" height="2" fill="white"/><rect x="9" y="4" width="2" height="4" fill="white"/><rect x="19" y="4" width="2" height="4" fill="white"/><rect x="7" y="8" width="2" height="6" fill="white"/><rect x="21" y="8" width="2" height="6" fill="white"/><rect x="9" y="14" width="2" height="2" fill="white"/><rect x="19" y="14" width="2" height="2" fill="white"/><rect x="11" y="16" width="8" height="2" fill="white"/><rect x="11" y="8" width="3" height="3" fill="black"/><rect x="16" y="8" width="3" height="3" fill="black"/><rect x="13" y="12" width="4" height="2" fill="black"/><rect x="10" y="18" width="10" height="4" fill="blue"/><rect x="8" y="20" width="2" height="6" fill="white"/><rect x="20" y="20" width="2" height="6" fill="white"/></svg>'); background-size: contain; background-repeat: no-repeat; margin-bottom: 20px; animation: idle 2s infinite alternate ease-in-out; }
    
    @keyframes idle { 0% { transform: translateY(0px); } 100% { transform: translateY(5px); } }
    @keyframes shake { 0%, 100% { transform: translateX(0); } 25% { transform: translateX(-5px); } 75% { transform: translateX(5px); } }
    .shake-anim { animation: shake 0.1s infinite; }

    #dialogue-box { position: absolute; bottom: 150px; left: 20px; right: 20px; min-height: 40px; max-height: 200px; border: 4px solid white; padding: 15px; box-sizing: border-box; font-size: 22px; white-space: pre-line; background: rgba(0,0,0,0.9); overflow: hidden; display: none; z-index: 5;}
    #sans-dialogue-box { position: absolute; top: 30px; left: 450px; width: 260px; min-height: 40px; border: 4px solid black; border-radius: 10px; padding: 12px; background: white; color: black; font-size: 18px; white-space: pre-line; display: none; z-index: 10; font-weight: normal; }
    
    #soul-box { position: absolute; bottom: 150px; left: 260px; width: 280px; height: 140px; border: 4px solid white; box-sizing: border-box; overflow: hidden; display: none; background: black; }
    #soul { width: 16px; height: 16px; background: red; position: absolute; z-index: 20;}
    
    /* Better Bones */
    .bone { position: absolute; background: white; border-radius: 8px; box-shadow: 0 0 2px rgba(255,255,255,0.8); }
    .bone::before, .bone::after { content: ''; position: absolute; background: white; border-radius: 50%; width: 140%; height: 8px; left: -20%; }
    .bone::before { top: -4px; }
    .bone::after { bottom: -4px; }
    
    /* Gaster Blasters via SVG */
    .blaster { position: absolute; width: 60px; height: 60px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 40 40"><path d="M10,5 L30,5 L35,15 L35,25 L25,35 L15,35 L5,25 L5,15 Z" fill="white"/><circle cx="15" cy="15" r="3" fill="black"/><circle cx="25" cy="15" r="3" fill="black"/><rect x="15" y="25" width="10" height="2" fill="black"/></svg>'); background-size: contain; background-repeat: no-repeat; z-index: 15;}
    .blast-beam { position: absolute; background: white; border: 3px solid cyan; box-shadow: 0 0 15px rgba(0,255,255,1); z-index: 14;}
    
    #ui-bar { position: absolute; bottom: 0; left: 0; width: 100%; height: 130px; display: flex; flex-direction: column; justify-content: space-between; padding: 10px 30px; box-sizing: border-box; background: black; }
    #stats-row { display: flex; align-items: center; gap: 20px; font-size: 24px; margin-bottom: 15px; }
    #hp-bar { display: flex; align-items: center; gap: 10px; font-size: 20px; }
    #hp-bar-inner { width: 80px; height: 20px; position: relative; background: red; } /* Shorter for LV 4 */
    #hp-fill { position: absolute; top: 0; left: 0; height: 100%; background: #ffb000; transition: width 0.1s; }
    #kr-fill { position: absolute; top: 0; right: 0; height: 100%; background: #f0f; width: 0%; transition: width 0.1s; }
    #kr-label { color: #f0f; display: none; margin-left: 10px; font-size: 16px;}
    
    #ui-menu { display: flex; justify-content: space-between; font-size: 24px; color: #ff8000; width: 90%; margin: 0 auto; }
    .menu-item { cursor: pointer; padding: 5px 15px; border: 2px solid transparent;}
    .menu-item.selected { color: yellow; border: 2px solid yellow; }
    
    #sub-menu { position: absolute; bottom: 130px; left: 0; width: 100%; height: 100px; display: none; justify-content: center; align-items: center; gap: 20px; font-size: 20px; background: black;}
    .sub-option { border: 2px solid white; padding: 6px 12px; cursor: pointer; }
    .sub-option.selected { color: yellow; border-color: yellow; }
    
    #fight-bar-container { position: absolute; bottom: 230px; left: 50%; transform: translateX(-50%); width: 400px; height: 60px; display: none; flex-direction: column; align-items: center; justify-content: center; color: white; }
    #fight-bar { width: 360px; height: 20px; border: 2px solid white; position: relative; margin-bottom: 8px; background: black; }
    #fight-zone { position: absolute; top: 0; height: 100%; width: 50px; background: yellow; left: 155px; opacity: 0.6; }
    #fight-pointer { position: absolute; top: 0; width: 4px; height: 100%; background: white; box-shadow: 0 0 6px rgba(255,255,255,0.8); }
    #damage-text { font-size: 20px; height: 20px; color: red;}
    
    #end-screen { position: absolute; inset: 0; background: black; color: white; display: none; justify-content: center; align-items: center; flex-direction: column; font-size: 24px; text-align: center; white-space: pre-line; z-index: 100;}
  </style>
</head>
<body>
<div id="game">
  <div id="enemy-area">
    <div id="enemy-sprite"></div>
  </div>
  <div id="dialogue-box"></div>
  <div id="sans-dialogue-box"></div>
  <div id="soul-box"><div id="soul"></div></div>
  
  <div id="fight-bar-container">
    <div id="fight-bar"><div id="fight-zone"></div><div id="fight-pointer"></div></div>
    <div id="damage-text"></div>
  </div>
  
  <div id="sub-menu"></div>
  
  <div id="ui-bar">
    <div id="stats-row">
      <span>CHARA</span>
      <span>LV 4</span>
      <div id="hp-bar">
        <span>HP</span>
        <div id="hp-bar-inner"><div id="hp-fill"></div><div id="kr-fill"></div></div>
        <span id="hp-text">32 / 32</span>
        <span id="kr-label">KR</span>
      </div>
    </div>
    <div id="ui-menu">
      <span class="menu-item selected" data-action="FIGHT">FIGHT</span>
      <span class="menu-item" data-action="ACT">ACT</span>
      <span class="menu-item" data-action="ITEM">ITEM</span>
      <span class="menu-item" data-action="MERCY">MERCY</span>
    </div>
  </div>
  <div id="end-screen"><div id="end-text"></div><div style="font-size: 18px; margin-top: 10px;">(refresh the page to restart)</div></div>
</div>

<script>
  // ========= STATS (LV 4 - Toriel Dead) =========
  let playerHP = 32;
  const playerMaxHP = 32;
  let kr = 0;
  let invincFrames = 0;
  let enemyHP = 1;

  const menuItems = ["FIGHT", "ACT", "ITEM", "MERCY"];
  let menuIndex = 0;
  const menuElements = Array.from(document.querySelectorAll(".menu-item"));

  let inSoulMode = false, canMoveSoul = false, phase = "INTRO_SEQUENCE";
  let turnCount = 0, pureAttackTurns = 0;
  let sansCanBeHit = false, canSpare = false, phase3aActive = false;

  const gameEl = document.getElementById("game");
  const dialogueBox = document.getElementById("dialogue-box");
  const sansDialogueBox = document.getElementById("sans-dialogue-box");
  const soulBox = document.getElementById("soul-box");
  const soul = document.getElementById("soul");
  const hpFill = document.getElementById("hp-fill");
  const krFill = document.getElementById("kr-fill");
  const hpText = document.getElementById("hp-text");
  const krLabel = document.getElementById("kr-label");
  const subMenu = document.getElementById("sub-menu");
  const fightBarContainer = document.getElementById("fight-bar-container");
  const fightBar = document.getElementById("fight-bar");
  const fightPointer = document.getElementById("fight-pointer");
  const fightZone = document.getElementById("fight-zone");
  const damageText = document.getElementById("damage-text");
  const endScreen = document.getElementById("end-screen");

  // Physics
  let soulX = 0, soulY = 0, soulColor = "red";
  let gravityEnabled = false, gravity = 0.25, yVelocity = 0, jumpStrength = -5.5, isJumping = false;
  let soulSpeed = 3.5;
  const keys = {};

  let currentSubType = null, currentSubOptions = [], subIndex = 0;
  let fightPointerPos = 0, fightPointerDir = 1, fightBarActive = false;
  let items = [ { id: "PIE", name: "Bscotch Pie", heal: 32 }, { id: "SNOW", name: "Snowman Piece", heal: 20 }, { id: "CIDER", name: "Spider Cider", heal: 15 } ];

  // Typewriter
  let playerTyping = null, sansTyping = null;
  let playerTextFull = "", sansTextFull = "", playerTextDone = true, sansTextDone = true;
  let waitingForDialogueAdvance = false, dialogueAdvanceCallback = null, currentPlayerStatusText = "";

  function typeText(element, fullText, target, onComplete) {
    if(target === "player") { clearTimeout(playerTyping); playerTextDone = false; playerTextFull = fullText; }
    else { clearTimeout(sansTyping); sansTextDone = false; sansTextFull = fullText; }
    element.textContent = "";
    let i = 0;
    function step() {
      if (i > fullText.length) return;
      element.textContent = fullText.slice(0, i);
      if (i === fullText.length) {
        if(target === "player") playerTextDone = true; else sansTextDone = true;
        setTimeout(() => { if(onComplete) onComplete(); }, 200);
        return;
      }
      i++;
      let delay = fullText.charAt(i-1) === ',' ? 150 : 30;
      let id = setTimeout(step, delay);
      if(target === "player") playerTyping = id; else sansTyping = id;
    }
    step();
  }

  function finishTextInstant(target) {
    if(target === "player" && !playerTextDone) { clearTimeout(playerTyping); dialogueBox.textContent = playerTextFull; playerTextDone = true; }
    if(target === "sans" && !sansTextDone) { clearTimeout(sansTyping); sansDialogueBox.textContent = sansTextFull; sansTextDone = true; }
  }

  function showPlayerDialogue(text, callback, allowAdvance, isStatus) {
    if (isStatus) currentPlayerStatusText = text;
    waitingForDialogueAdvance = allowAdvance; dialogueAdvanceCallback = callback;
    dialogueBox.style.display = "block";
    typeText(dialogueBox, text, "player", () => { if (!allowAdvance && callback) callback(); });
  }

  function showSansDialogue(text, callback, allowAdvance) {
    waitingForDialogueAdvance = allowAdvance; dialogueAdvanceCallback = callback;
    sansDialogueBox.style.display = "block";
    typeText(sansDialogueBox, text, "sans", () => { if (!allowAdvance && callback) { sansDialogueBox.style.display = "none"; callback(); } });
  }

  function hideDialogue() {
    clearTimeout(playerTyping); clearTimeout(sansTyping);
    dialogueBox.style.display = "none"; sansDialogueBox.style.display = "none";
    playerTextDone = true; sansTextDone = true;
  }

  // LORE: Toriel Dead, Snowdin Confrontation
  const introLines = [
    "heya.",
    "you've been busy, huh?",
    "there was an old lady behind a door...",
    "she's not gonna answer anymore, is she?",
    "i made a promise to protect whoever came out of there.",
    "but you... you're not a regular kid.",
    "let's see what happens when a promise gets broken."
  ];
  let introIndex = 0;

  function playNextIntroLine() {
    if (introIndex >= introLines.length) {
      phase = "PLAYER_TURN"; dialogueBox.style.display = "block";
      showPlayerDialogue("* sans blocks the way.\n* the air is freezing.", null, false, true);
      return;
    }
    showSansDialogue(introLines[introIndex++], playNextIntroLine, true);
  }

  function updateHUD() {
    if (playerHP < 0) playerHP = 0;
    const hpRatio = playerHP / playerMaxHP;
    const krRatio = kr / playerMaxHP;
    hpFill.style.width = (hpRatio * 100) + "%";
    krFill.style.width = (krRatio * 100) + "%";
    krFill.style.right = (100 - ((hpRatio + krRatio) * 100)) + "%";
    hpText.textContent = playerHP + " / " + playerMaxHP;
    if (playerHP <= 0) {
      endScreen.style.display = "flex"; endText.textContent = "YOU DIED.\n\n* guess promises are meant to be kept."; phase = "END";
    }
  }

  function setMenuIndex(index) {
    menuIndex = (index + menuItems.length) % menuItems.length;
    menuElements.forEach((el, i) => {
        el.classList.toggle("selected", i === menuIndex);
        el.style.border = i === menuIndex ? "2px solid yellow" : "2px solid transparent";
    });
  }

  function enterSoulMode() {
    inSoulMode = true; soulBox.style.display = "block";
    soulX = 132; soulY = 62; soul.style.left = soulX + "px"; soul.style.top = soulY + "px";
  }

  function exitSoulMode() {
    inSoulMode = false; canMoveSoul = false; soulBox.style.display = "none";
    document.querySelectorAll(".bone, .blaster, .blast-beam").forEach(b => b.remove());
    setSoulColor("red", false);
  }

  function rectsOverlap(a, b) {
      // Shrink soul hitbox slightly for fairness
      let sA = {top: a.top+2, bottom: a.bottom-2, left: a.left+2, right: a.right-2};
      return !(sA.right < b.left || sA.left > b.right || sA.bottom < b.top || sA.top > b.bottom);
  }

  function setSoulColor(color, enableGravity) {
    soul.style.background = (color === "blue" ? "blue" : "red");
    soul.style.boxShadow = color === "blue" ? "0 0 6px rgba(0,128,255,0.9)" : "0 0 6px rgba(255,0,0,0.9)";
    gravityEnabled = enableGravity; if (!enableGravity) yVelocity = 0;
  }

  function soulMovementLoop() {
    if (inSoulMode && canMoveSoul) {
      if (!gravityEnabled) {
        if (keys["ArrowUp"]) soulY -= soulSpeed;
        if (keys["ArrowDown"]) soulY += soulSpeed;
      } else {
        yVelocity += gravity; soulY += yVelocity;
        if (soulY >= 120) { soulY = 120; yVelocity = 0; isJumping = false; }
        if (soulY <= 0) { soulY = 0; yVelocity = 0; }
        if (keys["ArrowUp"] && !isJumping) { yVelocity = jumpStrength; isJumping = true; }
        if (!keys["ArrowUp"] && yVelocity < -1) { yVelocity = -1; }
      }
      if (keys["ArrowLeft"]) soulX -= soulSpeed;
      if (keys["ArrowRight"]) soulX += soulSpeed;
      if (soulX < 0) soulX = 0; if (soulX > 264) soulX = 264;
      soul.style.left = soulX + "px"; soul.style.top = soulY + "px";
    }
    
    // KR logic (Slightly slower for LV4 balance)
    if (kr > 0 && Math.random() < 0.03) {
      kr -= 1; playerHP -= 1;
      if (playerHP <= 1) { playerHP = 1; kr = 0; }
      updateHUD();
    }
    if (invincFrames > 0) invincFrames--;
    requestAnimationFrame(soulMovementLoop);
  }

  function damagePlayer(amount) {
    if (invincFrames > 0) return;
    playerHP -= 1; kr += 1; // 1 damage, 1 kr per hit
    krLabel.style.display = "inline";
    invincFrames = 5; // Slightly more i-frames for early-game fight
    updateHUD();
  }

  // --- ATTACK ENGINES ---
  function spawnBone(x, y, w, h) {
    const bone = document.createElement("div"); bone.classList.add("bone");
    bone.style.left = x + "px"; bone.style.top = y + "px"; bone.style.width = w + "px"; bone.style.height = h + "px";
    soulBox.appendChild(bone); return bone;
  }

  function spawnBlaster(x, y, delay, beamDuration) {
    const blaster = document.createElement("div"); blaster.classList.add("blaster");
    blaster.style.left = x + "px"; blaster.style.top = y + "px"; soulBox.appendChild(blaster);
    const beam = document.createElement("div"); beam.classList.add("blast-beam");
    
    setTimeout(() => {
      gameEl.classList.add("shake-anim"); setTimeout(()=>gameEl.classList.remove("shake-anim"), 200);
      soulBox.appendChild(beam);
      beam.style.width = "20px"; beam.style.height = "500px"; beam.style.left = (x + 20) + "px"; beam.style.top = (y + 50) + "px";
      const start = performance.now();
      function check() {
        if(!inSoulMode || performance.now() - start > beamDuration) { beam.remove(); blaster.remove(); return; }
        if(rectsOverlap(soul.getBoundingClientRect(), beam.getBoundingClientRect())) damagePlayer(1);
        requestAnimationFrame(check);
      }
      requestAnimationFrame(check);
    }, delay);
  }

  // "PROMISED" PHASE 1: Aggressive Blue Magic
  function attackBlueBones(onEnd) {
    setSoulColor("blue", true);
    const startTime = performance.now();
    let bones = [];
    
    // Floor
    bones.push({ el: spawnBone(0, 130, 280, 10), static: true });
    
    // Jumping gaps
    for(let i=0; i<6; i++) {
        bones.push({ el: spawnBone(280 + (i*90), 90, 15, 40), x: 280 + (i*90), y: 90, speed: 4 });
    }

    function loop() {
      if(!inSoulMode || performance.now() - startTime > 6000) { onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if(!b.static) { b.x -= b.speed; b.el.style.left = b.x + "px"; }
        if(rectsOverlap(sRect, b.el.getBoundingClientRect())) damagePlayer();
      });
      requestAnimationFrame(loop);
    }
    loop();
  }

  // PHASE 3a: "Broken Promise"
  function startPhase3a() {
    phase3aActive = true;
    hideDialogue();
    showSansDialogue("you just won't stay down.\nfine.\nthe lady behind the door...\nI HOPE SHE'S WATCHING.", () => {
      phase = "ENEMY_ATTACK"; enterSoulMode(); canMoveSoul = true;
      setSoulColor("red", false);
      
      // Fast Blaster Circle
      let delay = 0;
      for(let i=0; i<8; i++) {
          spawnBlaster(Math.random()*220, -50, delay, 400);
          delay += 400;
      }
      
      setTimeout(() => {
          exitSoulMode();
          showSansDialogue("* pant * * pant *\njust... give up.", () => {
              phase = "PLAYER_TURN"; showPlayerDialogue("* sans is losing his grip.", null, false, true);
          }, true);
      }, 4000);
    }, true);
  }

  function startSansAttack() {
    phase = "ENEMY_ATTACK"; turnCount++; pureAttackTurns++;
    hideDialogue(); enterSoulMode(); canMoveSoul = true;

    if (pureAttackTurns === 5 && !phase3aActive) {
        startPhase3a(); return;
    }

    attackBlueBones(() => {
      exitSoulMode();
      if(playerHP > 0) {
        phase = "PLAYER_TURN";
        showPlayerDialogue(pureAttackTurns < 3 ? "* sans is glaring at you." : "* sans looks angry.", null, false, true);
      }
    });
  }

  // UI ACTIONS
  function startFightBar() { 
      phase = "FIGHT_BAR"; hideDialogue(); fightBarContainer.style.display = "flex"; 
      fightPointerPos = 0; fightPointerDir = 1; fightBarActive = true; damageText.textContent = ""; 
  }

  function stopFightBar() {
    fightBarActive = false; fightBarContainer.style.display = "none";
    const barW = fightBar.clientWidth; const x = fightPointerPos * barW;
    const center = 180; // center of yellow zone
    let hitStrength = 1 - (Math.abs(x - center) / 25);
    
    hideDialogue();
    if(hitStrength > 0) {
        damageText.textContent = "MISS (sans dodged)";
        showSansDialogue("did you really think i'd just stand there?", null, true);
    } else {
        damageText.textContent = "MISS...";
        showSansDialogue("heh. your aim is getting sloppy.", null, true);
    }
    
    setTimeout(() => { damageText.textContent = ""; if(playerHP>0) startSansAttack(); }, 1500);
  }

  function fightBarLoop() {
    if (fightBarActive) {
      fightPointerPos += fightPointerDir * (0.02 + (pureAttackTurns * 0.005));
      if (fightPointerPos <= 0) { fightPointerPos = 0; fightPointerDir = 1; } 
      else if (fightPointerPos >= 0.98) { fightPointerPos = 0.98; fightPointerDir = -1; }
      fightPointer.style.left = (fightPointerPos * fightBar.clientWidth) + "px";
    }
    requestAnimationFrame(fightBarLoop);
  }

  function openSubMenu(options) {
    subIndex = 0; subMenu.innerHTML = ""; subMenu.style.display = "flex";
    options.forEach((opt, idx) => { 
        const span = document.createElement("span"); span.classList.add("sub-option"); 
        if (idx === 0) span.classList.add("selected"); span.textContent = opt.label; 
        subMenu.appendChild(span); 
    });
    phase = "MENU_SUB"; hideDialogue();
  }
  
  function closeSubMenu() { subMenu.style.display = "none"; phase = "PLAYER_TURN"; if(currentPlayerStatusText) showPlayerDialogue(currentPlayerStatusText, null, false, true); }

  function handleInput(key) {
    if (phase === "END") return;
    if (key === "z" || key === "Z" || key === "Enter") {
      if (waitingForDialogueAdvance) {
          waitingForDialogueAdvance = false; sansDialogueBox.style.display="none"; 
          if(dialogueAdvanceCallback) { let cb = dialogueAdvanceCallback; dialogueAdvanceCallback = null; cb(); }
          return;
      }
      if (!playerTextDone || !sansTextDone) { finishTextInstant("player"); finishTextInstant("sans"); return; }
      
      if (phase === "PLAYER_TURN") {
          const action = menuItems[menuIndex];
          if(action === "FIGHT") startFightBar();
          else if(action === "ACT") openSubMenu([{label: "Check"}, {label: "Taunt"}]);
          else if(action === "ITEM") { if(items.length===0) showPlayerDialogue("* No items left.", null, false, false); else openSubMenu(items.map(i=>({label: i.name, id: i.id}))); }
          else if(action === "MERCY") openSubMenu([{label: "Spare"}]);
      } else if (phase === "MENU_SUB") {
          // Process submenu
          subMenu.style.display = "none"; phase = "ENEMY_ATTACK";
          setTimeout(startSansAttack, 1000);
      } else if (phase === "FIGHT_BAR") {
          stopFightBar();
      }
    }
    if ((key === "x" || key === "X") && phase === "MENU_SUB") closeSubMenu();
    
    // Navigation
    if (phase === "PLAYER_TURN" && sansDialogueBox.style.display !== "block") {
      if (key === "ArrowLeft") setMenuIndex(menuIndex - 1);
      if (key === "ArrowRight") setMenuIndex(menuIndex + 1);
    } else if (phase === "MENU_SUB") {
      const subs = document.querySelectorAll(".sub-option");
      if (key === "ArrowLeft") { subIndex = (subIndex - 1 + subs.length) % subs.length; }
      if (key === "ArrowRight") { subIndex = (subIndex + 1) % subs.length; }
      subs.forEach((s, i) => { s.classList.toggle("selected", i === subIndex); s.style.borderColor = i === subIndex ? "yellow" : "white"; });
    }
  }

  document.addEventListener("keydown", e => { keys[e.key] = true; handleInput(e.key); });
  document.addEventListener("keyup", e => keys[e.key] = false);

  updateHUD(); setMenuIndex(0); hideDialogue();
  playNextIntroLine();
  requestAnimationFrame(soulMovementLoop); requestAnimationFrame(fightBarLoop);
</script>
</body>
</html>
