<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>학교 복도 탈출</title>
    
    <!-- CSS 스타일 코드 -->
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Nanum+Myeongjo:wght@700&display=swap');

        body {
            background-color: #000;
            color: #fff;
            font-family: 'Nanum Myeongjo', serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            overflow: hidden;
        }

        #game-container {
            position: relative;
            max-width: 1000px;
            width: 100%;
        }

        .background-image {
            width: 100%;
            display: block;
        }

        /* 클릭 가능한 영역의 기본 스타일 */
        .clickable-area {
            position: absolute;
            cursor: pointer;
            transition: background-color 0.2s, border 0.2s;
        }

        /* 마우스를 올렸을 때 클릭 영역을 시각적으로 표시 */
        .clickable-area:hover {
            background-color: rgba(255, 0, 0, 0.2); /* 붉은색 반투명 배경 */
            border: 2px solid rgba(255, 0, 0, 0.7); /* 붉은색 반투명 테두리 */
        }

        /* 각 클릭 영역의 위치와 크기 설정 */
        #area-quiz { /* 바닥의 종이 (퀴즈) */
            top: 548px; left: 142px;
            width: 206px; height: 94px;
        }
        #area-memory { /* 왼쪽 사물함 (메모리 게임) */
            top: 183px; left: 48px;
            width: 200px; height: 323px;
        }
        #area-hint { /* 천장 전등 (힌트) */
            top: 87px; left: 451px;
            width: 101px; height: 55px;
        }
        #area-door { /* 복도 끝의 문 (목표) */
            top: 257px; left: 461px;
            width: 79px; height: 191px;
        }


        #password-section {
            margin-top: 20px;
            display: flex;
            gap: 10px;
        }

        #password-section input {
            border: 2px solid #a00;
            background-color: #111;
            color: #fff;
            padding: 10px;
            font-size: 1.2em;
            width: 150px;
            text-align: center;
            font-family: 'Nanum Myeongjo', serif;
        }

        #password-section button {
            background-color: #a00;
            color: #fff;
            border: none;
            padding: 10px 20px;
            cursor: pointer;
            font-size: 1.2em;
            font-family: 'Nanum Myeongjo', serif;
        }

        /* Modal(팝업창) Styles */
        .modal-container {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            align-items: center;
            justify-content: center;
            z-index: 100;
        }

        .modal-content {
            background-color: #111;
            border: 2px solid #a00;
            padding: 30px;
            text-align: center;
            position: relative;
            max-width: 450px;
            width: 90%;
        }

        .close-btn {
            position: absolute;
            top: 10px;
            right: 15px;
            font-size: 2em;
            cursor: pointer;
            color: #fff;
        }

        .modal-content h2 {
            color: #a00;
            margin-top: 0;
        }

        .modal-close-btn {
            margin-top: 20px;
            padding: 10px 30px !important;
        }

        #result-text {
            font-size: 1.2em;
            line-height: 1.6;
        }

        /* Memory Game Styles */
        #memory-board {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
            margin-top: 20px;
        }

        .memory-card {
            width: 60px;
            height: 60px;
            background-color: #333;
            border: 2px solid #a00;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 2em;
            transform-style: preserve-3d;
            transition: transform 0.5s;
        }

        .memory-card.is-flipped {
            transform: rotateY(180deg);
        }

        .memory-card .card-face {
            position: absolute;
            backface-visibility: hidden;
        }

        .memory-card .card-back {
            transform: rotateY(180deg);
        }

        .memory-card.is-matched {
            visibility: hidden;
        }
    </style>
