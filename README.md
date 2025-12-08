<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Undertale Battle Demo</title>
  <style>
    body {
      background: black;
      color: white;
      font-family: monospace;
      text-align: center;
    }
    #battle-box {
      border: 2px solid white;
      width: 400px;
      height: 200px;
      margin: 20px auto;
      position: relative;
    }
    #soul {
      width: 20px;
      height: 20px;
      background: red;
      position: absolute;
      top: 90px;
      left: 190px;
    }
    .bullet {
      width: 10px;
      height: 10px;
      background: white;
      position: absolute;
    }
  </style>
</head>
<body>
  <h1>Undertale Battle Demo</h1>
  <div id="battle-box">
    <div id="soul"></div>
  </div>
  <script>
    const soul = document.getElementById("soul");
    let x = 190, y = 90;

    document.addEventListener("keydown", e => {
      if (e.key === "ArrowUp") y -= 10;
      if (e.key === "ArrowDown") y += 10;
      if (e.key === "ArrowLeft") x -= 10;
      if (e.key === "ArrowRight") x += 10;
      soul.style.left = x + "px";
      soul.style.top = y + "px";
    });

    // Example bullet
    const box = document.getElementById("battle-box");
    const bullet = document.createElement("div");
    bullet.className = "bullet";
    bullet.style.top = "0px";
    bullet.style.left = "200px";
    box.appendChild(bullet);

    let pos = 0;
    setInterval(() => {
      pos += 5;
      bullet.style.top = pos + "px";
      if (pos > 200) pos = 0;
    }, 100);
  </script>
</body>
</html>
