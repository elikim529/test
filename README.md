<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>슬롯 게임 정보 수집 프로그램</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.4.0/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
</head>
<body class="bg-gray-100 min-h-screen">
    <div class="container mx-auto px-4 py-8">
        <!-- 헤더 -->
        <div class="bg-white rounded-lg shadow-md p-6 mb-6">
            <h1 class="text-3xl font-bold text-gray-800 mb-2">
                <i class="fas fa-gamepad text-blue-600 mr-3"></i>
                슬롯 게임 정보 수집 프로그램
            </h1>
            <p class="text-gray-600">CSV 파일에서 슬롯 게임 URL을 읽어와 베팅 정보, RTP, Buy Feature 등을 자동 수집합니다.</p>
        </div>

        <!-- CSV 업로드 섹션 -->
        <div class="bg-white rounded-lg shadow-md p-6 mb-6">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">
                <i class="fas fa-upload text-green-600 mr-2"></i>
                CSV 파일 업로드
            </h2>
            <div class="border-2 border-dashed border-gray-300 rounded-lg p-6 text-center">
                <input type="file" id="csvFile" accept=".csv" class="hidden">
                <label for="csvFile" class="cursor-pointer">
                    <i class="fas fa-cloud-upload-alt text-4xl text-gray-400 mb-4 block"></i>
                    <p class="text-gray-600">CSV 파일을 선택하거나 여기에 드래그하세요</p>
                    <p class="text-sm text-gray-500 mt-2">A열에 게임 URL이 포함된 CSV 파일</p>
                </label>
            </div>
            <div id="fileInfo" class="mt-4 text-sm text-gray-600 hidden">
                <i class="fas fa-file-csv text-green-600 mr-2"></i>
                <span id="fileName"></span> - <span id="gameCount"></span>개 게임 발견
            </div>
        </div>

        <!-- 제어 패널 -->
        <div class="bg-white rounded-lg shadow-md p-6 mb-6">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">
                <i class="fas fa-cogs text-purple-600 mr-2"></i>
                수집 설정
            </h2>
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-2">대기 시간 (초)</label>
                    <input type="number" id="waitTime" value="5" min="1" max="30" class="w-full border border-gray-300 rounded-md px-3 py-2">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-2">재시도 횟수</label>
                    <input type="number" id="retryCount" value="3" min="1" max="10" class="w-full border border-gray-300 rounded-md px-3 py-2">
                </div>
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-2">자동 스크롤</label>
                    <select id="autoScroll" class="w-full border border-gray-300 rounded-md px-3 py-2">
                        <option value="true">활성화</option>
                        <option value="false">비활성화</option>
                    </select>
                </div>
            </div>
            <button id="startAnalysis" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded disabled:opacity-50" disabled>
                <i class="fas fa-play mr-2"></i>
                분석 시작
            </button>
            <button id="stopAnalysis" class="bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded ml-2 hidden">
                <i class="fas fa-stop mr-2"></i>
                중지
            </button>
        </div>

        <!-- 진행 상황 -->
        <div id="progressSection" class="bg-white rounded-lg shadow-md p-6 mb-6 hidden">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">
                <i class="fas fa-chart-line text-yellow-600 mr-2"></i>
                진행 상황
            </h2>
            <div class="bg-gray-200 rounded-full h-4 mb-4">
                <div id="progressBar" class="bg-blue-600 h-4 rounded-full transition-all duration-300" style="width: 0%"></div>
            </div>
            <div class="flex justify-between text-sm text-gray-600">
                <span id="currentGame">준비 중...</span>
                <span id="progressText">0 / 0</span>
            </div>
        </div>

        <!-- 웹뷰 섹션 -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
            <div class="bg-white rounded-lg shadow-md p-6">
                <h2 class="text-xl font-semibold text-gray-800 mb-4">
                    <i class="fas fa-window-maximize text-indigo-600 mr-2"></i>
                    게임 웹뷰
                </h2>
                <div class="border border-gray-300 rounded-lg overflow-hidden" style="height: 400px;">
                    <iframe id="gameFrame" class="w-full h-full" sandbox="allow-scripts allow-same-origin allow-forms"></iframe>
                </div>
                <div class="mt-4 text-sm text-gray-600">
                    <div id="currentUrl" class="truncate">URL: 게임을 선택하세요</div>
                    <div id="loadStatus" class="mt-1">상태: 대기 중</div>
                </div>
            </div>

            <div class="bg-white rounded-lg shadow-md p-6">
                <h2 class="text-xl font-semibold text-gray-800 mb-4">
                    <i class="fas fa-search text-orange-600 mr-2"></i>
                    실시간 정보 추출
                </h2>
                <div class="space-y-4">
                    <div class="bg-gray-50 rounded-lg p-4">
                        <h3 class="font-semibold text-gray-700 mb-2">베팅 금액</h3>
                        <div class="grid grid-cols-2 gap-4 text-sm">
                            <div>최소: <span id="minBet" class="font-mono text-green-600">-</span></div>
                            <div>최대: <span id="maxBet" class="font-mono text-red-600">-</span></div>
                        </div>
                    </div>
                    <div class="bg-gray-50 rounded-lg p-4">
                        <h3 class="font-semibold text-gray-700 mb-2">RTP</h3>
                        <div id="rtpValue" class="font-mono text-blue-600">-</div>
                    </div>
                    <div class="bg-gray-50 rounded-lg p-4">
                        <h3 class="font-semibold text-gray-700 mb-2">Buy Feature/Bonus</h3>
                        <div id="buyFeature" class="font-mono text-purple-600">-</div>
                    </div>
                </div>
                <button id="manualExtract" class="mt-4 bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded w-full">
                    <i class="fas fa-hand-pointer mr-2"></i>
                    수동 정보 추출
                </button>
            </div>
        </div>

        <!-- 결과 테이블 -->
        <div class="bg-white rounded-lg shadow-md p-6">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">
                <i class="fas fa-table text-teal-600 mr-2"></i>
                수집 결과
            </h2>
            <div class="overflow-x-auto">
                <table class="min-w-full table-auto">
                    <thead class="bg-gray-50">
                        <tr>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">순번</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">게임 URL</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">최소 베팅</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">최대 베팅</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">RTP</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Buy Feature</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">상태</th>
                        </tr>
                    </thead>
                    <tbody id="resultsTable" class="bg-white divide-y divide-gray-200">
                        <tr>
                            <td colspan="7" class="px-4 py-8 text-center text-gray-500">
                                <i class="fas fa-inbox text-3xl mb-2 block"></i>
                                CSV 파일을 업로드하여 시작하세요
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="mt-4 flex justify-between items-center">
                <div class="text-sm text-gray-600">
                    총 <span id="totalGames">0</span>개 게임 중 <span id="completedGames">0</span>개 완료
                </div>
                <button id="exportResults" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded">
                    <i class="fas fa-download mr-2"></i>
                    CSV 내보내기
                </button>
            </div>
        </div>
    </div>

    <script>
        class SlotGameAnalyzer {
            constructor() {
                this.games = [];
                this.currentIndex = 0;
                this.isRunning = false;
                this.results = [];
                this.initializeEventListeners();
            }

            initializeEventListeners() {
                // CSV 파일 업로드
                document.getElementById('csvFile').addEventListener('change', (e) => this.handleFileUpload(e));
                
                // 드래그 앤 드롭
                const uploadArea = document.querySelector('label[for="csvFile"]').parentElement;
                uploadArea.addEventListener('dragover', (e) => {
                    e.preventDefault();
                    uploadArea.classList.add('border-blue-400', 'bg-blue-50');
                });
                uploadArea.addEventListener('dragleave', () => {
                    uploadArea.classList.remove('border-blue-400', 'bg-blue-50');
                });
                uploadArea.addEventListener('drop', (e) => {
                    e.preventDefault();
                    uploadArea.classList.remove('border-blue-400', 'bg-blue-50');
                    const files = e.dataTransfer.files;
                    if (files.length > 0) {
                        this.handleFileUpload({ target: { files } });
                    }
                });

                // 분석 컨트롤
                document.getElementById('startAnalysis').addEventListener('click', () => this.startAnalysis());
                document.getElementById('stopAnalysis').addEventListener('click', () => this.stopAnalysis());
                document.getElementById('manualExtract').addEventListener('click', () => this.extractCurrentGameInfo());
                document.getElementById('exportResults').addEventListener('click', () => this.exportResults());
            }

            handleFileUpload(event) {
                const file = event.target.files[0];
                if (!file) return;

                Papa.parse(file, {
                    complete: (results) => {
                        this.games = results.data
                            .map((row, index) => ({ url: row[0]?.trim(), index: index + 1 }))
                            .filter(game => game.url && game.url.startsWith('http'));
                        
                        this.updateFileInfo(file.name, this.games.length);
                        this.updateResultsTable();
                        document.getElementById('startAnalysis').disabled = false;
                    },
                    error: (error) => {
                        alert('CSV 파일 읽기 오류: ' + error.message);
                    }
                });
            }

            updateFileInfo(fileName, gameCount) {
                document.getElementById('fileName').textContent = fileName;
                document.getElementById('gameCount').textContent = gameCount;
                document.getElementById('fileInfo').classList.remove('hidden');
                document.getElementById('totalGames').textContent = gameCount;
            }

            async startAnalysis() {
                if (this.games.length === 0) return;

                this.isRunning = true;
                this.currentIndex = 0;
                document.getElementById('startAnalysis').classList.add('hidden');
                document.getElementById('stopAnalysis').classList.remove('hidden');
                document.getElementById('progressSection').classList.remove('hidden');

                for (let i = 0; i < this.games.length && this.isRunning; i++) {
                    this.currentIndex = i;
                    await this.analyzeGame(this.games[i]);
                    this.updateProgress();
                }

                this.stopAnalysis();
            }

            stopAnalysis() {
                this.isRunning = false;
                document.getElementById('startAnalysis').classList.remove('hidden');
                document.getElementById('stopAnalysis').classList.add('hidden');
            }

            async analyzeGame(game) {
                try {
                    // 게임 로드
                    document.getElementById('currentGame').textContent = `게임 ${game.index} 분석 중...`;
                    document.getElementById('currentUrl').textContent = `URL: ${game.url}`;
                    document.getElementById('loadStatus').textContent = '상태: 로딩 중...';
                    
                    const iframe = document.getElementById('gameFrame');
                    iframe.src = game.url;

                    // 로드 대기
                    const waitTime = parseInt(document.getElementById('waitTime').value) * 1000;
                    await this.sleep(waitTime);

                    // 정보 추출
                    const gameInfo = await this.extractGameInfo(iframe);
                    
                    // 결과 저장
                    this.results[game.index - 1] = {
                        index: game.index,
                        url: game.url,
                        minBet: gameInfo.minBet,
                        maxBet: gameInfo.maxBet,
                        rtp: gameInfo.rtp,
                        buyFeature: gameInfo.buyFeature,
                        status: gameInfo.status
                    };

                    document.getElementById('loadStatus').textContent = '상태: 완료';
                    this.updateResultsTable();

                } catch (error) {
                    console.error('게임 분석 오류:', error);
                    this.results[game.index - 1] = {
                        index: game.index,
                        url: game.url,
                        minBet: '-',
                        maxBet: '-',
                        rtp: '-',
                        buyFeature: '-',
                        status: '오류'
                    };
                }
            }

            async extractGameInfo(iframe) {
                try {
                    // iframe 컨텐츠 접근 시도
                    const iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
                    
                    const info = {
                        minBet: '-',
                        maxBet: '-',
                        rtp: '-',
                        buyFeature: '-',
                        status: '성공'
                    };

                    // 베팅 금액 추출 패턴들
                    const betPatterns = [
                        // 일반적인 패턴들
                        'bet', 'coin', 'stake', 'wager',
                        // 한국어 패턴들
                        '베팅', '코인', '판돈',
                        // 특수 패턴들
                        'min-bet', 'max-bet', 'total-bet'
                    ];

                    // RTP 추출 패턴들
                    const rtpPatterns = [
                        'rtp', 'return', 'payout', '환급률', '수익률'
                    ];

                    // Buy Feature 추출 패턴들
                    const buyFeaturePatterns = [
                        'buy feature', 'buy bonus', 'feature buy', 'bonus buy',
                        '피처 구매', '보너스 구매', 'instant play'
                    ];

                    // DOM에서 정보 추출
                    const allText = iframeDoc.body?.innerText?.toLowerCase() || '';
                    const allElements = iframeDoc.querySelectorAll('*') || [];

                    // 베팅 금액 찾기
                    const betAmounts = this.findBetAmounts(allElements, allText);
                    if (betAmounts.length > 0) {
                        info.minBet = Math.min(...betAmounts).toString();
                        info.maxBet = Math.max(...betAmounts).toString();
                    }

                    // RTP 찾기
                    info.rtp = this.findRTP(allElements, allText);

                    // Buy Feature 찾기
                    info.buyFeature = this.findBuyFeature(allElements, allText) ? '있음' : '없음';

                    return info;

                } catch (error) {
                    // CORS 오류 등으로 iframe 접근이 불가능한 경우
                    return {
                        minBet: '접근 불가',
                        maxBet: '접근 불가',
                        rtp: '접근 불가',
                        buyFeature: '접근 불가',
                        status: 'CORS 제한'
                    };
                }
            }

            findBetAmounts(elements, text) {
                const amounts = [];
                const numberPattern = /(\d+(?:\.\d+)?)/g;
                
                // 요소들에서 베팅 관련 숫자 찾기
                elements.forEach(element => {
                    const elementText = element.textContent?.toLowerCase() || '';
                    if (this.containsBetKeyword(elementText)) {
                        const matches = elementText.match(numberPattern);
                        if (matches) {
                            matches.forEach(match => {
                                const num = parseFloat(match);
                                if (num > 0 && num < 10000) { // 합리적인 범위의 베팅 금액
                                    amounts.push(num);
                                }
                            });
                        }
                    }
                });

                return [...new Set(amounts)].sort((a, b) => a - b);
            }

            containsBetKeyword(text) {
                const keywords = ['bet', 'coin', 'stake', 'wager', '베팅', '코인', '판돈'];
                return keywords.some(keyword => text.includes(keyword));
            }

            findRTP(elements, text) {
                const rtpPattern = /(\d{2,3}(?:\.\d+)?)\s*%/g;
                
                for (const element of elements) {
                    const elementText = element.textContent || '';
                    if (elementText.toLowerCase().includes('rtp') || 
                        elementText.includes('환급률') || 
                        elementText.toLowerCase().includes('return')) {
                        const matches = elementText.match(rtpPattern);
                        if (matches) {
                            return matches[0];
                        }
                    }
                }
                return '-';
            }

            findBuyFeature(elements, text) {
                const buyKeywords = [
                    'buy feature', 'buy bonus', 'feature buy', 'bonus buy',
                    '피처 구매', '보너스 구매', 'instant play'
                ];

                return buyKeywords.some(keyword => 
                    text.includes(keyword.toLowerCase()) ||
                    Array.from(elements).some(el => 
                        el.textContent?.toLowerCase().includes(keyword.toLowerCase())
                    )
                );
            }

            async extractCurrentGameInfo() {
                const iframe = document.getElementById('gameFrame');
                if (!iframe.src) {
                    alert('먼저 게임을 로드해주세요.');
                    return;
                }

                try {
                    const info = await this.extractGameInfo(iframe);
                    
                    document.getElementById('minBet').textContent = info.minBet;
                    document.getElementById('maxBet').textContent = info.maxBet;
                    document.getElementById('rtpValue').textContent = info.rtp;
                    document.getElementById('buyFeature').textContent = info.buyFeature;

                } catch (error) {
                    alert('정보 추출 중 오류가 발생했습니다: ' + error.message);
                }
            }

            updateProgress() {
                const progress = ((this.currentIndex + 1) / this.games.length) * 100;
                document.getElementById('progressBar').style.width = `${progress}%`;
                document.getElementById('progressText').textContent = `${this.currentIndex + 1} / ${this.games.length}`;
                document.getElementById('completedGames').textContent = this.currentIndex + 1;
            }

            updateResultsTable() {
                const tbody = document.getElementById('resultsTable');
                if (this.games.length === 0) {
                    tbody.innerHTML = `
                        <tr>
                            <td colspan="7" class="px-4 py-8 text-center text-gray-500">
                                <i class="fas fa-inbox text-3xl mb-2 block"></i>
                                CSV 파일을 업로드하여 시작하세요
                            </td>
                        </tr>
                    `;
                    return;
                }

                tbody.innerHTML = this.games.map((game, index) => {
                    const result = this.results[index] || {};
                    const statusColor = this.getStatusColor(result.status);
                    
                    return `
                        <tr class="hover:bg-gray-50">
                            <td class="px-4 py-4 whitespace-nowrap text-sm font-medium text-gray-900">${game.index}</td>
                            <td class="px-4 py-4 text-sm text-gray-900 max-w-xs truncate" title="${game.url}">${game.url}</td>
                            <td class="px-4 py-4 whitespace-nowrap text-sm text-gray-900 font-mono">${result.minBet || '-'}</td>
                            <td class="px-4 py-4 whitespace-nowrap text-sm text-gray-900 font-mono">${result.maxBet || '-'}</td>
                            <td class="px-4 py-4 whitespace-nowrap text-sm text-gray-900 font-mono">${result.rtp || '-'}</td>
                            <td class="px-4 py-4 whitespace-nowrap text-sm text-gray-900">${result.buyFeature || '-'}</td>
                            <td class="px-4 py-4 whitespace-nowrap">
                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${statusColor}">
                                    ${result.status || '대기 중'}
                                </span>
                            </td>
                        </tr>
                    `;
                }).join('');
            }

            getStatusColor(status) {
                switch (status) {
                    case '성공': return 'bg-green-100 text-green-800';
                    case '오류': return 'bg-red-100 text-red-800';
                    case 'CORS 제한': return 'bg-yellow-100 text-yellow-800';
                    default: return 'bg-gray-100 text-gray-800';
                }
            }

            exportResults() {
                if (this.results.length === 0) {
                    alert('내보낼 결과가 없습니다.');
                    return;
                }

                const csvContent = [
                    ['순번', '게임 URL', '최소 베팅', '최대 베팅', 'RTP', 'Buy Feature', '상태'],
                    ...this.results.map(result => [
                        result.index,
                        result.url,
                        result.minBet,
                        result.maxBet,
                        result.rtp,
                        result.buyFeature,
                        result.status
                    ])
                ];

                const csv = Papa.unparse(csvContent);
                const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
                const link = document.createElement('a');
                link.href = URL.createObjectURL(blob);
                link.download = `slot_game_analysis_${new Date().toISOString().split('T')[0]}.csv`;
                link.click();
            }

            sleep(ms) {
                return new Promise(resolve => setTimeout(resolve, ms));
            }
        }

        // 애플리케이션 초기화
        document.addEventListener('DOMContentLoaded', () => {
            new SlotGameAnalyzer();
        });
    </script>
