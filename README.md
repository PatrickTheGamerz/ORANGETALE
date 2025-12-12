<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Sans Battle Prototype</title>
<style>
  body {
    background: #000;
    color: white;
    font-family: "Courier New", monospace;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
  }

  #gameContainer {
    position: relative;
    width: 640px;
    height: 480px;
    border: 4px solid white;
    box-sizing: border-box;
    background: black;
    overflow: hidden;
  }

  #enemyArea {
    position: absolute;
    top: 20px;
    left: 0;
    width: 100%;
    height: 200px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #sans {
    width: 80px;
    height: 80px;
    background: white;
    border: 2px solid #444;
  }

  /* Monster speech bubble to the right of Sans */
  #monsterDialogue {
    position: absolute;
    top: 25px;
    left: 340px;
    width: 260px;
    min-height: 60px;
    border: 2px solid white;
    padding: 10px;
    box-sizing: border-box;
    background: black;
    color: white;
    white-space: pre-line;
    display: none;
  }

  #dialogueBox {
    position: absolute;
    left: 20px;
    bottom: 120px;
    width: 600px;
    height: 80px;
    border: 2px solid white;
    padding: 10px;
    box-sizing: border-box;
    white-space: pre-line;
    display: none;
  }

  #dialogueBox .special {
    color: #ffff00;
  }

  #soulBox {
    position: absolute;
    left: 120px;
    bottom: 180px;
    width: 400px;
    height: 180px;
    border: 2px solid white;
    display: none;
    overflow: hidden;
    box-sizing: border-box;
  }

  #soul {
    position: absolute;
    width: 16px;
    height: 16px;
    background: red;
    box-shadow: 0 0 6px rgba(255,0,0,0.9);
  }

  .bone {
    position: absolute;
    background: white;
    box-shadow: 0 0 4px rgba(255,255,255,0.8);
  }

  .bone.blue {
    background: #0000ff;
    box-shadow: 0 0 8px rgba(0,128,255,0.9);
  }

  .blaster {
    position: absolute;
    width: 40px;
    height: 40px;
    background: #444;
    border: 2px solid white;
    box-sizing: border-box;
  }

  .blast-beam {
    position: absolute;
    background: #00ffff;
    opacity: 0.8;
  }

  #hpContainer {
    position: absolute;
    left: 20px;
    bottom: 20px;
    width: 300px;
    height: 80px;
    border: 2px solid white;
    padding: 8px;
    box-sizing: border-box;
  }

  #hpLabel {
    font-size: 14px;
    margin-bottom: 6px;
  }

  #hpBarOuter {
    width: 100%;
    height: 16px;
    border: 1px solid white;
    box-sizing: border-box;
    margin-bottom: 4px;
  }

  #hpBarInner {
    width: 100%;
    height: 100%;
    background: yellow;
  }

  #hpText {
    font-size: 12px;
  }

  #enemyHPContainer {
    position: absolute;
    top: 220px;
    left: 20px;
    width: 300px;
    height: 40px;
    border: 2px solid white;
    padding: 5px;
    box-sizing: border-box;
  }

  #enemyHPBarOuter {
    width: 100%;
    height: 10px;
    border: 1px solid white;
    box-sizing: border-box;
    margin-bottom: 4px;
  }

  #enemyHPBarInner {
    width: 100%;
    height: 100%;
    background: #ff0000;
  }

  #enemyHPText {
    font-size: 12px;
  }

  #menu {
    position: absolute;
    right: 20px;
    bottom: 20px;
    width: 280px;
    height: 80px;
    border: 2px solid white;
    display: flex;
    justify-content: space-around;
    align-items: center;
    box-sizing: border-box;
  }

  .menu-item {
    padding: 4px 8px;
    border: 1px solid transparent;
  }

  .menu-item.selected {
    border: 1px solid white;
  }

  /* Sub menu (ACT / ITEM / MERCY) */
  #subMenu {
    position: absolute;
    right: 20px;
    bottom: 110px;
    width: 280px;
    height: 80px;
    border: 2px solid white;
    display: none;
    justify-content: space-around;
    align-items: center;
    box-sizing: border-box;
  }

  .sub-option {
    padding: 4px 8px;
    border: 1px solid transparent;
  }

  .sub-option.selected {
    border: 1px solid white;
  }

  #fightBarContainer {
    position: absolute;
    left: 120px;
    bottom: 180px;
    width: 400px;
    height: 50px;
    border: 2px solid white;
    display: none;
    align-items: center;
    justify-content: center;
    box-sizing: border-box;
  }

  #fightBar {
    position: relative;
    width: 360px;
    height: 16px;
    border: 1px solid white;
    box-sizing: border-box;
  }

  #fightZone {
    position: absolute;
    left: 40%;
    width: 20%;
    height: 100%;
    background: #ffaaaa;
  }

  #fightPointer {
    position: absolute;
    top: -4px;
    width: 4px;
    height: 24px;
    background: white;
  }

  #damageText {
    position: absolute;
    top: 52px;
    width: 100%;
    text-align: center;
    font-size: 14px;
  }

  /* Target selection after FIGHT */
  #targetMenu {
    position: absolute;
    left: 120px;
    bottom: 180px;
    width: 400px;
    height: 180px;
    border: 2px solid white;
    box-sizing: border-box;
    display: none;
    padding: 10px;
  }

  .target-option {
    padding: 4px 8px;
    border: 1px solid transparent;
    display: inline-block;
    margin-right: 10px;
  }

  .target-option.selected {
    border: 1px solid white;
  }

  #endScreen {
    position: absolute;
    left: 120px;
    bottom: 180px;
    width: 400px;
    height: 180px;
    border: 2px solid white;
    background: black;
    display: none;
    align-items: center;
    justify-content: center;
    text-align: left;
    white-space: pre-line;
    padding: 10px;
    box-sizing: border-box;
  }

  #endText {
    font-size: 14px;
  }
