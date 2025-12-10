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

    /* Dialogue box */
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
  let phase = "INTRO"; // INTRO, PLAYER_TURN, ENEMY_ATTACK, FIGHT_BAR, MENU_SUB, PHASE2A, PHASE2A_ATTACK, END

  let turnCount = 0;
  let actCount = 0;
  let pureAttackTurns = 0;
  let sansCanBeHit = false;
  let canSpare = false;
  let usedFinalAttackPhase1 = false;
  let usedBlueIntro = false;

  let phase2AActive = false;
  let phase2ATurns = 0;
  const phase2ATurnTarget = 10; // survive 10 turns

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
  const game = document.getElementById("game");

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

  function showDialogue(text) {
    dialogueBox.style.display = "block";
    dialogueBox.textContent = text;
  }
  function hideDialogue() {
    dialogueBox.style.display = "none";
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
    if (enemyHP <= 0 && !phase2AActive) {
      // If he hits 0 HP in phase 1 and phase2A not started (you forced kill), we treat as kill ending
      endBattle(true, false, true);
    }
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
    // A bit faster if player keeps FIGHTing
    const pureFactor = Math.floor(pureAttackTurns / 2);
    soulSpeed = baseSoulSpeed + pureFactor * 0.5;
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

    // Blue attack joke early in the fight
    if (!usedBlueIntro && turnCount === 1) {
      exitSoulMode();
      showDialogue(
        "* \"i'll let ya have your turn first, kid.\"\n" +
        "* \"...then we get to the fun part.\""
      );
      setTimeout(() => {
        showDialogue(
          "* \"what's that look for?\"\n" +
          "* \"oh right, my blue attack.\"\n" +
          "* \"almost forgot about it, heh heh heh.\""
        );
        setTimeout(() => {
          showDialogue("* \"are ya ready? 'cause here it comes.\"");
          setTimeout(() => {
            blueAttackSequence(() => {
              showDialogue("* \"what are you looking so blue for?\"");
              phase = "PLAYER_TURN";
            });
          }, 600);
        }, 800);
      }, 800);
      usedBlueIntro = true;
      return;
    }

    // Phase 1 final attack BEFORE spare
    if (!usedFinalAttackPhase1 && turnCount >= 5) {
      usedFinalAttackPhase1 = true;
      exitSoulMode();
      showDialogue(
        "* \"alright. warm‑up's over.\"\n" +
        "* \"paps has this 'special attack' thing.\"\n" +
        "* \"guess i’ll borrow the idea.\""
      );
      setTimeout(() => {
        finalAttackPhase1(() => {
          showDialogue(
            "* \"welp, that sure was fun.\"\n" +
            "* \"you seem pretty much ready for waterfall.\"\n" +
            "* \"now just spare me and we can both be on our way.\""
          );
          canSpare = true;
          sansCanBeHit = true;
          phase = "PLAYER_TURN";
        });
      }, 1000);
      return;
    }

    enterSoulMode();

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
        showDialogue("* \"nice moves, kid.\"");
        phase = "PLAYER_TURN";
      }
    });
  }

  // ========= PHASE 2A ATTACKS =========
  function startSansAttackPhase2A() {
    phase = "PHASE2A_ATTACK";
    phase2ATurns++;

    enterSoulMode();

    if (phase2ATurns === phase2ATurnTarget) {
      exitSoulMode();
      showDialogue(
        "* \"heh. still standin', huh?\"\n" +
        "* \"guess that's enough... for both of us.\""
      );
      setTimeout(() => {
        finalAttackPhase2A(() => {
          endBattle(true, false, true); // dust ending
        });
      }, 1000);
      return;
    }

    const attacks = [
      phase2AFastBoneRain,
      phase2ABoneSlidersWithBlaster,
      phase2ABoneCircleAndBlaster
    ];
    const attack = attacks[Math.floor(Math.random() * attacks.length)];
    attack(() => {
      exitSoulMode();
      if (playerHP > 0) {
        showDialogue("* \"still up? not bad.\"");
        phase = "PLAYER_TURN";
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

    // wave from left
    for (let i = 0; i < 5; i++) {
      const bone = spawnBone(-40, 20 + i * 15, 40, 10);
      bones.push({ el: bone, x: -40, y: 20 + i * 15, speed: 2 });
    }

    // blue bone
    setTimeout(() => {
      const blue = spawnBone(width / 2 - 10, height - 20, 20, 20);
      blue.style.background = "#0000ff";
      blue.style.boxShadow = "0 0 8px rgba(0,128,255,0.9)";
      bones.push({ el: blue, blue: true });
    }, 1500);

    // side bones
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

  // ========= BLUE ATTACK SEQUENCE =========
  function blueAttackSequence(onEnd) {
    setSoulColor("blue", true);
    enterSoulMode();
    const boxRect = soulBox.getBoundingClientRect();
    soulX = (boxRect.width / 2) - 8;
    soulY = (boxRect.height / 2) - 8;
    soul.style.left = soulX + "px";
    soul.style.top = soulY + "px";

    const duration = 3500;
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

  // ========= FINAL ATTACK PHASE 1 =========
  function finalAttackPhase1(onEnd) {
    soulBox.style.width = "400px";
    soulBox.style.height = "180px";
    const boxRect = soulBox.getBoundingClientRect();
    enterSoulMode();
    soulX = boxRect.width / 2 - 8;
    soulY = boxRect.height / 2 - 8;

    const duration = 9000;
    const startTime = performance.now();
    const bones = [];

    const floor = spawnBone(0, boxRect.height - 20, boxRect.width, 20);
    bones.push({ el: floor });

    const bigBone = spawnBone(boxRect.width + 160, boxRect.height - 50, 160, 50);
    bones.push({ el: bigBone, x: boxRect.width + 160, y: boxRect.height - 50 });

    const smallBones = [];
    for (let i = 0; i < 6; i++) {
      const b = spawnBone(-20, boxRect.height - 30 - i * 20, 30, 10);
      smallBones.push({ el: b, x: -20, y: boxRect.height - 30 - i * 20 });
      bones.push({ el: b });
    }

    const startTimeSmall = Date.now();

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        soulBox.style.width = "280px";
        soulBox.style.height = "120px";
        onEnd();
        return;
      }
      if (t - startTime > duration) {
        bones.forEach(b => b.el.remove());
        soulBox.style.width = "280px";
        soulBox.style.height = "120px";
        exitSoulMode();
        onEnd();
        return;
      }

      const dt = (Date.now() - startTimeSmall) / 1000;
      smallBones.forEach(s => {
        s.x = Math.min(s.x + 2.2, 60 + Math.sin(dt * 2 + s.y) * 10);
        s.el.style.left = s.x + "px";
      });

      const soulRect = soul.getBoundingClientRect();

      bigBone.x -= 2.2;
      bigBone.el.style.left = bigBone.x + "px";

      const floorRect = floor.getBoundingClientRect();
      if (rectsOverlap(floorRect, soulRect)) damagePlayer(1);

      const bigRect = bigBone.el.getBoundingClientRect();
      if (rectsOverlap(bigRect, soulRect)) damagePlayer(2);

      smallBones.forEach(s => {
        const r = s.el.getBoundingClientRect();
        if (rectsOverlap(r, soulRect)) damagePlayer(1);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= PHASE 2A ATTACKS =========
  function phase2AFastBoneRain(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 7000;
    const startTime = performance.now();

    // Many bones, each moving slower but more of them
    const count = 14;
    for (let i = 0; i < count; i++) {
      const x = Math.random() * (width - 10);
      const bone = spawnBone(x, -20, 10, 24);
      bones.push({ el: bone, x, y: -20, speed: 1.4 + Math.random() * 0.6 });
    }

    // occasional blaster from top
    setTimeout(() => {
      const x = soulX;
      spawnBlaster(x, -20, 0, 800, 2, "down");
    }, 2000);

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

  function phase2ABoneSlidersWithBlaster(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 6500;
    const startTime = performance.now();

    // left-right bone sliders
    for (let i = 0; i < 4; i++) {
      const fromLeft = i % 2 === 0;
      const y = (height / 5) * (i + 1);
      const bone = spawnBone(fromLeft ? -40 : width + 40, y, 40, 10);
      bones.push({ el: bone, x: fromLeft ? -40 : width + 40, y, dir: fromLeft ? 1 : -1, speed: 2.1 });
    }

    // central blaster
    setTimeout(() => {
      spawnBlaster(width / 2 - 20, 0, 300, 900, 2, "down");
    }, 1200);

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

  function phase2ABoneCircleAndBlaster(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const cx = boxRect.width / 2;
    const cy = boxRect.height / 2;
    const bones = [];
    const count = 10;
    const radius = 45;
    const duration = 7000;
    const startTime = performance.now();

    for (let i = 0; i < count; i++) {
      const bone = spawnBone(cx, cy, 12, 12);
      bones.push({ el: bone, angle: (Math.PI * 2 / count) * i });
    }

    // cross blasters
    setTimeout(() => {
      spawnBlaster(-20, cy - 20, 200, 700, 2, "horizontal");
      spawnBlaster(boxRect.width - 20, cy - 20, 200, 700, 2, "horizontal");
    }, 1500);

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
        b.angle += 0.035;
        const x = cx + Math.cos(b.angle) * radius - 6;
        const y = cy + Math.sin(b.angle) * radius - 6;
        b.el.style.left = x + "px";
        b.el.style.top = y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= FINAL ATTACK PHASE 2A (DUST) =========
  function finalAttackPhase2A(onEnd) {
    enterSoulMode();
    setSoulColor("blue", true);
    const boxRect = soulBox.getBoundingClientRect();
    soulX = boxRect.width / 2 - 8;
    soulY = boxRect.height / 2 - 8;

    const totalDuration = 12000;
    const startTime = performance.now();
    const bones = [];

    // bone wave left
    for (let i = 0; i < 5; i++) {
      const y = (boxRect.height / 6) * (i + 1);
      const bone = spawnBone(-40, y, 40, 10);
      bones.push({ el: bone, x: -40, y, speed: 2.2 });
    }

    // then jumps
    setTimeout(() => {
      for (let i = 0; i < 5; i++) {
        const x = (boxRect.width / 6) * (i + 1);
        const bone = spawnBone(x, boxRect.height + 40, 20, 40);
        bones.push({ el: bone, x, y: boxRect.height + 40, phase: Math.random() * Math.PI * 2, jump: true });
      }
    }, 1500);

    // bone zone
    setTimeout(() => {
      const floor = spawnBone(0, boxRect.height - 18, boxRect.width, 18);
      floor.style.background = "#0000ff";
      floor.style.boxShadow = "0 0 10px rgba(0,128,255,0.9)";
      bones.push({ el: floor, zone: true });
    }, 3000);

    // corner blasters
    setTimeout(() => {
      spawnBlaster(-20, -20, 600, 800, 3, "down");
      spawnBlaster(boxRect.width - 20, -20, 600, 800, 3, "down");
    }, 4500);

    // player‑facing blasters
    let shotCount = 0;
    function spawnPlayerBlaster() {
      if (!inSoulMode) return;
      const x = soulX;
      spawnBlaster(x, -20, 0, 500 + shotCount * 150, 3, "down");
      shotCount++;
      if (shotCount < 4) setTimeout(spawnPlayerBlaster, 1200);
    }
    setTimeout(spawnPlayerBlaster, 6500);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        setSoulColor("red", false);
        onEnd();
        return;
      }
      if (t - startTime > totalDuration) {
        bones.forEach(b => b.el.remove());
        setSoulColor("red", false);
        exitSoulMode();
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.zone) {
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect) && gravityEnabled) damagePlayer(2);
        } else if (b.jump) {
          b.phase += 0.11;
          const offset = Math.abs(Math.sin(b.phase)) * 40;
          b.y = (boxRect.height - 40) - offset;
          b.el.style.top = b.y + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(2);
        } else {
          b.x += b.speed;
          b.el.style.left = b.x + "px";
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

    // Phase 2A: 9999 damage but nothing happens yet
    if (phase2AActive) {
      damageText.textContent = "9999";
      showDialogue("* you strike with everything you have.\n* it doesn't change anything.");
    } else if (!sansCanBeHit) {
      damageText.textContent = quality + " (sans dodged)";
      showDialogue("* \"heh. close.\"\n* \"but not that close.\"");
    } else {
      const damage = Math.max(1, Math.floor(hitStrength * 10));
      enemyHP -= damage;
      if (enemyHP <= 0) enemyHP = 1; // he won't die normally here
      updateEnemyHP();
      damageText.textContent = quality + " - " + damage;
      showDialogue("* you land a hit.\n* something feels... off.");
      // This triggers Phase 2A transition:
      if (!phase2AActive && canSpare) {
        startPhase2A();
      }
    }

    setTimeout(() => {
      damageText.textContent = "";
      if (playerHP > 0 && phase !== "END" && !phase2AActive) {
        startSansAttack();
      } else if (playerHP > 0 && phase2AActive) {
        startSansAttackPhase2A();
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

  // ========= PHASE 2A START =========
  function startPhase2A() {
    phase2AActive = true;
    phase2ATurns = 0;
    showDialogue(
      "* \"...heh. there it is.\"\n" +
      "* \"that feeling.\"\n" +
      "* \"guess you're not the only one with determination today.\""
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
    if (id === "SPARE") {
      if (!canSpare || phase2AActive) {
        showDialogue("* you reach for MERCY.\n* \"sorry pal, no mercy 'till you prove yourself.\"");
        closeSubMenu();
        setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1000);
        return;
      }
      showDialogue(
        "* you lower your hands.\n" +
        "* sans smiles faintly.\n" +
        "* \"guess you're ready after all.\"\n" +
        "* \"waterfall's waitin', kid.\""
      );
      closeSubMenu();
      setTimeout(() => endBattle(true, true, false), 2000);
    } else if (id === "FLEE") {
      showDialogue("* you think about running away...\n* but your feet won't move.");
      closeSubMenu();
      setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 900);
    }
  }

  function endBattle(playerWon, spared = false, killedSans = false) {
    endScreen.style.display = "flex";
    if (!playerWon) {
      endText.textContent =
        "YOU DIED.\n\n* \"whoops. guess that was a bit much.\"\n* \"sorry, kid.\"";
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
    const action = menuItems[menuIndex];

    if (action === "FIGHT") {
      pureAttackTurns++;
      showDialogue("* you get ready to attack.");
      setTimeout(() => {
        showDialogue("");
        startFightBar();
      }, 500);
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
      } else if (phase === "PLAYER_TURN") {
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
