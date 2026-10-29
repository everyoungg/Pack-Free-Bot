# Pack-Free-Bot
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pre-Pass : Pack-Free Bot</title>
    <style>
        /* 기본 스타일 초기화 및 폰트 설정 */
        @import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css');
        
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Pretendard', sans-serif;
        }

        body {
            background-color: #1a1a2e; /* 공항 키오스크에 어울리는 짙은 네이비 배경 */
            color: #ffffff;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
        }

        .kiosk-container {
            width: 100%;
            max-width: 1024px;
            height: 768px;
            background-color: #16213e;
            border-radius: 20px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
            display: flex;
            flex-direction: column;
            padding: 40px;
            position: relative;
        }

        /* 상단 헤더 */
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 40px;
            border-bottom: 2px solid #0f3460;
            padding-bottom: 20px;
        }

        .header h1 {
            font-size: 32px;
            font-weight: 700;
            color: #00d2ff;
        }

        .header .status {
            font-size: 18px;
            color: #a3a3a3;
        }

        /* 메인 콘텐츠 영역 */
        .main-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            text-align: center;
        }

        .instruction {
            font-size: 42px;
            font-weight: 700;
            margin-bottom: 20px;
            color: #e94560; /* 시선을 끄는 색상 */
        }

        .sub-instruction {
            font-size: 28px;
            color: #e0e0e0;
            margin-bottom: 50px;
        }

        /* 항공사 선택 버튼 그리드 */
        .airline-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 20px;
            width: 100%;
            max-width: 800px;
        }

        .airline-btn {
            background-color: #0f3460;
            border: 2px solid transparent;
            border-radius: 15px;
            padding: 30px 20px;
            font-size: 22px;
            font-weight: 600;
            color: white;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
        }

        .airline-btn:hover, .airline-btn:active {
            background-color: #e94560;
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(233, 69, 96, 0.4);
        }

        /* 결과 화면 (처음엔 숨김) */
        #result-screen, #loading-screen {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100%;
        }

        .result-title {
            font-size: 48px;
            color: #00d2ff;
            margin-bottom: 30px;
        }

        .result-desc {
            font-size: 32px;
            margin-bottom: 40px;
            line-height: 1.5;
            background: rgba(255,255,255,0.1);
            padding: 30px;
            border-radius: 15px;
        }

        .action-btn {
            background-color: #00d2ff;
            color: #1a1a2e;
            border: none;
            padding: 20px 40px;
            font-size: 28px;
            font-weight: 700;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 10px 20px rgba(0, 210, 255, 0.3);
            transition: all 0.3s ease;
        }

        .action-btn:hover {
            background-color: #00b8e6;
            transform: scale(1.05);
        }

        /* 로딩 애니메이션 */
        .spinner {
            width: 80px;
            height: 80px;
            border: 8px solid rgba(255,255,255,0.1);
            border-top: 8px solid #00d2ff;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-bottom: 30px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>

    <div class="kiosk-container">
        <!-- 헤더 영역 -->
        <div class="header">
            <h1>Pre-Pass : Pack-Free Bot</h1>
            <div class="status" id="time-display">2026. 10. 30 10:00 AM</div>
        </div>

        <!-- 1. 기본 화면 (항공사 선택) -->
        <div class="main-content" id="home-screen">
            <div class="instruction">보조배터리를 올려주세요</div>
            <div class="sub-instruction">판독을 위해 이용하실 항공사를 선택해주세요</div>
            
            <div class="airline-grid">
                <button class="airline-btn" onclick="startScan('대한항공')">대한항공</button>
                <button class="airline-btn" onclick="startScan('아시아나항공')">아시아나항공</button>
                <button class="airline-btn" onclick="startScan('제주항공')">제주항공</button>
                <button class="airline-btn" onclick="startScan('진에어')">진에어</button>
            </div>
        </div>

        <!-- 2. AI 판독(로딩) 화면 -->
        <div class="main-content" id="loading-screen">
            <div class="spinner"></div>
            <div class="instruction" id="loading-text">배터리 용량 및 단자를 판독 중입니다...</div>
            <div class="sub-instruction">잠시만 기다려주세요</div>
        </div>

        <!-- 3. 판독 결과 및 조치 안내 화면 -->
        <div class="main-content" id="result-screen">
            <div class="result-title">✔ 반입 가능 (조치 필요)</div>
            <div class="result-desc">
                <strong>[단자 노출 감지]</strong><br>
                절연테이프 부착이 필요합니다.<br>
                <span style="font-size: 24px; color:#a3a3a3; margin-top:15px; display:block;">(아래 디스펜서에서 절연테이프가 배출되었습니다. 단자 위에 붙여주세요)</span>
            </div>
            <button class="action-btn" onclick="issuePass()">디지털 검증 패스 받기</button>
        </div>
    </div>

    <script>
        // 시간 표시 기능
        function updateTime() {
            const now = new Date();
            document.getElementById('time-display').innerText = now.toLocaleString('ko-KR');
        }
        setInterval(updateTime, 1000);
        updateTime();

        // 스캔 시작 (항공사 버튼 클릭 시)
        function startScan(airline) {
            document.getElementById('home-screen').style.display = 'none';
            document.getElementById('loading-screen').style.display = 'flex';
            
            document.getElementById('loading-text').innerText = airline + ' 규정 연동 및 판독 중...';

            // 2.5초 후 결과 화면으로 전환
            setTimeout(() => {
                document.getElementById('loading-screen').style.display = 'none';
                document.getElementById('result-screen').style.display = 'flex';
            }, 2500);
        }

        // 디지털 패스 발급 완료 (초기화)
        function issuePass() {
            alert("디지털 검증 패스가 발급되었습니다. 스티커를 부착 후 검색대로 이동해주세요!");
            // 초기 화면으로 복귀
            document.getElementById('result-screen').style.display = 'none';
            document.getElementById('home-screen').style.display = 'flex';
        }
    </script>

</body>
</html>