</style>
</head>
<body>
<div id="gameContainer">
  <div id="enemyArea">
    <div id="sans"></div>
    <div id="monsterDialogue"></div>
  </div>

  <div id="enemyHPContainer">
    <div id="enemyHPBarOuter">
      <div id="enemyHPBarInner"></div>
    </div>
    <div id="enemyHPText"></div>
  </div>

  <div id="dialogueBox"></div>

  <div id="soulBox">
    <div id="soul"></div>
  </div>

  <div id="hpContainer">
    <div id="hpLabel">CHARA LV 19</div>
    <div id="hpBarOuter">
      <div id="hpBarInner"></div>
    </div>
    <div id="hpText"></div>
  </div>

  <div id="menu">
    <span class="menu-item" data-action="FIGHT">FIGHT</span>
    <span class="menu-item" data-action="ACT">ACT</span>
    <span class="menu-item" data-action="ITEM">ITEM</span>
    <span class="menu-item" data-action="MERCY">MERCY</span>
  </div>

  <div id="subMenu"></div>

  <div id="targetMenu">
    <span class="target-option selected" data-id="SANS">SANS</span>
  </div>

  <div id="fightBarContainer">
    <div id="fightBar">
      <div id="fightZone"></div>
      <div id="fightPointer"></div>
    </div>
    <div id="damageText"></div>
  </div>

  <div id="endScreen">
    <div id="endText"></div>
  </div>
</div>