</body>
</html>
    <script id="html_badge_script1">
        window.__genspark_remove_badge_link = "https://www.genspark.ai/api/html_badge/" +
            "remove_badge?token=To%2FBnjzloZ3UfQdcSaYfDgSHeT99KF1lwRa%2FqnOD3r70RXsmI9ehEyHg9YXfVQ%2F3w8kjMNrM3Byo%2FN6N71ayeXyLYDuMnvl68dt57B%2FQ8bcMrHWCWcJMqScdeIpJh96NjU%2BgyYyunZd60V6SSMCJVNUmYzzkwQnM9u0jx2M1nhD11PCMW2nkzSXABvMiaFRO9W3u2xl4CvvYsSu5DC17CVHwqVtdKBtMoTsVqRxwCd2qksOmyX867rYR3Gf16iVia59zQ0GVERXmzvvlDuT%2Fw1mPkPfaTJbAm0b4H3BC0Zo6nogBG79IxiSCTERyNtUdBnhNzcEBDqG4AGu1XFj1vzxmBc5W7lXmZ0ToCAZ8dEhe46BEfaoX0yqrejc6r5vJKWxlZLSk%2FC5VxikCw%2FEqhmKpYo%2Frp2BOmbqJQcRZOl7FE1hNN5%2BD55H2dZK0qhBpdkyd6MK2x9FOio6NAK3iFKaZDL6rE%2BvwlGy%2FNGvP8eJveBN%2Fs%2Frb2aGBnRS7kkOsb0Cx%2Bi2HUoY1Z87TBBy%2Baw%3D%3D";
        window.__genspark_locale = "ko-KR";
        window.__genspark_token = "To/BnjzloZ3UfQdcSaYfDgSHeT99KF1lwRa/qnOD3r70RXsmI9ehEyHg9YXfVQ/3w8kjMNrM3Byo/N6N71ayeXyLYDuMnvl68dt57B/Q8bcMrHWCWcJMqScdeIpJh96NjU+gyYyunZd60V6SSMCJVNUmYzzkwQnM9u0jx2M1nhD11PCMW2nkzSXABvMiaFRO9W3u2xl4CvvYsSu5DC17CVHwqVtdKBtMoTsVqRxwCd2qksOmyX867rYR3Gf16iVia59zQ0GVERXmzvvlDuT/w1mPkPfaTJbAm0b4H3BC0Zo6nogBG79IxiSCTERyNtUdBnhNzcEBDqG4AGu1XFj1vzxmBc5W7lXmZ0ToCAZ8dEhe46BEfaoX0yqrejc6r5vJKWxlZLSk/C5VxikCw/EqhmKpYo/rp2BOmbqJQcRZOl7FE1hNN5+D55H2dZK0qhBpdkyd6MK2x9FOio6NAK3iFKaZDL6rE+vwlGy/NGvP8eJveBN/s/rb2aGBnRS7kkOsb0Cx+i2HUoY1Z87TBBy+aw==";
    </script>
    
    <script id="html_notice_dialog_script" src="https://www.genspark.ai/notice_dialog.js"></script>
    
