<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>UNDERTALE: Eternal Genocide</title>
    <style>
        @font-face {
            font-family: 'Determination';
            src: url('https://fonts.cdnfonts.com/s/19732/DeterminationMono.woff') format('woff');
        }

        :root {
            --ut-red: #ff0000;
            --ut-yellow: #ffff00;
            --ut-white: #ffffff;
            --ut-bg: #000000;
            --ut-border: #ffffff;
            --panel-bg: rgba(20, 20, 20, 0.95);
        }

        * { box-sizing: border-box; }

        body {
            background-color: var(--ut-bg);
            color: var(--ut-white);
            font-family: 'Determination', 'Courier New', monospace;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            user-select: none;
        }

        /* CRT Effect Overlay */
        #crt::before {
            content: " ";
            display: block;
            position: absolute;
            top: 0; left: 0; bottom: 0; right: 0;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.25) 50%), linear-gradient(90deg, rgba(255, 0, 0, 0.06), rgba(0, 255, 0, 0.02), rgba(0, 0, 255, 0.06));
            z-index: 100;
            background-size: 100% 4px, 3px 100%;
            pointer-events: none;
        }

        #game-wrapper {
            width: 1100px;
            height: 750px;
            border: 6px solid var(--ut-border);
            display: grid;
            grid-template-columns: 320px 1fr;
            grid-template-rows: 1fr 280px;
            position: relative;
            background: #000;
        }

        /* SIDEBAR */
        #sidebar {
            grid-row: 1 / 3;
            border-right: 6px solid var(--ut-border);
            padding: 25px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            background: #050505;
        }

        .stat-box { margin-bottom: 15px; }
        .stat-label { font-size: 16px; color: #888; text-transform: uppercase; }
        .stat-val { font-size: 32px; display: block; }
        .love-val { color: var(--ut-red); }
        .gold-val { color: var(--ut-yellow); }

        /* BATTLE FIELD */
        #battle-field {
            grid-column: 2;
            grid-row: 1;
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background: url('https://www.transparenttextures.com/patterns/black-linen.png');
        }

        #monster-sprite-container {
            height: 220px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            margin-bottom: 20px;
        }

        #monster-name { font-size: 32px; margin: 0; }
        #monster-status { font-size: 18px; color: #888; }

        #soul-area {
            width: 150px;
            height: 150px;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            position: relative;
        }

        #soul {
            font-size: 80px;
            color: var(--ut-white);
            transform: rotate(180deg);
            transition: transform 0.1s, opacity 0.2s;
            filter: drop-shadow(0 0 5px rgba(255,255,255,0.5));
        }

        #soul.dodge { animation: soul-dodge 0.15s ease-in-out; }
        @keyframes soul-dodge {
            0% { transform: rotate(180deg) translateX(0); }
            25% { transform: rotate(180deg) translateX(-60px); }
            75% { transform: rotate(180deg) translateX(60px); }
            100% { transform: rotate(180deg) translateX(0); }
        }

        /* HUD BARS */
        .hud-bar-container {
            width: 400px;
            height: 25px;
            background: #400;
            border: 3px solid var(--ut-white);
            position: relative;
            margin: 10px 0;
        }
        #hp-bar { width: 100%; height: 100%; background: var(--ut-yellow); transition: width 0.1s; }
        #stamina-bar { width: 100%; height: 100%; background: #00ffff; transition: width 0.3s; }
        .hud-text {
            position: absolute; width: 100%; text-align: center; 
            font-size: 16px; font-weight: bold; line-height: 19px; color: #000;
        }

        /* MENU SYSTEM */
        #menu-area {
            grid-column: 2;
            grid-row: 2;
            border-top: 6px solid var(--ut-border);
            display: flex;
            background: #000;
        }

        .tab-sidebar {
            width: 180px;
            border-right: 4px solid var(--ut-border);
            display: flex;
            flex-direction: column;
        }

        .tab-btn {
            background: none;
            border: none;
            color: var(--ut-white);
            padding: 15px;
            font-family: inherit;
            font-size: 20px;
            cursor: pointer;
            text-align: left;
            transition: 0.2s;
        }
        .tab-btn:hover, .tab-btn.active { background: var(--ut-white); color: #000; }

        #tab-content {
            flex-grow: 1;
            padding: 20px;
            overflow-y: auto;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            align-content: start;
        }

        /* CARDS */
        .card {
            border: 2px solid #444;
            padding: 12px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #0a0a0a;
        }
        .card:hover { border-color: var(--ut-white); }
        .card-info b { font-size: 18px; display: block; }
        .card-info small { color: #888; }
        
        .buy-btn {
            background: #000;
            color: var(--ut-white);
            border: 2px solid var(--ut-white);
            padding: 8px 12px;
            font-family: inherit;
            cursor: pointer;
        }
        .buy-btn:disabled { color: #444; border-color: #444; cursor: not-allowed; }

        /* POPUPS */
        .dmg-popup {
            position: absolute;
            font-size: 38px;
            color: var(--ut-yellow);
            font-weight: bold;
            pointer-events: none;
            animation: rise-fade 0.8s forwards;
            z-index: 50;
        }
        @keyframes rise-fade {
            0% { transform: translateY(0); opacity: 1; }
            100% { transform: translateY(-120px); opacity: 0; }
        }

        .projectile {
            position: absolute;
            pointer-events: none;
            z-index: 10;
        }

        /* RESET SCREEN */
        #reset-overlay {
            position: absolute;
            top:0; left:0; width:100%; height:100%;
            background: #000;
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        .reset-msg { color: var(--ut-red); font-size: 40px; margin-bottom: 40px; }
    </style>
</head>
<body id="crt">

<div id="game-wrapper">
    <div id="sidebar">
        <div class="stat-box">
            <span class="stat-label">Location</span>
            <span class="stat-val" id="ui-region">RUINS</span>
        </div>
        <div class="stat-box">
            <span class="stat-label love">LOVE</span>
            <span class="stat-val love-val" id="ui-lv">1</span>
        </div>
        <div class="stat-box">
            <span class="stat-label">EXP</span>
            <span class="stat-val" id="ui-exp">0 / 50</span>
        </div>
        <div class="stat-box">
            <span class="stat-label gold">Gold</span>
            <span class="stat-val gold-val" id="ui-gold">0</span>
        </div>
        <div class="stat-box">
            <span class="stat-label">Resets</span>
            <span class="stat-val" id="ui-resets">0</span>
        </div>
        
        <div style="margin-top: auto;">
            <button class="buy-btn" style="width:100%; margin-bottom:10px" onclick="saveGame()">SAVE FILE</button>
            <button id="reset-trigger" class="buy-btn" style="width:100%; color:cyan; border-color:cyan; display:none;" onclick="startResetSequence()">RESET WORLD</button>
        </div>
    </div>

    <div id="battle-field">
        <div id="monster-sprite-container">
            <h1 id="monster-name">FROGGIT</h1>
            <span id="monster-status">* Ribbit, ribbit.</span>
        </div>

        <div id="soul-area" onclick="handleSoulClick(event)">
            <div id="soul">❤</div>
        </div>

        <div class="hud-bar-container">
            <div id="hp-bar"></div>
            <div class="hud-text" id="hp-text">HP: 30 / 30</div>
        </div>
        <div class="hud-bar-container" id="stamina-box" style="display:none; border-color: #0ff;">
            <div id="stamina-bar"></div>
            <div class="hud-text" style="color: #000">STAMINA</div>
        </div>
    </div>

    <div id="menu-area">
        <div class="tab-sidebar">
            <button class="tab-btn active" onclick="changeTab('recruits')">RECRUITS</button>
            <button class="tab-btn" onclick="changeTab('skills')">SKILLS</button>
            <button class="tab-btn" onclick="changeTab('items')">ITEMS</button>
        </div>
        <div id="tab-content">
            </div>
    </div>

    <div id="reset-overlay">
        <div class="reset-msg">THE WORLD WAS ERASED.</div>
        <button class="buy-btn" style="font-size: 24px;" onclick="finishReset()">START OVER</button>
    </div>
</div>

<script>
/**
 * UNDERTALE CLICKER ENGINE
 * STATE & DATA
 */

const DATA = {
    regions: [
        { name: "RUINS", killsNeeded: 10, boss: { name: "TORIEL", hp: 500, def: 5, gold: 100, exp: 200 } },
        { name: "SNOWDIN", killsNeeded: 15, boss: { name: "PAPYRUS", hp: 1200, def: 10, gold: 300, exp: 500 } },
        { name: "WATERFALL", killsNeeded: 20, boss: { name: "UNDYNE", hp: 2500, def: 20, gold: 1000, exp: 2000 } },
        { name: "HOTLAND", killsNeeded: 25, boss: { name: "METTATON NEO", hp: 1, def: -100, gold: 2000, exp: 10000 } },
        { name: "CORE", killsNeeded: 30, boss: { name: "ASGORE", hp: 5000, def: 40, gold: 5000, exp: 25000 } },
        { name: "JUDGEMENT", killsNeeded: 1, boss: { name: "SANS", hp: 1, def: 999, gold: 0, exp: 100000, isSans: true } }
    ],
    monsters: {
        RUINS: [{n: "Froggit", hp: 30, def: 2}, {n: "Whimsun", hp: 10, def: 0}, {n: "Loox", hp: 50, def: 4}],
        SNOWDIN: [{n: "Snowdrake", hp: 100, def: 10}, {n: "Lesser Dog", hp: 150, def: 8}, {n: "Jerry", hp: 200, def: 30}],
        WATERFALL: [{n: "Aaron", hp: 400, def: 15}, {n: "Woshua", hp: 350, def: 12}, {n: "Temmie", hp: 5, def: 0}],
        HOTLAND: [{n: "Vulkin", hp: 800, def: 20}, {n: "Tsunderplane", hp: 1200, def: 35}],
        CORE: [{n: "Knight Knight", hp: 2000, def: 50}, {n: "Madjick", hp: 1800, def: 40}]
    },
    recruits: [
        { id: 'flowey', name: 'Flowey', cost: 15, lvReq: 1, desc: "Shoots friendliness pellets." },
        { id: 'toriel', name: 'Toriel', cost: 250, lvReq: 3, desc: "Fires flame spirals." },
        { id: 'napsta', name: 'Napstablook', cost: 800, lvReq: 5, desc: "Cries acid rain." },
        { id: 'papyrus', name: 'Papyrus', cost: 2000, lvReq: 8, desc: "Spawns bone fields." },
        { id: 'undyne', name: 'Undyne', cost: 7000, lvReq: 12, desc: "Spears ignore 50% Defense." },
        { id: 'muffet', name: 'Muffet', cost: 15000, lvReq: 14, desc: "Spiders steal gold & dmg." },
        { id: 'mtt', name: 'Mettaton', cost: 40000, lvReq: 16, desc: "Bombs deal area damage." },
        { id: 'asgore', name: 'Asgore', cost: 100000, lvReq: 19, desc: "Trident sweeps hit hard." }
    ],
    items: [
        { id: 'stick', name: 'Stick', atk: 0, cost: 0 },
        { id: 'knife', name: 'Toy Knife', atk: 5, cost: 100 },
        { id: 'glove', name: 'Tough Glove', atk: 12, cost: 1000 },
        { id: 'shoes', name: 'Ballet Shoes', atk: 25, cost: 5000 },
        { id: 'apron', name: 'Stained Apron', atk: 40, cost: 15000 },
        { id: 'real_knife', name: 'Real Knife', atk: 99, cost: 100000 }
    ]
};

let state = {
    lv: 1, exp: 0, gold: 0, resets: 0,
    regionIdx: 0, killCount: 0,
    mHp: 30, mMaxHp: 30, mName: "Froggit", mDef: 2, mIsBoss: false,
    weapon: 'stick',
    ownedRecruits: [],
    skills: {}, // e.g., { flowey: 1 }
    currentTab: 'recruits',
    sansStamina: 100,
    isShattering: false
};

/**
 * CORE LOGIC
 */

function init() {
    loadGame();
    spawnMonster();
    renderAll();
    
    // Auto-attack loop
    setInterval(() => {
        if (!state.isShattering && state.mHp > 0) {
            processRecruitAttacks();
        }
    }, 1000);

    // Save loop
    setInterval(saveGame, 30000);
}

function spawnMonster() {
    state.isShattering = false;
    document.getElementById('soul').style.opacity = 1;
    document.getElementById('stamina-box').style.display = 'none';

    const reg = DATA.regions[state.regionIdx];
    
    if (state.killCount >= reg.killsNeeded) {
        // BOSS TIME
        let boss = reg.boss;
        state.mName = boss.name;
        state.mMaxHp = boss.hp;
        state.mHp = boss.hp;
        state.mDef = boss.def;
        state.mIsBoss = true;
        
        if (boss.isSans) {
            state.sansStamina = 100;
            document.getElementById('stamina-box').style.display = 'block';
            document.getElementById('monster-status').innerText = "* You feel like you're going to have a bad time.";
        } else {
            document.getElementById('monster-status').innerText = "* A formidable presence blocks the way.";
        }
    } else {
        // Trash mob
        const pool = DATA.monsters[reg.name];
        const m = pool[Math.floor(Math.random() * pool.length)];
        state.mName = m.n;
        state.mMaxHp = m.hp + (state.resets * 100);
        state.mHp = state.mMaxHp;
        state.mDef = m.def + (state.resets * 5);
        state.mIsBoss = false;
        document.getElementById('monster-status').innerText = "* Monster approaches!";
    }
    updateUI();
}

function handleSoulClick(e) {
    if (state.isShattering) return;

    const boss = DATA.regions[state.regionIdx].boss;
    
    // Sans Exhaustion Mechanic
    if (state.mIsBoss && boss.isSans && state.sansStamina > 0) {
        state.sansStamina -= 1;
        document.getElementById('soul').classList.add('dodge');
        setTimeout(() => document.getElementById('soul').classList.remove('dodge'), 150);
        createPopup("MISS", e.clientX, e.clientY);
        updateUI();
        return;
    }

    // Normal Damage
    const w = DATA.items.find(i => i.id === state.weapon);
    let dmg = (state.lv * 2) + w.atk - state.mDef;
    if (dmg < 1) dmg = 1;

    applyDamage(dmg, e.clientX, e.clientY);
}

function applyDamage(dmg, x, y) {
    state.mHp -= dmg;
    createPopup(dmg, x, y);
    if (state.mHp <= 0) shatterMonster();
    updateUI();
}

function shatterMonster() {
    state.isShattering = true;
    document.getElementById('soul').style.opacity = 0;

    // Calculate Rewards
    const reg = DATA.regions[state.regionIdx];
    let expGain, goldGain;

    if (state.mIsBoss) {
        expGain = reg.boss.exp;
        goldGain = reg.boss.gold;
        state.killCount = 0;
        
        if (reg.boss.isSans) {
            document.getElementById('reset-trigger').style.display = 'block';
        } else {
            state.regionIdx++;
        }
    } else {
        expGain = 5 + (state.regionIdx * 5);
        goldGain = 2 + (state.regionIdx * 3);
        state.killCount++;
    }

    state.exp += expGain;
    state.gold += goldGain;
    checkLevelUp();

    setTimeout(spawnMonster, 1000);
}

function checkLevelUp() {
    let req = state.lv * 50 * state.lv;
    if (state.exp >= req && state.lv < 20) {
        state.lv++;
        updateUI();
        renderTab(); // Re-render to check requirements
    }
}

/**
 * RECRUIT SYSTEM
 */

function processRecruitAttacks() {
    const sBox = document.getElementById('soul-area').getBoundingClientRect();
    const tx = sBox.left + 75;
    const ty = sBox.top + 75;

    state.ownedRecruits.forEach(rid => {
        const char = DATA.recruits.find(r => r.id === rid);
        const skillLv = state.skills[rid] || 1;
        
        let dmg = (char.cost / 10) * skillLv;
        
        // Visual Projectile
        spawnProjectile(rid, tx, ty, dmg);
    });
}

function spawnProjectile(type, tx, ty, dmg) {
    const p = document.createElement('div');
    p.className = 'projectile';
    
    // Style based on type
    if(type === 'flowey') { p.innerText = '•'; p.style.color = 'white'; }
    else if(type === 'toriel') { p.innerText = '🔥'; }
    else if(type === 'papyrus') { p.innerHTML = '<div style="width:4px;height:20px;background:white"></div>'; }
    else { p.innerText = '✦'; p.style.color = 'yellow'; }

    p.style.left = (Math.random() * 1000) + 'px';
    p.style.top = '-50px';
    document.getElementById('battle-field').appendChild(p);

    p.animate([
        { top: '-50px' },
        { top: ty + 'px', left: tx + 'px' }
    ], 800).onfinish = () => {
        p.remove();
        if(!state.isShattering) applyDamage(Math.floor(dmg), tx, ty);
    };
}

/**
 * UI & TABS
 */

function changeTab(tab) {
    state.currentTab = tab;
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    event.target.classList.add('active');
    renderTab();
}

function renderTab() {
    const container = document.getElementById('tab-content');
    container.innerHTML = '';

    if (state.currentTab === 'recruits') {
        DATA.recruits.forEach(r => {
            const owned = state.ownedRecruits.includes(r.id);
            const canAfford = state.gold >= r.cost && state.lv >= r.lvReq;
            container.innerHTML += `
                <div class="card">
                    <div class="card-info">
                        <b>${r.name}</b>
                        <small>Req: LV ${r.lvReq}</small>
                    </div>
                    <button class="buy-btn" ${ (owned || !canAfford) ? 'disabled' : ''} onclick="buyRecruit('${r.id}', ${r.cost})">
                        ${owned ? 'RECRUITED' : r.cost + 'G'}
                    </button>
                </div>
            `;
        });
    } else if (state.currentTab === 'skills') {
        if (state.ownedRecruits.length === 0) {
            container.innerHTML = "<p>Recruit monsters to unlock their skill trees.</p>";
            return;
        }
        state.ownedRecruits.forEach(rid => {
            const char = DATA.recruits.find(r => r.id === rid);
            const lv = state.skills[rid] || 1;
            const upCost = Math.floor(char.cost * lv * 1.5);
            container.innerHTML += `
                <div class="card">
                    <div class="card-info">
                        <b>${char.name} LV ${lv}</b>
                        <small>Next: +50% ATK</small>
                    </div>
                    <button class="buy-btn" ${state.gold < upCost ? 'disabled' : ''} onclick="upgradeSkill('${rid}', ${upCost})">
                        UPGRADE (${upCost}G)
                    </button>
                </div>
            `;
        });
    } else if (state.currentTab === 'items') {
        DATA.items.forEach(i => {
            const isEquipped = state.weapon === i.id;
            const canAfford = state.gold >= i.cost;
            container.innerHTML += `
                <div class="card">
                    <div class="card-info">
                        <b>${i.name}</b>
                        <small>ATK +${i.atk}</small>
                    </div>
                    <button class="buy-btn" ${ (isEquipped || !canAfford) ? 'disabled' : ''} onclick="buyItem('${i.id}', ${i.cost})">
                        ${isEquipped ? 'EQUIPPED' : i.cost + 'G'}
                    </button>
                </div>
            `;
        });
    }
}

function buyRecruit(id, cost) {
    state.gold -= cost;
    state.ownedRecruits.push(id);
    state.skills[id] = 1;
    renderAll();
}

function upgradeSkill(id, cost) {
    state.gold -= cost;
    state.skills[id]++;
    renderAll();
}

function buyItem(id, cost) {
    state.gold -= cost;
    state.weapon = id;
    renderAll();
}

function updateUI() {
    document.getElementById('ui-region').innerText = DATA.regions[state.regionIdx].name;
    document.getElementById('ui-lv').innerText = state.lv;
    document.getElementById('ui-exp').innerText = `${state.exp} / ${state.lv * 50 * state.lv}`;
    document.getElementById('ui-gold').innerText = Math.floor(state.gold);
    document.getElementById('ui-resets').innerText = state.resets;

    document.getElementById('monster-name').innerText = state.mName.toUpperCase();
    const hpPerc = (state.mHp / state.mMaxHp) * 100;
    document.getElementById('hp-bar').style.width = Math.max(0, hpPerc) + '%';
    document.getElementById('hp-text').innerText = `HP: ${Math.max(0, Math.floor(state.mHp))} / ${state.mMaxHp}`;

    if (state.mIsBoss && DATA.regions[state.regionIdx].boss.isSans) {
        document.getElementById('stamina-bar').style.width = state.sansStamina + '%';
    }
}

function createPopup(txt, x, y) {
    const p = document.createElement('div');
    p.className = 'dmg-popup';
    p.innerText = txt;
    p.style.left = x + 'px';
    p.style.top = y + 'px';
    document.body.appendChild(p);
    setTimeout(() => p.remove(), 800);
}

function renderAll() {
    updateUI();
    renderTab();
}

/**
 * PERSISTENCE & RESET
 */

function saveGame() {
    localStorage.setItem('ut_eternal_genocide', JSON.stringify(state));
}

function loadGame() {
    const s = localStorage.getItem('ut_eternal_genocide');
    if (s) state = Object.assign(state, JSON.parse(s));
}

function startResetSequence() {
    document.getElementById('reset-overlay').style.display = 'flex';
}

function finishReset() {
    state.resets++;
    state.lv = 1;
    state.exp = 0;
    state.gold = 0;
    state.regionIdx = 0;
    state.killCount = 0;
    state.ownedRecruits = [];
    state.skills = {};
    state.weapon = 'stick';
    document.getElementById('reset-overlay').style.display = 'none';
    document.getElementById('reset-trigger').style.display = 'none';
    spawnMonster();
    renderAll();
    saveGame();
}

// Start
init();

</script>
</body>
</html>
