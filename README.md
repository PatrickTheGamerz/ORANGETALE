<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TIC-TAC-TOE: COUNCIL OF SOULS</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');
        :root { --red: #ff0000; --gold: #ffcc00; --cyan: #00ffff; --purple: #d946ef; --green: #22c55e; --blue: #3b82f6; }
        
        body { background: #000; color: #fff; font-family: 'Courier Prime', monospace; margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        #layout { display: grid; grid-template-columns: 500px 350px; gap: 20px; padding: 20px; border: 5px double white; }

        /* THE BOARD */
        #game-board { display: grid; grid-template-columns: repeat(3, 1fr); grid-template-rows: repeat(3, 1fr); gap: 10px; width: 300px; height: 300px; margin: 0 auto; }
        .cell { border: 3px solid #fff; display: flex; align-items: center; justify-content: center; font-size: 3rem; cursor: pointer; transition: 0.2s; }
        .cell:hover { background: #222; }
        .cell.x { color: var(--cyan); text-shadow: 0 0 10px var(--cyan); }
        .cell.o { color: var(--red); text-shadow: 0 0 10px var(--red); }

        /* SIDEBAR & CHAT */
        #sidebar { display: flex; flex-direction: column; gap: 10px; }
        #chat-area { flex: 1; border: 1px solid #444; height: 200px; overflow-y: auto; padding: 10px; font-size: 0.85rem; background: #050505; }
        #whisper { font-size: 0.7rem; color: var(--purple); margin-bottom: 5px; font-style: italic; }

        /* INFLUENCE BARS */
        .soul-box { margin-bottom: 10px; padding: 5px; border-left: 4px solid #fff; }
        .bar-bg { width: 100%; height: 8px; background: #222; border: 1px solid #fff; margin-top: 4px; }
        .bar-fill { height: 100%; transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1); }

        .btn { background: #000; border: 2px solid #fff; color: #fff; padding: 5px; cursor: pointer; font-family: inherit; font-size: 0.7rem; margin-top: 5px; }
        .btn:hover { background: #fff; color: #000; }
        
        .glitch { animation: shake 0.1s infinite; color: var(--red) !important; }
        @keyframes shake { 0% { transform: translate(1px); } 50% { transform: translate(-1px); } }
    </style>
</head>
<body>

    <div id="layout">
        <div id="main-side">
            <h2 style="text-align:center; color:var(--gold); margin-top:0;">COUNCIL TTT</h2>
            <div id="game-board">
                <div class="cell" onclick="playerMove(0)"></div>
                <div class="cell" onclick="playerMove(1)"></div>
                <div class="cell" onclick="playerMove(2)"></div>
                <div class="cell" onclick="playerMove(3)"></div>
                <div class="cell" onclick="playerMove(4)"></div>
                <div class="cell" onclick="playerMove(5)"></div>
                <div class="cell" onclick="playerMove(6)"></div>
                <div class="cell" onclick="playerMove(7)"></div>
                <div class="cell" onclick="playerMove(8)"></div>
            </div>
            
            <div style="margin-top:20px; text-align:center;">
                <div id="turn-display">NEXT UP: <span id="current-spirit">ROLLING...</span></div>
                <button class="btn" style="width:150px; margin-top:10px;" onclick="resetBoard()">RESET BOARD</button>
            </div>
        </div>

        <div id="sidebar">
            <div id="whisper">[Whisper]: Frisk is listening...</div>
            <div id="chat-area"></div>
            
            <div id="council-ui">
                <div class="soul-box" style="border-color: var(--cyan);">
                    <div style="display:flex; justify-content:space-between"><b>PLAYER/FRISK</b> <span id="p-val">25%</span></div>
                    <div class="bar-bg"><div id="p-bar" class="bar-fill" style="background:var(--cyan); width:25%"></div></div>
                    <button class="btn" onclick="boost('player')">BOOST INFLUENCE</button>
                </div>
                <div class="soul-box" style="border-color: var(--blue);">
                    <div style="display:flex; justify-content:space-between"><b>SANS</b> <span id="s-val">25%</span></div>
                    <div class="bar-bg"><div id="s-bar" class="bar-fill" style="background:var(--blue); width:25%"></div></div>
                </div>
                <div class="soul-box" style="border-color: #fff;">
                    <div style="display:flex; justify-content:space-between"><b>PAPYRUS</b> <span id="pa-val">25%</span></div>
                    <div class="bar-bg"><div id="pa-bar" class="bar-fill" style="background:#fff; width:25%"></div></div>
                </div>
                <div class="soul-box" style="border-color: var(--red);">
                    <div style="display:flex; justify-content:space-between"><b>CHARA</b> <span id="c-val">25%</span></div>
                    <div class="bar-bg"><div id="c-bar" class="bar-fill" style="background:var(--red); width:25%"></div></div>
                </div>
            </div>
        </div>
    </div>

<script>
/** ⚙️ CORE STATE & PERSISTENCE **/
let souls = {
    player: 25, sans: 25, papy: 25, chara: 25
};
let board = Array(9).fill(null);
let turnSpirit = "player";
let gameActive = true;

function init() {
    // Load Saved State
    const saved = localStorage.getItem('soul_ttt_save');
    if(saved) {
        souls = JSON.parse(saved);
        updateBars();
    }
    
    // Auto-save every 60s
    setInterval(() => {
        localStorage.setItem('soul_ttt_save', JSON.stringify(souls));
        chat("SYSTEM", "Influence levels auto-saved to timeline.");
    }, 60000);

    rollTurn();
    chat("SANS", "hey. nice grid. looks like... a lot of work.");
}

/** 📊 INFLUENCE LOGIC **/
function boost(spirit) {
    const amount = 5;
    const others = Object.keys(souls).filter(k => k !== spirit);
    
    if(souls[spirit] + amount <= 100) {
        souls[spirit] += amount;
        others.forEach(k => {
            souls[k] -= amount / 3;
            if(souls[k] < 0) souls[k] = 0;
        });
    }
    balanceSouls();
    updateBars();
    
    if(souls.player > 60) whisper("You feel your Determination growing.");
    if(souls.chara > 60) chat("CHARA", "Efficiency is increasing.");
}

function balanceSouls() {
    let total = souls.player + souls.sans + souls.papy + souls.chara;
    let ratio = 100 / total;
    souls.player *= ratio;
    souls.sans *= ratio;
    souls.papy *= ratio;
    souls.chara *= ratio;
}

function updateBars() {
    document.getElementById('p-bar').style.width = souls.player + "%";
    document.getElementById('s-bar').style.width = souls.sans + "%";
    document.getElementById('pa-bar').style.width = souls.papy + "%";
    document.getElementById('c-bar').style.width = souls.chara + "%";
    
    document.getElementById('p-val').innerText = Math.round(souls.player) + "%";
    document.getElementById('s-val').innerText = Math.round(souls.sans) + "%";
    document.getElementById('pa-val').innerText = Math.round(souls.papy) + "%";
    document.getElementById('c-val').innerText = Math.round(souls.chara) + "%";
}

/** 🎮 GAMEPLAY & PERSONAS **/
function rollTurn() {
    let rand = Math.random() * 100;
    if(rand < souls.player) turnSpirit = "player";
    else if(rand < souls.player + souls.sans) turnSpirit = "sans";
    else if(rand < souls.player + souls.sans + souls.papy) turnSpirit = "papy";
    else turnSpirit = "chara";

    const display = document.getElementById('current-spirit');
    display.innerText = turnSpirit.toUpperCase();
    display.style.color = getSpiritColor(turnSpirit);
    
    if(turnSpirit !== "player") {
        setTimeout(aiMove, 1000);
    } else {
        chat("SYSTEM", "YOUR TURN. MAKE A MOVE.");
    }
}

function aiMove() {
    if(!gameActive) return;

    let available = board.map((v, i) => v === null ? i : null).filter(v => v !== null);
    if(available.length === 0) return;

    let choice;
    
    // PERSONA BEHAVIORS
    if(turnSpirit === "sans") {
        // 25%+ Skipping turn
        if(souls.sans > 25 && Math.random() > 0.75) {
            chat("SANS", "can't reach the board. i'm napping.");
            rollTurn();
            return;
        }
        choice = available[Math.floor(Math.random() * available.length)];
        chat("SANS", "here. whatever.");
    } 
    else if(turnSpirit === "papy") {
        // Tries to help player (blocks O or plays X)
        choice = available[0]; 
        chat("PAPYRUS", "NYEH HEH HEH! I SHALL ASSIST THE HUMAN!");
    } 
    else if(turnSpirit === "chara") {
        // 70%+ Can Overwrite or Reset
        if(souls.chara > 70 && Math.random() > 0.8) {
            chat("CHARA", "This timeline is flawed. Let's start over.");
            resetBoard();
            return;
        }
        choice = available[available.length - 1]; // Simply greedy for now
        chat("CHARA", "Strike.");
    }

    applyMove(choice, turnSpirit === "papy" ? "X" : "O");
}

function playerMove(i) {
    if(turnSpirit !== "player" || board[i] !== null || !gameActive) return;
    applyMove(i, "X");
}

function applyMove(i, mark) {
    board[i] = mark;
    const cell = document.getElementsByClassName('cell')[i];
    cell.innerText = mark;
    cell.classList.add(mark.toLowerCase());
    
    checkWinner();
    if(gameActive) rollTurn();
}

/** 🏆 LOGIC & META **/
function checkWinner() {
    const wins = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];
    for(let w of wins) {
        if(board[w[0]] && board[w[0]] === board[w[1]] && board[w[0]] === board[w[2]]) {
            gameActive = false;
            let winner = board[w[0]] === "X" ? "PACIFISTS" : "THE FALLEN";
            chat("SYSTEM", "GAME OVER. " + winner + " HAVE PREVAILED.");
            return;
        }
    }
    if(!board.includes(null)) {
        gameActive = false;
        chat("SANS", "it's a draw. guess nobody felt like winning.");
    }
}

function resetBoard() {
    board = Array(9).fill(null);
    const cells = document.getElementsByClassName('cell');
    for(let c of cells) { c.innerText = ""; c.className = "cell"; }
    gameActive = true;
    chat("SYSTEM", "TIMELINE RESET.");
    rollTurn();
}

function chat(who, msg) {
    const area = document.getElementById('chat-area');
    let color = getSpiritColor(who.toLowerCase());
    area.innerHTML += `<div><b style="color:${color}">${who}:</b> ${msg}</div>`;
    area.scrollTop = area.scrollHeight;
}

function whisper(msg) {
    document.getElementById('whisper').innerText = "[Whisper]: " + msg;
}

function getSpiritColor(s) {
    if(s === "player") return "var(--cyan)";
    if(s === "sans") return "var(--blue)";
    if(s === "papy") return "var(--white)";
    if(s === "chara") return "var(--red)";
    return "var(--gold)";
}

init();
</script>
</body>
</html>
