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
    }

    .bullet {
      width: 10px;
      height: 10px;
      background: white;
      position: absolute;
      border-radius: 2px;
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
    <div id="enemy-sprite">
      S A N S
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
  // --- Core state ---
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
  let pureAttackTurns = 0; // number of turns where player only used FIGHT
  let sansCanBeHit = false;

  // genocide memory (50% per page load)
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

  let difficultyLevel = 1; // increases when player keeps attacking
  let isHugTrapUsed = false; // genocide special "hug" trap

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
    if (playerHP <= 0) {
      endBattle(false);
    }
  }

  function updateEnemyHP() {
    if (enemyHP < 0) enemyHP = 0;
    const ratio = enemyHP / enemyMaxHP;
    enemyHPBar.style.width = (ratio * 100) + "%";
    enemyHPText.textContent = "HP: " + enemyHP + " / " + enemyMaxHP;
    if (enemyHP <= 0) {
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
  }

  function clearBullets() {
    const bullets = soulBox.querySelectorAll(".bullet, .bone, .blaster, .blast-beam");
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

  function soulMovementLoop() {
    if (inSoulMode && canMoveSoul) {
      const boxRect = soulBox.getBoundingClientRect();
      const boxWidth = boxRect.width;
      const boxHeight = boxRect.height;

      if (keys["w"] || keys["W"]) soulY -= soulSpeed;
      if (keys["s"] || keys["S"]) soulY += soulSpeed;
      if (keys["a"] || keys["A"]) soulX -= soulSpeed;
      if (keys["d"] || keys["D"]) soulX += soulSpeed;

      if (soulX < 0) soulX = 0;
      if (soulY < 0) soulY = 0;
      if (soulX > boxWidth - 16) soulX = boxWidth - 16;
      if (soulY > boxHeight - 16) soulY = boxHeight - 16;

      soul.style.left = soulX + "px";
      soul.style.top = soulY + "px";
    }
    requestAnimationFrame(soulMovementLoop);
  }

  function damagePlayer(amount) {
    playerHP -= amount;
    updatePlayerHP();
  }

  // --- Difficulty scaling ---
  function updateDifficulty() {
    difficultyLevel = 1 + Math.floor(pureAttackTurns / 2);
    if (difficultyLevel > 4) difficultyLevel = 4;
    soulSpeed = baseSoulSpeed + (difficultyLevel - 1) * 0.5;
  }

  // --- Attack dispatcher ---
  function startSansAttack() {
    phase = "ENEMY_ATTACK";
    turnCount++;

    if (!isGenocideRoute && (turnCount >= 7 || actCount >= 4)) {
      canSpare = true;
    }

    updateDifficulty();
    enterSoulMode();

    if (isGenocideRoute) {
      if (!isHugTrapUsed && turnCount === 3) {
        hugTrapAttack(() => {
          exitSoulMode();
          if (playerHP > 0) {
            showDialogue("* \"here, kid. how 'bout a hug?\"\n* something feels very wrong.");
            phase = "PLAYER_TURN";
          }
        });
        isHugTrapUsed = true;
        return;
      }

      const attacks = [
        blasterSweepAttack,
        strongBoneRain,
        fastBoneTunnel,
        boneCircleFast,
        mixedBonesAndBlaster
      ];
      const attack = attacks[Math.floor(Math.random() * attacks.length)];
      attack(() => {
        exitSoulMode();
        if (playerHP > 0) {
          showDialogue("* still standing, huh?");
          phase = "PLAYER_TURN";
        }
      });
    } else {
      const attacks = [
        gentleBoneSweep,
        softBoneRain,
        boneHopPattern,
        boneTunnel,
        boneCircle
      ];
      const attack = attacks[Math.floor(Math.random() * attacks.length)];
      attack(() => {
        exitSoulMode();
        if (playerHP > 0) {
          showDialogue("* heh.\n* not bad, kid.");
          phase = "PLAYER_TURN";
        }
      });
    }
  }

  // --- Pacifist attacks ---

  function gentleBoneSweep(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const height = boxRect.height;

    const speed = 2 + (difficultyLevel - 1) * 0.5;
    const duration = 5000;

    const waves = 1 + difficultyLevel;
    const bones = [];

    for (let i = 0; i < waves; i++) {
      const bone = document.createElement("div");
      bone.classList.add("bone");
      bone.style.width = "40px";
      bone.style.height = "10px";
      const y = (height / (waves + 1)) * (i + 1);
      bone.style.top = y + "px";
      bone.style.left = "-40px";
      soulBox.appendChild(bone);
      bones.push({ el: bone, x: -40 });
    }

    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.x += speed;
        b.el.style.left = b.x + "px";
        const boneRect = b.el.getBoundingClientRect();
        if (rectsOverlap(boneRect, soulRect)) {
          damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  function softBoneRain(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;

    const count = 6 + difficultyLevel * 2;
    const bones = [];
    const duration = 5500;

    for (let i = 0; i < count; i++) {
      const bone = document.createElement("div");
      bone.classList.add("bone");
      bone.style.width = "10px";
      bone.style.height = "30px";
      const x = Math.random() * (width - 10);
      bone.style.left = x + "px";
      bone.style.top = "-30px";
      soulBox.appendChild(bone);
      bones.push({ el: bone, y: -30, speed: 1.5 + Math.random() * 1 + (difficultyLevel - 1) * 0.5 });
    }

    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.y += b.speed;
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) {
          damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  function boneHopPattern(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const bones = [];
    const columns = 4 + difficultyLevel;
    for (let i = 0; i < columns; i++) {
      const bone = document.createElement("div");
      bone.classList.add("bone");
      bone.style.width = "20px";
      bone.style.height = "40px";
      const spacing = width / (columns + 1);
      const x = spacing * (i + 1) - 10;
      bone.style.left = x + "px";
      bone.style.top = height + "px";
      soulBox.appendChild(bone);
      bones.push({ el: bone, x, y: height, phase: Math.random() * Math.PI * 2 });
    }

    const duration = 6000;
    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.phase += 0.05 + (difficultyLevel - 1) * 0.01;
        const offset = Math.sin(b.phase) * (30 + difficultyLevel * 10);
        b.y = (height - 40) - Math.abs(offset);
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) {
          damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  function boneTunnel(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const gapHeight = 40;
    const centerY = height / 2;

    const topBone = document.createElement("div");
    topBone.classList.add("bone");
    topBone.style.left = width + "px";
    topBone.style.top = "0px";
    topBone.style.width = "40px";
    topBone.style.height = (centerY - gapHeight / 2) + "px";

    const bottomBone = document.createElement("div");
    bottomBone.classList.add("bone");
    bottomBone.style.left = width + "px";
    bottomBone.style.top = (centerY + gapHeight / 2) + "px";
    bottomBone.style.width = "40px";
    bottomBone.style.height = (height - (centerY + gapHeight / 2)) + "px";

    soulBox.appendChild(topBone);
    soulBox.appendChild(bottomBone);

    let x = width;
    const speed = 2 + (difficultyLevel - 1) * 0.7;
    const duration = 5500;
    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        topBone.remove();
        bottomBone.remove();
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        topBone.remove();
        bottomBone.remove();
        onEnd();
        return;
      }

      x -= speed;
      topBone.style.left = x + "px";
      bottomBone.style.left = x + "px";

      const soulRect = soul.getBoundingClientRect();
      const topRect = topBone.getBoundingClientRect();
      const bottomRect = bottomBone.getBoundingClientRect();
      if (rectsOverlap(soulRect, topRect) || rectsOverlap(soulRect, bottomRect)) {
        damagePlayer(1);
      }

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  function boneCircle(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const cx = boxRect.width / 2;
    const cy = boxRect.height / 2;
    const radius = 40 + difficultyLevel * 5;
    const bones = [];
    const count = 8 + difficultyLevel * 2;

    for (let i = 0; i < count; i++) {
      const bone = document.createElement("div");
      bone.classList.add("bone");
      bone.style.width = "14px";
      bone.style.height = "14px";
      soulBox.appendChild(bone);
      bones.push({ el: bone, angle: (Math.PI * 2 / count) * i });
    }

    const duration = 6000;
    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();

      bones.forEach(b => {
        b.angle += 0.02 + (difficultyLevel - 1) * 0.01;
        const x = cx + Math.cos(b.angle) * radius - 7;
        const y = cy + Math.sin(b.angle) * radius - 7;
        b.el.style.left = x + "px";
        b.el.style.top = y + "px";

        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) {
          damagePlayer(1);
        }
      });

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  // --- Genocide attacks (blasters etc.) ---

  function spawnBlaster(x, y, angle, delay, beamDuration, damage, speedMult) {
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
      const beamWidth = boxRect.width;
      const beamHeight = 16;

      beam.style.width = beamWidth + "px";
      beam.style.height = beamHeight + "px";
      beam.style.left = "0px";
      beam.style.top = (y + 12) + "px";

      const startTime = performance.now();

      function loop(t) {
        if (!inSoulMode) {
          beam.remove();
          blaster.remove();
          return;
        }
        const elapsed = t - startTime;
        if (elapsed > beamDuration) {
          beam.remove();
          blaster.remove();
          return;
        }
        const soulRect = soul.getBoundingClientRect();
        const rect = beam.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) {
          damagePlayer(damage);
        }
        requestAnimationFrame(loop);
      }
      requestAnimationFrame(loop);
    }, delay);
  }

  function blasterSweepAttack(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const height = boxRect.height;

    const beams = 3 + difficultyLevel;
    const delayStep = 500 - difficultyLevel * 50;
    const beamDuration = 900;
    const damage = 2;

    for (let i = 0; i < beams; i++) {
      const y = (height / (beams + 1)) * (i + 1);
      spawnBlaster(-20, y - 20, 0, i * delayStep, beamDuration, damage, 1);
    }

    const totalDuration = beams * delayStep + beamDuration + 500;
    setTimeout(() => {
      onEnd();
    }, totalDuration);
  }

  function strongBoneRain(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;

    const count = 10 + difficultyLevel * 4;
    const bones = [];
    const duration = 5500;

    for (let i = 0; i < count; i++) {
      const bone = document.createElement("div");
      bone.classList.add("bone");
      bone.style.width = "12px";
      bone.style.height = "34px";
      const x = Math.random() * (width - 12);
      bone.style.left = x + "px";
      bone.style.top = "-34px";
      soulBox.appendChild(bone);
      bones.push({ el: bone, y: -34, speed: 2 + Math.random() * 1.5 + (difficultyLevel - 1) * 0.5 });
    }

    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.y += b.speed;
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) {
          damagePlayer(2);
        }
      });

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  function fastBoneTunnel(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const gapHeight = 34;
    const centerY = height / 2;

    const topBone = document.createElement("div");
    topBone.classList.add("bone");
    topBone.style.left = width + "px";
    topBone.style.top = "0px";
    topBone.style.width = "50px";
    topBone.style.height = (centerY - gapHeight / 2) + "px";

    const bottomBone = document.createElement("div");
    bottomBone.classList.add("bone");
    bottomBone.style.left = width + "px";
    bottomBone.style.top = (centerY + gapHeight / 2) + "px";
    bottomBone.style.width = "50px";
    bottomBone.style.height = (height - (centerY + gapHeight / 2)) + "px";

    soulBox.appendChild(topBone);
    soulBox.appendChild(bottomBone);

    let x = width;
    const speed = 3 + difficultyLevel;
    const duration = 4500;
    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        topBone.remove();
        bottomBone.remove();
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        topBone.remove();
        bottomBone.remove();
        onEnd();
        return;
      }

      x -= speed;
      topBone.style.left = x + "px";
      bottomBone.style.left = x + "px";

      const soulRect = soul.getBoundingClientRect();
      const topRect = topBone.getBoundingClientRect();
      const bottomRect = bottomBone.getBoundingClientRect();
      if (rectsOverlap(soulRect, topRect) || rectsOverlap(soulRect, bottomRect)) {
        damagePlayer(2);
      }

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  function boneCircleFast(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const cx = boxRect.width / 2;
    const cy = boxRect.height / 2;
    const radius = 50 + difficultyLevel * 10;
    const bones = [];
    const count = 10 + difficultyLevel * 3;

    for (let i = 0; i < count; i++) {
      const bone = document.createElement("div");
      bone.classList.add("bone");
      bone.style.width = "14px";
      bone.style.height = "14px";
      soulBox.appendChild(bone);
      bones.push({ el: bone, angle: (Math.PI * 2 / count) * i });
    }

    const duration = 6000;
    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();

      bones.forEach(b => {
        b.angle += 0.04 + (difficultyLevel - 1) * 0.02;
        const x = cx + Math.cos(b.angle) * radius - 7;
        const y = cy + Math.sin(b.angle) * radius - 7;
        b.el.style.left = x + "px";
        b.el.style.top = y + "px";

        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) {
          damagePlayer(2);
        }
      });

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  function mixedBonesAndBlaster(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const bones = [];
    const count = 6 + difficultyLevel * 2;

    for (let i = 0; i < count; i++) {
      const bone = document.createElement("div");
      bone.classList.add("bone");
      bone.style.width = "10px";
      bone.style.height = "30px";
      const x = Math.random() * (width - 10);
      bone.style.left = x + "px";
      bone.style.top = "-30px";
      soulBox.appendChild(bone);
      bones.push({ el: bone, y: -30, speed: 2 + Math.random() * 1.5 });
    }

    const centerY = height / 2;
    spawnBlaster(-20, centerY - 20, 0, 800, 900, 2, 1);

    const duration = 6000;
    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        b.y += b.speed;
        b.el.style.top = b.y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) {
          damagePlayer(2);
        }
      });

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  // Hug trap: "here, give me a hug pal" -> bone zone
  function hugTrapAttack(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    const duration = 6500;
    const startTime = performance.now();
    const bones = [];

    function spawnRing() {
      const ringCount = 12;
      const radius = 20 + Math.random() * 40;

      for (let i = 0; i < ringCount; i++) {
        const bone = document.createElement("div");
        bone.classList.add("bone");
        bone.style.width = "12px";
        bone.style.height = "12px";
        const angle = (Math.PI * 2 / ringCount) * i;
        const x = width / 2 + Math.cos(angle) * radius - 6;
        const y = height / 2 + Math.sin(angle) * radius - 6;
        bone.style.left = x + "px";
        bone.style.top = y + "px";
        soulBox.appendChild(bone);
        bones.push({ el: bone, vx: (width / 2 - x) * 0.008, vy: (height / 2 - y) * 0.008 });
      }
    }

    spawnRing();
    setTimeout(spawnRing, 800);
    setTimeout(spawnRing, 1600);

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        const rectBefore = b.el.getBoundingClientRect();
        let x = parseFloat(b.el.style.left) || 0;
        let y = parseFloat(b.el.style.top) || 0;
        x += b.vx;
        y += b.vy;
        b.el.style.left = x + "px";
        b.el.style.top = y + "px";
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) {
          damagePlayer(2);
        }
      });

      requestAnimationFrame(loop);
    }

    requestAnimationFrame(loop);
  }

  // --- Fight bar ---
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
      showDialogue("* \"whoops. almost got me there.\"");
    } else {
      const damage = Math.max(1, Math.floor(hitStrength * 10));
      enemyHP -= damage;
      updateEnemyHP();
      damageText.textContent = quality + " -" + damage;
      showDialogue("* you actually hit sans.\n* \"guess i let my guard down.\"");
    }

    setTimeout(() => {
      damageText.textContent = "";
      if (enemyHP > 0 && playerHP > 0) {
        startSansAttack();
      }
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

  // --- Sub menus ---
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
    if (currentSubType === "ACT") {
      handleActOption(opt.id);
    } else if (currentSubType === "ITEM") {
      handleItemOption(opt.id);
    } else if (currentSubType === "MERCY") {
      handleMercyOption(opt.id);
    }
  }

  function handleActOption(id) {
    actCount++;

    if (!isGenocideRoute) {
      if (id === "CHECK") {
        showDialogue("* sans - atk 1 def 1.\n* this feels more like a test than a fight.");
      } else if (id === "JOKE") {
        showDialogue("* you tell a joke.\n* sans snorts.\n* \"heh. not bad.\"");
      } else if (id === "FLIRT") {
        showDialogue("* you try to flirt.\n* sans just grins.\n* \"wow, kid. bold move.\"");
      } else if (id === "APOLOGIZE") {
        showDialogue("* you apologize for something.\n* sans shrugs.\n* \"nah. we’re good.\"");
      } else if (id === "ENCOURAGE") {
        showDialogue("* you say you’ll do your best.\n* sans nods.\n* \"that’s all anyone can ask.\"");
      }
    } else {
      if (id === "CHECK") {
        showDialogue("* sans - atk 1 def 1.\n* he looks tired.\n* and disappointed.");
      } else if (id === "JOKE") {
        showDialogue("* you tell a joke.\n* sans doesn’t laugh this time.\n* \"...yeah.\"");
      } else if (id === "FLIRT") {
        showDialogue("* you try to flirt.\n* sans stares at you.\n* \"really? after all that?\"");
      } else if (id === "APOLOGIZE") {
        showDialogue("* you apologize.\n* sans is quiet.\n* \"sorry’s not a reset button, kid.\"");
      } else if (id === "ENCOURAGE") {
        showDialogue("* you say you’ll do better.\n* sans sighs.\n* \"hope you mean it.\"");
      }
    }

    closeSubMenu();

    setTimeout(() => {
      if (playerHP > 0) startSansAttack();
    }, 1000);
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

    setTimeout(() => {
      if (playerHP > 0) startSansAttack();
    }, 1000);
  }

  function handleMercyOption(id) {
    if (id === "SPARE") {
      if (isGenocideRoute) {
        showDialogue("* you try to spare sans.\n* he doesn’t move.\n* \"nah, kid. we’re past that.\"");
        closeSubMenu();
        setTimeout(() => {
          if (playerHP > 0) startSansAttack();
        }, 1200);
        return;
      }

      if (canSpare) {
        showDialogue("* you tell sans you’re done.\n* sans grins.\n* \"heh. you passed.\"\n* \"nice dodging, kid.\"");
        closeSubMenu();
        setTimeout(() => {
          endBattle(true, true, false);
        }, 2000);
      } else {
        showDialogue("* you try to spare sans.\n* he shakes his head.\n* \"not yet, kid.\"");
        closeSubMenu();
        setTimeout(() => {
          if (playerHP > 0) startSansAttack();
        }, 1000);
      }
    } else if (id === "FLEE") {
      showDialogue("* you consider running away...\n* but something keeps you here.");
      closeSubMenu();
      setTimeout(() => {
        if (playerHP > 0) startSansAttack();
      }, 1000);
    }
  }

  function endBattle(playerWon, spared = false, killedSans = false) {
    endScreen.style.display = "flex";

    if (!playerWon) {
      if (isGenocideRoute) {
        endText.textContent = "YOU DIED.\n\n* \"guess we’re done for now.\"\n* \"see ya next reset, kid.\"";
      } else {
        endText.textContent = "YOU DIED.\n\n* \"whoops. guess that was a bit much.\"\n* \"sorry, kid.\"";
      }
    } else if (spared) {
      const gold = 20;
      endText.textContent = "YOU WON!\nGained " + gold + "G.\n\n* you feel like you learned something.";
    } else if (killedSans) {
      const gold = 10;
      const exp = 1;
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
      if (!isGenocideRoute && pureAttackTurns >= 4) {
        sansCanBeHit = true;
        showDialogue("* you keep going for the hit.\n* sans stops moving quite as fast.");
      } else if (isGenocideRoute && pureAttackTurns >= 2) {
        sansCanBeHit = true;
      } else {
        showDialogue("* you get ready to attack.");
      }

      setTimeout(() => {
        showDialogue("");
        startFightBar();
      }, 500);
    } else if (action === "ACT") {
      openSubMenu("ACT", [
        { id: "CHECK", label: "Check" },
        { id: "JOKE", label: "Joke" },
        { id: "FLIRT", label: "Flirt" },
        { id: "APOLOGIZE", label: "Apologize" },
        { id: "ENCOURAGE", label: "Encourage" }
      ]);
    } else if (action === "ITEM") {
      if (items.length === 0) {
        showDialogue("* (you don’t have any items.)");
        setTimeout(() => {
          if (playerHP > 0) startSansAttack();
        }, 1000);
      } else {
        openSubMenu("ITEM", items.map(it => ({ id: it.id, label: it.name })));
      }
    } else if (action === "MERCY") {
      const spareLabel = (canSpare && !isGenocideRoute) ? "Spare (yellow)" : "Spare";
      openSubMenu("MERCY", [
        { id: "SPARE", label: spareLabel },
        { id: "FLEE", label: "Flee" }
      ]);
    }
  }

  // --- Input ---
  document.addEventListener("keydown", (e) => {
    keys[e.key] = true;

    if (phase === "END") return;

    if (e.key === "z" || e.key === "Z") {
      if (phase === "INTRO") {
        phase = "PLAYER_TURN";
        showDialogue(isGenocideRoute
          ? "* \"so. here we are again.\"\n* \"this time... let's see what you learned.\""
          : "* heya, kid.\n* how 'bout a little test of skill?");
      } else if (phase === "PLAYER_TURN") {
        confirmMenuSelection();
      } else if (phase === "MENU_SUB") {
        confirmSubSelection();
      } else if (phase === "FIGHT_BAR") {
        stopFightBar();
      }
    }

    if (e.key === "x" || e.key === "X") {
      if (phase === "MENU_SUB") {
        closeSubMenu();
      }
    }

    if (phase === "PLAYER_TURN") {
      if (e.key === "a" || e.key === "A") {
        setMenuIndex(menuIndex - 1);
      } else if (e.key === "d" || e.key === "D") {
        setMenuIndex(menuIndex + 1);
      }
    } else if (phase === "MENU_SUB") {
      if (e.key === "a" || e.key === "A") {
        setSubIndex(subIndex - 1);
      } else if (e.key === "d" || e.key === "D") {
        setSubIndex(subIndex + 1);
      }
    }
  });

  document.addEventListener("keyup", (e) => {
    keys[e.key] = false;
  });

  // --- Init ---
  function initIntro() {
    if (isGenocideRoute) {
      showDialogue("* \"heya.\"\n* \"back again, huh?\"\n* \"guess we’re doin’ this the hard way this time.\"");
    } else {
      showDialogue("* heya, kid.\n* how 'bout a little test of skill?");
    }
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
