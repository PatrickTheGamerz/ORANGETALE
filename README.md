v<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Undertale‑Style Engine</title>
<style>
body {
  margin: 0;
  background: black;
  color: white;
  font-family: "Courier New", monospace;
  overflow: hidden;
}

#game {
  width: 800px;
  height: 600px;
  border: 4px solid white;
  margin: auto;
  position: relative;
  background: black;
}

/* Player */
#player {
  width: 24px;
  height: 24px;
  background: yellow;
  position: absolute;
  left: 380px;
  top: 280px;
}

/* Dialogue box */
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
}

/* Fade */
#fade {
  position: absolute;
  inset: 0;
  background: black;
  opacity: 0;
  pointer-events: none;
}
</style>
</head>
<body>

<div id="game">
  <div id="player"></div>
  <div id="dialogue-box"></div>
  <div id="fade"></div>
</div>

<script>
/* ============================
   GAME STATE
============================ */
let mode = "overworld"; // overworld, dialogue, battle
let keys = {};
let player = document.getElementById("player");
let dialogueBox = document.getElementById("dialogue-box");
let fade = document.getElementById("fade");

let px = 380, py = 280;
let speed = 2.5;

let encounterChance = 0;
let encounterCooldown = 0;

/* ============================
   ROOM DATA
============================ */
let currentRoom = "ruins_entrance";

const rooms = {
  "ruins_entrance": {
    name: "Ruins Entrance",
    walls: [
      {x:0,y:0,w:800,h:40},
      {x:0,y:0,w:40,h:600},
      {x:760,y:0,w:40,h:600},
      {x:0,y:560,w:800,h:40}
    ],
    dialogueTriggers: [
      {
        x: 350, y: 200, w: 100, h: 40,
        text: [
          "* The ancient door stands silent.",
          "* A faint breeze slips through the cracks."
        ],
        once: true
      }
    ]
  }
};

/* ============================
   INPUT
============================ */
document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

/* ============================
   MOVEMENT + COLLISION
============================ */
function movePlayer() {
  if (mode !== "overworld") return;

  let dx = 0, dy = 0;
  if (keys["w"] || keys["W"]) dy -= speed;
  if (keys["s"] || keys["S"]) dy += speed;
  if (keys["a"] || keys["A"]) dx -= speed;
  if (keys["d"] || keys["D"]) dx += speed;

  let newX = px + dx;
  let newY = py + dy;

  if (!collides(newX, py)) px = newX;
  if (!collides(px, newY)) py = newY;

  player.style.left = px + "px";
  player.style.top = py + "px";

  handleDialogueTriggers();
  handleRandomEncounters();
}

function collides(x, y) {
  let room = rooms[currentRoom];
  let p = {x:x, y:y, w:24, h:24};

  for (let w of room.walls) {
    if (rectOverlap(p, w)) return true;
  }
  return false;
}

function rectOverlap(a, b) {
  return !(
    a.x + a.w < b.x ||
    a.x > b.x + b.w ||
    a.y + a.h < b.y ||
    a.y > b.y + b.h
  );
}

/* ============================
   DIALOGUE SYSTEM
============================ */
let dialogueQueue = [];
let dialogueIndex = 0;

function handleDialogueTriggers() {
  let room = rooms[currentRoom];
  let p = {x:px, y:py, w:24, h:24};

  for (let trig of room.dialogueTriggers) {
    if (!trig.used && rectOverlap(p, trig)) {
      startDialogue(trig.text);
      trig.used = trig.once;
    }
  }
}

function startDialogue(lines) {
  mode = "dialogue";
  dialogueQueue = lines;
  dialogueIndex = 0;
  dialogueBox.style.display = "block";
  dialogueBox.textContent = lines[0];
}

document.addEventListener("keydown", e => {
  if (mode === "dialogue" && (e.key === "z" || e.key === "Z")) {
    dialogueIndex++;
    if (dialogueIndex >= dialogueQueue.length) {
      dialogueBox.style.display = "none";
      mode = "overworld";
    } else {
      dialogueBox.textContent = dialogueQueue[dialogueIndex];
    }
  }
});

/* ============================
   RANDOM ENCOUNTERS
============================ */
function handleRandomEncounters() {
  if (encounterCooldown > 0) {
    encounterCooldown--;
    return;
  }

  if (keys["w"] || keys["a"] || keys["s"] || keys["d"]) {
    encounterChance += 0.4;
  }

  if (Math.random() * 100 < encounterChance) {
    startBattle();
  }
}

function startBattle() {
  mode = "battle";
  fadeToBlack(() => {
    window.location.href = "sans_test.html"; // load your battle file
  });
}

function fadeToBlack(callback) {
  let opacity = 0;
  let interval = setInterval(() => {
    opacity += 0.05;
    fade.style.opacity = opacity;
    if (opacity >= 1) {
      clearInterval(interval);
      callback();
    }
  }, 30);
}

/* ============================
   GAME LOOP
============================ */
function loop() {
  movePlayer();
  requestAnimationFrame(loop);
}
loop();
</script>

</body>
</html>
