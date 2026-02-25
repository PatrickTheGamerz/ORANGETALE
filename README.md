<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Undertale Clicker: Gain LOVE</title>
    <style>
        body {
            background-color: #000;
            color: #fff;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 20px;
            user-select: none; /* Prevents text highlighting when clicking fast */
        }
        
        #stats-box {
            border: 4px solid #fff;
            padding: 20px;
            width: 400px;
            text-align: center;
            margin-bottom: 30px;
        }

        h1 { margin: 0 0 10px 0; font-size: 24px; }
        .stat { font-size: 18px; font-weight: bold; margin: 5px 0; }

        /* The Clickable Soul */
        #soul-container {
            height: 150px;
            display: flex;
            justify-content: center;
            align-items: center;
            margin-bottom: 30px;
        }

        #monster-soul {
            font-size: 100px;
            color: #fff;
            transform: rotate(180deg); /* Upside down heart = monster soul */
            cursor: pointer;
            transition: transform 0.05s;
            text-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
        }

        #monster-soul:active {
            transform: rotate(180deg) scale(0.85); /* Squish effect when clicked */
            color: #ccc;
        }

        /* Upgrades Section */
        #upgrades-container {
            border: 4px solid #fff;
            width: 400px;
            padding: 20px;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .upgrade-btn {
            background-color: #000;
            color: #fff;
            border: 2px solid #fff;
            padding: 10px;
            font-family: 'Courier New', Courier, monospace;
            cursor: pointer;
            text-align: left;
            display: flex;
            justify-content: space-between;
            font-size: 16px;
            display: none; /* Hidden by default for the 'tree' effect */
        }

        .upgrade-btn:hover:not(:disabled) {
            background-color: #333;
        }

        .upgrade-btn:disabled {
            border-color: #555;
            color: #555;
            cursor: not-allowed;
        }

        .system-buttons {
            margin-top: 20px;
            display: flex;
            gap: 10px;
        }

        .sys-btn {
            background: none;
            color: #fff;
            border: 1px solid #fff;
            font-family: inherit;
            cursor: pointer;
            padding: 5px 10px;
        }
        
        .sys-btn:hover { background: #fff; color: #000; }
    </style>
</head>
<body>

    <div id="stats-box">
        <h1>* You feel your sins crawling on your back.</h1>
        <div class="stat">LOVE: <span id="love-display">0</span></div>
        <div class="stat">LOVE per click: <span id="click-power-display">1</span></div>
        <div class="stat">LOVE per sec: <span id="auto-power-display">0</span></div>
    </div>

    <div id="soul-container">
        <div id="monster-soul">♥</div>
    </div>

    <div id="upgrades-container">
        <h2 style="margin-top:0; border-bottom: 2px solid #fff; padding-bottom:5px;">Upgrades Tree</h2>
        
        <button class="upgrade-btn" id="upg-stick" onclick="buyUpgrade('stick')">
            <span>Stick (Auto +1/s)</span>
            <span>Cost: <span id="cost-stick">10</span></span>
        </button>

        <button class="upgrade-btn" id="upg-toyknife" onclick="buyUpgrade('toyknife')">
            <span>Toy Knife (Click +1)</span>
            <span>Cost: <span id="cost-toyknife">50</span></span>
        </button>

        <button class="upgrade-btn" id="upg-glove" onclick="buyUpgrade('glove')">
            <span>Tough Glove (Auto +5/s)</span>
            <span>Cost: <span id="cost-glove">200</span></span>
        </button>

        <button class="upgrade-btn" id="upg-shoes" onclick="buyUpgrade('shoes')">
            <span>Ballet Shoes (Click +5)</span>
            <span>Cost: <span id="cost-shoes">500</span></span>
        </button>
    </div>

    <div class="system-buttons">
        <button class="sys-btn" onclick="saveGame()">Save Game</button>
        <button class="sys-btn" onclick="hardReset()" style="border-color: red; color: red;">Erase Save</button>
    </div>

    <script>
        // --- GAME STATE ---
        let game = {
            love: 0,
            clickPower: 1,
            autoPower: 0,
            upgrades: {
                stick: { cost: 10, count: 0, autoAdd: 1, clickAdd: 0, req: 0 },
                toyknife: { cost: 50, count: 0, autoAdd: 0, clickAdd: 1, req: 10 },
                glove: { cost: 200, count: 0, autoAdd: 5, clickAdd: 0, req: 100 },
                shoes: { cost: 500, count: 0, autoAdd: 0, clickAdd: 5, req: 300 }
            },
            totalLoveGathered: 0 // Used to unlock the tree
        };

        // --- LOAD GAME ---
        function loadGame() {
            const savedData = localStorage.getItem('undertaleSave');
            if (savedData) {
                // Merge saved data with base game object to prevent missing data errors
                game = { ...game, ...JSON.parse(savedData) };
            }
            updateUI();
        }

        // --- SAVE GAME ---
        function saveGame() {
            localStorage.setItem('undertaleSave', JSON.stringify(game));
            console.log("Game Saved!");
        }

        // --- HARD RESET ---
        function hardReset() {
            if(confirm("* Are you sure you want to ERASE this world? All progress will be lost forever.")) {
                localStorage.removeItem('undertaleSave');
                location.reload();
            }
        }

        // --- CORE MECHANICS ---
        // Clicking the soul
        document.getElementById('monster-soul').addEventListener('mousedown', () => {
            game.love += game.clickPower;
            game.totalLoveGathered += game.clickPower;
            updateUI();
        });

        // Buying an upgrade
        function buyUpgrade(id) {
            let upg = game.upgrades[id];
            if (game.love >= upg.cost) {
                game.love -= upg.cost;
                upg.count++;
                
                // Apply effects
                game.autoPower += upg.autoAdd;
                game.clickPower += upg.clickAdd;
                
                // Increase cost by 15% each time
                upg.cost = Math.ceil(upg.cost * 1.15);
                
                updateUI();
            }
        }

        // --- UI UPDATES ---
        function updateUI() {
            // Update Text
            document.getElementById('love-display').innerText = Math.floor(game.love);
            document.getElementById('click-power-display').innerText = game.clickPower;
            document.getElementById('auto-power-display').innerText = game.autoPower;

            // Loop through all upgrades to update costs, buttons, and "Tree" visibility
            for (const [id, upg] of Object.entries(game.upgrades)) {
                let btn = document.getElementById('upg-' + id);
                let costText = document.getElementById('cost-' + id);
                
                // Update cost text
                costText.innerText = upg.cost;

                // Disable button if not enough LOVE
                btn.disabled = game.love < upg.cost;

                // TREE LOGIC: Only show upgrade if total LOVE gathered meets the requirement
                if (game.totalLoveGathered >= upg.req) {
                    btn.style.display = "flex";
                }
            }
        }

        // --- GAME LOOP ---
        // Runs every second to add Auto-LOVE
        setInterval(() => {
            if (game.autoPower > 0) {
                game.love += game.autoPower;
                game.totalLoveGathered += game.autoPower;
                updateUI();
            }
        }, 1000);

        // Auto-save every 10 seconds
        setInterval(saveGame, 10000);

        // Start the game by loading and doing an initial UI update
        loadGame();
    </script>

</body>
</html>
