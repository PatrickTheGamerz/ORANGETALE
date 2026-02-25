<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>UNDERTALE: The Genocide Clicker</title>
    <style>
        @font-face {
            font-family: 'Determination';
            src: url('https://fonts.cdnfonts.com/s/19732/DeterminationMono.woff') format('woff');
        }

        body {
            background-color: #000;
            color: #fff;
            font-family: 'Determination', monospace;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            overflow: hidden;
            user-select: none;
        }

        #game-container {
            width: 900px;
            height: 650px;
            border: 6px solid #fff;
            display: grid;
            grid-template-columns: 300px 1fr;
            background: #000;
        }

        /* Sidebar Stats */
        #sidebar {
            border-right: 4px solid #fff;
            padding: 15px;
            display: flex;
            flex-direction: column;
            background: #050505;
        }

        .stat-group { margin-bottom: 20px; border-bottom: 2px solid #333; padding-bottom: 10px; }
        .lv-text { color: #ff0000; font-size: 28px; }
        .g-text { color: #ffff00; font-size: 22px; }
        .reset-text { color: #00ffff; font-size: 18px; }

        /* Battle Screen */
        #battle-screen {
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            overflow: hidden;
        }

        #monster-sprite-box {
            height: 200px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 20px;
            text-align: center;
        }

        /* The Monster Soul */
        #soul-target {
            width: 100px;
            height: 100px;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: crosshair;
            position: relative;
            margin: 40px 0;
        }

        #soul {
            font-size: 80px;
            color: #fff;
            transform: rotate(180deg);
            text-shadow: 0 0 10px #fff;
            transition: transform 0.1s;
        }

        /* UI Bars */
        .hp-bar-outer { width: 400px; height: 30px; background: #c00; border: 3px solid #fff; position: relative; }
        #hp-bar-inner { width: 100%; height: 100%; background: #ff0; transition: width 0.1s; }
        #hp-text { position: absolute; width: 100%; text-align: center; color: #000; font-weight: bold; line-height: 30px; }

        /* Menu Tabs */
        #menu-box {
            margin-top: auto;
            height: 250px;
            width: 100%;
            border-top: 4px solid #fff;
            display: flex;
            flex-direction: column;
        }

        .tabs { display: flex; border-bottom: 2px solid #fff; }
        .tab { flex: 1; padding: 10px; text-align: center; cursor: pointer; border-right: 1px solid #fff; }
        .tab:hover, .tab.active { background: #fff; color: #000; }

        #tab-content { padding: 10px; overflow-y: auto; flex-grow: 1; }

        .upgrade-card {
            border: 1px solid #555;
            padding: 8px;
            margin-bottom: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .upgrade-card:hover { border-color: #fff; }
        .buy-btn { background: #000; color: #fff; border: 1px solid #fff; padding: 5px 10px; cursor: pointer; font-family: inherit; }
        .buy-btn:disabled { color: #444; border-color: #444; }

        /* Projectiles & Damage */
        .pellet { position: absolute; width: 12px; height: 12px; background: #fff; border-radius: 2px; z-index: 5; }
        .dmg-num {
            position: absolute;
            color: #ffff00;
            font-size: 30px;
            font-weight: bold;
            pointer-events: none;
            animation: dmgRise 0.7s forwards;
            z-index: 10;
        }

        @keyframes dmgRise {
            0% { transform: translateY(0); opacity: 1; }
            100% { transform: translateY(-80px); opacity: 0; }
        }
    </style>
</head>
<body>

<div id="game-container">
    <div id="sidebar">
        <div class="stat-group">
            <div class="lv-text">LV <span id="val-lv">1</span></div>
            <div>EXP: <span id="val-exp">0</span> / <span id="val-next-exp">50</span></div>
            <div class="reset-text">Resets: <span id="val-resets">0</span></div>
        </div>
        <div class="stat-group">
            <div class="g-text">GOLD: <span id="val-gold">0</span></div>
        </div>
        <div id="active-allies" style="font-size: 14px;">
            Characters: <br><span id="ally-list">None</span>
        </div>
        <button onclick="manualSave()" style="margin-top:auto; background:none; color:white; border:1px solid white; cursor:pointer">SAVE FILE</button>
        <button id="reset-btn" onclick="triggerReset()" style="display:none; margin-top:10px; border-color: cyan; color: cyan;">[RESET WORLD]</button>
    </div>

    <div id="battle-screen">
        <div id="monster-sprite-box">
            <h2 id="monster-name">* Whimsun</h2>
        </div>

        <div id="soul-target" onclick="clickAttack(event)">
            <div id="soul">❤</div>
        </div>

        <div class="hp-bar-outer">
            <div id="hp-bar-inner"></div>
            <div id="hp-text">10 / 10</div>
        </div>

        <div id="menu-box">
            <div class="tabs">
                <div class="tab active" onclick="switchTab('upgrades')">CHARACTERS</div>
                <div class="tab" onclick="switchTab('weapons')">WEAPONS</div>
                <div class="tab" onclick="switchTab('skills')">SKILL TREE</div>
            </div>
            <div id="tab-content"></div>
        </div>
    </div>
</div>

<script>
    // --- DATABASE ---
    const REGIONS = [
        { name: "RUINS", monsters: [
            { name: "Whimsun", hp: 10, def: 0, gold: 2, exp: 4 },
            { name: "Froggit", hp: 30, def: 2, gold: 5, exp: 10 },
            { name: "Loox", hp: 45, def: 5, gold: 10, exp: 15 }
        ]},
        { name: "SNOWDIN", monsters: [
            { name: "Snowdrake", hp: 80, def: 10, gold: 20, exp: 40 },
            { name: "Greater Dog", hp: 200, def: 15, gold: 50, exp: 100 }
        ]},
        { name: "WATERFALL", monsters: [
            { name: "Aaron", hp: 400, def: 20, gold: 100, exp: 250 },
            { name: "Undyne (Shadow)", hp: 1200, def: 30, gold: 500, exp: 1000 }
        ]},
        { name: "HOTLAND/CORE", monsters: [
            { name: "Madjick", hp: 2500, def: 40, gold: 800, exp: 3000 },
            { name: "Mettaton NEO", hp: 1, def: -100, gold: 5000, exp: 20000 }
        ]}
    ];

    const SANS_BOSS = { name: "SANS", hp: 1, def: 9999, gold: 0, exp: 50000, isSans: true };

    const WEAPONS = [
        { name: "Stick", atk: 0, price: 0, lv: 1 },
        { name: "Toy Knife", atk: 5, price: 50, lv: 1 },
        { name: "Tough Glove", atk: 12, price: 500, lv: 4 },
        { name: "Ballet Shoes", atk: 25, price: 2500, lv: 8 },
        { name: "Real Knife", atk: 99, price: 50000, lv: 18 }
    ];

    // --- GAME STATE ---
    let state = {
        lv: 1, exp: 0, gold: 0, resets: 0,
        weapon: "Stick", weaponAtk: 0,
        mHp: 10, mMaxHp: 10, mName: "Whimsun", mDef: 0,
        currentRegion: 0,
        allies: [],
        skills: [],
        currentTab: 'upgrades',
        isShattering: false
    };

    // --- INITIALIZATION ---
    function init() {
        loadGame();
        updateUI();
        gameLoop();
        spawnMonster();
    }

    function spawnMonster() {
        if (state.lv >= 19) {
            setMonster(SANS_BOSS);
        } else {
            let regIdx = Math.min(REGIONS.length - 1, Math.floor(state.lv / 5));
            let pool = REGIONS[regIdx].monsters;
            setMonster(pool[Math.floor(Math.random() * pool.length)]);
        }
    }

    function setMonster(m) {
        state.mName = m.name;
        state.mMaxHp = m.hp;
        state.mHp = m.hp;
        state.mDef = m.def;
        state.isShattering = false;
        document.getElementById('soul').style.opacity = "1";
    }

    // --- COMBAT ---
    function clickAttack(e) {
        if (state.isShattering) return;
        
        let dmg = (state.lv * 2) + state.weaponAtk - state.mDef;
        // Sans Logic: Miss 95% of time unless specific triggers (simplified clicker logic)
        if (state.mName === "SANS" && Math.random() > 0.05) dmg = 0; 
        if (dmg < 1 && state.mName !== "SANS") dmg = 1;

        dealDamage(dmg, e.clientX, e.clientY);
    }

    function dealDamage(dmg, x, y) {
        if (dmg === 0 && state.mName === "SANS") {
            createDmgNum("MISS", x, y, "#888");
            return;
        }

        state.mHp -= dmg;
        createDmgNum(dmg, x, y, "#ffff00");

        if (state.mHp <= 0) {
            shatter();
        }
        updateUI();
    }

    function shatter() {
        state.isShattering = true;
        document.getElementById('soul').style.opacity = "0";
        
        // Find reward
        let expGain = 0;
        let goldGain = 0;

        if (state.mName === "SANS") {
            expGain = 50000;
            state.lv = 20;
            document.getElementById('reset-btn').style.display = "block";
        } else {
            let m = findMonsterData(state.mName);
            expGain = m.exp;
            goldGain = m.gold;
        }

        state.exp += expGain;
        state.gold += goldGain;
        
        checkLevelUp();
        setTimeout(spawnMonster, 800);
    }

    function checkLevelUp() {
        let req = state.lv * state.lv * 50;
        if (state.exp >= req && state.lv < 20) {
            state.lv++;
            updateUI();
        }
    }

    // --- SKILL TREE & ALLIES ---
    const ALLY_DATA = {
        'Flowey': { cost: 20, lv: 1, desc: "Shoots pellets every 3s" },
        'Toriel': { cost: 500, lv: 5, desc: "Fireballs deal high burst" },
        'Papyrus': { cost: 2000, lv: 8, desc: "Bones deal constant DPS" },
        'Undyne': { cost: 10000, lv: 12, desc: "Spears ignore 50% DEF" },
        'Sans': { cost: 100000, lv: 19, desc: "Gaster Blasters deal massive DMG" }
    };

    const SKILLS = [
        { id: 'f_pellet3', name: 'Triple Pellets', cost: 100, parent: 'Flowey', desc: "Flowey fires 3 pellets" },
        { id: 'f_circle', name: 'Pellet Circle', cost: 1000, parent: 'Flowey', desc: "Surrounds soul (leaves 1 HP)" },
        { id: 'f_omega', name: 'OMEGA FLOWEY', cost: 50000, parent: 'Flowey', resets: 1, desc: "The ultimate form." }
    ];

    function buyAlly(name) {
        let d = ALLY_DATA[name];
        if (state.gold >= d.cost && state.lv >= d.lv && !state.allies.includes(name)) {
            state.gold -= d.cost;
            state.allies.push(name);
            renderTab();
            updateUI();
        }
    }

    function buySkill(s) {
        if (state.gold >= s.cost && !state.skills.includes(s.id)) {
            state.gold -= s.cost;
            state.skills.push(s.id);
            renderTab();
        }
    }

    // --- TICK LOGIC (PROJECTILES) ---
    function gameLoop() {
        if (!state.isShattering && state.mHp > 0) {
            // Flowey Logic
            if (state.allies.includes('Flowey')) {
                if (Math.random() < 0.01) { // Random frequency
                    if (state.skills.includes('f_circle')) {
                        spawnPelletCircle();
                    } else {
                        spawnPellet(state.skills.includes('f_pellet3') ? 3 : 1);
                    }
                }
            }
        }
        requestAnimationFrame(gameLoop);
    }

    function spawnPellet(count) {
        for(let i=0; i<count; i++) {
            const p = document.createElement('div');
            p.className = 'pellet';
            p.style.left = Math.random() * 400 + "px";
            p.style.top = "-10px";
            document.getElementById('battle-screen').appendChild(p);
            
            p.animate([
                { top: '-10px' },
                { top: '250px', left: '200px' }
            ], 1500).onfinish = () => {
                p.remove();
                dealDamage(2 + state.lv, 450, 300);
            };
        }
    }

    function spawnPelletCircle() {
        const center = { x: 200, y: 250 };
        for(let i=0; i<8; i++) {
            const p = document.createElement('div');
            p.className = 'pellet';
            let angle = (i / 8) * Math.PI * 2;
            let startDist = 150;
            p.style.left = (center.x + Math.cos(angle) * startDist) + "px";
            p.style.top = (center.y + Math.sin(angle) * startDist) + "px";
            document.getElementById('battle-screen').appendChild(p);

            p.animate([
                { opacity: 1 },
                { left: center.x + 'px', top: center.y + 'px' }
            ], 2000).onfinish = () => {
                p.remove();
                // "Leave 1 HP" Logic
                let d = 10;
                if (state.mHp - d < 1) d = Math.max(0, state.mHp - 1);
                dealDamage(Math.floor(d), 450, 300);
            };
        }
    }

    // --- UI ENGINE ---
    function switchTab(t) {
        state.currentTab = t;
        document.querySelectorAll('.tab').forEach(el => el.classList.remove('active'));
        event.target.classList.add('active');
        renderTab();
    }

    function renderTab() {
        const cont = document.getElementById('tab-content');
        cont.innerHTML = '';
        
        if (state.currentTab === 'upgrades') {
            for (let name in ALLY_DATA) {
                let d = ALLY_DATA[name];
                let owned = state.allies.includes(name);
                cont.innerHTML += `
                    <div class="upgrade-card">
                        <div><b>${name}</b><br><small>${d.desc}</small></div>
                        <button class="buy-btn" ${ (state.gold < d.cost || state.lv < d.lv || owned) ? 'disabled' : '' } onclick="buyAlly('${name}')">
                            ${owned ? 'RECRUITED' : d.cost + ' G'}
                        </button>
                    </div>`;
            }
        } else if (state.currentTab === 'skills') {
            SKILLS.forEach(s => {
                let hasParent = state.allies.includes(s.parent);
                let owned = state.skills.includes(s.id);
                let resetReq = s.resets ? (state.resets >= s.resets) : true;
                cont.innerHTML += `
                    <div class="upgrade-card" style="opacity: ${hasParent ? 1 : 0.4}">
                        <div><b>${s.name}</b><br><small>${s.desc}</small></div>
                        <button class="buy-btn" ${ (!hasParent || state.gold < s.cost || owned || !resetReq) ? 'disabled' : '' } onclick="buySkill(${JSON.stringify(s).replace(/"/g, '&quot;')})">
                            ${owned ? 'OWNED' : (resetReq ? s.cost + ' G' : 'Needs Reset')}
                        </button>
                    </div>`;
            });
        } else if (state.currentTab === 'weapons') {
            WEAPONS.forEach(w => {
                let active = state.weapon === w.name;
                cont.innerHTML += `
                    <div class="upgrade-card">
                        <div><b>${w.name}</b> (ATK +${w.atk})</div>
                        <button class="buy-btn" ${ (state.gold < w.price || active || state.lv < w.lv) ? 'disabled' : '' } onclick="buyWeapon('${w.name}', ${w.atk}, ${w.price})">
                            ${active ? 'EQUIPPED' : w.price + ' G'}
                        </button>
                    </div>`;
            });
        }
    }

    function buyWeapon(name, atk, price) {
        state.gold -= price;
        state.weapon = name;
        state.weaponAtk = atk;
        renderTab();
        updateUI();
    }

    function updateUI() {
        document.getElementById('val-lv').innerText = state.lv;
        document.getElementById('val-exp').innerText = state.exp;
        document.getElementById('val-next-exp').innerText = state.lv * state.lv * 50;
        document.getElementById('val-gold').innerText = Math.floor(state.gold);
        document.getElementById('val-resets').innerText = state.resets;
        document.getElementById('ally-list').innerText = state.allies.length ? state.allies.join(', ') : "None";
        
        document.getElementById('monster-name').innerText = "* " + state.mName;
        let p = (state.mHp / state.mMaxHp) * 100;
        document.getElementById('hp-bar-inner').style.width = Math.max(0, p) + "%";
        document.getElementById('hp-text').innerText = Math.max(0, Math.floor(state.mHp)) + " / " + state.mMaxHp;
    }

    function createDmgNum(txt, x, y, color) {
        const d = document.createElement('div');
        d.className = 'dmg-num';
        d.innerText = txt;
        d.style.left = (x - 20) + "px";
        d.style.top = (y - 50) + "px";
        d.style.color = color;
        document.body.appendChild(d);
        setTimeout(() => d.remove(), 700);
    }

    function findMonsterData(name) {
        for(let r of REGIONS) {
            let m = r.monsters.find(x => x.name === name);
            if(m) return m;
        }
        return {exp: 0, gold: 0};
    }

    // --- RESET SYSTEM ---
    function triggerReset() {
        if(confirm("Erase this world and move on?")) {
            state.resets++;
            state.lv = 1;
            state.exp = 0;
            state.gold = 0;
            state.allies = [];
            state.skills = [];
            state.weapon = "Stick";
            state.weaponAtk = 0;
            document.getElementById('reset-btn').style.display = "none";
            spawnMonster();
            updateUI();
            manualSave();
        }
    }

    // --- STORAGE ---
    function manualSave() {
        localStorage.setItem('ut_genocide_save', JSON.stringify(state));
    }
    function loadGame() {
        const s = localStorage.getItem('ut_genocide_save');
        if (s) Object.assign(state, JSON.parse(s));
    }
    setInterval(manualSave, 10000); // Auto-save 10s

    init();
    renderTab();
</script>

</body>
</html>
