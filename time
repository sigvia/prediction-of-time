<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>タイム予測ランニングアプリ</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            max-width: 600px;
            margin: 30px auto;
            padding: 20px;
            background-color: #f8f9fa;
            color: #333;
        }
        h1 {
            font-size: 1.5rem;
            text-align: center;
        }
        .settings, .controls, .display, .laps {
            background: #ffffff;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            margin-bottom: 15px;
        }
        .row {
            display: flex;
            gap: 15px;
            margin-bottom: 10px;
        }
        .col {
            flex: 1;
        }
        label {
            display: block;
            font-size: 0.85rem;
            margin-bottom: 5px;
            color: #666;
        }
        input {
            width: 100%;
            padding: 8px;
            box-sizing: border-box;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        .button-group {
            display: flex;
            gap: 8px;
            flex-wrap: wrap;
        }
        button {
            flex: 1;
            min-width: 110px;
            padding: 10px;
            font-size: 0.95rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            color: white;
            font-weight: bold;
        }
        #btn-start { background-color: #28a745; }
        #btn-undo { background-color: #17a2b8; }
        #btn-stop { background-color: #ffc107; color: #333; }
        #btn-reset { background-color: #dc3545; }
        button:active { opacity: 0.8; }
        .time-display {
            font-size: 1.4rem;
            font-weight: bold;
            margin: 5px 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 10px;
        }
        .prediction-display {
            font-size: 1.2rem;
            color: #007bff;
            margin: 5px 0;
        }
        h3, h4 {
            margin: 0 0 10px 0;
        }
        #lap-list {
            list-style: none;
            padding: 0;
            margin: 0;
        }
        #lap-list li {
            padding: 8px 0;
            border-bottom: 1px solid #f1f1f1;
            font-size: 0.9rem;
        }
    </style>
</head>
<body>

    <h1>🏃‍♂️ タイム予測ランニングアプリ</h1>

    <!-- 設定エリア -->
    <div class="settings">
        <div class="row">
            <div class="col">
                <label for="lap-interval">ラップ距離 (m)</label>
                <input type="number" id="lap-interval" value="400" step="100">
            </div>
            <div class="col">
                <label for="total-distance">ゴール距離 (m)</label>
                <input type="number" id="total-distance" value="5000" step="100">
            </div>
        </div>
    </div>

    <!-- コントロールボタン -->
    <div class="controls">
        <div class="button-group">
            <button id="btn-start" onclick="handleStartLap()">スタート/ラップ</button>
            <button id="btn-undo" onclick="handleUndoLap()">直前のラップ削除</button>
            <button id="btn-stop" onclick="handleStop()">ストップ</button>
            <button id="btn-reset" onclick="handleReset()">リセット</button>
        </div>
    </div>

    <!-- 経過時間・予測表示 -->
    <div class="display">
        <div id="time-display" class="time-display">
            <span id="total-time-text">⏱️ タイム: 00:00.00</span>
            <span id="current-lap-display" style="color: #666; font-size: 1.1rem;">ラップ: 00:00.00</span>
        </div>
        <div id="prediction-display" class="prediction-display"></div>
    </div>

    <!-- ラップ一覧の表示 -->
    <div class="laps">
        <h3>📜 ラップ記録一覧</h3>
        <ul id="lap-list"></ul>
    </div>

    <script>
        let running = false;
        let startTime = 0;
        let elapsedTime = 0;
        let timerInterval = null;
        let laps = [];

        function formatTime(seconds) {
            const mins = Math.floor(seconds / 60);
            const secs = Math.floor(seconds % 60);
            const millis = Math.floor((seconds % 1) * 100);
            return `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}.${String(millis).padStart(2, '0')}`;
        }

        function updateDisplay() {
            if (running) {
                elapsedTime = (Date.now() - startTime) / 1000;
            }
            document.getElementById('total-time-text').innerText = `⏱️ タイム: ${formatTime(elapsedTime)}`;

            const previousTotalTime = laps.length > 0 ? laps[laps.length - 1].time : 0;
            const currentLapTime = elapsedTime - previousTotalTime;
            document.getElementById('current-lap-display').innerText = `ラップ: ${formatTime(currentLapTime)}`;

            if (laps.length > 0) {
                const lastLap = laps[laps.length - 1];
                document.getElementById('prediction-display').innerText = `🎯 ゴール予測: ${formatTime(lastLap.predicted)}`;
            } else {
                document.getElementById('prediction-display').innerText = '';
            }
        }

        function handleStartLap() {
            const lapInterval = parseFloat(document.getElementById('lap-interval').value);
            const totalDistance = parseFloat(document.getElementById('total-distance').value);

            if (!running) {
                // スタート処理
                running = true;
                startTime = Date.now() - (elapsedTime * 1000);
                timerInterval = setInterval(updateDisplay, 50);
            } else {
                // ラップ記録処理
                const currentTime = (Date.now() - startTime) / 1000;
                let currentDist = (laps.length + 1) * lapInterval;
                
                let isFinished = false;
                if (currentDist >= totalDistance) {
                    currentDist = totalDistance;
                    isFinished = true;
                }

                const previousTotalTime = laps.length > 0 ? laps[laps.length - 1].time : 0;
                const lapTime = currentTime - previousTotalTime;

                const predicted = (currentTime / currentDist) * totalDistance;

                laps.push({
                    distance: currentDist,
                    lapTime: lapTime,
                    time: currentTime,
                    predicted: predicted
                });

                renderLaps();
                updateDisplay();

                // ゴール距離に達したら自動でストップ
                if (isFinished) {
                    handleStop();
                }
            }
        }

        function handleUndoLap() {
            if (laps.length > 0) {
                laps.pop();
                renderLaps();
                updateDisplay();
            }
        }

        function handleStop() {
            running = false;
            clearInterval(timerInterval);
            updateDisplay();
        }

        function handleReset() {
            running = false;
            clearInterval(timerInterval);
            startTime = 0;
            elapsedTime = 0;
            laps = [];
            updateDisplay();
            renderLaps();
        }

        function renderLaps() {
            const lapListEl = document.getElementById('lap-list');
            lapListEl.innerHTML = '';
            laps.forEach(lap => {
                const li = document.createElement('li');
                li.innerHTML = `<strong>${lap.distance}m</strong> | ラップ: <strong>${formatTime(lap.lapTime)}</strong> | 通過: ${formatTime(lap.time)} | 予測: ${formatTime(lap.predicted)}`;
                lapListEl.appendChild(li);
            });
        }
    </script>
</body>
</html>