</head>
<body>
    <!-- HTML 구조 -->
    <div id="game-container">
        <img src="https://i.ibb.co/yn2nPWW0/Whisk-34d86011da.jpg" alt="귀신 나오는 복도" class="background-image">
        
        <!-- 마우스 오버 시 표시되는 클릭 가능 영역들 -->
        <div id="area-quiz" class="clickable-area" onclick="openModal('quiz-modal')"></div>
        <div id="area-memory" class="clickable-area" onclick="openModal('memory-game-modal'); startMemoryGame();"></div>
        <div id="area-hint" class="clickable-area" onclick="showResultModal('\'바닥의 숫자\'와 \'사물함의 숫자\'를 순서대로 조합하면 길이 열릴 것이다...\'는 속삭임이 들린다.')"></div>
        <div id="area-door" class="clickable-area" onclick="showResultModal('복도 끝의 문은 굳게 잠겨있다. 4자리 비밀번호가 필요한 것 같다.')"></div>
    </div>

    <div id="password-section">
        <input type="number" id="password-input" placeholder="비밀번호 4자리">
        <button onclick="checkPassword()">탈출 시도</button>
    </div>

    <!-- 상식 퀴즈 모달 -->
    <div id="quiz-modal" class="modal-container">
        <div class="modal-content">
            <span class="close-btn" onclick="closeModal('quiz-modal')">×</span>
            <h2>바닥에 떨어진 쪽지</h2>
            <p>대한민국의 수도는 어디인가?</p>
            <input type="text" id="quiz-answer" placeholder="정답">
            <button onclick="checkQuiz()">정답 확인</button>
        </div>
    </div>

    <!-- 메모리 게임 모달 -->
    <div id="memory-game-modal" class="modal-container">
        <div class="modal-content">
            <span class="close-btn" onclick="closeModal('memory-game-modal')">×</span>
            <h2>낡은 사물함</h2>
            <p>원혼의 흩어진 기억을 맞추시오.</p>
            <div id="memory-board"></div>
            <p id="memory-game-message"></p>
        </div>
    </div>

    <!-- 모든 알림을 위한 범용 모달 -->
    <div id="result-modal" class="modal-container">
        <div class="modal-content">
            <p id="result-text"></p>
            <button class="modal-close-btn" onclick="closeModal('result-modal')">확인</button>
        </div>
    </div>

    <!-- JavaScript 코드 -->
    <script>
        // --- 게임 상태 변수 ---
        let quizSolved = false;
        let memoryGameSolved = false;
        const finalPassword = "0248"; // 최종 비밀번호

        // --- 모달 관리 함수 ---
        function openModal(modalId) {
            document.getElementById(modalId).style.display = 'flex';
        }

        function closeModal(modalId) {
            document.getElementById(modalId).style.display = 'none';
        }

        function showResultModal(message) {
            document.getElementById('result-text').innerText = message;
            openModal('result-modal');
        }

        // --- 상식 퀴즈 기능 ---
        function checkQuiz() {
            const answer = document.getElementById('quiz-answer').value;
            // '서울' 또는 'seoul'을 정답으로 인정
            if (answer.toLowerCase() === "서울" || answer.toLowerCase() === "seoul") {
                quizSolved = true;
                closeModal('quiz-modal');
                showResultModal("정답이다! 쪽지 뒷면에 서울의 지역번호인 숫자 [02]가 나타났다.");
            } else {
                showResultModal("틀렸다... 복도에 싸늘한 기운이 감돈다.");
            }
        }

        // --- 메모리 게임 기능 ---
        const cardEmojis = ['👻', '👻', '💀', '💀', '🔪', '🔪', '🩸', '🩸'];
        let flippedCards = [];
        let matchedPairs = 0;
        let lockBoard = false;

        function shuffle(array) {
            array.sort(() => Math.random() - 0.5);
        }

        function startMemoryGame() {
            if (memoryGameSolved) {
                showResultModal("이미 사물함의 기억을 모두 찾았다.");
                closeModal('memory-game-modal'); // 이미 풀었으면 바로 닫기
                return;
            }
            
            matchedPairs = 0;
            const board = document.getElementById('memory-board');
            board.innerHTML = '';
            document.getElementById('memory-game-message').innerText = '';
            shuffle(cardEmojis);

            cardEmojis.forEach(emoji => {
                const card = document.createElement('div');
                card.classList.add('memory-card');
                card.dataset.emoji = emoji;

                const frontFace = document.createElement('div');
                frontFace.classList.add('card-face');
                const backFace = document.createElement('div');
                backFace.classList.add('card-face', 'card-back');
                backFace.innerText = emoji;
                
                card.appendChild(frontFace);
                card.appendChild(backFace);
                card.addEventListener('click', flipCard);
                board.appendChild(card);
            });
        }

        function flipCard() {
            if (lockBoard || this.classList.contains('is-flipped')) return;

            this.classList.add('is-flipped');
            flippedCards.push(this);

            if (flippedCards.length === 2) {
                lockBoard = true;
                checkForMatch();
            }
        }

        function checkForMatch() {
            const [card1, card2] = flippedCards;
            const isMatch = card1.dataset.emoji === card2.dataset.emoji;

            isMatch ? disableCards() : unflipCards();
        }

        function disableCards() {
            flippedCards[0].classList.add('is-matched');
            flippedCards[1].classList.add('is-matched');
            matchedPairs++;
            resetBoard();
            
            if (matchedPairs === cardEmojis.length / 2) {
                memoryGameSolved = true;
                document.getElementById('memory-game-message').innerText = "모든 기억을 찾았다! 사물함 바닥에 [48]이 새겨져 있다.";
            }
        }

        function unflipCards() {
            setTimeout(() => {
                flippedCards[0].classList.remove('is-flipped');
                flippedCards[1].classList.remove('is-flipped');
                resetBoard();
            }, 1200);
        }
        
        function resetBoard() {
             [flippedCards, lockBoard] = [[], false];
        }

        // --- 최종 비밀번호 확인 ---
        function checkPassword() {
            const userInput = document.getElementById('password-input').value;

            if (!quizSolved || !memoryGameSolved) {
                showResultModal("아직 모든 단서를 찾지 못했다...");
                return;
            }

            if (userInput === finalPassword) {
                showResultModal("철컥! 복도 끝의 문이 열리는 소리가 들린다. 당신은 끔찍한 복도에서 무사히 빠져나왔다. 탈출 성공!");
                document.getElementById('password-input').disabled = true;
                document.querySelector('#password-section button').disabled = true;
            } else {
                showResultModal("비밀번호가 틀렸다. 문이 더욱 단단히 잠기는 것 같다.");
            }
        }

        // 페이지 로드 시 모든 모달을 확실히 숨김
        document.addEventListener('DOMContentLoaded', () => {
            document.querySelectorAll('.modal-container').forEach(modal => modal.style.display = 'none');
        });
    </script>
</body>
</html>```
