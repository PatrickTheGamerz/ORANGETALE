<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Undertale-style Overworld + Battle</title>
<style>
  body {
    margin: 0;
    background: black;
    color: white;
    font-family: "Courier New", monospace;
    overflow: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #game {
    width: 800px;
    height: 600px;
    border: 4px solid white;
    position: relative;
    background: #1b1020; /* dark purple ruins tone */
    overflow: hidden;
  }

  /* Overworld view */
  #overworld {
    position: absolute;
    inset: 0;
    background: linear-gradient(#2b1633, #1b1020);
    display: block;
  }

  .room-decor {
    position: absolute;
    background: #3a2145;
  }

  .room-floor {
    position: absolute;
    inset: 60px 60px 60px 60px;
    background: #2a1733;
    border: 2px solid #623b79;
    box-shadow: 0 0 0 3px #150916 inset;
  }

  .room-wall {
    position: absolute;
    background: #3a2145;
  }

  .room-door {
    position: absolute;
    background: #d2a35b;
  }

  #player {
    width: 24px;
    height: 24px;
    background: #f9dc5c;
    position: absolute;
  }
  #player::after {
    content: "";
    position: absolute;
    left: 4px;
    bottom: -4px;
    width: 16px;
    height: 6px;
    background: rgba(0,0,0,0.3);
    border-radius: 50%;
  }

  #dialogue-box {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 160px;
    border-top: 4px solid white;
    padding: 10px;
    box-sizing: border-box;
    display: none;
    white-space: pre-line;
    font-size: 18px;
    background: rgba(0,0,0,0.8);
  }

  #fade {
    position: absolute;
    inset: 0;
    background: black;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.35s;
  }

  /* Battle view */
  #battle {
    position: absolute;
    inset: 0;
    background: black;
    display: none;
  }

  #battle-box {
    position: absolute;
    inset: 60px 60px 160px 60px;
    border: 4px solid white;
  }

  #battle-enemy {
    position: absolute;
    top: 80px;
    left: 50%;
    transform: translateX(-50%);
    width: 100px;
    height: 100px;
    border: 2px solid white;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 12px;
  }

  #battle-dialogue {
    position: absolute;
    left: 60px;
    right: 60px;
    bottom: 160px;
    height: 100px;
    border: 4px solid white;
    box-sizing: border-box;
    padding: 8px;
    font-size: 18px;
    white-space: pre-line;
  }

  #battle-ui {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    height: 160px;
    border-top: 4px solid white;
    box-sizing: border-box;
    padding: 10px 20px;
  }

  #battle-menu {
    display: flex;
    gap: 20px;
    font-size: 22px;
  }

  .battle-menu-item {
    cursor: pointer;
  }
  .battle-menu-item.selected {
    color: yellow;
  }

  #battle-hp {
    margin-top: 20px;
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 16px;
  }
  #battle-hp-bar {
    position: relative;
    width: 200px;
    height: 16px;
    border: 2px solid white;
  }
  #battle-hp-fill {
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    background: #ffb000;
    width: 100%;
  }

  #battle-submenu {
    position: absolute;
    bottom: 160px;
    left: 0;
    right: 0;
    height: 80px;
    display: none;
    justify-content: center;
    align-items: center;
    gap: 20px;
    font-size: 18px;
  }
  .battle-sub-item {
    border: 2px solid white;
    padding: 4px 10px;
  }
  .battle-sub-item.selected {
    color: yellow;
    border-color: yellow;
  }

  #soul-box {
    position: absolute;
    left: 260px;
    right: 260px;
    bottom: 180px;
    height: 120px;
    border: 4px solid white;
    display: none;
    overflow: hidden;
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
</style>
</head>
<body>
<div id="game">
  <div id="overworld">
    <div class="room-floor" id="room-floor"></div>
    <!-- walls, doors, decor will be generated dynamically -->
    <div id="player"></div>
    <div id="dialogue-box"></div>
    <div id="fade"></div>
  </div>

  <div id="battle">
    <div id="battle-box"></div>
    <div id="battle-enemy">DUMMY</div>
    <div id="battle-dialogue">* A training dummy appears.</div>
    <div id="soul-box">
      <div id="soul"></div>
    </div>
    <div id="battle-submenu"></div>
    <div id="battle-ui">
      <div id="battle-menu">
        <span class="battle-menu-item selected" data-action="FIGHT">FIGHT</span>
        <span class="battle-menu-item" data-action="ACT">ACT</span>
        <span class="battle-menu-item" data-action="ITEM">ITEM</span>
        <span class="battle-menu-item" data-action="MERCY">MERCY</span>
      </div>
      <div id="battle-hp">
        <span>HP</span>
        <div id="battle-hp-bar">
          <div id="battle-hp-fill"></div>
        </div>
        <span id="battle-hp-text">20 / 20</span>
      </div>
    </div>
  </div>
