<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>A Test of Skill - Sans Fight</title>
  <style>
    body { margin: 0; background: black; color: white; font-family: "Courier New", monospace; display: flex; justify-content: center; align-items: center; height: 100vh; }
    #game { width: 800px; height: 600px; border: 4px solid white; box-sizing: border-box; position: relative; background: radial-gradient(circle at top, #3a2950 0, #050008 70%); overflow: hidden; }
    #enemy-area { position: absolute; top: 20px; left: 0; width: 100%; height: 180px; display: flex; flex-direction: column; align-items: center; justify-content: flex-start; opacity: 1; pointer-events: none; }
    #enemy-name { font-size: 24px; margin-bottom: 8px; text-transform: lowercase; }
    #enemy-sprite { width: 96px; height: 96px; border: 2px solid white; display: flex; align-items: center; justify-content: center; margin-bottom: 8px; font-size: 12px; box-shadow: 0 0 10px rgba(255,255,255,0.4); position: relative; }
    #enemy-hp-container { width: 300px; height: 16px; border: 2px solid white; position: relative; box-shadow: 0 0 4px rgba(0,255,255,0.8); }
    #enemy-hp-bar { position: absolute; top: 0; left: 0; height: 100%; background: cyan; width: 100%; }
    #enemy-hp-text { margin-top: 4px; font-size: 14px; }
    #dialogue-box { position: absolute; bottom: 150px; left: 20px; right: 20px; min-height: 40px; max-height: 200px; border: 4px solid white; padding: 10px; box-sizing: border-box; font-size: 18px; white-space: pre-line; background: rgba(0,0,0,0.8); overflow: hidden; display: none; }
    #sans-dialogue-box { position: absolute; top: 60px; left: 480px; min-height: 40px; max-height: 160px; border: 4px solid white; box-sizing: border-box; padding: 8px; background: white; color: black; font-size: 18px; white-space: pre-line; display: none; overflow: hidden; }
    #soul-box { position: absolute; bottom: 150px; left: 260px; width: 280px; height: 120px; border: 4px solid white; box-sizing: border-box; overflow: hidden; display: none; background: rgba(0,0,0,0.6); }
    #soul { width: 16px; height: 16px; background: red; position: absolute; border-radius: 3px; box-shadow: 0 0 6px rgba(255,0,0,0.9); }
    .bone { position: absolute; background: white; box-shadow: 0 0 4px rgba(255,255,255,0.8); }
    .blaster { position: absolute; width: 40px; height: 40px; border: 2px solid white; border-radius: 50%; box-sizing: border-box; box-shadow: 0 0 8px rgba(0,255,255,0.9); }
    .blast-beam { position: absolute; background: cyan; opacity: 0.8; box-shadow: 0 0 15px rgba(0,255,255,1); }
    #ui-bar { position: absolute; bottom: 0; left: 0; width: 100%; height: 130px; border-top: 4px solid white; box-sizing: border-box; display: flex; flex-direction: column; justify-content: space-between; padding: 8px 16px; background: rgba(0,0,0,0.9); opacity: 0; pointer-events: none; }
    #ui-menu { display: flex; gap: 20px; font-size: 22px; }
    .menu-item { cursor: pointer; }
    .menu-item.selected { color: yellow; }
    #hp-bar { display: flex; align-items: center; gap: 10px; font-size: 16px; }
    #hp-bar-inner { width: 200px; height: 16px; border: 2px solid white; position: relative; background: red; }
    #hp-fill { position: absolute; top: 0; left: 0; height: 100%; background: #ffb000; width: 100%; transition: width 0.1s; }
    #kr-fill { position: absolute; top: 0; right: 0; height: 100%; background: #f0f; width: 0%; transition: width 0.1s; }
    #kr-label { color: #f0f; display: none; margin-left: 10px; font-weight: bold; }
    #sub-menu { position: absolute; bottom: 130px; left: 0; width: 100%; height: 100px; display: none; justify-content: center; align-items: center; gap: 20px; font-size: 18px; }
    .sub-option { border: 2px solid white; padding: 6px 12px; cursor: pointer; }
    .sub-option.selected { color: yellow; border-color: yellow; }
    #fight-bar-container { position: absolute; bottom: 230px; left: 50%; transform: translateX(-50%); width: 400px; height: 60px; display: none; flex-direction: column; align-items: center; justify-content: center; color: white; }
    #fight-bar { width: 360px; height: 20px; border: 2px solid white; position: relative; margin-bottom: 8px; background: black; }
    #fight-zone { position: absolute; top: 0; height: 100%; width: 80px; background: yellow; left: 140px; opacity: 0.4; }
    #fight-pointer { position: absolute; top: 0; width: 4px; height: 100%; background: red; box-shadow: 0 0 6px rgba(255,0,0,0.8); }
    #damage-text { font-size: 16px; height: 20px; }
    #end-screen { position: absolute; inset: 0; background: black; color: white; display: none; justify-content: center; align-items: center; flex-direction: column; font-size: 24px; text-align: center; white-space: pre-line; }
  </style>
</head>
<body>
<div id="game">
  <div id="enemy-area">
    <div id="enemy-name">sans</div>
    <div id="enemy-sprite">S A N S</div>
    <div id="enemy-hp-container"><div id="enemy-hp-bar"></div></div>
    <div id="enemy-hp-text">HP: 1 / 1</div>
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
    <div id="ui-menu">
      <span class="menu-item selected" data-action="FIGHT">FIGHT</span>
      <span class="menu-item" data-action="ACT">ACT</span>
      <span class="menu-item" data-action="ITEM">ITEM</span>
      <span class="menu-item" data-action="MERCY">MERCY</span>
    </div>
    <div id="hp-bar">
      <span>HP</span>
      <div id="hp-bar-inner"><div id="hp-fill"></div><div id="kr-fill"></div></div>
      <span id="hp-text">92 / 92</span>
      <span id="kr-label">KR</span>
    </div>
  </div>
  <div id="end-screen">
    <div id="end-text"></div>
    <div style="font-size: 18px; margin-top: 10px;">(refresh the page to restart)</div>
  </div>
