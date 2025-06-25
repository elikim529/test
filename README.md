<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>퀴즈 부저 - 완전 독립형</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.4.0/css/all.min.css">
    <style>
        .buzzer-button {
            background: linear-gradient(45deg, #ff6b6b, #ee5a24);
            box-shadow: 0 8px 15px rgba(255, 107, 107, 0.3);
            transition: all 0.3s ease;
        }
        .buzzer-button:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 12px 25px rgba(255, 107, 107, 0.4);
        }
        .buzzer-button:active:not(:disabled) {
            transform: translateY(0);
            box-shadow: 0 4px 10px rgba(255, 107, 107, 0.3);
        }
        .buzzer-button:disabled {
            background: linear-gradient(45deg, #95a5a6, #7f8c8d);
            cursor: not-allowed;
        }
        .host-button {
            background: linear-gradient(45deg, #4834d4, #686de0);
        }
        .join-button {
            background: linear-gradient(45deg, #00d2d3, #01a3a4);
        }
        .winner-display {
            animation: pulse 2s infinite;
        }
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.7; }
        }
        .participant-card {
            transition: all 0.3s ease;
        }
        .participant-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
        }
        .connection-indicator {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1000;
        }
        input {
            color: #000 !important;
        }
        input::placeholder {
            color: #666 !important;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-purple-400 via-pink-500 to-red-500 min-h-screen">
    <!-- 연결 상태 표시 -->
    <div id="connectionStatus" class="connection-indicator bg-white px-3 py-1 rounded-full shadow-lg">
        <span id="statusIcon" class="text-green-500">🟢</span>
        <span id="statusText" class="text-sm font-medium text-gray-700">연결됨</span>
    </div>

    <div class="container mx-auto px-4 py-8">
        <!-- 메인 화면 -->
        <div id="mainScreen" class="text-center">
            <h1 class="text-6xl font-bold text-white mb-4">
                <i class="fas fa-bolt text-yellow-300"></i>
                퀴즈 부저
            </h1>
            <p class="text-xl text-white mb-12 opacity-90">완전 독립형 P2P 버전</p>
            
            <div class="max-w-md mx-auto bg-white rounded-3xl p-8 shadow-2xl">
                <div class="mb-6">
                    <label class="block text-gray-700 text-sm font-bold mb-2">닉네임</label>
                    <input type="text" id="nicknameInput" placeholder="닉네임을 입력하세요" 
                           class="w-full px-4 py-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500">
                </div>
                
                <div class="space-y-4">
                    <button id="createRoomBtn" class="w-full py-4 rounded-xl text-white font-bold text-lg host-button hover:opacity-90 transition-all">
                        <i class="fas fa-plus mr-2"></i>방 만들기
                    </button>
                    <button id="joinRoomBtn" class="w-full py-4 rounded-xl text-white font-bold text-lg join-button hover:opacity-90 transition-all">
                        <i class="fas fa-sign-in-alt mr-2"></i>방 참가
                    </button>
                </div>
            </div>
        </div>

        <!-- 방 참가 화면 -->
        <div id="joinScreen" class="text-center hidden">
            <h2 class="text-4xl font-bold text-white mb-8">방 참가</h2>
            
            <div class="max-w-md mx-auto bg-white rounded-3xl p-8 shadow-2xl">
                <div class="mb-6">
                    <label class="block text-gray-700 text-sm font-bold mb-2">닉네임</label>
                    <input type="text" id="joinNicknameInput" placeholder="닉네임을 입력하세요" 
                           class="w-full px-4 py-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500">
                </div>
                
                <div class="mb-6">
                    <label class="block text-gray-700 text-sm font-bold mb-2">방 코드</label>
                    <input type="text" id="roomCodeInput" placeholder="방 코드를 입력하세요" 
                           class="w-full px-4 py-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500">
                </div>
                
                <div class="space-y-4">
                    <button id="joinConfirmBtn" class="w-full py-4 rounded-xl text-white font-bold text-lg join-button hover:opacity-90 transition-all">
                        <i class="fas fa-arrow-right mr-2"></i>참가하기
                    </button>
                    <button id="backToMainBtn" class="w-full py-3 rounded-xl text-gray-600 font-medium border border-gray-300 hover:bg-gray-50 transition-all">
                        <i class="fas fa-arrow-left mr-2"></i>돌아가기
                    </button>
                </div>
            </div>
        </div>

        <!-- 호스트 화면 -->
        <div id="hostScreen" class="hidden">
            <div class="text-center mb-8">
                <h2 class="text-4xl font-bold text-white mb-4">
                    <i class="fas fa-crown text-yellow-300 mr-2"></i>호스트 대시보드
                </h2>
                <div class="bg-white rounded-2xl p-4 inline-block shadow-lg">
                    <p class="text-gray-600">방 코드</p>
                    <p id="roomCodeDisplay" class="text-3xl font-bold text-purple-600"></p>
                </div>
            </div>

            <div class="grid md:grid-cols-2 gap-8">
                <!-- 참가자 목록 -->
                <div class="bg-white rounded-3xl p-6 shadow-2xl">
                    <h3 class="text-2xl font-bold text-gray-800 mb-6 flex items-center">
                        <i class="fas fa-users mr-2 text-blue-500"></i>참가자 목록
                    </h3>
                    <div id="participantsList" class="space-y-3">
                        <!-- 참가자 카드들이 여기에 추가됩니다 -->
                    </div>
                </div>

                <!-- 게임 제어 -->
                <div class="bg-white rounded-3xl p-6 shadow-2xl">
                    <h3 class="text-2xl font-bold text-gray-800 mb-6 flex items-center">
                        <i class="fas fa-gamepad mr-2 text-green-500"></i>게임 제어
                    </h3>
                    
                    <div class="space-y-6">
                        <div id="gameStatus" class="text-center p-4 rounded-xl bg-gray-100">
                            <p class="text-lg font-medium text-gray-600">게임 대기 중</p>
                        </div>
                        
                        <button id="startGameBtn" class="w-full py-4 rounded-xl text-white font-bold text-lg bg-green-500 hover:bg-green-600 transition-all">
                            <i class="fas fa-play mr-2"></i>게임 시작
                        </button>
                        
                        <button id="stopGameBtn" class="w-full py-4 rounded-xl text-white font-bold text-lg bg-red-500 hover:bg-red-600 transition-all hidden">
                            <i class="fas fa-stop mr-2"></i>게임 정지
                        </button>
                        
                        <button id="nextRoundBtn" class="w-full py-4 rounded-xl text-white font-bold text-lg bg-blue-500 hover:bg-blue-600 transition-all hidden">
                            <i class="fas fa-forward mr-2"></i>다음 라운드
                        </button>
                    </div>
                    
                    <!-- 승자 표시 -->
                    <div id="winnerDisplay" class="mt-6 p-6 rounded-xl bg-gradient-to-r from-yellow-400 to-orange-500 text-white text-center winner-display hidden">
                        <h4 class="text-2xl font-bold mb-2">
                            <i class="fas fa-trophy mr-2"></i>승자!
                        </h4>
                        <p id="winnerInfo" class="text-xl"></p>
                    </div>
                </div>
            </div>
        </div>

        <!-- 클라이언트 화면 -->
        <div id="clientScreen" class="text-center hidden">
            <div class="mb-8">
                <h2 class="text-4xl font-bold text-white mb-4">
                    <i class="fas fa-user mr-2"></i><span id="clientNickname"></span>
                </h2>
                <p class="text-xl text-white opacity-90">참가자 번호: <span id="clientNumber" class="font-bold"></span></p>
            </div>

            <div class="max-w-lg mx-auto">
                <div id="clientGameStatus" class="bg-white rounded-2xl p-4 mb-8 shadow-lg">
                    <p class="text-lg font-medium text-gray-600">게임 대기 중...</p>
                </div>

                <button id="buzzerButton" class="buzzer-button w-full h-64 rounded-3xl text-white font-bold text-4xl shadow-2xl disabled:cursor-not-allowed" disabled>
                    <i class="fas fa-hand-pointer text-6xl mb-4 block"></i>
                    BUZZER!
                </button>

                <div class="mt-8">
                    <button id="leaveRoomBtn" class="px-6 py-3 rounded-xl text-white font-medium bg-red-500 hover:bg-red-600 transition-all">
                        <i class="fas fa-sign-out-alt mr-2"></i>방 나가기
                    </button>
                </div>
            </div>
        </div>
    </div>

    <script>
        class QuizBuzzerApp {
            constructor() {
                this.currentUser = null;
                this.currentRoom = null;
                this.isHost = false;
                this.participants = [];
                this.gameStatus = 'waiting'; // waiting, playing, stopped
                this.winner = null;
                this.pollInterval = null;
                
                this.initializeEventListeners();
                this.updateConnectionStatus('connected');
            }

            initializeEventListeners() {
                // 메인 화면 버튼들
                document.getElementById('createRoomBtn').addEventListener('click', () => this.createRoom());
                document.getElementById('joinRoomBtn').addEventListener('click', () => this.showJoinScreen());
                
                // 방 참가 화면 버튼들
                document.getElementById('joinConfirmBtn').addEventListener('click', () => this.joinRoom());
                document.getElementById('backToMainBtn').addEventListener('click', () => this.showMainScreen());
                
                // 호스트 화면 버튼들
                document.getElementById('startGameBtn').addEventListener('click', () => this.startGame());
                document.getElementById('stopGameBtn').addEventListener('click', () => this.stopGame());
                document.getElementById('nextRoundBtn').addEventListener('click', () => this.nextRound());
                
                // 클라이언트 화면 버튼들
                document.getElementById('buzzerButton').addEventListener('click', () => this.pressBuzzer());
                document.getElementById('leaveRoomBtn').addEventListener('click', () => this.leaveRoom());
            }

            updateConnectionStatus(status) {
                const statusIcon = document.getElementById('statusIcon');
                const statusText = document.getElementById('statusText');
                
                switch(status) {
                    case 'connected':
                        statusIcon.textContent = '🟢';
                        statusText.textContent = '연결됨';
                        break;
                    case 'connecting':
                        statusIcon.textContent = '🟡';
                        statusText.textContent = '연결 중';
                        break;
                    case 'disconnected':
                        statusIcon.textContent = '🔴';
                        statusText.textContent = '연결 끊김';
                        break;
                }
            }

            generateRoomCode() {
                return 'ROOM' + Math.random().toString(36).substr(2, 4).toUpperCase();
            }

            generateUserId() {
                return 'user_' + Math.random().toString(36).substr(2, 9);
            }

            createRoom() {
                const nickname = document.getElementById('nicknameInput').value.trim();
                if (!nickname) {
                    alert('닉네임을 입력해주세요.');
                    return;
                }

                this.currentUser = {
                    id: this.generateUserId(),
                    nickname: nickname
                };

                this.currentRoom = {
                    code: this.generateRoomCode(),
                    hostId: this.currentUser.id,
                    participants: [
                        {
                            id: this.currentUser.id,
                            nickname: nickname,
                            number: 1,
                            connected: true,
                            joinedAt: Date.now()
                        }
                    ],
                    gameStatus: 'waiting',
                    winner: null,
                    createdAt: Date.now()
                };

                this.isHost = true;
                this.participants = this.currentRoom.participants;
                
                // 로컬 스토리지에 저장
                localStorage.setItem('quiz_room_' + this.currentRoom.code, JSON.stringify(this.currentRoom));
                localStorage.setItem('quiz_current_user', JSON.stringify(this.currentUser));
                localStorage.setItem('quiz_current_room_code', this.currentRoom.code);
                localStorage.setItem('quiz_is_host', 'true');

                this.showHostScreen();
                this.startPolling();
            }

            showJoinScreen() {
                this.hideAllScreens();
                document.getElementById('joinScreen').classList.remove('hidden');
            }

            joinRoom() {
                const nickname = document.getElementById('joinNicknameInput').value.trim();
                const roomCode = document.getElementById('roomCodeInput').value.trim().toUpperCase();
                
                if (!nickname || !roomCode) {
                    alert('닉네임과 방 코드를 모두 입력해주세요.');
                    return;
                }

                // 방 존재 확인
                const roomData = localStorage.getItem('quiz_room_' + roomCode);
                if (!roomData) {
                    alert('존재하지 않는 방 코드입니다.');
                    return;
                }

                const room = JSON.parse(roomData);
                
                // 중복 닉네임 확인
                if (room.participants.some(p => p.nickname === nickname)) {
                    alert('이미 사용 중인 닉네임입니다.');
                    return;
                }

                this.currentUser = {
                    id: this.generateUserId(),
                    nickname: nickname
                };

                // 참가자 추가
                const newParticipant = {
                    id: this.currentUser.id,
                    nickname: nickname,
                    number: room.participants.length + 1,
                    connected: true,
                    joinedAt: Date.now()
                };

                room.participants.push(newParticipant);
                
                // 업데이트된 방 정보 저장
                localStorage.setItem('quiz_room_' + roomCode, JSON.stringify(room));
                localStorage.setItem('quiz_current_user', JSON.stringify(this.currentUser));
                localStorage.setItem('quiz_current_room_code', roomCode);
                localStorage.setItem('quiz_is_host', 'false');

                this.currentRoom = room;
                this.isHost = false;
                this.participants = room.participants;

                this.showClientScreen();
                this.startPolling();
            }

            startGame() {
                if (!this.isHost) return;

                this.gameStatus = 'playing';
                this.winner = null;
                this.currentRoom.gameStatus = 'playing';
                this.currentRoom.winner = null;

                localStorage.setItem('quiz_room_' + this.currentRoom.code, JSON.stringify(this.currentRoom));

                this.updateHostUI();
            }

            stopGame() {
                if (!this.isHost) return;

                this.gameStatus = 'stopped';
                this.currentRoom.gameStatus = 'stopped';

                localStorage.setItem('quiz_room_' + this.currentRoom.code, JSON.stringify(this.currentRoom));

                this.updateHostUI();
            }

            nextRound() {
                if (!this.isHost) return;

                this.gameStatus = 'waiting';
                this.winner = null;
                this.currentRoom.gameStatus = 'waiting';
                this.currentRoom.winner = null;

                localStorage.setItem('quiz_room_' + this.currentRoom.code, JSON.stringify(this.currentRoom));

                this.updateHostUI();
            }

            pressBuzzer() {
                if (this.gameStatus !== 'playing') return;

                const participant = this.participants.find(p => p.id === this.currentUser.id);
                if (!participant) return;

                // 승자 설정
                this.winner = {
                    id: participant.id,
                    nickname: participant.nickname,
                    number: participant.number,
                    timestamp: Date.now()
                };

                this.currentRoom.winner = this.winner;
                this.currentRoom.gameStatus = 'stopped';

                localStorage.setItem('quiz_room_' + this.currentRoom.code, JSON.stringify(this.currentRoom));

                // 버튼 비활성화
                document.getElementById('buzzerButton').disabled = true;
                this.updateClientUI();
            }

            leaveRoom() {
                if (this.isHost) {
                    // 호스트가 나가면 방 삭제
                    localStorage.removeItem('quiz_room_' + this.currentRoom.code);
                } else {
                    // 참가자가 나가면 목록에서 제거
                    const roomData = localStorage.getItem('quiz_room_' + this.currentRoom.code);
                    if (roomData) {
                        const room = JSON.parse(roomData);
                        room.participants = room.participants.filter(p => p.id !== this.currentUser.id);
                        localStorage.setItem('quiz_room_' + this.currentRoom.code, JSON.stringify(room));
                    }
                }

                localStorage.removeItem('quiz_current_user');
                localStorage.removeItem('quiz_current_room_code');
                localStorage.removeItem('quiz_is_host');

                this.stopPolling();
                this.showMainScreen();
            }

            startPolling() {
                this.stopPolling();
                this.pollInterval = setInterval(() => {
                    this.syncRoomData();
                }, 1000);
            }

            stopPolling() {
                if (this.pollInterval) {
                    clearInterval(this.pollInterval);
                    this.pollInterval = null;
                }
            }

            syncRoomData() {
                if (!this.currentRoom) return;

                const roomData = localStorage.getItem('quiz_room_' + this.currentRoom.code);
                if (!roomData) {
                    // 방이 삭제됨
                    alert('방이 종료되었습니다.');
                    this.showMainScreen();
                    return;
                }

                const room = JSON.parse(roomData);
                this.currentRoom = room;
                this.participants = room.participants;
                this.gameStatus = room.gameStatus;
                this.winner = room.winner;

                if (this.isHost) {
                    this.updateHostUI();
                } else {
                    this.updateClientUI();
                }
            }

            updateHostUI() {
                // 방 코드 표시
                document.getElementById('roomCodeDisplay').textContent = this.currentRoom.code;

                // 참가자 목록 업데이트
                const participantsList = document.getElementById('participantsList');
                participantsList.innerHTML = '';

                this.participants.forEach(p => {
                    const participantCard = document.createElement('div');
                    participantCard.className = 'participant-card bg-gray-50 p-4 rounded-xl flex justify-between items-center';
                    participantCard.innerHTML = `
                        <div class="flex items-center">
                            <div class="w-10 h-10 bg-blue-500 text-white rounded-full flex items-center justify-center font-bold mr-3">
                                ${p.number}
                            </div>
                            <div>
                                <p class="font-medium text-gray-800">${p.nickname}</p>
                                <p class="text-sm text-gray-500">${p.id === this.currentRoom.hostId ? '호스트' : '참가자'}</p>
                            </div>
                        </div>
                        <div class="text-green-500">
                            <i class="fas fa-circle"></i>
                        </div>
                    `;
                    participantsList.appendChild(participantCard);
                });

                // 게임 상태 업데이트
                const gameStatus = document.getElementById('gameStatus');
                const startBtn = document.getElementById('startGameBtn');
                const stopBtn = document.getElementById('stopGameBtn');
                const nextBtn = document.getElementById('nextRoundBtn');
                const winnerDisplay = document.getElementById('winnerDisplay');

                switch(this.gameStatus) {
                    case 'waiting':
                        gameStatus.innerHTML = '<p class="text-lg font-medium text-gray-600">게임 대기 중</p>';
                        startBtn.classList.remove('hidden');
                        stopBtn.classList.add('hidden');
                        nextBtn.classList.add('hidden');
                        winnerDisplay.classList.add('hidden');
                        break;
                    case 'playing':
                        gameStatus.innerHTML = '<p class="text-lg font-medium text-green-600">게임 진행 중</p>';
                        startBtn.classList.add('hidden');
                        stopBtn.classList.remove('hidden');
                        nextBtn.classList.add('hidden');
                        winnerDisplay.classList.add('hidden');
                        break;
                    case 'stopped':
                        gameStatus.innerHTML = '<p class="text-lg font-medium text-red-600">게임 정지</p>';
                        startBtn.classList.remove('hidden');
                        stopBtn.classList.add('hidden');
                        if (this.winner) {
                            nextBtn.classList.remove('hidden');
                            winnerDisplay.classList.remove('hidden');
                            document.getElementById('winnerInfo').textContent = 
                                `${this.winner.number}번 - ${this.winner.nickname}`;
                        }
                        break;
                }
            }

            updateClientUI() {
                const participant = this.participants.find(p => p.id === this.currentUser.id);
                if (!participant) return;

                document.getElementById('clientNickname').textContent = participant.nickname;
                document.getElementById('clientNumber').textContent = participant.number;

                const clientGameStatus = document.getElementById('clientGameStatus');
                const buzzerButton = document.getElementById('buzzerButton');

                switch(this.gameStatus) {
                    case 'waiting':
                        clientGameStatus.innerHTML = '<p class="text-lg font-medium text-gray-600">게임 대기 중...</p>';
                        buzzerButton.disabled = true;
                        break;
                    case 'playing':
                        clientGameStatus.innerHTML = '<p class="text-lg font-medium text-green-600">게임 진행 중! 부저를 누르세요!</p>';
                        buzzerButton.disabled = false;
                        break;
                    case 'stopped':
                        if (this.winner) {
                            const isWinner = this.winner.id === this.currentUser.id;
                            clientGameStatus.innerHTML = `
                                <p class="text-lg font-medium text-red-600">게임 종료</p>
                                <p class="text-sm ${isWinner ? 'text-green-600 font-bold' : 'text-gray-500'}">
                                    승자: ${this.winner.number}번 - ${this.winner.nickname}
                                    ${isWinner ? ' (축하합니다!)' : ''}
                                </p>
                            `;
                        } else {
                            clientGameStatus.innerHTML = '<p class="text-lg font-medium text-red-600">게임 정지</p>';
                        }
                        buzzerButton.disabled = true;
                        break;
                }
            }

            showMainScreen() {
                this.hideAllScreens();
                document.getElementById('mainScreen').classList.remove('hidden');
                this.stopPolling();
                
                // 입력 필드 초기화
                document.getElementById('nicknameInput').value = '';
                document.getElementById('joinNicknameInput').value = '';
                document.getElementById('roomCodeInput').value = '';
            }

            showHostScreen() {
                this.hideAllScreens();
                document.getElementById('hostScreen').classList.remove('hidden');
                this.updateHostUI();
            }

            showClientScreen() {
                this.hideAllScreens();
                document.getElementById('clientScreen').classList.remove('hidden');
                this.updateClientUI();
            }

            hideAllScreens() {
                document.getElementById('mainScreen').classList.add('hidden');
                document.getElementById('joinScreen').classList.add('hidden');
                document.getElementById('hostScreen').classList.add('hidden');
                document.getElementById('clientScreen').classList.add('hidden');
            }
        }

        // 앱 초기화
        const app = new QuizBuzzerApp();

        // 페이지 로드 시 이전 세션 복구
        window.addEventListener('load', () => {
            const currentUser = localStorage.getItem('quiz_current_user');
            const currentRoomCode = localStorage.getItem('quiz_current_room_code');
            const isHost = localStorage.getItem('quiz_is_host') === 'true';

            if (currentUser && currentRoomCode) {
                app.currentUser = JSON.parse(currentUser);
                
                const roomData = localStorage.getItem('quiz_room_' + currentRoomCode);
                if (roomData) {
                    app.currentRoom = JSON.parse(roomData);
                    app.isHost = isHost;
                    app.participants = app.currentRoom.participants;
                    app.gameStatus = app.currentRoom.gameStatus;
                    app.winner = app.currentRoom.winner;

                    if (isHost) {
                        app.showHostScreen();
                    } else {
                        app.showClientScreen();
                    }
                    app.startPolling();
                }
            }
        });

        // 페이지 언로드 시 정리
        window.addEventListener('beforeunload', () => {
            if (app.currentRoom && !app.isHost) {
                // 참가자 연결 상태 업데이트
                const roomData = localStorage.getItem('quiz_room_' + app.currentRoom.code);
                if (roomData) {
                    const room = JSON.parse(roomData);
                    const participant = room.participants.find(p => p.id === app.currentUser.id);
                    if (participant) {
                        participant.connected = false;
                        localStorage.setItem('quiz_room_' + app.currentRoom.code, JSON.stringify(room));
                    }
                }
            }
        });
    </script>
<script defer src="https://static.cloudflareinsights.com/beacon.min.js/vcd15cbe7772f49c399c6a5babf22c1241717689176015" integrity="sha512-ZpsOmlRQV6y907TI0dKBHq9Md29nnaEIPlkf84rnaERnq6zvWvPUqr2ft8M1aS28oN72PdrCzSjY4U6VaAw1EQ==" data-cf-beacon='{"rayId":"955569ee6a5b29da","serverTiming":{"name":{"cfExtPri":true,"cfEdge":true,"cfOrigin":true,"cfL4":true,"cfSpeedBrain":true,"cfCacheStatus":true}},"version":"2025.6.2","token":"4edd5f8ec12a48cfa682ab8261b80a79"}' crossorigin="anonymous"></script>
</body>
</html>
    <script id="html_badge_script1">
        window.__genspark_remove_badge_link = "https://www.genspark.ai/api/html_badge/" +
            "remove_badge?token=To%2FBnjzloZ3UfQdcSaYfDmPpzTxUjNGUERaw4tVyT5eLSuXX6wtDG0HMnoynHay%2FhQ%2BD4%2FJryVVBHaHz9rBJqgvJyatzmp%2FpbrVXhuH%2B%2FXV%2BeTk0pC3i5hOMa6Mx5X1LHQEerYPWjA%2B9XX5f8A2TtAj750oPotyCick%2BkZJNtobLU5k2BFQxfrIi4prKVT5%2B20DjYZlWF8nK9TMb0Q77O8F6GG0PxRSpGJDtalPmdPzJS6qtvs4MRLP%2FIx4rWvXomMiQuWGiNzmOeQX3MMEG47Z7bJmbDS%2BmC%2BLXCNVz2PpcmtxGHZiMSKv02h2nODZJWiFcXhC9A8EgwGs5wAb4KH4KBHsmy9LaicZPFz5kOxKDgtpe%2FMDGSpIzBDujJavBJEhJa1mE6bvr9YzNci6J555HQB%2FB%2BOn74Id5HsljU6ET8blUFD2wPde4H%2BgK%2BlidwIWTahCk4SS%2Bn0s0dRlBJeV%2ByQd3t76ulbFq6OpW3sogPTrUTLy5815XMIfo8m1xMoKTsJpplJv7e1k6HNnFeVboTaSat1TgvbH3y4J%2FZ%2BA%3D";
        window.__genspark_locale = "ko-KR";
        window.__genspark_token = "To/BnjzloZ3UfQdcSaYfDmPpzTxUjNGUERaw4tVyT5eLSuXX6wtDG0HMnoynHay/hQ+D4/JryVVBHaHz9rBJqgvJyatzmp/pbrVXhuH+/XV+eTk0pC3i5hOMa6Mx5X1LHQEerYPWjA+9XX5f8A2TtAj750oPotyCick+kZJNtobLU5k2BFQxfrIi4prKVT5+20DjYZlWF8nK9TMb0Q77O8F6GG0PxRSpGJDtalPmdPzJS6qtvs4MRLP/Ix4rWvXomMiQuWGiNzmOeQX3MMEG47Z7bJmbDS+mC+LXCNVz2PpcmtxGHZiMSKv02h2nODZJWiFcXhC9A8EgwGs5wAb4KH4KBHsmy9LaicZPFz5kOxKDgtpe/MDGSpIzBDujJavBJEhJa1mE6bvr9YzNci6J555HQB/B+On74Id5HsljU6ET8blUFD2wPde4H+gK+lidwIWTahCk4SS+n0s0dRlBJeV+yQd3t76ulbFq6OpW3sogPTrUTLy5815XMIfo8m1xMoKTsJpplJv7e1k6HNnFeVboTaSat1TgvbH3y4J/Z+A=";
    </script>
    
    <script id="html_notice_dialog_script" src="https://www.genspark.ai/notice_dialog.js"></script>
    
