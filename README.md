<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Undertale-Style Engine</title>
    <style>
        body {
            background-color: #000;
            color: #fff;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-top: 50px;
        }
        #game-container {
            width: 640px;
            height: 480px;
            background-color: #111;
            border: 4px solid #fff;
            position: relative;
            overflow: hidden;
        }
        #player {
            width: 20px;
            height: 20px;
            background-color: red; /* Represents the SOUL/Frisk */
            position: absolute;
            top: 230px;
            left: 310px;
        }
        .instructions {
            margin-top: 20px;
            text-align: center;
        }
    </style>
</head>
<body>

    <h1>Welcome to the Underground</h1>
    
    <div id="game-container">
        <div id="player"></div>
    </div>

    <div class="instructions">
        <p>Use the <b>Arrow Keys</b> to move.</p>
    </div>

    <script>
        // Game State
        const player = document.getElementById('player');
        const container = document.getElementById('game-container');
        
        let x = 310;
        let y = 230;
        const speed = 4;
        
        // Track which keys are pressed
        const keys = {
            ArrowUp: false,
            ArrowDown: false,
            ArrowLeft: false,
            ArrowRight: false
        };

        // Listen for key presses
        window.addEventListener('keydown', (e) => {
            if (keys.hasOwnProperty(e.key)) {
                keys[e.key] = true;
                e.preventDefault(); // Stop window from scrolling
            }
        });

        window.addEventListener('keyup', (e) => {
            if (keys.hasOwnProperty(e.key)) {
                keys[e.key] = false;
            }
        });

        // The Game Loop
        function update() {
            // Movement Logic
            if (keys.ArrowUp) y -= speed;
            if (keys.ArrowDown) y += speed;
            if (keys.ArrowLeft) x -= speed;
            if (keys.ArrowRight) x += speed;

            // Collision with walls (keep player inside the box)
            if (x < 0) x = 0;
            if (y < 0) y = 0;
            if (x > container.clientWidth - player.clientWidth) x = container.clientWidth - player.clientWidth;
            if (y > container.clientHeight - player.clientHeight) y = container.clientHeight - player.clientHeight;

            // Update player position on screen
            player.style.left = x + 'px';
            player.style.top = y + 'px';

            // Loop the update function
            requestAnimationFrame(update);
        }

        // Start the game
        update();
    </script>

</body>
</html>
