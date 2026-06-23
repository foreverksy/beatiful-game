<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>열기구 아일랜드: 가시 피하기 게임</title>
    <style>
        /* 외부 폰트 로드 및 기본 폰트 설정 */
        @import url('https://fonts.googleapis.com/css2?family=Jua&display=swap');
        
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            font-family: 'Jua', 'Apple SD Gothic Neo', 'Malgun Gothic', sans-serif;
            background-color: #0f172a; /* 주변 배경을 깔끔하고 어두운 톤으로 정리 */
            color: #f8fafc;
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none;
        }

        /* 중앙 게임 카드 컨테이너 */
        .game-container {
            position: relative;
            width: 100%;
            max-width: 420px;
            height: 100vh;
            max-height: 850px;
            background-color: #1e293b;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.6);
            overflow: hidden;
            display: flex;
            flex-direction: column;
            border: 1px solid #334155;
        }

        @media (min-width: 768px) {
            .game-container {
                border-radius: 24px;
            }
        }

        /* 상단 UI 정보 바 */
        .hud-bar {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 16px;
            z-index: 10;
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            background: linear-gradient(to bottom, rgba(15, 23, 42, 0.7), transparent);
            pointer-events: none;
        }

        .score-box {
            display: flex;
            flex-direction: column;
            gap: 2px;
        }

        .score-title {
            font-size: 1.15rem;
            color: #ffffff;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.8);
        }

        .score-value {
            font-size: 1.8rem;
            color: #facc15;
            font-weight: bold;
        }

        .highscore-value {
            font-size: 0.8rem;
            color: #cbd5e1;
        }

        .status-box {
            display: flex;
            flex-direction: column;
            gap: 8px;
            align-items: flex-end;
        }

        /* 버프 상태 알림 배지 */
        .badge {
            display: flex;
            align-items: center;
            gap: 6px;
            padding: 6px 12px;
            border-radius: 9999px;
            font-size: 0.85rem;
            color: #ffffff;
            border: 1px solid rgba(255, 255, 255, 0.3);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.2);
            animation: pulse 1.5s infinite;
        }

        .badge.hidden {
            display: none !important;
        }

        .badge-magnet {
            background-color: rgba(234, 179, 8, 0.9);
            border-color: #fef08a;
        }

        .badge-shield {
            background-color: rgba(239, 68, 68, 0.9);
            border-color: #fca5a5;
        }

        .level-badge {
            background-color: rgba(37, 99, 235, 0.8);
            border-color: #93c5fd;
            padding: 4px 10px;
            border-radius: 9999px;
            font-size: 0.8rem;
        }

        /* 캔버스 */
        canvas {
            width: 100%;
            height: 100%;
            display: block;
            background: linear-gradient(to bottom, #38bdf8, #bae6fd, #6366f1);
        }

        /* 조작법 안내 문구 */
        .control-guide {
            position: absolute;
            bottom: 16px;
            left: 0;
            width: 100%;
            display: flex;
            justify-content: center;
            pointer-events: none;
            padding: 0 16px;
            text-align: center;
        }

        .guide-bubble {
            background-color: rgba(15, 23, 42, 0.65);
            color: rgba(255, 255, 255, 0.9);
            font-size: 0.75rem;
            padding: 6px 14px;
            border-radius: 9999px;
            backdrop-filter: blur(4px);
        }

        /* 전체 오버레이 창 */
        .overlay {
            position: absolute;
            inset: 0;
            background-color: rgba(15, 23, 42, 0.93);
            backdrop-filter: blur(8px);
            -webkit-backdrop-filter: blur(8px);
            z-index: 20;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 24px;
            text-align: center;
        }

        .overlay.hidden {
            display: none !important;
        }

        /* 인트로 벌룬 모션 */
        .bounce-animation {
            animation: bounce 2s infinite ease-in-out;
            margin-bottom: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        /* 타이틀 명칭 */
        .game-title {
            font-size: 2.5rem;
            font-weight: 800;
            margin-bottom: 8px;
            letter-spacing: -0.025em;
            background: linear-gradient(to right, #fde047, #f97316, #ef4444);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-shadow: 0 4px 12px rgba(239, 68, 68, 0.3);
        }

        .game-desc {
            color: #cbd5e1;
            font-size: 0.85rem;
            line-height: 1.5;
            margin-bottom: 24px;
            max-width: 280px;
        }

        .highlight-red { color: #ef4444; font-weight: bold; }
        .highlight-sky { color: #38bdf8; font-weight: bold; }

        /* 가이드 카드 형태 */
        .rules-card {
            background-color: rgba(30, 41, 59, 0.8);
            border: 1px solid #475569;
            border-radius: 16px;
            padding: 16px;
            width: 100%;
            max-width: 310px;
            margin-bottom: 28px;
            text-align: left;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .rule-item {
            display: flex;
            align-items: center;
            gap: 12px;
            font-size: 0.75rem;
            color: #e2e8f0;
        }

        .icon-box {
            width: 24px;
            height: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            flex-shrink: 0;
        }

        .bubble-indicator {
            width: 16px;
            height: 16px;
            background-color: #38bdf8;
            border: 2px solid rgba(255, 255, 255, 0.6);
            border-radius: 50%;
            box-shadow: 0 0 4px #0284c7;
        }

        /* 하단 액션 버튼 */
        .btn-action {
            width: 100%;
            max-width: 310px;
            color: #ffffff;
            font-size: 1.25rem;
            font-weight: bold;
            padding: 16px;
            border: none;
            border-radius: 16px;
            cursor: pointer;
            transition: all 0.15s ease-in-out;
            font-family: 'Jua', sans-serif;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.3);
        }

        .btn-start {
            background: linear-gradient(to right, #10b981, #0d9488);
        }
        .btn-start:active {
            transform: scale(0.97);
            background: linear-gradient(to right, #059669, #0f766e);
        }

        .btn-restart {
            background: linear-gradient(to right, #f59e0b, #ea580c);
        }
        .btn-restart:active {
            transform: scale(0.97);
            background: linear-gradient(to right, #d97706, #ca8a04);
        }

        /* 스코어보드 디자인 */
        .result-board {
            background-color: rgba(15, 23, 42, 0.95);
            border: 1px solid #334155;
            border-radius: 16px;
            padding: 24px 16px;
            width: 100%;
            max-width: 310px;
            margin-bottom: 24px;
            display: flex;
            flex-direction: column;
            gap: 16px;
        }

        .result-title {
            color: #94a3b8;
            font-size: 0.85rem;
            text-transform: uppercase;
            letter-spacing: 0.05em;
        }

        .result-score {
            font-size: 2.8rem;
            color: #facc15;
            font-weight: 800;
            line-height: 1;
        }

        .result-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 12px 0 12px;
            border-top: 1px solid #1e293b;
            font-size: 0.85rem;
        }

        .result-row-label {
            color: #94a3b8;
        }

        .result-row-val {
            color: #ffffff;
            font-weight: bold;
        }

        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-12px); }
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.85; transform: scale(1.03); }
        }

        .svg-icon {
            width: 100%;
            height: 100%;
            fill: currentColor;
        }
    </style>
</head>
<body>

    <div class="game-container">
        
        <!-- 실시간 HUD 정보바 -->
        <div class="hud-bar">
            <div class="score-box">
                <div class="score-title">점수: <span id="uiScore" class="score-value">0</span></div>
                <div class="highscore-value">최고 점수: <span id="uiHighScore">0</span></div>
            </div>
            
            <div class="status-box">
                <!-- 자석 지속시간 타이머 -->
                <div id="magnetStatus" class="badge badge-magnet hidden">
                    <div class="icon-box" style="width:16px; height:16px;">
                        <svg class="svg-icon" viewBox="0 0 512 512" style="color: #ffffff;">
                            <path d="M425.7 256c-13.8 0-25 11.2-25 25v16.1c0 62.1-48.4 113.8-110.1 114.9-63 1.1-114.6-50.1-114.6-113.1v-163c0-44.1 35.9-80 80-80s80 35.9 80 80v63c0 13.8 11.2 25 25 25s25-11.2 25-25v-63c0-71.7-58.3-130-130-130S126 121.3 126 193v163c0 91.1 74.4 165.6 166.1 163 87.5-2.5 158.6-74.8 158.6-163.9V281c0-13.8-11.2-25-25-25z"/>
                        </svg>
                    </div>
                    <span>자석: <span id="magnetTime">5.0</span>초</span>
                </div>
                
                <!-- 무적 지속시간 타이머 -->
                <div id="shieldStatus" class="badge badge-shield hidden">
                    <div class="icon-box" style="width:16px; height:16px;">
                        <svg class="svg-icon" viewBox="0 0 512 512" style="color: #ffffff;">
                            <path d="M256 32C164 158 112 242 112 308c0 79.5 64.5 144 144 144s144-64.5 144-144c0-66-52-150-144-276zm0 372c-50.8 0-92-41.2-92-92 0-39 28.5-93.5 92-180.5 63.5 87 92 141.5 92 180.5 0 50.8-41.2 92-92 92z"/>
                        </svg>
                    </div>
                    <span>무적: <span id="shieldTime">5.0</span>초</span>
                </div>

                <!-- 현재 난이도 정보 -->
                <div class="level-badge">
                    <span id="uiLevel">Level 1</span>
                </div>
            </div>
        </div>

        <!-- 캔버스 그리기 화면 -->
        <canvas id="gameCanvas"></canvas>

        <!-- 하단 드래그 가이드 -->
        <div class="control-guide">
            <span class="guide-bubble">
                클릭/터치 드래그로 열기구를 이리저리 움직이세요!
            </span>
        </div>

        <!-- 게임 대기 인트로 오버레이 -->
        <div id="startScreen" class="overlay">
            <div class="bounce-animation">
                <svg width="105" height="120" viewBox="0 0 100 120" style="filter: drop-shadow(0 10px 8px rgba(0,0,0,0.4))">
                    <path d="M 50 10 A 35 35 0 1 1 50 80 Q 50 85 45 95 L 55 95 Q 50 85 50 80 Z" fill="#f43f5e"/>
                    <path d="M 38 30 A 35 35 0 0 1 62 30 Q 50 80 38 30" fill="#fb923c"/>
                    <path d="M 46 20 A 35 35 0 0 1 54 20 Q 50 80 46 20" fill="#facc15"/>
                    <line x1="43" y1="95" x2="45" y2="105" stroke="#94a3b8" stroke-width="2.5"/>
                    <line x1="57" y1="95" x2="55" y2="105" stroke="#94a3b8" stroke-width="2.5"/>
                    <rect x="42" y="105" width="16" height="13" rx="2" fill="#b45309"/>
                </svg>
            </div>
            
            <h1 class="game-title">벌룬 어드벤처</h1>
            <p class="game-desc">
                하늘에서 떨어지는 <span class="highlight-red">뾰족뾰족 가시 판</span>을 피해 <span class="highlight-sky">물방울</span>을 수집하세요! 시간이 흐를수록 난이도가 상승합니다!
            </p>

            <!-- 2번 스크린샷 완벽 싱크 가이드 카드 -->
            <div class="rules-card">
                <div class="rule-item">
                    <div class="icon-box">
                        <div class="bubble-indicator"></div>
                    </div>
                    <span><strong>파란 물방울:</strong> 획득 시 점수 획득</span>
                </div>
                <div class="rule-item">
                    <div class="icon-box" style="color: #eab308;">
                        <svg class="svg-icon" viewBox="0 0 512 512">
                            <path d="M425.7 256c-13.8 0-25 11.2-25 25v16.1c0 62.1-48.4 113.8-110.1 114.9-63 1.1-114.6-50.1-114.6-113.1v-163c0-44.1 35.9-80 80-80s80 35.9 80 80v63c0 13.8 11.2 25 25 25s25-11.2 25-25v-63c0-71.7-58.3-130-130-130S126 121.3 126 193v163c0 91.1 74.4 165.6 166.1 163 87.5-2.5 158.6-74.8 158.6-163.9V281c0-13.8-11.2-25-25-25z"/>
                        </svg>
                    </div>
                    <span><strong>자석 아이템 (노랑):</strong> 5초 동안 주변 방울 흡수</span>
                </div>
                <div class="rule-item">
                    <div class="icon-box" style="color: #ef4444; width:18px;">
                        <svg class="svg-icon" viewBox="0 0 512 512">
                            <path d="M256 32C164 158 112 242 112 308c0 79.5 64.5 144 144 144s144-64.5 144-144c0-66-52-150-144-276zm0 372c-50.8 0-92-41.2-92-92 0-39 28.5-93.5 92-180.5 63.5 87 92 141.5 92 180.5 0 50.8-41.2 92-92 92z"/>
                        </svg>
                    </div>
                    <span><strong>빨간 물방울:</strong> 5초 동안 가시 부딪혀도 무적!</span>
                </div>
                <div class="rule-item" style="color:#f87171; font-weight: bold;">
                    <div class="icon-box" style="color: #ef4444;">
                        <svg class="svg-icon" viewBox="0 0 512 512">
                            <path d="M449.07 399.08L285 117.13a29.18 29.18 0 00-50.07 0L70.93 399.08A28.53 28.53 0 0096 440h320a28.53 28.53 0 0025.07-40.92zM256 368a16 16 0 1116-16 16 16 0 01-16 16zm16-48h-32v-96h32z"/>
                        </svg>
                    </div>
                    <span>주의! 시간이 흐를수록 속도가 대폭 증가!</span>
                </div>
            </div>

            <!-- 대형 녹색 시작 버튼 -->
            <button id="btnStart" class="btn-action btn-start">게임 시작하기</button>
        </div>

        <!-- 게임 오버 레이아웃 -->
        <div id="gameOverScreen" class="overlay hidden">
            <div style="width:70px; height:70px; margin-bottom: 20px;">
                <svg class="svg-icon" viewBox="0 0 512 512" style="color:#ef4444;">
                    <path d="M448 224C448 102.49 349.51 4 228 4S8 102.49 8 224c0 74.07 36.42 139.77 92 179.91V448a32 32 0 0032 32h192a32 32 0 0032-32v-44.09c55.58-40.14 92-105.84 92-179.91zm-288-40a24 24 0 1124-24 24 24 0 01-24 24zm144 0a24 24 0 1124-24 24 24 0 01-24 24zm-112 144c10-21.2 30-32 64-32s54 10.8 64 32a16 16 0 11-28.66 14.34c-4.9-9.8-15.12-16.34-35.34-16.34s-30.44 6.54-35.34 16.34A16 16 0 11192 328z"/>
                </svg>
            </div>
            
            <h2 class="game-title" style="background: linear-gradient(to right, #f87171, #ef4444); -webkit-background-clip: text;">GAME OVER</h2>
            <p class="game-desc" style="margin-bottom: 16px;">가시 판에 부딪혀 열기구가 파괴되었습니다!</p>

            <div class="result-board">
                <div>
                    <span class="result-title">최종 획득 점수</span>
                    <div id="finalScore" class="result-score">0</div>
                </div>
                <div class="result-row">
                    <span class="result-row-label">최고 점수 기록</span>
                    <span id="finalHighScore" class="result-row-val">0</span>
                </div>
                <div class="result-row">
                    <span class="result-row-label">도달한 최종 레벨</span>
                    <span id="finalLevel" class="result-row-val" style="color: #60a5fa;">Level 1</span>
                </div>
            </div>

            <button id="btnRestart" class="btn-action btn-restart">다시 시작하기</button>
        </div>

    </div>

    <!-- 스크립트 엔진 -->
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // 요소 매핑
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const btnStart = document.getElementById('btnStart');
        const btnRestart = document.getElementById('btnRestart');

        const uiScore = document.getElementById('uiScore');
        const uiHighScore = document.getElementById('uiHighScore');
        const uiLevel = document.getElementById('uiLevel');
        const finalScore = document.getElementById('finalScore');
        const finalHighScore = document.getElementById('finalHighScore');
        const finalLevel = document.getElementById('finalLevel');

        const magnetStatus = document.getElementById('magnetStatus');
        const magnetTime = document.getElementById('magnetTime');
        const shieldStatus = document.getElementById('shieldStatus');
        const shieldTime = document.getElementById('shieldTime');

        // 가상 논리 정밀도 (400 x 700 해상도 고정)
        const VIEW_WIDTH = 400;
        const VIEW_HEIGHT = 700;

        function resizeCanvas() {
            const rect = canvas.getBoundingClientRect();
            const dpr = window.devicePixelRatio || 1;
            canvas.width = rect.width * dpr;
            canvas.height = rect.height * dpr;
            ctx.scale(dpr, dpr);
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        // 상태 변수
        let gameState = 'START';
        let score = 0;
        let highScore = parseInt(localStorage.getItem('balloon_highscore') || '0');
        let gameTime = 0;
        let level = 1;

        // 버프 유지시간
        let magnetDuration = 0;
        let shieldDuration = 0;

        // 요소 배열
        let spikes = [];
        let bubbles = [];
        let particles = [];
        let clouds = [
            { x: 40, y: 80, speed: 0.15, scale: 0.9 },
            { x: 260, y: 220, speed: 0.1, scale: 1.3 },
            { x: 100, y: 410, speed: 0.2, scale: 0.7 },
            { x: 320, y: 580, speed: 0.08, scale: 1.1 }
        ];

        let spikeSpawnTimer = 0;
        let bubbleSpawnTimer = 0;

        // 열기구 모델 정의
        const balloon = {
            x: VIEW_WIDTH / 2,
            y: VIEW_HEIGHT - 130,
            radius: 20,
            targetX: VIEW_WIDTH / 2,
            targetY: VIEW_HEIGHT - 130,
            speed: 0.2,
            tilt: 0,

            draw: function(ctx, sX, sY) {
                const rx = this.x * sX;
                const ry = this.y * sY;
                const rRadius = this.radius * sX;

                ctx.save();
                ctx.translate(rx, ry);
                ctx.rotate(this.tilt);

                // 빨간 구체 무적 아우라
                if (shieldDuration > 0) {
                    ctx.beginPath();
                    ctx.arc(0, 0, rRadius * 1.6, 0, Math.PI * 2);
                    ctx.fillStyle = `rgba(239, 68, 68, ${0.15 + Math.sin(Date.now() / 120) * 0.08})`;
                    ctx.strokeStyle = '#ef4444';
                    ctx.lineWidth = 3;
                    ctx.stroke();
                    ctx.fill();
                }

                // 자석 기운 아우라
                if (magnetDuration > 0) {
                    ctx.beginPath();
                    ctx.arc(0, 0, rRadius * 2.5, 0, Math.PI * 2);
                    ctx.strokeStyle = `rgba(234, 179, 8, ${0.22 + Math.sin(Date.now() / 160) * 0.1})`;
                    ctx.lineWidth = 2;
                    ctx.setLineDash([6, 6]);
                    ctx.stroke();
                    ctx.setLineDash([]);
                }

                // 바구니 연결 줄
                ctx.beginPath();
                ctx.moveTo(-rRadius * 0.4, rRadius * 0.85);
                ctx.lineTo(-rRadius * 0.25, rRadius * 1.3);
                ctx.moveTo(rRadius * 0.4, rRadius * 0.85);
                ctx.lineTo(rRadius * 0.25, rRadius * 1.3);
                ctx.strokeStyle = '#475569';
                ctx.lineWidth = 1.8;
                ctx.stroke();

                // 갈색 나무 바구니
                ctx.fillStyle = '#b45309';
                ctx.fillRect(-rRadius * 0.25, rRadius * 1.3, rRadius * 0.5, rRadius * 0.4);
                ctx.strokeStyle = '#78350f';
                ctx.lineWidth = 1;
                ctx.strokeRect(-rRadius * 0.25, rRadius * 1.3, rRadius * 0.5, rRadius * 0.4);

                // 열기구 벌룬 외관
                ctx.beginPath();
                ctx.arc(0, -rRadius * 0.1, rRadius, 0.15 * Math.PI, 0.85 * Math.PI, true);
                ctx.lineTo(-rRadius * 0.4, rRadius * 0.85);
                ctx.lineTo(rRadius * 0.4, rRadius * 0.85);
                ctx.closePath();

                const grad = ctx.createLinearGradient(-rRadius, 0, rRadius, 0);
                if (shieldDuration > 0 && Math.floor(Date.now() / 100) % 2 === 0) {
                    grad.addColorStop(0, '#f87171');
                    grad.addColorStop(0.5, '#ef4444');
                    grad.addColorStop(1, '#b91c1c');
                } else {
                    grad.addColorStop(0, '#f43f5e');
                    grad.addColorStop(0.3, '#fb923c');
                    grad.addColorStop(0.5, '#facc15');
                    grad.addColorStop(0.7, '#fb923c');
                    grad.addColorStop(1, '#f43f5e');
                }
                ctx.fillStyle = grad;
                ctx.fill();

                ctx.strokeStyle = 'rgba(15,23,42,0.2)';
                ctx.lineWidth = 2.5;
                ctx.stroke();

                // 하이라이트 광택
                ctx.beginPath();
                ctx.arc(-rRadius * 0.35, -rRadius * 0.4, rRadius * 0.22, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
                ctx.fill();

                ctx.restore();
            }
        };

        // 가시 클래스 정의
        class Spike {
            constructor() {
                this.width = Math.random() * 40 + 75;
                this.height = 36;
                this.x = Math.random() * (VIEW_WIDTH - this.width);
                this.y = -this.height - 10;
                this.speed = Math.random() * 1.4 + 2.2 + (level * 0.55);
                this.spikeCount = Math.floor(this.width / 14);
            }

            update() {
                this.y += this.speed;
            }

            draw(ctx, sX, sY) {
                const rx = this.x * sX;
                const ry = this.y * sY;
                const rw = this.width * sX;
                const rh = this.height * sY;

                ctx.save();

                // 지지대 철판
                ctx.fillStyle = '#475569';
                ctx.fillRect(rx, ry, rw, rh * 0.3);

                // 고정 고리 못 디테일
                ctx.fillStyle = '#cbd5e1';
                ctx.beginPath();
                ctx.arc(rx + 8, ry + (rh * 0.15), 2.5, 0, Math.PI*2);
                ctx.arc(rx + rw - 8, ry + (rh * 0.15), 2.5, 0, Math.PI*2);
                ctx.fill();

                // 아래를 향해 튀어나온 뾰족한 가시 뿔들
                const step = rw / this.spikeCount;
                for (let i = 0; i < this.spikeCount; i++) {
                    const sx = rx + (i * step);
                    ctx.beginPath();
                    ctx.moveTo(sx, ry + (rh * 0.3));
                    ctx.lineTo(sx + step, ry + (rh * 0.3));
                    ctx.lineTo(sx + (step / 2), ry + rh);
                    ctx.closePath();

                    ctx.fillStyle = (i % 2 === 0) ? '#94a3b8' : '#cbd5e1';
                    ctx.fill();

                    ctx.strokeStyle = '#1e293b';
                    ctx.lineWidth = 1;
                    ctx.stroke();
                }

                ctx.restore();
            }

            checkCollision(bObj) {
                const bx = bObj.x;
                const by = bObj.y;
                const br = bObj.radius - 3.5;

                const cx = Math.max(this.x, Math.min(bx, this.x + this.width));
                const cy = Math.max(this.y, Math.min(by, this.y + this.height));

                const dx = bx - cx;
                const dy = by - cy;
                const distSquared = (dx * dx) + (dy * dy);

                return distSquared < (br * br);
            }
        }

        // 아이템 물방울 클래스 정의
        class Bubble {
            constructor() {
                this.x = Math.random() * (VIEW_WIDTH - 40) + 20;
                this.y = -20;
                this.radius = 9;
                this.speed = Math.random() * 1.1 + 1.4 + (level * 0.22);

                const rand = Math.random();
                if (rand < 0.82) {
                    this.type = 'NORMAL';
                    this.radius = 10;
                } else if (rand < 0.91) {
                    this.type = 'MAGNET';
                    this.radius = 13;
                } else {
                    this.type = 'SHIELD';
                    this.radius = 13;
                }

                this.wiggleOffset = Math.random() * 100;
            }

            update() {
                this.y += this.speed;
                this.x += Math.sin((gameTime * 2.2) + this.wiggleOffset) * 0.35;

                // 자석 획득 효과 처리
                if (magnetDuration > 0) {
                    const dx = balloon.x - this.x;
                    const dy = balloon.y - this.y;
                    const dist = Math.sqrt(dx * dx + dy * dy);
                    const range = 180;

                    if (dist < range) {
                        const pullPower = (range - dist) / range;
                        const finalSpeed = 8.5 * pullPower;
                        this.x += (dx / dist) * finalSpeed;
                        this.y += (dy / dist) * finalSpeed;
                    }
                }

                if (this.x < this.radius) this.x = this.radius;
                if (this.x > VIEW_WIDTH - this.radius) this.x = VIEW_WIDTH - this.radius;
            }

            draw(ctx, sX, sY) {
                const rx = this.x * sX;
                const ry = this.y * sY;
                const rad = this.radius * sX;

                ctx.save();
                ctx.translate(rx, ry);

                if (this.type === 'NORMAL') {
                    const grad = ctx.createRadialGradient(-rad * 0.2, -rad * 0.2, rad * 0.1, 0, 0, rad);
                    grad.addColorStop(0, '#ffffff');
                    grad.addColorStop(0.3, '#7dd3fc');
                    grad.addColorStop(0.8, '#0284c7');
                    grad.addColorStop(1, 'rgba(2,132,199,0.3)');

                    ctx.beginPath();
                    ctx.arc(0, 0, rad, 0, Math.PI * 2);
                    ctx.fillStyle = grad;
                    ctx.fill();

                    ctx.strokeStyle = 'rgba(255,255,255,0.7)';
                    ctx.lineWidth = 1;
                    ctx.stroke();

                } else if (this.type === 'MAGNET') {
                    const grad = ctx.createRadialGradient(-rad * 0.2, -rad * 0.2, rad * 0.1, 0, 0, rad);
                    grad.addColorStop(0, '#fef08a');
                    grad.addColorStop(0.4, '#eab308');
                    grad.addColorStop(1, '#a16207');

                    ctx.beginPath();
                    ctx.arc(0, 0, rad, 0, Math.PI * 2);
                    ctx.fillStyle = grad;
                    ctx.fill();

                    ctx.strokeStyle = '#ffffff';
                    ctx.lineWidth = 2.5;
                    ctx.lineCap = 'round';
                    ctx.beginPath();
                    ctx.arc(0, rad * 0.1, rad * 0.4, Math.PI, 0, true);
                    ctx.moveTo(-rad * 0.4, rad * 0.1);
                    ctx.lineTo(-rad * 0.4, -rad * 0.2);
                    ctx.moveTo(rad * 0.4, rad * 0.1);
                    ctx.lineTo(rad * 0.4, -rad * 0.2);
                    ctx.stroke();

                    ctx.fillStyle = '#ef4444';
                    ctx.fillRect(-rad * 0.5, -rad * 0.35, rad * 0.2, rad * 0.15);
                    ctx.fillStyle = '#3b82f6';
                    ctx.fillRect(rad * 0.3, -rad * 0.35, rad * 0.2, rad * 0.15);

                } else if (this.type === 'SHIELD') {
                    ctx.beginPath();
                    ctx.moveTo(0, -rad);
                    ctx.quadraticCurveTo(rad, -rad * 0.2, rad, rad * 0.45);
                    ctx.arc(0, rad * 0.45, rad, 0, Math.PI, false);
                    ctx.quadraticCurveTo(-rad, -rad * 0.2, 0, -rad);
                    ctx.closePath();

                    const grad = ctx.createRadialGradient(0, 0, rad * 0.2, 0, rad * 0.2, rad);
                    grad.addColorStop(0, '#fca5a5');
                    grad.addColorStop(0.5, '#ef4444');
                    grad.addColorStop(1, '#7f1d1d');
                    ctx.fillStyle = grad;
                    ctx.fill();

                    ctx.beginPath();
                    ctx.ellipse(-rad * 0.3, 0, rad * 0.18, rad * 0.35, Math.PI / 4, 0, Math.PI * 2);
                    ctx.fillStyle = 'rgba(255,255,255,0.55)';
                    ctx.fill();

                    ctx.strokeStyle = '#ffffff';
                    ctx.lineWidth = 1.5;
                    ctx.stroke();
                }

                ctx.restore();
            }

            checkCollision(bObj) {
                const dx = this.x - bObj.x;
                const dy = this.y - bObj.y;
                const dist = Math.sqrt(dx * dx + dy * dy);
                return dist < (bObj.radius + this.radius - 1.5);
            }
        }

        // 스파크 파티클
        class Particle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                this.color = color;
                this.radius = Math.random() * 3 + 2.5;
                this.vx = (Math.random() - 0.5) * 6.5;
                this.vy = (Math.random() - 0.5) * 6.5;
                this.alpha = 1;
                this.decay = Math.random() * 0.025 + 0.015;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.vy += 0.08;
                this.alpha -= this.decay;
            }

            draw(ctx, sX, sY) {
                ctx.save();
                ctx.globalAlpha = this.alpha;
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x * sX, this.y * sY, this.radius * sX, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            }
        }

        function triggerExplosion(x, y, color, count = 10) {
            for (let i = 0; i < count; i++) {
                particles.push(new Particle(x, y, color));
            }
        }

        // 이벤트 터치 제어 통합
        let isPointerActive = false;

        function processInput(clientX, clientY) {
            const rect = canvas.getBoundingClientRect();
            const canvasX = clientX - rect.left;
            const canvasY = clientY - rect.top;

            const virtualX = (canvasX / rect.width) * VIEW_WIDTH;
            const virtualY = (canvasY / rect.height) * VIEW_HEIGHT;

            balloon.targetX = Math.max(balloon.radius, Math.min(VIEW_WIDTH - balloon.radius, virtualX));
            balloon.targetY = Math.max(balloon.radius, Math.min(VIEW_HEIGHT - balloon.radius, virtualY));
        }

        canvas.addEventListener('mousedown', (e) => {
            isPointerActive = true;
            processInput(e.clientX, e.clientY);
        });
        canvas.addEventListener('mousemove', (e) => {
            if (isPointerActive) processInput(e.clientX, e.clientY);
        });
        window.addEventListener('mouseup', () => { isPointerActive = false; });

        canvas.addEventListener('touchstart', (e) => {
            isPointerActive = true;
            if (e.touches.length > 0) {
                processInput(e.touches[0].clientX, e.touches[0].clientY);
            }
        }, { passive: true });

        canvas.addEventListener('touchmove', (e) => {
            if (isPointerActive && e.touches.length > 0) {
                processInput(e.touches[0].clientX, e.touches[0].clientY);
            }
        }, { passive: true });

        canvas.addEventListener('touchend', () => { isPointerActive = false; });

        // 키보드 조작 수용
        const pressedKeys = {};
        window.addEventListener('keydown', (e) => { pressedKeys[e.key] = true; });
        window.addEventListener('keyup', (e) => { pressedKeys[e.key] = false; });

        function processKeyboardInput() {
            const speed = 6;
            if (pressedKeys['ArrowLeft'] || pressedKeys['a'] || pressedKeys['A']) {
                balloon.targetX = Math.max(balloon.radius, balloon.targetX - speed);
            }
            if (pressedKeys['ArrowRight'] || pressedKeys['d'] || pressedKeys['D']) {
                balloon.targetX = Math.min(VIEW_WIDTH - balloon.radius, balloon.targetX + speed);
            }
            if (pressedKeys['ArrowUp'] || pressedKeys['w'] || pressedKeys['W']) {
                balloon.targetY = Math.max(balloon.radius, balloon.targetY - speed);
            }
            if (pressedKeys['ArrowDown'] || pressedKeys['s'] || pressedKeys['S']) {
                balloon.targetY = Math.min(VIEW_HEIGHT - balloon.radius, balloon.targetY + speed);
            }
        }

        // 게임 리사이클 데이터 초기화
        function initGame() {
            score = 0;
            gameTime = 0;
            level = 1;
            spikes = [];
            bubbles = [];
            particles = [];
            magnetDuration = 0;
            shieldDuration = 0;

            balloon.x = VIEW_WIDTH / 2;
            balloon.y = VIEW_HEIGHT - 130;
            balloon.targetX = VIEW_WIDTH / 2;
            balloon.targetY = VIEW_HEIGHT - 130;
            balloon.tilt = 0;

            spikeSpawnTimer = 0;
            bubbleSpawnTimer = 0;

            updateUI();

            magnetStatus.classList.add('hidden');
            shieldStatus.classList.add('hidden');
        }

        function updateUI() {
            uiScore.textContent = score;
            uiHighScore.textContent = highScore;
            uiLevel.textContent = `Level ${level}`;
        }

        let previousTime = 0;

        function runGameLoop(timestamp) {
            if (!previousTime) previousTime = timestamp;
            const delta = timestamp - previousTime;
            previousTime = timestamp;

            if (gameState === 'PLAYING') {
                update(delta);
                render();
                requestAnimationFrame(runGameLoop);
            }
        }

        function update(dt) {
            gameTime += dt / 1000;

            // 15초 마다 확실하게 레벨 업 트리거
            const curLevel = Math.floor(gameTime / 15) + 1;
            if (curLevel !== level) {
                level = curLevel;
                updateUI();
                triggerExplosion(VIEW_WIDTH / 2, 70, '#60a5fa', 20);
                triggerExplosion(VIEW_WIDTH / 2, 70, '#fbbf24', 20);
            }

            processKeyboardInput();

            const dx = balloon.targetX - balloon.x;
            const dy = balloon.targetY - balloon.y;
            balloon.x += dx * balloon.speed;
            balloon.y += dy * balloon.speed;
            balloon.tilt = dx * 0.025;

            // 아이템 버프 유효시간 산정
            if (magnetDuration > 0) {
                magnetDuration -= dt;
                if (magnetDuration <= 0) {
                    magnetDuration = 0;
                    magnetStatus.classList.add('hidden');
                } else {
                    magnetStatus.classList.remove('hidden');
                    magnetTime.textContent = (magnetDuration / 1000).toFixed(1);
                }
            }

            if (shieldDuration > 0) {
                shieldDuration -= dt;
                if (shieldDuration <= 0) {
                    shieldDuration = 0;
                    shieldStatus.classList.add('hidden');
                } else {
                    shieldStatus.classList.remove('hidden');
                    shieldTime.textContent = (shieldDuration / 1000).toFixed(1);
                }
            }

            // 구름 기류
            clouds.forEach(c => {
                c.y += c.speed;
                if (c.y > VIEW_HEIGHT + 60) {
                    c.y = -60;
                    c.x = Math.random() * VIEW_WIDTH;
                }
            });

            // 가시 스폰 타이머 조정
            spikeSpawnTimer += dt;
            const spawnInterval = Math.max(550, 1600 - (level * 130));
            if (spikeSpawnTimer >= spawnInterval) {
                spikes.push(new Spike());
                spikeSpawnTimer = 0;
            }

            // 가시 판과의 충돌
            for (let i = spikes.length - 1; i >= 0; i--) {
                const s = spikes[i];
                s.update();

                if (s.y > VIEW_HEIGHT + 40) {
                    spikes.splice(i, 1);
                    continue;
                }

                if (s.checkCollision(balloon)) {
                    if (shieldDuration > 0) {
                        triggerExplosion(s.x + s.width / 2, s.y + s.height, '#ef4444', 15);
                        triggerExplosion(s.x + s.width / 2, s.y + s.height, '#94a3b8', 10);
                        spikes.splice(i, 1);
                        score += 5;
                        updateUI();
                    } else {
                        triggerGameOver();
                        return;
                    }
                }
            }

            // 물방울 및 버프 스폰
            bubbleSpawnTimer += dt;
            if (bubbleSpawnTimer >= 800) {
                bubbles.push(new Bubble());
                bubbleSpawnTimer = 0;
            }

            // 물방울과의 충돌
            for (let i = bubbles.length - 1; i >= 0; i--) {
                const b = bubbles[i];
                b.update();

                if (b.y > VIEW_HEIGHT + 30) {
                    bubbles.splice(i, 1);
                    continue;
                }

                if (b.checkCollision(balloon)) {
                    if (b.type === 'NORMAL') {
                        score += 10;
                        triggerExplosion(b.x, b.y, '#38bdf8', 8);
                    } else if (b.type === 'MAGNET') {
                        magnetDuration = 5000;
                        triggerExplosion(b.x, b.y, '#facc15', 15);
                    } else if (b.type === 'SHIELD') {
                        shieldDuration = 5000;
                        triggerExplosion(b.x, b.y, '#f87171', 18);
                    }

                    bubbles.splice(i, 1);
                    updateUI();
                }
            }

            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.update();
                if (p.alpha <= 0) particles.splice(i, 1);
            }
        }

        // 전체 화면 렌더링
        function render() {
            const rect = canvas.getBoundingClientRect();
            const sX = rect.width / VIEW_WIDTH;
            const sY = rect.height / VIEW_HEIGHT;

            ctx.clearRect(0, 0, VIEW_WIDTH, VIEW_HEIGHT);

            // 시간 레벨에 따라 밤하늘로 점차 어두워짐
            const skyGrad = ctx.createLinearGradient(0, 0, 0, rect.height);
            const darkness = Math.min(100, level * 9);
            skyGrad.addColorStop(0, `rgb(${Math.max(12, 56 - darkness)}, ${Math.max(25, 189 - darkness * 1.5)}, ${Math.max(45, 248 - darkness)})`);
            skyGrad.addColorStop(1, `rgb(${Math.max(15, 99 - darkness)}, ${Math.max(15, 102 - darkness)}, ${Math.max(45, 241 - darkness)})`);
            ctx.fillStyle = skyGrad;
            ctx.fillRect(0, 0, rect.width, rect.height);

            // 구름
            ctx.fillStyle = 'rgba(255, 255, 255, 0.22)';
            clouds.forEach(c => {
                const cx = c.x * sX;
                const cy = c.y * sY;
                const cScale = c.scale * sX;

                ctx.beginPath();
                ctx.arc(cx, cy, 20 * cScale, 0, Math.PI * 2);
                ctx.arc(cx + 15 * cScale, cy - 10 * cScale, 26 * cScale, 0, Math.PI * 2);
                ctx.arc(cx + 36 * cScale, cy, 20 * cScale, 0, Math.PI * 2);
                ctx.arc(cx + 18 * cScale, cy + 10 * cScale, 20 * cScale, 0, Math.PI * 2);
                ctx.closePath();
                ctx.fill();
            });

            bubbles.forEach(b => b.draw(ctx, sX, sY));
            spikes.forEach(s => s.draw(ctx, sX, sY));
            balloon.draw(ctx, sX, sY);
            particles.forEach(p => p.draw(ctx, sX, sY));
        }

        // 게임오버 트리거
        function triggerGameOver() {
            gameState = 'GAMEOVER';

            triggerExplosion(balloon.x, balloon.y, '#f43f5e', 35);
            triggerExplosion(balloon.x, balloon.y, '#f97316', 25);
            triggerExplosion(balloon.x, balloon.y, '#eab308', 20);

            if (score > highScore) {
                highScore = score;
                localStorage.setItem('balloon_highscore', highScore.toString());
            }

            finalScore.textContent = score;
            finalHighScore.textContent = highScore;
            finalLevel.textContent = `Level ${level}`;

            gameOverScreen.classList.remove('hidden');
        }

        // 버튼 상호작용
        btnStart.addEventListener('click', () => {
            startScreen.classList.add('hidden');
            gameState = 'PLAYING';
            initGame();
            requestAnimationFrame(runGameLoop);
        });

        btnRestart.addEventListener('click', () => {
            gameOverScreen.classList.add('hidden');
            gameState = 'PLAYING';
            initGame();
            requestAnimationFrame(runGameLoop);
        });

        setTimeout(render, 100);

    </script>
</body>
</html>