</div>

<script>
/* ========= OVERWORLD CORE ========= */
let mode = "overworld"; // overworld, dialogue, battle
let keys = {};
let player = document.getElementById("player");
let dialogueBox = document.getElementById("dialogue-box");
let fade = document.getElementById("fade");

let px = 380, py = 280;
let speed = 2.4;

let encounterMeter = 0;
let encounterCooldown = 0;

let currentRoomId = "ruins_entrance";
const roomFloor = document.getElementById("room-floor");
const overworld = document.getElementById("overworld");

const rooms = {
  ruins_entrance: {
    name: "Ruins Entrance",
    background: "#2a1733",
    walls: [
      {x:60,y:60,w:680,h:20},    // top
      {x:60,y:520,w:680,h:20},   // bottom
      {x:60,y:60,w:20,h:480},    // left
      {x:720,y:60,w:20,h:480}    // right
    ],
    doors: [
      {
        x:380,y:70,w:40,h:20,
        to: "ruins_hall",
        spawnX: 380, spawnY: 480
      }
    ],
    decor: [
      {x:120,y:120,w:80,h:20},
      {x:600,y:160,w:40,h:60},
      {x:180,y:440,w:40,h:80}
    ],
    dialogueTriggers: [
      {
        x: 330, y: 260, w: 140, h: 40,
        text: [
          "* The stone door feels old and heavy.",
          "* You can almost hear someone humming on the other side."
        ],
        once: true
      }
    ]
  },
  ruins_hall: {
    name: "Ruins Hall",
    background: "#261430",
    walls: [
      {x:60,y:60,w:680,h:20},
      {x:60,y:520,w:680,h:20},
      {x:60,y:60,w:20,h:480},
      {x:720,y:60,w:20,h:480}
    ],
    doors: [
      {
        x:380,y:520,w:40,h:20,
        to: "ruins_entrance",
        spawnX: 380, spawnY: 100
      },
      {
        x:720,y:260,w:20,h:40,
        to: "ruins_crossroads",
        spawnX: 100, spawnY: 260
      }
    ],
    decor: [
      {x:150,y:140,w:40,h:80},
      {x:620,y:400,w:60,h:20}
    ],
    dialogueTriggers: [
      {
        x: 200, y: 260, w: 80, h: 40,
        text: [
          "* A cracked pillar leans against the wall.",
          "* It looks like it has been fixed more than once."
        ],
        once: true
      }
    ]
  },
  ruins_crossroads: {
    name: "Ruins Crossroads",
    background: "#23112c",
    walls: [
      {x:60,y:60,w:680,h:20},
      {x:60,y:520,w:680,h:20},
      {x:60,y:60,w:20,h:480},
      {x:720,y:60,w:20,h:480}
    ],
    doors: [
      {
        x:60,y:260,w:20,h:40,
        to: "ruins_hall",
        spawnX: 700, spawnY: 260
      }
    ],
    decor: [
      {x:280,y:140,w:80,h:20},
      {x:520,y:160,w:40,h:80},
      {x:400,y:400,w:60,h:40}
    ],
    dialogueTriggers: [
      {
        x: 360, y: 260, w: 80, h: 40,
        text: [
          "* Three paths stretch out in front of you.",
          "* Only one leads forward.",
          "* The others lead to more training... or more trouble."
        ],
        once: false
      }
    ]
  }
};

