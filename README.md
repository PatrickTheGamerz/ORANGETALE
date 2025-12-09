<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sans - A Test of Skill</title>
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
      background: black;
      overflow: hidden;
    }

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
    }
    #enemy-hp-container {
      width: 300px;
      height: 16px;
      border: 2px solid white;
      position: relative;
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
    }

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
    }
    #soul {
      width: 16px;
      height: 16px;
      background: red;
      position: absolute;
      border-radius: 3px;
    }

    .bone {
      position: absolute;
      background: white;
    }
    .blaster {
      position: absolute;
      width: 40px;
      height: 40px;
      border: 2px solid white;
      border-radius: 50%;
      box-sizing: border-box;
    }
    .blast-beam {
      position: absolute;
      background: cyan;
      opacity: 0.7;
    }

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
    }
    #damage-text {
      font-size: 16px;
      height: 20px;
    }

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
  let phase = "INTRO"; // INTRO, PLAYER_TURN, ENEMY_ATTACK, FIGHT_BAR, MENU_SUB, END
  let turnCount = 0;
  let actCount = 0;
  let pureAttackTurns = 0;
  let sansCanBeHit = false;

  const isGenocideRoute = Math.random() < 0.5;

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

  let canSpare = false;
  let difficultyLevel = 1;
  let isHugTrapUsed = false;
  let usedBlueIntro = false;
  let usedFinalAttackPacifist = false;
  let usedFinalAttackGenocide = false;

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
    if (enemyHP <= 0) endBattle(true, false, true);
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
    difficultyLevel = 1 + Math.floor(pureAttackTurns / 2);
    if (difficultyLevel > 4) difficultyLevel = 4;
    soulSpeed = baseSoulSpeed + (difficultyLevel - 1) * 0.5;
  }

  // ========= ATTACK DISPATCH =========
  function startSansAttack() {
    phase = "ENEMY_ATTACK";
    turnCount++;
    updateDifficulty();
    enterSoulMode();

    // Pacifist: unlock spare after enough time/ACT
    if (!isGenocideRoute && (turnCount >= 7 || actCount >= 4)) canSpare = true;

    // Pacifist: blue attack joke
    if (!isGenocideRoute && !usedBlueIntro && turnCount === 2) {
      exitSoulMode();
      showDialogue(
        "* \"what's that look for?\"\n" +
        "* \"oh right, my blue attack.\"\n" +
        "* \"almost forgot about it, heh heh heh.\"\n" +
        "* \"are ya ready? 'cause here it comes.\""
      );
      usedBlueIntro = true;
      setTimeout(() => {
        blueAttackSequence(() => {
          showDialogue("* \"what are you looking so blue for?\"");
          phase = "PLAYER_TURN";
        });
      }, 800);
      return;
    }

    // Pacifist final Papyrus-style attack
    if (!isGenocideRoute && !usedFinalAttackPacifist && turnCount >= 8) {
      usedFinalAttackPacifist = true;
      exitSoulMode();
      showDialogue("* \"welp, time for the big one.\"\n* \"paps would be proud of this one.\"");
      setTimeout(() => finalPapyrusAttack(() => {
        showDialogue(
          "* \"welp, that sure was fun.\"\n" +
          "* \"you seem pretty much ready for waterfall.\"\n" +
          "* \"now just spare me and we can both be on our way.\""
        );
        canSpare = true;
        phase = "PLAYER_TURN";
      }), 1200);
      return;
    }

    // Genocide: final special attack
    if (isGenocideRoute && !usedFinalAttackGenocide && turnCount >= 8) {
      usedFinalAttackGenocide = true;
      exitSoulMode();
      showDialogue(
        "* \"and you know what?\"\n" +
        "* \"this whole ordeal was caused by you.\"\n" +
        "* \"so get ready...\"\n" +
        "* \"cuz i'm about to use my special attack.\""
      );
      setTimeout(() => finalGenocideAttack(() => {
        exitSoulMode();
        showDialogue("* sans looks tired.\n* \"...so. what's it gonna be, kid?\"");
        canSpare = true;
        sansCanBeHit = true;
        phase = "PLAYER_TURN";
      }), 1200);
      return;
    }

    // Normal attacks
    if (!isGenocideRoute) {
      const attacks = [
        pacifistBottomZoneJump,
        pacifistDoubleSpinBones,
        pacifistFacingBones,
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
    } else {
      const attacks = [
        genoBoneJumpsWithSafeSpot,
        genoBoneSliders,
        genoBoneJumpsPlusBlueZone,
        genoMixedBlastersBones,
        genoRandomZone
      ];
      const attack = attacks[Math.floor(Math.random() * attacks.length)];
      attack(() => {
        exitSoulMode();
        if (playerHP > 0) {
          showDialogue("* \"still standin'.\"\n* \"guess we keep goin'.\"");
          phase = "PLAYER_TURN";
        }
      });
    }
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

  // ========= PACIFIST ATTACKS =========

  // 1) Bottom bone zone, Sans "jumps" (player dodges around)
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
      gap.style.background = "black";
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

  // 2) Two rotating bones around center
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

  // 3) Three bones facing where the player was (approx)
  function pacifistFacingBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const shots = 3;
    const startTime = performance.now();
    const duration = 5000;
    const bones = [];

    function fireShot() {
      const sx = soulX;
      const sy = soulY;
      const fromLeft = Math.random() < 0.5;
      const x = fromLeft ? -20 : width + 20;
      const bone = spawnBone(x, sy, 20, 10);
      const dir = fromLeft ? 1 : -1;
      bones.push({ el: bone, x, y: sy, dir, speed: 2.6 });
    }

    for (let i = 0; i < shots; i++) {
      setTimeout(fireShot, i * 900);
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
        b.x += b.speed * b.dir;
        b.el.style.left = b.x + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(1);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // 4) Bone wave, then blue bone, then side bones
  function pacifistWaveBlueThenSideBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const bones = [];
    const duration = 7000;
    const startTime = performance.now();

    for (let i = 0; i < 5; i++) {
      const y = (height / 6) * (i + 1);
      const bone = spawnBone(-40, y, 40, 10);
      bones.push({ el: bone, x: -40, y, speed: 2 });
    }

    setTimeout(() => {
      const blue = spawnBone(width / 2 - 10, height - 20, 20, 20);
      blue.style.background = "#0000ff";
      bones.push({ el: blue, x: width / 2 - 10, y: height - 20, speed: 0, blue: true });
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
          if (rectsOverlap(rect, soulRect)) {
            if (gravityEnabled) damagePlayer(1);
          }
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

  // 5) Bone waves from top, blue bone at bottom
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

  // 6) Top then bottom alternating lines
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

  // ========= PAPYRUS-STYLE FINAL ATTACK (PACIFIST) =========
  function finalPapyrusAttack(onEnd) {
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
        s.x = Math.min(s.x + 2.2, 40 + Math.sin(dt * 2 + s.y) * 10);
        s.el.style.left = s.x + "px";
      });

      const boxRect2 = soulBox.getBoundingClientRect();
      const soulRect = soul.getBoundingClientRect();

      bigBone.x -= 2.5;
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

  // ========= GENOCIDE ATTACKS =========
  function genoBoneJumpsWithSafeSpot(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const count = 7;
    const bones = [];
    const duration = 6500;
    const startTime = performance.now();

    const safeIndex = Math.floor(Math.random() * count);

    for (let i = 0; i < count; i++) {
      if (i === safeIndex) continue;
      const x = (width / (count + 1)) * (i + 1) - 10;
      const bone = spawnBone(x, height + 40, 20, 40);
      bones.push({ el: bone, x, y: height + 40, phase: Math.random() * Math.PI * 2 });
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
        b.phase += 0.08;
        const offset = Math.abs(Math.sin(b.phase)) * 40;
        b.y = (height - 40) - offset;
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function genoBoneSliders(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const bones = [];
    const duration = 6000;
    const startTime = performance.now();

    for (let i = 0; i < 4; i++) {
      const fromLeft = i % 2 === 0;
      const y = (height / 5) * (i + 1);
      const bone = spawnBone(fromLeft ? -40 : width + 40, y, 40, 10);
      bones.push({ el: bone, x: fromLeft ? -40 : width + 40, y, dir: fromLeft ? 1 : -1, speed: 3 });
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
        b.x += b.speed * b.dir;
        b.el.style.left = b.x + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function genoBoneJumpsPlusBlueZone(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const bones = [];
    const duration = 7000;
    const startTime = performance.now();

    const blueZone = spawnBone(0, height - 20, width, 20);
    blueZone.style.background = "#0000ff";
    bones.push({ el: blueZone, blue: true });

    for (let i = 0; i < 5; i++) {
      const x = (width / 6) * (i + 1);
      const bone = spawnBone(x, height + 40, 20, 40);
      bones.push({ el: bone, x, y: height + 40, phase: Math.random() * Math.PI * 2 });
    }

    setSoulColor("blue", true);

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

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.blue) {
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect) && gravityEnabled) damagePlayer(2);
        } else {
          b.phase += 0.09;
          const offset = Math.abs(Math.sin(b.phase)) * 40;
          b.y = (height - 40) - offset;
          b.el.style.top = b.y + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(2);
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function genoMixedBlastersBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 7000;
    const startTime = performance.now();

    for (let i = 0; i < 4; i++) {
      const x = (width / 5) * (i + 1);
      const bone = spawnBone(x, -20, 10, 30);
      bones.push({ el: bone, x, y: -20, speed: 2.2 });
    }

    setTimeout(() => {
      const y = height / 2 - 20;
      spawnBlaster(width / 2 - 20, y, 0, 1000, 2, "horizontal");
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
        b.y += b.speed;
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function genoRandomZone(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 6500;
    const startTime = performance.now();

    for (let i = 0; i < 8; i++) {
      const fromTop = Math.random() < 0.5;
      const y = fromTop ? -20 : height + 20;
      const x = Math.random() * (width - 20);
      const bone = spawnBone(x, y, 20, 20);
      bones.push({ el: bone, x, y, speed: fromTop ? 2.4 : -2.4 });
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

  // ========= GENOCIDE SPECIAL ATTACK =========
  function finalGenocideAttack(onEnd) {
    enterSoulMode();
    setSoulColor("blue", true);
    const boxRect = soulBox.getBoundingClientRect();
    soulX = boxRect.width / 2 - 8;
    soulY = boxRect.height / 2 - 8;

    const totalDuration = 14000;
    const startTime = performance.now();
    const bones = [];

    // Bone wave from left
    for (let i = 0; i < 5; i++) {
      const y = (boxRect.height / 6) * (i + 1);
      const bone = spawnBone(-40, y, 40, 10);
      bones.push({ el: bone, x: -40, y, speed: 3 });
    }

    // After that, bone jumps
    setTimeout(() => {
      for (let i = 0; i < 5; i++) {
        const x = (boxRect.width / 6) * (i + 1);
        const bone = spawnBone(x, boxRect.height + 40, 20, 40);
        bones.push({ el: bone, x, y: boxRect.height + 40, phase: Math.random() * Math.PI * 2, jump: true });
      }
    }, 1500);

    // Bone zone
    setTimeout(() => {
      const floor = spawnBone(0, boxRect.height - 18, boxRect.width, 18);
      bones.push({ el: floor, zone: true });
    }, 3000);

    // Corner blasters
    setTimeout(() => {
      spawnBlaster(-20, -20, 600, 900, 3, "down");
      spawnBlaster(boxRect.width - 20, -20, 600, 900, 3, "down");
    }, 4500);

    // Player-facing blasters
    let shotCount = 0;
    function spawnPlayerBlaster() {
      if (!inSoulMode) return;
      const x = soulX;
      const y = 0;
      spawnBlaster(x, y, 0, 600 + shotCount * 200, 3, "down");
      shotCount++;
      if (shotCount < 4) setTimeout(spawnPlayerBlaster, 1300);
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
          b.phase += 0.12;
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

    if (!sansCanBeHit) {
      damageText.textContent = quality + " (sans dodged)";
      showDialogue("* \"i'll let ya have your turn first, kid.\"\n* \"...still not hittin' me, though.\"");
    } else {
      const damage = Math.max(1, Math.floor(hitStrength * 10));
      enemyHP -= damage;
      updateEnemyHP();
      damageText.textContent = quality + " -" + damage;
      showDialogue("* you actually hit sans.\n* \"guess i let my guard down.\"");
    }

    setTimeout(() => {
      damageText.textContent = "";
      if (enemyHP > 0 && playerHP > 0) startSansAttack();
    }, 1000);
  }

  function fightBarLoop() {
    if (fightBarActive) {
      const barWidth = fightBar.clientWidth;
      const pointerWidth = fightPointer.clientWidth;
      const max = (barWidth - pointerWidth) / barWidth;
      const speed = 0.013 + (difficultyLevel - 1) * 0.004;
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
    subs.forEach((s, i) => {
      s.classList.toggle("selected", i === subIndex);
    });
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
    if (!isGenocideRoute) {
      if (id === "CHECK") {
        showDialogue("* sans - atk 1 def 1.\n* the second skeleton brother stands firm.");
      } else if (id === "JOKE") {
        showDialogue("* you tell a joke.\n* sans chuckles.\n* \"heh. not bad, kid.\"");
      } else if (id === "FLIRT") {
        showDialogue("* you try to flirt.\n* sans just grins.\n* \"you know i'm not the tall one, right?\"");
      } else if (id === "TALK") {
        showDialogue("* you talk about paps.\n* sans smiles.\n* \"yeah. he’s really lookin' forward to you.\"");
      }
    } else {
      if (id === "CHECK") {
        showDialogue("* sans - atk 1 def 1.\n* he looks tired.\n* and disappointed.");
      } else if (id === "JOKE") {
        showDialogue("* you tell a joke.\n* sans doesn’t laugh.\n* \"...yeah. not really in the mood.\"");
      } else if (id === "FLIRT") {
        showDialogue("* you try to flirt.\n* sans stares.\n* \"after all this? really?\"");
      } else if (id === "TALK") {
        showDialogue("* you try to talk.\n* \"we already know how this story goes, kid.\"");
      }
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
      if (!isGenocideRoute && !canSpare) {
        showDialogue("* you reach for MERCY.\n* a wall of bones blocks your hand.\n* \"sorry pal, no mercy 'till you prove yourself.\"");
        closeSubMenu();
        setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1000);
        return;
      }
      if (isGenocideRoute && !canSpare) {
        showDialogue("* you try to spare sans.\n* he doesn’t move.\n* \"nah, kid. not this time.\"");
        closeSubMenu();
        setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1000);
        return;
      }
      // can spare
      showDialogue("* you lower your hands.\n* sans smiles faintly.\n* \"guess we’re done here, huh?\"");
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
      endText.textContent = isGenocideRoute
        ? "YOU DIED.\n\n* \"guess that's it for now.\"\n* \"see ya next reset, kid.\""
        : "YOU DIED.\n\n* \"whoops. guess i pushed a bit too hard.\"\n* \"sorry, kid.\"";
    } else if (spared) {
      const gold = isGenocideRoute ? 0 : 20;
      endText.textContent = "YOU WON!\nGained " + gold + "G.\n\n* you feel like you understand him a little better.";
    } else if (killedSans) {
      const gold = isGenocideRoute ? 10 : 10;
      const exp = isGenocideRoute ? 50 : 1;
      endText.textContent = "YOU WON.\nEXP: " + exp + "\nG: " + gold + "\n\n* the silence after the joke hurts more than the fight.";
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
      if (!isGenocideRoute && pureAttackTurns >= 4) sansCanBeHit = true;
      if (isGenocideRoute && pureAttackTurns >= 2) sansCanBeHit = true;
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
      const spareLabel = canSpare ? "Spare (yellow)" : "Spare";
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
        if (!isGenocideRoute) {
          showDialogue(
            "* \"heya.\"\n" +
            "* \"you've probably wondering what i'm doing here, huh?\"\n" +
            "* \"well, paps is kinda busy at the moment.\"\n" +
            "* \"a little dog stole his 'special attack'.\""
          );
        } else {
          showDialogue(
            "* \"human.\"\n" +
            "* \"turn around and face me.\"\n\n" +
            "* \"you're probably wondering what i'm doin' here so early in the morning.\"\n" +
            "* \"well... it's a long story.\"\n" +
            "* \"but one i'm sure we both know.\"\n" +
            "* \"what you've already done...\"\n" +
            "* \"and what i have to do.\"\n" +
            "* \"you're lucky i never remembered...\"\n" +
            "* \"cause then... i'd have broken my promise a long time ago.\""
          );
        }
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
