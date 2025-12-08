<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Mini Undertale Battle</title>
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
      height: 180px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
    }

    #enemy-name {
      font-size: 24px;
      margin-bottom: 8px;
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
      background: orange;
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
      display: flex;
      align-items: center;
      justify-content: center;
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
      border-radius: 50%;
    }

    /* UI bar at bottom */
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
  </style>
</head>
<body>
  <div id="game">
    <div id="enemy-area">
      <div id="enemy-name">Dummy</div>
      <div id="enemy-sprite">ENEMY</div>
      <div id="enemy-hp-container">
        <div id="enemy-hp-bar"></div>
      </div>
      <div id="enemy-hp-text">HP: 30 / 30</div>
    </div>

    <!-- Dialogue box (text mode) -->
    <div id="dialogue-box" style="display: block;">
      * Dummy blocks the way.
    </div>

    <!-- Soul (battle) box -->
    <div id="soul-box" style="display: none;">
      <div id="soul"></div>
    </div>

    <!-- Bottom UI -->
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
  </div>

  <script>
    // --- Game state ---
    let playerHP = 20;
    let playerMaxHP = 20;
    let enemyHP = 30;
    let enemyMaxHP = 30;

    let inSoulMode = false;
    let canMoveSoul = false;
    let menuIndex = 0;
    const menuItems = ["FIGHT", "ACT", "ITEM", "MERCY"];

    const game = document.getElementById("game");
    const dialogueBox = document.getElementById("dialogue-box");
    const soulBox = document.getElementById("soul-box");
    const soul = document.getElementById("soul");
    const hpFill = document.getElementById("hp-fill");
    const hpText = document.getElementById("hp-text");
    const enemyHPBar = document.getElementById("enemy-hp-bar");
    const enemyHPText = document.getElementById("enemy-hp-text");

    const menuElements = Array.from(document.querySelectorAll(".menu-item"));

    // Initial soul position
    let soulX = 0;
    let soulY = 0;
    let soulSpeed = 4;

    // --- Helper functions ---
    function updatePlayerHP() {
      if (playerHP < 0) playerHP = 0;
      const ratio = playerHP / playerMaxHP;
      hpFill.style.width = (ratio * 100) + "%";
      hpText.textContent = playerHP + " / " + playerMaxHP;
    }

    function updateEnemyHP() {
      if (enemyHP < 0) enemyHP = 0;
      const ratio = enemyHP / enemyMaxHP;
      enemyHPBar.style.width = (ratio * 100) + "%";
      enemyHPText.textContent = "HP: " + enemyHP + " / " + enemyMaxHP;

      if (enemyHP === 0) {
        dialogueBox.style.display = "block";
        soulBox.style.display = "none";
        inSoulMode = false;
        dialogueBox.textContent = "* Dummy is defeated!";
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
      dialogueBox.style.display = "none";
      soulBox.style.display = "block";
      canMoveSoul = true;

      // Center soul in the box
      const boxRect = soulBox.getBoundingClientRect();
      const gameRect = game.getBoundingClientRect();
      soulX = (boxRect.width / 2) - 8;
      soulY = (boxRect.height / 2) - 8;
      soul.style.left = soulX + "px";
      soul.style.top = soulY + "px";

      startSimpleAttack();
    }

    function exitSoulMode() {
      inSoulMode = false;
      canMoveSoul = false;
      soulBox.style.display = "none";
      dialogueBox.style.display = "block";
      dialogueBox.textContent = "* The attack ends.";
      clearBullets();
    }

    function clearBullets() {
      const bullets = soulBox.querySelectorAll(".bullet");
      bullets.forEach(b => b.remove());
    }

    // Simple attack: one bullet moves left to right
    function startSimpleAttack() {
      clearBullets();
      const bullet = document.createElement("div");
      bullet.classList.add("bullet");

      const boxRect = soulBox.getBoundingClientRect();
      const y = Math.random() * (boxRect.height - 10);
      bullet.style.left = "0px";
      bullet.style.top = y + "px";

      soulBox.appendChild(bullet);

      let x = 0;
      const speed = 3;
      const attackDuration = 3000; // ms
      const startTime = performance.now();

      function attackLoop(time) {
        const elapsed = time - startTime;
        if (elapsed > attackDuration || !inSoulMode) {
          bullet.remove();
          exitSoulMode();
          return;
        }

        x += speed;
        bullet.style.left = x + "px";

        // Collision check with soul
        const soulRect = soul.getBoundingClientRect();
        const bulletRect = bullet.getBoundingClientRect();

        if (rectsOverlap(soulRect, bulletRect)) {
          playerHP -= 1;
          updatePlayerHP();
        }

        requestAnimationFrame(attackLoop);
      }

      requestAnimationFrame(attackLoop);
    }

    function rectsOverlap(a, b) {
      return !(
        a.right < b.left ||
        a.left > b.right ||
        a.bottom < b.top ||
        a.top > b.bottom
      );
    }

    // --- Menu action ---
    function confirmMenuSelection() {
      const action = menuItems[menuIndex];
      if (action === "FIGHT") {
        // Simple attack: deal fixed damage and trigger enemy attack
        const damage = 8 + Math.floor(Math.random() * 5);
        enemyHP -= damage;
        updateEnemyHP();

        if (enemyHP > 0) {
          dialogueBox.textContent = "* You hit the Dummy for " + damage + " damage.";
          setTimeout(() => {
            dialogueBox.textContent = "* Dummy prepares an attack...";
            setTimeout(() => {
              enterSoulMode();
            }, 700);
          }, 800);
        }
      } else if (action === "ACT") {
        dialogueBox.textContent = "* You talk to the Dummy. It doesn't seem much for conversation.";
      } else if (action === "ITEM") {
        if (playerHP < playerMaxHP) {
          const heal = 5;
          playerHP = Math.min(playerMaxHP, playerHP + heal);
          updatePlayerHP();
          dialogueBox.textContent = "* You use a healing item. +" + heal + " HP.";
        } else {
          dialogueBox.textContent = "* Your HP is already full.";
        }
      } else if (action === "MERCY") {
        if (enemyHP < enemyMaxHP / 2) {
          dialogueBox.textContent = "* You spare the Dummy. You won!";
          // You could end the game here or go to next scene
        } else {
          dialogueBox.textContent = "* The Dummy doesn't seem ready to be spared.";
        }
      }
    }

    // --- Input handling ---
    const keys = {};
    document.addEventListener("keydown", (e) => {
      keys[e.key] = true;

      if (!inSoulMode) {
        // Menu navigation
        if (e.key === "ArrowRight") {
          setMenuIndex(menuIndex + 1);
        } else if (e.key === "ArrowLeft") {
          setMenuIndex(menuIndex - 1);
        } else if (e.key === "Enter" || e.key === "z" || e.key === "Z") {
          confirmMenuSelection();
        }
      }
    });

    document.addEventListener("keyup", (e) => {
      keys[e.key] = false;
    });

    // --- Soul movement loop ---
    function soulMovementLoop() {
      if (inSoulMode && canMoveSoul) {
        const boxRect = soulBox.getBoundingClientRect();
        const boxWidth = boxRect.width;
        const boxHeight = boxRect.height;

        if (keys["ArrowUp"]) soulY -= soulSpeed;
        if (keys["ArrowDown"]) soulY += soulSpeed;
        if (keys["ArrowLeft"]) soulX -= soulSpeed;
        if (keys["ArrowRight"]) soulX += soulSpeed;

        // Keep soul inside box
        if (soulX < 0) soulX = 0;
        if (soulY < 0) soulY = 0;
        if (soulX > boxWidth - 16) soulX = boxWidth - 16;
        if (soulY > boxHeight - 16) soulY = boxHeight - 16;

        soul.style.left = soulX + "px";
        soul.style.top = soulY + "px";
      }

      requestAnimationFrame(soulMovementLoop);
    }

    // Initialize UI
    updatePlayerHP();
    updateEnemyHP();
    setMenuIndex(0);

    // Start movement loop
    requestAnimationFrame(soulMovementLoop);
  </script>
</body>
</html>