<script>
  // ========= CORE STATE =========
  let playerMaxHP = 92;
  let playerHP = 92;
  let enemyMaxHP = 1;
  let enemyHP = 1;
  let enemyShownFractionHP = 1; // used for 0.0000128 / 1 illusion
  let phase = "INTRO";

  let inSoulMode = false;
  let canMoveSoul = false;

  let soulX = 0;
  let soulY = 0;
  let soulColor = "red";
  let gravityEnabled = false;
  let gravity = 0.12;
  let yVelocity = 0;
  let baseSoulSpeed = 3.5;
  let soulSpeed = baseSoulSpeed;

  // Blue jump control
  let blueJumpHeldFrames = 0;
  let blueJumpMaxFrames = 20; // limit how long W gives upward push
  let blueJumpCooldownFrames = 10;

  const keys = {};
  let menuIndex = 0;
  const menuItems = ["FIGHT", "ACT", "ITEM", "MERCY"];

  let currentSubType = null;
  let currentSubOptions = [];
  let subIndex = 0;

  let fightPointerPos = 0;
  let fightPointerDir = 1;
  let fightBarActive = false;

  let playerCanActThisTurn = true; // prevent multiple actions per turn
  let targetMenuActive = false;
  let targetIndex = 0;

  let items = [
    { id: "PIE", name: "Butterscotch Pie", heal: 20 },
    { id: "CREAM", name: "Nice Cream", heal: 12 },
    { id: "BANDAGE", name: "Bandage", heal: 7 }
  ];

  // Phase flags and counters
  let turnCount = 0;
  let actCount = 0;
  let pureAttackTurns = 0;
  let usedBlueIntro = false;
  let usedFinalAttackPhase1 = false;

  let canSpare = false;
  let sansCanBeHit = false;

  let phase2AActive = false;
  let phase2ATurns = 0;
  let phase2ATurnTarget = 5;
  let phase2AStartTargets = 8;

  let phase2BQueued = false;

  let phase2CActive = false;
  let last4AttacksMode = false;

  let phase2AExhausted = false; // after huge spiral final done

  // DOM
  const soulBox = document.getElementById("soulBox");
  const soul = document.getElementById("soul");
  const dialogueBox = document.getElementById("dialogueBox");
  const monsterDialogue = document.getElementById("monsterDialogue");
  const hpFill = document.getElementById("hpBarInner");
  const hpText = document.getElementById("hpText");
  const enemyHPBar = document.getElementById("enemyHPBarInner");
  const enemyHPText = document.getElementById("enemyHPText");
  const menuElements = Array.from(document.querySelectorAll(".menu-item"));
  const subMenu = document.getElementById("subMenu");
  const fightBarContainer = document.getElementById("fightBarContainer");
  const fightBar = document.getElementById("fightBar");
  const fightZone = document.getElementById("fightZone");
  const fightPointer = document.getElementById("fightPointer");
  const damageText = document.getElementById("damageText");
  const endScreen = document.getElementById("endScreen");
  const endText = document.getElementById("endText");
  const targetMenu = document.getElementById("targetMenu");

  // ========= DIALOGUE HELPERS =========
  function showDialogue(text, special = false) {
    dialogueBox.style.display = "block";
    if (!special) {
      dialogueBox.textContent = text;
    } else {
      dialogueBox.innerHTML = `<span class="special">${text}</span>`;
    }
  }

  function hideDialogue() {
    dialogueBox.style.display = "none";
    dialogueBox.textContent = "";
  }

  function showMonsterDialogue(text) {
    monsterDialogue.style.display = "block";
    monsterDialogue.textContent = text;
  }

  function hideMonsterDialogue() {
    monsterDialogue.style.display = "none";
    monsterDialogue.textContent = "";
  }

  function updatePlayerHP() {
    if (playerHP < 0) playerHP = 0;
    const ratio = playerHP / playerMaxHP;
    hpFill.style.width = (ratio * 100) + "%";
    hpText.textContent = "HP " + playerHP + " / " + playerMaxHP;
    if (playerHP <= 0) endBattle(false);
  }

  function updateEnemyHP() {
    // During exhausted 2A we want the illusion 0.0000128 / 1 etc.
    const ratio = enemyHP / enemyMaxHP;
    enemyHPBar.style.width = (ratio * 100) + "%";
    if (phase2AExhausted) {
      const displayVal =
        enemyHP <= 0
          ? "0"
          : "0.0000128";
      enemyHPText.textContent = "HP: " + displayVal + " / 1";
    } else {
      enemyHPText.textContent = "HP: " + enemyHP + " / " + enemyMaxHP;
    }
  }

  function setMenuIndex(index) {
    menuIndex = (index + menuItems.length) % menuItems.length;
    menuElements.forEach((el, i) => {
      el.classList.toggle("selected", i === menuIndex);
    });
  }

  function setTargetIndex(index) {
    const options = Array.from(targetMenu.querySelectorAll(".target-option"));
    if (!options.length) return;
    targetIndex = (index + options.length) % options.length;
    options.forEach((opt, i) => {
      opt.classList.toggle("selected", i === targetIndex);
    });
  }

  function enterSoulMode() {
    inSoulMode = true;
    canMoveSoul = true;
    soulBox.style.display = "block";
    hideDialogue();
    hideMonsterDialogue();
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
    yVelocity = 0;
    blueJumpHeldFrames = 0;
  }

  // ========= MOVEMENT LOOP =========
  function soulMovementLoop() {
    if (inSoulMode && canMoveSoul) {
      const boxRect = soulBox.getBoundingClientRect();
      const boxWidth = boxRect.width;
      const boxHeight = boxRect.height;

      if (!gravityEnabled) {
        if (keys["w"] || keys["W"]) soulY -= soulSpeed;
        if (keys["s"] || keys["S"]) soulY += soulSpeed;
      } else {
        // Undertale-like: short controlled upward impulse, not infinite flight
        const wHeld = keys["w"] || keys["W"];
        if (wHeld && blueJumpHeldFrames < blueJumpMaxFrames) {
          yVelocity -= 0.3;
          blueJumpHeldFrames++;
        } else if (!wHeld) {
          if (blueJumpHeldFrames > 0) {
            blueJumpHeldFrames -= blueJumpCooldownFrames;
            if (blueJumpHeldFrames < 0) blueJumpHeldFrames = 0;
          }
        }
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

  function spawnBlaster(x, y, delay, beamDuration, damage, orientation, speedFactor = 1) {
    const blaster = document.createElement("div");
    blaster.classList.add("blaster");
    blaster.style.left = x + "px";
    blaster.style.top = y + "px";
    soulBox.appendChild(blaster);

    const beam = document.createElement("div");
    beam.classList.add("blast-beam");

    // Always 0.8s before the beam appears
    const totalDelay = delay + 800;

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
        if (t - startTime > beamDuration * speedFactor) {
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
    }, totalDelay);
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

    // Make them blue bones and also rotate themselves
    bone1.classList.add("blue");
    bone2.classList.add("blue");

    let angle = 0;
    let selfAngle1 = 0;
    let selfAngle2 = 0;

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
      selfAngle1 += 0.2;
      selfAngle2 -= 0.2;
      const r = 35;
      const x1 = cx + Math.cos(angle) * r;
      const y1 = cy + Math.sin(angle) * r - 20;
      const x2 = cx + Math.cos(angle + Math.PI) * r;
      const y2 = cy + Math.sin(angle + Math.PI) * r - 20;

      bone1.style.left = (x1 - 5) + "px";
      bone1.style.top = y1 + "px";
      bone1.style.transform = `rotate(${selfAngle1}rad)`;
      bone2.style.left = (x2 - 5) + "px";
      bone2.style.top = y2 + "px";
      bone2.style.transform = `rotate(${selfAngle2}rad)`;

      const soulRect = soul.getBoundingClientRect();
      const r1 = bone1.getBoundingClientRect();
      const r2 = bone2.getBoundingClientRect();
      // blue rule: damage only if player is moving
      const moving = (keys["w"] || keys["W"] || keys["a"] || keys["A"] || keys["s"] || keys["S"] || keys["d"] || keys["D"]);
      if (moving && (rectsOverlap(r1, soulRect) || rectsOverlap(r2, soulRect))) {
        damagePlayer(1);
      }

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

    // Wave from left
    for (let i = 0; i < 5; i++) {
      const bone = spawnBone(-40, 20 + i * 15, 40, 10);
      bones.push({ el: bone, x: -40, y: 20 + i * 15, speed: 2, bounced: false, bluePhase: false, blueTimer: 0 });
    }

    // Blue marker bone
    setTimeout(() => {
      const blue = spawnBone(width / 2 - 10, height - 20, 20, 20);
      blue.classList.add("blue");
      bones.push({ el: blue, blue: true });
    }, 1500);

    // Two "return" bones
    setTimeout(() => {
      const top = spawnBone(-40, 10, 40, 10);
      const bottom = spawnBone(width + 40, height - 20, 40, 10);
      bones.push({ el: top, x: -40, y: 10, speed: 3, dir: 1, specialBounce: true, bouncedTimes: 0 });
      bones.push({ el: bottom, x: width + 40, y: height - 20, speed: 3, dir: -1, specialBounce: true, bouncedTimes: 0 });
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
        } else if (b.specialBounce) {
          // special 2 bones that go one way and bounce
          b.x += b.speed * b.dir;
          const boxRect = soulBox.getBoundingClientRect();
          if (b.dir === 1 && b.x > boxRect.width - 40) {
            b.dir = -1;
            b.bouncedTimes++;
          } else if (b.dir === -1 && b.x < 0) {
            b.dir = 1;
            b.bouncedTimes++;
          }
          if (b.bouncedTimes > 1) {
            b.el.remove();
            b.dead = true;
            return;
          }
          b.el.style.left = b.x + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(1);
        } else {
          // the "wave" bones that bounce off wall then become blue for a bit
          if (b.dir == null) b.dir = 1;
          b.x += b.speed * b.dir;
          const boxRect = soulBox.getBoundingClientRect();
          if (!b.bounced) {
            if (b.x > boxRect.width - 40 || b.x < 0) {
              b.dir *= -1;
              b.bounced = true;
              // turn into blue bones for a few seconds
              b.bluePhase = true;
              b.blueTimer = 180; // frames approx
              b.el.classList.add("blue");
            }
          }
          if (b.bluePhase) {
            b.blueTimer--;
            if (b.blueTimer <= 0) {
              b.bluePhase = false;
              b.el.classList.remove("blue");
            }
          }
          b.el.style.left = b.x + "px";
          const rect = b.el.getBoundingClientRect();
          if (!b.bluePhase) {
            if (rectsOverlap(rect, soulRect)) damagePlayer(1);
          } else {
            // Blue-phase rule: damage only if moving
            const moving = (keys["w"] || keys["W"] || keys["a"] || keys["A"] || keys["s"] || keys["S"] || keys["d"] || keys["D"]);
            if (moving && rectsOverlap(rect, soulRect)) damagePlayer(1);
          }
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
    blue.classList.add("blue");
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

  // ========= SPECIAL ATTACK PHASE 1 (before spare) =========
  function specialAttackPhase1(onEnd) {
    enterSoulMode();
    setSoulColor("red", false);
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 6500;
    const startTime = performance.now();

    const floor = spawnBone(0, height - 18, width, 18);
    bones.push({ el: floor });

    // waves from sides
    for (let i = 0; i < 5; i++) {
      const y = 20 + i * 16;
      const boneL = spawnBone(-40, y, 40, 10);
      const boneR = spawnBone(width + 40, y + 8, 40, 10);
      bones.push({ el: boneL, x: -40, y, dir: 1, speed: 2.6 });
      bones.push({ el: boneR, x: width + 40, y: y + 8, dir: -1, speed: 2.6 });
    }

    // after the two lots pass, spawn 4 targeting bones
    setTimeout(() => {
      const count = 4;
      const boxRect = soulBox.getBoundingClientRect();
      const h = boxRect.height;
      for (let i = 0; i < count; i++) {
        const fromLeft = i % 2 === 0;
        const startY = Math.random() * (h - 10);
        const sx = soulX;
        const sy = soulY;
        const x = fromLeft ? -30 : boxRect.width + 30;
        const bone = spawnBone(x, startY, 20, 10);
        const dirX = sx - x;
        const dirY = sy - startY;
        const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
        bones.push({
          el: bone,
          x,
          y: startY,
          vx: (dirX / len) * 2.0,
          vy: (dirY / len) * 2.0,
          targeting: true
        });
      }
    }, 2800);

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
      const floorRect = floor.getBoundingClientRect();
      if (rectsOverlap(floorRect, soulRect)) damagePlayer(1);

      bones.forEach(b => {
        if (b.dir != null && !b.targeting) {
          b.x += b.speed * b.dir;
          b.el.style.left = b.x + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(1);
        } else if (b.targeting) {
          b.x += b.vx;
          b.y += b.vy;
          b.el.style.left = b.x + "px";
          b.el.style.top = b.y + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(2);
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= PHASE 2A TARGETING BONES + X/Y BLASTERS (modified) =========
  function phase2ATargetingBonesAndXYBlasters(count, onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 6500;
    const startTime = performance.now();

    // first wave: quick targeting bones
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
        vx: (dirX / len) * 2.4,
        vy: (dirY / len) * 2.4,
        mode: "chase"
      });
    }

    // When they hit side walls, bounce slightly then fall down and exit
    function updateBonesForPhase2A(b, boxRect) {
      if (b.mode === "chase") {
        b.x += b.vx;
        b.y += b.vy;
        if (b.x < 0 || b.x > boxRect.width - 20) {
          b.mode = "fall";
          b.vx = 0;
          b.vy = 2.0;
        }
      } else if (b.mode === "fall") {
        b.y += b.vy;
        if (b.y > boxRect.height + 20) {
          b.dead = true;
        }
      }
      b.el.style.left = b.x + "px";
      b.el.style.top = b.y + "px";
    }

    // X-shaped blasters (slower beams)
    setTimeout(() => {
      spawnBlaster(-20, -20, 200, 600, 2, "diagX", 0.7);
      spawnBlaster(width - 20, -20, 200, 600, 2, "diagX", 0.7);
    }, 1200);

    // Y-shape
    setTimeout(() => {
      spawnBlaster(width / 2 - 20, -20, 200, 600, 2, "down", 0.7);
      spawnBlaster(-20, height / 2 - 20, 400, 500, 2, "horizontal", 0.7);
      spawnBlaster(width - 20, height / 2 - 20, 400, 500, 2, "horizontal", 0.7);
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
      const boxRect2 = soulBox.getBoundingClientRect();
      bones.forEach(b => {
        if (b.dead) return;
        updateBonesForPhase2A(b, boxRect2);
        const rect = b.el.getBoundingClientRect();
        if (rectsOverlap(rect, soulRect)) damagePlayer(2);
      });
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
  }

  // ========= SPIRAL GASTER FINAL (PHASE 2A) – REWORKED =========
  function spiralGasterFinal(onEnd) {
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
    const spawnInterval = 400;
    let lastSpawn = 0;
    let spiralDone = false;
    let topBlastSpawned = false;
    let finalBoneFloodStarted = false;
    const bones = [];
    let storedPlayerPos = { x: soulX, y: soulY };

    function loop(t) {
      if (!inSoulMode) {
        bones.forEach(b => b.el.remove());
        setSoulColor("red", false);
        onEnd();
        return;
      }

      const elapsed = t - startTime;

      // First: single spiral wave of blasters
      if (!spiralDone) {
        if (elapsed - lastSpawn > spawnInterval) {
          lastSpawn = elapsed;
          angle += Math.PI / 6;
          const x = cx + Math.cos(angle) * (boxRect.width / 3);
          const y = cy + Math.sin(angle) * (boxRect.height / 3);
          spawnBlaster(x - 20, y - 20, 0, 500, 3, "down", 0.8);
          if (angle >= Math.PI * 2) {
            spiralDone = true;
            storedPlayerPos = { x: soulX, y: soulY };
          }
        }
      }

      // Next: 8 gaster blasters from top pointing roughly to stored player pos
      if (spiralDone && !topBlastSpawned && elapsed > 3000) {
        topBlastSpawned = true;
        const segments = 8;
        for (let i = 0; i < segments; i++) {
          const x = (boxRect.width / segments) * (i + 0.5);
          spawnBlaster(x - 20, -20, 0, 500, 2, "down", 0.8);
        }

        // Also spawn target bones from both sides, mix of blue/white
        for (let i = 0; i < 10; i++) {
          const fromLeft = i % 2 === 0;
          const y = 20 + (boxRect.height - 40) * Math.random();
          const x = fromLeft ? -30 : boxRect.width + 30;
          const bone = spawnBone(x, y, 20, 10);
          const dirX = storedPlayerPos.x - x;
          const dirY = storedPlayerPos.y - y;
          const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
          const blue = Math.random() < 0.4;
          if (blue) bone.classList.add("blue");
          bones.push({
            el: bone,
            x,
            y,
            vx: (dirX / len) * 2.0,
            vy: (dirY / len) * 2.0,
            blue,
            mode: "target"
          });
        }
      }

      // Finally: huge flood of target bones that slow down and disappear
      if (topBlastSpawned && !finalBoneFloodStarted && elapsed > 4500) {
        finalBoneFloodStarted = true;
        const floodCount = 40;
        for (let i = 0; i < floodCount; i++) {
          const fromLeft = Math.random() < 0.5;
          const y = Math.random() * (boxRect.height - 20);
          const x = fromLeft ? -30 : boxRect.width + 30;
          const bone = spawnBone(x, y, 20, 10);
          const dirX = cx - x;
          const dirY = cy - y;
          const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
          bones.push({
            el: bone,
            x,
            y,
            vx: (dirX / len) * (1.8 + Math.random() * 0.7),
            vy: (dirY / len) * (1.8 + Math.random() * 0.7),
            mode: "flood",
            slowdown: 0.97
          });
        }
      }

      if (elapsed > duration) {
        bones.forEach(b => b.el.remove());
        setSoulColor("red", false);
        exitSoulMode();
        phase2AExhausted = true;
        enemyHP = 1;
        enemyShownFractionHP = 0.0000128;
        updateEnemyHP();
        // He is exhausted now
        showMonsterDialogue("he looks like he's about to\nuse his special attack.");
        onEnd();
        return;
      }

      const soulRect = soul.getBoundingClientRect();
      bones.forEach(b => {
        if (b.mode === "target") {
          b.x += b.vx;
          b.y += b.vy;
          b.el.style.left = b.x + "px";
          b.el.style.top = b.y + "px";
        } else if (b.mode === "flood") {
          b.vx *= b.slowdown;
          b.vy *= b.slowdown;
          b.x += b.vx;
          b.y += b.vy;
          b.el.style.left = b.x + "px";
          b.el.style.top = b.y + "px";
        }
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

    if (phase2AActive && !phase2AExhausted) {
      damageText.textContent = "9999";
      showDialogue("* you strike with everything you have.\n* it doesn't change anything.");
    } else if (phase2AExhausted) {
      // he can actually die or be spared now
      const damage = Math.max(1, Math.floor(hitStrength * 10));
      damageText.textContent = quality + " - " + damage;
      enemyHP = 1; // physically still 1
      enemyShownFractionHP = 0.0000128;
      // Show 0/1 briefly then back
      enemyHPText.textContent = "HP: 0 / 1";
      setTimeout(() => {
        if (phase !== "END") {
          enemyHPText.textContent = "HP: 0.0000128 / 1";
        }
      }, 700);
      showDialogue("* you hit him again.\n* the blow barely seems to register.");
    } else if (!sansCanBeHit) {
      damageText.textContent = quality + " (sans dodged)";
      showDialogue("* \"heh. close.\"\n* \"but not that close.\"");
    } else {
      const damage = Math.max(1, Math.floor(hitStrength * 10));
      enemyHP -= damage;
      if (enemyHP <= 0) enemyHP = 1;
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
        if (phase2AExhausted) {
          // after exhausting spiral, we move into mercy/kill branch
          phase = "PLAYER_TURN";
          showDialogue("* sans is barely standing.\n* you feel like anything could end this.", true);
        } else if (phase2AActive) {
          startSansAttackPhase2A();
        } else if (phase2CActive) {
          startSansAttackPhase2C();
        } else {
          startSansAttack();
        }
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
    showMonsterDialogue("heh.\nthat one went a little deep.\n0.0001 hp left. promise.");
  }

  // ========= PHASE 2A ATTACKS (slashed Sans) =========
  function startSansAttackPhase2A() {
    phase = "PHASE2A_ATTACK";
    phase2ATurns++;

    // more fight variants before final spiral
    enterSoulMode();

    if (phase2ATurns >= phase2ATurnTarget) {
      exitSoulMode();
      showMonsterDialogue("...still standin', huh?\nlet's just end this already.");
      setTimeout(() => {
        spiralGasterFinal(() => {
          // after spiral, we don't end directly; we go into tired state
          phase = "PLAYER_TURN";
        });
      }, 1000);
      return;
    }

    const remainingFactor = Math.max(0, phase2AStartTargets - phase2ATurns);
    const numTargetBones = Math.max(3, remainingFactor);

    const attackVariants = [
      (cb) => phase2ATargetingBonesAndXYBlasters(numTargetBones, cb),
      (cb) => pacifistBottomZoneJump(cb),
      (cb) => pacifistTopBottomAlternating(cb)
    ];
    const attack = attackVariants[Math.floor(Math.random() * attackVariants.length)];
    attack(() => {
      exitSoulMode();
      if (playerHP > 0) {
        showMonsterDialogue("still up?\nnot bad.");
        phase = "PLAYER_TURN";
      }
    });
  }

  // ========= PHASE 2C ATTACKS (Bad Time style) =========
  function startSansAttackPhase2C() {
    phase = "PHASE2C_ATTACK";
    last4AttacksMode = true;
    enterSoulMode();
    const attacks = [
      phase2CBoneFlood,
      phase2CSlidersCross,
      phase2CBoneZoneSpam
    ];
    const attack = attacks[Math.floor(Math.random() * attacks.length)];
    attack(() => {
      exitSoulMode();
      if (playerHP > 0) {
        phase = "PLAYER_TURN";
        showMonsterDialogue("getting tired yet?");
      }
    });
  }

  // ========= PHASE 2B FINAL TEST (REWORKED) =========
  function phase2BFinalTest(onEnd) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;
    const bones = [];
    const duration = 8000;
    const startTime = performance.now();

    // Start: bones from one side (left)
    for (let i = 0; i < 5; i++) {
      const y = 20 + i * 16;
      const boneL = spawnBone(-40, y, 40, 10);
      bones.push({ el: boneL, x: -40, y, dir: 1, speed: 2.2 });
    }

    // 1s later: bones from the other side that are blue for 4 seconds then normal
    setTimeout(() => {
      for (let i = 0; i < 5; i++) {
        const y = 26 + i * 16;
        const boneR = spawnBone(width + 40, y, 40, 10);
        boneR.classList.add("blue");
        bones.push({
          el: boneR,
          x: width + 40,
          y,
          dir: -1,
          speed: 2.2,
          bluePhase: true,
          blueTimer: 240
        });
      }
    }, 1000);

    // Then three target bones from both sides
    setTimeout(() => {
      for (let i = 0; i < 3; i++) {
        const fromLeft = (i % 2 === 0);
        const y = 40 + i * 20;
        const x = fromLeft ? -30 : width + 30;
        const bone = spawnBone(x, y, 20, 10);
        const sx = soulX;
        const sy = soulY;
        const dirX = sx - x;
        const dirY = sy - y;
        const len = Math.max(1, Math.sqrt(dirX * dirX + dirY * dirY));
        bones.push({
          el: bone,
          x,
          y,
          vx: (dirX / len) * 2.0,
          vy: (dirY / len) * 2.0,
          targeting: true
        });
      }
    }, 3000);

    // Finally gaster blaster from top, then end
    setTimeout(() => {
      spawnBlaster(width / 2 - 20, -20, 0, 900, 2, "down");
    }, 4500);

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
        if (b.targeting) {
          b.x += b.vx;
          b.y += b.vy;
          b.el.style.left = b.x + "px";
          b.el.style.top = b.y + "px";
          const rect = b.el.getBoundingClientRect();
          if (rectsOverlap(rect, soulRect)) damagePlayer(1);
        } else if (b.dir != null) {
          b.x += b.speed * b.dir;
          b.el.style.left = b.x + "px";
          const rect = b.el.getBoundingClientRect();
          if (b.bluePhase) {
            b.blueTimer--;
            if (b.blueTimer <= 0) {
              b.bluePhase = false;
              b.el.classList.remove("blue");
            }
            const moving = (keys["w"] || keys["W"] || keys["a"] || keys["A"] || keys["s"] || keys["S"] || keys["d"] || keys["D"]);
            if (moving && rectsOverlap(rect, soulRect)) damagePlayer(1);
          } else {
            if (rectsOverlap(rect, soulRect)) damagePlayer(1);
          }
        }
      });

      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);
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

    // If player only fights, after 5-6 fights, trigger phase2C
    if (!phase2CActive && pureAttackTurns >= 6 && !phase2AActive && !phase2BQueued) {
      phase2CActive = true;
      showMonsterDialogue("ya haven't tried anything else, huh?\nguess i'll have to get\n a bit more serious.");
      return;
    }

    // Blue attack intro
    if (!usedBlueIntro && turnCount === 1) {
      exitSoulMode();
      showMonsterDialogue("i'll let ya have your turn first.\n...then we get to the fun part.");
      setTimeout(() => {
        showMonsterDialogue("what's that look for?\noh right, my blue attack.\nalmost forgot about it.");
        setTimeout(() => {
          showMonsterDialogue("are ya ready?\n'cause here it comes.");
          setTimeout(() => {
            blueAttackSequence(() => {
              showMonsterDialogue("what are you looking so blue for?");
              phase = "PLAYER_TURN";
            });
          }, 600);
        }, 800);
      }, 800);
      usedBlueIntro = true;
      return;
    }

    // Phase 1 special attack that DOES end (before spare)
    if (!usedFinalAttackPhase1 && turnCount >= 5 && !phase2BQueued) {
      usedFinalAttackPhase1 = true;
      exitSoulMode();
      showMonsterDialogue("alright. warm-up's over.\npaps has this 'special attack' thing.\nthink i'll borrow the idea.");
      setTimeout(() => {
        specialAttackPhase1(() => {
          showMonsterDialogue("welp, that sure was fun.\nyou seem ready for waterfall.\nnow just spare me and\nwe can both be on our way.");
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
        showMonsterDialogue("nice moves, kid.");
        phase = "PLAYER_TURN";
      }
    });
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
      if (phase2AExhausted) {
        // Post-final, mercy path: last few target bones then sleep
        showDialogue("* you lower your hands.\n* sans doesn't move.\n* he sighs.");
        closeSubMenu();
        setTimeout(() => {
          enterSoulMode();
          const boxRect = soulBox.getBoundingClientRect();
          const width = boxRect.width;
          const height = boxRect.height;
          const bones = [];
          const startTime = performance.now();
          const duration = 3500;

          for (let i = 0; i < 6; i++) {
            const fromLeft = i % 2 === 0;
            const y = Math.random() * (height - 20);
            const x = fromLeft ? -30 : width + 30;
            const bone = spawnBone(x, y, 20, 10);
            const sx = soulX;
            const sy = soulY;
            const dx = sx - x;
            const dy = sy - y;
            const len = Math.max(1, Math.sqrt(dx * dx + dy * dy));
            bones.push({
              el: bone,
              x,
              y,
              vx: (dx / len) * 2.0,
              vy: (dy / len) * 2.0
            });
          }

          function loop(t) {
            if (!inSoulMode) {
              bones.forEach(b => b.el.remove());
              return;
            }
            if (t - startTime > duration) {
              bones.forEach(b => b.el.remove());
              exitSoulMode();
              showMonsterDialogue("...guess that's it.\nthink i'm gonna take a nap.");
              setTimeout(() => {
                endBattle(true, true, false);
              }, 1200);
              return;
            }
            const soulRect = soul.getBoundingClientRect();
            bones.forEach(b => {
              b.x += b.vx;
              b.y += b.vy;
              b.el.style.left = b.x + "px";
              b.el.style.top = b.y + "px";
              const rect = b.el.getBoundingClientRect();
              if (rectsOverlap(rect, soulRect)) damagePlayer(1);
            });
            requestAnimationFrame(loop);
          }
          requestAnimationFrame(loop);
        }, 1000);
        return;
      }

      if (!canSpare || phase2AActive || phase2CActive) {
        showDialogue("* you reach for MERCY.\n* \"sorry pal, no mercy 'till you prove yourself.\"");
        closeSubMenu();
        setTimeout(() => { if (playerHP > 0) startSansAttack(); }, 1000);
        return;
      }
      // Pacifist Spare -> Phase 2B special
      showDialogue(
        "* you lower your hands.\n" +
        "* sans smiles.\n" +
        "* \"heh. not bad.\"\n" +
        "* \"how 'bout one last test,\nthen we both move on?\""
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

  // ========= ENDING =========
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
      const exp = 120;
      endText.textContent =
        "YOU WON.\nEXP: " + exp + "\nG: " + gold +
        "\n\n* your LOVE increased.\n* the dust settles.\n* jokes feel a lot heavier now.";
    } else {
      endText.textContent = "YOU WON!";
    }
    phase = "END";
  }

  // ========= MENU CONFIRM =========
  function confirmMenuSelection() {
    if (phase !== "PLAYER_TURN") return;
    if (!playerCanActThisTurn) return;
    playerCanActThisTurn = false;

    const action = menuItems[menuIndex];

    if (action === "FIGHT") {
      pureAttackTurns++;
      showDialogue("* you get ready to attack.");
      setTimeout(() => {
        // open target menu first instead of fight bar
        dialogueBox.textContent = "";
        targetMenu.style.display = "block";
        targetMenuActive = true;
        phase = "TARGET_SELECT";
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
        playerCanActThisTurn = false;
      } else {
        openSubMenu("ITEM", items.map(it => ({ id: it.id, label: it.name })));
      }
    } else if (action === "MERCY") {
      const spareLabel = (canSpare && !phase2AActive && !phase2CActive) || phase2AExhausted
        ? "Spare (yellow)"
        : "Spare";
      openSubMenu("MERCY", [
        { id: "SPARE", label: spareLabel },
        { id: "FLEE", label: "Flee" }
      ]);
    }
  }

  function confirmTargetSelection() {
    if (!targetMenuActive) return;
    // only SANS here
    targetMenu.style.display = "none";
    targetMenuActive = false;
    startFightBar();
  }

  // ========= INPUT =========
  document.addEventListener("keydown", (e) => {
    keys[e.key] = true;
    if (phase === "END") return;

    if (e.key === "z" || e.key === "Z") {
      if (phase === "INTRO") {
        phase = "PLAYER_TURN";
        showMonsterDialogue("heya.\nyou're probably wondering\nwhat i'm doing here, huh?\nwell, paps is kinda busy.\nlil' dog stole his\nspecial attack.");
      } else if (phase === "PLAYER_TURN") {
        confirmMenuSelection();
      } else if (phase === "MENU_SUB") {
        confirmSubSelection();
      } else if (phase === "FIGHT_BAR") {
        stopFightBar();
      } else if (phase === "TARGET_SELECT") {
        confirmTargetSelection();
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
    } else if (phase === "TARGET_SELECT") {
      // only one target now, but left/right preserved
      if (e.key === "a" || e.key === "A") setTargetIndex(targetIndex - 1);
      else if (e.key === "d" || e.key === "D") setTargetIndex(targetIndex + 1);
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

  // each time player's turn begins, allow exactly one action
  const originalStartSansAttack = startSansAttack;
  // We already call startSansAttack directly from code; so to ensure
  // each new player turn gets a fresh action, we reset in the flow:
  const playerTurnWatcher = new MutationObserver(() => {
    if (phase === "PLAYER_TURN") {
      playerCanActThisTurn = true;
    }
  });
  playerTurnWatcher.observe(document.body, { attributes: true, childList: true, subtree: true });
</script>
</body>
</html>