function createRoomVisuals(roomId) {
  const room = rooms[roomId];
  roomFloor.style.background = room.background;

  // Remove old walls, doors, decor
  overworld.querySelectorAll(".room-wall, .room-door, .room-decor")
    .forEach(el => el.remove());

  // Walls
  room.walls.forEach(w => {
    const el = document.createElement("div");
    el.className = "room-wall";
    el.style.left = w.x + "px";
    el.style.top = w.y + "px";
    el.style.width = w.w + "px";
    el.style.height = w.h + "px";
    el.style.background = "#3b2345";
    overworld.appendChild(el);
  });

  // Doors
  room.doors.forEach(d => {
    const el = document.createElement("div");
    el.className = "room-door";
    el.style.left = d.x + "px";
    el.style.top = d.y + "px";
    el.style.width = d.w + "px";
    el.style.height = d.h + "px";
    el.style.background = "#d9a65e";
    overworld.appendChild(el);
  });

  // Decor
  room.decor.forEach(d => {
    const el = document.createElement("div");
    el.className = "room-decor";
    el.style.left = d.x + "px";
    el.style.top = d.y + "px";
    el.style.width = d.w + "px";
    el.style.height = d.h + "px";
    el.style.background = "#4a2c59";
    overworld.appendChild(el);
  });
}

/* ===== Input ===== */
document.addEventListener("keydown", e => {
  keys[e.key] = true;

  if (mode === "dialogue" && (e.key === "z" || e.key === "Z")) {
    advanceDialogue();
  }

  if (mode === "battle") handleBattleKeyDown(e);
});

document.addEventListener("keyup", e => {
  keys[e.key] = false;
});

/* ===== Overworld movement and collision ===== */
function rectOverlap(a, b) {
  return !(
    a.x + a.w < b.x ||
    a.x > b.x + b.w ||
    a.y + a.h < b.y ||
    a.y > b.y + b.h
  );
}

function collidesWithWalls(x, y) {
  const room = rooms[currentRoomId];
  const p = {x, y, w:24, h:24};
  for (let w of room.walls) {
    if (rectOverlap(p, w)) return true;
  }
  return false;
}

function checkDoors(x, y) {
  const room = rooms[currentRoomId];
  const p = {x, y, w:24, h:24};
  for (let d of room.doors) {
    const r = {x:d.x, y:d.y, w:d.w, h:d.h};
    if (rectOverlap(p, r)) return d;
  }
  return null;
}

function handleDialogueTriggersOverworld() {
  const room = rooms[currentRoomId];
  const p = {x:px, y:py, w:24, h:24};
  if (!room.dialogueTriggers) return;
  for (let trig of room.dialogueTriggers) {
    if (trig.once && trig.used) continue;
    const r = {x:trig.x, y:trig.y, w:trig.w, h:trig.h};
    if (rectOverlap(p, r)) {
      startDialogue(trig.text, () => {
        mode = "overworld";
      });
      trig.used = true;
      break;
    }
  }
}

function movePlayerOverworld() {
  if (mode !== "overworld") return;

  let dx = 0, dy = 0;
  if (keys["w"] || keys["W"]) dy -= speed;
  if (keys["s"] || keys["S"]) dy += speed;
  if (keys["a"] || keys["A"]) dx -= speed;
  if (keys["d"] || keys["D"]) dx += speed;

  let newX = px + dx;
  let newY = py + dy;

  if (!collidesWithWalls(newX, py)) px = newX;
  if (!collidesWithWalls(px, newY)) py = newY;

  player.style.left = px + "px";
  player.style.top = py + "px";

  const door = checkDoors(px, py);
  if (door) changeRoom(door.to, door.spawnX, door.spawnY);

  handleDialogueTriggersOverworld();
  handleRandomEncounters();
}

