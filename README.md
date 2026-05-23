<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Online Multiplayer Tic-Tac-Toe</title>
    <script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>
    <style>
        :root {
            --bg-color: #0f172a;
            --panel-bg: #1e293b;
            --text-color: #f8fafc;
            --player-x: #38bdf8;
            --player-o: #f43f5e;
            --accent: #10b981;
            --danger: #ef4444;
            --gray-border: #334155;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', sans-serif; }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .screen { display: none; width: 100%; max-width: 400px; text-align: center; }
        .active { display: block !important; }

        .panel {
            background: var(--panel-bg);
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.4);
        }

        h1 {
            font-size: 2.5rem;
            margin-bottom: 15px;
            font-weight: 800;
            background: linear-gradient(to right, var(--player-x), var(--player-o));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        input[type="text"] {
            width: 100%;
            padding: 12px;
            background: var(--bg-color);
            border: 2px solid var(--gray-border);
            border-radius: 8px;
            color: white;
            font-size: 16px;
            margin-bottom: 12px;
            outline: none;
            text-align: center;
        }
        input[type="text"]:focus { border-color: var(--player-x); }

        .btn {
            background: linear-gradient(135deg, #0284c7, #0369a1);
            color: white;
            border: none;
            padding: 14px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            width: 100%;
            margin-bottom: 10px;
            transition: opacity 0.2s, transform 0.1s;
        }
        .btn:hover { opacity: 0.95; }
        .btn-success { background: linear-gradient(135deg, #22c55e, #16a34a); }
        .btn-danger { background: linear-gradient(135deg, #ef4444, #dc2626); }
        .btn-bot { background: linear-gradient(135deg, #f59e0b, #d97706); }

        .top-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .timer-badge {
            background: var(--danger);
            color: white;
            padding: 4px 12px;
            border-radius: 20px;
            font-weight: bold;
            font-size: 14px;
            display: none;
        }

        .status {
            font-size: 1.2rem;
            margin-bottom: 20px;
            color: #94a3b8;
            min-height: 28px;
        }

        .board {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
            background: var(--bg-color);
            padding: 12px;
            border-radius: 12px;
            margin-bottom: 20px;
            border: 2px solid var(--gray-border);
        }

        .cell {
            aspect-ratio: 1;
            background-color: var(--panel-bg);
            border: none;
            border-radius: 8px;
            font-size: 3rem;
            font-weight: bold;
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: background 0.2s, transform 0.1s;
        }
        .cell:hover:not(.taken) { background-color: #334155; transform: scale(1.02); }
        .cell.taken { cursor: not-allowed; }
        .cell.X { color: var(--player-x); text-shadow: 0 0 10px rgba(56, 189, 248, 0.4); }
        .cell.O { color: var(--player-o); text-shadow: 0 0 10px rgba(244, 63, 94, 0.4); }
        .cell.winning-cell { background-color: var(--accent); color: var(--bg-color); text-shadow: none; }
    </style>
</head>
<body>

    <div id="lobby-screen" class="screen active panel">
        <h1>Tic-Tac-Toe</h1>
        <input type="text" id="username" placeholder="Enter Nickname" value="Player">
        <button class="btn btn-success" onclick="createRoom()">Create Room</button>
        <div style="margin: 8px 0; color: #64748b; font-weight: bold;">OR</div>
        <input type="text" id="join-room-id" placeholder="Enter Room ID">
        <button class="btn" onclick="joinRoom()">Join Room</button>
        <button class="btn" style="background: #475569;" onclick="joinRandom()">Play Random Match</button>
        <button class="btn btn-bot" onclick="startBotGame()">🤖 Play vs Unbeatable Bot</button>
    </div>

    <div id="waiting-screen" class="screen panel">
        <h3>Waiting for Player...</h3>
        <div id="display-room-id" style="background:#0f172a; padding:15px; border-radius:8px; font-family:monospace; margin:15px 0; border: 1px solid var(--gray-border);">Generating ID...</div>
        <p style="font-size: 14px; color:#94a3b8; margin-bottom: 20px;">Share this ID with a friend to connect.</p>
        <button class="btn btn-danger" onclick="location.reload()">Exit Room</button>
    </div>

    <div id="game-screen" class="screen panel">
        <div class="top-row">
            <div style="display:flex; flex-direction:column; align-items:flex-start; font-size:14px; color:#94a3b8;">
                <span id="player-profile">You: X</span>
                <span id="opponent-profile">Opponent: O</span>
            </div>
            <div class="timer-badge" id="timer-box"><span id="timer-count">30</span>s</div>
        </div>
        
        <div class="status" id="status">Connecting...</div>

        <div class="board" id="board">
            <button class="cell" data-index="0"></button>
            <button class="cell" data-index="1"></button>
            <button class="cell" data-index="2"></button>
            <button class="cell" data-index="3"></button>
            <button class="cell" data-index="4"></button>
            <button class="cell" data-index="5"></button>
            <button class="cell" data-index="6"></button>
            <button class="cell" data-index="7"></button>
            <button class="cell" data-index="8"></button>
        </div>

        <button class="btn" id="reset-btn" onclick="requestReset()" style="display:none;">Play Again</button>
        <button class="btn btn-danger" onclick="location.reload()" style="padding: 10px; font-size:14px;">Leave Match</button>
    </div>

<script>
    let peer = null, conn = null;
    let myName = "Player", opponentName = "Opponent";
    let mySign = "X", opponentSign = "O";
    let isMyTurn = false;
    let isBotMode = false;
    let isGameActive = false;
    let board = ["", "", "", "", "", "", "", "", ""];
    let turnTimer = null;
    let timeLeft = 30;

    const cells = document.querySelectorAll('.cell');
    const statusElement = document.getElementById('status');
    const timerBox = document.getElementById('timer-box');
    const timerCount = document.getElementById('timer-count');
    const winConditions = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];

    cells.forEach(cell => cell.addEventListener('click', handleCellClick));

    function showScreen(id) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById(id).classList.add('active');
    }

    /* TURN TIMER MANAGEMENT */
    function startTurnTimer() {
        stopTurnTimer();
        if (!isGameActive) return;

        timeLeft = 30;
        timerCount.innerText = timeLeft;
        timerBox.style.display = "block";

        turnTimer = setInterval(() => {
            timeLeft--;
            timerCount.innerText = timeLeft;

            if (timeLeft <= 0) {
                stopTurnTimer();
                if (isMyTurn) {
                    autoChooseMove();
                }
            }
        }, 1000);
    }

    function stopTurnTimer() {
        if (turnTimer) clearInterval(turnTimer);
        timerBox.style.display = "none";
    }

    function autoChooseMove() {
        // Collect all open slots on the board
        let availableIndices = [];
        for (let i = 0; i < board.length; i++) {
            if (board[i] === "") availableIndices.push(i);
        }

        if (availableIndices.length > 0) {
            // Pick a random spot
            let randomIdx = availableIndices[Math.floor(Math.random() * availableIndices.length)];
            executeMoveSequence(randomIdx, mySign);

            if (!isBotMode && conn && conn.open) {
                conn.send({ type: 'move', index: randomIdx });
            }
            
            postMoveCheck();
        }
    }

    /* PEER ENGINE */
    function initPeer(customId = null) {
        myName = document.getElementById('username').value.trim() || "Player";
        peer = customId ? new Peer(customId) : new Peer();
        
        peer.on('open', (id) => {
            if (!customId) document.getElementById('display-room-id').innerText = id;
        });
        peer.on('connection', (c) => {
            if (conn) { c.close(); return; }
            conn = c;
            setupConnection();
        });
        peer.on('error', (err) => {
            if(err.type === 'unavailable-id') joinRandom();
            else { alert("Connection error: " + err.type); location.reload(); }
        });
    }

    function createRoom() { mySign = "X"; opponentSign = "O"; isMyTurn = true; showScreen('waiting-screen'); initPeer(); }

    function joinRoom() {
        let rId = document.getElementById('join-room-id').value.trim();
        if(!rId) return alert("Enter Room ID");
        mySign = "O"; opponentSign = "X"; isMyTurn = false;
        showScreen('waiting-screen');
        peer = new Peer();
        peer.on('open', () => {
            conn = peer.connect(rId);
            setupConnection();
        });
    }

    function joinRandom() {
        let bucket = Math.floor(Math.random() * 5) + 1;
        let randomId = "TTT_ROOM_POOL_" + bucket;
        mySign = "O"; opponentSign = "X"; isMyTurn = false;
        showScreen('waiting-screen');
        peer = new Peer();
        peer.on('open', () => {
            let testConn = peer.connect(randomId);
            let active = false;
            testConn.on('open', () => { active = true; conn = testConn; setupConnection(); });
            setTimeout(() => {
                if(!active) {
                    testConn.close(); peer.destroy();
                    mySign = "X"; opponentSign = "O"; isMyTurn = true; initPeer(randomId);
                }
            }, 2000);
        });
    }

    function startBotGame() {
        isBotMode = true;
        isGameActive = true;
        mySign = "X";
        opponentSign = "O";
        isMyTurn = true;
        opponentName = "RoboBot 🤖";
        
        showScreen('game-screen');
        updateProfiles();
        statusElement.innerText = "Your turn (X)";
        startTurnTimer();
    }

    function setupConnection() {
        conn.on('open', () => {
            showScreen('game-screen');
            conn.send({ type: 'handshake', name: myName });
        });
        conn.on('data', handleData);
        conn.on('close', () => {
            statusElement.innerText = "Opponent left the match.";
            isGameActive = false;
            stopTurnTimer();
        });
    }

    function handleData(data) {
        if(data.type === 'handshake') {
            opponentName = data.name;
            updateProfiles();
            resetGameBoard();
            isGameActive = true;
            startTurnTimer();
        } else if(data.type === 'move') {
            executeMoveSequence(data.index, opponentSign);
            if(!checkGameResult()) {
                isMyTurn = true;
                statusElement.innerText = "Your turn!";
                startTurnTimer();
            }
        } else if(data.type === 'reset-request') {
            resetGameBoard();
            isGameActive = true;
            startTurnTimer();
        }
    }

    function updateProfiles() {
        document.getElementById('player-profile').innerText = `${myName}: ${mySign}`;
        document.getElementById('opponent-profile').innerText = `${opponentName}: ${opponentSign}`;
    }

    /* GAMEPLAY ENGINE */
    function executeMoveSequence(index, sign) {
        board[index] = sign;
        cells[index].innerText = sign;
        cells[index].classList.add('taken', sign);
    }

    function handleCellClick(e) {
        let idx = parseInt(e.target.getAttribute('data-index'));
        if(!isGameActive || !isMyTurn || board[idx] !== "") return;

        stopTurnTimer();
        executeMoveSequence(idx, mySign);
        isMyTurn = false;

        if(isBotMode) {
            postMoveCheck();
        } else {
            conn.send({ type: 'move', index: idx });
            if(!checkGameResult()) {
                statusElement.innerText = `Waiting for ${opponentName}...`;
                startTurnTimer(); // Track opponent's 30s turn budget limit
            }
        }
    }

    function postMoveCheck() {
        if(!checkGameResult()) {
            statusElement.innerText = "🤖 AI is planning your demise...";
            setTimeout(() => {
                let aiIdx = findBestMove(board);
                executeMoveSequence(aiIdx, opponentSign);
                if(!checkGameResult()) {
                    isMyTurn = true;
                    statusElement.innerText = "Your turn (X)";
                    startTurnTimer();
                }
            }, 500);
        }
    }

    function checkGameResult() {
        for (let condition of winConditions) {
            const [a, b, c] = condition;
            if (board[a] && board[a] === board[b] && board[a] === board[c]) {
                isGameActive = false;
                stopTurnTimer();
                if(board[a] === mySign) statusElement.innerText = "You won! 🎉";
                else statusElement.innerText = `${opponentName} wins! 💥`;
                condition.forEach(i => cells[i].classList.add('winning-cell'));
                document.getElementById('reset-btn').style.display = "block";
                return true;
            }
        }
        if (!board.includes("")) {
            isGameActive = false;
            stopTurnTimer();
            statusElement.innerText = "It's a draw! 🤝";
            document.getElementById('reset-btn').style.display = "block";
            return true;
        }
        return false;
    }

    function requestReset() {
        resetGameBoard();
        isGameActive = true;
        if(!isBotMode && conn && conn.open) {
            conn.send({ type: 'reset-request' });
        }
        startTurnTimer();
    }

    function resetGameBoard() {
        board = ["", "", "", "", "", "", "", "", ""];
        cells.forEach(c => { c.innerText = ""; c.className = "cell"; });
        document.getElementById('reset-btn').style.display = "none";
        if(isBotMode) {
            isMyTurn = true;
            statusElement.innerText = "Your turn (X)";
        } else {
            isMyTurn = (mySign === "X");
            statusElement.innerText = isMyTurn ? "Your turn!" : `Waiting for ${opponentName}...`;
        }
    }

    /* UNBEATABLE MINIMAX MATRIX */
    function evalWinner(b) {
        for(let c of winConditions) {
            if(b[c[0]] && b[c[0]] === b[c[1]] && b[c[0]] === b[c[2]]) return b[c[0]];
        }
        return b.includes("") ? null : "Tie";
    }

    function findBestMove(currentBoard) {
        let bestScore = -Infinity, move = 0;
        for(let i=0; i<9; i++) {
            if(currentBoard[i] === "") {
                currentBoard[i] = opponentSign;
                let score = minimax(currentBoard, 0, false);
                currentBoard[i] = "";
                if(score > bestScore) { bestScore = score; move = i; }
            }
        }
        return move;
    }

    function minimax(currentBoard, depth, isMax) {
        let scoreState = evalWinner(currentBoard);
        if(scoreState === opponentSign) return 10 - depth;
        if(scoreState === mySign) return depth - 10;
        if(scoreState === "Tie") return 0;

        if(isMax) {
            let bestScore = -Infinity;
            for(let i=0; i<9; i++) {
                if(currentBoard[i] === "") {
                    currentBoard[i] = opponentSign;
                    bestScore = Math.max(bestScore, minimax(currentBoard, depth+1, false));
                    currentBoard[i] = "";
                }
            }
            return bestScore;
        } else {
            let bestScore = Infinity;
            for(let i=0; i<9; i++) {
                if(currentBoard[i] === "") {
                    currentBoard[i] = mySign;
                    bestScore = Math.min(bestScore, minimax(currentBoard, depth+1, true));
                    currentBoard[i] = "";
                }
            }
            return bestScore;
        }
    }
</script>
</body>
</html>