</div>

<script>
  let playerHP = 92;
  const playerMaxHP = 92;
  let kr = 0;
  let invincFrames = 0;
  let enemyHP = 1;
  const enemyMaxHP = 1;

  const menuItems = ["FIGHT", "ACT", "ITEM", "MERCY"];
  let menuIndex = 0;
  const menuElements = Array.from(document.querySelectorAll(".menu-item"));

  let inSoulMode = false;
  let canMoveSoul = false;
  let phase = "INTRO_SEQUENCE";

  let turnCount = 0;
  let actCount = 0;
  let pureAttackTurns = 0;
  let sansCanBeHit = false;
  let canSpare = false;
  let usedFinalAttackPhase1 = false;
  let usedBlueIntro = false;

  let phase2AActive = false;
  let phase2ATurns = 0;
  const phase2AStartTargets = 8;
  const phase2ATurnTarget = 10;
  let phase2BQueued = false;
  let phase2CActive = false;

  const dialogueBox = document.getElementById("dialogue-box");
  const sansDialogueBox = document.getElementById("sans-dialogue-box");
  const soulBox = document.getElementById("soul-box");
  const soul = document.getElementById("soul");
  const hpFill = document.getElementById("hp-fill");
  const hpText = document.getElementById("hp-text");
  const enemyHPBar = document.getElementById("enemy-hp-bar");
  const enemyHPText = document.getElementById("enemy-hp-text");
  const subMenu = document.getElementById("sub-menu");
  const fightBarContainer = document.getElementById("fight-bar-container");
  const fightBar = document.getElementById("fight-bar");
  const fightPointer = document.getElementById("fight-pointer");
  const fightZone = document.getElementById("fight-zone");
  const damageText = document.getElementById("damage-text");
  const endScreen = document.getElementById("end-screen");
  const endText = document.getElementById("end-text");
  const uiBar = document.getElementById("ui-bar");

  let soulX = 0, soulY = 0;
  let soulColor = "red";
  let gravityEnabled = false;
  let gravity = 0.25;
  let yVelocity = 0;
  let jumpStrength = -6;
  let isJumping = false;
  let baseSoulSpeed = 3.5;
  let soulSpeed = baseSoulSpeed;

  const keys = {};
  let currentSubType = null;
  let currentSubOptions = [];
  let subIndex = 0;
  let fightPointerPos = 0, fightPointerDir = 1, fightBarActive = false;

  let items = [
    { id: "PIE", name: "Butterscotch Pie", heal: 92 },
    { id: "STEAK", name: "Face Steak", heal: 60 },
    { id: "NOODLES", name: "Insta-Noodles", heal: 90 }
  ];

  let lastActionTime = 0;
  const ACTION_COOLDOWN = 1000;

  const BASE_CHAR_DELAY = 50, COMMA_DELAY = 150, DOT_DELAY = 1000, MIN_ADVANCE_DELAY = 1000;
  let playerTyping = null, sansTyping = null;
  let playerTextFull = "", sansTextFull = "";
  let playerTextDone = true, sansTextDone = true;
  let playerTextStartTime = 0, sansTextStartTime = 0;
  let waitingForDialogueAdvance = false, dialogueAdvanceCallback = null, dialogueLastTarget = null;
  let currentPlayerStatusText = "", lastTextHadEndPause = false, playerIsStatus = false;
  let sansAutoAdvanceTimer = null;

  function clearSansAutoAdvanceTimer() { if (sansAutoAdvanceTimer) { clearTimeout(sansAutoAdvanceTimer); sansAutoAdvanceTimer = null; } }
  function clearTypewriter(target) {
    if (target === "player" && playerTyping) { clearTimeout(playerTyping); playerTyping = null; }
    if (target === "sans" && sansTyping) { clearTimeout(sansTyping); sansTyping = null; }
  }
  function charDelay(ch) { return ch === "," ? COMMA_DELAY : BASE_CHAR_DELAY; }

  function typeText(element, fullText, target, onComplete) {
    clearTypewriter(target);
    if (target === "sans") clearSansAutoAdvanceTimer();
    element.textContent = "";
    let i = 0;
    if (target === "player") { playerTextFull = fullText; playerTextDone = false; }
    else { sansTextFull = fullText; sansTextDone = false; }

    function step() {
      if ((target === "player" && playerTyping === null && i > 0) || (target === "sans" && sansTyping === null && i > 0)) return;
      if (i > fullText.length) return;
      element.textContent = fullText.slice(0, i);

      if (i === fullText.length) {
        const now = performance.now();
        if (target === "player") { playerTyping = null; playerTextDone = true; playerTextStartTime = now; }
        else { sansTyping = null; sansTextDone = true; sansTextStartTime = now; }
        const last = fullText.trim().slice(-1);
        const punctDelay = (last === "." || last === "?" || last === "!") ? DOT_DELAY : 0;
        lastTextHadEndPause = (punctDelay > 0);
        setTimeout(() => { if (onComplete) onComplete(); }, punctDelay);
        if (target === "sans") {
          sansAutoAdvanceTimer = setTimeout(() => {
            if (waitingForDialogueAdvance && sansTextDone && phase !== "END") doDialogueAdvance();
          }, MIN_ADVANCE_DELAY + punctDelay + 1000);
        }
        return;
      }
      i++;
      const id = setTimeout(step, charDelay(fullText.charAt(i - 2) || ""));
      if (target === "player") playerTyping = id; else sansTyping = id;
    }
    const id = setTimeout(step, charDelay(fullText.charAt(0) || ""));
    if (target === "player") playerTyping = id; else sansTyping = id;
  }

  function finishTextInstant(target) {
    if (target === "player" && !playerTextDone) { clearTypewriter("player"); dialogueBox.textContent = playerTextFull; playerTextDone = true; playerTextStartTime = performance.now(); }
    if (target === "sans" && !sansTextDone) { clearTypewriter("sans"); sansDialogueBox.textContent = sansTextFull; sansTextDone = true; sansTextStartTime = performance.now(); }
  }

  function canAdvance(target) {
    if (target === "player" && playerIsStatus) return false;
    const now = performance.now();
    if (target === "player" && playerTextDone) return (now - playerTextStartTime) >= MIN_ADVANCE_DELAY;
    if (target === "sans" && sansTextDone) return (now - sansTextStartTime) >= MIN_ADVANCE_DELAY;
    return false;
  }

  function beginDialogue(text, target, callback, allowAdvance, isStatus) {
    waitingForDialogueAdvance = allowAdvance; dialogueAdvanceCallback = callback; dialogueLastTarget = target;
    if (target === "player") {
      playerIsStatus = !!isStatus; dialogueBox.style.display = "block";
      typeText(dialogueBox, text, "player", () => { if (!allowAdvance && callback) callback(); });
    } else {
      clearSansAutoAdvanceTimer(); sansDialogueBox.style.display = "block";
      typeText(sansDialogueBox, text, "sans", () => { if (!allowAdvance && callback) { sansDialogueBox.style.display = "none"; callback(); } });
    }
  }

  function doDialogueAdvance() {
    if (!waitingForDialogueAdvance) return;
    waitingForDialogueAdvance = false; clearSansAutoAdvanceTimer();
    const cb = dialogueAdvanceCallback; dialogueAdvanceCallback = null;
    if (cb) cb();
  }

  function handleDialogueZ() {
    if (dialogueLastTarget === "sans" && !sansTextDone) { finishTextInstant("sans"); return; }
    if (dialogueLastTarget === "player" && !playerTextDone) { finishTextInstant("player"); return; }
    if (!canAdvance(dialogueLastTarget)) return;
    doDialogueAdvance();
  }

  function showPlayerDialogue(text, callback, allowAdvance, isStatus) {
    if (isStatus) currentPlayerStatusText = text;
    beginDialogue(text, "player", callback || null, !!allowAdvance, !!isStatus);
  }

  function showSansDialogue(text, callback, allowAdvance) {
    beginDialogue(text, "sans", () => { sansDialogueBox.style.display = "none"; if (callback) callback(); }, !!allowAdvance, false);
  }

  function hidePlayerDialogue() {
    clearTypewriter("player"); dialogueBox.textContent = ""; dialogueBox.style.display = "none";
    playerTextFull = ""; playerTextDone = true; playerIsStatus = false;
  }

  function hideSansDialogue() {
    clearTypewriter("sans"); clearSansAutoAdvanceTimer(); sansDialogueBox.textContent = ""; sansDialogueBox.style.display = "none";
    sansTextFull = ""; sansTextDone = true;
  }

  function restorePlayerStatusWithTypewriter() { if (currentPlayerStatusText) showPlayerDialogue(currentPlayerStatusText, null, false, true); }

  const introLines = [
    "it's a beautiful day outside.",
    "birds are singing, flowers are blooming...",
    "on days like these, kids like you...",
    "S H O U L D  B E  B U R N I N G  I N  H E L L."
  ];
  let introIndex = 0;

  function playNextIntroLine() {
    if (introIndex >= introLines.length) {
      uiBar.style.opacity = "1"; uiBar.style.pointerEvents = "auto"; phase = "PLAYER_TURN";
      dialogueBox.style.display = "block"; showPlayerDialogue("* sans looks relaxed.", null, false, true);
      return;
    }
    showSansDialogue(introLines[introIndex++], () => { playNextIntroLine(); }, true);
  }

  function startIntroSequence() {
    uiBar.style.opacity = "0"; uiBar.style.pointerEvents = "none"; introIndex = 0;
    hidePlayerDialogue(); hideSansDialogue(); playNextIntroLine();
  }

  function updatePlayerHP() {
    if (playerHP < 0) playerHP = 0;
    const hpRatio = playerHP / playerMaxHP;
    const krRatio = kr / playerMaxHP;
    
    hpFill.style.width = (hpRatio * 100) + "%";
    const krFillElem = document.getElementById("kr-fill");
    if (krFillElem) {
      krFillElem.style.width = (krRatio * 100) + "%";
      krFillElem.style.right = (100 - ((hpRatio + krRatio) * 100)) + "%";
    }
    
    hpText.textContent = playerHP + " / " + playerMaxHP;
    if (playerHP <= 0) endBattle(false);
  }

  function updateEnemyHP() {
    if (enemyHP < 0) enemyHP = 0;
    enemyHPBar.style.width = ((enemyHP / enemyMaxHP) * 100) + "%";
    enemyHPText.textContent = "HP: " + enemyHP + " / " + enemyMaxHP;
  }

  function setMenuIndex(index) {
    menuIndex = (index + menuItems.length) % menuItems.length;
    menuElements.forEach((el, i) => el.classList.toggle("selected", i === menuIndex));
  }
  function clearMenuSelection() { menuElements.forEach(el => el.classList.remove("selected")); }

  function enterSoulMode() {
    inSoulMode = true; soulBox.style.display = "block";
    const boxRect = soulBox.getBoundingClientRect();
    soulX = (boxRect.width / 2) - 8; soulY = (boxRect.height / 2) - 8;
    soul.style.left = soulX + "px"; soul.style.top = soulY + "px";
  }

  function exitSoulMode() {
    inSoulMode = false; canMoveSoul = false; soulBox.style.display = "none";
    clearBullets(); setSoulColor("red", false);
  }

  function clearBullets() { document.querySelectorAll(".bone, .blaster, .blast-beam").forEach(b => b.remove()); }

  function rectsOverlap(a, b) { return !(a.right < b.left || a.left > b.right || a.bottom < b.top || a.top > b.bottom); }

  function setSoulColor(color, enableGravity) {
    soulColor = color;
    soul.style.background = (color === "blue" ? "blue" : "red");
    soul.style.boxShadow = color === "blue" ? "0 0 6px rgba(0,128,255,0.9)" : "0 0 6px rgba(255,0,0,0.9)";
    gravityEnabled = enableGravity;
    if (!enableGravity) yVelocity = 0;
  }

  function soulMovementLoop() {
    if (inSoulMode && canMoveSoul) {
      const boxRect = soulBox.getBoundingClientRect();
      if (!gravityEnabled) {
        if (keys["w"] || keys["W"]) soulY -= soulSpeed;
        if (keys["s"] || keys["S"]) soulY += soulSpeed;
      } else {
        yVelocity += gravity; soulY += yVelocity;
        if (soulY >= boxRect.height - 16) { soulY = boxRect.height - 16; yVelocity = 0; isJumping = false; }
        if (soulY <= 0) { soulY = 0; yVelocity = 0; }
        if ((keys["w"] || keys["W"]) && !isJumping) { yVelocity = jumpStrength; isJumping = true; }
        if (!(keys["w"] || keys["W"]) && yVelocity < -1) { yVelocity = -1; }
      }
      if (keys["a"] || keys["A"]) soulX -= soulSpeed;
      if (keys["d"] || keys["D"]) soulX += soulSpeed;
      if (soulX < 0) soulX = 0;
      if (soulX > boxRect.width - 16) soulX = boxRect.width - 16;
      soul.style.left = soulX + "px"; soul.style.top = soulY + "px";
    }
    
    if (kr > 0 && Math.random() < 0.05) {
      kr -= 1; playerHP -= 1;
      if (playerHP <= 1) { playerHP = 1; kr = 0; }
      updatePlayerHP();
    }
    if (invincFrames > 0) invincFrames--;
    requestAnimationFrame(soulMovementLoop);
  }

  function damagePlayer(amount) {
    if (invincFrames > 0) return;
    playerHP -= 1; kr += amount;
    document.getElementById("kr-label").style.display = "inline";
    invincFrames = 2; updatePlayerHP();
  }

  function updateDifficulty() { soulSpeed = baseSoulSpeed + Math.floor(pureAttackTurns / 2) * 0.5; }

  function startSansAttack() {
    if (phase2AActive) { startSansAttackPhase2A(); return; }
    if (phase2CActive) { startSansAttackPhase2C(); return; }
    phase = "ENEMY_ATTACK"; turnCount++; updateDifficulty();
    hideSansDialogue(); hidePlayerDialogue();

    if (!phase2CActive && pureAttackTurns >= 6 && !phase2AActive && !phase2BQueued) {
      phase2CActive = true; showSansDialogue("ya haven't tried anything else, huh?\nguess i'll have to get a bit more serious.", null, true); return;
    }

    if (!usedBlueIntro && turnCount === 1) {
      showSansDialogue("i'll let ya have your turn first, kid.\n...then we get to the fun part.", () => {
        showSansDialogue("what's that look for?\noh right, my blue attack.\nalmost forgot about it.", () => {
          showSansDialogue("ready?\n'cause here it comes.", () => {
            enterSoulMode(); setSoulColor("red", false); canMoveSoul = true;
            setTimeout(() => {
              setSoulColor("blue", true); canMoveSoul = true;
              setTimeout(() => {
                blueAttackSequence(() => {
                  setSoulColor("red", false); exitSoulMode();
                  showSansDialogue("what are you looking so blue for?", () => { phase = "PLAYER_TURN"; dialogueBox.style.display = "block"; showPlayerDialogue("* sans looks amused.", null, false, true); }, true);
                });
              }, 1000);
            }, 3000);
          }, true);
        }, true);
      }, true);
      usedBlueIntro = true; return;
    }

    if (!usedFinalAttackPhase1 && turnCount >= 5 && !phase2BQueued) {
      usedFinalAttackPhase1 = true;
      showSansDialogue("alright. warm‑up's over.\npaps has this 'special attack' thing.\nguess i'll borrow the idea.", () => {
        enterSoulMode(); canMoveSoul = true;
        specialAttackPhase1(() => {
          exitSoulMode();
          showSansDialogue("welp, that sure was fun.\nyou seem pretty much ready for waterfall.\nnow just spare me and we can both be on our way.", () => {
            canSpare = true; sansCanBeHit = true; phase = "PLAYER_TURN"; dialogueBox.style.display = "block"; showPlayerDialogue("* sans looks pretty satisfied.", null, false, true);
          }, true);
        });
      }, true); return;
    }

    enterSoulMode(); canMoveSoul = true;
    const attacks = [pacifistBottomZoneJump, pacifistDoubleSpinBones, pacifistWaveBlueThenSideBones, pacifistTopWavesBlueBottom, pacifistTopBottomAlternating];
    attacks[Math.floor(Math.random() * attacks.length)](() => {
      exitSoulMode();
      if (playerHP > 0) showSansDialogue("nice moves, kid.", () => { phase = "PLAYER_TURN"; dialogueBox.style.display = "block"; showPlayerDialogue("* sans looks a bit more tired.", null, false, true); }, true);
    });
  }

  function startSansAttackPhase2A() {
    phase = "PHASE2A_ATTACK"; phase2ATurns++; enterSoulMode(); canMoveSoul = true;
    if (phase2ATurns >= phase2ATurnTarget) {
      exitSoulMode();
      showSansDialogue("heh. still standin', huh?\nguess that's enough... for both of us.", () => {
        enterSoulMode(); canMoveSoul = true;
        spiralGasterFinal(() => { exitSoulMode(); endBattle(true, false, true); });
      }, true); return;
    }
    phase2ATargetingBonesAndXYBlasters(Math.max(3, phase2AStartTargets - phase2ATurns), () => {
      exitSoulMode();
      if (playerHP > 0) showSansDialogue("still up?\nnot bad.", () => { phase = "PLAYER_TURN"; dialogueBox.style.display = "block"; showPlayerDialogue("* sans looks very tired.", null, false, true); }, true);
    });
  }

  function startSansAttackPhase2C() {
    phase = "PHASE2C_ATTACK"; enterSoulMode(); canMoveSoul = true;
    const attacks = [phase2CBoneFlood, phase2CSlidersCross, phase2CBoneZoneSpam];
    attacks[Math.floor(Math.random() * attacks.length)](() => {
      exitSoulMode();
      if (playerHP > 0) showSansDialogue("getting tired yet?", () => { phase = "PLAYER_TURN"; dialogueBox.style.display = "block"; showPlayerDialogue("* sans is giving you a bad time.", null, false, true); }, true);
    });
  }

  function spawnBone(x, y, w, h) {
    const bone = document.createElement("div"); bone.classList.add("bone");
    bone.style.left = x + "px"; bone.style.top = y + "px"; bone.style.width = w + "px"; bone.style.height = h + "px";
    soulBox.appendChild(bone); return bone;
  }

  function spawnBlaster(x, y, delay, beamDuration, damage, orientation) {
    const blaster = document.createElement("div"); blaster.classList.add("blaster");
    blaster.style.left = x + "px"; blaster.style.top = y + "px"; soulBox.appendChild(blaster);
    const beam = document.createElement("div"); beam.classList.add("blast-beam");

    setTimeout(() => {
      soulBox.appendChild(beam); const boxRect = soulBox.getBoundingClientRect();
      if (orientation === "down") { beam.style.width = "16px"; beam.style.height = boxRect.height + "px"; beam.style.left = (x + 12) + "px"; beam.style.top = "0px"; }
      else if (orientation === "horizontal") { beam.style.width = boxRect.width + "px"; beam.style.height = "16px"; beam.style.left = "0px"; beam.style.top = (y + 12) + "px"; }
      else if (orientation === "diagX") { beam.style.width = "16px"; beam.style.height = boxRect.height + 40 + "px"; beam.style.left = (x + 12) + "px"; beam.style.top = "-20px"; }

      const startTime = performance.now();
      function loop(t) {
        if (!inSoulMode || t - startTime > beamDuration) { beam.remove(); blaster.remove(); return; }
        if (rectsOverlap(beam.getBoundingClientRect(), soul.getBoundingClientRect())) damagePlayer(damage);
        requestAnimationFrame(loop);
      }
      requestAnimationFrame(loop);
    }, delay);
  }

  function pacifistBottomZoneJump(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const ground = spawnBone(0, boxRect.height - 25, boxRect.width, 25);
    const startTime = performance.now();
    for (let i = 0; i < 3; i++) {
      const gap = spawnBone((boxRect.width / 4) * (i + 1) - 10, boxRect.height - 25, 20, 25);
      gap.style.background = "#000"; gap.style.boxShadow = "none";
    }
    function loop(t) {
      if (!inSoulMode || t - startTime > 5000) { ground.remove(); onEnd(); return; }
      if (rectsOverlap(ground.getBoundingClientRect(), soul.getBoundingClientRect())) damagePlayer(1);
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function pacifistDoubleSpinBones(onEnd) {
    const cx = soulBox.clientWidth / 2, cy = soulBox.clientHeight / 2;
    const bone1 = spawnBone(cx - 5, cy - 40, 10, 40), bone2 = spawnBone(cx - 5, cy, 10, 40);
    let angle = 0; const startTime = performance.now();
    function loop(t) {
      if (!inSoulMode || t - startTime > 6000) { bone1.remove(); bone2.remove(); onEnd(); return; }
      angle += 0.03;
      bone1.style.left = (cx + Math.cos(angle) * 35 - 5) + "px"; bone1.style.top = (cy + Math.sin(angle) * 35 - 20) + "px";
      bone2.style.left = (cx + Math.cos(angle + Math.PI) * 35 - 5) + "px"; bone2.style.top = (cy + Math.sin(angle + Math.PI) * 35 - 20) + "px";
      const sRect = soul.getBoundingClientRect();
      if (rectsOverlap(bone1.getBoundingClientRect(), sRect) || rectsOverlap(bone2.getBoundingClientRect(), sRect)) damagePlayer(1);
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function pacifistWaveBlueThenSideBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect(), bones = [];
    const startTime = performance.now();
    for (let i = 0; i < 5; i++) bones.push({ el: spawnBone(-40, 20 + i * 15, 40, 10), x: -40, y: 20 + i * 15, speed: 2 });
    setTimeout(() => { const b = spawnBone(boxRect.width / 2 - 10, boxRect.height - 20, 20, 20); b.style.background = "#0000ff"; bones.push({ el: b, blue: true }); }, 1500);
    setTimeout(() => {
      bones.push({ el: spawnBone(-40, 10, 40, 10), x: -40, y: 10, speed: 3, dir: 1 });
      bones.push({ el: spawnBone(boxRect.width + 40, boxRect.height - 20, 40, 10), x: boxRect.width + 40, y: boxRect.height - 20, speed: 3, dir: -1 });
    }, 3000);
    function loop(t) {
      if (!inSoulMode || t - startTime > 7000) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.blue) { if (rectsOverlap(b.el.getBoundingClientRect(), sRect) && gravityEnabled) damagePlayer(1); }
        else { b.x += b.speed * (b.dir || 1); b.el.style.left = b.x + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(1); }
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function pacifistTopWavesBlueBottom(onEnd) {
    const boxRect = soulBox.getBoundingClientRect(), bones = [], startTime = performance.now();
    for (let i = 0; i < 5; i++) bones.push({ el: spawnBone((boxRect.width / 6) * (i + 1), -20, 10, 30), x: (boxRect.width / 6) * (i + 1), y: -20, speed: 1.8 });
    const blue = spawnBone(boxRect.width / 2 - 15, boxRect.height - 18, 30, 18); blue.style.background = "#0000ff"; bones.push({ el: blue, blue: true });
    function loop(t) {
      if (!inSoulMode || t - startTime > 6500) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.blue) { if (rectsOverlap(b.el.getBoundingClientRect(), sRect) && gravityEnabled) damagePlayer(1); }
        else { b.y += b.speed; b.el.style.top = b.y + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(1); }
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function pacifistTopBottomAlternating(onEnd) {
    const boxRect = soulBox.getBoundingClientRect(), bones = [], startTime = performance.now();
    function spawnRow(fromTop) { for (let i = 0; i < 5; i++) { const x = (boxRect.width / 6) * (i + 1) - 10; const y = fromTop ? -20 : boxRect.height + 20; bones.push({ el: spawnBone(x, y, 16, 30), x, y, speed: fromTop ? 2 : -2 }); } }
    spawnRow(true); setTimeout(() => spawnRow(false), 1200); setTimeout(() => spawnRow(true), 2400);
    function loop(t) {
      if (!inSoulMode || t - startTime > 6500) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      bones.forEach(b => { b.y += b.speed; b.el.style.top = b.y + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(1); });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function blueAttackSequence(onEnd) {
    const bone = spawnBone(0, soulBox.clientHeight - 18, soulBox.clientWidth, 18);
    const startTime = performance.now();
    function loop(t) {
      if (!inSoulMode || t - startTime > 3500) { bone.remove(); onEnd(); return; }
      if (rectsOverlap(bone.getBoundingClientRect(), soul.getBoundingClientRect())) damagePlayer(1);
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function specialAttackPhase1(onEnd) {
    setSoulColor("red", false); const boxRect = soulBox.getBoundingClientRect(), bones = [], startTime = performance.now();
    const floor = spawnBone(0, boxRect.height - 18, boxRect.width, 18); bones.push({ el: floor });
    for (let i = 0; i < 5; i++) {
      const y = 20 + i * 16;
      bones.push({ el: spawnBone(-40, y, 40, 10), x: -40, y, dir: 1, speed: 2.6 });
      bones.push({ el: spawnBone(boxRect.width + 40, y + 8, 40, 10), x: boxRect.width + 40, y: y + 8, dir: -1, speed: 2.6 });
    }
    function loop(t) {
      if (!inSoulMode || t - startTime > 5500) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      if (rectsOverlap(floor.getBoundingClientRect(), sRect)) damagePlayer(1);
      bones.forEach(b => { if (b.dir) { b.x += b.speed * b.dir; b.el.style.left = b.x + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(1); } });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function phase2ATargetingBonesAndXYBlasters(count, onEnd) {
    const boxRect = soulBox.getBoundingClientRect(), bones = [], startTime = performance.now();
    for (let i = 0; i < count; i++) {
      const startY = Math.random() * (boxRect.height - 10), x = Math.random() < 0.5 ? -30 : boxRect.width + 30;
      const len = Math.max(1, Math.sqrt(Math.pow(soulX - x, 2) + Math.pow(soulY - startY, 2)));
      bones.push({ el: spawnBone(x, startY, 20, 10), x, y: startY, vx: ((soulX - x) / len) * 1.8, vy: ((soulY - startY) / len) * 1.8 });
    }
    setTimeout(() => { spawnBlaster(-20, -20, 200, 800, 2, "diagX"); spawnBlaster(boxRect.width - 20, -20, 200, 800, 2, "diagX"); }, 1200);
    setTimeout(() => { spawnBlaster(boxRect.width / 2 - 20, -20, 200, 800, 2, "down"); spawnBlaster(-20, boxRect.height / 2 - 20, 400, 700, 2, "horizontal"); spawnBlaster(boxRect.width - 20, boxRect.height / 2 - 20, 400, 700, 2, "horizontal"); }, 2600);
    function loop(t) {
      if (!inSoulMode || t - startTime > 6500) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      bones.forEach(b => { b.x += b.vx; b.y += b.vy; b.el.style.left = b.x + "px"; b.el.style.top = b.y + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(2); });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function spiralGasterFinal(onEnd) {
    setSoulColor("blue", true); const cx = soulBox.clientWidth / 2, cy = soulBox.clientHeight / 2;
    soulX = cx - 8; soulY = cy - 8; soul.style.left = soulX + "px"; soul.style.top = soulY + "px";
    const startTime = performance.now(); let angle = 0, lastSpawn = 0;
    function loop(t) {
      if (!inSoulMode || t - startTime > 9000) { setSoulColor("red", false); onEnd(); return; }
      if (t - startTime - lastSpawn > 400) {
        lastSpawn = t - startTime; angle += Math.PI / 6;
        spawnBlaster(cx + Math.cos(angle) * (soulBox.clientWidth / 3) - 20, cy + Math.sin(angle) * (soulBox.clientHeight / 3) - 20, 0, 700, 3, "down");
      }
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function phase2CBoneFlood(onEnd) {
    const boxRect = soulBox.getBoundingClientRect(), bones = [], startTime = performance.now();
    for (let i = 0; i < 14; i++) { const fromTop = i % 2 === 0, x = Math.random() * (boxRect.width - 20); bones.push({ el: spawnBone(x, fromTop ? -20 : boxRect.height + 20, 20, 20), x, y: fromTop ? -20 : boxRect.height + 20, speed: fromTop ? 3 : -3 }); }
    function loop(t) {
      if (!inSoulMode || t - startTime > 5000) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      bones.forEach(b => { b.y += b.speed; b.el.style.top = b.y + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(2); });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function phase2CSlidersCross(onEnd) {
    const boxRect = soulBox.getBoundingClientRect(), bones = [], startTime = performance.now();
    for (let i = 0; i < 4; i++) { const fromLeft = i % 2 === 0, y = (boxRect.height / 5) * (i + 1); bones.push({ el: spawnBone(fromLeft ? -40 : boxRect.width + 40, y, 40, 10), x: fromLeft ? -40 : boxRect.width + 40, y, dir: fromLeft ? 1 : -1, speed: 3 }); }
    setTimeout(() => { spawnBlaster(-20, boxRect.height / 2 - 20, 0, 800, 2, "horizontal"); spawnBlaster(boxRect.width - 20, boxRect.height / 2 - 20, 0, 800, 2, "horizontal"); }, 800);
    function loop(t) {
      if (!inSoulMode || t - startTime > 5000) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      bones.forEach(b => { b.x += b.speed * b.dir; b.el.style.left = b.x + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(2); });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function phase2CBoneZoneSpam(onEnd) {
    const boxRect = soulBox.getBoundingClientRect(), bones = [], startTime = performance.now();
    const floor = spawnBone(0, boxRect.height - 18, boxRect.width, 18); bones.push({ el: floor });
    setTimeout(() => { for (let i = 0; i < 4; i++) bones.push({ el: spawnBone((boxRect.width / 5) * (i + 1), -20, 16, 40), x: (boxRect.width / 5) * (i + 1), y: -20, speed: 3 }); }, 1000);
    function loop(t) {
      if (!inSoulMode || t - startTime > 5500) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      if (rectsOverlap(floor.getBoundingClientRect(), sRect)) damagePlayer(2);
      bones.forEach(b => { if (b.speed) { b.y += b.speed; b.el.style.top = b.y + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(2); } });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function startFightBar() { phase = "FIGHT_BAR"; hidePlayerDialogue(); fightBarContainer.style.display = "flex"; fightPointerPos = 0; fightPointerDir = 1; fightBarActive = true; damageText.textContent = ""; }

  function stopFightBar() {
    fightBarActive = false; fightBarContainer.style.display = "none";
    const zoneRect = fightZone.getBoundingClientRect(), barRect = fightBar.getBoundingClientRect();
    const zoneStart = zoneRect.left - barRect.left, zoneEnd = zoneStart + fightZone.clientWidth, pointerX = fightPointerPos * fightBar.clientWidth;
    let quality = "MISS...", hitStrength = 0;
    if (pointerX >= zoneStart && pointerX <= zoneEnd) { quality = "GOOD HIT!"; hitStrength = 1 - (Math.abs(pointerX - (zoneStart + zoneEnd) / 2) / (fightZone.clientWidth / 2)); }
    hideSansDialogue(); hidePlayerDialogue();

    if (phase2AActive) { damageText.textContent = "9999"; showPlayerDialogue("* you strike with everything you have.\n* it doesn't change anything.", null, false, true); }
    else if (!sansCanBeHit) { damageText.textContent = quality + " (sans dodged)"; showSansDialogue("heh.\nclose.\n\nbut not that close.", null, true); }
    else {
      enemyHP -= Math.max(1, Math.floor(hitStrength * 10)); if (enemyHP <= 0) enemyHP = 1; updateEnemyHP();
      damageText.textContent = quality + " - " + Math.max(1, Math.floor(hitStrength * 10));
      showPlayerDialogue("* you land a hit.\n* something feels... wrong.", null, false, true);
      if (!phase2AActive && canSpare) startPhase2A();
    }
    setTimeout(() => { damageText.textContent = ""; if (playerHP > 0 && phase !== "END") { if (phase2AActive) startSansAttackPhase2A(); else if (phase2CActive) startSansAttackPhase2C(); else startSansAttack(); } }, 1000);
  }

  function fightBarLoop() {
    if (fightBarActive) {
      const max = (fightBar.clientWidth - fightPointer.clientWidth) / fightBar.clientWidth;
      fightPointerPos += fightPointerDir * (0.02 + pureAttackTurns * 0.003);
      if (fightPointerPos <= 0) { fightPointerPos = 0; fightPointerDir = 1; } else if (fightPointerPos >= max) { fightPointerPos = max; fightPointerDir = -1; }
      fightPointer.style.left = (fightPointerPos * fightBar.clientWidth) + "px";
    }
    requestAnimationFrame(fightBarLoop);
  }

  function startPhase2A() { phase2AActive = true; phase2ATurns = 0; hideSansDialogue(); hidePlayerDialogue(); showSansDialogue("your blow lands awkwardly.\ni stagger a bit.\nheh. guess that one went a little deep.\ndon't worry.\ni've got... 0.0001 hp left.", null, true); }

  function openSubMenu(type, options) {
    currentSubType = type; currentSubOptions = options; subIndex = 0; subMenu.innerHTML = ""; subMenu.style.display = "flex";
    options.forEach((opt, idx) => { const span = document.createElement("span"); span.classList.add("sub-option"); if (idx === 0) span.classList.add("selected"); span.textContent = opt.label; subMenu.appendChild(span); });
    phase = "MENU_SUB"; hidePlayerDialogue(); clearMenuSelection();
  }

  function closeSubMenu() { subMenu.style.display = "none"; subMenu.innerHTML = ""; currentSubType = null; currentSubOptions = []; phase = "PLAYER_TURN"; setMenuIndex(menuIndex); if (currentPlayerStatusText) { dialogueBox.style.display = "block"; restorePlayerStatusWithTypewriter(); } else hidePlayerDialogue(); }

  function setSubIndex(index) { if (!currentSubOptions.length) return; subIndex = (index + currentSubOptions.length) % currentSubOptions.length; Array.from(subMenu.querySelectorAll(".sub-option")).forEach((s, i) => s.classList.toggle("selected", i === subIndex)); }

  function confirmSubSelection() {
    if (!currentSubOptions.length) return; const opt = currentSubOptions[subIndex];
    if (currentSubType === "ACT") {
      actCount++; hideSansDialogue(); hidePlayerDialogue(); dialogueBox.style.display = "block";
      if (opt.id === "CHECK") showPlayerDialogue("* sans - atk 1 def 1.\n* the second skeleton brother stands firm.", null, false, false);
      else if (opt.id === "JOKE") showPlayerDialogue("* you tell sans a joke.", () => showSansDialogue("heh.\nnot bad, kid.", null, true), true, false);
      else if (opt.id === "FLIRT") showPlayerDialogue("* you try to flirt.", () => showSansDialogue("you know i'm not the tall one, right?", null, true), true, false);
      else if (opt.id === "TALK") showPlayerDialogue("* you talk about paps.", () => showSansDialogue("yeah.\nhe's really lookin' forward to you.", null, true), true, false);
      closeSubMenu(); setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1500);
    }
    else if (currentSubType === "ITEM") {
      const item = items.find(i => i.id === opt.id); hideSansDialogue(); hidePlayerDialogue(); dialogueBox.style.display = "block";
      playerHP = Math.min(playerMaxHP, playerHP + item.heal); updatePlayerHP();
      showPlayerDialogue("* you eat the " + item.name.toLowerCase() + ".\n* you recovered " + item.heal + " hp.", null, false, false);
      items = items.filter(i => i.id !== opt.id); closeSubMenu(); setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1500);
    }
    else if (currentSubType === "MERCY") {
      hideSansDialogue(); hidePlayerDialogue(); dialogueBox.style.display = "block";
      if (opt.id === "SPARE") {
        if (!canSpare || phase2AActive || phase2CActive) { showPlayerDialogue("* you reach for MERCY.", () => showSansDialogue("sorry, pal.\nno mercy 'till you prove yourself.", null, true), true, false); closeSubMenu(); setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1600); return; }
        showPlayerDialogue("* you lower your hands.", () => showSansDialogue("heh.\nnot bad.\nhow 'bout one last test,\nthen we both move on?", null, true), true, false);
        phase2BQueued = true; closeSubMenu(); setTimeout(() => { phase = "PHASE2B_SPECIAL"; enterSoulMode(); canMoveSoul = true; phase2BFinalTest(() => { exitSoulMode(); endBattle(true, true, false); }); }, 1600);
      } else { showPlayerDialogue("* you think about running away...\n* but your feet won't move.", null, false, false); closeSubMenu(); setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1500); }
    }
  }

  function phase2BFinalTest(onEnd) {
    const boxRect = soulBox.getBoundingClientRect(), bones = [], startTime = performance.now();
    for (let i = 0; i < 5; i++) bones.push({ el: spawnBone(-40, 20 + i * 16, 40, 10), x: -40, y: 20 + i * 16, dir: 1, speed: 2.2 });
    setTimeout(() => { const blue = spawnBone(boxRect.width / 2 - 10, boxRect.height - 18, 20, 18); blue.style.background = "#0000ff"; bones.push({ el: blue, blue: true }); }, 1800);
    setTimeout(() => { spawnBlaster(boxRect.width / 2 - 20, -20, 0, 900, 2, "down"); }, 3500);
    function loop(t) {
      if (!inSoulMode || t - startTime > 8000) { bones.forEach(b => b.el.remove()); onEnd(); return; }
      const sRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.blue) { if (rectsOverlap(b.el.getBoundingClientRect(), sRect) && gravityEnabled) damagePlayer(1); }
        else if (b.dir) { b.x += b.speed * b.dir; b.el.style.left = b.x + "px"; if (rectsOverlap(b.el.getBoundingClientRect(), sRect)) damagePlayer(1); }
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function endBattle(playerWon, spared = false, killedSans = false) {
    endScreen.style.display = "flex";
    if (!playerWon) endText.textContent = "YOU DIED.\n\n* whoops.\n* guess that was a bit much.\n* sorry, kid.";
    else if (spared) endText.textContent = "YOU WON!\nGained 20G.\n\n* you feel like you're ready for what's next.";
    else if (killedSans) endText.textContent = "YOU WON.\nEXP: 1\nG: 10\n\n* the dust settles.\n* jokes feel a lot heavier now.";
    else endText.textContent = "YOU WON!";
    phase = "END";
  }

  function confirmMenuSelection() {
    if (phase !== "PLAYER_TURN" || sansDialogueBox.style.display === "block" || performance.now() - lastActionTime < ACTION_COOLDOWN) return;
    lastActionTime = performance.now(); const action = menuItems[menuIndex]; hideSansDialogue();
    if (action === "FIGHT") { hidePlayerDialogue(); pureAttackTurns++; showPlayerDialogue("* you get ready to attack.", () => startFightBar(), true, false); }
    else if (action === "ACT") openSubMenu("ACT", [{ id: "CHECK", label: "Check" }, { id: "JOKE", label: "Joke" }, { id: "FLIRT", label: "Flirt" }, { id: "TALK", label: "Talk" }]);
    else if (action === "ITEM") { if (items.length === 0) { hidePlayerDialogue(); dialogueBox.style.display = "block"; showPlayerDialogue("* (you don’t have any items.)", null, false, false); setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1500); } else openSubMenu("ITEM", items.map(it => ({ id: it.id, label: it.name }))); }
    else if (action === "MERCY") openSubMenu("MERCY", [{ id: "SPARE", label: canSpare && !phase2AActive && !phase2CActive ? "Spare (yellow)" : "Spare" }, { id: "FLEE", label: "Flee" }]);
  }

  document.addEventListener("keydown", (e) => {
    keys[e.key] = true; if (phase === "END") return;
    if (e.key === "z" || e.key === "Z") {
      if (waitingForDialogueAdvance || !sansTextDone || !playerTextDone) { handleDialogueZ(); return; }
      if (phase === "PLAYER_TURN") confirmMenuSelection(); else if (phase === "MENU_SUB") confirmSubSelection(); else if (phase === "FIGHT_BAR") stopFightBar();
    }
    if ((e.key === "x" || e.key === "X") && phase === "MENU_SUB") closeSubMenu();
    if (phase === "PLAYER_TURN" && sansDialogueBox.style.display !== "block") { if (e.key === "a" || e.key === "A") setMenuIndex(menuIndex - 1); else if (e.key === "d" || e.key === "D") setMenuIndex(menuIndex + 1); }
    else if (phase === "MENU_SUB") { if (e.key === "a" || e.key === "A") setSubIndex(subIndex - 1); else if (e.key === "d" || e.key === "D") setSubIndex(subIndex + 1); }
  });

  document.addEventListener("keyup", (e) => keys[e.key] = false);

  updatePlayerHP(); updateEnemyHP(); setMenuIndex(0); startIntroSequence();
  requestAnimationFrame(soulMovementLoop); requestAnimationFrame(fightBarLoop);
</script>
</body>
</html>
