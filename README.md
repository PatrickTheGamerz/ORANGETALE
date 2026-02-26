<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UTD: ABSOLUTE DUSTTALE</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --blue: #3b82f6; --purple: #d946ef; --dust: #8a2be2; }
        body { background: #000; color: #fff; font-family: 'Press Start 2P', cursive; margin: 0; overflow: hidden; height: 100vh; display: flex; align-items: center; justify-content: center; }
        
        #game-frame { width: 900px; height: 700px; border: 4px solid #fff; position: relative; background: #000; overflow: hidden; }

        /* HUD & STATS */
        header { position: absolute; top: 10px; width: 100%; display: flex; justify-content: space-around; font-size: 14px; z-index: 100; }
        #kr-container { color: #ff00ff; font-size: 12px; }

        /* ARENA & BOX */
        #arena { width: 100%; height: 400px; position: relative; }
        #hooded-sans { position: absolute; top: 50px; left: 50%; transform: translateX(-50%); text-align: center; }
        #sans-sprite { font-size: 80px; filter: drop-shadow(0 0 10px var(--dust)); }
        
        #battle-box { position: absolute; bottom: 180px; left: 50%; transform: translateX(-50%); width: 350px; height: 180px; border: 5px solid #fff; background: #000; }
        #soul { width: 16px; height: 16px; position: absolute; z-index: 1000; }

        /* MENU BUTTONS */
        #menu-container { position: absolute; bottom: 20px; width: 100%; display: flex; justify-content: space-around; padding: 0 40px; box-sizing: border-box; }
        .menu-btn { width: 160px; height: 50px; border: 3px solid #ffaa00; display: flex; align-items: center; justify-content: center; color: #ffaa00; font-size: 16px; cursor: pointer; }
        .menu-btn.active { background: #ffaa00; color: #000; box-shadow: 0 0 15px #ffaa00; }

        /* DIALOGUE */
        #text-box { position: absolute; bottom: 180px; left: 50%; transform: translateX(-50%); width: 340px; height: 170px; background: #000; border: 4px solid #fff; padding: 15px; font-size: 18px; line-height: 1.5; box-sizing: border-box; display: none; z-index: 10; }

        /* VFX */
        .bone { position: absolute; background: #fff; box-shadow: 0 0 5px #fff; }
        .blue-bone { background: var(--cyan); box-shadow: 0 0 8px var(--cyan); }
        .blaster-beam { position: absolute; background: #fff; box-shadow: 0 0 20px #fff; opacity: 0; z-index: 50; }
    </style>
</head>
<body>

<div id="game-frame">
    <header>
        <div id="owner-tag" style="color:var(--cyan)">OWNER: PLAYER</div>
        <div>CHARA LV 10</div>
        <div>HP <div style="display:inline-block; width:80px; height:15px; background:red; border:1px solid #fff; vertical-align:middle;"><div id="hp-bar" style="width:100%; height:100%; background:var(--gold);"></div></div></div>
        <div id="kr-container">KR 56 / 56</div>
    </header>

    <div id="arena">
        <div id="hooded-sans">
            <div id="sans-sprite">💀</div>
            <div style="width:180px; height:10px; border:2px solid #fff; margin:10px auto;"><div id="enemy-hp" style="width:100%; height:100%; background:var(--dust);"></div></div>
        </div>
        <div id="battle-box">
            <div id="soul">
                <svg viewBox="0 0 32 32"><path id="soul-svg" d="M16,28.3L3.7,16c-2.3-2.3-2.3-6.1,0-8.5c2.3-2.3,6.1-2.3,8.5,0l3.8,3.8l3.8-3.8c2.3-2.3,6.1-2.3,8.5,0c2.3,2.3,2.3,6.1,0,8.5L16,28.3z" fill="red"/></svg>
            </div>
        </div>
        <div id="text-box"></div>
    </div>

    <div id="menu-container">
        <div class="menu-btn active" id="btn0">FIGHT</div>
        <div class="menu-btn" id="btn1">ACT</div>
        <div class="menu-btn" id="btn2">ITEM</div>
        <div class="menu-btn" id="btn3">MERCY</div>
    </div>
</div>

<script>
/** ⚙️ THE DUST PARADOX ENGINE **/
const STATE = {
    hp: 56, maxHp: 56, kr: 56, lv: 10,
    owner: 'PLAYER', // PLAYER, FRISK
    phase: 'MENU', 
    menuIdx: 0,
    soul: { x: 167, y: 82, vY: 0, gravity: false, jumpPower: -6, isGrounded: false, invuln: 0, moving: false },
    keys: {},
    enemyHP: 1000,
    textQueue: [
        "* You feel the dust of the \nUnderground on your hands.",
        "* Sans is staring... \nwith an empty hood.",
        "* The air is heavy with Karma."
    ]
};

const soul = document.getElementById('soul');
const box = document.getElementById('battle-box');
const textBox = document.getElementById('text-box');

/** 💓 PHYSICS LOOP **/
function update() {
    if (STATE.phase === 'ATTACK') {
        if (STATE.soul.invuln > 0) STATE.soul.invuln--;
        
        let speed = 3.5;
        let oldX = STATE.soul.x;
        let oldY = STATE.soul.y;

        if (STATE.owner === 'PLAYER') {
            // Blue Soul Jumping Logic
            if (STATE.soul.gravity) {
                if ((STATE.keys['ArrowUp'] || STATE.keys['z'] || STATE.keys['Z']) && STATE.soul.isGrounded) {
                    STATE.soul.vY = STATE.soul.jumpPower;
                    STATE.soul.isGrounded = false;
                }
                STATE.soul.vY += 0.25; // Gravity Constant
                STATE.soul.y += STATE.soul.vY;
            } else {
                if (STATE.keys['ArrowUp']) STATE.soul.y -= speed;
                if (STATE.keys['ArrowDown']) STATE.soul.y += speed;
            }
            if (STATE.keys['ArrowLeft']) STATE.soul.x -= speed;
            if (STATE.keys['ArrowRight']) STATE.soul.x += speed;
        } else {
            // Frisk AI Control
            friskAutoControl();
        }

        // Box Collision
        STATE.soul.x = Math.max(0, Math.min(box.clientWidth - 21, STATE.soul.x));
        STATE.soul.y = Math.max(0, Math.min(box.clientHeight - 21, STATE.soul.y));
        
        if (STATE.soul.y >= box.clientHeight - 21) {
            STATE.soul.y = box.clientHeight - 21;
            STATE.soul.vY = 0;
            STATE.soul.isGrounded = true;
        } else {
            STATE.soul.isGrounded = false;
        }

        STATE.soul.moving = (oldX !== STATE.soul.x || oldY !== STATE.soul.y);
        
        soul.style.left = STATE.soul.x + 'px';
        soul.style.top = STATE.soul.y + 'px';
    }

    requestAnimationFrame(update);
}

/** 🤖 FRISK AI **/
function friskAutoControl() {
    // Frisk refuses to move much, but dodges projectiles
    const hazards = document.querySelectorAll('.bone, .blaster-beam');
    hazards.forEach(h => {
        const hR = h.getBoundingClientRect();
        const sR = soul.getBoundingClientRect();
        if (Math.abs(hR.left - sR.left) < 50) {
            STATE.soul.x += (sR.left < hR.left ? -2 : 2);
            if (STATE.soul.gravity && hR.top > sR.top) {
                if (STATE.soul.isGrounded) STATE.soul.vY = -6;
            }
        }
    });
}

/** ⚔️ ATTACK VARIANTS **/
function startEnemyTurn() {
    STATE.phase = 'ATTACK';
    textBox.style.display = 'none';
    
    // Choose attack variant
    const r = Math.random();
    if (r < 0.33) spiralBlasterAttack();
    else if (r < 0.66) boneRainAttack();
    else blueSoulJumpAttack();

    setTimeout(() => {
        STATE.phase = 'MENU';
        STATE.soul.gravity = false;
        document.getElementById('soul-svg').setAttribute('fill', 'red');
        box.innerHTML = '<div id="soul">' + soul.innerHTML + '</div>'; // Clear arena
        showTypewriter(STATE.textQueue[Math.floor(Math.random()*STATE.textQueue.length)]);
    }, 7000);
}

function spiralBlasterAttack() {
    for (let i = 0; i < 12; i++) {
        setTimeout(() => {
            if (STATE.phase !== 'ATTACK') return;
            spawnBlaster(150, 70, i * 30); // Rotational spiral
        }, i * 500);
    }
}

function boneRainAttack() {
    for (let i = 0; i < 20; i++) {
        setTimeout(() => {
            const isBlue = Math.random() > 0.7;
            spawnBone(Math.random() * 300, isBlue);
        }, i * 300);
    }
}

function blueSoulJumpAttack() {
    STATE.soul.gravity = true;
    document.getElementById('soul-svg').setAttribute('fill', 'blue');
    for (let i = 0; i < 10; i++) {
        setTimeout(() => spawnFloorBone(350, -4), i * 600);
    }
}

/** 💥 VFX SPAWNING **/
function spawnBlaster(x, y, angle) {
    const beam = document.createElement('div');
    beam.className = 'blaster-beam';
    beam.style.left = '0px'; beam.style.top = y + 'px';
    beam.style.width = '350px'; beam.style.height = '40px';
    beam.style.transform = `rotate(${angle}deg)`;
    box.appendChild(beam);
    
    setTimeout(() => beam.style.opacity = '0.3', 200);
    setTimeout(() => {
        beam.style.opacity = '1';
        beam.style.background = '#fff';
        const hit = setInterval(() => checkCollision(beam, 6), 16);
        setTimeout(() => { clearInterval(hit); beam.remove(); }, 400);
    }, 1000);
}

function spawnBone(x, isBlue) {
    const b = document.createElement('div');
    b.className = isBlue ? 'bone blue-bone' : 'bone';
    b.style.width = '10px'; b.style.height = '60px';
    b.style.left = x + 'px'; b.style.top = '-70px';
    box.appendChild(b);
    let curY = -70;
    const move = setInterval(() => {
        curY += 4; b.style.top = curY + 'px';
        checkCollision(b, 2, isBlue);
        if (curY > 200) { clearInterval(move); b.remove(); }
    }, 16);
}

function spawnFloorBone(x, vx) {
    const b = document.createElement('div');
    b.className = 'bone';
    b.style.width = '20px'; b.style.height = '40px';
    b.style.left = x + 'px'; b.style.bottom = '0px';
    box.appendChild(b);
    let curX = x;
    const move = setInterval(() => {
        curX += vx; b.style.left = curX + 'px';
        checkCollision(b, 2);
        if (curX < -30) { clearInterval(move); b.remove(); }
    }, 16);
}

function checkCollision(el, dmg, isBlue = false) {
    if (STATE.soul.invuln > 0) return;
    const sR = soul.getBoundingClientRect();
    const eR = el.getBoundingClientRect();
    if (!(sR.right < eR.left || sR.left > eR.right || sR.bottom < eR.top || sR.top > eR.bottom)) {
        // Blue bone mechanic: only damage if moving
        if (isBlue && !STATE.soul.moving) return;
        
        STATE.hp -= 2;
        STATE.soul.invuln = 40;
        document.getElementById('hp-bar').style.width = (STATE.hp/STATE.maxHp*100)+'%';
        if (STATE.hp <= 0) alert("GAME OVER");
    }
}

/** 💬 UI & SOVEREIGNTY **/
function showTypewriter(txt) {
    textBox.style.display = 'block';
    textBox.innerHTML = "";
    let i = 0;
    const timer = setInterval(() => {
        textBox.innerHTML += txt[i];
        i++;
        if (i >= txt.length) clearInterval(timer);
    }, 40);
}

function toggleQ() {
    if (STATE.owner === 'PLAYER') {
        STATE.owner = 'FRISK';
        document.getElementById('owner-tag').innerText = "OWNER: FRISK";
        document.getElementById('owner-tag').style.color = "var(--purple)";
        friskAiTurn();
    }
}

function friskAiTurn() {
    // Frisk automatically chooses ACT or MERCY
    setTimeout(() => {
        STATE.menuIdx = 1; // Always ACT
        updateMenu();
        setTimeout(() => confirmSelection(), 1500);
    }, 1000);
}

function confirmSelection() {
    if (STATE.owner === 'FRISK' && STATE.menuIdx === 0) return; // Frisk cannot fight
    startEnemyTurn();
}

function updateMenu() {
    for(let i=0; i<4; i++) document.getElementById('btn'+i).classList.toggle('active', i === STATE.menuIdx);
}

window.onkeydown = (e) => {
    STATE.keys[e.key] = true;
    if (e.key === 'q' || e.key === 'Q') toggleQ();
    if (STATE.owner === 'PLAYER' && STATE.phase === 'MENU') {
        if (e.key === 'ArrowLeft') { STATE.menuIdx = (STATE.menuIdx + 3) % 4; updateMenu(); }
        if (e.key === 'ArrowRight') { STATE.menuIdx = (STATE.menuIdx + 1) % 4; updateMenu(); }
        if (e.key === 'z' || e.key === 'Z') confirmSelection();
    }
};
window.onkeyup = (e) => STATE.keys[e.key] = false;

// Initialize
update();
showTypewriter(STATE.textQueue[0]);

</script>
</body>
</html>