/* ===== Room changing ===== */
function changeRoom(roomId, spawnX, spawnY) {
  currentRoomId = roomId;
  createRoomVisuals(roomId);
  px = spawnX;
  py = spawnY;
  player.style.left = px + "px";
  player.style.top = py + "px";
  encounterMeter = 0;
}

/* ===== Dialogue engine ===== */
let dialogueLines = [];
let dialogueIndex = 0;
let dialogueCallback = null;

function startDialogue(lines, callback) {
  mode = "dialogue";
  dialogueLines = lines;
  dialogueIndex = 0;
  dialogueCallback = callback || null;
  dialogueBox.style.display = "block";
  dialogueBox.textContent = dialogueLines[0];
}

function advanceDialogue() {
  dialogueIndex++;
  if (dialogueIndex >= dialogueLines.length) {
    dialogueBox.style.display = "none";
    dialogueLines = [];
    if (dialogueCallback) dialogueCallback();
    else mode = "overworld";
  } else {
    dialogueBox.textContent = dialogueLines[dialogueIndex];
  }
}

/* ===== Random encounters ===== */
function handleRandomEncounters() {
  if (encounterCooldown > 0) {
    encounterCooldown--;
    return;
  }
  if (keys["w"] || keys["a"] || keys["s"] || keys["d"]) {
    encounterMeter += 0.35;
  }
  if (Math.random() * 100 < encounterMeter) {
    encounterMeter = 0;
    encounterCooldown = 180; // ~3s
    startBattle();
  }
}

/* ===== Simple fade ===== */
function fadeToBlack(callback) {
  fade.style.transition = "opacity 0.35s";
  fade.style.opacity = 1;
  setTimeout(() => {
    if (callback) callback();
  }, 350);
}
function fadeFromBlack() {
  fade.style.opacity = 0;
}

/* ========= BATTLE SYSTEM (SIMPLE STUB) ========= */
const battleView = document.getElementById("battle");
const battleDialogue = document.getElementById("battle-dialogue");
const battleHPFill = document.getElementById("battle-hp-fill");
const battleHPText = document.getElementById("battle-hp-text");
const battleMenuItems = Array.from(document.querySelectorAll(".battle-menu-item"));
const battleSubmenu = document.getElementById("battle-submenu");
const soulBox = document.getElementById("soul-box");
const soul = document.getElementById("soul");

let battlePlayerHP = 20;
const battlePlayerMaxHP = 20;
let battleMenuIndex = 0;
let battlePhase = "PLAYER_MENU"; // PLAYER_MENU, ENEMY_ATTACK, PLAYER_ACT
let inSoulMode = false;
let canMoveSoul = false;
let soulX = 0, soulY = 0;
let soulSpeed = 3;
let battleSubOptions = [];
let battleSubIndex = 0;

function updateBattleHP() {
  if (battlePlayerHP < 0) battlePlayerHP = 0;
  const ratio = battlePlayerHP / battlePlayerMaxHP;
  battleHPFill.style.width = (ratio * 100) + "%";
  battleHPText.textContent = battlePlayerHP + " / " + battlePlayerMaxHP;
}

function startBattle() {
  mode = "transition";
  fadeToBlack(() => {
    overworld.style.display = "none";
    battleView.style.display = "block";
    fadeFromBlack();
    mode = "battle";
    battlePhase = "PLAYER_MENU";
    battleDialogue.textContent = "* The second skeleton brother stands firm.\n* (But for now, let's practice on a dummy.)";
    battlePlayerHP = 20;
    updateBattleHP();
    battleMenuIndex = 0;
    setBattleMenuIndex(0);
  });
}

