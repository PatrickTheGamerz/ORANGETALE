<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>A Test of Skill - Sans Fight</title>
  <style>
    body {
      margin: 0;
      background: black;
      color: white;
      font-family: "Courier New", monospace;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    #game {
      width: 800px;
      height: 600px;
      border: 4px solid white;
      box-sizing: border-box;
      position: relative;
      background: radial-gradient(circle at top, #3a2950 0, #050008 70%);
      overflow: hidden;
    }

    /* Enemy area */
    #enemy-area {
      position: absolute;
      top: 20px;
      left: 0;
      width: 100%;
      height: 180px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      opacity: 1;
      pointer-events: none;
    }
    #enemy-name {
      font-size: 24px;
      margin-bottom: 8px;
      text-transform: lowercase;
    }
    #enemy-sprite {
      width: 96px;
      height: 96px;
      border: 2px solid white;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 8px;
      font-size: 12px;
      box-shadow: 0 0 10px rgba(255,255,255,0.4);
      position: relative;
    }
    #enemy-hp-container {
      width: 300px;
      height: 16px;
      border: 2px solid white;
      position: relative;
      box-shadow: 0 0 4px rgba(0,255,255,0.8);
    }
    #enemy-hp-bar {
      position: absolute;
      top: 0;
      left: 0;
      height: 100%;
      background: cyan;
      width: 100%;
    }
    #enemy-hp-text {
      margin-top: 4px;
      font-size: 14px;
    }

    /* Player dialogue box */
    #dialogue-box {
      position: absolute;
      bottom: 150px;
      left: 20px;
      right: 20px;
      min-height: 40px;
      max-height: 200px;
      border: 4px solid white;
      padding: 10px;
      box-sizing: border-box;
      font-size: 18px;
      white-space: pre-line;
      background: rgba(0,0,0,0.8);
      overflow: hidden;
      display: none;
    }

    /* Sans dialogue box */
    #sans-dialogue-box {
      position: absolute;
      top: 60px;
      left: 480px;
      min-height: 40px;
      max-height: 160px;
      border: 4px solid white;
      box-sizing: border-box;
      padding: 8px;
      background: white;
      color: black;
      font-size: 18px;
      white-space: pre-line;
      display: none;
      overflow: hidden;
    }

    /* Soul box */
    #soul-box {
      position: absolute;
      bottom: 150px;
      left: 260px;
      width: 280px;
      height: 120px;
      border: 4px solid white;
      box-sizing: border-box;
      overflow: hidden;
      display: none;
      background: rgba(0,0,0,0.6);
    }
    #soul {
      width: 16px;
      height: 16px;
      background: red;
      position: absolute;
      border-radius: 3px;
      box-shadow: 0 0 6px rgba(255,0,0,0.9);
    }
    .bone {
      position: absolute;
      background: white;
      box-shadow: 0 0 4px rgba(255,255,255,0.8);
    }
    .blaster {
      position: absolute;
      width: 40px;
      height: 40px;
      border: 2px solid white;
      border-radius: 50%;
      box-sizing: border-box;
      box-shadow: 0 0 8px rgba(0,255,255,0.9);
    }
    .blast-beam {
      position: absolute;
      background: cyan;
      opacity: 0.8;
      box-shadow: 0 0 15px rgba(0,255,255,1);
    }

    /* Bottom UI */
    #ui-bar {
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 130px;
      border-top: 4px solid white;
      box-sizing: border-box;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      padding: 8px 16px;
      background: rgba(0,0,0,0.9);
      opacity: 0;
      pointer-events: none;
    }
    #ui-menu {
      display: flex;
      gap: 20px;
      font-size: 22px;
    }
    .menu-item {
      cursor: pointer;
    }
    .menu-item.selected {
      color: yellow;
    }

    #hp-bar {
      display: flex;
      align-items: center;
      gap: 10px;
      font-size: 16px;
    }
    #hp-bar-inner {
      width: 200px;
      height: 16px;
      border: 2px solid white;
      position: relative;
    }
    #hp-fill {
      position: absolute;
      top: 0;
      left: 0;
      height: 100%;
      background: #ffb000;
      width: 100%;
    }

    /* Sub-menu */
    #sub-menu {
      position: absolute;
      bottom: 130px;
      left: 0;
      width: 100%;
      height: 80px;
      display: none;
      justify-content: center;
      align-items: center;
      gap: 20px;
      font-size: 18px;
      pointer-events: auto;
    }
    .sub-option {
      border: 2px solid white;
      padding: 6px 12px;
      cursor: pointer;
    }
    .sub-option.selected {
      color: yellow;
      border-color: yellow;
    }

    /* Fight bar */
    #fight-bar-container {
      position: absolute;
      bottom: 230px;
      left: 50%;
      transform: translateX(-50%);
      width: 400px;
      height: 60px;
      display: none;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      color: white;
    }
    #fight-bar {
      width: 360px;
      height: 20px;
      border: 2px solid white;
      position: relative;
      margin-bottom: 8px;
      background: black;
    }
    #fight-zone {
      position: absolute;
      top: 0;
      height: 100%;
      width: 80px;
      background: yellow;
      left: 140px;
      opacity: 0.4;
    }
    #fight-pointer {
      position: absolute;
      top: 0;
      width: 4px;
      height: 100%;
      background: red;
      box-shadow: 0 0 6px rgba(255,0,0,0.8);
    }
    #damage-text {
      font-size: 16px;
      height: 20px;
    }

    /* End screen */
    #end-screen {
      position: absolute;
      inset: 0;
      background: black;
      color: white;
      display: none;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      font-size: 24px;
      text-align: center;
      white-space: pre-line;
    }
  </style>
