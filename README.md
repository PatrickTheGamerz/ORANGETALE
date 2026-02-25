<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UNDERTALE: Soul Crusher Pro</title>
    <style>
        @font-face {
            font-family: 'Determination';
            src: url('https://fonts.cdnfonts.com/s/19732/DeterminationMono.woff') format('woff');
        }

        body {
            background-color: #000;
            color: #fff;
            font-family: 'Determination', 'Courier New', monospace;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            overflow: hidden;
            user-select: none;
        }

        /* UI Layout */
        #game-window {
            width: 800px;
            height: 600px;
            border: 4px solid #fff;
            display: flex;
            flex-direction: column;
            position: relative;
            background: #000;
        }

        #top-stats {
            padding: 15px;
            border-bottom: 4px solid #fff;
            display: flex;
            justify-content: space-around;
            font-size: 24px;
        }

        /* Battle Area */
        #battle-area {
            flex-grow: 1;
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background: #000;
        }

        /* The Soul (Monster Soul: White, Upside Down) */
        #soul-container {
            position: relative;
            cursor: crosshair;
        }

        #monster-soul {
            font-size: 80px;
            color: #fff;
            transform: rotate(180deg);
            display: inline-block;
            transition: transform 0.05s;
        }

        .soul-shard {
            position: absolute;
            width: 10px;
            height: 10px;
            background: #fff;
            pointer-events: none;
        }

        /* HP Bar */
        .monster-info { text-align: center; margin-bottom: 20px; font-size: 20px; }
        .hp-container {
            width: 300px;
            height: 25px;
            background: #c00;
            border: 2px solid #fff;
            position: relative;
        }
        #hp-bar { width: 100%; height: 100%; background: #ff0; transition: width 0.2s; }

        /* Menu System */
        #menu-area {
            height: 200px;
            border-top: 4px solid #fff;
            display: flex;
        }

        .tab-buttons {
            width: 150px;
            border-right: 4px solid #fff;
            display: flex;
            flex-direction: column;
        }

        .tab-btn {
            background: #000;
            color: #fff;
            border: none;
            padding: 15px;
            font-family: inherit;
            font-size: 18px;
            cursor: pointer;
            text-align: left;
        }

        .tab-btn:hover, .tab-btn.active { background: #fff; color: #000; }

        .tab-content {
            flex-grow: 1;
            padding: 15px;
            overflow-y: auto;
        }

        .item-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border: 1px solid #333;
            padding: 10px;
            margin-bottom: 5px;
        }

        .item-row:hover { border-color: #fff; }

        .buy-btn {
            background: #000;
            color: #ff0;
            border: 1px solid #ff0;
            padding: 5px 10px;
            cursor: pointer;
            font-family: inherit;
        }

        .buy-btn:disabled { color: #444; border-color: #444; }

        /* Projectiles */
        .pellet {
            position: absolute;
            width: 12px;
            height: 12px;
            background: #fff;
            border-radius: 2px;
            pointer-events: none;
        }

        .damage-popup {
            position: absolute;
            color: #ff0;
            font-weight: bold;
            font-size: 24px;
            animation: floatDamage 0.8s ease-out forwards;
            pointer-events: none;
            z-index: 10;
        }

        @keyframes floatDamage {
            0% { transform: translateY(0); opacity: 1; }
            100% { transform: translateY(-60px); opacity: 0; }
        }
    </style>
</head>
<body>

<div id="game-window">
    <div id="top-stats">
        <span>LV <span id="lv">1</span></span>
        <span>EXP <span id="exp">0</span></span>
        <span>G <span id="gold">0</span></span>
        <span style="color: #ff0;">Weapon: <span id="weapon-name">Stick</span></span>
    </div>

    <div id="battle-area">
        <div class="monster-info">
            <div id="monster-name">* Whimsun</div>
            <div class="hp-container"><div id="hp-bar"></div></div>
            <div id="hp-text">HP: 10 / 10</div>
        </div>

        <div id="soul-container" onclick="playerAttack(event)">
            <div id="monster-soul">❤</div>
        </div>
    </div>

    <div id="menu-area">
        <div class="tab-buttons">
            <button class="tab-btn active" onclick="openTab('weapons')">WEAPONS</button>
            <button class="tab-btn" onclick="openTab('allies')">ALLIES</button>
            <button class="tab-btn" onclick="openTab('skills')">SKILLS</button>
        </div>
        <div id="weapons" class="tab-content"></div>
        <div id="allies" class="tab-content" style="display:none"></div>
        <div id="skills" class="tab-content" style="display:none"></div>
    </div>
</div>

<script>
    // --- DATABASE ---
    const MONSTERS = [
        { name: "Whimsun", hp: 10, def: 0, gold: 2, exp: 3, lvReq: 1 },
        { name: "Froggit", hp: 30, def: 2, gold: 4, exp: 10, lvReq: 1 },
        { name: "Moldsmal", hp: 50, def: 0, gold: 6, exp: 15, lvReq: 2 },
        { name: "Loox", hp: 50, def: 6, gold: 12, exp: 20, lvReq: 3 },
        { name: "Vegetoid", hp: 70, def: 0, gold: 15, exp: 25, lvReq: 4 },
        { name: "Greater Dog", hp: 200, def: 10, gold: 50, exp: 100, lvReq: 7 }
    ];

    const WEAPONS = [
        { id: 'stick', name: 'Stick', atk: 0, cost: 0, lvReq: 1 },
        { id: 'knife', name: 'Toy Knife', atk: 3, cost: 20, lvReq: 1 },
        { id: 'glove', name: 'Tough Glove', atk: 7, cost: 100, lvReq: 3 },
        { id: 'shoes', name: 'Ballet Shoes', atk: 12, cost: 400, lvReq: 6 },
        { id: 'notebook', name: 'Torn Notebook', atk: 15, cost: 1000, lvReq: 10 }
    ];

    const ALLIES = {
        flowey: { 
            name: "Flowey", 
            cost: 50, 
            lvReq: 2, 
            bought: false, 
            level: 1, 
            cooldown: 3000, 
            lastShot: 0,
            desc: "Shoots friendliness pellets." 
        }
    };

    // --- STATE ---
    let state = {
        lv: 1, exp: 0, gold: 0,
        weapon: WEAPONS[0],
        monster: null,
        monsterHp: 0,
        unlockedWeapons: ['stick'],
        activeTab: 'weapons'
    };

    // --- CORE ENGINE ---
    function init() {
        spawnMonster();
        renderShop();
        gameLoop();
    }

    function spawnMonster() {
        const possible = MONSTERS.filter(m => state.lv >= m.lvReq);
        state.monster = possible[Math.floor(Math.random() * possible.length)];
        state.monsterHp = state.monster.hp;
        document.getElementById('monster-soul').style.opacity = '1';
        updateUI();
    }

    function playerAttack(e) {
        if (state.monsterHp <= 0) return;
        
        // Damage = (LV + WeaponATK) - MonsterDEF
        let dmg = (state.lv + state.weapon.atk) - state.monster.def;
        if (dmg < 1) dmg = 1;

        applyDamage(dmg, e.clientX, e.clientY);
    }

    function applyDamage(dmg, x, y) {
        state.monsterHp -= dmg;
        createDamagePopup(dmg, x, y);
        
        if (state.monsterHp <= 0) {
            shatterSoul();
        }
        updateUI();
    }

    function shatterSoul() {
        const soul = document.getElementById('monster-soul');
        const container = document.getElementById('soul-container');
        soul.style.opacity = '0';

        // Rewards
        state.gold += state.monster.gold;
        state.exp += state.monster.exp;
        checkLevelUp();

        // Particle effect
        for(let i=0; i<8; i++) {
            const shard = document.createElement('div');
            shard.className = 'soul-shard';
            shard.style.left = '40px';
            shard.style.top = '40px';
            container.appendChild(shard);

            const angle = (Math.PI * 2 / 8) * i;
            const velocity = 5 + Math.random() * 5;
            let sx = Math.cos(angle) * velocity;
            let sy = Math.sin(angle) * velocity;

            let posX = 40;
            let posY = 40;

            const anim = setInterval(() => {
                posX += sx;
                posY += sy;
                sy += 0.5; // Gravity
                shard.style.left = posX + 'px';
                shard.style.top = posY + 'px';
                if (posY > 300) {
                    shard.remove();
                    clearInterval(anim);
                }
            }, 30);
        }

        setTimeout(spawnMonster, 1000);
    }

    function gameLoop() {
        const now = Date.now();

        // Flowey Attack Logic
        if (ALLIES.flowey.bought) {
            let cd = ALLIES.flowey.level === 1 ? 3000 : 1500;
            if (now - ALLIES.flowey.lastShot > cd && state.monsterHp > 0) {
                shootPellet(ALLIES.flowey.level);
                ALLIES.flowey.lastShot = now;
            }
        }

        requestAnimationFrame(gameLoop);
    }

    function shootPellet(level) {
        const count = level === 1 ? 1 : 5;
        const rect = document.getElementById('soul-container').getBoundingClientRect();
        
        for (let i = 0; i < count; i++) {
            const pellet = document.createElement('div');
            pellet.className = 'pellet';
            // Start from random edges
            pellet.style.left = (Math.random() * 600) + 'px';
            pellet.style.top = "-20px";
            document.getElementById('battle-area').appendChild(pellet);

            // Move towards soul
            const targetX = rect.left + 40 - document.getElementById('battle-area').offsetLeft;
            const targetY = rect.top + 40 - document.getElementById('battle-area').offsetTop;
            
            pellet.animate([
                { top: '-20px' },
                { top: targetY + 'px', left: targetX + 'px' }
            ], { duration: 1000, fill: 'forwards' }).onfinish = () => {
                pellet.remove();
                if (state.monsterHp > 0) applyDamage(2 * level, rect.left, rect.top);
            };
        }
    }

    // --- UI HELPERS ---
    function updateUI() {
        document.getElementById('lv').innerText = state.lv;
        document.getElementById('exp').innerText = state.exp;
        document.getElementById('gold').innerText = state.gold;
        document.getElementById('weapon-name').innerText = state.weapon.name;
        document.getElementById('monster-name').innerText = "* " + state.monster.name;
        
        const perc = (state.monsterHp / state.monster.hp) * 100;
        document.getElementById('hp-bar').style.width = Math.max(0, perc) + "%";
        document.getElementById('hp-text').innerText = `HP: ${Math.max(0, Math.floor(state.monsterHp))} / ${state.monster.hp}`;
    }

    function createDamagePopup(dmg, x, y) {
        const p = document.createElement('div');
        p.className = 'damage-popup';
        p.innerText = dmg;
        p.style.left = x + 'px';
        p.style.top = y + 'px';
        document.body.appendChild(p);
        setTimeout(() => p.remove(), 800);
    }

    function checkLevelUp() {
        const nextExp = state.lv * 100;
        if (state.exp >= nextExp) {
            state.lv++;
            renderShop();
        }
    }

    function openTab(tabId) {
        document.querySelectorAll('.tab-content').forEach(t => t.style.display = 'none');
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
        document.getElementById(tabId).style.display = 'block';
        event.target.classList.add('active');
        state.activeTab = tabId;
        renderShop();
    }

    function renderShop() {
        // Weapons
        const wDiv = document.getElementById('weapons');
        wDiv.innerHTML = '';
        WEAPONS.forEach(w => {
            const isOwned = state.weapon.id === w.id;
            const canAfford = state.gold >= w.cost && state.lv >= w.lvReq;
            wDiv.innerHTML += `
                <div class="item-row">
                    <div><b>${w.name}</b> (ATK: ${w.atk})<br><small>Req: LV ${w.lvReq}</small></div>
                    <button class="buy-btn" ${(!canAfford || isOwned) ? 'disabled' : ''} onclick="buyWeapon('${w.id}')">
                        ${isOwned ? 'EQUIPPED' : w.cost + 'G'}
                    </button>
                </div>`;
        });

        // Allies
        const aDiv = document.getElementById('allies');
        aDiv.innerHTML = '';
        for (let key in ALLIES) {
            const ally = ALLIES[key];
            const canBuy = state.gold >= ally.cost && state.lv >= ally.lvReq && !ally.bought;
            aDiv.innerHTML += `
                <div class="item-row">
                    <div><b>${ally.name}</b><br><small>${ally.desc}</small></div>
                    <button class="buy-btn" ${!canBuy ? 'disabled' : ''} onclick="buyAlly('${key}')">
                        ${ally.bought ? 'RECRUITED' : ally.cost + 'G'}
                    </button>
                </div>`;
        }

        // Skills
        const sDiv = document.getElementById('skills');
        sDiv.innerHTML = '';
        if (ALLIES.flowey.bought) {
            const cost = 500;
            sDiv.innerHTML += `
                <div class="item-row">
                    <div><b>Flowey: Pellet Circle</b><br><small>Fire 5 pellets at once.</small></div>
                    <button class="buy-btn" ${(state.gold < cost || ALLIES.flowey.level > 1) ? 'disabled' : ''} onclick="upgradeFlowey()">
                        ${ALLIES.flowey.level > 1 ? 'MAXED' : cost + 'G'}
                    </button>
                </div>`;
        } else {
            sDiv.innerHTML = "Recruit allies to see their skill trees.";
        }
    }

    function buyWeapon(id) {
        const w = WEAPONS.find(x => x.id === id);
        if (state.gold >= w.cost) {
            state.gold -= w.cost;
            state.weapon = w;
            renderShop();
            updateUI();
        }
    }

    function buyAlly(key) {
        const a = ALLIES[key];
        if (state.gold >= a.cost) {
            state.gold -= a.cost;
            a.bought = true;
            renderShop();
            updateUI();
        }
    }

    function upgradeFlowey() {
        if (state.gold >= 500) {
            state.gold -= 500;
            ALLIES.flowey.level = 2;
            renderShop();
        }
    }

    init();
</script>

</body>
</html>