function endBattle() {
  mode = "transition";
  fadeToBlack(() => {
    battleView.style.display = "none";
    overworld.style.display = "block";
    fadeFromBlack();
    mode = "overworld";
    encounterCooldown = 180;
  });
}

function setBattleMenuIndex(i) {
  battleMenuIndex = (i + battleMenuItems.length) % battleMenuItems.length;
  battleMenuItems.forEach((el, idx) => {
    el.classList.toggle("selected", idx === battleMenuIndex);
  });
}

function openBattleSubmenu(type) {
  battleSubOptions = [];
  battleSubIndex = 0;
  battleSubmenu.innerHTML = "";
  battleSubmenu.style.display = "flex";

  if (type === "ACT") {
    battleSubOptions = [
      {id:"CHECK", label:"Check"},
      {id:"JOKE", label:"Joke"},
      {id:"FLIRT", label:"Flirt"},
      {id:"TALK", label:"Talk"}
    ];
  } else if (type === "ITEM") {
    battleSubOptions = [
      {id:"HEAL", label:"Heal"}
    ];
  } else if (type === "MERCY") {
    battleSubOptions = [
      {id:"SPARE", label:"Spare"},
      {id:"FLEE", label:"Flee"}
    ];
  }

  battleSubOptions.forEach((opt, idx) => {
    const el = document.createElement("span");
    el.className = "battle-sub-item" + (idx === 0 ? " selected" : "");
    el.textContent = opt.label;
    battleSubmenu.appendChild(el);
  });

  battlePhase = "PLAYER_ACT";
}

function setBattleSubIndex(i) {
  if (!battleSubOptions.length) return;
  battleSubIndex = (i + battleSubOptions.length) % battleSubOptions.length;
  const els = Array.from(battleSubmenu.querySelectorAll(".battle-sub-item"));
  els.forEach((el, idx) => {
    el.classList.toggle("selected", idx === battleSubIndex);
  });
}

function closeBattleSubmenu() {
  battleSubmenu.style.display = "none";
  battleSubmenu.innerHTML = "";
  battleSubOptions = [];
  battlePhase = "PLAYER_MENU";
}

function handleBattleKeyDown(e) {
  if (battlePhase === "PLAYER_MENU") {
    if (e.key === "a" || e.key === "A") setBattleMenuIndex(battleMenuIndex - 1);
    if (e.key === "d" || e.key === "D") setBattleMenuIndex(battleMenuIndex + 1);
    if (e.key === "z" || e.key === "Z") confirmBattleMenu();
  } else if (battlePhase === "PLAYER_ACT") {
    if (e.key === "a" || e.key === "A") setBattleSubIndex(battleSubIndex - 1);
    if (e.key === "d" || e.key === "D") setBattleSubIndex(battleSubIndex + 1);
    if (e.key === "z" || e.key === "Z") confirmBattleSub();
    if (e.key === "x" || e.key === "X") closeBattleSubmenu();
  }
}

function confirmBattleMenu() {
  const action = battleMenuItems[battleMenuIndex].dataset.action;
  if (action === "FIGHT") {
    battleDialogue.textContent = "* You swing at the dummy.\n* It wobbles cheerfully.";
    setTimeout(() => enemyAttack(), 600);
  } else if (action === "ACT") {
    openBattleSubmenu("ACT");
  } else if (action === "ITEM") {
    openBattleSubmenu("ITEM");
  } else if (action === "MERCY") {
    openBattleSubmenu("MERCY");
  }
}

