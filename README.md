<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>우리 반 톡톡 (Talk Talk)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome Icons -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Nanum+Round:wght@400;700&display=swap');
        body { font-family: 'Nanum Round', sans-serif; background-color: #f1f5f9; -webkit-tap-highlight-color: transparent; }
        
        .message-appear { animation: fadeIn 0.3s ease-out forwards; }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(15px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        /* 스크롤바 디자인 */
        #chat-display::-webkit-scrollbar { width: 8px; }
        #chat-display::-webkit-scrollbar-track { background: #e2e8f0; border-radius: 10px; }
        #chat-display::-webkit-scrollbar-thumb { 
            background: #6366f1; 
            border-radius: 10px;
            border: 2px solid #e2e8f0;
        }
        #chat-display::-webkit-scrollbar-thumb:hover { background: #4338ca; }
        
        #chat-display { scroll-behavior: smooth; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        
        /* 입력창 고정 및 모바일 최적화 */
        footer { position: relative; padding-bottom: env(safe-area-inset-bottom); }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden text-slate-900">

    <!-- Header -->
    <header class="bg-indigo-600 text-white p-4 shadow-lg flex justify-between items-center z-30 shrink-0">
        <div class="flex items-center gap-3">
            <div class="bg-white p-2 rounded-xl text-indigo-600 shadow-md">
                <i class="fa-solid fa-comments text-xl"></i>
            </div>
            <div>
                <h1 class="text-lg font-bold leading-tight">우리 반 톡톡</h1>
                <p class="text-[10px] opacity-80 uppercase tracking-widest">Multi-Lang Classroom</p>
            </div>
        </div>
        <div class="hidden sm:flex bg-indigo-700 rounded-full px-4 py-1.5 text-[10px] font-bold gap-3 border border-indigo-400">
            <span>🇰🇷 KO</span>
            <span>🇷🇺 RU</span>
            <span>🇨🇳 ZH</span>
            <span>🇹🇭 TH</span>
        </div>
    </header>

    <!-- Role & Base Lang Settings -->
    <div class="bg-white border-b px-4 py-3 flex items-center justify-between shadow-sm z-20 shrink-0 overflow-x-auto no-scrollbar">
        <div class="flex items-center gap-3 shrink-0">
            <span class="text-slate-400 text-[11px] font-bold uppercase">Role:</span>
            <div class="flex bg-slate-100 p-1 rounded-full border border-slate-200">
                <button id="role-teacher" onclick="setRole('teacher')" class="px-5 py-1.5 rounded-full text-xs font-bold transition-all bg-indigo-600 text-white shadow-sm">
                    선생님
                </button>
                <button id="role-student" onclick="setRole('student')" class="px-5 py-1.5 rounded-full text-xs font-bold transition-all text-slate-500 hover:text-indigo-500">
                    학생
                </button>
            </div>
        </div>
        <div id="student-lang-area" class="flex items-center gap-2 border-l pl-4 opacity-30 pointer-events-none transition-all">
            <span class="text-slate-400 text-[11px] font-bold uppercase">Lang:</span>
            <select id="student-base-lang" class="bg-slate-100 border-none rounded-xl px-2 py-1.5 text-xs font-bold outline-none text-slate-700" onchange="updateRoleSelection()">
                <option value="ru">🇷🇺 RU</option>
                <option value="zh">🇨🇳 ZH</option>
                <option value="th" selected>🇹🇭 TH</option>
            </select>
        </div>
    </div>

    <!-- Main Chat Window -->
    <main id="chat-display" class="flex-1 overflow-y-auto p-4 space-y-10 relative bg-[#f8fafc]">
        <div class="absolute inset-0 opacity-[0.03] pointer-events-none bg-[url('https://www.transparenttextures.com/patterns/cubes.png')]"></div>
        
        <div class="flex flex-col items-start message-appear relative z-10">
            <div class="max-w-[95%] rounded-3xl p-5 shadow-sm bg-white border border-indigo-50 rounded-tl-none">
                <div class="flex items-center gap-2 mb-2">
                    <span class="text-[9px] font-black text-white bg-indigo-500 px-1.5 py-0.5 rounded tracking-tighter uppercase">KOR</span>
                </div>
                <p class="font-bold text-lg text-indigo-900 leading-snug">
                    안녕하세요! 선생님과 친구들이 대화할 수 있는 공간입니다.<br>
                    <span class="text-sm font-normal text-slate-500 italic">모든 친구들의 언어로 동시 번역됩니다.</span>
                </p>
            </div>
        </div>
    </main>

    <!-- Loading / Error -->
    <div id="loading" class="hidden px-6 py-2 bg-indigo-50 text-indigo-600 text-[11px] font-bold italic border-t border-indigo-100">
        <i class="fa-solid fa-spinner fa-spin mr-2"></i> 번역 중...
    </div>

    <div id="error-banner" class="hidden m-4 p-4 bg-rose-50 border-2 border-rose-100 rounded-2xl flex items-center justify-between text-rose-700 shadow-xl z-50 animate-bounce">
        <div class="flex items-center gap-3 text-sm">
            <i class="fa-solid fa-circle-exclamation text-xl"></i>
            <span id="error-text">오류가 발생했습니다.</span>
        </div>
        <button onclick="hideError()" class="text-rose-400 p-1"><i class="fa-solid fa-xmark text-lg"></i></button>
    </div>

    <!-- Input Footer -->
    <footer class="bg-white p-4 border-t border-slate-200 z-30 shadow-[0_-8px_30px_-10px_rgba(0,0,0,0.1)] shrink-0">
        <div class="max-w-4xl mx-auto space-y-4">
            <div class="flex bg-slate-100 p-1 rounded-2xl overflow-x-auto no-scrollbar shadow-inner gap-1">
                <button id="lang-ko" onclick="setInputLang('ko')" class="flex-1 min-w-[80px] py-2.5 rounded-xl text-[11px] font-black transition-all bg-white shadow-sm text-indigo-600">🇰🇷 KOR</button>
                <button id="lang-ru" onclick="setInputLang('ru')" class="flex-1 min-w-[80px] py-2.5 rounded-xl text-[11px] font-black transition-all text-slate-400">🇷🇺 RUS</button>
                <button id="lang-zh" onclick="setInputLang('zh')" class="flex-1 min-w-[80px] py-2.5 rounded-xl text-[11px] font-black transition-all text-slate-400">🇨🇳 CHI</button>
                <button id="lang-th" onclick="setInputLang('th')" class="flex-1 min-w-[80px] py-2.5 rounded-xl text-[11px] font-black transition-all text-slate-400">🇹🇭 THA</button>
            </div>

            <div class="flex gap-2">
                <input id="text-input" type="text" autocomplete="off" placeholder="메시지를 적어주세요..." 
                       class="flex-1 border-2 border-slate-100 rounded-2xl px-5 py-4 focus:outline-none focus:border-indigo-500 transition-all text-base font-medium shadow-sm bg-slate-50">
                <button onclick="handleSend()" id="send-btn" 
                        class="bg-indigo-600 text-white px-6 rounded-2xl shadow-lg transition-all active:scale-90 flex items-center justify-center hover:bg-indigo-700">
                    <i class="fa-solid fa-paper-plane text-xl"></i>
                </button>
            </div>
        </div>
    </footer>

    <script>
        const API_KEY = "AIzaSyAamHgTQJ6GUB7-HlUp7jQgpNPr8zib8Dk";
        let currentRole = 'teacher';
        let currentInputLang = 'ko';
        let voices = [];

        function loadVoices() {
            voices = window.speechSynthesis.getVoices();
        }
        if (window.speechSynthesis.onvoiceschanged !== undefined) {
            window.speechSynthesis.onvoiceschanged = loadVoices;
        }
        loadVoices();

        function setRole(role) {
            currentRole = role;
            document.getElementById('role-teacher').className = role === 'teacher' ? 'px-5 py-1.5 rounded-full text-xs font-bold transition-all bg-indigo-600 text-white shadow-sm' : 'px-5 py-1.5 rounded-full text-xs font-bold transition-all text-slate-500';
            document.getElementById('role-student').className = role === 'student' ? 'px-5 py-1.5 rounded-full text-xs font-bold transition-all bg-amber-500 text-white shadow-sm' : 'px-5 py-1.5 rounded-full text-xs font-bold transition-all text-slate-500';
            
            const area = document.getElementById('student-lang-area');
            if(role === 'student') {
                area.classList.remove('opacity-30', 'pointer-events-none');
                setInputLang(document.getElementById('student-base-lang').value);
            } else {
                area.classList.add('opacity-30', 'pointer-events-none');
                setInputLang('ko');
            }
        }

        function updateRoleSelection() {
            if(currentRole === 'student') {
                setInputLang(document.getElementById('student-base-lang').value);
            }
        }

        function setInputLang(lang) {
            currentInputLang = lang;
            ['ko', 'ru', 'zh', 'th'].forEach(l => {
                const btn = document.getElementById(`lang-${l}`);
                if(l === lang) {
                    btn.className = 'flex-1 min-w-[80px] py-2.5 rounded-xl text-[11px] font-black transition-all bg-white shadow-sm text-indigo-600 ring-1 ring-indigo-50';
                } else {
                    btn.className = 'flex-1 min-w-[80px] py-2.5 rounded-xl text-[11px] font-black transition-all text-slate-400';
                }
            });
        }

        async function translate(text, source, target) {
            const url = `https://translation.googleapis.com/language/translate/v2?key=${API_KEY}`;
            const res = await fetch(url, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ q: text, source: source, target: target, format: 'text' })
            });
            const data = await res.json();
            if (!res.ok) throw new Error(data.error?.message || "Error");
            return data.data.translations[0].translatedText;
        }

        function speak(text, lang) {
            if (!window.speechSynthesis) return;
            window.speechSynthesis.cancel();
            const uttr = new SpeechSynthesisUtterance(text);
            const map = { 'ko': 'ko-KR', 'ru': 'ru-RU', 'zh': 'zh-CN', 'th': 'th-TH' };
            uttr.lang = map[lang] || lang;
            if (voices.length === 0) voices = window.speechSynthesis.getVoices();
            const voice = voices.find(v => v.lang === uttr.lang) || voices.find(v => v.lang.startsWith(lang));
            if (voice) uttr.voice = voice;
            uttr.rate = 0.9;
            window.speechSynthesis.speak(uttr);
        }

        function showError(msg) {
            document.getElementById('error-text').innerText = msg;
            document.getElementById('error-banner').classList.remove('hidden');
        }

        function hideError() {
            document.getElementById('error-banner').classList.add('hidden');
        }

        async function handleSend() {
            const input = document.getElementById('text-input');
            const text = input.value.trim();
            if (!text || document.getElementById('loading').classList.contains('hidden') === false) return;

            input.value = '';
            document.getElementById('loading').classList.remove('hidden');
            hideError();

            try {
                const langs = ['ko', 'ru', 'zh', 'th'];
                const targets = langs.filter(l => l !== currentInputLang);
                const results = await Promise.all(targets.map(l => translate(text, currentInputLang, l)));
                const transObj = {};
                targets.forEach((l, i) => transObj[l] = results[i]);
                displayMessage(text, currentInputLang, transObj);
            } catch (e) {
                console.error(e);
                showError("번역 오류: 설정을 확인하세요.");
            } finally {
                document.getElementById('loading').classList.add('hidden');
            }
        }

        function displayMessage(orig, sLang, trans) {
            const chat = document.getElementById('chat-display');
            const wrap = document.createElement('div');
            wrap.className = `flex flex-col ${currentRole === 'teacher' ? 'items-start' : 'items-end'} message-appear`;

            let rows = '';
            const labels = { 'ko': '🇰🇷 KOR', 'ru': '🇷🇺 RUS', 'zh': '🇨🇳 CHI', 'th': '🇹🇭 THA' };

            for (const [lang, text] of Object.entries(trans)) {
                const safe = text.replace(/'/g, "\\'").replace(/"/g, '&quot;');
                rows += `
                    <div class="flex items-start gap-3 p-3.5 bg-slate-50 rounded-2xl border border-slate-100 shadow-sm">
                        <div class="flex-1">
                            <span class="text-[9px] font-black text-slate-400 uppercase tracking-tighter">${labels[lang]}</span>
                            <p class="text-base font-medium text-slate-800 leading-snug">${text}</p>
                        </div>
                        <button onclick="speak('${safe}', '${lang}')" class="p-2 text-slate-300 hover:text-indigo-600 transition-colors">
                            <i class="fa-solid fa-volume-high"></i>
                        </button>
                    </div>
                `;
            }

            const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            const sSafe = orig.replace(/'/g, "\\'").replace(/"/g, '&quot;');

            wrap.innerHTML = `
                <div class="max-w-[95%] sm:max-w-[85%] rounded-[2rem] p-5 shadow-md relative bg-white border ${currentRole === 'teacher' ? 'border-indigo-100 rounded-tl-none' : 'border-indigo-500 rounded-tr-none'}">
                    <div class="flex justify-between items-start gap-4 mb-4">
                        <div class="flex-1">
                            <div class="flex items-center gap-2 mb-1">
                                <span class="text-[9px] font-black text-indigo-600 bg-indigo-50 px-2 py-0.5 rounded tracking-tighter uppercase">${labels[sLang].split(' ')[1]}</span>
                                <span class="text-[9px] text-slate-400 font-bold">${currentRole === 'teacher' ? '선생님' : '학생'}</span>
                            </div>
                            <p class="font-bold text-xl leading-tight text-slate-900">${orig}</p>
                        </div>
                        <button onclick="speak('${sSafe}', '${sLang}')" class="p-2.5 bg-indigo-50 text-indigo-600 rounded-full shadow-sm">
                            <i class="fa-solid fa-volume-high"></i>
                        </button>
                    </div>
                    <div class="space-y-3 pt-3 border-t border-slate-100">${rows}</div>
                    <span class="text-[9px] font-bold absolute -bottom-5 ${currentRole === 'teacher' ? 'left-4 text-slate-400' : 'right-4 text-indigo-400'}">${time}</span>
                </div>
            `;

            chat.appendChild(wrap);
            chat.scrollTop = chat.scrollHeight;
        }

        document.getElementById('text-input').addEventListener('keypress', (e) => { if (e.key === 'Enter') handleSend(); });
        setRole('teacher');
    </script>
</body>
</html>
