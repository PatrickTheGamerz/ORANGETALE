<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UNDERTALE: Soul Crusher</title>
    <style>
        @font-face {
            font-family: 'Determination';
            src: url('https://fonts.cdnfonts.com/s/19732/DeterminationMono.woff') format('woff');
        }

        body {
            background-color: #000;
            color: #fff;
            font-family: 'Determination', 'Courier New', monospace;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
        }

        #game-layout {
            display: grid;
            grid-template-columns: 300px 600px;
            grid-gap: 20px;
            border: 4px solid #fff;
            padding: 20px;
            background: #000;
        }

        /* Battle Area */
        #battle-screen {
            border: 4px solid #fff;
            height: 400px;
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }

        #soul {
            font-size: 80px;
            color: #fff;
            cursor: pointer;
            user-select: none;
            transition: transform 0.05s;
            filter: drop-shadow(0 0 10px #fff);
        }

        #soul.shattering { animation: shatter 0.5s forwards; pointer-events: none; }

        @keyframes shatter {
            0% { transform: scale(1) rotate(0deg); opacity: 1; }
            50% { transform: scale(1.2) rotate(10deg); opacity: 0.5; }
            100% { transform: scale(0) rotate(-20deg); opacity: 0; }
        }

        .hp-bar-bg { width: 200px; height: 20px; background: #444; margin-top: 20px; border: 2px solid #fff; }
        #hp-fill { width: 100%; height: 100%; background: #ff0000; transition: width 0.1s; }

        /* Stats & Menu */
        #menu-side {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .stat-line { font-size: 20px; margin-bottom: 5px; }
        .gold { color: #ffff00; }
        .love { color: #ff0000; }

        .tab-content {
            border: 2px solid #fff;
            height: 250px;
            overflow-y: auto;
            padding: 10px;
            background: #111;
        }

        .upgrade-item {
            border: 1px solid #555;
            padding: 8px;
            margin-bottom: 8px;
            cursor: pointer;
            font-size: 14px;
        }

        .upgrade-item:hover { background: #222; border-color: #fff; }
        .upgrade-item.locked { color: #444; border-color: #333; cursor: not-allowed; }

        .dmg-text {
            position: absolute;
            color: #ff0000;
            font-weight: bold;
            pointer-events: none;
            animation: floatUp 0.6s ease-out forwards;
        }

        @keyframes floatUp {
            0% { opacity: 1; transform: translateY(0); }
            100% { opacity: 0; transform: translateY(-50px); }
        }
    </style>
</head>
<body>

<div id="game-layout">
    <div id="menu-side">
        <div class="stat-line love">LV <span id="lv-val">1</span></div>
        <div class="stat-line">EXP: <span id="exp-val">0</span> / <span id="next-lv">50</span></div>
        <div class="stat-line gold">G: <span id="gold-val">0</span></div>
        
        <div style="border-bottom: 2px solid #fff; margin: 10px 0;">RECRUIT (UPGRADES)</div>
        <div class="tab-content" id="shop-list">
            </div>
        <button onclick="saveGame()" style="background:#000; color:#fff; border:1px solid #fff; cursor:pointer;">SAVE DATA</button>
    </div>

    <div id="battle-screen">
        <div id="soul-name">* Monster Soul</div>
        <div id="soul" onclick="dealDamage(event)">❤</div>
        <div class="hp-bar-bg"><div id="hp-fill"></div></div>
        <div id="hp-text">HP: 10 / 10</div>
    </div>
</div>

<script>
    let state = {
        lv: 1,
        exp: 0,
        gold: 0,
        clickDmg: 1,
        autoDmg: 0,
        monsterMaxHp: 10,
        monsterHp: 10,
        kills: 0
    };

    const upgrades = [
        { id: 'flowey', name: 'Flowey', cost: 15, lvReq: 1, dps: 1, desc: 'Petals deal 1 DPS' },
        { id: 'papyrus', name: 'Papyrus', cost: 100, lvReq: 3, dps: 5, desc: 'Blue Bones deal 5 DPS' },
        { id: 'undyne', name: 'Undyne', cost: 500, lvReq: 7, dps: 20, desc: 'Spears deal 20 DPS' },
        { id: 'sans', name: 'Sans', cost: 2500, lvReq: 15, dps: 100, desc: 'Gaster Blasters deal 100 DPS' }
    ];

    let purchased = {};

    function dealDamage(e) {
        if (state.monsterHp <= 0) return;

        spawnDmgText(e.clientX, e.clientY, state.clickDmg);
        state.monsterHp -= state.clickDmg;
        
        if (state.monsterHp <= 0) {
            killMonster();
        }
        updateUI();
    }

    function spawnDmgText(x, y, amt) {
        const div = document.createElement('div');
        div.className = 'dmg-text';
        div.style.left = (x - 20) + 'px';
        div.style.top = (y - 20) + 'px';
        div.innerText = amt;
        document.body.appendChild(div);
        setTimeout(() => div.remove(), 600);
    }

    function killMonster() {
        const soul = document.getElementById('soul');
        soul.classList.add('shattering');
        
        // Rewards
        const rewardG = 5 + (state.lv * 2);
        const rewardExp = 10 + (state.lv * 5);
        state.gold += rewardG;
        state.exp += rewardExp;
        state.kills++;

        // Level Up Logic
        let nextLvExp = state.lv * 50 * state.lv;
        if (state.exp >= nextLvExp) {
            state.lv++;
            state.clickDmg += 2;
        }

        setTimeout(() => {
            state.monsterMaxHp = Math.floor(10 * Math.pow(1.3, state.kills));
            state.monsterHp = state.monsterMaxHp;
            soul.classList.remove('shattering');
            updateUI();
        }, 600);
    }

    function buyUpgrade(idx) {
        const upg = upgrades[idx];
        if (state.gold >= upg.cost && state.lv >= upg.lvReq) {
            state.gold -= upg.cost;
            state.autoDmg += upg.dps;
            upg.cost = Math.floor(upg.cost * 1.5);
            updateUI();
        }
    }

    function updateUI() {
        document.getElementById('lv-val').innerText = state.lv;
        document.getElementById('exp-val').innerText = state.exp;
        document.getElementById('next-lv').innerText = state.lv * 50 * state.lv;
        document.getElementById('gold-val').innerText = state.gold;
        
        const hpPercent = (state.monsterHp / state.monsterMaxHp) * 100;
        document.getElementById('hp-fill').style.width = Math.max(0, hpPercent) + '%';
        document.getElementById('hp-text').innerText = `HP: ${Math.max(0, state.monsterHp)} / ${state.monsterMaxHp}`;

        // Render Shop
        const shop = document.getElementById('shop-list');
        shop.innerHTML = '';
        upgrades.forEach((upg, i) => {
            const isLocked = state.lv < upg.lvReq;
            const div = document.createElement('div');
            div.className = `upgrade-item ${isLocked ? 'locked' : ''}`;
            div.innerHTML = `
                <b>${isLocked ? '???' : upg.name}</b> [Cost: ${upg.cost}G]<br>
                <small>${isLocked ? 'Requires LV ' + upg.lvReq : upg.desc}</small>
            `;
            if (!isLocked) div.onclick = () => buyUpgrade(i);
            shop.appendChild(div);
        });
    }

    // Auto Damage Loop
    setInterval(() => {
        if (state.autoDmg > 0 && state.monsterHp > 0) {
            state.monsterHp -= (state.autoDmg / 10);
            if (state.monsterHp <= 0) killMonster();
            updateUI();
        }
    }, 100);

    function saveGame() {
        localStorage.setItem('ut_clicker_save', JSON.stringify(state));
        alert("Progress Saved to the Void.");
    }

    // Init
    const saved = localStorage.getItem('ut_clicker_save');
    if (saved) state = JSON.parse(saved);
    updateUI();
</script>

</body>
</html>
