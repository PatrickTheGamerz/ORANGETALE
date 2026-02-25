<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UNDERTALE: The Last Corridor Clicker</title>
    <style>
        @font-face {
            font-family: 'Determination';
            src: url('https://fonts.cdnfonts.com/s/19732/DeterminationMono.woff') format('woff');
        }

        body {
            background-color: #000;
            color: #fff;
            font-family: 'Determination', monospace;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            user-select: none;
        }

        #game-ui {
            width: 1000px;
            height: 700px;
            border: 4px solid #fff;
            display: grid;
            grid-template-columns: 280px 1fr;
            background: #000;
            position: relative;
        }

        /* Sidebar */
        #sidebar {
            border-right: 4px solid #fff;
            padding: 20px;
            background: #080808;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .stat-label { font-size: 18px; color: #aaa; }
        .stat-val { font-size: 24px; color: #fff; margin-bottom: 10px; }
        .gold { color: #ffff00; }
        .love { color: #ff0000; }

        /* Battle Field */
        #battle-field {
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background-image: radial-gradient(circle, #111 1px, transparent 1px);
            background-size: 30px 30px;
        }

        #monster-area {
            height: 250px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            text-align: center;
        }

        #soul-box {
            width: 120px;
            height: 120px;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            position: relative;
            z-index: 10;
        }

        #soul {
            font-size: 70px;
            color: #fff;
            transform: rotate(180deg); /* Monster Soul */
            transition: transform 0.05s, opacity 0.2s;
        }

        #soul.dodging {
            animation: dodge 0.2s infinite;
            color: #888;
        }

        @keyframes dodge {
            0% { transform: rotate(180deg) translateX(0); }
            50% { transform: rotate(180deg) translateX(50px); }
            100% { transform: rotate(180deg) translateX(-50px); }
        }

        /* Menu */
        #menu-container {
            position: absolute;
            bottom: 0;
            right: 0;
            width: 716px;
            height: 220px;
            border-top: 4px solid #fff;
            background: #000;
            display: flex;
        }

        .tab-list {
            width: 140px;
            border-right: 4px solid #fff;
            display: flex;
            flex-direction: column;
        }

        .tab-item {
            padding: 12px;
            border: none;
            background: none;
            color: #fff;
            font-family: inherit;
            font-size: 16px;
            cursor: pointer;
            text-align: left;
        }

        .tab-item.active { background: #fff; color: #000; }

        #tab-view {
            flex-grow: 1;
            padding: 15px;
            overflow-y: auto;
        }

        /* UI Elements */
        .hp-bar-bg { width: 300px; height: 20px; background: #440000; border: 2px solid #fff; margin: 10px 0; }
        #hp-fill { width: 100%; height: 100%; background: #ff0; }

        .item-card {
            border: 1px solid #444;
            padding: 10px;
            margin-bottom: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .buy-btn { background: #000; color: #fff; border: 1px solid #fff; padding: 5px; cursor: pointer; }
        .buy-btn:disabled { color: #444; border-color: #444; }

        /* Projectiles */
        .proj { position: absolute; pointer-events: none; z-index: 5; }
        .pellet { width: 10px; height: 10px; background: #fff; border-radius: 2px; }
        .fireball { width: 20px; height: 20px; background: #ff6600; border-radius: 50%; box-shadow: 0 0 10px #ff0; }
        .bone { width: 8px; height: 40px; background: #fff; border-radius: 4px; }

        .dmg-popup {
            position: absolute;
            color: #ffff00;
            font-size: 32px;
            font-weight: bold;
            pointer-events: none;
            animation: riseFade 0.8s forwards;
            z-index: 100;
        }

        @keyframes riseFade {
            0% { transform: translateY(0); opacity: 1; }
            100% { transform: translateY(-100px); opacity: 0; }
        }

        #reset-zone {
            margin-top: auto;
            border-top: 2px solid #333;
            padding-top: 10px;
        }

        .reset-btn {
            width: 100%;
            padding: 10px;
            background: #000;
            color: #f00;
            border: 2px solid #f00;
            font-family: inherit;
            cursor: pointer;
            display: none;
        }
        
        .reset-btn:hover { background: #f00; color: #000; }
    </style>
</head>
<body>

<div id="game-ui">
    <div id="sidebar">
        <div class="stat-label">AREA</div>
        <div class="stat-val" id="area-name">RUINS</div>

        <div class="stat-label love">LOVE</div>
        <div class="stat-val" id="lv-val">1</div>

        <div class="stat-label">EXP</div>
        <div class="stat-val" id="exp-val">0</div>

        <div class="stat-label gold">GOLD</div>
        <div class="stat-val" id="gold-val">0</div>

        <div id="reset-zone">
            <div class="stat-label">RESETS: <span id="reset-val">0</span></div>
            <button id="main-reset-btn" class="reset-btn" onclick="performReset()">[ RESET WORLD ]</button>
        </div>
    </div>

    <div id="battle-field">
        <div id="monster-area">
            <h2 id="m-name">* Froggit</h2>
            <div class="hp-bar-bg"><div id="hp-fill"></div></div>
            <div id="m-hp-text">30 / 30</div>
            <div id="sans-status" style="color: cyan; font-size: 14px; display:none;">Dodges remaining: 100</div>
        </div>

        <div id="soul-box" onclick="handleManualClick(event)">
            <div id="soul">❤</div>
        </div>

        <div id="menu-container">
            <div class="tab-list">
                <button class="tab-item active" onclick="setTab('chars')">CHARACTERS</button>
                <button class="tab-item" onclick="setTab('skills')">SKILL TREE</button>
                <button class="tab-item" onclick="setTab('weapons')">EQUIP</button>
            </div>
            <div id="tab-view"></div>
        </div>
    </div>
</div>

<script>
    // --- DATABASE & BALANCING ---
    const CONFIG = {
        regions: ["RUINS", "SNOWDIN", "WATERFALL", "HOTLAND", "CORE", "JUDGEMENT"],
        bosses: {
            "RUINS": { name: "Toriel", hp: 400, def: 5, gold: 50, exp: 100 },
            "SNOWDIN": { name: "Papyrus", hp: 800, def: 10, gold: 150, exp: 300 },
            "WATERFALL": { name: "Undyne", hp: 1500, def: 20, gold: 500, exp: 1000 },
            "HOTLAND": { name: "Mettaton NEO", hp: 1, def: -100, gold: 1000, exp: 5000 },
            "JUDGEMENT": { name: "SANS", hp: 1, def: 999, gold: 0, exp: 10000, isSans: true }
        }
    };

    const WEAPONS = [
        { name: "Stick", atk: 0, price: 0 },
        { name: "Toy Knife", atk: 4, price: 50 },
        { name: "Tough Glove", atk: 10, price: 200 },
        { name: "Ballet Shoes", atk: 22, price: 800 },
        { name: "Real Knife", atk: 99, price: 5000 }
    ];

    let state = {
        lv: 1, exp: 0, gold: 0, resets: 0,
        regionIdx: 0, killCount: 0,
        weapon: WEAPONS[0],
        mHp: 30, mMaxHp: 30, mName: "Froggit", mDef: 2, mIsBoss: false,
        sansDodges: 100,
        chars: {
            flowey: { owned: false, level: 1, cost: 20 },
            toriel: { owned: false, level: 1, cost: 300 },
            papyrus: { owned: false, level: 1, cost: 1000 }
        },
        activeTab: 'chars'
    };

    // --- ENGINE CORE ---
    function init() {
        load();
        spawnRandom();
        setInterval(gameTick, 1000); // For auto-attacks
        setInterval(save, 10000);
        render();
    }

    function spawnRandom() {
        state.mIsBoss = false;
        document.getElementById('sans-status').style.display = "none";
        
        // If ready for boss
        if (state.killCount >= 10) {
            let boss = CONFIG.bosses[CONFIG.regions[state.regionIdx]];
            state.mName = boss.name;
            state.mMaxHp = (state.resets >= 2 && boss.name === "Undyne") ? 10000 : boss.hp;
            if (state.resets >= 2 && boss.name === "Undyne") state.mName = "Undyne the Undying";
            state.mHp = state.mMaxHp;
            state.mDef = boss.def;
            state.mIsBoss = true;
            if (boss.isSans) {
                state.sansDodges = 100;
                document.getElementById('sans-status').style.display = "block";
            }
        } else {
            // Normal trash mobs
            const mobs = [
                { name: "Whimsun", hp: 10, def: 0 },
                { name: "Froggit", hp: 30, def: 2 },
                { name: "Moldsmal", hp: 20, def: 0 }
            ];
            let m = mobs[Math.floor(Math.random() * mobs.length)];
            state.mName = m.name;
            state.mMaxHp = m.hp + (state.regionIdx * 100);
            state.mHp = state.mMaxHp;
            state.mDef = m.def + (state.regionIdx * 5);
        }
        document.getElementById('soul').style.opacity = 1;
        updateUI();
    }

    function handleManualClick(e) {
        if (state.mHp <= 0) return;

        // Sans Logic
        if (state.mName === "SANS" && state.sansDodges > 0) {
            state.sansDodges--;
            createPopup("MISS", e.clientX, e.clientY);
            document.getElementById('soul').classList.add('dodging');
            setTimeout(() => document.getElementById('soul').classList.remove('dodging'), 100);
            document.getElementById('sans-status').innerText = `Dodges remaining: ${state.sansDodges}`;
            return;
        }

        let dmg = (state.lv * 3) + state.weapon.atk - state.mDef;
        if (dmg < 1) dmg = 1;
        applyDamage(dmg, e.clientX, e.clientY);
    }

    function applyDamage(dmg, x, y) {
        state.mHp -= dmg;
        createPopup(dmg, x, y);
        if (state.mHp <= 0) die();
        updateUI();
    }

    function die() {
        document.getElementById('soul').style.opacity = 0;
        
        // Rewards
        let goldReward = state.mIsBoss ? 100 : 5;
        let expReward = state.mIsBoss ? 500 : 20;
        
        state.gold += goldReward;
        state.exp += expReward;
        
        if (state.mIsBoss) {
            state.killCount = 0;
            state.regionIdx++;
            if (state.regionIdx >= CONFIG.regions.length) {
                state.regionIdx = CONFIG.regions.length - 1;
                document.getElementById('main-reset-btn').style.display = "block";
            }
        } else {
            state.killCount++;
        }

        checkLv();
        setTimeout(spawnRandom, 800);
    }

    function checkLv() {
        let req = state.lv * 100;
        if (state.exp >= req && state.lv < 20) {
            state.lv++;
            updateUI();
        }
    }

    // --- VISUAL ATTACKS ---
    function gameTick() {
        if (state.mHp <= 0) return;

        const sBox = document.getElementById('soul-box').getBoundingClientRect();
        const centerX = sBox.left + 60;
        const centerY = sBox.top + 60;

        if (state.chars.flowey.owned) {
            spawnProjectile('pellet', centerX, centerY, 5 + state.chars.flowey.level);
        }
        if (state.chars.toriel.owned) {
            spawnProjectile('fireball', centerX, centerY, 20 * state.chars.toriel.level);
        }
        if (state.chars.papyrus.owned) {
            spawnProjectile('bone', centerX, centerY, 15 * state.chars.papyrus.level);
        }
    }

    function spawnProjectile(type, tx, ty, dmg) {
        const p = document.createElement('div');
        p.className = `proj ${type}`;
        p.style.left = (Math.random() * 800) + "px";
        p.style.top = "-50px";
        document.getElementById('battle-field').appendChild(p);

        p.animate([
            { top: '-50px' },
            { top: ty + 'px', left: tx + 'px' }
        ], 1000).onfinish = () => {
            p.remove();
            if (state.mHp > 0) applyDamage(dmg, tx, ty);
        };
    }

    // --- UI & TABS ---
    function setTab(t) {
        state.activeTab = t;
        render();
    }

    function render() {
        const view = document.getElementById('tab-view');
        view.innerHTML = '';
        
        document.querySelectorAll('.tab-item').forEach(btn => {
            btn.classList.toggle('active', btn.innerText.toLowerCase().includes(state.activeTab));
        });

        if (state.activeTab === 'chars') {
            for (let id in state.chars) {
                let c = state.chars[id];
                view.innerHTML += `
                    <div class="item-card">
                        <div><b>${id.toUpperCase()}</b><br><small>Level ${c.level}</small></div>
                        <button class="buy-btn" ${state.gold < c.cost || c.owned ? 'disabled' : ''} onclick="buyChar('${id}')">
                            ${c.owned ? 'RECRUITED' : c.cost + 'G'}
                        </button>
                    </div>`;
            }
        } else if (state.activeTab === 'skills') {
            // Simplified skill tree: Upgrading existing characters
            for (let id in state.chars) {
                let c = state.chars[id];
                if (!c.owned) continue;
                let upCost = c.level * 200;
                view.innerHTML += `
                    <div class="item-card">
                        <div><b>${id.toUpperCase()} Training</b><br><small>Power up attacks</small></div>
                        <button class="buy-btn" ${state.gold < upCost ? 'disabled' : ''} onclick="upgradeChar('${id}')">
                            UPGRADE (${upCost}G)
                        </button>
                    </div>`;
            }
        } else if (state.activeTab === 'weapons') {
            WEAPONS.forEach(w => {
                let isEquipped = state.weapon.name === w.name;
                view.innerHTML += `
                    <div class="item-card">
                        <div><b>${w.name}</b><br><small>ATK +${w.atk}</small></div>
                        <button class="buy-btn" ${state.gold < w.price || isEquipped ? 'disabled' : ''} onclick="buyWeapon('${w.name}')">
                            ${isEquipped ? 'EQUIPPED' : w.price + 'G'}
                        </button>
                    </div>`;
            });
        }
    }

    function buyChar(id) {
        if (state.gold >= state.chars[id].cost) {
            state.gold -= state.chars[id].cost;
            state.chars[id].owned = true;
            render();
            updateUI();
        }
    }

    function upgradeChar(id) {
        let cost = state.chars[id].level * 200;
        if (state.gold >= cost) {
            state.gold -= cost;
            state.chars[id].level++;
            render();
            updateUI();
        }
    }

    function buyWeapon(name) {
        let w = WEAPONS.find(x => x.name === name);
        if (state.gold >= w.price) {
            state.gold -= w.price;
            state.weapon = w;
            render();
            updateUI();
        }
    }

    function updateUI() {
        document.getElementById('lv-val').innerText = state.lv;
        document.getElementById('exp-val').innerText = state.exp;
        document.getElementById('gold-val').innerText = state.gold;
        document.getElementById('reset-val').innerText = state.resets;
        document.getElementById('area-name').innerText = CONFIG.regions[state.regionIdx];
        
        document.getElementById('m-name').innerText = "* " + state.mName;
        let p = (state.mHp / state.mMaxHp) * 100;
        document.getElementById('hp-fill').style.width = Math.max(0, p) + "%";
        document.getElementById('m-hp-text').innerText = `${Math.ceil(state.mHp)} / ${state.mMaxHp}`;
    }

    function createPopup(txt, x, y) {
        const d = document.createElement('div');
        d.className = 'dmg-popup';
        d.innerText = txt;
        d.style.left = x + "px";
        d.style.top = y + "px";
        document.body.appendChild(d);
        setTimeout(() => d.remove(), 800);
    }

    function performReset() {
        if (!confirm("Erase this world?")) return;
        state.resets++;
        state.lv = 1;
        state.exp = 0;
        state.gold = 0;
        state.regionIdx = 0;
        state.killCount = 0;
        state.weapon = WEAPONS[0];
        // Keep characters but reset levels? Or full wipe? 
        // Let's keep characters but reset their power for balance.
        for(let id in state.chars) {
            state.chars[id].level = 1;
        }
        document.getElementById('main-reset-btn').style.display = "none";
        spawnRandom();
        updateUI();
        save();
    }

    function save() { localStorage.setItem('ut_clicker_v2', JSON.stringify(state)); }
    function load() {
        let s = localStorage.getItem('ut_clicker_v2');
        if (s) state = Object.assign(state, JSON.parse(s));
    }

    init();
</script>

</body>
</html>
