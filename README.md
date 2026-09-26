<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>두근두근 예민의 진급</title>
    <style>
        :root {
            --bg-color: #0f1115;
            --gb-body: #8a99ad;
            --screen-frame: #23272a;
            --screen-bg: #9bbc0f;
            --pixel-dark: #0f380f;
            --btn-red: #e74c3c;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
            -webkit-user-select: none;
        }

        body {
            background-color: var(--bg-color);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            overflow: hidden;
        }

        .gameboy-container {
            width: 100vw;
            max-width: 380px;
            height: 100vh;
            max-height: 780px;
            background: var(--gb-body);
            border-radius: 24px;
            padding: 16px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8);
            border: 2px solid #5a697a;
        }

        .screen-frame {
            width: 100%;
            height: 55%;
            background: var(--screen-frame);
            border-radius: 12px;
            padding: 10px;
            display: flex;
            flex-direction: column;
            position: relative;
        }

        .screen-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: #727c8a;
            font-size: 11px;
            font-weight: bold;
            margin-bottom: 6px;
        }

        .star-display {
            color: #f1c40f;
            font-size: 12px;
            font-weight: bold;
        }

        .canvas-wrapper {
            position: relative;
            width: 100%;
            flex-grow: 1;
            border: 3px solid #0f380f;
            border-radius: 4px;
            overflow: hidden;
            background: var(--screen-bg);
        }

        #game-canvas {
            width: 100%;
            height: 100%;
            image-rendering: pixelated;
            display: block;
        }

        .dialog-box {
            position: absolute;
            bottom: 6px;
            left: 5%;
            width: 90%;
            background: #f8f9fa;
            border: 2px solid #0f380f;
            border-radius: 6px;
            padding: 8px;
            font-size: 11px;
            color: #0f380f;
            font-weight: bold;
            line-height: 1.3;
            display: none;
            box-shadow: 2px 2px 0px #0f380f;
            z-index: 30;
        }

        .dialog-next {
            text-align: right;
            font-size: 8px;
            color: #7f8c8d;
            margin-top: 4px;
        }

        .controller-area {
            width: 100%;
            height: 38%;
            display: flex;
            justify-content: space-around;
            align-items: center;
        }

        .dpad-grid {
            display: grid;
            grid-template-columns: repeat(3, 42px);
            grid-template-rows: repeat(3, 42px);
        }

        .dpad-btn {
            background: #2c3e50;
            border: 1px solid #1a252f;
            box-shadow: inset 0 1px 0 rgba(255,255,255,0.2);
            cursor: pointer;
        }
        .dpad-btn:active { background: #415b76; }

        .dpad-up { grid-column: 2; grid-row: 1; border-radius: 6px 6px 0 0; }
        .dpad-left { grid-column: 1; grid-row: 2; border-radius: 6px 0 0 6px; }
        .dpad-center { grid-column: 2; grid-row: 2; background: #2c3e50; }
        .dpad-right { grid-column: 3; grid-row: 2; border-radius: 0 6px 6px 0; }
        .dpad-down { grid-column: 2; grid-row: 3; border-radius: 0 0 6px 6px; }

        .btn-a-container {
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .btn-a {
            width: 72px;
            height: 72px;
            background: var(--btn-red);
            border-radius: 50%;
            border: 3px solid #922b21;
            color: white;
            font-size: 22px;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            box-shadow: 0 5px 0 #641e16;
            cursor: pointer;
        }

        .btn-a:active {
            transform: translateY(3px);
            box-shadow: 0 2px 0 #641e16;
        }
    </style>
</head>
<body>

<div class="gameboy-container">
    <div class="screen-frame">
        <div class="screen-header">
            <span>GAME BOY COLOR</span>
            <span class="star-display" id="star-display">⭐ x 0</span>
        </div>
        <div class="canvas-wrapper">
            <canvas id="game-canvas" width="240" height="240"></canvas>
            <div id="dialog" class="dialog-box">
                <div id="dialog-text">대화 내용이 출력됩니다.</div>
                <div class="dialog-next">[A 버튼을 누르세요]</div>
            </div>
        </div>
    </div>

    <div class="controller-area">
        <div class="dpad-grid">
            <div class="dpad-btn dpad-up" onclick="handleInput('UP')"></div>
            <div class="dpad-btn dpad-left" onclick="handleInput('LEFT')"></div>
            <div class="dpad-btn dpad-center"></div>
            <div class="dpad-btn dpad-right" onclick="handleInput('RIGHT')"></div>
            <div class="dpad-btn dpad-down" onclick="handleInput('DOWN')"></div>
        </div>
        <div class="btn-a-container">
            <div class="btn-a" onclick="handleInput('A')">A</div>
        </div>
    </div>
</div>

<script>
    const canvas = document.getElementById('game-canvas');
    const ctx = canvas.getContext('2d');
    const dialogBox = document.getElementById('dialog');
    const dialogText = document.getElementById('dialog-text');
    const starDisplay = document.getElementById('star-display');

    let gameState = {
        stage: 0,
        stars: 0,
        player: { x: 110, y: 110 },
        partner: { x: 90, y: 110 },
        dialogIndex: 0,
        dialogList: [],
        endingStep: 0
    };

    function loadGame() {
        const saved = localStorage.getItem('yemin_game_save');
        if (saved) {
            const data = JSON.parse(saved);
            gameState.stage = data.stage || 0;
            gameState.stars = data.stars || 0;
            updateStarDisplay();
        }
    }

    function saveGame() {
        localStorage.setItem('yemin_game_save', JSON.stringify({
            stage: gameState.stage,
            stars: gameState.stars
        }));
    }

    function updateStarDisplay() {
        starDisplay.innerText = `⭐ x ${gameState.stars}`;
    }

    function playBeep(freq = 440) {
        try {
            const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = 'square';
            osc.frequency.value = freq;
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            osc.start();
            gain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 0.05);
            osc.stop(audioCtx.currentTime + 0.05);
        } catch(e) {}
    }

    function showDialog(lines, onComplete) {
        gameState.dialogList = lines;
        gameState.dialogIndex = 0;
        gameState.onDialogComplete = onComplete;
        dialogText.innerText = gameState.dialogList[0];
        dialogBox.style.display = 'block';
    }

    function handleInput(type) {
        playBeep(type === 'A' ? 520 : 280);

        if (dialogBox.style.display === 'block') {
            if (type === 'A') {
                gameState.dialogIndex++;
                if (gameState.dialogIndex < gameState.dialogList.length) {
                    dialogText.innerText = gameState.dialogList[gameState.dialogIndex];
                } else {
                    dialogBox.style.display = 'none';
                    if (gameState.onDialogComplete) gameState.onDialogComplete();
                }
            }
            return;
        }

        switch(gameState.stage) {
            case 0:
                if (type === 'A') startStage1();
                break;
            case 1:
                if (type === 'A') {
                    gameState.player.x += 8;
                    if (gameState.player.x >= 160) finishStage1();
                }
                break;
            case 2:
                movePlayer(type);
                if (type === 'A') checkCabinetItem();
                break;
            case 3:
                movePlayer(type);
                if (type === 'A') checkQuizAnswer();
                break;
            case 4:
                movePlayer(type);
                if (type === 'A') checkPlayground();
                break;
            case 5:
                movePlayer(type);
                gameState.partner.x = gameState.player.x - 20;
                gameState.partner.y = gameState.player.y;
                if (type === 'A') checkIceRink();
                break;
            case 6:
                if (type === 'A') advanceEnding();
                break;
        }
    }

    function movePlayer(type) {
        const speed = 10;
        if (type === 'UP' && gameState.player.y > 30) gameState.player.y -= speed;
        if (type === 'DOWN' && gameState.player.y < 190) gameState.player.y += speed;
        if (type === 'LEFT' && gameState.player.x > 20) gameState.player.x -= speed;
        if (type === 'RIGHT' && gameState.player.x < 200) gameState.player.x += speed;
    }

    // --- STAGE LOGIC ---

    function startStage1() {
        gameState.stage = 1;
        gameState.player = { x: 20, y: 120 };
        showDialog(["상병 진급 축하해! 🎖️", "예서와의 소중한 추억의 별 5개를 모아야 해!", "STAGE 1: A버튼을 마구 연타해서 한강 오리배를 저어가자!"]);
    }

    function finishStage1() {
        gameState.stars = 1;
        updateStarDisplay();
        saveGame();
        showDialog([
            "우리 작년에 한강에서 오리배 탈때",
            "오빠가 노래 불러준거 기억 나?",
            "너무너무 행복했어 🎵",
            "⭐ 첫 번째 별 획득!"
        ], () => startStage2());
    }

    function startStage2() {
        gameState.stage = 2;
        gameState.player = { x: 40, y: 180 };
        showDialog(["STAGE 2: 관물대에서 추억의 물건 '명찰'을 찾아봐!"]);
    }

    function checkCabinetItem() {
        if (Math.abs(gameState.player.x - 170) < 30 && Math.abs(gameState.player.y - 70) < 30) {
            gameState.stars = 2;
            updateStarDisplay();
            saveGame();
            showDialog(["우리의 소중한 '명찰'을 찾았다!", "⭐ 두 번째 별 획득!"], () => startStage3());
        } else {
            showDialog(["복권, 에어팟, 패딩이 있네.", "'명찰'은 어디 있을까?"]);
        }
    }

    function startStage3() {
        gameState.stage = 3;
        gameState.player = { x: 110, y: 180 };
        showDialog(["STAGE 3: 우리의 첫 데이트 장소 발판으로 이동해서 A버튼을 눌러!"]);
    }

    function checkQuizAnswer() {
        if (gameState.player.x > 120 && gameState.player.y < 90) {
            gameState.stars = 3;
            updateStarDisplay();
            saveGame();
            showDialog(["정답! 우리의 첫 데이트 장소는 '야탑'! 💖", "⭐ 세 번째 별 획득!"], () => startStage4());
        } else {
            showDialog(["땡! 다시 생각해봐~ 😜"]);
        }
    }

    function startStage4() {
        gameState.stage = 4;
        gameState.player = { x: 30, y: 30 };
        showDialog(["STAGE 4: 둘만 아는 특별한 장소 '서희 놀이터'를 찾아가자!"]);
    }

    function checkPlayground() {
        if (Math.abs(gameState.player.x - 120) < 30 && Math.abs(gameState.player.y - 120) < 30) {
            gameState.stars = 4;
            updateStarDisplay();
            saveGame();
            showDialog(["둘만 아는 장소, '서희 놀이터' 발견!", "⭐ 네 번째 별 획득!"], () => startStage5());
        } else {
            showDialog(["가로등과 벤치가 있어. 놀이터 기구 쪽으로 가보자!"]);
        }
    }

    function startStage5() {
        gameState.stage = 5;
        gameState.player = { x: 60, y: 120 };
        gameState.partner = { x: 40, y: 120 };
        showDialog(["STAGE 5: 예서와 함께 아이스링크장 중앙으로 가서 A버튼을 눌러!"]);
    }

    function checkIceRink() {
        if (Math.abs(gameState.player.x - 170) < 30) {
            gameState.stars = 5;
            updateStarDisplay();
            saveGame();
            showDialog([
                "손 꼭 잡고 탈때 진짜 재미있었는데!",
                "기억에 남는 아이스링크장 데이트 완성! ⛸️",
                "⭐ 마지막 다섯 번째 별 획득!"
            ], () => startEnding());
        }
    }

    function startEnding() {
        gameState.stage = 6;
        gameState.endingStep = 0;
        advanceEnding();
    }

    function advanceEnding() {
        const credits = [
            "9개월동안 너무 고생 많았어",
            "조금만 더 힘내서 꽃길 걷자!",
            "상병된거 너무 축하해",
            "✨ 사랑해! ✨"
        ];

        if (gameState.endingStep < credits.length) {
            showDialog([credits[gameState.endingStep]]);
            gameState.endingStep++;
        } else {
            showDialog(["🎉 게임을 모두 마쳤습니다! 🎉", "[다시 시작하려면 새로고침 해주세요]"]);
        }
    }

    // --- DRAWING GRAPHICS ---

    function drawPixelRect(x, y, w, h, color) {
        ctx.fillStyle = color;
        ctx.fillRect(Math.floor(x), Math.floor(y), w, h);
    }

    function drawCharacter(x, y, isYemin = true) {
        // 테두리
        drawPixelRect(x, y, 18, 22, '#0f380f');
        // 머리 (예민: 검정 숏컷 / 예서: 갈색 단발)
        drawPixelRect(x+2, y+2, 14, 6, isYemin ? '#111' : '#8b4513');
        // 얼굴
        drawPixelRect(x+3, y+6, 12, 6, '#ffdfc4');
        // 눈
        drawPixelRect(x+5, y+8, 2, 2, '#000');
        drawPixelRect(x+11, y+8, 2, 2, '#000');
        // 볼터치 (예서만)
        if(!isYemin) drawPixelRect(x+3, y+10, 2, 2, '#ff7675');
        // 옷 (예민: 해병대/국방색 / 예서: 핑크)
        drawPixelRect(x+2, y+12, 14, 6, isYemin ? '#2d572c' : '#fd79a8');
        // 신발
        drawPixelRect(x+3, y+18, 4, 3, '#000');
        drawPixelRect(x+11, y+18, 4, 3, '#000');
    }

    // 오리배 전용 드로잉 함수
    function drawDuckBoat(x, y) {
        // 물결
        drawPixelRect(x-10, y+18, 55, 3, '#2980b9');
        // 오리 몸통 (배)
        drawPixelRect(x, y+6, 38, 14, '#ffffff');
        drawPixelRect(x, y+6, 38, 14, '#0f380f');
        drawPixelRect(x+2, y+8, 34, 10, '#f1c40f');
        // 오리 머리
        drawPixelRect(x+26, y-6, 12, 12, '#f1c40f');
        drawPixelRect(x+30, y-2, 2, 2, '#000'); // 눈
        drawPixelRect(x+36, y, 8, 4, '#e67e22'); // 부리
        // 탑승자 캐릭터
        drawCharacter(x+8, y-2, true);
    }

    // 에어팟 도트 드로잉
    function drawAirpods(x, y) {
        drawPixelRect(x, y, 14, 12, '#0f380f');
        drawPixelRect(x+2, y+2, 10, 8, '#ffffff');
        drawPixelRect(x+6, y+5, 2, 2, '#2ecc71'); // 충전 불빛
    }

    // 패딩 도트 드로잉
    function drawPadding(x, y) {
        drawPixelRect(x, y, 16, 18, '#0f380f');
        drawPixelRect(x+2, y+2, 12, 14, '#34495e');
        drawPixelRect(x+7, y+2, 2, 14, '#ffffff'); // 자크 선
    }

    function render() {
        ctx.fillStyle = '#8bac0f';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        switch(gameState.stage) {
            case 0:
                ctx.fillStyle = '#0f380f';
                ctx.font = 'bold 13px sans-serif';
                ctx.fillText("두근두근 예민의 진급", 55, 70);
                ctx.font = 'bold 11px sans-serif';
                ctx.fillText("상병 진급 축하 기념 🎮", 55, 95);
                ctx.fillText("[ A 버튼을 누르세요 ]", 55, 160);
                drawCharacter(110, 115, true);
                break;

            case 1:
                ctx.fillStyle = '#74b9ff';
                ctx.fillRect(0, 0, 240, 240); // 한강 물 표현
                ctx.fillStyle = '#0f380f';
                ctx.font = 'bold 12px sans-serif';
                ctx.fillText("A버튼을 연타해 오리배를 저으세요! 🦆", 20, 30);
                drawDuckBoat(gameState.player.x, gameState.player.y);
                break;

            case 2:
                // 관물대 배경
                drawPixelRect(20, 40, 50, 70, '#0f380f');
                drawPixelRect(23, 43, 44, 64, '#34495e');
                ctx.fillStyle = '#fff'; ctx.font = '10px sans-serif'; ctx.fillText("관물대 1", 26, 60);

                drawPixelRect(170, 40, 50, 70, '#0f380f');
                drawPixelRect(173, 43, 44, 64, '#34495e');
                ctx.fillStyle = '#fff'; ctx.fillText("관물대 2", 176, 60);

                // 바닥 아이템 디테일 배치
                drawAirpods(40, 130);
                drawPadding(180, 130);

                // 명찰
                drawPixelRect(185, 75, 20, 8, '#e74c3c');
                ctx.fillStyle = '#fff'; ctx.font = '7px sans-serif'; ctx.fillText("예민", 188, 82);

                ctx.fillStyle = '#0f380f';
                ctx.font = 'bold 12px sans-serif';
                ctx.fillText("[관물대에서 '명찰' 찾기]", 50, 25);
                drawCharacter(gameState.player.x, gameState.player.y, true);
                break;

            case 3:
                ctx.fillStyle = '#0f380f';
                ctx.font = 'bold 12px sans-serif';
                ctx.fillText("Q. 우리의 첫 데이트 장소는?", 35, 25);

                // 선명한 퀴즈 발판
                const options = [
                    { name: "1. 서현", x: 20, y: 40 },
                    { name: "2. 야탑", x: 130, y: 40 },
                    { name: "3. 시내", x: 20, y: 110 },
                    { name: "4. 판교", x: 130, y: 110 }
                ];

                options.forEach(opt => {
                    drawPixelRect(opt.x, opt.y, 90, 50, '#0f380f');
                    drawPixelRect(opt.x+3, opt.y+3, 84, 44, '#ffffff');
                    ctx.fillStyle = '#0f380f';
                    ctx.font = 'bold 14px sans-serif';
                    ctx.fillText(opt.name, opt.x + 20, opt.y + 28);
                });

                drawCharacter(gameState.player.x, gameState.player.y, true);
                break;

            case 4:
                // 서희 놀이터 기구
                drawPixelRect(100, 90, 40, 40, '#0f380f');
                drawPixelRect(103, 93, 34, 34, '#e67e22'); // 미끄럼틀 기구
                ctx.fillStyle = '#fff'; ctx.font = '10px sans-serif'; ctx.fillText("놀이기구", 100, 112);

                // 가로등
                drawPixelRect(40, 60, 4, 30, '#0f380f');
                drawPixelRect(36, 56, 12, 6, '#f1c40f');

                ctx.fillStyle = '#0f380f';
                ctx.font = 'bold 12px sans-serif';
                ctx.fillText("🎠 서희 놀이터", 80, 30);
                drawCharacter(gameState.player.x, gameState.player.y, true);
                break;

            case 5:
                // 빙판 표현
                drawPixelRect(10, 10, 220, 220, '#0f380f');
                drawPixelRect(13, 13, 214, 214, '#dff9fb');
                ctx.fillStyle = '#0f380f';
                ctx.font = 'bold 12px sans-serif';
                ctx.fillText("❄️ 아이스링크장 ❄️", 70, 35);

                drawCharacter(gameState.player.x, gameState.player.y, true);
                drawCharacter(gameState.partner.x, gameState.partner.y, false);
                break;

            case 6:
                ctx.fillStyle = '#0f380f';
                ctx.font = 'bold 13px sans-serif';
                ctx.fillText("🎉 진급을 축하합니다! 🎉", 45, 50);
                drawCharacter(95, 100, true);
                drawCharacter(125, 100, false);

                // 폭죽 이펙트
                drawPixelRect(30, 80, 6, 6, '#e74c3c');
                drawPixelRect(200, 70, 6, 6, '#9b59b6');
                drawPixelRect(60, 150, 6, 6, '#f1c40f');
                drawPixelRect(180, 160, 6, 6, '#2ecc71');
                break;
        }

        requestAnimationFrame(render);
    }

    loadGame();
    render();
</script>
</body>
</html>
