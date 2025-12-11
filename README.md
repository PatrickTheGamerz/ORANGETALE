<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>A Test of Skill - Sans Fight (Base, Fixed)</title>
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

    /* Enemy area */
    #enemy-area {
      position: absolute;
      top: 20px;
      left: 0;
      width: 100%;
      height: 160px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      z-index: 5;
    }
    #enemy-name {
      font-size: 24px;
      margin-bottom: 8px;
      text-transform: lowercase;
    }
    #enemy-sprite {
      width: 80px;
      height: 80px;
      border: 2px solid white;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 8px;
      font-size: 12px;
      position: relative;
      background: black;
    }

    /* Monster bubble to the right of Sans */
    #monster-bubble {
      position: absolute;
      top: -10px;
      left: 90px;
      max-width: 260px;
      padding: 8px;
      border: 2px solid white;
      background: white;
      color: black;
      font-size: 16px;
      white-space: pre-line;
      display: none;
    }
    #monster-bubble.special {
      border-color: #ff8000;
      color: #ff8000;
    }

    #enemy-hp-container {
      width: 280px;
      height: 16px;
      border: 2px solid white;
      position: relative;
      margin-top: 4px;
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

    /* Bottom main box */
    #bottom-box {
      position: absolute;
      left: 40px;
      right: 40px;
      bottom: 40px;
      height: 160px;
      border: 4px solid white;
      box-sizing: border-box;
      background: black;
      z-index: 1;
    }

    /* Soul box */
    #soul-box {
      position: absolute;
      left: 160px;
      right: 160px;
      top: 20px;
      height: 80px;
      border: 2px solid white;
      box-sizing: border-box;
      overflow: hidden;
      display: none;
      background: black;
      z-index: 2;
    }

    #soul {
      width: 14px;
      height: 14px;
      background: red;
      position: absolute;
      border-radius: 50% 50% 40% 40%;
      transform: rotate(45deg);
      box-shadow: 0 0 6px rgba(255,0,0,0.9);
    }

    .bone {
      position: absolute;
      background: white;
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

    /* Dialogue box (bottom) */
    #dialogue-box {
      position: absolute;
      left: 10px;
      right: 10px;
      top: 10px;
      height: 80px;
      border: 2px solid white;
      box-sizing: border-box;
      background: white;
      color: black;
      font-size: 18px;
      white-space: pre-line;
      padding: 6px;
      display: none;
      z-index: 3;
    }

    /* Target menu (inside bottom box) */
    #target-menu {
      position: absolute;
      left: 180px;
      right: 180px;
      top: 30px;
      height: 40px;
      border: 2px solid white;
      background: black;
      display: none;
      justify-content: center;
      align-items: center;
      gap: 20px;
      font-size: 18px;
      z-index: 4;
    }
    .target-option {
      padding: 2px 8px;
      border: 2px solid white;
    }
    .target-option.selected {
      color: yellow;
      border-color: yellow;
    }

    /* Fight bar (above dialogue) */
    #fight-bar-container {
      position: absolute;
      left: 20px;
      right: 20px;
      top: 30px;
      height: 40px;
      display: none;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      color: white;
      z-index: 5;
    }

    #fight-bar {
      width: 360px;
      height: 20px;
      border: 2px solid white;
      position: relative;
      margin-bottom: 4px;
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

    /* Bottom UI (HP, menu) */
    #ui-bar {
      position: absolute;
      left: 50px;
      right: 50px;
      bottom: 50px;
      height: 140px;
      pointer-events: none;
      z-index: 6;
    }

    #hp-line {
      position: absolute;
      left: 0;
      right: 0;
      bottom: 90px;
      height: 30px;
      display: flex;
      align-items: center;
      justify-content: flex-start;
      gap: 10px;
      font-size: 18px;
      pointer-events: none;
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
      background: #ff0000;
      width: 100%;
    }

    #menu-line {
      position: absolute;
      left: 0;
      right: 0;
      bottom: 10px;
      height: 60px;
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 40px;
      font-size: 22px;
      pointer-events: auto;
    }

    .menu-item {
      cursor: pointer;
    }
    .menu-item.selected {
      color: yellow;
    }

    #sub-menu {
      position: absolute;
      left: 200px;
      right: 200px;
      bottom: 80px;
      height: 40px;
      display: none;
      justify-content: center;
      align-items: center;
      gap: 20px;
      font-size: 18px;
      pointer-events: auto;
    }
    .sub-option {
      border: 2px solid white;
      padding: 4px 8px;
      cursor: pointer;
      background: white;
      color: black;
    }
    .sub-option.selected {
      color: yellow;
      border-color: yellow;
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
      z-index: 100;
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

  <div id="bottom-box">
    <div id="soul-box">
      <div id="soul"></div>
    </div>

    <div id="dialogue-box"></div>

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
  </div>

  <div id="ui-bar">
    <div id="hp-line">
      <span>CHARA</span>
      <span>LV 1</span>
      <span>HP</span>
      <div id="hp-bar-inner">
        <div id="hp-fill"></div>
      </div>
      <span id="hp-text">20 / 20</span>
    </div>

    <div id="menu-line">
      <span class="menu-item selected" data-action="FIGHT">FIGHT</span>
      <span class="menu-item" data-action="ACT">ACT</span>
      <span class="menu-item" data-action="ITEM">ITEM</span>
      <span class="menu-item" data-action="MERCY">MERCY</span>
    </div>

    <div id="sub-menu"></div>
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
  let enemyHP = 1; // numeric only, no NaN hacks
  const enemyMaxHP = 1;

  const menuItems = ["FIGHT", "ACT", "ITEM", "MERCY"];
  let menuIndex = 0;
  const menuElements = Array.from(document.querySelectorAll(".menu-item"));

  let inSoulMode = false;
  let canMoveSoul = false;
  let phase = "INTRO";
  // INTRO, PLAYER_TURN, ENEMY_ATTACK, FIGHT_TARGET, FIGHT_BAR, MENU_SUB, END

  let turnCount = 0;
  let actCount = 0;
  let pureAttackTurns = 0;
  let sansCanBeHit = false;
  let canSpare = false;
  let usedBlueIntro = false;
  let usedPreSpareSpecial = false;

  // Anti-spam
  let playerActionUsedThisTurn = false;

  const dialogueBox = document.getElementById("dialogue-box");
  const monsterBubble = document.getElementById("monster-bubble");
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
  const targetMenu = document.getElementById("target-menu");
  const targetOptions = Array.from(document.querySelectorAll(".target-option"));

  let soulX = 0;
  let soulY = 0;
  let soulColor = "red";
  let gravityEnabled = false;
  let gravity = 0.12;
  let yVelocity = 0;
  let jumpHold = 0;
  const maxJumpHold = 18;
  let baseSoulSpeed = 3.0;
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
    { id: "CREAM", name: "Nice Cream", heal: 12 }
  ];

  // Text writer
  let textWriterActive = false;
  let textWriterBuffer = "";
  let textWriterIndex = 0;
  let textWriterTargetElement = null;
  let textWriterCallback = null;
  const TEXT_SPEED = 50;

  function writeText(element, text, callback = null) {
    textWriterActive = true;
    textWriterBuffer = text;
    textWriterIndex = 0;
    textWriterTargetElement = element;
    textWriterCallback = callback;
    element.textContent = "";
    stepTextWriter();
  }

  function stepTextWriter() {
    if (!textWriterActive || !textWriterTargetElement) return;
    if (textWriterIndex > textWriterBuffer.length) {
      textWriterActive = false;
      if (textWriterCallback) textWriterCallback();
      return;
    }
    textWriterTargetElement.textContent = textWriterBuffer.slice(0, textWriterIndex);
    textWriterIndex++;
    setTimeout(stepTextWriter, TEXT_SPEED);
  }

  function showDialogue(text, afterDelayCallback=null) {
    dialogueBox.style.display = "block";
    writeText(dialogueBox, text, () => {
      if (afterDelayCallback) {
        setTimeout(afterDelayCallback, 2000);
      }
    });
  }
  function hideDialogue() {
    dialogueBox.style.display = "none";
  }

  function showMonsterBubble(text, isSpecial=false) {
    monsterBubble.style.display = "block";
    monsterBubble.classList.toggle("special", isSpecial);
    writeText(monsterBubble, text);
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
    enemyHPText.textContent = "HP: " + enemyHP.toFixed(1) + " / 1";
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
    soulX = (boxRect.width / 2) - 7;
    soulY = (boxRect.height / 2) - 7;
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

      if (gravityEnabled) {
        if (keys["w"] || keys["W"]) {
          if (jumpHold < maxJumpHold) {
            yVelocity -= 0.4;
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
      if (soulX > boxWidth - 14) soulX = boxWidth - 14;
      if (soulY < 0) {
        soulY = 0;
        if (gravityEnabled) yVelocity = 0;
      }
      if (soulY > boxHeight - 14) {
        soulY = boxHeight - 14;
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
    soulSpeed = baseSoulSpeed + pureFactor * 0.2;
  }

  // ========= ATTACK DISPATCH =========
  function startSansAttack() {
    phase = "ENEMY_ATTACK";
    turnCount++;
    updateDifficulty();
    playerActionUsedThisTurn = false;

    if (!usedBlueIntro && turnCount === 1) {
      exitSoulMode();
      showMonsterBubble("heya.\nlet's see what you can do.");
      setTimeout(() => {
        showMonsterBubble("oh right, my blue attack.\nalmost forgot about it,\nheh heh heh.");
        setTimeout(() => {
          showMonsterBubble("are ya ready?\n'cause here it comes.");
          setTimeout(() => {
            blueSoulZoneAttack(() => {
              hideMonsterBubble();
              showDialogue("* \"what are you looking so blue for?\"", () => {
                phase = "PLAYER_TURN";
              });
            });
          }, 600);
        }, 800);
      }, 800);
      usedBlueIntro = true;
      return;
    }

    if (!usedPreSpareSpecial && turnCount >= 4 && !canSpare) {
      usedPreSpareSpecial = true;
      exitSoulMode();
      showMonsterBubble("time for a little\nspecial demonstration.", true);
      setTimeout(() => {
        phase1PreSpareSpecial(() => {
          hideMonsterBubble();
          showMonsterBubble("...yeah.\ni think you're ready.");
          showDialogue(
            "* \"welp, that sure was fun.\"\n" +
            "* \"now just spare me and we can both be on our way.\"",
            () => {
              canSpare = true;
              sansCanBeHit = true;
              phase = "PLAYER_TURN";
            }
          );
        });
      }, 800);
      return;
    }

    enterSoulMode();

    const attacks = [
      phase1WaveAndBounce,
      phase1SpinningBlueBones
    ];
    const attack = attacks[Math.floor(Math.random() * attacks.length)];
    attack(() => {
      exitSoulMode();
      if (playerHP > 0) {
        const lines = [
          "* \"nice dodging.\"",
          "* \"still standin', huh?\"",
          "* \"not bad, kid.\""
        ];
        showDialogue(lines[Math.floor(Math.random() * lines.length)], () => {
          phase = "PLAYER_TURN";
        });
      }
    });
  }

  // ========= BLUE SOUL ZONE ATTACK =========
  function blueSoulZoneAttack(onEnd) {
    setSoulColor("blue", true);
    enterSoulMode();
    const boxRect = soulBox.getBoundingClientRect();
    const h = boxRect.height;

    const zoneDelay = 1000;
    const zoneDuration = 1800;
    let zone;
    setTimeout(() => {
      zone = spawnBone(0, h - 18, boxRect.width, 18, false);
    }, zoneDelay);

    const startTime = performance.now();

    function loop(t) {
      if (!inSoulMode) {
        if (zone) zone.remove();
        setSoulColor("red", false);
        onEnd();
        return;
      }
      const elapsed = t - startTime;
      if (elapsed > zoneDelay + zoneDuration) {
        if (zone) zone.remove();
        setSoulColor("red", false);
        exitSoulMode();
        onEnd();
        return;
      }

      if (zone) {
        const soulRect = soul.getBoundingClientRect();
        const rect = zone.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(1);
      }

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= BONE & BLASTER HELPERS =========
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

  // ========= PHASE 1 ATTACKS =========
  function phase1WaveAndBounce(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const w = boxRect.width;
    const h = boxRect.height;
    const bones = [];
    const duration = 6000;
    const startTime = performance.now();

    for (let i = 0; i < 5; i++) {
      const y = 20 + i * 12;
      const bone = spawnBone(-40, y, 40, 10);
      bones.push({ el: bone, x: -40, y, vx: 2.2, bounced: false, blueTime: 0 });
    }

    const b1 = spawnBone(w + 40, 20, 40, 10);
    const b2 = spawnBone(w + 40, h - 40, 40, 10);
    bones.push({ el: b1, x: w + 40, y: 20, vx: -2.4, bounceOnly: true });
    bones.push({ el: b2, x: w + 40, y: h - 40, vx: -2.4, bounceOnly: true });

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
        const rect = b.el.getBoundingClientRect();

        if (!b.bounceOnly && !b.bounced && (b.x <= 0 || b.x + 40 >= w)) {
          b.vx = -b.vx;
          b.bounced = true;
          b.el.classList.add("blue");
        } else if (!b.bounceOnly && b.bounced) {
          b.blueTime++;
          if (b.blueTime > 90) b.el.classList.remove("blue");
        }

        if (b.bounceOnly && (b.x <= 0 || b.x + 40 >= w)) {
          b.vx = -b.vx;
        }

        const blue = b.el.classList.contains("blue");
        if (rectsOverlap(rect, soulRect)) {
          if (blue && gravityEnabled) damagePlayer(1);
          if (!blue) damagePlayer(1);
        }
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  function phase1SpinningBlueBones(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const cx = boxRect.width / 2;
    const cy = boxRect.height / 2;
    setSoulColor("blue", true);
    soulX = cx - 7;
    soulY = cy - 7;
    soul.style.left = soulX + "px";
    soul.style.top = soulY + "px";

    const bones = [];
    const count = 6;
    const radius = 40;
    let angle = 0;
    const duration = 5000;
    const startTime = performance.now();

    for (let i = 0; i < count; i++) {
      const bone = spawnBone(cx, cy, 12, 12, true);
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

      angle += 0.035;
      const soulRect = soul.getBoundingClientRect();
      const moving = keys["w"] || keys["a"] || keys["s"] || keys["d"] ||
                     keys["W"] || keys["A"] || keys["S"] || keys["D"];

      bones.forEach(b => {
        b.spin += 0.2;
        const x = cx + Math.cos(angle + b.angleOffset) * radius;
        const y = cy + Math.sin(angle + b.angleOffset) * radius;
        b.el.style.left = (x - 6) + "px";
        b.el.style.top = (y - 6) + "px";
        b.el.style.transform = "rotate(" + (b.spin * 180 / Math.PI) + "deg)";
        const rect = b.el.getBoundingClientRect();
        if (moving && rectsOverlap(rect, soulRect)) damagePlayer(1);
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

    for (let i = 0; i < 4; i++) {
      const y = 20 + i * 14;
      const left = spawnBone(-40, y, 40, 10);
      const right = spawnBone(w + 40, y + 8, 40, 10);
      bones.push({ el: left, x: -40, y, vx: 2.6 });
      bones.push({ el: right, x: w + 40, y: y + 8, vx: -2.6 });
    }

    setTimeout(() => {
      for (let i = 0; i < 4; i++) {
        const fromLeft = i % 2 === 0;
        const startY = 20 + i * 14;
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

    if (!sansCanBeHit) {
      damageText.textContent = quality + " (sans dodged)";
      showDialogue("* \"heh. close.\"\n* \"but not that close.\"", () => {
        if (playerHP > 0) startSansAttack();
      });
    } else {
      const damage = Math.max(1, Math.floor(hitStrength * 10));
      enemyHP = Math.max(0, enemyHP - damage);
      updateEnemyHP();
      damageText.textContent = quality + " - " + damage;
      showDialogue("* you land a hit.\n* something feels... wrong.", () => {
        if (playerHP > 0) startSansAttack();
      });
    }
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
      showDialogue("* sans - atk 1 def 1.\n* the second skeleton brother stands firm.", () => {
        if (playerHP > 0) startSansAttack();
      });
    } else if (id === "JOKE") {
      showDialogue("* you tell a joke.\n* sans chuckles.\n* \"heh. not bad, kid.\"", () => {
        if (playerHP > 0) startSansAttack();
      });
    } else if (id === "FLIRT") {
      showDialogue("* you try to flirt.\n* sans just grins.\n* \"you know i'm not the tall one, right?\"", () => {
        if (playerHP > 0) startSansAttack();
      });
    } else if (id === "TALK") {
      showDialogue("* you talk about paps.\n* sans smiles.\n* \"yeah. he’s really lookin' forward to you.\"", () => {
        if (playerHP > 0) startSansAttack();
      });
    }
    closeSubMenu();
  }

  function handleItemOption(id) {
    if (playerActionUsedThisTurn) return;
    playerActionUsedThisTurn = true;
    const item = items.find(i => i.id === id);
    if (!item) {
      showDialogue("* (you fumble with your pockets.)", () => {
        if (playerHP > 0) startSansAttack();
      });
    } else {
      playerHP = Math.min(playerMaxHP, playerHP + item.heal);
      updatePlayerHP();
      showDialogue("* you eat the " + item.name.toLowerCase() + ".\n* you recovered " + item.heal + " hp.", () => {
        if (playerHP > 0) startSansAttack();
      });
      items = items.filter(i => i.id !== id);
    }
    closeSubMenu();
  }

  function handleMercyOption(id) {
    if (playerActionUsedThisTurn) return;
    playerActionUsedThisTurn = true;
    if (id === "SPARE") {
      if (!canSpare) {
        showDialogue("* you reach for MERCY.\n* \"sorry pal, no mercy 'till you prove yourself.\"", () => {
          if (playerHP > 0) startSansAttack();
        });
        closeSubMenu();
        return;
      }
      showDialogue(
        "* you lower your hands.\n" +
        "* sans smiles.\n" +
        "* \"heh. not bad.\"\n" +
        "* \"guess we can both move on now.\"",
        () => endBattle(true, true, false)
      );
      closeSubMenu();
    } else if (id === "FLEE") {
      showDialogue("* you think about running away...\n* but your feet won't move.", () => {
        if (playerHP > 0) startSansAttack();
      });
      closeSubMenu();
    }
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
      showDialogue("* you get ready to attack.", () => {
        hideDialogue();
        openTargetMenu();
      });
    } else if (action === "ACT") {
      openSubMenu("ACT", [
        { id: "CHECK", label: "Check" },
        { id: "JOKE", label: "Joke" },
        { id: "FLIRT", label: "Flirt" },
        { id: "TALK", label: "Talk" }
      ]);
    } else if (action === "ITEM") {
      if (items.length === 0) {
        showDialogue("* (you don’t have any items.)", () => {
          if (playerHP > 0) startSansAttack();
        });
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
        showDialogue("* heya.\n* how 'bout a little test of skill?");
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
      if (phase === "MENU_SUB") {
        closeSubMenu();
        playerActionUsedThisTurn = false;
      }
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
    dialogueBox.style.display = "block";
    writeText(dialogueBox, "* ...");
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
