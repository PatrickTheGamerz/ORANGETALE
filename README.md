<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>A Test of Skill - Sans Fight (Base)</title>
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

    /* Monster bubble (to the right of Sans) */
    #monster-bubble {
      position: absolute;
      top: 10px;
      left: 110px;
      max-width: 280px;
      padding: 8px 10px;
      border: 2px solid white;
      background: black;
      font-size: 16px;
      white-space: pre-line;
      display: none;
    }
    #monster-bubble.special {
      color: #ff8000; /* special attack color */
      border-color: #ff8000;
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

    /* Dialogue box (player text) */
    #dialogue-box {
      position: absolute;
      bottom: 150px;
      left: 20px;
      right: 20px;
      height: 120px;
      border: 4px solid white;
      padding: 10px;
      box-sizing: border-box;
      font-size: 18px;
      white-space: pre-line;
      background: rgba(0,0,0,0.8);
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
    .bone.blue {
      background: #0000ff;
      box-shadow: 0 0 6px rgba(0,128,255,0.9);
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
      height: 100px;
      display: none;
      justify-content: center;
      align-items: center;
      gap: 20px;
      font-size: 18px;
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

    /* Target selector (who to FIGHT/ACT) */
    #target-menu {
      position: absolute;
      bottom: 230px;
      left: 50%;
      transform: translateX(-50%);
      width: 300px;
      height: 80px;
      border: 4px solid white;
      background: black;
      display: none;
      justify-content: center;
      align-items: center;
      gap: 20px;
      font-size: 18px;
    }
    .target-option {
      padding: 4px 10px;
      border: 2px solid white;
    }
    .target-option.selected {
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
    <div id="enemy-sprite">
      S A N S
      <div id="monster-bubble"></div>
    </div>
    <div id="enemy-hp-container">
      <div id="enemy-hp-bar"></div>
    </div>
    <div id="enemy-hp-text">HP: 1 / 1</div>
  </div>

  <div id="dialogue-box"></div>

  <div id="soul-box">
    <div id="soul"></div>
  </div>

  <div id="target-menu">
    <span class="target-option selected" data-target="sans">sans</span>
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
  let phase = "INTRO";
  // INTRO, PLAYER_TURN, ENEMY_ATTACK, FIGHT_TARGET, FIGHT_BAR, MENU_SUB,
  // PHASE2A, PHASE2A_ATTACK, PHASE2B_SPECIAL, END

  let turnCount = 0;
  let actCount = 0;
  let pureAttackTurns = 0;
  let sansCanBeHit = false;
  let canSpare = false;
  let usedBlueIntro = false;
  let usedPhase1PreSpareSpecial = false;

  // Phase 2A
  let phase2AActive = false;
  let phase2ATurns = 0;
  const phase2ATurnTarget = 8;

  // Phase 2B (post-spare special)
  let phase2BQueued = false;

  // Anti-spam: one action per player turn
  let playerActionUsedThisTurn = false;

  const dialogueBox = document.getElementById("dialogue-box");
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
  const monsterBubble = document.getElementById("monster-bubble");
  const targetMenu = document.getElementById("target-menu");
  const targetOptions = Array.from(document.querySelectorAll(".target-option"));

  let soulX = 0;
  let soulY = 0;
  let soulColor = "red";
  let gravityEnabled = false;
  let gravity = 0.12;
  let yVelocity = 0;
  let jumpHold = 0;
  const maxJumpHold = 15; // few frames of "W" before gravity wins
  let baseSoulSpeed = 3.5;
  let soulSpeed = baseSoulSpeed;

  const keys = {};
  let currentSubType = null;
  let currentSubOptions = [];
  let subIndex = 0;

  let fightPointerPos = 0;
  let fightPointerDir = 1;
  let fightBarActive = false;

  let targetMenuIndex = 0;

  let items = [
    { id: "PIE", name: "Butterscotch Pie", heal: 20 },
    { id: "CREAM", name: "Nice Cream", heal: 12 },
    { id: "BANDAGE", name: "Bandage", heal: 7 }
  ];

  function showDialogue(text) {
    dialogueBox.style.display = "block";
    dialogueBox.textContent = text;
  }
  function hideDialogue() {
    dialogueBox.style.display = "none";
  }

  function showMonsterBubble(text, isSpecial=false) {
    monsterBubble.textContent = text;
    monsterBubble.classList.toggle("special", isSpecial);
    monsterBubble.style.display = "block";
  }
  function hideMonsterBubble() {
    monsterBubble.style.display = "none";
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
    enemyHPText.textContent = "HP: " + enemyHP.toFixed(7) + " / 1";
  }

  function setMenuIndex(index) {
    menuIndex = (index + menuItems.length) % menuItems.length;
    menuElements.forEach((el, i) => {
      el.classList.toggle("selected", i === menuIndex);
    });
  }

  function enterSoulMode() {
    inSoulMode = true;
    canMoveSoul = true;
    soulBox.style.display = "block";
    hideDialogue();
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
    if (!enableGravity) {
      yVelocity = 0;
      jumpHold = 0;
    }
  }

  function soulMovementLoop() {
    if (inSoulMode && canMoveSoul) {
      const boxRect = soulBox.getBoundingClientRect();
      const boxWidth = boxRect.width;
      const boxHeight = boxRect.height;

      // Blue mode: W only gives a short impulse
      if (gravityEnabled) {
        if (keys["w"] || keys["W"]) {
          if (jumpHold < maxJumpHold) {
            yVelocity -= 0.3;
            jumpHold++;
          }
        } else {
          jumpHold = 0;
        }
        yVelocity += gravity;
        soulY += yVelocity;
      } else {
        if (keys["w"] || keys["W"]) soulY -= soulSpeed;
        if (keys["s"] || keys["S"]) soulY += soulSpeed;
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
    soulSpeed = baseSoulSpeed + pureFactor * 0.3;
  }

  // ========= ATTACK DISPATCH =========
  function startSansAttack() {
    if (phase2AActive) {
      startSansAttackPhase2A();
      return;
    }

    phase = "ENEMY_ATTACK";
    turnCount++;
    updateDifficulty();

    // Blue intro
    if (!usedBlueIntro && turnCount === 1) {
      exitSoulMode();
      showDialogue(
        "* \"i'll let ya have your turn first, kid.\"\n" +
        "* \"...then we get to the fun part.\""
      );
      setTimeout(() => {
        showMonsterBubble("oh right, my blue attack.\nalmost forgot about it, heh heh heh.");
        setTimeout(() => {
          showMonsterBubble("are ya ready?\n'cause here it comes.");
          setTimeout(() => {
            blueAttackSequence(() => {
              hideMonsterBubble();
              showDialogue("* \"what are you looking so blue for?\"");
              phase = "PLAYER_TURN";
              playerActionUsedThisTurn = false;
            });
          }, 600);
        }, 800);
      }, 800);
      usedBlueIntro = true;
      return;
    }

    // Phase 1 pre-spare special, then unlock spare + hit
    if (!usedPhase1PreSpareSpecial && turnCount >= 4 && !phase2BQueued) {
      usedPhase1PreSpareSpecial = true;
      exitSoulMode();
      showMonsterBubble("time for a little\nspecial demonstration.", true);
      setTimeout(() => {
        phase1PreSpareSpecial(() => {
          hideMonsterBubble();
          showMonsterBubble("...yeah.\ni think you're ready.");
          showDialogue(
            "* \"welp, that sure was fun.\"\n" +
            "* \"now just spare me and we can both be on our way.\""
          );
          canSpare = true;
          sansCanBeHit = true;
          phase = "PLAYER_TURN";
          playerActionUsedThisTurn = false;
        });
      }, 800);
      return;
    }

    // Normal Phase 1 attacks
    enterSoulMode();

    const attacks = [
      phase1BouncingBlueBones,
      phase1SpinningBlueBones,
      phase1MixedWaves
    ];
    const attack = attacks[Math.floor(Math.random() * attacks.length)];
    attack(() => {
      exitSoulMode();
      if (playerHP > 0) {
        // varied dialogue
        const lines = [
          "* \"nice dodging.\"",
          "* \"still standin', huh?\"",
          "* \"not bad, kid.\""
        ];
        showDialogue(lines[Math.floor(Math.random() * lines.length)]);
        phase = "PLAYER_TURN";
        playerActionUsedThisTurn = false;
      }
    });
  }

  // ========= PHASE 2A ATTACKS =========
  function startSansAttackPhase2A() {
    phase = "PHASE2A_ATTACK";
    phase2ATurns++;

    enterSoulMode();

    // Last turn before final special
    if (phase2ATurns >= phase2ATurnTarget) {
      exitSoulMode();
      showMonsterBubble("heh.\nlooks like we're both at our limit.", true);
      setTimeout(() => {
        spiralGasterFinalPhase2A(() => {
          // at end: tired, can spare or kill
          exitSoulMode();
          canSpare = true;
          sansCanBeHit = true;
          showMonsterBubble("...guess that's it.", true);
          showDialogue("* he looks like he is about to use his special attack.\n* but nothing comes.");
          phase = "PLAYER_TURN";
          playerActionUsedThisTurn = false;
        });
      }, 1000);
      return;
    }

    // normal 2A pattern: target bones bouncing then blaster
    phase2ATargetBonesThenBlaster(() => {
      exitSoulMode();
      if (playerHP > 0) {
        const lines = [
          "* \"still goin'?\"",
          "* \"hangin' in there.\"",
          "* \"this is rough, huh?\""
        ];
        showDialogue(lines[Math.floor(Math.random() * lines.length)]);
        phase = "PLAYER_TURN";
        playerActionUsedThisTurn = false;
      }
    });
  }

  // ========= BLUE ATTACK SEQUENCE =========
  function blueAttackSequence(onEnd) {
    setSoulColor("blue", true);
    enterSoulMode();
    const boxRect = soulBox.getBoundingClientRect();
    soulX = (boxRect.width / 2) - 8;
    soulY = (boxRect.height / 2) - 8;
    soul.style.left = soulX + "px";
    soul.style.top = soulY + "px";

    const duration = 2800;
    const startTime = performance.now();

    const bone = spawnBone(0, boxRect.height - 18, boxRect.width, 18);

    function loop(t) {
      if (!inSoulMode) {
        bone.remove();
        setSoulColor("red", false);
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bone.remove();
        setSoulColor("red", false);
        exitSoulMode();
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

  // ========= HELPER: BONES & BLASTERS =========
  function spawnBone(x, y, w, h, blue=false) {
    const bone = document.createElement("div");
    bone.classList.add("bone");
    if (blue) bone.classList.add("blue");
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
    }, delay + 800); // 0.8 sec delay before firing
  }

  // ========= PHASE 1 ATTACKS (improved) =========

  // 1) Bouncing bones that turn blue after wall collision, then back
  function phase1BouncingBlueBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const w = boxRect.width;
    const h = boxRect.height;
    const bones = [];
    const duration = 6000;
    const startTime = performance.now();

    // many bones from left
    for (let i = 0; i < 6; i++) {
      const y = 20 + i * 15;
      const bone = spawnBone(-40, y, 40, 10);
      bones.push({ el: bone, x: -40, y, vx: 2.2, vy: 0, bounced: false, special: true, blueTime: 0 });
    }

    // 2 extra "bouncer" bones from right
    for (let i = 0; i < 2; i++) {
      const y = 30 + i * 40;
      const bone = spawnBone(w + 40, y, 40, 10);
      bones.push({ el: bone, x: w + 40, y, vx: -2.6, vy: 0, bounceOnly: true });
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
        b.x += b.vx;
        b.el.style.left = b.x + "px";

        // collisions with walls
        if (!b.bounced && b.special) {
          if (b.x <= 0 || b.x + 40 >= w) {
            b.vx = -b.vx;
            b.bounced = true;
            // turn special bones blue for a short time
            b.el.classList.add("blue");
            b.blueTime = 0;
          }
        } else if (b.special && b.bounced) {
          b.blueTime++;
          if (b.blueTime > 60) {
            b.el.classList.remove("blue");
          }
        }

        if (b.bounceOnly && (b.x <= 0 || b.x + 40 >= w)) {
          b.vx = -b.vx;
        }

        const rect = b.el.getBoundingClientRect();
        const isBlue = b.el.classList.contains("blue");
        if (rectsOverlap(rect, soulRect)) {
          if (isBlue && gravityEnabled) damagePlayer(1);
          if (!isBlue) damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // 2) Spinning blue bones around player (move = damage)
  function phase1SpinningBlueBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const cx = boxRect.width / 2;
    const cy = boxRect.height / 2;
    setSoulColor("blue", true);
    soulX = cx - 8;
    soulY = cy - 8;
    soul.style.left = soulX + "px";
    soul.style.top = soulY + "px";

    const bones = [];
    const count = 6;
    const radius = 45;
    let angle = 0;
    const duration = 5000;
    const startTime = performance.now();

    for (let i = 0; i < count; i++) {
      const bone = spawnBone(cx, cy, 14, 14, true);
      bones.push({ el: bone, angleOffset: (Math.PI * 2 / count) * i, spin: 0 });
    }

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        setSoulColor("red", false);
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        setSoulColor("red", false);
        exitSoulMode();
        onEnd();
        return;
      }

      angle += 0.03;
      const soulRect = soul.getBoundingClientRect();
      const moving = keys["w"] || keys["a"] || keys["s"] || keys["d"] ||
                     keys["W"] || keys["A"] || keys["S"] || keys["D"];

      bones.forEach(b => {
        b.spin += 0.2;
        const x = cx + Math.cos(angle + b.angleOffset) * radius;
        const y = cy + Math.sin(angle + b.angleOffset) * radius;
        b.el.style.left = (x - 7) + "px";
        b.el.style.top = (y - 7) + "px";
        b.el.style.transform = "rotate(" + (b.spin * 180 / Math.PI) + "deg)";
        const rect = b.el.getBoundingClientRect();
        if (moving && rectsOverlap(rect, soulRect)) {
          damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // 3) Mixed wave + later target bones (before spare)
  function phase1MixedWaves(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const w = boxRect.width;
    const h = boxRect.height;
    const bones = [];
    const duration = 7000;
    const startTime = performance.now();

    // wave from left and right
    for (let i = 0; i < 4; i++) {
      const y = 20 + i * 18;
      const left = spawnBone(-40, y, 40, 10);
      const right = spawnBone(w + 40, y + 10, 40, 10);
      bones.push({ el: left, x: -40, y, vx: 2 });
      bones.push({ el: right, x: w + 40, y: y + 10, vx: -2 });
    }

    // 4 targeting bones appear later
    setTimeout(() => {
      for (let i = 0; i < 4; i++) {
        const fromLeft = i % 2 === 0;
        const startY = 20 + i * 18;
        const startX = fromLeft ? -20 : w + 20;
        const bone = spawnBone(startX, startY, 20, 10);
        const sx = soulX;
        const sy = soulY;
        const dirX = sx - startX;
        const dirY = sy - startY;
        const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
        bones.push({
          el: bone,
          x: startX,
          y: startY,
          vx: (dirX / len) * 2.2,
          vy: (dirY / len) * 2.2
        });
      }
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
        b.x += b.vx || 0;
        b.y += b.vy || 0;
        b.el.style.left = b.x + "px";
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        const isBlue = b.el.classList.contains("blue");
        if (rectsOverlap(rect, soulRect)) {
          if (isBlue && gravityEnabled) damagePlayer(1);
          if (!isBlue) damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= PHASE 1 PRE-SPARE SPECIAL =========
  function phase1PreSpareSpecial(onEnd) {
    enterSoulMode();
    const boxRect = soulBox.getBoundingClientRect();
    const w = boxRect.width;
    const h = boxRect.height;
    const bones = [];
    const duration = 6000;
    const startTime = performance.now();

    // first wave from both sides
    for (let i = 0; i < 4; i++) {
      const y = 20 + i * 15;
      const left = spawnBone(-40, y, 40, 10);
      const right = spawnBone(w + 40, y + 10, 40, 10);
      bones.push({ el: left, x: -40, y, vx: 2.4 });
      bones.push({ el: right, x: w + 40, y: y + 10, vx: -2.4 });
    }

    // bounce and target bones
    setTimeout(() => {
      bones.forEach(b => {
        if (!b.vy) {
          // bounce off walls
          if (b.x <= 0 || b.x + 40 >= w) b.vx = -b.vx;
        }
      });

      for (let i = 0; i < 4; i++) {
        const fromLeft = i % 2 === 0;
        const startY = 30 + i * 16;
        const startX = fromLeft ? -20 : w + 20;
        const bone = spawnBone(startX, startY, 20, 10);
        const sx = soulX;
        const sy = soulY;
        const dirX = sx - startX;
        const dirY = sy - startY;
        const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
        bones.push({
          el: bone,
          x: startX,
          y: startY,
          vx: (dirX / len) * 2.4,
          vy: (dirY / len) * 2.4
        });
      }
    }, 2400);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        exitSoulMode();
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.x += b.vx || 0;
        b.y += b.vy || 0;
        b.el.style.left = b.x + "px";
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(1);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= PHASE 2A ATTACK: target bones that bounce, then blaster =========
  function phase2ATargetBonesThenBlaster(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const w = boxRect.width;
    const h = boxRect.height;
    const bones = [];
    const duration = 7000;
    const startTime = performance.now();

    const numBones = 6;
    for (let i = 0; i < numBones; i++) {
      const fromLeft = Math.random() < 0.5;
      const sy = Math.random() * (h - 20);
      const sxStart = fromLeft ? -20 : w + 20;
      const bone = spawnBone(sxStart, sy, 20, 10);
      const targetX = soulX;
      const targetY = soulY;
      const dirX = targetX - sxStart;
      const dirY = targetY - sy;
      const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
      bones.push({
        el: bone,
        x: sxStart,
        y: sy,
        vx: (dirX / len) * 2.2,
        vy: (dirY / len) * 1.8,
        bounced: false
      });
    }

    // slower blaster at end
    setTimeout(() => {
      spawnBlaster(w / 2 - 20, -20, 0, 600, 2, "down");
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
        b.x += b.vx;
        b.y += b.vy;
        b.el.style.left = b.x + "px";
        b.el.style.top = b.y + "px";

        if (!b.bounced && (b.x <= 0 || b.x + 20 >= w)) {
          b.vx = -b.vx;
          b.vy = 1.0;
          b.bounced = true;
        }

        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= PHASE 2A FINAL SPIRAL (simplified but works) =========
  function spiralGasterFinalPhase2A(onEnd) {
    enterSoulMode();
    setSoulColor("blue", true);
    const boxRect = soulBox.getBoundingClientRect();
    const cx = boxRect.width / 2;
    const cy = boxRect.height / 2;
    soulX = cx - 8;
    soulY = cy - 8;

    const duration = 9000;
    const startTime = performance.now();
    let angle = 0;
    let lastSpawn = 0;
    const spawnInterval = 450;

    function loop(t) {
      if (!inSoulMode) {
        setSoulColor("red", false);
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        setSoulColor("red", false);
        exitSoulMode();
        onEnd();
        return;
      }

      if (elapsed - lastSpawn > spawnInterval) {
        lastSpawn = elapsed;
        angle += Math.PI / 6;
        const x = cx + Math.cos(angle) * (boxRect.width / 3);
        const y = cy + Math.sin(angle) * (boxRect.height / 3);
        spawnBlaster(x - 20, y - 20, 0, 500, 2, "down");
      }

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= FIGHT BAR & TARGET MENU =========
  function openTargetMenu() {
    targetMenu.style.display = "flex";
    targetMenuIndex = 0;
    targetOptions.forEach((opt, i) => {
      opt.classList.toggle("selected", i === targetMenuIndex);
    });
    phase = "FIGHT_TARGET";
  }

  function closeTargetMenu() {
    targetMenu.style.display = "none";
  }

  function setTargetMenuIndex(i) {
    targetMenuIndex = (i + targetOptions.length) % targetOptions.length;
    targetOptions.forEach((opt, idx) => {
      opt.classList.toggle("selected", idx === targetMenuIndex);
    });
  }

  function confirmTargetSelection() {
    const target = targetOptions[targetMenuIndex].dataset.target;
    if (target === "sans") {
      closeTargetMenu();
      startFightBar();
    }
  }

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

    if (phase2AActive) {
      damageText.textContent = "9999";
      enemyHP = 0;
      updateEnemyHP();
      setTimeout(() => {
        enemyHP = 0.0000128;
        updateEnemyHP();
      }, 100);
      showDialogue("* you strike with everything you have.\n* it doesn't change anything.");
    } else if (!sansCanBeHit) {
      damageText.textContent = quality + " (sans dodged)";
      showDialogue("* \"heh. close.\"\n* \"but not that close.\"");
    } else {
      const damage = Math.max(1, Math.floor(hitStrength * 10));
      enemyHP -= damage;
      if (!phase2AActive && canSpare && enemyHP <= 0) {
        enemyHP = 0.0000128;
      }
      updateEnemyHP();
      damageText.textContent = quality + " - " + damage;
      showDialogue("* you land a hit.\n* something feels... wrong.");
      if (!phase2AActive && canSpare) {
        startPhase2A();
      }
    }

    setTimeout(() => {
      damageText.textContent = "";
      if (playerHP > 0 && phase !== "END") {
        if (phase2AActive) startSansAttackPhase2A();
        else startSansAttack();
      }
    }, 1000);
  }

  function fightBarLoop() {
    if (fightBarActive) {
      const barWidth = fightBar.clientWidth;
      const pointerWidth = fightPointer.clientWidth;
      const max = (barWidth - pointerWidth) / barWidth;
      const speed = 0.013 + pureAttackTurns * 0.002;
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

  function startPhase2A() {
    phase2AActive = true;
    phase2ATurns = 0;
    showMonsterBubble("heh.\nthat one actually hurt.", true);
    showDialogue("* he looks like he is about to use his special attack.");
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
    if (playerActionUsedThisTurn) return;
    playerActionUsedThisTurn = true;
    actCount++;
    if (id === "CHECK") {
      showDialogue("* sans - atk 1 def 1.\n* the second skeleton brother stands firm.");
    } else if (id === "JOKE") {
      showDialogue("* you tell a joke.\n* sans chuckles.\n* \"heh. not bad, kid.\"");
    } else if (id === "FLIRT") {
      showDialogue("* you try to flirt.\n* sans just grins.\n* \"you know i'm not the tall one, right?\"");
    } else if (id === "TALK") {
      showDialogue("* you talk about paps.\n* sans smiles.\n* \"yeah. he’s really lookin' forward to you.\"");
    }
    closeSubMenu();
    setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 900);
  }

  function handleItemOption(id) {
    if (playerActionUsedThisTurn) return;
    playerActionUsedThisTurn = true;
    const item = items.find(i => i.id === id);
    if (!item) {
      showDialogue("* (you fumble with your pockets.)");
    } else {
      playerHP = Math.min(playerMaxHP, playerHP + item.heal);
      updatePlayerHP();
      showDialogue("* you eat the " + item.name.toLowerCase() + ".\n* you recovered " + item.heal + " hp.");
      items = items.filter(i => i.id !== id);
    }
    closeSubMenu();
    setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 900);
  }

  function handleMercyOption(id) {
    if (playerActionUsedThisTurn) return;
    playerActionUsedThisTurn = true;
    if (id === "SPARE") {
      if (!canSpare || phase2AActive) {
        showDialogue("* you reach for MERCY.\n* \"sorry pal, no mercy 'till you prove yourself.\"");
        closeSubMenu();
        setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1000);
        return;
      }
      // Pacifist spare -> Phase 2B special
      showDialogue(
        "* you lower your hands.\n" +
        "* sans smiles.\n" +
        "* \"heh. not bad.\"\n" +
        "* \"how 'bout one last test, then we both move on?\""
      );
      phase2BQueued = true;
      closeSubMenu();
      setTimeout(() => {
        phase = "PHASE2B_SPECIAL";
        enterSoulMode();
        phase2BFinalTest(() => {
          exitSoulMode();
          endBattle(true, true, false);
        });
      }, 1200);
    } else if (id === "FLEE") {
      showDialogue("* you think about running away...\n* but your feet won't move.");
      closeSubMenu();
      setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 900);
    }
  }

  // ========= PHASE 2B FINAL TEST (simplified version) =========
  function phase2BFinalTest(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const w = boxRect.width;
    const h = boxRect.height;
    const bones = [];
    const duration = 6000;
    const startTime = performance.now();

    // first bones both sides
    for (let i = 0; i < 4; i++) {
      const y = 20 + i * 15;
      const left = spawnBone(-40, y, 40, 10);
      const right = spawnBone(w + 40, y + 10, 40, 10);
      bones.push({ el: left, x: -40, y, vx: 2.2 });
      bones.push({ el: right, x: w + 40, y: y + 10, vx: -2.2, canBlue: true, blueTimer: 0 });
    }

    // make right-side bones blue for first 4 seconds
    const blueDurationFrames = 60 * 4 / (1000/60);

    // 3 target bones
    setTimeout(() => {
      for (let i = 0; i < 3; i++) {
        const fromLeft = i % 2 === 0;
        const startY = 30 + i * 20;
        const startX = fromLeft ? -20 : w + 20;
        const bone = spawnBone(startX, startY, 20, 10);
        const sx = soulX;
        const sy = soulY;
        const dirX = sx - startX;
        const dirY = sy - startY;
        const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
        bones.push({
          el: bone,
          x: startX,
          y: startY,
          vx: (dirX / len) * 2.4,
          vy: (dirY / len) * 2.4
        });
      }
    }, 2500);

    // one blaster at end
    setTimeout(() => {
      spawnBlaster(w / 2 - 20, -20, 0, 700, 2, "down");
    }, 4000);

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
        b.x += b.vx || 0;
        b.y += b.vy || 0;
        b.el.style.left = b.x + "px";
        b.el.style.top = b.y + "px";

        if (b.canBlue) {
          if (b.blueTimer === 0) b.el.classList.add("blue");
          b.blueTimer++;
          if (b.blueTimer > blueDurationFrames) {
            b.el.classList.remove("blue");
            b.canBlue = false;
          }
        }

        const rect = b.el.getBoundingClientRect();
        const isBlue = b.el.classList.contains("blue");
        if (rectsOverlap(rect, soulRect)) {
          if (isBlue && gravityEnabled) damagePlayer(1);
          if (!isBlue) damagePlayer(1);
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
        "YOU DIED.\n\n* \"whoops. guess that was a bit much.\"\n* \"sorry, kid.\"";
    } else if (spared) {
      endText.textContent =
        "YOU WON!\nGained 20G.\n\n* you feel like you're ready for what's next.";
    } else if (killedSans) {
      endText.textContent =
        "YOU WON.\nEXP: 120\nG: 10\n\n* your LOVE increased.\n* the silence after the joke hurts more than the fight.";
    } else {
      endText.textContent = "YOU WON!";
    }
    phase = "END";
  }

  function confirmMenuSelection() {
    if (phase !== "PLAYER_TURN" || playerActionUsedThisTurn) return;
    playerActionUsedThisTurn = true;

    const action = menuItems[menuIndex];

    if (action === "FIGHT") {
      pureAttackTurns++;
      showDialogue("* you get ready to attack.");
      setTimeout(() => {
        showDialogue("");
        openTargetMenu();
      }, 300);
    } else if (action === "ACT") {
      openSubMenu("ACT", [
        { id: "CHECK", label: "Check" },
        { id: "JOKE", label: "Joke" },
        { id: "FLIRT", label: "Flirt" },
        { id: "TALK", label: "Talk" }
      ]);
    } else if (action === "ITEM") {
      if (items.length === 0) {
        showDialogue("* (you don’t have any items.)");
        playerActionUsedThisTurn = true;
        setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 900);
      } else {
        openSubMenu("ITEM", items.map(it => ({ id: it.id, label: it.name })));
      }
    } else if (action === "MERCY") {
      const spareLabel = canSpare && !phase2AActive ? "Spare (yellow)" : "Spare";
      openSubMenu("MERCY", [
        { id: "SPARE", label: spareLabel },
        { id: "FLEE", label: "Flee" }
      ]);
    }
  }

  // ========= INPUT =========
  document.addEventListener("keydown", (e) => {
    keys[e.key] = true;
    if (phase === "END") return;

    if (e.key === "z" || e.key === "Z") {
      if (phase === "INTRO") {
        phase = "PLAYER_TURN";
        showDialogue(
          "* \"heya.\"\n" +
          "* \"you've probably wondering what i'm doing here, huh?\"\n" +
          "* \"well, paps is kinda busy at the moment.\"\n" +
          "* \"a little dog stole his 'special attack'.\""
        );
        playerActionUsedThisTurn = false;
      } else if (phase === "PLAYER_TURN") {
        confirmMenuSelection();
      } else if (phase === "MENU_SUB") {
        confirmSubSelection();
      } else if (phase === "FIGHT_BAR") {
        stopFightBar();
      } else if (phase === "FIGHT_TARGET") {
        confirmTargetSelection();
      }
    }

    if (e.key === "x" || e.key === "X") {
      if (phase === "MENU_SUB") closeSubMenu();
      if (phase === "FIGHT_TARGET") {
        closeTargetMenu();
        phase = "PLAYER_TURN";
        playerActionUsedThisTurn = false;
      }
    }

    if (phase === "PLAYER_TURN") {
      if (e.key === "a" || e.key === "A") setMenuIndex(menuIndex - 1);
      else if (e.key === "d" || e.key === "D") setMenuIndex(menuIndex + 1);
    } else if (phase === "MENU_SUB") {
      if (e.key === "a" || e.key === "A") setSubIndex(subIndex - 1);
      else if (e.key === "d" || e.key === "D") setSubIndex(subIndex + 1);
    } else if (phase === "FIGHT_TARGET") {
      if (e.key === "a" || e.key === "A") setTargetMenuIndex(targetMenuIndex - 1);
      else if (e.key === "d" || e.key === "D") setTargetMenuIndex(targetMenuIndex + 1);
    }
  });

  document.addEventListener("keyup", (e) => {
    keys[e.key] = false;
  });

  // ========= INIT =========
  function initIntro() {
    showDialogue("* ...");
    phase = "INTRO";
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