</head>
<body>
<div id="game">
  <div id="enemy-area">
    <div id="enemy-name">sans</div>
    <div id="enemy-sprite">S A N S</div>
    <div id="enemy-hp-container">
      <div id="enemy-hp-bar"></div>
    </div>
    <div id="enemy-hp-text">HP: 1 / 1</div>
  </div>

  <div id="dialogue-box"></div>
  <div id="sans-dialogue-box"></div>

  <div id="soul-box">
    <div id="soul"></div>
  </div>

  <div id="fight-bar-container">
    <div id="fight-bar">
      <div id="fight-zone"></div>
      <div id="fight-pointer"></div>
    </div>
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
      <div id="hp-bar-inner">
        <div id="hp-fill"></div>
      </div>
      <span id="hp-text">20 / 20</span>
    </div>
  </div>

  <div id="end-screen">
    <div id="end-text"></div>
    <div style="font-size: 18px; margin-top: 10px;">(refresh the page to restart)</div>
  </div>
</div>

<script>
  // ========= CORE STATE =========
  let playerHP = 20;
  const playerMaxHP = 20;
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

  // ========= SOUL STATE =========
  let soulX = 0;
  let soulY = 0;
  let soulColor = "red";
  let gravityEnabled = false;
  let gravity = 0.12;
  let yVelocity = 0;
  let baseSoulSpeed = 3.5;
  let soulSpeed = baseSoulSpeed;

  const keys = {};
  let currentSubType = null;
  let currentSubOptions = [];
  let subIndex = 0;

  let fightPointerPos = 0;
  let fightPointerDir = 1;
  let fightBarActive = false;

  let items = [
    { id: "PIE", name: "Butterscotch Pie", heal: 20 },
    { id: "CREAM", name: "Nice Cream", heal: 12 },
    { id: "BANDAGE", name: "Bandage", heal: 7 }
  ];

  // ========= DIALOGUE SYSTEM =========
  const BASE_CHAR_DELAY = 50;
  const COMMA_DELAY = 150;
  const DOT_DELAY = 1500;
  const AUTO_ADVANCE_AFTER_END = 1500; // after text is fully typed AND ends with . or ?
  const MIN_ADVANCE_DELAY = 500;       // 0.5s before Z is allowed

  let playerTyping = null;
  let sansTyping = null;
  let playerTextFull = "";
  let sansTextFull = "";
  let playerTextDone = true;
  let sansTextDone = true;
  let playerTextFinishTime = 0;
  let sansTextFinishTime = 0;

  let waitingForDialogueAdvance = false;
  let dialogueAdvanceCallback = null;
  let dialogueLastTarget = null;

  let currentPlayerStatusText = "";

  function clearTypewriter(target) {
    if (target === "player" && playerTyping) {
      clearTimeout(playerTyping);
      playerTyping = null;
    }
    if (target === "sans" && sansTyping) {
      clearTimeout(sansTyping);
      sansTyping = null;
    }
  }

  function charDelay(ch) {
    if (ch === "." || ch === "?") return DOT_DELAY;
    if (ch === ",") return COMMA_DELAY;
    return BASE_CHAR_DELAY;
  }

  function typeText(element, fullText, target, onComplete) {
    clearTypewriter(target);
    element.textContent = "";
    let i = 0;

    if (target === "player") {
      playerTextFull = fullText;
      playerTextDone = false;
    } else {
      sansTextFull = fullText;
      sansTextDone = false;
    }

    function markFinished() {
      const now = performance.now();
      const lastChar = fullText.trim().slice(-1);
      if (target === "player") {
        playerTyping = null;
        playerTextDone = true;
        playerTextFinishTime = now;
      } else {
        sansTyping = null;
        sansTextDone = true;
        sansTextFinishTime = now;
      }

      // Auto-advance after 1.5s if last char is . or ?
      if (lastChar === "." || lastChar === "?") {
        setTimeout(() => {
          if (!waitingForDialogueAdvance) return;
          if (onComplete) onComplete();
        }, AUTO_ADVANCE_AFTER_END);
      } else {
        if (onComplete) onComplete();
      }
    }

    function step() {
      if (target === "player" && playerTyping === null && i > 0) return;
      if (target === "sans" && sansTyping === null && i > 0) return;

      if (i > fullText.length) return;

      const current = fullText.slice(0, i);
      element.textContent = current;

      if (i === fullText.length) {
        markFinished();
        return;
      }

      const prevChar = fullText.charAt(i - 1) || "";
      const delay = charDelay(prevChar);

      i++;
      const id = setTimeout(step, delay);
      if (target === "player") playerTyping = id;
      else sansTyping = id;
    }

    i = 1;
    const firstDelay = charDelay(fullText.charAt(0) || "");
    const id = setTimeout(step, firstDelay);
    if (target === "player") playerTyping = id;
    else sansTyping = id;
  }

  function finishTextInstant(target) {
    if (target === "player" && !playerTextDone) {
      clearTypewriter("player");
      dialogueBox.textContent = playerTextFull;
      playerTextDone = true;
      playerTextFinishTime = performance.now();
    }
    if (target === "sans" && !sansTextDone) {
      clearTypewriter("sans");
      sansDialogueBox.textContent = sansTextFull;
      sansTextDone = true;
      sansTextFinishTime = performance.now();
    }
  }

  function canAdvance(target) {
    const now = performance.now();
    if (target === "player" && playerTextDone) {
      return (now - playerTextFinishTime) >= MIN_ADVANCE_DELAY;
    }
    if (target === "sans" && sansTextDone) {
      return (now - sansTextFinishTime) >= MIN_ADVANCE_DELAY;
    }
    return false;
  }

  function beginDialogue(text, target, callback, allowAdvance) {
    waitingForDialogueAdvance = allowAdvance;
    dialogueAdvanceCallback = callback || null;
    dialogueLastTarget = target;

    if (target === "player") {
      dialogueBox.style.display = "block";
      typeText(dialogueBox, text, "player", () => {
        if (!allowAdvance && callback) callback();
      });
    } else {
      sansDialogueBox.style.display = "block";
      typeText(sansDialogueBox, text, "sans", () => {
        if (!allowAdvance && callback) {
          sansDialogueBox.style.display = "none";
          callback();
        }
      });
    }
  }

  function doDialogueAdvance() {
    if (!waitingForDialogueAdvance) return;
    waitingForDialogueAdvance = false;
    const cb = dialogueAdvanceCallback;
    dialogueAdvanceCallback = null;
    if (cb) cb();
  }

  function handleDialogueZ() {
    if (dialogueLastTarget === "sans" && !sansTextDone) {
      finishTextInstant("sans");
      return;
    }
    if (dialogueLastTarget === "player" && !playerTextDone) {
      finishTextInstant("player");
      return;
    }
    if (!canAdvance(dialogueLastTarget)) return;
    doDialogueAdvance();
  }

  function showPlayerDialogue(text, callback, allowAdvance, isStatus) {
    if (isStatus) currentPlayerStatusText = text;
    dialogueBox.style.display = "block";
    beginDialogue(text, "player", callback || null, !!allowAdvance);
  }

  function showSansDialogue(text, callback, allowAdvance) {
    beginDialogue(text, "sans", () => {
      sansDialogueBox.style.display = "none";
      if (callback) callback();
    }, !!allowAdvance);
  }

  function hidePlayerDialogue() {
    clearTypewriter("player");
    dialogueBox.textContent = "";
    dialogueBox.style.display = "none";
    playerTextFull = "";
    playerTextDone = true;
  }

  function hideSansDialogue() {
    clearTypewriter("sans");
    sansDialogueBox.textContent = "";
    sansDialogueBox.style.display = "none";
    sansTextFull = "";
    sansTextDone = true;
  }

  function restorePlayerStatusWithTypewriter() {
    if (!currentPlayerStatusText) return;
    showPlayerDialogue(currentPlayerStatusText, null, false, true);
  }

  // ========= INTRO SEQUENCE =========
  const introLines = [
    "heya.",
    "so.\nwe're really doin' this, huh?",
    "paps is busy messin' with his \"special attack\".",
    "so i figured i'd give you a little test of my own.",
    "nothing too crazy.\njust enough to see what you're made of."
  ];
  let introIndex = 0;

  function playNextIntroLine() {
    if (introIndex >= introLines.length) {
      uiBar.style.opacity = "1";
      uiBar.style.pointerEvents = "auto";
      phase = "PLAYER_TURN";
      dialogueBox.style.display = "block";
      showPlayerDialogue("* sans looks relaxed.", null, false, true);
      return;
    }
    const text = introLines[introIndex];
    introIndex++;
    showSansDialogue(text, () => {
      playNextIntroLine();
    }, true);
  }

  function startIntroSequence() {
    uiBar.style.opacity = "0";
    uiBar.style.pointerEvents = "none";
    introIndex = 0;
    hidePlayerDialogue();
    hideSansDialogue();
    playNextIntroLine();
  }

  function showDialogue(text) {
    showPlayerDialogue(text, null, false, true);
  }

  function updatePlayerHP() {
    if (playerHP < 0) playerHP = 0;
    const ratio = playerHP / playerMaxHP;
    hpFill.style.width = (ratio * 100) + "%";
    hpText.textContent = playerHP + " / " + playerMaxHP;
    if (playerHP <= 0) endBattle(false);
  }

  function updateEnemyHP() {
    if (enemyHP < 0) enemyHP = 0;
    const ratio = enemyHP / enemyMaxHP;
    enemyHPBar.style.width = (ratio * 100) + "%";
    enemyHPText.textContent = "HP: " + enemyHP + " / " + enemyMaxHP;
  }

  function setMenuIndex(index) {
    menuIndex = (index + menuItems.length) % menuItems.length;
    menuElements.forEach((el, i) => {
      el.classList.toggle("selected", i === menuIndex);
    });
  }

  function enterSoulMode() {
    inSoulMode = true;
    soulBox.style.display = "block";
    const boxRect = soulBox.getBoundingClientRect();
    soulX = (boxRect.width / 2) - 8;
    soulY = (boxRect.height / 2) - 8;
    soul.style.left = soulX + "px";
    soul.style.top = soulY + "px";
  }

  function exitSoulMode() {
    inSoulMode = false;
    canMoveSoul = false;
    soulBox.style.display = "none";
    clearBullets();
    setSoulColor("red", false);
  }

  function clearBullets() {
    const bullets = soulBox.querySelectorAll(".bone, .blaster, .blast-beam");
    bullets.forEach(b => b.remove());
  }

  function rectsOverlap(a, b) {
    return !(
      a.right < b.left ||
      a.left > b.right ||
      a.bottom < b.top ||
      a.top > b.bottom
    );
  }

  function setSoulColor(color, enableGravity) {
    soulColor = color;
    soul.style.background = (color === "blue" ? "blue" : "red");
    soul.style.boxShadow = color === "blue"
      ? "0 0 6px rgba(0,128,255,0.9)"
      : "0 0 6px rgba(255,0,0,0.9)";
    gravityEnabled = enableGravity;
    if (!enableGravity) yVelocity = 0;
  }

  function soulMovementLoop() {
    if (inSoulMode && canMoveSoul) {
      const boxRect = soulBox.getBoundingClientRect();
      const boxWidth = boxRect.width;
      const boxHeight = boxRect.height;

      if (!gravityEnabled) {
        if (keys["w"] || keys["W"]) soulY -= soulSpeed;
        if (keys["s"] || keys["S"]) soulY += soulSpeed;
      } else {
        if (keys["w"] || keys["W"]) yVelocity -= 0.3;
        yVelocity += gravity;
        soulY += yVelocity;
      }

      if (keys["a"] || keys["A"]) soulX -= soulSpeed;
      if (keys["d"] || keys["D"]) soulX += soulSpeed;

      if (soulX < 0) soulX = 0;
      if (soulX > boxWidth - 16) soulX = boxWidth - 16;
      if (soulY < 0) {
        soulY = 0;
        if (gravityEnabled) yVelocity = 0;
      }
      if (soulY > boxHeight - 16) {
        soulY = boxHeight - 16;
        if (gravityEnabled) yVelocity = 0;
      }

      soul.style.left = soulX + "px";
      soul.style.top = soulY + "px";
    }
    requestAnimationFrame(soulMovementLoop);
  }

  function damagePlayer(amount) {
    playerHP -= amount;
    updatePlayerHP();
  }

  function updateDifficulty() {
    const pureFactor = Math.floor(pureAttackTurns / 2);
    soulSpeed = baseSoulSpeed + pureFactor * 0.5;
  }

  // ========= ATTACK DISPATCH =========
  function startSansAttack() {
    if (phase2AActive) {
      startSansAttackPhase2A();
      return;
    }
    if (phase2CActive) {
      startSansAttackPhase2C();
      return;
    }

    phase = "ENEMY_ATTACK";
    turnCount++;
    updateDifficulty();

    hideSansDialogue();
    hidePlayerDialogue();

    // pure fight route -> phase2C
    if (!phase2CActive && pureAttackTurns >= 6 && !phase2AActive && !phase2BQueued) {
      phase2CActive = true;
      showSansDialogue(
        "ya haven't tried anything else, huh?\n" +
        "guess i'll have to get a bit more serious.",
        null,
        true
      );
      return;
    }

    // blue attack intro
    if (!usedBlueIntro && turnCount === 1) {
      showSansDialogue(
        "i'll let ya have your turn first, kid.\n" +
        "...then we get to the fun part.",
        () => {
          showSansDialogue(
            "what's that look for?\n" +
            "oh right, my blue attack.\n" +
            "almost forgot about it, heh heh heh.",
            () => {
              showSansDialogue("ready?\n'cause here it comes.", () => {
                enterSoulMode();
                setSoulColor("red", false);
                canMoveSoul = true;

                setTimeout(() => {
                  setSoulColor("blue", true);
                  canMoveSoul = true;
                  setTimeout(() => {
                    blueAttackSequence(() => {
                      setSoulColor("red", false);
                      exitSoulMode();
                      showSansDialogue("what are you looking so blue for?", () => {
                        phase = "PLAYER_TURN";
                        showPlayerDialogue("* sans looks amused.", null, false, true);
                      }, true);
                    });
                  }, 1000);
                }, 3000);
              }, true);
            },
            true
          );
        },
        true
      );
      usedBlueIntro = true;
      return;
    }

    // phase 1 special
    if (!usedFinalAttackPhase1 && turnCount >= 5 && !phase2BQueued) {
      usedFinalAttackPhase1 = true;
      showSansDialogue(
        "alright. warm‑up's over.\n" +
        "paps has this 'special attack' thing.\n" +
        "guess i'll borrow the idea.",
        () => {
          enterSoulMode();
          canMoveSoul = true;
          specialAttackPhase1(() => {
            exitSoulMode();
            showSansDialogue(
              "welp, that sure was fun.\n" +
              "you seem pretty much ready for waterfall.\n" +
              "now just spare me and we can both be on our way.",
              () => {
                canSpare = true;
                sansCanBeHit = true;
                phase = "PLAYER_TURN";
                showPlayerDialogue("* sans looks pretty satisfied.", null, false, true);
              },
              true
            );
          });
        },
        true
      );
      return;
    }

    // normal phase 1 attacks
    enterSoulMode();
    canMoveSoul = true;
    const attacks = [
      pacifistBottomZoneJump,
      pacifistDoubleSpinBones,
      pacifistWaveBlueThenSideBones,
      pacifistTopWavesBlueBottom,
      pacifistTopBottomAlternating
    ];
    const attack = attacks[Math.floor(Math.random() * attacks.length)];
    attack(() => {
      exitSoulMode();
      if (playerHP > 0) {
        showSansDialogue("nice moves, kid.", () => {
          phase = "PLAYER_TURN";
          showPlayerDialogue("* sans looks a bit more tired.", null, false, true);
        }, true);
      }
    });
  }

  // ========= PHASE 2A ATTACKS =========
  function startSansAttackPhase2A() {
    phase = "PHASE2A_ATTACK";
    phase2ATurns++;

    enterSoulMode();
    canMoveSoul = true;

    if (phase2ATurns >= phase2ATurnTarget) {
      exitSoulMode();
      showSansDialogue(
        "heh. still standin', huh?\n" +
        "guess that's enough... for both of us.",
        () => {
          enterSoulMode();
          canMoveSoul = true;
          spiralGasterFinal(() => {
            exitSoulMode();
            endBattle(true, false, true);
          });
        },
        true
      );
      return;
    }

    const remainingFactor = Math.max(0, phase2AStartTargets - phase2ATurns);
    const numTargetBones = Math.max(3, remainingFactor);

    phase2ATargetingBonesAndXYBlasters(numTargetBones, () => {
      exitSoulMode();
      if (playerHP > 0) {
        showSansDialogue("still up?\nnot bad.", () => {
          phase = "PLAYER_TURN";
          showPlayerDialogue("* sans looks very tired.", null, false, true);
        }, true);
      }
    });
  }

  // ========= PHASE 2C ATTACKS WRAPPER =========
  function startSansAttackPhase2C() {
    phase = "PHASE2C_ATTACK";
    enterSoulMode();
    canMoveSoul = true;
    const attacks = [
      phase2CBoneFlood,
      phase2CSlidersCross,
      phase2CBoneZoneSpam
    ];
    const attack = attacks[Math.floor(Math.random() * attacks.length)];
    attack(() => {
      exitSoulMode();
      if (playerHP > 0) {
        showSansDialogue("getting tired yet?", () => {
          phase = "PLAYER_TURN";
          showPlayerDialogue("* sans is giving you a bad time.", null, false, true);
        }, true);
      }
    });
  }

  // ========= BASIC ATTACK HELPERS =========
  function spawnBone(x, y, w, h) {
    const bone = document.createElement("div");
    bone.classList.add("bone");
    bone.style.left = x + "px";
    bone.style.top = y + "px";
    bone.style.width = w + "px";
    bone.style.height = h + "px";
    soulBox.appendChild(bone);
    return bone;
  }

  function spawnBlaster(x, y, delay, beamDuration, damage, orientation) {
    const blaster = document.createElement("div");
    blaster.classList.add("blaster");
    blaster.style.left = x + "px";
    blaster.style.top = y + "px";
    soulBox.appendChild(blaster);

    const beam = document.createElement("div");
    beam.classList.add("blast-beam");

    setTimeout(() => {
      soulBox.appendChild(beam);
      const boxRect = soulBox.getBoundingClientRect();
      if (orientation === "down") {
        beam.style.width = "16px";
        beam.style.height = boxRect.height + "px";
        beam.style.left = (x + 12) + "px";
        beam.style.top = "0px";
      } else if (orientation === "horizontal") {
        beam.style.width = boxRect.width + "px";
        beam.style.height = "16px";
        beam.style.left = "0px";
        beam.style.top = (y + 12) + "px";
      } else if (orientation === "diagX") {
        beam.style.width = "16px";
        beam.style.height = boxRect.height + 40 + "px";
        beam.style.left = (x + 12) + "px";
        beam.style.top = "-20px";
      }

      const startTime = performance.now();
      function loop(t) {
        if (!inSoulMode) {
          beam.remove();
          blaster.remove();
          return;
        }
        if (t - startTime > beamDuration) {
          beam.remove();
          blaster.remove();
          return;
        }
        const soulRect = soul.getBoundingClientRect();
        const rect = beam.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(damage);
        requestAnimationFrame(loop);
      }
      requestAnimationFrame(loop);
    }, delay);
  }

  // ========= PHASE 1 ATTACKS =========
  function pacifistBottomZoneJump(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const groundZoneHeight = 25;
    const duration = 5000;
    const startTime = performance.now();

    const ground = spawnBone(0, height - groundZoneHeight, width, groundZoneHeight);
    const gaps = 3;
    const gapWidth = 20;
    for (let i = 0; i < gaps; i++) {
      const gx = (width / (gaps + 1)) * (i + 1) - gapWidth / 2;
      const gap = spawnBone(gx, height - groundZoneHeight, gapWidth, groundZoneHeight);
      gap.style.background = "#000";
      gap.style.boxShadow = "none";
    }

    function loop(t) {
      if (!inSoulMode) {
        ground.remove();
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        ground.remove();
        onEnd();
        return;
      }
      const soulRect = soul.getBoundingClientRect();
      const rect = ground.getBoundingClientRect();
      if (rectsOverlap(rect, soulRect)) damagePlayer(1);
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function pacifistDoubleSpinBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const cx = boxRect.width / 2;
    const cy = boxRect.height / 2;

    const bone1 = spawnBone(cx - 5, cy - 40, 10, 40);
    const bone2 = spawnBone(cx - 5, cy, 10, 40);

    let angle = 0;
    const duration = 6000;
    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        bone1.remove();
        bone2.remove();
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bone1.remove();
        bone2.remove();
        onEnd();
        return;
      }

      angle += 0.03;
      const r = 35;
      const x1 = cx + Math.cos(angle) * r;
      const y1 = cy + Math.sin(angle) * r - 20;
      const x2 = cx + Math.cos(angle + Math.PI) * r;
      const y2 = cy + Math.sin(angle + Math.PI) * r - 20;

      bone1.style.left = (x1 - 5) + "px";
      bone1.style.top = y1 + "px";
      bone2.style.left = (x2 - 5) + "px";
      bone2.style.top = y2 + "px";

      const soulRect = soul.getBoundingClientRect();
      const r1 = bone1.getBoundingClientRect();
      const r2 = bone2.getBoundingClientRect();
      if (rectsOverlap(r1, soulRect) || rectsOverlap(r2, soulRect)) damagePlayer(1);

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function pacifistWaveBlueThenSideBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const bones = [];
    const duration = 7000;
    const startTime = performance.now();

    for (let i = 0; i < 5; i++) {
      const bone = spawnBone(-40, 20 + i * 15, 40, 10);
      bones.push({ el: bone, x: -40, y: 20 + i * 15, speed: 2 });
    }

    setTimeout(() => {
      const blue = spawnBone(width / 2 - 10, height - 20, 20, 20);
      blue.style.background = "#0000ff";
      blue.style.boxShadow = "0 0 8px rgba(0,128,255,0.9)";
      bones.push({ el: blue, blue: true });
    }, 1500);

    setTimeout(() => {
      const top = spawnBone(-40, 10, 40, 10);
      const bottom = spawnBone(width + 40, height - 20, 40, 10);
      bones.push({ el: top, x: -40, y: 10, speed: 3, dir: 1 });
      bones.push({ el: bottom, x: width + 40, y: height - 20, speed: 3, dir: -1 });
    }, 3000);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.blue) {
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect) && gravityEnabled) damagePlayer(1);
        } else {
          if (b.dir == null) b.dir = 1;
          b.x += b.speed * b.dir;
          b.el.style.left = b.x + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(1);
        }
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function pacifistTopWavesBlueBottom(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 6500;
    const startTime = performance.now();

    for (let i = 0; i < 5; i++) {
      const x = (width / 6) * (i + 1);
      const bone = spawnBone(x, -20, 10, 30);
      bones.push({ el: bone, x, y: -20, speed: 1.8 });
    }

    const blue = spawnBone(width / 2 - 15, height - 18, 30, 18);
    blue.style.background = "#0000ff";
    blue.style.boxShadow = "0 0 8px rgba(0,128,255,0.9)";
    bones.push({ el: blue, blue: true });

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.blue) {
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect) && gravityEnabled) damagePlayer(1);
        } else {
          b.y += b.speed;
          b.el.style.top = b.y + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(1);
        }
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function pacifistTopBottomAlternating(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 6500;
    const startTime = performance.now();

    function spawnRow(fromTop) {
      const count = 5;
      for (let i = 0; i < count; i++) {
        const x = (width / (count + 1)) * (i + 1) - 10;
        const y = fromTop ? -20 : height + 20;
        const bone = spawnBone(x, y, 16, 30);
        bones.push({ el: bone, x, y, speed: fromTop ? 2 : -2 });
      }
    }

    spawnRow(true);
    setTimeout(() => spawnRow(false), 1200);
    setTimeout(() => spawnRow(true), 2400);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.y += b.speed;
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(1);
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= BLUE ATTACK FLOOR =========
  function blueAttackSequence(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const duration = 3500;
    const startTime = performance.now();

    const bone = spawnBone(0, height - 18, width, 18);

    function loop(t) {
      if (!inSoulMode) {
        bone.remove();
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bone.remove();
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      const rect = bone.getBoundingClientRect();
      if (rectsOverlap(rect, soulRect)) damagePlayer(1);

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= SPECIAL ATTACK PHASE 1 =========
  function specialAttackPhase1(onEnd) {
    setSoulColor("red", false);
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 5500;
    const startTime = performance.now();

    const floor = spawnBone(0, height - 18, width, 18);
    bones.push({ el: floor });

    for (let i = 0; i < 5; i++) {
      const y = 20 + i * 16;
      const boneL = spawnBone(-40, y, 40, 10);
      const boneR = spawnBone(width + 40, y + 8, 40, 10);
      bones.push({ el: boneL, x: -40, y, dir: 1, speed: 2.6 });
      bones.push({ el: boneR, x: width + 40, y: y + 8, dir: -1, speed: 2.6 });
    }

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      const floorRect = floor.getBoundingClientRect();
      if (rectsOverlap(floorRect, soulRect)) damagePlayer(1);

      bones.forEach(b => {
        if (b.dir != null) {
          b.x += b.speed * b.dir;
          b.el.style.left = b.x + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= PHASE 2A TARGETING + BLASTERS =========
  function phase2ATargetingBonesAndXYBlasters(count, onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 6500;
    const startTime = performance.now();

    for (let i = 0; i < count; i++) {
      const fromLeft = Math.random() < 0.5;
      const startY = Math.random() * (height - 10);
      const sx = soulX;
      const sy = soulY;
      const x = fromLeft ? -30 : width + 30;
      const bone = spawnBone(x, startY, 20, 10);
      const dirX = sx - x;
      const dirY = sy - startY;
      const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
      bones.push({
        el: bone,
        x,
        y: startY,
        vx: (dirX / len) * 1.8,
        vy: (dirY / len) * 1.8
      });
    }

    setTimeout(() => {
      spawnBlaster(-20, -20, 200, 800, 2, "diagX");
      spawnBlaster(width - 20, -20, 200, 800, 2, "diagX");
    }, 1200);

    setTimeout(() => {
      spawnBlaster(width / 2 - 20, -20, 200, 800, 2, "down");
      spawnBlaster(-20, height / 2 - 20, 400, 700, 2, "horizontal");
      spawnBlaster(width - 20, height / 2 - 20, 400, 700, 2, "horizontal");
    }, 2600);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.x += b.vx;
        b.y += b.vy;
        b.el.style.left = b.x + "px";
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= SPIRAL GASTER FINAL =========
  function spiralGasterFinal(onEnd) {
    setSoulColor("blue", true);
    const boxRect = soulBox.getBoundingClientRect();
    const cx = boxRect.width / 2;
    const cy = boxRect.height / 2;
    soulX = cx - 8;
    soulY = cy - 8;
    soul.style.left = soulX + "px";
    soul.style.top = soulY + "px";

    const duration = 9000;
    const startTime = performance.now();
    const bones = [];

    let angle = 0;
    const spawnInterval = 400;
    let lastSpawn = 0;

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el && b.el.remove());
        setSoulColor("red", false);
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el && b.el.remove());
        setSoulColor("red", false);
        onEnd();
        return;
      }

      const elapsed = t - startTime;
      if (elapsed - lastSpawn > spawnInterval) {
        lastSpawn = elapsed;
        angle += Math.PI / 6;
        const x = cx + Math.cos(angle) * (boxRect.width / 3);
        const y = cy + Math.sin(angle) * (boxRect.height / 3);
        spawnBlaster(x - 20, y - 20, 0, 700, 3, "down");
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(3);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= PHASE 2C ATTACKS =========
  function phase2CBoneFlood(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 5000;
    const startTime = performance.now();

    for (let i = 0; i < 14; i++) {
      const fromTop = i % 2 === 0;
      const x = Math.random() * (width - 20);
      const y = fromTop ? -20 : height + 20;
      const bone = spawnBone(x, y, 20, 20);
      bones.push({ el: bone, x, y, speed: fromTop ? 3 : -3 });
    }

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.y += b.speed;
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function phase2CSlidersCross(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 5000;
    const startTime = performance.now();

    for (let i = 0; i < 4; i++) {
      const fromLeft = i % 2 === 0;
      const y = (height / 5) * (i + 1);
      const bone = spawnBone(fromLeft ? -40 : width + 40, y, 40, 10);
      bones.push({ el: bone, x: fromLeft ? -40 : width + 40, y, dir: fromLeft ? 1 : -1, speed: 3 });
    }

    setTimeout(() => {
      spawnBlaster(-20, height / 2 - 20, 0, 800, 2, "horizontal");
      spawnBlaster(width - 20, height / 2 - 20, 0, 800, 2, "horizontal");
    }, 800);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.x += b.speed * b.dir;
        b.el.style.left = b.x + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function phase2CBoneZoneSpam(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 5500;
    const startTime = performance.now();

    const floor = spawnBone(0, height - 18, width, 18);
    bones.push({ el: floor });

    setTimeout(() => {
      for (let i = 0; i < 4; i++) {
        const x = (width / 5) * (i + 1);
        const bone = spawnBone(x, -20, 16, 40);
        bones.push({ el: bone, x, y: -20, speed: 3 });
      }
    }, 1000);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const soulRect = soul.getBoundingClientRect();
      const floorRect = floor.getBoundingClientRect();
      if (rectsOverlap(floorRect, soulRect)) damagePlayer(2);

      bones.forEach(b => {
        if (b.speed) {
          b.y += b.speed;
          b.el.style.top = b.y + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(2);
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= FIGHT BAR =========
  function startFightBar() {
    phase = "FIGHT_BAR";
    fightBarContainer.style.display = "flex";
    fightPointerPos = 0;
    fightPointerDir = 1;
    fightBarActive = true;
    damageText.textContent = "";
  }

  function stopFightBar() {
    fightBarActive = false;
    fightBarContainer.style.display = "none";

    const barWidth = fightBar.clientWidth;
    const zoneRect = fightZone.getBoundingClientRect();
    const barRect = fightBar.getBoundingClientRect();
    const zoneStart = zoneRect.left - barRect.left;
    const zoneEnd = zoneStart + fightZone.clientWidth;
    const pointerX = fightPointerPos * barWidth;

    let quality = "";
    let hitStrength = 0;
    if (pointerX >= zoneStart && pointerX <= zoneEnd) {
      quality = "GOOD HIT!";
      const center = (zoneStart + zoneEnd) / 2;
      const dist = Math.abs(pointerX - center);
      const maxDist = fightZone.clientWidth / 2;
      hitStrength = 1 - (dist / maxDist);
    } else {
      quality = "MISS...";
      hitStrength = 0;
    }

    hideSansDialogue();
    hidePlayerDialogue();

    if (phase2AActive) {
      damageText.textContent = "9999";
      showPlayerDialogue("* you strike with everything you have.\n* it doesn't change anything.", null, false, true);
    } else if (!sansCanBeHit) {
      damageText.textContent = quality + " (sans dodged)";
      showSansDialogue("heh.\nclose.\n\nbut not that close.", null, true);
    } else {
      const damage = Math.max(1, Math.floor(hitStrength * 10));
      enemyHP -= damage;
      if (enemyHP <= 0) enemyHP = 1;
      updateEnemyHP();
      damageText.textContent = quality + " - " + damage;
      showPlayerDialogue("* you land a hit.\n* something feels... wrong.", null, false, true);

      if (!phase2AActive && canSpare) {
        startPhase2A();
      }
    }

    setTimeout(() => {
      damageText.textContent = "";
      if (playerHP > 0 && phase !== "END") {
        if (phase2AActive) startSansAttackPhase2A();
        else if (phase2CActive) startSansAttackPhase2C();
        else startSansAttack();
      }
    }, 1000);
  }

  function fightBarLoop() {
    if (fightBarActive) {
      const barWidth = fightBar.clientWidth;
      const pointerWidth = fightPointer.clientWidth;
      const max = (barWidth - pointerWidth) / barWidth;
      const speed = 0.02 + pureAttackTurns * 0.003;
      fightPointerPos += fightPointerDir * speed;
      if (fightPointerPos <= 0) {
        fightPointerPos = 0;
        fightPointerDir = 1;
      } else if (fightPointerPos >= max) {
        fightPointerPos = max;
        fightPointerDir = -1;
      }
      const x = fightPointerPos * barWidth;
      fightPointer.style.left = x + "px";
    }
    requestAnimationFrame(fightBarLoop);
  }

  // ========= PHASE 2A START =========
  function startPhase2A() {
    phase2AActive = true;
    phase2ATurns = 0;
    hideSansDialogue();
    hidePlayerDialogue();
    showSansDialogue(
      "your blow lands awkwardly.\n" +
      "i stagger a bit.\n" +
      "heh. guess that one went a little deep.\n" +
      "don't worry.\n" +
      "i've got... 0.0001 hp left.",
      null,
      true
    );
  }

  // ========= SUB MENUS =========
  function openSubMenu(type, options) {
    currentSubType = type;
    currentSubOptions = options;
    subIndex = 0;
    subMenu.innerHTML = "";
    subMenu.style.display = "flex";

    options.forEach((opt, idx) => {
      const span = document.createElement("span");
      span.classList.add("sub-option");
      if (idx === 0) span.classList.add("selected");
      span.textContent = opt.label;
      subMenu.appendChild(span);
    });

    phase = "MENU_SUB";
    dialogueBox.style.display = "block";
  }

  function closeSubMenu() {
    subMenu.style.display = "none";
    subMenu.innerHTML = "";
    currentSubType = null;
    currentSubOptions = [];
    phase = "PLAYER_TURN";
  }

  function setSubIndex(index) {
    if (!currentSubOptions.length) return;
    subIndex = (index + currentSubOptions.length) % currentSubOptions.length;
    const subs = Array.from(subMenu.querySelectorAll(".sub-option"));
    subs.forEach((s, i) => s.classList.toggle("selected", i === subIndex));
  }

  function confirmSubSelection() {
    if (!currentSubOptions.length) return;
    const opt = currentSubOptions[subIndex];
    if (currentSubType === "ACT") handleActOption(opt.id);
    else if (currentSubType === "ITEM") handleItemOption(opt.id);
    else if (currentSubType === "MERCY") handleMercyOption(opt.id);
  }

  function handleActOption(id) {
    actCount++;
    // fully close submenu before showing new text
    closeSubMenu();
    hideSansDialogue();
    hidePlayerDialogue();

    if (id === "CHECK") {
      showPlayerDialogue("* sans - atk 1 def 1.\n* the second skeleton brother stands firm.", null, false, true);
    } else if (id === "JOKE") {
      showPlayerDialogue("* you tell sans a joke.", null, false, false);
      setTimeout(() => {
        showSansDialogue("heh.\nnot bad, kid.", null, true);
      }, 900);
    } else if (id === "FLIRT") {
      showPlayerDialogue("* you try to flirt.", null, false, false);
      setTimeout(() => {
        showSansDialogue("you know i'm not the tall one, right?", null, true);
      }, 900);
    } else if (id === "TALK") {
      showPlayerDialogue("* you talk about paps.", null, false, false);
      setTimeout(() => {
        showSansDialogue("yeah.\nhe's really lookin' forward to you.", null, true);
      }, 900);
    }

    setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1500);
  }

  function handleItemOption(id) {
    const item = items.find(i => i.id === id);
    closeSubMenu();
    hideSansDialogue();
    hidePlayerDialogue();

    if (!item) {
      showPlayerDialogue("* (you fumble with your pockets.)", null, false, false);
    } else {
      playerHP = Math.min(playerMaxHP, playerHP + item.heal);
      updatePlayerHP();
      showPlayerDialogue("* you eat the " + item.name.toLowerCase() + ".\n* you recovered " + item.heal + " hp.", null, false, false);
      items = items.filter(i => i.id !== id);
    }
    setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1500);
  }

  function handleMercyOption(id) {
    closeSubMenu();
    hideSansDialogue();
    hidePlayerDialogue();

    if (id === "SPARE") {
      if (!canSpare || phase2AActive || phase2CActive) {
        showPlayerDialogue("* you reach for MERCY.", null, false, false);
        setTimeout(() => {
          showSansDialogue("sorry, pal.\nno mercy 'till you prove yourself.", null, true);
        }, 900);
        setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1600);
        return;
      }

      showPlayerDialogue("* you lower your hands.", null, false, false);
      setTimeout(() => {
        showSansDialogue(
          "heh.\nnot bad.\n" +
          "how 'bout one last test,\nthen we both move on?",
          null,
          true
        );
      }, 900);
      phase2BQueued = true;

      setTimeout(() => {
        phase = "PHASE2B_SPECIAL";
        enterSoulMode();
        canMoveSoul = true;
        phase2BFinalTest(() => {
          exitSoulMode();
          endBattle(true, true, false);
        });
      }, 1600);
    } else if (id === "FLEE") {
      showPlayerDialogue("* you think about running away...\n* but your feet won't move.", null, false, false);
      setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1500);
    }
  }

  // ========= PHASE 2B FINAL TEST =========
  function phase2BFinalTest(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 8000;
    const startTime = performance.now();

    for (let i = 0; i < 5; i++) {
      const y = 20 + i * 16;
      const boneL = spawnBone(-40, y, 40, 10);
      bones.push({ el: boneL, x: -40, y, dir: 1, speed: 2.2 });
    }

    setTimeout(() => {
      const blue = spawnBone(width / 2 - 10, height - 18, 20, 18);
      blue.style.background = "#0000ff";
      blue.style.boxShadow = "0 0 8px rgba(0,128,255,0.9)";
      bones.push({ el: blue, blue: true });
    }, 1800);

    setTimeout(() => {
      spawnBlaster(width / 2 - 20, -20, 0, 900, 2, "down");
    }, 3500);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.blue) {
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect) && gravityEnabled) damagePlayer(1);
        } else if (b.dir != null) {
          b.x += b.speed * b.dir;
          b.el.style.left = b.x + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= ENDING =========
  function endBattle(playerWon, spared = false, killedSans = false) {
    endScreen.style.display = "flex";
    if (!playerWon) {
      endText.textContent =
        "YOU DIED.\n\n* whoops.\n* guess that was a bit much.\n* sorry, kid.";
    } else if (spared) {
      const gold = 20;
      endText.textContent =
        "YOU WON!\nGained " + gold + "G.\n\n* you feel like you're ready for what's next.";
    } else if (killedSans) {
      const gold = 10;
      const exp = 1;
      endText.textContent =
        "YOU WON.\nEXP: " + exp + "\nG: " + gold + "\n\n* the dust settles.\n* jokes feel a lot heavier now.";
    } else {
      endText.textContent = "YOU WON!";
    }
    phase = "END";
  }

  function confirmMenuSelection() {
    if (phase !== "PLAYER_TURN") return;
    if (sansDialogueBox.style.display === "block") return;

    const action = menuItems[menuIndex];
    hideSansDialogue();

    if (action === "FIGHT") {
      hidePlayerDialogue();
      pureAttackTurns++;
      showPlayerDialogue("* you get ready to attack.", () => {
        startFightBar();
      }, true, false);
    } else if (action === "ACT") {
      openSubMenu("ACT", [
        { id: "CHECK", label: "Check" },
        { id: "JOKE", label: "Joke" },
        { id: "FLIRT", label: "Flirt" },
        { id: "TALK", label: "Talk" }
      ]);
    } else if (action === "ITEM") {
      if (items.length === 0) {
        hidePlayerDialogue();
        showPlayerDialogue("* (you don’t have any items.)", null, false, false);
        setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1500);
      } else {
        openSubMenu("ITEM", items.map(it => ({ id: it.id, label: it.name })));
      }
    } else if (action === "MERCY") {
      openSubMenu("MERCY", [
        { id: "SPARE", label: canSpare && !phase2AActive && !phase2CActive ? "Spare (yellow)" : "Spare" },
        { id: "FLEE", label: "Flee" }
      ]);
    }
  }

  // ========= INPUT =========
  document.addEventListener("keydown", (e) => {
    keys[e.key] = true;
    if (phase === "END") return;

    if (e.key === "z" || e.key === "Z") {
      if (waitingForDialogueAdvance || !sansTextDone || !playerTextDone) {
        handleDialogueZ();
        return;
      }

      if (phase === "PLAYER_TURN") {
        confirmMenuSelection();
      } else if (phase === "MENU_SUB") {
        confirmSubSelection();
      } else if (phase === "FIGHT_BAR") {
        stopFightBar();
      }
    }

    if (e.key === "x" || e.key === "X") {
      if (phase === "MENU_SUB") closeSubMenu();
    }

    if (phase === "PLAYER_TURN") {
      if (sansDialogueBox.style.display === "block") return;
      if (e.key === "a" || e.key === "A") setMenuIndex(menuIndex - 1);
      else if (e.key === "d" || e.key === "D") setMenuIndex(menuIndex + 1);
    } else if (phase === "MENU_SUB") {
      if (e.key === "a" || e.key === "A") setSubIndex(subIndex - 1);
      else if (e.key === "d" || e.key === "D") setSubIndex(subIndex + 1);
    }
  });

  document.addEventListener("keyup", (e) => {
    keys[e.key] = false;
  });

  // ========= INIT =========
  function initIntro() {
    phase = "INTRO_SEQUENCE";
    startIntroSequence();
  }

  updatePlayerHP();
  updateEnemyHP();
  setMenuIndex(0);
  initIntro();
  requestAnimationFrame(soulMovementLoop);
  requestAnimationFrame(fightBarLoop);
</script>
</body>
</html>
