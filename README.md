<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mahjong Edu-Sim</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { 
            background-color: #121212; 
            color: white; 
            font-family: 'Segoe UI', sans-serif; 
            display: flex; 
            flex-direction: column; 
            align-items: center; 
            padding: 20px; 
            margin: 0;
        }
        #game-container { 
            background: #1e272e; 
            padding: 25px; 
            border-radius: 20px; 
            border: 3px solid #f1c40f; 
            text-align: center; 
            width: 90%;
            max-width: 380px; 
            box-shadow: 0 10px 30px rgba(0,0,0,0.7); 
        }
        .header-title { margin: 0; color: #f1c40f; font-size: 22px; text-transform: uppercase; }
        .creator-tag { font-size: 12px; color: #bdc3c7; margin-bottom: 15px; }
        .balance-box { background: #000; padding: 15px; border-radius: 12px; margin-bottom: 15px; font-size: 1.8em; color: #2ecc71; font-weight: bold; border: 2px solid #27ae60; }
        .slot-machine { display: flex; justify-content: space-around; background: #0f1418; padding: 15px; border-radius: 15px; margin-bottom: 20px; gap: 10px; }
        .reel { width: 90px; height: 110px; background: #fff; color: #2c3e50; font-size: 55px; line-height: 110px; border-radius: 10px; font-weight: bold; }
        .controls { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        button { border: none; padding: 15px; font-size: 14px; font-weight: bold; border-radius: 10px; cursor: pointer; transition: 0.3s; }
        .btn-spin { background: #f1c40f; color: #000; }
        .btn-spin:hover { background: #d4ac0d; }
        .btn-auto { background: #e67e22; color: #fff; }
        .btn-auto:hover { background: #d35400; }
        button:disabled { background: #7f8c8d !important; opacity: 0.6; cursor: not-allowed; }
        #message { margin: 15px 0; font-weight: bold; height: 24px; color: #f1c40f; }
        
        .chart-container { width: 90%; max-width: 380px; margin-top: 20px; background: #2c3e50; padding: 15px; border-radius: 15px; box-sizing: border-box; }
        .stats-panel { margin-top: 20px; background: #2c3e50; padding: 20px; border-radius: 15px; width: 90%; max-width: 380px; box-sizing: border-box; }
        .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
        .stat-item { background: rgba(0,0,0,0.4); padding: 10px; border-radius: 8px; font-size: 12px; text-align: center; }
    </style>
</head>
<body>

<div id="game-container">
    <h2 class="header-title">MAHJONG SIMULATOR</h2>
    <div class="creator-tag">Edukasi Analisis oleh: <strong>Nama Anda</strong></div>
    <div class="balance-box">Rp <span id="balance">1000</span></div>
    
    <div class="slot-machine">
        <div class="reel" id="reel1">🀄</div>
        <div class="reel" id="reel2">🀆</div>
        <div class="reel" id="reel3">🀀</div>
    </div>

    <div class="controls">
        <button id="spinBtn" class="btn-spin" onclick="startSpin()">SPIN (-10)</button>
        <button id="autoBtn" class="btn-auto" onclick="toggleAuto()">AUTO 100X</button>
    </div>
    <div id="message">Pantau tren saldo Anda!</div>
</div>

<div class="chart-container">
    <canvas id="balanceChart"></canvas>
</div>

<div class="stats-panel">
    <div class="stats-grid">
        <div class="stat-item">Putaran: <span id="stat-spins">0</span></div>
        <div class="stat-item">Win Rate: <span id="stat-rate">0%</span></div>
    </div>
    <div style="font-size: 11px; color: #bdc3c7; margin-top: 15px; font-style: italic; text-align: center;">
        Analisis: Garis menurun menunjukkan bagaimana sistem dirancang menguntungkan "House".
    </div>
</div>

<script>
    const symbols = ['🀀', '🀄', '🀆', '🀇', '🀈', '🀉']; 
    let balance = 1000;
    let stats = { spins: 0, wins: 0 };
    let isSpinning = false;
    let isAutoMode = false;
    let autoCount = 0;

    const ctx = document.getElementById('balanceChart').getContext('2d');
    let balanceHistory = [1000];
    let labels = [0];
    const chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Tren Saldo (Rp)',
                data: balanceHistory,
                borderColor: '#2ecc71',
                backgroundColor: 'rgba(46, 204, 113, 0.1)',
                borderWidth: 2,
                fill: true,
                pointRadius: 0
            }]
        },
        options: {
            responsive: true,
            scales: {
                x: { display: false },
                y: { ticks: { color: '#bdc3c7' } }
            },
            plugins: { legend: { labels: { color: 'white' } } }
        }
    });

    function startSpin() {
        if (isSpinning || balance < 10) { stopAuto(); return; }
        isSpinning = true;
        balance -= 10;
        stats.spins++;
        updateUI();
        
        let shuffleCount = 0;
        const anim = setInterval(() => {
            document.querySelectorAll('.reel').forEach(r => {
                r.innerText = symbols[Math.floor(Math.random() * symbols.length)];
            });
            if (++shuffleCount > 8) {
                clearInterval(anim);
                calculateResult();
            }
        }, 50);
    }

    function calculateResult() {
        const r = Array.from(document.querySelectorAll('.reel')).map(el => el.innerText);
        if (r[0] === r[1] && r[1] === r[2]) {
            balance += 150; stats.wins++;
            document.getElementById('message').innerText = "JACKPOT! +150";
        } else if (r[0] === r[1] || r[1] === r[2] || r[0] === r[2]) {
            balance += 15; stats.wins++;
            document.getElementById('message').innerText = "MENANG KECIL! +15";
        } else {
            document.getElementById('message').innerText = "COBA LAGI";
        }

        balanceHistory.push(balance);
        labels.push(stats.spins);
        if (balanceHistory.length > 50) {
            balanceHistory.shift();
            labels.shift();
        }
        chart.update('none');

        updateUI();
        isSpinning = false;

        if (isAutoMode && autoCount > 1 && balance >= 10) {
            autoCount--;
            document.getElementById('autoBtn').innerText = `STOP (${autoCount})`;
            setTimeout(startSpin, 100); 
        } else { stopAuto(); }
    }

    function toggleAuto() {
        if (isAutoMode) { stopAuto(); } else {
            if (balance < 10) return;
            isAutoMode = true; autoCount = 100;
            document.getElementById('spinBtn').disabled = true;
            startSpin();
        }
    }

    function stopAuto() {
        isAutoMode = false; autoCount = 0;
        document.getElementById('spinBtn').disabled = false;
        document.getElementById('autoBtn').innerText = "AUTO 100X";
    }

    function updateUI() {
        document.getElementById('balance').innerText = balance;
        document.getElementById('stat-spins').innerText = stats.spins;
        const rate = stats.spins > 0 ? ((stats.wins / stats.spins) * 100).toFixed(1) : 0;
        document.getElementById('stat-rate').innerText = rate + "%";
        if (balance < 10) {
            document.getElementById('message').innerText = "SALDO HABIS!";
            document.getElementById('spinBtn').disabled = true;
        }
    }
</script>

</body>
</html>
