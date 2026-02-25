<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Sans - Last Breath</title>
<style>
body { margin: 0; background: black; color: white; font-family: "Courier New", monospace; display: flex; justify-content: center; align-items: center; height: 100vh; overflow: hidden; font-weight: bold; }
#game { width: 800px; height: 600px; border: 4px solid white; position: relative; background: black; overflow: hidden; }
#enemy-area { position: absolute; top: 20px; left: 0; width: 100%; height: 180px; display: flex; flex-direction: column; align-items: center; pointer-events: none; }
#enemy-sprite { width: 100px; height: 100px; background-size: contain; background-repeat: no-repeat; animation: idle 2s infinite alternate ease-in-out; }
.phase1 { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 30"><rect x="11" y="2" width="8" height="2" fill="white"/><rect x="9" y="4" width="2" height="4" fill="white"/><rect x="19" y="4" width="2" height="4" fill="white"/><rect x="7" y="8" width="2" height="6" fill="white"/><rect x="21" y="8" width="2" height="6" fill="white"/><rect x="9" y="14" width="2" height="2" fill="white"/><rect x="19" y="14" width="2" height="2" fill="white"/><rect x="11" y="16" width="8" height="2" fill="white"/><rect x="11" y="8" width="3" height="3" fill="black"/><rect x="16" y="8" width="3" height="3" fill="black"/><rect x="13" y="12" width="4" height="2" fill="black"/><rect x="10" y="18" width="10" height="4" fill="white"/><rect x="8" y="20" width="2" height="6" fill="white"/><rect x="20" y="20" width="2" height="6" fill="white"/></svg>'); }
.phase2 { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 30"><rect x="11" y="2" width="8" height="2" fill="white"/><rect x="9" y="4" width="2" height="4" fill="white"/><rect x="19" y="4" width="2" height="4" fill="white"/><rect x="7" y="8" width="2" height="6" fill="white"/><rect x="21" y="8" width="2" height="6" fill="white"/><rect x="9" y="14" width="2" height="2" fill="white"/><rect x="19" y="14" width="2" height="2" fill="white"/><rect x="11" y="16" width="8" height="2" fill="white"/><rect x="11" y="8" width="3" height="3" fill="black"/><rect x="16" y="8" width="4" height="4" fill="red"/><rect x="13" y="12" width="4" height="2" fill="black"/><rect x="10" y="18" width="10" height="4" fill="white"/><rect x="8" y="20" width="2" height="6" fill="white"/><rect x="20" y="20" width="2" height="6" fill="white"/><path d="M15,2 L14,6 L16,8 L15,12" stroke="black" stroke-width="1" fill="none"/></svg>'); animation: intense 0.5s infinite alternate ease-in-out !important;}
.phase3 { background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 30"><rect x="11" y="2" width="4" height="2" fill="white"/><rect x="9" y="4" width="2" height="4" fill="white"/><rect x="7" y="8" width="2" height="6" fill="white"/><rect x="21" y="8" width="2" height="6" fill="white"/><rect x="9" y="14" width="2" height="2" fill="white"/><rect x="19" y="14" width="2" height="2" fill="white"/><rect x="11" y="16" width="8" height="2" fill="white"/><rect x="11" y="8" width="3" height="3" fill="black"/><rect x="16" y="8" width="4" height="4" fill="red"/><rect x="13" y="12" width="4" height="2" fill="black"/><rect x="10" y="18" width="10" height="4" fill="white"/><rect x="8" y="20" width="2" height="6" fill="white"/><rect x="20" y="20" width="2" height="6" fill="white"/><path d="M15,2 L14,6 L16,8 L15,12 L18,10" stroke="black" stroke-width="1" fill="none"/></svg>'); filter: drop-shadow(0 0 10px red); animation: glitch 0.1s infinite !important;}
@keyframes idle { 0% { transform: translateY(0px); } 100% { transform: translateY(6px); } }
@keyframes intense { 0% { transform: translate(0px, 0px); } 100% { transform: translate(2px, 5px); } }
@keyframes glitch { 0% { transform: translate(-2px, 1px); } 50% { transform: translate(2px, -1px); } 100% { transform: translate(-1px, 2px); } }
#dialogue-box { position: absolute; bottom: 150px; left: 20px; right: 20px; height: 140px; border: 4px solid white; padding: 15px; box-sizing: border-box; font-size: 24px; white-space: pre-line; background: black; display: none; z-index: 5;}
#sans-dialogue-box { position: absolute; top: 30px; left: 450px; width: 280px; min-height: 40px; border: 4px solid black; border-radius: 10px; padding: 12px; background: white; color: black; font-size: 18px; white-space: pre-line; display: none; z-index: 10; font-weight: normal; }
#soul-box { position: absolute; bottom: 150px; left: 220px; width: 360px; height: 180px; border: 4px solid white; box-sizing: border-box; overflow: hidden; display: none; background: black; }
#soul { width: 16px; height: 16px; background: red; position: absolute; z-index: 20; box-shadow: 0 0 5px red;}
.soul-hurt { animation: blink 0.2s infinite; }
@keyframes blink { 0% { opacity: 1; } 50% { opacity: 0.2; } 100% { opacity: 1; } }
.bone { position: absolute; background: white; border-radius: 6px; box-shadow: 0 0 4px white; }
.bone-blue { background: #00a2e8; box-shadow: 0 0 6px #00a2e8; }
.bone-orange { background: #ff7f27; box-shadow: 0 0 6px #ff7f27; }
.blaster { position: absolute; width: 60px; height: 60px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 40 40"><path d="M10,5 L30,5 L35,15 L35,25 L25,35 L15,35 L5,25 L5,15 Z" fill="white"/><circle cx="15" cy="15" r="3" fill="black"/><circle cx="25" cy="15" r="3" fill="black"/><rect x="15" y="25" width="10" height="2" fill="black"/></svg>'); background-size: contain; background-repeat: no-repeat; z-index: 15;}
.blaster-red { filter: drop-shadow(0 0 5px red); }
.blast-beam { position: absolute; background: white; border: 3px solid cyan; box-shadow: 0 0 15px cyan; z-index: 14;}
.blast-beam-red { border: 3px solid red; box-shadow: 0 0 15px red; }
#ui-bar { position: absolute; bottom: 0; left: 0; width: 100%; height: 130px; display: flex; flex-direction: column; justify-content: space-between; padding: 10px 30px; box-sizing: border-box; background: black; }
#stats-row { display: flex; align-items: center; gap: 20px; font-size: 24px; margin-bottom: 15px; }
#hp-bar-inner { width: 100px; height: 20px; position: relative; background: red; }
#hp-fill { position: absolute; top: 0; left: 0; height: 100%; background: #ffb000; transition: width 0.2s ease-out; }
#ui-menu { display: flex; justify-content: space-between; font-size: 24px; color: #ff8000; width: 90%; margin: 0 auto; }
.menu-item { cursor: pointer; padding: 5px 15px; border: 2px solid transparent;}
.menu-item.selected { color: yellow; border: 2px solid yellow; }
#sub-menu { position: absolute; bottom: 150px; left: 30px; width: 700px; height: 140px; display: none; flex-wrap: wrap; align-content: flex-start; gap: 20px; font-size: 24px; padding: 20px; box-sizing: border-box; z-index: 6;}
.sub-option { width: 45%; cursor: pointer; color: white;}
.sub-option::before { content: '* '; }
.sub-option.selected { color: yellow; }
#fight-bar-container { position: absolute; bottom: 150px; left: 220px; width: 360px; height: 180px; display: none; flex-direction: column; align-items: center; justify-content: center; background: black; border: 4px solid white; box-sizing: border-box; z-index: 7;}
#fight-bar { width: 320px; height: 20px; border: 2px solid white; position: relative; background: black; }
#fight-zone { position: absolute; top: 0; height: 100%; width: 40px; background: yellow; left: 140px; opacity: 0.6; }
#fight-pointer { position: absolute; top: 0; width: 4px; height: 100%; background: white; box-shadow: 0 0 6px white; }
#damage-text { font-size: 30px; margin-top: 20px; color: red; height: 35px; }
#end-screen { position: absolute; inset: 0; background: black; color: white; display: none; justify-content: center; align-items: center; flex-direction: column; font-size: 24px; text-align: center; white-space: pre-line; z-index: 100;}
</style>
</head>
<body>
<div id="game">
<div id="enemy-area"><div id="enemy-sprite" class="phase1"></div></div>
<div id="dialogue-box"></div>
<div id="sans-dialogue-box"></div>
<div id="soul-box"><div id="soul"></div></div>
<div id="fight-bar-container"><div id="fight-bar"><div id="fight-zone"></div><div id="fight-pointer"></div></div><div id="damage-text"></div></div>
<div id="sub-menu"></div>
<div id="ui-bar">
<div id="stats-row">
<span>FRISK</span><span>LV 19</span>
<div style="display: flex; align-items: center; gap: 10px;">
<span>HP</span><div id="hp-bar-inner"><div id="hp-fill"></div></div><span id="hp-text">92 / 92</span>
</div>
</div>
<div id="ui-menu">
<span class="menu-item selected">FIGHT</span><span class="menu-item">ACT</span><span class="menu-item">ITEM</span><span class="menu-item">MERCY</span>
</div>
</div>
<div id="end-screen"><div id="end-text"></div></div>
</div>
<script>
let playerHP = 92, playerMaxHP = 92, invincFrames = 0;
let enemyHP = 40;
let fightPhase = 1;
let phase = "INTRO"; 
let turnCount = 0;
const gameEl = document.getElementById("game");
const dialogueBox = document.getElementById("dialogue-box");
const sansDialogueBox = document.getElementById("sans-dialogue-box");
const soulBox = document.getElementById("soul-box");
const soul = document.getElementById("soul");
const subMenu = document.getElementById("sub-menu");
const hpFill = document.getElementById("hp-fill");
const hpText = document.getElementById("hp-text");
const enemySprite = document.getElementById("enemy-sprite");
const fightBarContainer = document.getElementById("fight-bar-container");
const fightPointer = document.getElementById("fight-pointer");
const damageText = document.getElementById("damage-text");
const menuItems = ["FIGHT", "ACT", "ITEM", "MERCY"];
let menuIndex = 0, subIndex = 0, currentSubOptions = [], currentMenuType = "";
let items = [ { id: "PIE", name: "Pie", heal: 92 }, { id: "NOODLES", name: "Noodles", heal: 90 }, { id: "STEAK", name: "Steak", heal: 60 } ];
let inSoulMode = false, isBlue = false;
let soulX = 172, soulY = 160, soulSpeed = 4.5;
let yVelocity = 0, gravity = 0.22, jumpStrength = -7.5, isJumping = false; 
const keys = {};
let typingTimer = null, textIsDone = true, currentCallback = null;
let fightPointerPos = 0, fightPointerDir = 1, fightBarActive = false;
let activeAttacks = [];
function typeText(element, text, speed, onComplete) {
clearTimeout(typingTimer);
element.textContent = "";
textIsDone = false;
let i = 0;
function step() {
if (i > text.length) return;
element.textContent = text.slice(0, i);
if (i === text.length) {
textIsDone = true;
if(onComplete) onComplete();
return;
}
i++;
typingTimer = setTimeout(step, text.charAt(i-1) === ',' ? speed*3 : speed);
}
step();
}
function updateHUD() {
if (playerHP < 0) playerHP = 0;
hpFill.style.width = ((playerHP / playerMaxHP) * 100) + "%";
hpText.textContent = playerHP + " / " + playerMaxHP;
if (playerHP <= 0) {
document.getElementById("end-screen").style.display = "flex";
document.getElementById("end-text").textContent = "YOU DIED.\n\n* stay determined.";
phase = "END";
}
}
function checkPhaseTransition() {
if (enemyHP <= 20 && fightPhase === 1) {
fightPhase = 2;
enemySprite.className = "phase2";
sansDialogueBox.style.display = "block";
typeText(sansDialogueBox, "you're really pushing it...", 40, () => { setTimeout(startEnemyTurn, 1500); });
return true;
} else if (enemyHP <= 10 && fightPhase === 2) {
fightPhase = 3;
enemySprite.className = "phase3";
sansDialogueBox.style.display = "block";
typeText(sansDialogueBox, "I CAN'T AFFORD TO LOSE.", 40, () => { setTimeout(startEnemyTurn, 1500); });
return true;
} else if (enemyHP <= 0) {
document.getElementById("end-screen").style.display = "flex";
document.getElementById("end-text").textContent = "YOU WON.\n\n* he finally stopped.";
phase = "END";
return true;
}
return false;
}
function setMenuIndex(index) {
menuIndex = (index + menuItems.length) % menuItems.length;
document.querySelectorAll(".menu-item").forEach((el, i) => {
el.classList.toggle("selected", i === menuIndex);
el.style.borderColor = i === menuIndex ? "yellow" : "transparent";
});
}
function openSubMenu(type, options) {
currentMenuType = type; currentSubOptions = options; subIndex = 0;
subMenu.innerHTML = ""; subMenu.style.display = "flex"; dialogueBox.style.display = "none";
options.forEach((opt, idx) => {
const span = document.createElement("span");
span.classList.add("sub-option");
if (idx === 0) span.classList.add("selected");
span.textContent = opt.name || opt.label;
subMenu.appendChild(span);
});
phase = "MENU_SUB";
}
function updateSubMenu() {
document.querySelectorAll(".sub-option").forEach((el, i) => {
el.classList.toggle("selected", i === subIndex);
});
}
function setBlueSoul(active) {
isBlue = active;
soul.style.background = active ? "blue" : "red";
soul.style.boxShadow = active ? "0 0 5px blue" : "0 0 5px red";
if (!active) yVelocity = 0;
}
function damagePlayer(amt) {
if (invincFrames > 0) return;
playerHP -= amt; 
invincFrames = 40; 
soul.classList.add("soul-hurt");
setTimeout(() => soul.classList.remove("soul-hurt"), 700);
updateHUD();
}
function rectsOverlap(a, b) {
let sA = {top: a.top+3, bottom: a.bottom-3, left: a.left+3, right: a.right-3}; 
return !(sA.right < b.left || sA.left > b.right || sA.bottom < b.top || sA.top > b.bottom);
}
function isMoving() {
return keys["ArrowUp"] || keys["w"] || keys["W"] || keys["ArrowDown"] || keys["s"] || keys["S"] || keys["ArrowLeft"] || keys["a"] || keys["A"] || keys["ArrowRight"] || keys["d"] || keys["D"];
}
function physicsLoop() {
if (inSoulMode && phase === "ENEMY_ATTACK") {
let up = keys["ArrowUp"] || keys["w"] || keys["W"];
let down = keys["ArrowDown"] || keys["s"] || keys["S"];
let left = keys["ArrowLeft"] || keys["a"] || keys["A"];
let right = keys["ArrowRight"] || keys["d"] || keys["D"];
if (isBlue) {
yVelocity += gravity; soulY += yVelocity;
if (soulY >= 160) { soulY = 160; yVelocity = 0; isJumping = false; } 
if (soulY <= 0) { soulY = 0; yVelocity = 0; }
if (up && !isJumping) { yVelocity = jumpStrength; isJumping = true; }
if (!up && yVelocity < -2) { yVelocity = -2; } 
} else {
if (up) soulY -= soulSpeed;
if (down) soulY += soulSpeed;
}
if (left) soulX -= soulSpeed;
if (right) soulX += soulSpeed;
if (soulX < 0) soulX = 0; if (soulX > 344) soulX = 344; 
if (!isBlue) { if (soulY < 0) soulY = 0; if (soulY > 160) soulY = 160; }
soul.style.left = soulX + "px"; soul.style.top = soulY + "px";
const sRect = soul.getBoundingClientRect();
activeAttacks.forEach(hb => {
if(hb.el && hb.el.parentNode) {
if (rectsOverlap(sRect, hb.el.getBoundingClientRect())) {
if(hb.type === "blue" && isMoving()) damagePlayer(fightPhase*2);
else if(hb.type === "orange" && !isMoving()) damagePlayer(fightPhase*2);
else if(hb.type === "normal" || hb.type === "beam") damagePlayer(fightPhase*2);
}
}
});
}
if (invincFrames > 0) invincFrames--;
if (fightBarActive) {
fightPointerPos += fightPointerDir * (0.025 + (fightPhase * 0.005));
if (fightPointerPos <= 0) { fightPointerPos = 0; fightPointerDir = 1; } 
else if (fightPointerPos >= 0.98) { fightPointerPos = 0.98; fightPointerDir = -1; }
fightPointer.style.left = (fightPointerPos * 320) + "px";
}
requestAnimationFrame(physicsLoop);
}
function spawnBone(x, y, w, h, type="normal") {
const bone = document.createElement("div"); bone.classList.add("bone");
if(type==="blue") bone.classList.add("bone-blue");
if(type==="orange") bone.classList.add("bone-orange");
bone.style.left = x + "px"; bone.style.top = y + "px"; bone.style.width = w + "px"; bone.style.height = h + "px";
soulBox.appendChild(bone); 
let obj = {el: bone, x: x, y: y, type: type, w: w, h: h};
activeAttacks.push(obj);
return obj;
}
function spawnBlaster(x, y, delay, duration, isTracking=false, horizontal=false) {
const blaster = document.createElement("div"); blaster.classList.add("blaster");
if(fightPhase===3) blaster.classList.add("blaster-red");
blaster.style.left = x + "px"; blaster.style.top = y + "px"; 
soulBox.appendChild(blaster);
let lockX = x, lockY = y;
let interval = null;
if(isTracking) {
interval = setInterval(() => {
lockX = horizontal ? x : soulX - 20;
lockY = horizontal ? soulY - 20 : y;
blaster.style.left = lockX + "px"; blaster.style.top = lockY + "px";
}, 50);
}
setTimeout(() => {
if(interval) clearInterval(interval);
const beam = document.createElement("div"); beam.classList.add("blast-beam");
if(fightPhase===3) beam.classList.add("blast-beam-red");
if(horizontal) {
beam.style.width = "400px"; beam.style.height = "20px"; beam.style.left = (lockX+50) + "px"; beam.style.top = (lockY+20) + "px";
} else {
beam.style.width = "20px"; beam.style.height = "300px"; beam.style.left = (lockX+20) + "px"; beam.style.top = (lockY+50) + "px";
}
soulBox.appendChild(beam);
let obj = {el: beam, type: "beam"};
activeAttacks.push(obj);
setTimeout(() => { 
beam.remove(); blaster.remove(); 
activeAttacks = activeAttacks.filter(a => a.el !== beam);
}, duration);
}, delay);
}
function clearAttacks() {
document.querySelectorAll(".bone, .blaster, .blast-beam").forEach(b => b.remove());
activeAttacks = [];
}
function endTurnText() {
inSoulMode = false; soulBox.style.display = "none";
clearAttacks();
phase = "PLAYER_TURN"; dialogueBox.style.display = "block";
let fText = "* Sans awaits your move.";
if(fightPhase === 2) fText = "* Sans is breathing heavily.";
if(fightPhase === 3) fText = "* SANS IS DETERMINED.";
typeText(dialogueBox, fText, 20);
}
function attackPatternPhase1() {
setBlueSoul(true);
let t = 0;
for(let i=0; i<6; i++) {
let c = Math.random() < 0.5 ? "blue" : "orange";
spawnBone(360 + t, 140, 15, 40, c);
t += 120;
}
let attackStart = performance.now();
function move() {
if (phase !== "ENEMY_ATTACK") return;
if (performance.now() - attackStart > 6000) { endTurnText(); return; }
activeAttacks.forEach(b => {
if(b.type !== "beam") { b.x -= 3; b.el.style.left = b.x + "px"; }
});
requestAnimationFrame(move);
}
move();
}
function attackPatternPhase2() {
setBlueSoul(false);
spawnBlaster(150, -20, 1000, 1000, true, false);
spawnBlaster(300, 80, 2500, 1000, true, true);
let t = 0;
for(let i=0; i<4; i++) {
spawnBone(360 + t, Math.random()*100, 20, 80, "normal");
t += 150;
}
let attackStart = performance.now();
function move() {
if (phase !== "ENEMY_ATTACK") return;
if (performance.now() - attackStart > 5000) { endTurnText(); return; }
activeAttacks.forEach(b => {
if(b.type !== "beam") { b.x -= 4; b.el.style.left = b.x + "px"; }
});
requestAnimationFrame(move);
}
move();
}
function attackPatternPhase3() {
setBlueSoul(true);
spawnBlaster(150, -20, 800, 800, true, false);
spawnBlaster(250, -20, 1600, 800, true, false);
spawnBlaster(360, 50, 2400, 800, true, true);
let t = 0;
for(let i=0; i<8; i++) {
let c = Math.random() < 0.33 ? "blue" : (Math.random() < 0.5 ? "orange" : "normal");
spawnBone(360 + t, 130, 20, 50, c);
t += 80;
}
let attackStart = performance.now();
function move() {
if (phase !== "ENEMY_ATTACK") return;
if (performance.now() - attackStart > 6500) { endTurnText(); return; }
activeAttacks.forEach(b => {
if(b.type !== "beam") { b.x -= 6; b.el.style.left = b.x + "px"; }
});
requestAnimationFrame(move);
}
move();
}
function startEnemyTurn() {
turnCount++;
phase = "ENEMY_ATTACK";
dialogueBox.style.display = "none";
sansDialogueBox.style.display = "none";
inSoulMode = true; soulBox.style.display = "block";
soulX = 172; soulY = 160; 
clearAttacks();
if(fightPhase === 1) attackPatternPhase1();
else if(fightPhase === 2) attackPatternPhase2();
else attackPatternPhase3();
}
function startFightBar() {
phase = "FIGHT_BAR"; dialogueBox.style.display = "none"; fightBarContainer.style.display = "flex";
fightPointerPos = 0; fightPointerDir = 1; fightBarActive = true; damageText.textContent = "";
}
function stopFightBar() {
fightBarActive = false;
let ptr = fightPointerPos * 320;
let dist = Math.abs(ptr - 160);
let dmg = 0;
if(dist < 20) dmg = 4;
else if(dist < 50) dmg = 2;
else if(dist < 100) dmg = 1;
if(dmg > 0) {
damageText.textContent = dmg;
enemyHP -= dmg;
} else {
damageText.textContent = "MISS";
}
setTimeout(() => {
fightBarContainer.style.display = "none";
if(!checkPhaseTransition()) {
sansDialogueBox.style.display = "block";
typeText(sansDialogueBox, dmg>0 ? "ugh..." : "nice try.", 30, () => { setTimeout(startEnemyTurn, 1000); });
}
}, 1500);
}
function handleInput(key) {
if (phase === "END") return;
if (key === "z" || key === "Z" || key === "Enter") {
if (!textIsDone) return;
if (phase === "INTRO") {
sansDialogueBox.style.display = "none"; phase = "PLAYER_TURN"; dialogueBox.style.display = "block";
typeText(dialogueBox, "* The air crackles with tension.", 20);
}
else if (phase === "PLAYER_TURN") {
const action = menuItems[menuIndex];
if (action === "ITEM") {
if (items.length === 0) typeText(dialogueBox, "* You have no items left.", 20);
else openSubMenu("ITEM", items);
} else if (action === "ACT") {
openSubMenu("ACT", [{label: "Check"}, {label: "Taunt"}]);
} else if (action === "MERCY") {
openSubMenu("MERCY", [{label: "Spare"}]);
} else if (action === "FIGHT") {
startFightBar();
}
}
else if (phase === "MENU_SUB") {
subMenu.style.display = "none"; dialogueBox.style.display = "block"; phase = "DIALOGUE";
if (currentMenuType === "ITEM") {
const selected = currentSubOptions[subIndex];
playerHP = Math.min(playerMaxHP, playerHP + selected.heal); updateHUD();
items = items.filter(i => i.id !== selected.id);
typeText(dialogueBox, `* You ate the ${selected.name}.`, 20, () => { setTimeout(startEnemyTurn, 1000); });
}
else if (currentMenuType === "ACT") {
if (currentSubOptions[subIndex].label === "Check") {
typeText(dialogueBox, `* SANS - HP ${enemyHP}\n* He remembers what you did.`, 20, () => { setTimeout(startEnemyTurn, 1500); });
} else {
typeText(dialogueBox, "* You taunted Sans.", 20, () => {
setTimeout(() => {
sansDialogueBox.style.display = "block";
typeText(sansDialogueBox, "...", 30, () => { setTimeout(startEnemyTurn, 1000); });
}, 500);
});
}
}
else if (currentMenuType === "MERCY") {
typeText(dialogueBox, "* You spared Sans.", 20, () => {
setTimeout(() => {
sansDialogueBox.style.display = "block";
typeText(sansDialogueBox, "there is no mercy here.", 40, () => { setTimeout(startEnemyTurn, 1000); });
}, 500);
});
}
}
else if (phase === "FIGHT_BAR") {
stopFightBar();
}
}
if ((key === "x" || key === "X" || key === "Shift") && phase === "MENU_SUB") {
subMenu.style.display = "none"; phase = "PLAYER_TURN"; dialogueBox.style.display = "block";
typeText(dialogueBox, "* Sans is waiting.", 20);
}
let left = key === "ArrowLeft" || key === "a" || key === "A";
let right = key === "ArrowRight" || key === "d" || key === "D";
if (phase === "PLAYER_TURN") {
if (left) setMenuIndex(menuIndex - 1);
if (right) setMenuIndex(menuIndex + 1);
} else if (phase === "MENU_SUB") {
if (left) { subIndex = (subIndex - 1 + currentSubOptions.length) % currentSubOptions.length; updateSubMenu(); }
if (right) { subIndex = (subIndex + 1) % currentSubOptions.length; updateSubMenu(); }
}
}
document.addEventListener("keydown", e => { keys[e.key] = true; handleInput(e.key); });
document.addEventListener("keyup", e => keys[e.key] = false);
updateHUD(); setMenuIndex(0);
sansDialogueBox.style.display = "block";
typeText(sansDialogueBox, "do you really think\nyou're in control?", 40);
requestAnimationFrame(physicsLoop);
</script>
</body>
</html>