function confirmBattleSub() {
  const opt = battleSubOptions[battleSubIndex];
  if (!opt) return;
  if (opt.id === "CHECK") {
    battleDialogue.textContent = "* Dummy - ATK 0 DEF 0.\n* It's just here for practice.";
  } else if (opt.id === "JOKE") {
    battleDialogue.textContent = "* You tell a joke.\n* The dummy cannot laugh, but you feel better.";
  } else if (opt.id === "FLIRT") {
    battleDialogue.textContent = "* You flirt with the dummy.\n* Nothing happens.";
  } else if (opt.id === "TALK") {
    battleDialogue.textContent = "* You talk to the dummy.\n* It doesn't respond.";
  } else if (opt.id === "HEAL") {
    battlePlayerHP = Math.min(battlePlayerMaxHP, battlePlayerHP + 10);
    updateBattleHP();
    battleDialogue.textContent = "* You eat a candy.\n* You recovered 10 HP.";
  } else if (opt.id === "SPARE") {
    battleDialogue.textContent = "* You spare the dummy.\n* It seems satisfied.";
    setTimeout(() => endBattle(), 1200);
    closeBattleSubmenu();
    return;
  } else if (opt.id === "FLEE") {
    battleDialogue.textContent = "* You run away...";
    setTimeout(() => endBattle(), 800);
    closeBattleSubmenu();
    return;
  }
  closeBattleSubmenu();
  setTimeout(() => enemyAttack(), 600);
}

/* ===== Simple enemy attack with bones ===== */
function enemyAttack() {
  battlePhase = "ENEMY_ATTACK";
  battleDialogue.textContent = "* The dummy shuffles in place.";

  enterSoulMode();
  dummyBoneAttack(() => {
    exitSoulMode();
    battlePhase = "PLAYER_MENU";
    battleDialogue.textContent = "* The second skeleton brother stands firm.";
  });
}

function enterSoulMode() {
  inSoulMode = true;
  canMoveSoul = true;
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
  soulBox.querySelectorAll(".bone").forEach(b => b.remove());
}

function dummyBoneAttack(onEnd) {
  const boxRect = soulBox.getBoundingClientRect();
  const width = boxRect.width;
  const height = boxRect.height;
  const bones = [];

  for (let i = 0; i < 5; i++) {
    const bone = document.createElement("div");
    bone.className = "bone";
    bone.style.width = "40px";
    bone.style.height = "10px";
    bone.style.top = (20 + i * 15) + "px";
    bone.style.left = "-40px";
    soulBox.appendChild(bone);
    bones.push({el: bone, x: -40, y: 20 + i * 15});
  }

  const duration = 3500;
  const startTime = performance.now();

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
      b.x += 2;
      b.el.style.left = b.x + "px";
      const rect = b.el.getBoundingClientRect();
      if (rectOverlap(rect, soulRect)) {
        battlePlayerHP -= 1;
        updateBattleHP();
      }
    });

    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);
}

/* ===== Soul movement in battle ===== */
function soulLoop() {
  if (inSoulMode && canMoveSoul) {
    const boxRect = soulBox.getBoundingClientRect();
    const width = boxRect.width;
    const height = boxRect.height;

    if (keys["w"] || keys["W"]) soulY -= soulSpeed;
    if (keys["s"] || keys["S"]) soulY += soulSpeed;
    if (keys["a"] || keys["A"]) soulX -= soulSpeed;
    if (keys["d"] || keys["D"]) soulX += soulSpeed;

    if (soulX < 0) soulX = 0;
    if (soulY < 0) soulY = 0;
    if (soulX > width - 16) soulX = width - 16;
    if (soulY > height - 16) soulY = height - 16;

    soul.style.left = soulX + "px";
    soul.style.top = soulY + "px";
  }
  requestAnimationFrame(soulLoop);
}

/* ===== Main loop ===== */
function mainLoop() {
  if (mode === "overworld") movePlayerOverworld();
  requestAnimationFrame(mainLoop);
}

/* Init */
createRoomVisuals(currentRoomId);
player.style.left = px + "px";
player.style.top = py + "px";
updateBattleHP();
mainLoop();
soulLoop();
</script>
</body>
</html>
