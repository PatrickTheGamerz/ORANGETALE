<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Undertale-Inspired Demo</title>
  <style>
    body {
      background: black;
      color: white;
      font-family: monospace;
      margin: 0;
      overflow: hidden;
    }
    #game {
      width: 640px;
      height: 480px;
      margin: auto;
      background: #222;
      position: relative;
    }
    .player {
      width: 32px;
      height: 32px;
      background: red;
      position: absolute;
      transition: top 0.1s linear, left 0.1s linear;
    }
    .npc, .object {
      width: 32px;
      height: 32px;
      background: yellow;
      position: absolute;
    }
    #dialogue {
      position: absolute;
      bottom: 0;
      width: 100%;
      background: black;
      color: white;
      padding: 10px;
      display: none;
    }
  </style>
</head>
<body>
  <div id="game">
    <div id="player" class="player"></div>
    <div id="npc" class="npc" style="top:200px;left:300px;"></div>
    <div id="rock" class="object" style="top:100px;left:100px;"></div>
    <div id="dialogue"></div>
  </div>

  <script>
    const game = document.getElementById("game");
    const player = document.getElementById("player");
    const dialogue = document.getElementById("dialogue");

    let posX = 50, posY = 50;
    player.style.left = posX + "px";
    player.style.top = posY + "px";

    let encounterTimer = 0;

    document.addEventListener("keydown", e => {
      const step = 10;
      if (dialogue.style.display === "block") return; // freeze during dialogue

      if (e.key === "ArrowUp") posY = Math.max(0, posY - step);
      if (e.key === "ArrowDown") posY = Math.min(448, posY + step);
      if (e.key === "ArrowLeft") posX = Math.max(0, posX - step);
      if (e.key === "ArrowRight") posX = Math.min(608, posX + step);

      player.style.left = posX + "px";
      player.style.top = posY + "px";

      // Random encounter chance
      encounterTimer++;
      if (encounterTimer > 20 && Math.random() < 0.05) {
        startEncounter();
        encounterTimer = 0;
      }
    });

    document.addEventListener("keydown", e => {
      if (e.key === " ") { // interact
        if (checkCollision(player, document.getElementById("npc"))) {
          showDialogue("Hello traveler! Beware of monsters...");
        } else if (checkCollision(player, document.getElementById("rock"))) {
          showDialogue("It's just a rock.");
        }
      }
    });

    function checkCollision(a, b) {
      const aRect = a.getBoundingClientRect();
      const bRect = b.getBoundingClientRect();
      return !(
        aRect.top > bRect.bottom ||
        aRect.bottom < bRect.top ||
        aRect.left > bRect.right ||
        aRect.right < bRect.left
      );
    }

    function showDialogue(text) {
      dialogue.innerText = text + " (press Enter)";
      dialogue.style.display = "block";
      document.addEventListener("keydown", e => {
        if (e.key === "Enter") dialogue.style.display = "none";
      }, { once: true });
    }

    function startEncounter() {
      alert("Monster encounter! (Battle system would start here)");
      // Here you’d swap to your battle engine
    }
  </script>
</body>
</html>
