<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sans Fight - Pacifist Phase 3a</title>
  <style>
    body { margin: 0; background: black; color: white; font-family: "Courier New", monospace; display: flex; justify-content: center; align-items: center; height: 100vh; overflow: hidden; font-weight: bold; }
    #game { width: 800px; height: 600px; border: 4px solid white; position: relative; background: black; overflow: hidden; }
    
    /* Sprites */
    #enemy-area { position: absolute; top: 20px; left: 0; width: 100%; height: 180px; display: flex; flex-direction: column; align-items: center; pointer-events: none; }
    #enemy-sprite { width: 100px; height: 100px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 30 30"><rect x="11" y="2" width="8" height="2" fill="white"/><rect x="9" y="4" width="2" height="4" fill="white"/><rect x="19" y="4" width="2" height="4" fill="white"/><rect x="7" y="8" width="2" height="6" fill="white"/><rect x="21" y="8" width="2" height="6" fill="white"/><rect x="9" y="14" width="2" height="2" fill="white"/><rect x="19" y="14" width="2" height="2" fill="white"/><rect x="11" y="16" width="8" height="2" fill="white"/><rect x="11" y="8" width="3" height="3" fill="black"/><rect x="16" y="8" width="3" height="3" fill="black"/><rect x="13" y="12" width="4" height="2" fill="black"/><rect x="10" y="18" width="10" height="4" fill="white"/><rect x="8" y="20" width="2" height="6" fill="white"/><rect x="20" y="20" width="2" height="6" fill="white"/></svg>'); background-size: contain; background-repeat: no-repeat; animation: idle 2s infinite alternate ease-in-out; }
    @keyframes idle { 0% { transform: translateY(0px); } 100% { transform: translateY(6px); } }
    
    /* Dialogue Boxes */
    #dialogue-box { position: absolute; bottom: 150px; left: 20px; right: 20px; height: 140px; border: 4px solid white; padding: 15px; box-sizing: border-box; font-size: 24px; white-space: pre-line; background: black; display: none; z-index: 5;}
    #sans-dialogue-box { position: absolute; top: 30px; left: 450px; width: 280px; min-height: 40px; border: 4px solid black; border-radius: 10px; padding: 12px; background: white; color: black; font-size: 18px; white-space: pre-line; display: none; z-index: 10; font-weight: normal; }
    
    /* Arena & Soul */
    #soul-box { position: absolute; bottom: 150px; left: 260px; width: 280px; height: 140px; border: 4px solid white; box-sizing: border-box; overflow: hidden; display: none; background: black; }
    #soul { width: 16px; height: 16px; background: red; position: absolute; z-index: 20; box-shadow: 0 0 5px red;}
    .soul-hurt { animation: blink 0.2s infinite; }
    @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0.2; } 100% { opacity: 1; } }
    
    /* Projectiles */
    .bone { position: absolute; background: white; border-radius: 6px; box-shadow: 0 0 4px white; }
    .blaster { position: absolute; width: 60px; height: 60px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 40 40"><path d="M10,5 L30,5 L35,15 L35,25 L25,35 L15,35 L5,25 L5,15 Z" fill="white"/><circle cx="15" cy="15" r="3" fill="black"/><circle cx="25" cy="15" r="3" fill="black"/><rect x="15" y="25" width="10" height="2" fill="black"/></svg>'); background-size: contain; background-repeat: no-repeat; z-index: 15;}
    .blast-beam { position: absolute; background: white; border: 3px solid cyan; box-shadow: 0 0 15px cyan; z-index: 14;}
    
    /* UI Bar */
    #ui-bar { position: absolute; bottom: 0; left: 0; width: 100%; height: 130px; display: flex; flex-direction: column; justify-content: space-between; padding: 10px 30px; box-sizing: border-box; background: black; }
    #stats-row { display: flex; align-items: center; gap: 20px; font-size: 24px; margin-bottom: 15px; }
    #hp-bar-inner { width: 100px; height: 20px; position: relative; background: red; }
    #hp-fill { position: absolute; top: 0; left: 0; height: 100%; background: #ffb000; transition: width 0.2s ease-out; }
    #kr-fill { position: absolute; top: 0; right: 0; height: 100%; background: #f0f; width: 0%; transition: width 0.2s; }
    #kr-label { color: #f0f; display: none; margin-left: 10px; font-size: 16px;}
    
    /* Menus */
    #ui-menu { display: flex; justify-content: space-between; font-size: 24px; color: #ff8000; width: 90%; margin: 0 auto; }
    .menu-item { cursor: pointer; padding: 5px 15px; border: 2px solid transparent;}
    .menu-item.selected { color: yellow; border: 2px solid yellow; }
    #sub-menu { position: absolute; bottom: 150px; left: 30px; width: 700px; height: 140px; display: none; flex-wrap: wrap; align-content: flex-start; gap: 20px; font-size: 24px; padding: 20px; box-sizing: border-box; z-index: 6;}
    .sub-option { width: 45%; cursor: pointer; color: white;}
    .sub-option::before { content: '* '; }
    .sub-option.selected { color: yellow; }
    
    #end-screen { position: absolute; inset: 0; background: black; color: white; display: none; justify-content: center; align-items: center; flex-direction: column; font-size: 24px; text-align: center; white-space: pre-line; z-index: 100;}
  </style>
</head>
<body>
<div id="game">
  <div id="enemy-area"><div id="enemy-sprite"></div></div>
  <div id="dialogue-box"></div>
  <div id="sans-dialogue-box"></div>
  <div id="soul-box"><div id="soul"></div></div>
  
  <div id="sub-menu"></div>
  
  <div id="ui-bar">
    <div id="stats-row">
      <span>FRISK</span>
      <span>LV 1</span>
      <div style="display: flex; align-items: center; gap: 10px;">
        <span>HP</span>
        <div id="hp-bar-inner"><div id="hp-fill"></div><div id="kr-fill"></div></div>
        <span id="hp-text">20 / 20</span>
        <span id="kr-label">KR</span>
      </div>
    </div>
    <div id="ui-menu">
      <span class="menu-item selected">FIGHT</span>
      <span class="menu-item">ACT</span>
      <span class="menu-item">ITEM</span>
      <span class="menu-item">MERCY</span>
    </div>
  </div>
  <div id="end-screen"><div id="end-text"></div><div style="font-size: 18px; margin-top: 10px;">(refresh page to restart)</div></div>
</div>

<script>
  // ========= ENGINE STATE =========
  let playerHP = 20, playerMaxHP = 20, kr = 0, invincFrames = 0;
  let phase = "INTRO"; // Phases: INTRO, PLAYER_TURN, MENU_SUB, DIALOGUE, ENEMY_ATTACK, END
  let turnCount = 0;
  
  // Elements
  const gameEl = document.getElementById("game");
  const dialogueBox = document.getElementById("dialogue-box");
  const sansDialogueBox = document.getElementById("sans-dialogue-box");
  const soulBox = document.getElementById("soul-box");
  const soul = document.getElementById("soul");
  const subMenu = document.getElementById("sub-menu");
  const hpFill = document.getElementById("hp-fill");
  const krFill = document.getElementById("kr-fill");
  const hpText = document.getElementById("hp-text");
  
  // Menus & Items
  const menuItems = ["FIGHT", "ACT", "ITEM", "MERCY"];
  let menuIndex = 0, subIndex = 0, currentSubOptions = [], currentMenuType = "";
  let items = [ { id: "PIE", name: "Butterscotch Pie", heal: 20 }, { id: "NOODLES", name: "Instant Noodles", heal: 15 } ];
  
  // Physics (Fairer Blue Soul)
  let inSoulMode = false, isBlue = false;
  let soulX = 132, soulY = 62, soulSpeed = 4;
  let yVelocity = 0, gravity = 0.20, jumpStrength = -5.5, isJumping = false; // Tweaked for hang-time
  const keys = {};

  // Text Typewriter
  let typingTimer = null, textIsDone = true, currentCallback = null;

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
        if(onComplete) currentCallback = onComplete;
        return;
      }
      i++;
      typingTimer = setTimeout(step, text.charAt(i-1) === ',' ? speed*3 : speed);
    }
    step();
  }

  // --- UI AND MENUS ---
  function updateHUD() {
    if (playerHP < 0) playerHP = 0;
    const hpRatio = playerHP / playerMaxHP;
    hpFill.style.width = (hpRatio * 100) + "%";
    hpText.textContent = playerHP + " / " + playerMaxHP;
    if (playerHP <= 0) {
      document.getElementById("end-screen").style.display = "flex";
      document.getElementById("end-text").textContent = "YOU DIED.\n\n* don't give up now.";
      phase = "END";
    }
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

  // --- PHYSICS & DAMAGE ---
  function setBlueSoul(active) {
    isBlue = active;
    soul.style.background = active ? "blue" : "red";
    soul.style.boxShadow = active ? "0 0 5px blue" : "0 0 5px red";
    if (!active) yVelocity = 0;
  }

  function damagePlayer() {
    if (invincFrames > 0) return;
    playerHP -= 3; // Standard damage
    invincFrames = 45; // 45 frames of invincibility (very fair!)
    soul.classList.add("soul-hurt");
    setTimeout(() => soul.classList.remove("soul-hurt"), 750);
    updateHUD();
  }

  function rectsOverlap(a, b) {
      let sA = {top: a.top+3, bottom: a.bottom-3, left: a.left+3, right: a.right-3}; // Forgiving hitbox
      return !(sA.right < b.left || sA.left > b.right || sA.bottom < b.top || sA.top > b.bottom);
  }

  function physicsLoop() {
    if (inSoulMode && phase === "ENEMY_ATTACK") {
      // Gravity & Jumping
      if (isBlue) {
        yVelocity += gravity; soulY += yVelocity;
        if (soulY >= 120) { soulY = 120; yVelocity = 0; isJumping = false; }
        if (soulY <= 0) { soulY = 0; yVelocity = 0; }
        
        if (keys["ArrowUp"] && !isJumping) { yVelocity = jumpStrength; isJumping = true; }
        if (!keys["ArrowUp"] && yVelocity < -1) { yVelocity = -1; } // Variable jump height
      } else {
        if (keys["ArrowUp"]) soulY -= soulSpeed;
        if (keys["ArrowDown"]) soulY += soulSpeed;
      }
      
      // Horizontal
      if (keys["ArrowLeft"]) soulX -= soulSpeed;
      if (keys["ArrowRight"]) soulX += soulSpeed;
      
      // Bounds
      if (soulX < 0) soulX = 0; if (soulX > 264) soulX = 264;
      if (!isBlue) { if (soulY < 0) soulY = 0; if (soulY > 120) soulY = 120; }
      
      soul.style.left = soulX + "px"; soul.style.top = soulY + "px";

      // Collision Detection
      const sRect = soul.getBoundingClientRect();
      document.querySelectorAll(".bone, .blast-beam").forEach(hb => {
          if (rectsOverlap(sRect, hb.getBoundingClientRect())) damagePlayer();
      });
    }

    if (invincFrames > 0) invincFrames--;
    requestAnimationFrame(physicsLoop);
  }

  // --- ATTACK LOGIC ---
  function spawnBone(x, y, w, h) {
    const bone = document.createElement("div"); bone.classList.add("bone");
    bone.style.left = x + "px"; bone.style.top = y + "px"; bone.style.width = w + "px"; bone.style.height = h + "px";
    soulBox.appendChild(bone); return bone;
  }

  function startEnemyTurn() {
    phase = "ENEMY_ATTACK";
    dialogueBox.style.display = "none";
    sansDialogueBox.style.display = "none";
    
    inSoulMode = true;
    soulBox.style.display = "block";
    soulX = 132; soulY = 120; // Start on floor
    setBlueSoul(true); // Fair Blue Soul mode!
    
    // Attack Sequence: Fair sliding bones you must jump over
    const bones = [];
    let spawnTimer = 0;
    
    // Spawn 5 bones, coming from the right, with gaps to jump
    for(let i=0; i<5; i++) {
        bones.push({ el: spawnBone(300 + spawnTimer, 100, 15, 40), x: 300 + spawnTimer });
        spawnTimer += 120; // Distance between bones
    }

    const attackStart = performance.now();
    function moveBones() {
        if (phase !== "ENEMY_ATTACK") return;
        if (performance.now() - attackStart > 6000) {
            // End Attack
            inSoulMode = false;
            soulBox.style.display = "none";
            document.querySelectorAll(".bone").forEach(b => b.remove());
            phase = "PLAYER_TURN";
            dialogueBox.style.display = "block";
            typeText(dialogueBox, "* Sans is breathing heavily.", 20);
            return;
        }
        
        bones.forEach(b => {
            b.x -= 3; // Fair speed
            b.el.style.left = b.x + "px";
        });
        requestAnimationFrame(moveBones);
    }
    moveBones();
  }

  // --- GAME FLOW & INPUT ---
  function handleInput(key) {
    if (phase === "END") return;

    if (key === "z" || key === "Z" || key === "Enter") {
        if (!textIsDone) return; // Wait for text to finish typing

        if (phase === "INTRO") {
            sansDialogueBox.style.display = "none";
            phase = "PLAYER_TURN";
            dialogueBox.style.display = "block";
            typeText(dialogueBox, "* Sans stands in your way.\n* He looks completely exhausted.", 20);
        }
        else if (phase === "PLAYER_TURN") {
            const action = menuItems[menuIndex];
            if (action === "ITEM") {
                if (items.length === 0) {
                    typeText(dialogueBox, "* You have no items left.", 20);
                } else {
                    openSubMenu("ITEM", items);
                }
            } else if (action === "ACT") {
                openSubMenu("ACT", [{label: "Check"}, {label: "Talk"}]);
            } else if (action === "MERCY") {
                openSubMenu("MERCY", [{label: "Spare"}]);
            } else if (action === "FIGHT") {
                dialogueBox.style.display = "block";
                phase = "DIALOGUE";
                typeText(dialogueBox, "* You swung your weapon.\n* Sans effortlessly dodged it.", 20, () => {
                    setTimeout(() => {
                        sansDialogueBox.style.display = "block";
                        typeText(sansDialogueBox, "heh. you really like swinging that thing, huh?", 30, () => {
                            setTimeout(startEnemyTurn, 1000);
                        });
                    }, 500);
                });
            }
        }
        else if (phase === "MENU_SUB") {
            subMenu.style.display = "none";
            dialogueBox.style.display = "block";
            phase = "DIALOGUE";
            
            if (currentMenuType === "ITEM") {
                const selected = currentSubOptions[subIndex];
                playerHP = Math.min(playerMaxHP, playerHP + selected.heal);
                updateHUD();
                items = items.filter(i => i.id !== selected.id); // Remove item
                typeText(dialogueBox, `* You ate the ${selected.name}.\n* You recovered HP!`, 20, () => {
                    setTimeout(startEnemyTurn, 1000); // Proceed to enemy turn
                });
            }
            else if (currentMenuType === "ACT") {
                const selected = currentSubOptions[subIndex];
                if (selected.label === "Check") {
                     typeText(dialogueBox, "* SANS - ATK 1 DEF 1\n* He can't dodge forever. Keep attacking.", 20, () => { setTimeout(startEnemyTurn, 1500); });
                } else {
                     typeText(dialogueBox, "* You told Sans you don't want to fight.", 20, () => {
                         setTimeout(() => {
                            sansDialogueBox.style.display = "block";
                            typeText(sansDialogueBox, "too late for that, kid.", 30, () => { setTimeout(startEnemyTurn, 1000); });
                         }, 500);
                     });
                }
            }
            else if (currentMenuType === "MERCY") {
                typeText(dialogueBox, "* You spared Sans.", 20, () => {
                    setTimeout(() => {
                        sansDialogueBox.style.display = "block";
                        typeText(sansDialogueBox, "... i can't afford to care anymore.", 40, () => { setTimeout(startEnemyTurn, 1000); });
                    }, 500);
                });
            }
        }
    }
    
    if ((key === "x" || key === "X") && phase === "MENU_SUB") {
        subMenu.style.display = "none";
        phase = "PLAYER_TURN";
        dialogueBox.style.display = "block";
        typeText(dialogueBox, "* Sans is waiting.", 20);
    }

    // Navigation
    if (phase === "PLAYER_TURN") {
        if (key === "ArrowLeft") setMenuIndex(menuIndex - 1);
        if (key === "ArrowRight") setMenuIndex(menuIndex + 1);
    } else if (phase === "MENU_SUB") {
        if (key === "ArrowLeft") { subIndex = (subIndex - 1 + currentSubOptions.length) % currentSubOptions.length; updateSubMenu(); }
        if (key === "ArrowRight") { subIndex = (subIndex + 1) % currentSubOptions.length; updateSubMenu(); }
    }
  }

  // --- INIT ---
  document.addEventListener("keydown", e => { keys[e.key] = true; handleInput(e.key); });
  document.addEventListener("keyup", e => keys[e.key] = false);

  updateHUD();
  setMenuIndex(0);
  
  // Start Intro
  sansDialogueBox.style.display = "block";
  typeText(sansDialogueBox, "you think you can just\nwalk away from this?", 40);

  requestAnimationFrame(physicsLoop);
</script>
</body>
</html>
