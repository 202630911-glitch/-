<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>수행평가 헬퍼</title>
    <style>
        :root {
            --primary-color: #4a90e2;
            --secondary-color: #f5a623;
            --bg-color: #f8f9fa;
            --text-color: #333;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            width: 100%;
            max-width: 600px;
            background: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08);
            text-align: center;
            box-sizing: border-box;
        }

        h1 { color: var(--primary-color); margin-bottom: 30px; }
        .hidden { display: none !important; }

        /* 메인 메뉴 스타일 */
        .menu-btn {
            display: block;
            width: 100%;
            padding: 20px;
            margin: 15px 0;
            font-size: 18px;
            font-weight: bold;
            border: none;
            border-radius: 12px;
            cursor: pointer;
            transition: all 0.2s;
        }
        .btn-schedule { background-color: var(--primary-color); color: white; }
        .btn-emergency { background-color: #ff6b6b; color: white; }
        .menu-btn:hover { transform: translateY(-2px); opacity: 0.9; }

        /* 달력 스타일 */
        .calendar-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }
        .calendar-grid {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            gap: 8px;
        }
        .day-name { font-weight: bold; padding: 5px; color: #777; }
        .day {
            aspect-ratio: 1;
            border: 1px solid #eef0f2;
            border-radius: 8px;
            padding: 4px;
            cursor: pointer;
            position: relative;
            background: #fff;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
        }
        .day:hover { background: #e3faffc7; }
        .day.has-event { border: 2px solid var(--primary-color); }
        .day-num { font-size: 14px; font-weight: 600; }
        .weather-icon { font-size: 14px; }

        /* 모달 및 상세 정보 */
        .detail-card {
            margin-top: 20px;
            padding: 20px;
            border-radius: 12px;
            background: #f1f3f5;
            text-align: left;
        }
        .detail-card h3 { margin-top: 0; color: var(--primary-color); }
        .back-btn {
            background: #6c757d; color: white; border: none;
            padding: 10px 20px; border-radius: 8px; cursor: pointer; margin-top: 20px;
        }
    </style>
</head>
<body>

    <div class="container">
        <!-- 1화면: 처음 선택 화면 -->
        <div id="screen-home">
            <h1>🎒 수행평가 헬퍼</h1>
            <button class="menu-btn btn-schedule" onclick="switchScreen('schedule')">📅 수행평가 일정 & 준비물</button>
            <button class="menu-btn btn-emergency" onclick="switchScreen('emergency')">🚨 준비물이 없다!</button>
        </div>

        <!-- 2화면: 달력 및 일정 화면 -->
        <div id="screen-schedule" class="hidden">
            <div class="calendar-header">
                <h2>2026년 9월</h2>
                <button class="back-btn" style="margin:0;" onclick="switchScreen('home')">홈으로</button>
            </div>
            
            <div class="calendar-grid" id="calendar">
                <!-- 요일 -->
                <div class="day-name">일</div><div class="day-name">월</div><div class="day-name">화</div>
                <div class="day-name">수</div><div class="day-name">목</div><div class="day-name">금</div>
                <div class="day-name">토</div>
                <!-- 달력 날짜는 JS가 자동 생성 -->
            </div>

            <!-- 상세 정보 표시 칸 -->
            <div id="detail-area" class="detail-card hidden">
                <h3 id="selected-date">날짜</h3>
                <p><strong>📝 수행평가:</strong> <span id="eval-name">-</span></p>
                <p><strong>🎒 준비물:</strong> <span id="eval-supplies">-</span></p>
                <p><strong>☁️ 날씨 예보:</strong> <span id="eval-weather">-</span></p>
            </div>
        </div>

        <!-- 3화면: 준비물이 없다! 화면 -->
        <div id="screen-emergency" class="hidden">
            <h2>🚨 준비물 비상대책 모색</h2>
            <div class="detail-card">
                <p style="font-size: 16px; line-height: 1.6;">
                    "나중에 생각하자"고 하셨으니 숨을 한 번 크게 쉬고 다음에 대책을 세워봅시다! 타임아웃! ☕
                </p>
            </div>
            <button class="back-btn" onclick="switchScreen('home')">돌아가기</button>
        </div>
    </div>

    <script>
        // 가상의 데이터 (9월 일정 및 날씨 데이터)
        // weather: 'rain'(비)이면 우산 표시
        const sampleData = {
            4: { test: "국어 서술형 평가", supplies: "검은색 볼펜, 교과서", weather: "rain", weatherText: "비 ☔ (우산 챙기세요!)" },
            9: { test: "수학 수행평가", supplies: "공학용 계산기, 연습장", weather: "clear", weatherText: "맑음 ☀️" },
            17: { test: "과학 실험 보고서", supplies: "실험관찰 노트", weather: "rain", weatherText: "오후에 비 ☔" },
            22: { test: "영어 말하기 평가", supplies: "대본 스크립트", weather: "cloudy", weatherText: "흐림 ☁️" }
        };

        // 화면 전환 함수
        function switchScreen(screenId) {
            document.getElementById('screen-home').classList.add('hidden');
            document.getElementById('screen-schedule').classList.add('hidden');
            document.getElementById('screen-emergency').classList.add('hidden');

            document.getElementById(`screen-${screenId}`).classList.remove('hidden');
        }

        // 달력 생성 함수 (2026년 9월 기준: 1일이 화요일 시작, 총 30일)
        function generateCalendar() {
            const calendarEl = document.getElementById('calendar');
            const startDayOffset = 2; // 화요일 시작 (일:0, 월:1, 화:2...)
            const totalDays = 30;

            // 시작 전 빈칸 채우기
            for (let i = 0; i < startDayOffset; i++) {
                const emptyCell = document.createElement('div');
                calendarEl.appendChild(emptyCell);
            }

            // 날짜 칸 만들기
            for (let day = 1; day <= totalDays; day++) {
                const dayEl = document.createElement('div');
                dayEl.className = 'day';
                
                // 날짜 숫자 추가
                const numEl = document.createElement('span');
                numEl.className = 'day-num';
                numEl.innerText = day;
                dayEl.appendChild(numEl);

                // 데이터가 있는 날 처리
                if (sampleData[day]) {
                    dayEl.classList.add('has-event');
                    
                    // 비가 오면 우산 이모티콘 표시
                    if (sampleData[day].weather === 'rain') {
                        const umbrellaEl = document.createElement('span');
                        umbrellaEl.className = 'weather-icon';
                        umbrellaEl.innerText = '☔';
                        dayEl.appendChild(umbrellaEl);
                    }
                    
                    // 클릭 이벤트 추가
                    dayEl.onclick = () => showDetail(day);
                } else {
                    // 일정이 없는 날 클릭 시
                    dayEl.onclick = () => showEmpty(day);
                }

                calendarEl.appendChild(dayEl);
            }
        }

        // 일정 상세 보기
        function showDetail(day) {
            const data = sampleData[day];
            document.getElementById('detail-area').classList.remove('hidden');
            document.getElementById('selected-date').innerText = `9월 ${day}일 일정`;
            document.getElementById('eval-name').innerText = data.test;
            document.getElementById('eval-supplies').innerText = data.supplies;
            document.getElementById('eval-weather').innerText = data.weatherText;
        }

        // 일정 없는 날 보기
        function showEmpty(day) {
            document.getElementById('detail-area').classList.remove('hidden');
            document.getElementById('selected-date').innerText = `9월 ${day}일`;
            document.getElementById('eval-name').innerText = "예정된 수행평가가 없습니다. 🎉";
            document.getElementById('eval-supplies').innerText = "없음";
            document.getElementById('eval-weather').innerText = "맑음 또는 정보 없음 🌤️";
        }

        // 페이지 로드 시 달력 생성
        generateCalendar();
    </script>
</body>
</html>
