<!DOCTYPE html>
<html lang="ko" class="">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CATCH POINT 생기부 매니저</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        slate: {
                            905: '#0f172a',
                            955: '#020617',
                        }
                    }
                }
            }
        }
    </script>
    <!-- React & ReactDOM CDN -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    <!-- Babel CDN (For JSX compilation in browser) -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <style>
        /* 기본 스크롤바 커스텀 */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 10px;
        }
        .dark ::-webkit-scrollbar-thumb {
            background: #475569;
        }
    </style>
</head>
<body class="bg-slate-50 dark:bg-slate-955 text-slate-800 dark:text-slate-100 transition-colors duration-300 min-h-screen">
    <div id="root"></div>

    <!-- Firebase SDK v11 (ES Modules) -->
    <script type="module" babel-target="true">
        import { initializeApp, getApps } from 'https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js';
        import { 
            getAuth, 
            createUserWithEmailAndPassword, 
            signInWithEmailAndPassword, 
            onAuthStateChanged, 
            signOut,
            signInWithCustomToken,
            signInAnonymously,
            setPersistence,
            browserLocalPersistence,
            browserSessionPersistence
        } from 'https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js';
        import { 
            getFirestore, 
            doc, 
            setDoc, 
            onSnapshot,
            collection,
            query,
            getDocs,
            addDoc,
            deleteDoc
        } from 'https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js';

        // 글로벌 변수로 Firebase 모듈 바인딩 (Babel Script와 연동 목적)
        window.FirebaseSDK = {
            initializeApp, getApps, getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword,
            onAuthStateChanged, signOut, signInWithCustomToken, signInAnonymously, setPersistence,
            browserLocalPersistence, browserSessionPersistence, getFirestore, doc, setDoc, onSnapshot,
            collection, query, getDocs, addDoc, deleteDoc
        };
    </script>

    <!-- React App JSX Script -->
    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        // Firebase 설정 기입
        const firebaseConfig = {
            apiKey: "AIzaSyCNncvAHxoiL4HZE4ephrST0THGPmbLfnc",
            authDomain: "app123-9a3c9.firebaseapp.com",
            projectId: "app123-9a3c9",
            storageBucket: "app123-9a3c9.firebasestorage.app",
            messagingSenderId: "933256403505",
            appId: "1:933256403505:web:a2ad048181197cc6139310",
            measurementId: "G-LJ6GL7YWVX"
        };

        let app, auth, db;
        const appId = 'student-record-app';

        // Firebase 비동기 초기화 보장 헬퍼
        const initFirebase = () => {
            if (!window.FirebaseSDK) return false;
            const { initializeApp, getApps, getAuth, getFirestore } = window.FirebaseSDK;
            app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0];
            auth = getAuth(app);
            db = getFirestore(app);
            return true;
        };

        // ==========================================================
        // [수동 인라인 SVG 아이콘 컴포넌트 - lucide 의존성 완화]
        // ==========================================================
        const SvgIcon = ({ name, className = "w-5 h-5" }) => {
            const icons = {
                'file-text': <path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7Z"/><path d="M14 2v4a2 2 0 0 0 2 2h4"/><path d="M10 9H8"/><path d="M16 13H8"/><path d="M16 17H8"/>,
                'award': <circle cx="12" cy="8" r="6"/><path d="M15.477 12.89 17 22l-5-3-5 3 1.523-9.11"/>,
                'activity': <path d="M22 12h-4l-3 9L9 3l-3 9H2"/>,
                'book-marked': <path d="M4 19.5v-15A2.5 2.5 0 0 1 6.5 2H20v20H6.5a2.5 2.5 0 0 1-2.5-2.5Z"/><path d="M6 6h10M6 10h10"/>,
                'graduation-cap': <path d="M21.42 10.922a1 1 0 0 0-.019-1.838L12.83 5.18a2 2 0 0 0-1.66 0L2.6 9.08a1 1 0 0 0 0 1.832l8.57 3.908a2 2 0 0 0 1.66 0z"/><path d="M6 12v5c0 2 2 3 6 3s6-1 6-3v-5"/>,
                'search': <circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/>,
                'x': <path d="M18 6 6 18M6 6l12 12"/>,
                'check': <path d="M20 6 9 17l-5-5"/>,
                'brain-circuit': <path d="M12 2v3M19 9h3M3 9h3M12 19v3M19 15h3M3 15h3M6.5 6.5l2 2M15.5 15.5l2 2M15.5 6.5l2-2M6.5 15.5l2-2"/>,
                'refresh-cw': <path d="M21 12a9 9 0 0 0-9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/><path d="M3 3v5h5M3 12a9 9 0 0 0 9 9 9.75 9.75 0 0 0 6.74-2.74L21 16"/><path d="M16 16h5v5"/>,
                'sparkles': <path d="m12 3-1.912 5.813a2 2 0 0 1-1.275 1.275L3 12l5.813 1.912a2 2 0 0 1 1.275 1.275L12 21l1.912-5.813a2 2 0 0 1 1.275-1.275L21 12l-5.813-1.912a2 2 0 0 1-1.275-1.275Z"/>,
                'settings': <path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.08a2 2 0 0 1-1-1.74v-.5a2 2 0 0 1 1-1.74l.15-.1a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"/><circle cx="12" cy="12" r="3"/>,
                'moon': <path d="M12 3a6 6 0 0 0 9 9 9 9 0 1 1-9-9Z"/>,
                'sun': <circle cx="12" cy="12" r="4"/><path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M6.34 17.66l-1.41 1.41M19.07 4.93l-1.41 1.41"/>,
                'alert-circle': <circle cx="12" cy="12" r="10"/><line x1="12" x2="12" y1="8" y2="12"/><line x1="12" x2="12.01" y1="16" y2="16"/>,
                'edit-3': <path d="M12 20h9M16.5 3.5a2.12 2.12 0 0 1 3 3L7 19l-4 1 1-4Z"/>,
                'users': <path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M22 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/>,
                'hash': <line x1="4" x2="20" y1="9" y2="9"/><line x1="4" x2="20" y1="15" y2="15"/><line x1="10" x2="8" y1="3" y2="21"/><line x1="16" x2="14" y1="3" y2="21"/>,
                'lock-keyhole': <circle cx="12" cy="16" r="1"/><rect x="3" y="10" width="18" height="12" rx="2"/><path d="M7 10V7a5 5 0 0 1 10 0v3"/>,
                'door-open': <path d="M13 4h6a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2h-6M2 20h9M3 20V6a2 2 0 0 1 2-2h4v16"/>,
                'message-square': <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>,
                'eye': <path d="M2 12s3-7 10-7 10 7 10 7-3 7-10 7-10-7-10-7Z"/><circle cx="12" cy="12" r="3"/>,
                'eye-off': <path d="M9.88 9.88a3 3 0 1 0 4.24 4.24M10.73 5.08A10.43 10.43 0 0 1 12 5c7 0 10 7 10 7a13.16 13.16 0 0 1-1.67 2.68M6.61 6.61A13.52 13.52 0 0 0 2 12s3 7 10 7a9.74 9.74 0 0 0 5.39-1.61M2 2l20 20"/>,
                'spell-check': <path d="m6 16 6-12 6 12M8 12h8M16 20l2 2 4-4"/>,
                'logout': <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4M16 17l5-5-5-5M21 12H9"/>,
                'save': <path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"/><polyline points="17 21 17 13 7 13 7 21"/><polyline points="7 3 7 8 15 8"/>,
                'plus': <line x1="12" x2="12" y1="5" y2="19"/><line x1="5" x2="19" y1="12" y2="12"/>,
                'trash-2': <polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/><line x1="10" x2="10" y1="11" y2="17"/><line x1="14" x2="14" y1="11" y2="17"/>,
                'send': <line x1="22" x2="11" y1="2" y2="13"/><polygon points="22 2 15 22 11 13 2 9 22 2"/>,
                'rotate-ccw': <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/><polyline points="3 3 3 8 8 8"/>
            };

            return (
                <svg 
                    xmlns="http://www.w3.org/2000/svg" 
                    viewBox="0 0 24 24" 
                    fill="none" 
                    stroke="currentColor" 
                    strokeWidth="2" 
                    strokeLinecap="round" 
                    strokeLinejoin="round" 
                    className={className}
                >
                    {icons[name] || null}
                </svg>
            );
        };

        const FIXED_STRUCTURE = {
            autonomous: { 
                label: "자율활동", 
                limit: 500, 
                placeholder: "학급 회의, 행사 기획, 자치 활동 등 본인의 주도적 자율활동 내용을 입력하세요.",
                tip: "역할에 따른 주도성, 공동체 기여도, 위기 극복 능력이 드러나면 좋습니다."
            },
            club: { 
                label: "동아리활동", 
                limit: 500, 
                placeholder: "동아리 내에서의 구체적 역할, 탐구 주제, 실험 내용 등을 입력하세요.",
                tip: "수업 중 품은 의문을 탐구해 나간 심화 학습 과정을 중심으로 기록해보세요."
            },
            career: { 
                label: "진로활동", 
                limit: 700, 
                placeholder: "자신의 진로 방향성 탐색, 관련 전공 독서 및 연구 분석 활동 내용을 입력하세요.",
                tip: "단순히 '강연을 들었다'가 아닌, 강연 후 찾아본 심화 독서나 탐구 보고서가 핵심입니다."
            },
            service: { 
                label: "봉사활동 (특기사항)", 
                limit: 500, 
                placeholder: "교내외 봉사활동 중 진정성 있는 참여 태도와 배운 점을 기록하세요.",
                tip: "지속적이고 헌신적으로 참여한 활동 및 그를 통한 내적 성장을 기술하세요."
            },
            comprehensive: { 
                label: "행동특성 및 종합의견", 
                limit: 1000, 
                placeholder: "한 해를 마무리하며 교사가 작성할 추천 의견의 초안을 작성하세요.",
                tip: "담임 교사가 작성하는 부분인 만큼, 객관적이고 긍정적인 에피소드 위주로 초안을 정리합니다."
            }
        };

        const DEFAULT_GRADE_SUBJECTS = {
            '1': [
                { name: "공통국어", limit: 500, content: "" },
                { name: "공통수학", limit: 500, content: "" },
                { name: "공통영어", limit: 500, content: "" },
                { name: "통합과학", limit: 500, content: "" },
                { name: "통합사회", limit: 500, content: "" },
                { name: "한국사", limit: 500, content: "" },
                { name: "정보", limit: 500, content: "" },
                { name: "과학탐구실험", limit: 500, content: "" },
                { name: "체육", limit: 500, content: "" },
                { name: "음악", limit: 500, content: "" },
                { name: "미술", limit: 500, content: "" },
                { name: "진로와직업", limit: 500, content: "" }
            ],
            '2': [
                { name: "문학", limit: 500, content: "" },
                { name: "독서와 작문", limit: 500, content: "" },
                { name: "대수", limit: 500, content: "" },
                { name: "미적분Ⅰ", limit: 500, content: "" },
                { name: "영어Ⅰ", limit: 500, content: "" },
                { name: "영어Ⅱ", limit: 500, content: "" },
                { name: "스포츠생활", limit: 500, content: "" }
            ],
            '3': [
                { name: "화법과 언어", limit: 500, content: "" },
                { name: "확률과 통계", limit: 500, content: "" },
                { name: "영어 독해와 작문", limit: 500, content: "" },
                { name: "스포츠 과학", limit: 500, content: "" },
                { name: "스포츠 문화", limit: 500, content: "" },
                { name: "음악 감상과 비평", limit: 500, content: "" },
                { name: "미술 창작", limit: 500, content: "" },
                { name: "생태와 환경", limit: 500, content: "" }
            ]
        };

        const SELECTABLE_OPTIONAL_SUBJECTS = {
            '1': [],
            '2': [
                "인공지능 수학", "세계사", "세계시민과 지리", "사회와 문화", "현대사회와 윤리", 
                "물리학", "화학", "생명과학", "지구과학", "인공지능 기초", 
                "일본어", "중국어", "교육의 이해", "보건", "문학과 영상", 
                "기하", "영어 발표와 토론", "동아시아 역사 기행", "한국지리 탐구", 
                "법과 사회", "경제", "윤리와 사상", "역학과 에너지", 
                "물질과 에너지", "세포와 물질대사", "지구시스템과학", "일본 문화", 
                "한문", "인간과 철학"
            ],
            '3': [
                "주제 탐구 독서", "미적분2", "영미 문학 읽기", "도시의 미래탐구", 
                "정치", "인문학과 윤리", "역사로 탐구하는 현대 세계", "전자기와 양자", 
                "화학 반응의 세계", "생물의 유전", "행성우주과학", "데이터 과학", 
                "일본어", "중국어", "인간과 심리", "언어생활 탐구", 
                "실용 통계", "수학과제 탐구", "심화 영어", "미디어 영어", 
                "기후변화와 지속가능한 세계", "윤리문제 탐구", "기후변화와 환경생태", 
                "융합과학 탐구", "인간과 경제활동"
            ]
        };

        const EXCLUDED_LIBERAL_ARTS = [
            '보건', '교육', '철학', '심리', '종교', '진로', '직업', '논리학', '논술', '환경', '실용', '연극', '영화', '미디어', '정보보호',
            '체육', '음악', '미술', '스포츠', '창작', '비평', '생태', '세계시민과 지리', '일본 문화', '한문'
        ];

        const NEIS_FORBIDDEN_CHARS_REGEX = /[★☆○●▣■◆◇▲▼▶◀♥♡♣♤※☎☏✉♨✈✔✖➕➖➗❓❗]/g;

        const isLiberalArtSubject = (name) => {
            if (!name) return false;
            return EXCLUDED_LIBERAL_ARTS.some(keyword => name.includes(keyword));
        };

        const getErrorMessage = (code) => {
            switch (code) {
                case 'auth/user-not-found':
                case 'auth/wrong-password':
                case 'auth/invalid-credential':
                    return '이메일 혹은 비밀번호가 일치하지 않습니다.';
                case 'auth/email-already-in-use':
                    return '이미 가입된 이메일 주소입니다.';
                case 'auth/weak-password':
                    return '비밀번호는 최소 6자 이상으로 설정해야 합니다.';
                case 'auth/invalid-email':
                    return '유효하지 않은 이메일 형식입니다.';
                default:
                    return '인증 처리 도중 예기치 못한 오류가 발생했습니다.';
            }
        };

        const callGeminiFeedbackAPI = async (userInput, systemPrompt) => {
            const apiKey = ""; 
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            
            let delay = 1000;
            for (let attempt = 1; attempt <= 5; attempt++) {
                try {
                    const response = await fetch(url, {
                        method: "POST",
                        headers: { "Content-Type": "application/json" },
                        body: JSON.stringify({
                            contents: [{ parts: [{ text: userInput }] }],
                            systemInstruction: { parts: [{ text: systemPrompt }] }
                        })
                    });
                    if (!response.ok) throw new Error(`API 응답 실패 (코드: ${response.status})`);
                    const data = await response.json();
                    return data?.candidates?.[0]?.content?.parts?.[0]?.text || "결과를 받아오지 못했습니다.";
                } catch (err) {
                    if (attempt === 5) throw err;
                    await new Promise(resolve => setTimeout(resolve, delay));
                    delay *= 2;
                }
            }
        };

        function App() {
            const [sdkReady, setSdkReady] = useState(false);
            const [user, setUser] = useState(null);
            const [loading, setLoading] = useState(true);
            const [isTransitioning, setIsTransitioning] = useState(false);
            const [view, setView] = useState('login'); 
            const [activeTab, setActiveTab] = useState('record'); 
            
            const [email, setEmail] = useState('');
            const [password, setPassword] = useState('');
            const [message, setMessage] = useState({ type: '', text: '' });

            const [customId, setCustomId] = useState('');
            const [shareGrades, setShareGrades] = useState(false);
            const [joinedRoomId, setJoinedRoomId] = useState(''); 

            const [rememberMe, setRememberMe] = useState(() => {
                if (typeof window !== 'undefined') {
                    const savedPref = localStorage.getItem('remember_me');
                    return savedPref !== 'false';
                }
                return true;
            });

            const [grade, setGrade] = useState('1'); 
            const [saveStatus, setSaveStatus] = useState(''); 
            const [isAddingSubject, setIsAddingSubject] = useState(false);
            const [newSubjectName, setNewSubjectName] = useState('');
            const [byteCalcMode, setByteCalcMode] = useState('char'); 

            const [selectedDept, setSelectedDept] = useState('컴퓨터공학과');

            const [aiFeedbackText, setAiFeedbackText] = useState('');
            const [isGeneratingFeedback, setIsGeneratingFeedback] = useState(false);

            const [checkText, setCheckText] = useState(''); 
            const [spellCheckResult, setSpellCheckResult] = useState(''); 
            const [isCheckingSpell, setIsCheckingSpell] = useState(false); 
            const [detectedForbiddenChars, setDetectedForbiddenChars] = useState([]); 

            const [isDarkMode, setIsDarkMode] = useState(() => {
                if (typeof window !== 'undefined') {
                    const savedTheme = localStorage.getItem('theme');
                    return savedTheme === 'dark';
                }
                return false;
            });
            const [isSettingsOpen, setIsSettingsOpen] = useState(false);
            const [modal, setModal] = useState({ show: false, title: '', message: '', onConfirm: null });

            const [allGradesData, setAllGradesData] = useState({
                '1': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' }, 
                '2': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' },
                '3': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' }
            });

            const [groupCategory, setGroupCategory] = useState(''); 
            const [groupName, setGroupName] = useState(''); 
            const [groupPassword, setGroupPassword] = useState(''); 
            const [searchCategory, setSearchCategory] = useState(''); 
            const [matchingRooms, setMatchingRooms] = useState([]); 
            const [activeRoom, setActiveRoom] = useState(null); 
            const [roomUsersData, setRoomUsersData] = useState([]); 
            const [chatMessages, setChatMessages] = useState([]); 
            const [inputChat, setInputChat] = useState(''); 
            const [roomSearchStatus, setRoomSearchStatus] = useState(''); 

            const [passwordModal, setPasswordModal] = useState({ show: false, targetRoom: null, typedPassword: '' });

            const canvasRef = useRef(null);
            const chatEndRef = useRef(null);

            // SDK 대기 폴링 및 테마 세팅
            useEffect(() => {
                const checkSDK = setInterval(() => {
                    if (window.FirebaseSDK) {
                        const ready = initFirebase();
                        if (ready) {
                            setSdkReady(true);
                            clearInterval(checkSDK);
                        }
                    }
                }, 100);
                return () => clearInterval(checkSDK);
            }, []);

            useEffect(() => {
                if (isDarkMode) {
                    document.documentElement.classList.add('dark');
                    localStorage.setItem('theme', 'dark');
                } else {
                    document.documentElement.classList.remove('dark');
                    localStorage.setItem('theme', 'light');
                }
            }, [isDarkMode]);

            const processEmailInput = (emailStr) => {
                const trimmed = emailStr.trim();
                if (trimmed.toLowerCase() === 'admin') {
                    return 'admin@admin.com';
                }
                return trimmed;
            };

            const isValidEmail = (emailStr) => {
                const processed = processEmailInput(emailStr);
                return /\S+@\S+\.\S+/.test(processed);
            };

            const getByteLength = (str) => {
                if (!str) return 0;
                let byteLength = 0;
                for (let i = 0; i < str.length; i++) {
                    const code = str.charCodeAt(i);
                    if (code === 10) { 
                        byteLength += 2; 
                    } else if (code > 127) {
                        byteLength += 3; 
                    } else {
                        byteLength += 1; 
                    }
                }
                return byteLength;
            };

            // Firebase 인증 초기화 및 상태 구독
            useEffect(() => {
                if (!sdkReady) return;
                const { onAuthStateChanged, signInWithCustomToken, signInAnonymously } = window.FirebaseSDK;

                const initAuth = async () => {
                    try {
                        if (typeof window !== 'undefined') {
                            const savedEmail = localStorage.getItem('saved_email');
                            if (savedEmail) {
                                setEmail(savedEmail);
                            }
                        }

                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            await signInWithCustomToken(auth, __initial_auth_token);
                        } else {
                            await signInAnonymously(auth);
                        }
                    } catch (err) {
                        console.warn("인증 처리 대체 시도:", err);
                        try {
                            await signInAnonymously(auth);
                        } catch (anonErr) {
                            console.error("익명 대체 로그인 실패", anonErr);
                        }
                    } finally {
                        setLoading(false);
                    }
                };
                initAuth();

                const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
                    setUser(currentUser);
                    setLoading(false);
                    if (currentUser) {
                        setActiveTab('record');
                        setView('main');
                        setMessage({ type: '', text: '' });
                    } else {
                        setView('login');
                        setActiveRoom(null);
                        setJoinedRoomId('');
                        setChatMessages([]);
                        setRoomUsersData([]);
                    }
                });
                return () => unsubscribe();
            }, [sdkReady]);

            useEffect(() => {
                if (user) {
                    setIsTransitioning(true);
                    const timer = setTimeout(() => {
                        setIsTransitioning(false);
                    }, 2000);
                    return () => clearTimeout(timer);
                }
            }, [user]);

            // 내신 계산 알고리즘
            const calculateGradeFromRank = (rank, total, jointCount = 1) => {
                const r = Number(rank);
                const t = Number(total);
                const j = Number(jointCount) || 1;

                if (!r || !t || r <= 0 || t <= 0 || r > t) {
                    return { grade9: null, grade5: null, percentage: null };
                }

                const midRank = r + (j - 1) / 2;
                const ratio = midRank / t;
                const percentage = ratio * 100;

                let grade9 = 9;
                if (ratio <= 0.04) grade9 = 1;
                else if (ratio <= 0.11) grade9 = 2;
                else if (ratio <= 0.23) grade9 = 3;
                else if (ratio <= 0.40) grade9 = 4;
                else if (ratio <= 0.60) grade9 = 5;
                else if (ratio <= 0.77) grade9 = 6;
                else if (ratio <= 0.89) grade9 = 7;
                else if (ratio <= 0.96) grade9 = 8;

                let grade5 = 5;
                if (ratio <= 0.10) grade5 = 1;
                else if (ratio <= 0.34) grade5 = 2;
                else if (ratio <= 0.66) grade5 = 3;
                else if (ratio <= 0.90) grade5 = 4;

                return { grade9, grade5, percentage };
            };

            const analyzeTrend = (gradePoints) => {
                const validPoints = gradePoints.filter(p => p.val !== null);
                if (validPoints.length < 2) {
                    return { text: "분석을 위한 성적 데이터가 부족합니다.", icon: "neutral" };
                }

                let first = validPoints[0];
                let last = validPoints[validPoints.length - 1];
                const diff = first.val - last.val; 

                if (diff > 0.15) {
                    return { 
                        text: "학년이 갈수록 등급이 낮아지는(성적이 상승하는) 대표적인 '지속 우상향(학업 성취도 개선형)' 흐름입니다. 대입 학생부 종합전형에서 학업 주도성과 발전 가능성 부문에서 매우 긍정적인 정성 평가를 받을 확률이 높습니다.", 
                        icon: "up" 
                    };
                } else if (diff < -0.15) {
                    return { 
                        text: "학년이 올라갈수록 등급 수치가 올라가는 '우하향(학업 관리 보완 필요형)' 흐름이 감지되었습니다. 3학년 1학기 최종 내신 및 주력 선택과목들의 성적 방어가 필요하며, 학생부 기재 내용(세특)을 통해 학업적 성실성을 추가 보강하는 보완 전략이 요구됩니다.", 
                        icon: "down" 
                    };
                } else {
                    return { 
                        text: "일정한 내신 등급 구간을 유지하는 '성적 안정형(지속 유지형)' 흐름입니다. 급격한 편차 없이 성실함을 꾸준히 증명하고 있으므로, 전공 핵심 권장 과목의 이수 여부와 성취도 수준이 입시 성공의 핵심 열쇠가 될 것입니다.", 
                        icon: "stable" 
                    };
                }
            };

            const getGradesTabSubjects = () => {
                const activeSubjects = allGradesData[grade]?.subjects || [];
                if (grade !== '1') return activeSubjects;

                const splitSubjects = [];
                activeSubjects.forEach(sub => {
                    const targetNamesToSplit = ["공통국어", "공통수학", "공통영어", "통합사회", "통합과학"];
                    if (targetNamesToSplit.includes(sub.name)) {
                        splitSubjects.push({
                            id: `${sub.id}_1`,
                            name: `${sub.name}1`,
                            limit: sub.limit,
                            isSplit: true
                        });
                        splitSubjects.push({
                            id: `${sub.id}_2`,
                            name: `${sub.name}2`,
                            limit: sub.limit,
                            isSplit: true
                        });
                    } else {
                        splitSubjects.push(sub);
                    }
                });
                return splitSubjects;
            };

            const calculateGPAByGrade = (targetGrade) => {
                const currentGrades = allGradesData[targetGrade]?.grades || {};
                const activeSubjects = allGradesData[targetGrade]?.subjects || [];

                const processingSubjects = [];
                if (targetGrade === '1') {
                    activeSubjects.forEach(sub => {
                        const targetNamesToSplit = ["공통국어", "공통수학", "공통영어", "통합사회", "통합과학"];
                        if (targetNamesToSplit.includes(sub.name)) {
                            processingSubjects.push({ id: `${sub.id}_1`, name: `${sub.name}1` });
                            processingSubjects.push({ id: `${sub.id}_2`, name: `${sub.name}2` });
                        } else {
                            processingSubjects.push(sub);
                        }
                    });
                } else {
                    processingSubjects.push(...activeSubjects);
                }

                let totalUnits9 = 0;
                let sumGrades9 = 0;
                let totalUnits5 = 0;
                let sumGrades5 = 0;

                processingSubjects.forEach(sub => {
                    if (!isLiberalArtSubject(sub.name)) {
                        const subGrade = currentGrades[sub.id] || { units: 0, rank: 0, total: 0, joint: 1 };
                        const units = subGrade.units || 0;
                        const { grade9, grade5 } = calculateGradeFromRank(subGrade.rank, subGrade.total, subGrade.joint);

                        if (units > 0) {
                            if (grade9 !== null) {
                                totalUnits9 += units;
                                sumGrades9 += (grade9 * units);
                            }
                            if (grade5 !== null) {
                                totalUnits5 += units;
                                sumGrades5 += (grade5 * units);
                            }
                        }
                    }
                });

                const average9 = totalUnits9 > 0 ? Number((sumGrades9 / totalUnits9).toFixed(2)) : null;
                const average5 = totalUnits5 > 0 ? Number((sumGrades5 / totalUnits5).toFixed(2)) : null;

                return { average9, average5, totalUnits9, totalUnits5 };
            };

            const calculateGPA = () => {
                const data = calculateGPAByGrade(grade);
                return {
                    average9: data.average9 !== null ? data.average9 : '-',
                    average5: data.average5 !== null ? data.average5 : '-',
                    totalUnits9: data.totalUnits9,
                    totalUnits5: data.totalUnits5
                };
            };

            // 차트 렌더링
            useEffect(() => {
                if (activeTab !== 'grades' || !canvasRef.current) return;

                const ctx = canvasRef.current.getContext('2d');
                const container = canvasRef.current.parentElement;
                
                const dpr = window.devicePixelRatio || 1;
                const width = container.clientWidth || 400;
                const height = 220;

                canvasRef.current.width = width * dpr;
                canvasRef.current.height = height * dpr;
                canvasRef.current.style.width = `${width}px`;
                canvasRef.current.style.height = `${height}px`;
                ctx.scale(dpr, dpr);

                const points = [
                    { label: '고1', val: calculateGPAByGrade('1').average9 },
                    { label: '고2', val: calculateGPAByGrade('2').average9 },
                    { label: '고3', val: calculateGPAByGrade('3').average9 }
                ];

                const textColor = isDarkMode ? '#cbd5e1' : '#475569';
                const gridColor = isDarkMode ? 'rgba(255,255,255,0.06)' : 'rgba(0,0,0,0.05)';
                const primaryColor = '#2563eb'; 

                const margin = { top: 30, right: 40, bottom: 30, left: 40 };
                const chartWidth = width - margin.left - margin.right;
                const chartHeight = height - margin.top - margin.bottom;

                ctx.clearRect(0, 0, width, height);

                const yLevels = [1, 3, 5, 7, 9];
                ctx.font = 'bold 10px sans-serif';
                ctx.fillStyle = textColor;
                ctx.textAlign = 'right';
                ctx.textBaseline = 'middle';

                yLevels.forEach(level => {
                    const y = margin.top + ((level - 1) / 8) * chartHeight;
                    ctx.beginPath();
                    ctx.strokeStyle = gridColor;
                    ctx.lineWidth = 1;
                    ctx.moveTo(margin.left, y);
                    ctx.lineTo(width - margin.right, y);
                    ctx.stroke();

                    ctx.fillText(`${level}등급`, margin.left - 8, y);
                });

                const xPositions = [
                    margin.left,
                    margin.left + chartWidth / 2,
                    width - margin.right
                ];

                const coordPoints = points.map((p, i) => {
                    if (p.val === null) return null;
                    const valRatio = (p.val - 1) / 8;
                    const y = margin.top + valRatio * chartHeight;
                    return { x: xPositions[i], y, label: p.label, val: p.val };
                });

                ctx.beginPath();
                ctx.strokeStyle = primaryColor;
                ctx.lineWidth = 3;
                ctx.lineCap = 'round';
                ctx.lineJoin = 'round';

                let firstPoint = true;
                coordPoints.forEach(pt => {
                    if (pt) {
                        if (firstPoint) {
                            ctx.moveTo(pt.x, pt.y);
                            firstPoint = false;
                        } else {
                            ctx.lineTo(pt.x, pt.y);
                        }
                    }
                });
                ctx.stroke();

                coordPoints.forEach((pt, i) => {
                    ctx.fillStyle = textColor;
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'top';
                    ctx.fillText(points[i].label, xPositions[i], height - margin.bottom + 8);

                    if (pt) {
                        ctx.beginPath();
                        ctx.arc(pt.x, pt.y, 5, 0, Math.PI * 2);
                        ctx.fillStyle = '#ffffff';
                        ctx.fill();
                        ctx.strokeStyle = primaryColor;
                        ctx.lineWidth = 3;
                        ctx.stroke();

                        ctx.fillStyle = primaryColor;
                        ctx.font = 'extrabold 11px sans-serif';
                        ctx.textAlign = 'center';
                        ctx.textBaseline = 'bottom';
                        ctx.fillText(`${pt.val.toFixed(2)}`, pt.x, pt.y - 8);
                    } else {
                        ctx.fillStyle = isDarkMode ? '#475569' : '#94a3b8';
                        ctx.font = 'normal 10px sans-serif';
                        ctx.textAlign = 'center';
                        ctx.textBaseline = 'middle';
                        ctx.fillText('기록없음', xPositions[i], margin.top + chartHeight / 2);
                    }
                });

            }, [activeTab, allGradesData, isDarkMode]);

            // 클라우드 저장
            const saveToCloud = async () => {
                if (!user || !sdkReady) return;
                const { doc, setDoc } = window.FirebaseSDK;
                setSaveStatus('saving');
                try {
                    const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'records', `grade_${grade}`);
                    const dataToSave = allGradesData[grade];
                    await setDoc(docRef, {
                        fixed: dataToSave.fixed || {},
                        subjects: dataToSave.subjects || [],
                        grades: dataToSave.grades || {},
                        targetDept: selectedDept,
                        lastUpdated: new Date().toISOString()
                    });

                    const metaRef = doc(db, 'artifacts', appId, 'public', 'data', 'all_member_grades', user.uid);
                    await setDoc(metaRef, {
                        userId: user.uid,
                        customId: customId || (user.email ? user.email.split('@')[0] : '익명학생'),
                        shareGrades: shareGrades,
                        joinedRoomId: joinedRoomId || '', 
                        gpa1: calculateGPAByGrade('1').average9,
                        gpa2: calculateGPAByGrade('2').average9,
                        gpa3: calculateGPAByGrade('3').average9,
                        gpa1_5: calculateGPAByGrade('1').average5, 
                        gpa2_5: calculateGPAByGrade('2').average5,
                        gpa3_5: calculateGPAByGrade('3').average5,
                        lastUpdated: new Date().toISOString()
                    }, { merge: true });

                    setSaveStatus('success');
                    setTimeout(() => setSaveStatus(''), 2500);
                } catch (err) {
                    console.error("기록 저장 실패", err);
                    setSaveStatus('error');
                    setTimeout(() => setSaveStatus(''), 3000);
                }
            };

            // Firestore 실시간 모니터링
            useEffect(() => {
                if (!user || isTransitioning || !sdkReady) return;
                const { doc, setDoc, onSnapshot } = window.FirebaseSDK;

                const profileRef = doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'settings');
                const unsubProfile = onSnapshot(profileRef, (snap) => {
                    if (snap.exists()) {
                        const data = snap.data();
                        setCustomId(data.customId || '');
                        setShareGrades(!!data.shareGrades);
                        setJoinedRoomId(data.joinedRoomId || ''); 
                    } else {
                        setCustomId(user.email ? user.email.split('@')[0] : '익명학생');
                        setShareGrades(false);
                        setJoinedRoomId('');
                    }
                }, (error) => {
                    console.warn("프로필 초기 수신 대기:", error.message);
                });

                const unsubscribes = ['1', '2', '3'].map((g) => {
                    const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'records', `grade_${g}`);
                    return onSnapshot(docRef, (docSnap) => {
                        if (docSnap.exists()) {
                            const data = docSnap.data();
                            setAllGradesData(prev => ({
                                ...prev,
                                [g]: {
                                    fixed: data.fixed || {},
                                    subjects: data.subjects || [],
                                    grades: data.grades || {},
                                    targetDept: data.targetDept || '컴퓨터공학과'
                                }
                            }));
                            if (g === grade) {
                                setSelectedDept(data.targetDept || '컴퓨터공학과');
                            }
                        } else {
                            const initialFixed = {};
                            Object.keys(FIXED_STRUCTURE).forEach(key => initialFixed[key] = "");

                            const defaultSubjectsForGrade = DEFAULT_GRADE_SUBJECTS[g].map((sub, index) => ({
                                id: `default_${g}_${index}_${Date.now()}`,
                                name: sub.name,
                                content: sub.content,
                                limit: sub.limit
                            }));

                            setAllGradesData(prev => ({
                                ...prev,
                                [g]: {
                                    fixed: initialFixed,
                                    subjects: defaultSubjectsForGrade,
                                    grades: {},
                                    targetDept: '컴퓨터공학과'
                                }
                            }));
                            if (g === grade) {
                                setSelectedDept('컴퓨터공학과');
                            }
                        }
                    }, (error) => {
                        console.warn(`${g}학년 세션 연동 대기:`, error.message);
                    });
                });

                return () => {
                    unsubProfile();
                    unsubscribes.forEach(unsub => unsub());
                };
            }, [user, grade, isTransitioning, sdkReady]);

            const saveProfileSettings = async (newId, newShare) => {
                if (!user || !sdkReady) return;
                const { doc, setDoc } = window.FirebaseSDK;
                try {
                    const profileRef = doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'settings');
                    await setDoc(profileRef, {
                        customId: newId,
                        shareGrades: newShare,
                        userId: user.uid,
                        joinedRoomId: joinedRoomId, 
                        lastUpdated: new Date().toISOString()
                    }, { merge: true });

                    const metaRef = doc(db, 'artifacts', appId, 'public', 'data', 'all_member_grades', user.uid);
                    await setDoc(metaRef, {
                        userId: user.uid,
                        customId: newId,
                        shareGrades: newShare,
                        joinedRoomId: joinedRoomId, 
                        gpa1: calculateGPAByGrade('1').average9,
                        gpa2: calculateGPAByGrade('2').average9,
                        gpa3: calculateGPAByGrade('3').average9,
                        gpa1_5: calculateGPAByGrade('1').average5, 
                        gpa2_5: calculateGPAByGrade('2').average5,
                        gpa3_5: calculateGPAByGrade('3').average5,
                        lastUpdated: new Date().toISOString()
                    }, { merge: true });

                } catch (e) {
                    console.error("사용자 정보 업데이트 실패:", e);
                }
            };

            const handleAuthAction = async (e, actionType) => {
                e.preventDefault();
                if (!sdkReady) return;
                const { createUserWithEmailAndPassword, signInWithEmailAndPassword, setPersistence, browserLocalPersistence, browserSessionPersistence } = window.FirebaseSDK;
                setMessage({ type: '', text: '' });

                const finalEmail = processEmailInput(email);

                if (!isValidEmail(finalEmail)) {
                    setMessage({ type: 'error', text: '정확한 이메일 형식을 기입해 주세요.' });
                    return;
                }

                try {
                    if (typeof window !== 'undefined') {
                        localStorage.setItem('remember_me', rememberMe ? 'true' : 'false');
                        if (rememberMe) {
                            localStorage.setItem('saved_email', email);
                        } else {
                            localStorage.removeItem('saved_email');
                        }
                    }

                    const persistenceType = rememberMe ? browserLocalPersistence : browserSessionPersistence;
                    await setPersistence(auth, persistenceType);

                    if (actionType === 'signup') {
                        if (password.length < 6) {
                            setMessage({ type: 'error', text: '비밀번호는 최소 6자 이상이어야 합니다.' });
                            return;
                        }
                        await createUserWithEmailAndPassword(auth, finalEmail, password);
                        setMessage({ type: 'success', text: '성공적으로 가입되었습니다! 로그인 중입니다...' });
                    } else if (actionType === 'login') {
                        setActiveRoom(null);
                        setJoinedRoomId('');
                        setChatMessages([]);
                        setRoomUsersData([]);
                        setActiveTab('record');
                        
                        await signInWithEmailAndPassword(auth, finalEmail, password);
                    }
                } catch (err) {
                    if (actionType === 'login' && (err.code === 'auth/user-not-found' || err.code === 'auth/invalid-credential' || err.code === 'auth/wrong-password')) {
                        if (email.trim() === 'admin' || email.trim() === 'admin@admin.com') {
                            try {
                                setMessage({ type: 'success', text: '관리자 데모 계정을 생성 중입니다...' });
                                await createUserWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
                                await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
                                return;
                            } catch (signUpErr) {
                                console.error("데모계정 생성 실패", signUpErr);
                            }
                        }
                    }
                    setMessage({ type: 'error', text: getErrorMessage(err.code) });
                }
            };

            const handleDemoLogin = async () => {
                if (!sdkReady) return;
                const { signInWithEmailAndPassword, createUserWithEmailAndPassword, setPersistence, browserLocalPersistence, browserSessionPersistence } = window.FirebaseSDK;
                setMessage({ type: '', text: '' });
                setEmail('admin');
                setPassword('admin09');
                
                setActiveRoom(null);
                setJoinedRoomId('');
                setChatMessages([]);
                setRoomUsersData([]);
                setActiveTab('record');

                try {
                    const persistenceType = rememberMe ? browserLocalPersistence : browserSessionPersistence;
                    await setPersistence(auth, persistenceType);
                    await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
                } catch (err) {
                    if (err.code === 'auth/user-not-found' || err.code === 'auth/invalid-credential' || err.code === 'auth/wrong-password') {
                        try {
                            await createUserWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
                            await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
                        } catch (createErr) {
                            setMessage({ type: 'error', text: '데모 계정 생성 오류: ' + createErr.message });
                        }
                    } else {
                        setMessage({ type: 'error', text: getErrorMessage(err.code) });
                    }
                }
            };

            const handleLogout = async () => {
                if (!sdkReady) return;
                const { signOut } = window.FirebaseSDK;
                try {
                    setUser(null);
                    setActiveRoom(null);
                    setJoinedRoomId('');
                    setChatMessages([]);
                    setRoomUsersData([]);
                    setActiveTab('record');
                    setAllGradesData({
                        '1': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' },
                        '2': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' },
                        '3': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' }
                    });
                    if (!rememberMe) {
                        setEmail('');
                    }
                    setPassword('');
                    setView('login');
                    setMessage({ type: 'success', text: '안전하게 로그아웃 되었습니다.' });
                    await signOut(auth);
                } catch (err) {
                    console.error("비동기 로그아웃 실패", err);
                    setUser(null);
                    setView('login');
                }
            };

            const updateFixedField = (field, value) => {
                const limit = FIXED_STRUCTURE[field].limit;
                const currentFixed = allGradesData[grade]?.fixed || {};
                const updatedValue = value.length > limit ? value.substring(0, limit) : value;
                
                setAllGradesData(prev => ({
                    ...prev,
                    [grade]: {
                        ...prev[grade],
                        fixed: { ...currentFixed, [field]: updatedValue }
                    }
                }));
            };

            const handleAddSelectedSubjectName = (subjectName) => {
                const currentSubjects = allGradesData[grade]?.subjects || [];

                if (currentSubjects.some(sub => sub.name === subjectName)) {
                    setMessage({ type: 'error', text: `이미 '${subjectName}' 과목이 리스트에 존재합니다.` });
                    setTimeout(() => setMessage({ type: '', text: '' }), 3000);
                    setIsAddingSubject(false);
                    return;
                }

                const newSub = {
                    id: `custom_${Date.now()}`,
                    name: subjectName,
                    content: '',
                    limit: 500 
                };

                setAllGradesData(prev => ({
                    ...prev,
                    [grade]: {
                        ...prev[grade],
                        subjects: [...currentSubjects, newSub]
                    }
                }));

                setIsAddingSubject(false);
                setMessage({ type: '', text: '' });
            };

            const handleAddManualSubject = () => {
                const trimmedName = newSubjectName.trim();
                if (!trimmedName) return;
                handleAddSelectedSubjectName(trimmedName);
                setNewSubjectName('');
            };

            const showConfirm = (title, message, onConfirm) => {
                setModal({
                    show: true,
                    title,
                    message,
                    onConfirm: () => {
                        onConfirm();
                        setModal({ show: false, title: '', message: '', onConfirm: null });
                    }
                });
            };

            const handleDeleteSubject = (id, name) => {
                showConfirm(
                    '과목 삭제 확인',
                    `정말 '${name}' 과목 기록을 삭제하시겠습니까? 삭제한 정보는 되돌릴 수 없습니다.`,
                    () => {
                        const currentSubjects = allGradesData[grade]?.subjects || [];
                        const currentGrades = allGradesData[grade]?.grades || {};
                        const updatedGrades = { ...currentGrades };
                        
                        if (grade === '1') {
                            delete updatedGrades[`${id}_1`];
                            delete updatedGrades[`${id}_2`];
                        }
                        delete updatedGrades[id];

                        setAllGradesData(prev => ({
                            ...prev,
                            [grade]: {
                                ...prev[grade],
                                subjects: currentSubjects.filter(s => s.id !== id),
                                grades: updatedGrades
                            }
                        }));
                    }
                );
            };

            const updateSubjectContent = (id, value) => {
                const currentSubjects = allGradesData[grade]?.subjects || [];
                const updatedSubjects = currentSubjects.map(s => {
                    if (s.id === id) {
                        return {
                            ...s,
                            content: value.length > s.limit ? value.substring(0, s.limit) : value
                        };
                    }
                    return s;
                });

                setAllGradesData(prev => ({
                    ...prev,
                    [grade]: {
                        ...prev[grade],
                        subjects: updatedSubjects
                    }
                }));
            };

            const updateGradeData = (subId, field, value) => {
                const currentGrades = allGradesData[grade]?.grades || {};
                const targetObj = currentGrades[subId] || { units: 0, rank: 0, total: 0, joint: 1 };
                
                const updatedValue = Number(value);
                const updatedObj = { ...targetObj, [field]: updatedValue };

                const { grade9, grade5 } = calculateGradeFromRank(updatedObj.rank, updatedObj.total, updatedObj.joint || 1);
                updatedObj.grade9 = grade9;
                updatedObj.grade5 = grade5;

                const updatedGrades = { ...currentGrades, [subId]: updatedObj };

                setAllGradesData(prev => ({
                    ...prev,
                    [grade]: {
                        ...prev[grade],
                        grades: updatedGrades
                    }
                }));
            };

            const handleDepartmentChange = (deptName) => {
                setSelectedDept(deptName);
                setAllGradesData(prev => ({
                    ...prev,
                    [grade]: {
                        ...prev[grade],
                        targetDept: deptName
                    }
                }));
            };

            // 그룹방 찾기
            const handleSearchRooms = async () => {
                if (!searchCategory.trim() || !sdkReady) return;
                const { collection, getDocs } = window.FirebaseSDK;
                setRoomSearchStatus('searching');
                setMatchingRooms([]);

                try {
                    const roomsColRef = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');
                    const querySnap = await getDocs(roomsColRef);
                    
                    const rooms = [];
                    querySnap.forEach((doc) => {
                        const data = doc.data();
                        if (data.category && data.category.toLowerCase().includes(searchCategory.trim().toLowerCase())) {
                            rooms.push({ id: doc.id, ...data });
                        }
                    });

                    setMatchingRooms(rooms);
                    setRoomSearchStatus(rooms.length === 0 ? 'empty' : '');
                } catch (e) {
                    console.error("방 검색 실패:", e);
                    setRoomSearchStatus('empty');
                }
            };

            // 방 생성
            const handleCreateRoom = async (e) => {
                e.preventDefault();
                if (!groupCategory.trim() || !groupName.trim() || !groupPassword.trim() || !user || !sdkReady) {
                    setMessage({ type: 'error', text: '방 개설 정보를 모두 기입해 주세요.' });
                    return;
                }
                const { collection, addDoc, doc, setDoc } = window.FirebaseSDK;

                try {
                    const roomPayload = {
                        category: groupCategory.trim(),
                        name: groupName.trim(),
                        password: groupPassword,
                        creatorId: user.uid,
                        creatorName: customId || '익명학생',
                        createdAt: new Date().toISOString()
                    };

                    const roomsColRef = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');
                    const docRef = await addDoc(roomsColRef, roomPayload);

                    const newRoom = { id: docRef.id, ...roomPayload };
                    setActiveRoom(newRoom);
                    
                    setJoinedRoomId(newRoom.id);
                    const profileRef = doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'settings');
                    await setDoc(profileRef, { joinedRoomId: newRoom.id }, { merge: true });

                    const metaRef = doc(db, 'artifacts', appId, 'public', 'data', 'all_member_grades', user.uid);
                    await setDoc(metaRef, { 
                        joinedRoomId: newRoom.id,
                        gpa1: calculateGPAByGrade('1').average9,
                        gpa2: calculateGPAByGrade('2').average9,
                        gpa3: calculateGPAByGrade('3').average9,
                        gpa1_5: calculateGPAByGrade('1').average5, 
                        gpa2_5: calculateGPAByGrade('2').average5,
                        gpa3_5: calculateGPAByGrade('3').average5
                    }, { merge: true });

                    setGroupCategory('');
                    setGroupName('');
                    setGroupPassword('');
                    setMessage({ type: 'success', text: `방 '${newRoom.name}'이 개설 및 입장되었습니다.` });
                    setTimeout(() => setMessage({ type: '', text: '' }), 3000);
                } catch (err) {
                    console.error("방 개설 실패", err);
                    setMessage({ type: 'error', text: '방 생성에 실패했습니다.' });
                }
            };

            const handleJoinRoom = async (targetRoom, typedPassword) => {
                if (targetRoom.password !== typedPassword) {
                    setMessage({ type: 'error', text: '비밀번호가 일치하지 않습니다.' });
                    setTimeout(() => setMessage({ type: '', text: '' }), 3000);
                    return;
                }
                if (!sdkReady) return;
                const { doc, setDoc } = window.FirebaseSDK;

                setActiveRoom(targetRoom);
                setJoinedRoomId(targetRoom.id);
                if (user) {
                    const profileRef = doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'settings');
                    await setDoc(profileRef, { joinedRoomId: targetRoom.id }, { merge: true });

                    const metaRef = doc(db, 'artifacts', appId, 'public', 'data', 'all_member_grades', user.uid);
                    await setDoc(metaRef, { 
                        joinedRoomId: targetRoom.id,
                        gpa1: calculateGPAByGrade('1').average9,
                        gpa2: calculateGPAByGrade('2').average9,
                        gpa3: calculateGPAByGrade('3').average9,
                        gpa1_5: calculateGPAByGrade('1').average5, 
                        gpa2_5: calculateGPAByGrade('2').average5,
                        gpa3_5: calculateGPAByGrade('3').average5
                    }, { merge: true });
                }

                setMessage({ type: 'success', text: `'${targetRoom.name}' 방에 참가했습니다!` });
                setTimeout(() => setMessage({ type: '', text: '' }), 3000);
            };

            const handleDeleteRoom = async () => {
                if (!activeRoom || !user || !sdkReady) return;
                const { doc, setDoc, deleteDoc } = window.FirebaseSDK;
                
                if (activeRoom.creatorId !== user.uid) {
                    alert("방 개설자 본인만 이 그룹방을 삭제할 수 있습니다.");
                    return;
                }

                try {
                    setJoinedRoomId('');
                    const profileRef = doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'settings');
                    await setDoc(profileRef, { joinedRoomId: '' }, { merge: true });

                    const metaRef = doc(db, 'artifacts', appId, 'public', 'data', 'all_member_grades', user.uid);
                    await setDoc(metaRef, { joinedRoomId: '' }, { merge: true });

                    const roomDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'rooms', activeRoom.id);
                    await deleteDoc(roomDocRef);

                    setActiveRoom(null);
                    setChatMessages([]);
                    setMatchingRooms([]);
                    setSearchCategory('');
                    
                    setMessage({ type: 'success', text: '그룹방이 안전하게 영구 폐쇄되었습니다.' });
                    setTimeout(() => setMessage({ type: '', text: '' }), 3000);
                } catch (err) {
                    console.error("그룹방 삭제 오류:", err);
                    alert("그룹방 삭제 도중 에러가 발생했습니다.");
                }
            };

            const triggerDeleteRoomConfirm = () => {
                showConfirm(
                    '그룹방 영구 폐쇄',
                    `정말로 '${activeRoom?.name}' 그룹방을 영구히 삭제하시겠습니까?`,
                    handleDeleteRoom
                );
            };

            const handleExitActiveRoom = async () => {
                if (!activeRoom || !user || !sdkReady) return;
                const { doc, setDoc } = window.FirebaseSDK;
                setActiveRoom(null);
                setChatMessages([]);
                setJoinedRoomId('');
                
                try {
                    const profileRef = doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'settings');
                    await setDoc(profileRef, { joinedRoomId: '' }, { merge: true });

                    const metaRef = doc(db, 'artifacts', appId, 'public', 'data', 'all_member_grades', user.uid);
                    await setDoc(metaRef, { joinedRoomId: '' }, { merge: true });
                } catch (e) {
                    console.warn("방 퇴장 실패:", e);
                }
            };

            // 자동 입장 복구
            useEffect(() => {
                if (!user || !joinedRoomId || activeRoom || !sdkReady) return;
                const { collection, getDocs } = window.FirebaseSDK;

                const autoConnectRoom = async () => {
                    try {
                        const roomsColRef = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');
                        const querySnap = await getDocs(roomsColRef);
                        
                        let foundRoom = null;
                        querySnap.forEach((doc) => {
                            if (doc.id === joinedRoomId) {
                                foundRoom = { id: doc.id, ...doc.data() };
                            }
                        });

                        if (foundRoom) {
                            const checkSnap = await getDocs(collection(db, 'artifacts', appId, 'users', user.uid, 'profile'));
                            let actualDBJoinedRoomId = '';
                            checkSnap.forEach((d) => {
                                if (d.id === 'settings') {
                                    actualDBJoinedRoomId = d.data().joinedRoomId || '';
                                }
                            });

                            if (actualDBJoinedRoomId === joinedRoomId) {
                                setActiveRoom(foundRoom);
                            } else {
                                setJoinedRoomId('');
                            }
                        } else {
                            setJoinedRoomId('');
                        }
                    } catch (err) {
                        console.warn("방 재입장 복구 실패:", err);
                    }
                };
                autoConnectRoom();
            }, [user, joinedRoomId, activeRoom, sdkReady]);

            // 실시간 리스너 구독
            useEffect(() => {
                if (!user || !activeRoom || !sdkReady) return;
                const { collection, onSnapshot, doc, setDoc } = window.FirebaseSDK;

                const chatCol = collection(db, 'artifacts', appId, 'public', 'data', `chats_room_${activeRoom.id}`);
                const unsubChat = onSnapshot(chatCol, (snap) => {
                    const msgs = [];
                    snap.forEach(doc => {
                        msgs.push({ id: doc.id, ...doc.data() });
                    });
                    msgs.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt));
                    setChatMessages(msgs);
                }, (error) => {
                    console.error("채팅 로딩 실패:", error);
                });

                const roomsColRef = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');
                const unsubRoomsExist = onSnapshot(roomsColRef, (snap) => {
                    let exist = false;
                    snap.forEach(d => {
                        if (d.id === activeRoom.id) exist = true;
                    });
                    if (!exist) {
                        setActiveRoom(null);
                        setChatMessages([]);
                        setJoinedRoomId('');
                        
                        const profileRef = doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'settings');
                        setDoc(profileRef, { joinedRoomId: '' }, { merge: true }).catch(err => console.warn(err));

                        const metaRef = doc(db, 'artifacts', appId, 'public', 'data', 'all_member_grades', user.uid);
                        setDoc(metaRef, { joinedRoomId: '' }, { merge: true }).catch(err => console.warn(err));
                    }
                });

                const gradesCol = collection(db, 'artifacts', appId, 'public', 'data', 'all_member_grades');
                const unsubGrades = onSnapshot(gradesCol, (snap) => {
                    const list = [];
                    snap.forEach(doc => {
                        const item = doc.data();
                        if (item.shareGrades) {
                            list.push(item);
                        }
                    });
                    setRoomUsersData(list);
                }, (error) => {
                    console.error("전체 성적 구독 실패:", error);
                });

                return () => {
                    unsubChat();
                    unsubRoomsExist();
                    unsubGrades();
                };
            }, [user, activeRoom, sdkReady]);

            useEffect(() => {
                if (chatEndRef.current) {
                    chatEndRef.current.scrollIntoView({ behavior: 'smooth' });
                }
            }, [chatMessages]);

            const handleSendChatMessage = async (e) => {
                e.preventDefault();
                if (!inputChat.trim() || !activeRoom || !user || !sdkReady) return;
                const { collection, addDoc } = window.FirebaseSDK;

                try {
                    const chatCol = collection(db, 'artifacts', appId, 'public', 'data', `chats_room_${activeRoom.id}`);
                    await addDoc(chatCol, {
                        senderId: user.uid,
                        senderName: customId || '익명학생',
                        text: inputChat.trim(),
                        createdAt: new Date().toISOString()
                    });
                    setInputChat('');
                } catch (err) {
                    console.error("메시지 전송 오류:", err);
                }
            };

            const handleCheckTextChange = (value) => {
                setCheckText(value);
                const found = value.match(NEIS_FORBIDDEN_CHARS_REGEX);
                if (found) {
                    setDetectedForbiddenChars(Array.from(new Set(found)));
                } else {
                    setDetectedForbiddenChars([]);
                }
            };

            const handleImportFromRecords = () => {
                const fixedActivities = allGradesData[grade]?.fixed || {};
                const activeSubjects = allGradesData[grade]?.subjects || [];
                
                let combinedText = "";
                Object.entries(fixedActivities).forEach(([key, val]) => {
                    if (val) {
                        combinedText += `[${FIXED_STRUCTURE[key].label}]\n${val}\n\n`;
                    }
                });

                activeSubjects.forEach(sub => {
                    if (sub.content) {
                        combinedText += `[${sub.name} 세특]\n${sub.content}\n\n`;
                    }
                });

                if (!combinedText.trim()) {
                    alert("기록장에 작성된 텍스트가 아직 없습니다.");
                    return;
                }

                handleCheckTextChange(combinedText);
                setSpellCheckResult('');
            };

            const handleCheckSpellAndStyle = async () => {
                if (!checkText.trim()) return;
                setIsCheckingSpell(true);
                setSpellCheckResult('');

                const systemPrompt = `
                당신은 대한민국 고등학교 학생부 기록 전문가 및 한국어 교정 전문가입니다.
                학생이 입력한 학생부용 텍스트를 분석하여, 다음 3가지 사항을 상세히 교정 및 수정 제안해 주세요:
                1. 맞춤법 및 띄어쓰기 오류 수정
                2. 생기부에 어울리지 않는 구어체 표현을 격식 있는 객관적 문체로 수정
                3. 글자수를 아낄 수 있도록 문장을 군더더기 없이 간결하게 다듬기

                [출력 포맷]:
                **[종합 코멘트 및 주의점]**
                - (텍스트 완성도 평가 조언)

                **[맞춤법 & 띄어쓰기 교정 제안]**
                - (틀린 단어) -> (교정안) : 사유 서술

                **[생기부 최적화 추천 완성본]**
                - (학생이 복사 붙여넣기 할 수 있는 완벽한 최종 보정본 문장)
                `;

                try {
                    const result = await callGeminiFeedbackAPI(checkText, systemPrompt);
                    setSpellCheckResult(result);
                } catch (e) {
                    console.error("맞춤법 검사 API 실패:", e);
                    setSpellCheckResult("⚠️ 맞춤법 분석 연동에 실패했습니다. 잠시 후 다시 시도해 주세요.");
                } finally {
                    setIsCheckingSpell(false);
                }
            };

            const handleGenerateAIFeedback = async () => {
                if (!selectedDept.trim()) return;
                setIsGeneratingFeedback(true);
                setAiFeedbackText('');

                const targetGradeData = allGradesData[grade] || {};
                const fixedContent = Object.entries(targetGradeData.fixed || {})
                    .map(([k, v]) => `${FIXED_STRUCTURE[k]?.label}: ${v}`)
                    .join('\n');
                const subjectContent = (targetGradeData.subjects || [])
                    .map(s => `${s.name}: ${s.content}`)
                    .join('\n');

                const totalTextToAnalyze = `
                    [학년]: 고등학교 ${grade}학년
                    [목표학과]: ${selectedDept}
                    
                    [공통/자율 활동기록]:
                    ${fixedContent}
                    
                    [교과세특 활동기록]:
                    ${subjectContent}
                `;

                const systemPrompt = `
                당신은 대입 학생부종합전형 수시 컨설팅 전문가입니다.
                학생이 입력한 고등학교 학생부 활동기록을 기반으로, '${selectedDept}' 학과에 입학하기 위해 필요한 전공 적합성 수준을 분석하는 '수시 맞춤형 피드백 평가 보고서'를 완성도 있게 발행해 주세요.

                아래 구성 요소를 한글로 격식 있게 작성해 주세요:
                1. 학과 소양 매핑도 분석 (해당 학과의 요구 역량이 기록에 충분히 구현되었는지)
                2. 강점(Strong Point) 및 보완점(Weak Point)
                3. 구체적인 다음 탐구 연계 추천 키워드 및 독서 제안
                `;

                try {
                    const result = await callGeminiFeedbackAPI(totalTextToAnalyze, systemPrompt);
                    setAiFeedbackText(result);
                } catch (e) {
                    console.error("AI 피드백 연동 실패:", e);
                    setAiFeedbackText("⚠️ AI 분석 피드백 결과를 생성하지 못했습니다. 다시 시도해 주세요.");
                } finally {
                    setIsGeneratingFeedback(false);
                }
            };

            const currentGradeData = allGradesData[grade] || { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' };
            const records = currentGradeData.fixed || {};
            const subjects = currentGradeData.subjects || [];
            const grades = currentGradeData.grades || {};
            const gradesTabSubjects = getGradesTabSubjects();
            const { average9, average5, totalUnits9, totalUnits5 } = calculateGPA();

            const gradePoints9 = [
                { label: '고1', val: calculateGPAByGrade('1').average9 },
                { label: '고2', val: calculateGPAByGrade('2').average9 },
                { label: '고3', val: calculateGPAByGrade('3').average9 }
            ];
            const trendAnalysis = analyzeTrend(gradePoints9);

            // 대기화면 및 인스턴스 컴파일러 로딩바
            if (!sdkReady || loading) {
                return (
                    <div className="min-h-screen flex items-center justify-center bg-slate-955 text-white">
                        <div className="flex flex-col items-center gap-3">
                            <div className="w-8 h-8 rounded-full border-4 border-blue-500/10 border-t-blue-600 animate-spin"></div>
                            <p className="text-xs font-bold text-slate-400">시스템을 안전하게 호출 중입니다...</p>
                        </div>
                    </div>
                );
            }

            if (view === 'login' || view === 'signup') {
                return (
                    <div className="min-h-screen flex items-center justify-center p-4">
                        <div className="max-w-md w-full rounded-3xl p-8 border shadow-xl bg-white dark:bg-slate-900 border-slate-100 dark:border-slate-800">
                            <div className="flex flex-col items-center mb-8">
                                <div className="w-14 h-14 bg-gradient-to-tr from-blue-600 to-indigo-600 rounded-2xl flex items-center justify-center text-white shadow-lg mb-4">
                                    <SvgIcon name="book-marked" className="w-7 h-7" />
                                </div>
                                <h1 className="text-2xl font-black tracking-tight text-slate-800 dark:text-white">CATCH POINT 생기부 매니저</h1>
                                <p className="text-xs mt-1.5 font-bold text-slate-500 dark:text-slate-400">
                                    {view === 'login' ? '학업 기록 및 교과 세특 통합 관리 플랫폼' : '신규 회원 가입 후 포트폴리오를 작성하세요'}
                                </p>
                            </div>

                            {message.text && (
                                <div className={`p-4 mb-6 rounded-xl border text-xs font-bold flex items-center gap-2 ${
                                    message.type === 'success' 
                                        ? 'bg-green-500/10 text-green-600 dark:text-green-400 border-green-500/20' 
                                        : 'bg-red-500/10 text-red-600 dark:text-red-400 border-red-500/20'
                                }`}>
                                    <SvgIcon name="alert-circle" className="w-4.5 h-4.5 shrink-0" />
                                    <span>{message.text}</span>
                                </div>
                            )}

                            <form onSubmit={(e) => handleAuthAction(e, view === 'login' ? 'login' : 'signup')} className="space-y-4">
                                <div>
                                    <label className="block text-[10px] font-black text-slate-400 mb-1.5 uppercase tracking-wider">이메일 주소</label>
                                    <div className="relative flex items-center">
                                        <div className="absolute left-3.5"><SvgIcon name="users" className="w-4.5 h-4.5 text-slate-400" /></div>
                                        <input 
                                            type="text" 
                                            value={email}
                                            onChange={e => setEmail(e.target.value)}
                                            placeholder="name@example.com (데모: 'admin')"
                                            className="w-full pl-11 pr-4 py-2.5 border rounded-xl font-semibold text-xs outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50 dark:bg-slate-800 border-slate-200 dark:border-slate-700 text-slate-900 dark:text-white"
                                            required
                                        />
                                    </div>
                                </div>

                                <div>
                                    <label className="block text-[10px] font-black text-slate-400 mb-1.5 uppercase tracking-wider">비밀번호</label>
                                    <div className="relative flex items-center">
                                        <div className="absolute left-3.5"><SvgIcon name="lock-keyhole" className="w-4.5 h-4.5 text-slate-400" /></div>
                                        <input 
                                            type="password" 
                                            value={password}
                                            onChange={e => setPassword(e.target.value)}
                                            placeholder="최소 6자 이상 비밀번호 입력"
                                            className="w-full pl-11 pr-4 py-2.5 border rounded-xl font-semibold text-xs outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50 dark:bg-slate-800 border-slate-200 dark:border-slate-700 text-slate-900 dark:text-white"
                                            required
                                        />
                                    </div>
                                </div>

                                {view === 'login' && (
                                    <div className="flex items-center justify-between py-1">
                                        <label className="flex items-center gap-2 cursor-pointer select-none text-xs font-bold text-slate-500 dark:text-slate-400">
                                            <input 
                                                type="checkbox" 
                                                checked={rememberMe}
                                                onChange={(e) => setRememberMe(e.target.checked)}
                                                className="w-4 h-4 rounded border-slate-300 dark:border-slate-700 text-blue-600 focus:ring-blue-500" 
                                            />
                                            <span>이메일 기억하기</span>
                                        </label>
                                    </div>
                                )}

                                <button 
                                    type="submit" 
                                    className="w-full py-3.5 bg-blue-600 hover:bg-blue-700 text-white rounded-xl font-black text-xs shadow-lg transition-all active:scale-95"
                                >
                                    {view === 'login' ? '생기부 매니저 시작하기' : '동의하고 회원가입'}
                                </button>
                            </form>

                            {view === 'login' && (
                                <div className="mt-4">
                                    <button 
                                        onClick={handleDemoLogin}
                                        className="w-full py-3 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl font-black text-xs shadow-md transition-all active:scale-95"
                                    >
                                        데모 어드민 계정으로 바로 입장하기
                                    </button>
                                </div>
                            )}

                            <div className="mt-8 pt-6 border-t border-slate-100 dark:border-slate-800 text-center">
                                {view === 'login' ? (
                                    <p className="text-xs text-slate-404 font-bold">
                                        아직 계정이 없으신가요?{' '}
                                        <button onClick={() => setView('signup')} className="text-blue-600 hover:underline font-extrabold ml-1">
                                            회원가입 하기
                                        </button>
                                    </p>
                                ) : (
                                    <button onClick={() => setView('login')} className="text-xs font-extrabold text-blue-600 hover:underline">
                                        로그인 화면으로 돌아가기
                                    </button>
                                )}
                            </div>
                        </div>
                    </div>
                );
            }

            return (
                <div className="min-h-screen font-sans pb-24 transition-colors duration-300">
                    {/* 헤더바 */}
                    <header className="sticky top-0 z-20 border-b backdrop-blur-md bg-white/80 dark:bg-slate-900/80 border-slate-200/50 dark:border-slate-800/80">
                        <div className="max-w-5xl mx-auto px-4 h-16 flex items-center justify-between">
                            <div className="flex items-center gap-2.5">
                                <div className="w-9 h-9 bg-gradient-to-tr from-blue-600 to-indigo-600 rounded-xl flex items-center justify-center text-white shadow-sm">
                                    <SvgIcon name="book-marked" className="w-5 h-5" />
                                </div>
                                <div className="flex flex-col">
                                    <span className="font-extrabold text-sm tracking-tight text-slate-800 dark:text-white">CATCH POINT</span>
                                    <span className="text-[10px] font-bold hidden sm:block text-slate-400 dark:text-slate-500">생기부 매니저</span>
                                </div>
                            </div>

                            <div className="flex items-center p-1.5 rounded-xl border bg-slate-100 dark:bg-slate-800 border-slate-200/20 dark:border-slate-700/50">
                                {['1', '2', '3'].map((g) => (
                                    <button 
                                        key={g} 
                                        onClick={() => setGrade(g)} 
                                        className={`px-4 py-1.5 rounded-lg text-xs font-black transition-all ${
                                            grade === g 
                                                ? 'bg-white dark:bg-slate-700 text-blue-600 dark:text-blue-400 shadow-sm border border-slate-250/10'
                                                : 'text-slate-400 hover:text-slate-350'
                                        }`}
                                    >
                                        고{g}
                                    </button>
                                ))}
                            </div>

                            <div className="flex items-center gap-2.5">
                                <button 
                                    onClick={() => setIsSettingsOpen(true)}
                                    className="p-2 rounded-xl transition-all text-slate-500 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-800"
                                    title="설정 및 다크모드"
                                >
                                    <SvgIcon name="settings" className="w-5 h-5" />
                                </button>

                                <button 
                                    onClick={handleLogout} 
                                    className="flex items-center gap-1.5 px-3 py-2 rounded-xl text-slate-400 hover:text-red-500 dark:hover:text-red-400 font-bold text-xs"
                                >
                                    <SvgIcon name="logout" className="w-4 h-4" />
                                    <span className="hidden md:inline">로그아웃</span>
                                </button>
                            </div>
                        </div>
                    </header>

                    {/* 대시보드 */}
                    <main className="max-w-4xl mx-auto px-4 pt-6">
                        <div className="mb-8 flex justify-center">
                            <div className="flex p-1.5 rounded-2xl w-full max-w-2xl shadow-sm border bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800">
                                <button
                                    onClick={() => setActiveTab('record')}
                                    className={`flex-1 flex items-center justify-center gap-2 py-3 rounded-xl text-xs md:text-sm font-black transition-all ${
                                        activeTab === 'record' ? 'bg-blue-600 text-white shadow-md' : 'text-slate-500 dark:text-slate-400 hover:bg-slate-50 dark:hover:bg-slate-800/45'
                                    }`}
                                >
                                    <SvgIcon name="file-text" className="w-4 h-4" />
                                    기록장
                                </button>
                                <button
                                    onClick={() => setActiveTab('grades')}
                                    className={`flex-1 flex items-center justify-center gap-2 py-3 rounded-xl text-xs md:text-sm font-black transition-all ${
                                        activeTab === 'grades' ? 'bg-blue-600 text-white shadow-md' : 'text-slate-500 dark:text-slate-400 hover:bg-slate-50 dark:hover:bg-slate-800/45'
                                    }`}
                                >
                                    <SvgIcon name="activity" className="w-4 h-4" />
                                    성적 분석
                                </button>
                                <button
                                    onClick={() => setActiveTab('feedback')}
                                    className={`flex-1 flex items-center justify-center gap-2 py-3 rounded-xl text-xs md:text-sm font-black transition-all ${
                                        activeTab === 'feedback' ? 'bg-blue-600 text-white shadow-md' : 'text-slate-500 dark:text-slate-400 hover:bg-slate-50 dark:hover:bg-slate-800/45'
                                    }`}
                                >
                                    <SvgIcon name="brain-circuit" className="w-4 h-4" />
                                    AI 피드백
                                </button>
                                <button
                                    onClick={() => setActiveTab('group')}
                                    className={`flex-1 flex items-center justify-center gap-2 py-3 rounded-xl text-xs md:text-sm font-black transition-all ${
                                        activeTab === 'group' ? 'bg-blue-600 text-white shadow-md' : 'text-slate-500 dark:text-slate-400 hover:bg-slate-50 dark:hover:bg-slate-800/45'
                                    }`}
                                >
                                    <SvgIcon name="users" className="w-4 h-4" />
                                    그룹방
                                </button>
                            </div>
                        </div>

                        {/* 안내 패널 */}
                        <div className="mb-10 flex flex-col md:flex-row md:items-end justify-between gap-6">
                            <div className="space-y-2">
                                <div className="inline-flex items-center gap-1.5 px-2.5 py-1 border rounded-full text-[10px] font-black bg-gradient-to-r from-blue-50 to-indigo-50 dark:from-blue-950/30 border-blue-100/50 dark:border-blue-900/50 text-blue-600 dark:text-blue-400">
                                    <SvgIcon name="sparkles" className="w-3.5 h-3.5 text-blue-505 animate-pulse" />
                                    2022 개정 교육과정 규칙 완벽 세팅됨
                                </div>
                                <h2 className="text-2xl md:text-3xl font-extrabold tracking-tight text-slate-800 dark:text-white">
                                    고등학교 {grade}학년 {
                                        activeTab === 'record' ? '생기부 기록장' : 
                                        activeTab === 'grades' ? '교과 성적 관리기' : 
                                        activeTab === 'feedback' ? 'AI 피드백 가이드' : '수시 정보 공유 그룹방'
                                    }
                                </h2>
                                <p className="text-xs font-semibold leading-relaxed text-slate-500 dark:text-slate-400">
                                    {activeTab === 'record' && '학년별 교과세특과 창의적 체험활동을 체계적으로 기록하고 바이트를 연산해보세요.'}
                                    {activeTab === 'grades' && '본인의 석차와 총원 정보를 적으면, 9등급제 및 개정 5등급제 등급이 계산됩니다.'}
                                    {activeTab === 'feedback' && '목표 대학 학과 맞춤 피드백과 생기부 맞춤법 및 특수문자 검사를 진행해 보세요.'}
                                    {activeTab === 'group' && '학교별 스터디 그룹방에 소속되어 멤버 간 성적을 공유하고 입시 정보를 공유하세요.'}
                                </p>
                            </div>
                            
                            <div className="flex items-center gap-3">
                                {saveStatus === 'saving' && (
                                    <span className="text-[11px] font-bold text-slate-400 flex items-center gap-1.5">
                                        <SvgIcon name="refresh-cw" className="w-3 h-3 animate-spin text-blue-500" />
                                        보안 서버에 안전하게 백업 중...
                                    </span>
                                )}
                                {saveStatus === 'success' && (
                                    <span className="text-[11px] font-extrabold text-green-600 bg-green-50 dark:bg-green-950/30 px-3 py-1.5 rounded-xl border border-green-100 dark:border-green-900/50 flex items-center gap-1">
                                        <SvgIcon name="check" className="w-3.5 h-3.5" />
                                        클라우드 저장 및 백업 완료!
                                    </span>
                                )}
                                {saveStatus === 'error' && (
                                    <span className="text-[11px] font-extrabold text-red-500 bg-red-50/30 px-3 py-1.5 rounded-xl border border-red-100 dark:border-red-900/50 flex items-center gap-1">
                                        <SvgIcon name="alert-circle" className="w-3.5 h-3.5" />
                                        임시 지연 발생 (재시도 중)
                                    </span>
                                )}
                                <button 
                                    onClick={saveToCloud} 
                                    className="flex items-center gap-2 px-6 py-3.5 bg-blue-600 hover:bg-blue-700 text-white rounded-2xl font-black shadow-lg transition-all active:scale-95 text-xs md:text-sm"
                                >
                                    <SvgIcon name="save" className="w-4.5 h-4.5" />
                                    클라우드 영구 저장
                                </button>
                            </div>
                        </div>

                        {/* ==========================================
                            [TAB 1: RECORD]
                        ========================================== */}
                        {activeTab === 'record' && (
                            <>
                                <section className="mb-14">
                                    <div className="flex items-center justify-between mb-5">
                                        <h3 className="text-lg font-black flex items-center gap-2 text-slate-800 dark:text-white">
                                            <span className="w-1.5 h-5 bg-gradient-to-b from-blue-500 to-indigo-600 rounded-full inline-block" />
                                            교과 학습 발달 상황 (과목별 세특)
                                        </h3>
                                        {grade !== '1' && (
                                            <button 
                                                onClick={() => setIsAddingSubject(true)}
                                                className="flex items-center gap-1 px-3 py-1.5 rounded-lg font-extrabold text-xs transition-all bg-blue-50 dark:bg-slate-800 text-blue-600 dark:text-blue-400 hover:bg-blue-100"
                                            >
                                                <SvgIcon name="plus" className="w-3.5 h-3.5" />
                                                과목 추가 및 선택
                                            </button>
                                        )}
                                    </div>

                                    {isAddingSubject && grade !== '1' && (
                                        <div className="mb-6 p-6 border rounded-2xl shadow-sm bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 space-y-5">
                                            {SELECTABLE_OPTIONAL_SUBJECTS[grade] && SELECTABLE_OPTIONAL_SUBJECTS[grade].length > 0 && (
                                                <div>
                                                    <span className="block text-xs font-black text-slate-400 mb-2.5 tracking-wide">
                                                        추천 선택과목 목록 (클릭 시 자동 추가됨)
                                                    </span>
                                                    <div className="flex flex-wrap gap-2 max-h-48 overflow-y-auto p-1 border border-slate-100 dark:border-slate-800 rounded-xl bg-slate-50/40 dark:bg-slate-955/20">
                                                        {SELECTABLE_OPTIONAL_SUBJECTS[grade].map((subName, index) => {
                                                            const isAlreadyAdded = subjects.some(s => s.name === subName);
                                                            return (
                                                                <button
                                                                    key={index}
                                                                    onClick={() => handleAddSelectedSubjectName(subName)}
                                                                    disabled={isAlreadyAdded}
                                                                    className={`px-3 py-1.5 text-xs font-bold rounded-lg border transition-all ${
                                                                        isAlreadyAdded 
                                                                            ? 'bg-slate-100 dark:bg-slate-800 text-slate-400 border-transparent cursor-not-allowed'
                                                                            : 'bg-white dark:bg-slate-800 text-slate-700 dark:text-slate-300 border-slate-200 dark:border-slate-700 hover:border-blue-300'
                                                                    }`}
                                                                >
                                                                    {subName} {isAlreadyAdded && '(추가됨)'}
                                                                </button>
                                                            );
                                                        })}
                                                    </div>
                                                </div>
                                            )}

                                            <div className="pt-2 border-t border-slate-100 dark:border-slate-800/50">
                                                <label className="block text-[10px] font-extrabold text-slate-400 mb-2 tracking-wider">직접 수동 추가</label>
                                                <div className="flex gap-2">
                                                    <input 
                                                        type="text" 
                                                        value={newSubjectName}
                                                        onChange={e => setNewSubjectName(e.target.value)}
                                                        onKeyDown={e => e.key === 'Enter' && handleAddManualSubject()}
                                                        placeholder="예: 고급 수학, 인공지능 기초"
                                                        className="flex-1 px-4 py-2.5 border rounded-xl focus:ring-2 focus:ring-blue-500 font-semibold text-xs outline-none bg-slate-50 dark:bg-slate-800 border-slate-100 dark:border-slate-700 text-slate-900 dark:text-white placeholder:text-slate-400"
                                                        maxLength={25}
                                                    />
                                                    <button onClick={handleAddManualSubject} className="px-5 bg-blue-600 text-white font-bold rounded-xl hover:bg-blue-700 transition-colors text-xs">등록</button>
                                                    <button onClick={() => { setIsAddingSubject(false); setNewSubjectName(''); }} className="px-3 font-bold rounded-xl text-xs bg-slate-100 dark:bg-slate-800 text-slate-400">취소</button>
                                                </div>
                                            </div>
                                        </div>
                                    )}

                                    <div className="grid gap-6">
                                        {subjects.length === 0 && !isAddingSubject && (
                                            <div className="py-14 text-center rounded-2xl border border-dashed bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800">
                                                <SvgIcon name="file-text" className="w-10 h-10 text-slate-200 dark:text-slate-800 mx-auto mb-3" />
                                                <p className="font-semibold text-xs text-slate-400">등록된 과목 세특란이 없습니다.</p>
                                            </div>
                                        )}
                                        
                                        {subjects.map((sub) => {
                                            const charCount = sub.content ? sub.content.length : 0;
                                            const byteCount = getByteLength(sub.content);
                                            const isWarning = byteCalcMode === 'char' ? charCount >= sub.limit * 0.9 : byteCount >= (sub.limit * 3) * 0.9;
                                            const currentCount = byteCalcMode === 'char' ? charCount : byteCount;
                                            const maxLimit = byteCalcMode === 'char' ? sub.limit : (sub.limit * 3);
                                            const isLiberal = isLiberalArtSubject(sub.name);

                                            return (
                                                <div key={sub.id} className="rounded-2xl border shadow-sm overflow-hidden bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800">
                                                    <div className="px-6 py-4 border-b flex items-center justify-between bg-slate-50/50 dark:bg-slate-900/50 border-slate-100 dark:border-slate-800">
                                                        <div className="flex items-center gap-2 text-slate-800 dark:text-white">
                                                            <span className="w-2 h-2 rounded-full bg-blue-500" />
                                                            <span className="font-bold text-sm">{sub.name} 세부능력 및 특기사항</span>
                                                            {isLiberal && (
                                                                <span className="text-[9px] bg-amber-50 dark:bg-amber-950/20 text-amber-600 border border-amber-200 dark:border-amber-900/50 font-black px-1.5 py-0.5 rounded-md">
                                                                    성적 제외 과목
                                                                </span>
                                                            )}
                                                        </div>
                                                        <div className="flex items-center gap-3">
                                                            <div className={`text-[10px] font-black px-2 py-1 rounded-md border ${
                                                                isWarning 
                                                                    ? 'bg-red-50 dark:bg-red-950/20 text-red-500 border-red-100 dark:border-red-900/50 animate-pulse' 
                                                                    : 'bg-white dark:bg-slate-800 text-slate-400 border-slate-200 dark:border-slate-700'
                                                            }`}>
                                                                {currentCount} / {maxLimit}{byteCalcMode === 'char' ? '자' : 'Byte'}
                                                            </div>
                                                            {grade !== '1' && (
                                                                <button 
                                                                    onClick={() => handleDeleteSubject(sub.id, sub.name)} 
                                                                    className="text-slate-300 dark:text-slate-650 hover:text-red-500 transition-colors p-1"
                                                                    title="과목 삭제"
                                                                >
                                                                    <SvgIcon name="trash-2" className="w-4 h-4" />
                                                                </button>
                                                            )}
                                                        </div>
                                                    </div>
                                                    <div className="p-5">
                                                        <textarea
                                                            value={sub.content}
                                                            onChange={(e) => updateSubjectContent(sub.id, e.target.value)}
                                                            placeholder={`${sub.name} 과목 수업 중 학업 기여 내역을 기입해보세요.`}
                                                            className="w-full h-36 bg-transparent border-none resize-none leading-relaxed font-semibold text-sm outline-none focus:ring-0 text-slate-700 dark:text-slate-200 placeholder:text-slate-400"
                                                        />
                                                    </div>
                                                </div>
                                            );
                                        })}
                                    </div>
                                </section>

                                <section>
                                    <h3 className="text-lg font-black mb-5 flex items-center gap-2 text-slate-800 dark:text-white">
                                        <span className="w-1.5 h-5 bg-gradient-to-b from-blue-500 to-indigo-600 rounded-full inline-block" />
                                        창의적 체험활동 및 행동특성 종합의견 초안
                                    </h3>
                                    <div className="grid gap-6">
                                        {Object.entries(FIXED_STRUCTURE).map(([key, config]) => {
                                            const value = records[key] || "";
                                            const charCount = value.length;
                                            const byteCount = getByteLength(value);
                                            const isWarning = byteCalcMode === 'char' ? charCount >= config.limit * 0.9 : byteCount >= (config.limit * 3) * 0.9;
                                            const currentCount = byteCalcMode === 'char' ? charCount : byteCount;
                                            const maxLimit = byteCalcMode === 'char' ? config.limit : (config.limit * 3);

                                            return (
                                                <div key={key} className="rounded-2xl border p-6 shadow-sm bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800">
                                                    <div className="flex items-center justify-between mb-3.5">
                                                        <div className="flex flex-col text-slate-800 dark:text-white">
                                                            <label className="text-sm font-extrabold">{config.label}</label>
                                                            <span className="text-[10px] text-slate-400 font-semibold mt-0.5">{config.tip}</span>
                                                        </div>
                                                        <div className={`text-[10px] font-black px-2 py-1 rounded-md border ${
                                                            isWarning 
                                                                ? 'bg-red-50 dark:bg-red-950/20 text-red-500 border-red-100 dark:border-red-900/50' 
                                                                : 'bg-slate-50 dark:bg-slate-800 text-slate-400 border-slate-100 dark:border-slate-700'
                                                        }`}>
                                                            {currentCount} / {maxLimit}{byteCalcMode === 'char' ? '자' : 'Byte'}
                                                        </div>
                                                    </div>
                                                    <textarea
                                                        value={value}
                                                        onChange={(e) => updateFixedField(key, e.target.value)}
                                                        placeholder={config.placeholder}
                                                        className="w-full h-40 p-4 border rounded-xl focus:ring-2 focus:outline-none transition-all resize-none leading-relaxed font-semibold text-sm bg-slate-50/50 dark:bg-slate-800 border-slate-100 dark:border-slate-700 focus:ring-blue-100 dark:focus:ring-blue-900 text-slate-700 dark:text-slate-200 placeholder:text-slate-400"
                                                    />
                                                </div>
                                            );
                                        })}
                                    </div>
                                </section>
                            </>
                        )}

                        {/* ==========================================
                            [TAB 2: GRADES]
                        ========================================== */}
                        {activeTab === 'grades' && (
                            <div className="space-y-8">
                                <div className="grid grid-cols-1 md:grid-cols-2 gap-6 p-6 rounded-3xl border bg-gradient-to-tr from-blue-50/60 to-indigo-50/50 dark:from-slate-900 border-blue-100/50 dark:border-slate-800">
                                    <div className="p-6 rounded-2xl flex items-center justify-between bg-white dark:bg-slate-900/50 shadow-sm border border-slate-100 dark:border-slate-800">
                                        <div className="space-y-1">
                                            <span className="text-[11px] font-extrabold flex items-center gap-1 text-slate-500 dark:text-slate-400">
                                                <SvgIcon name="award" className="w-3.5 h-3.5 text-blue-500" />
                                                현재 학년 종합 평점 (9등급제)
                                            </span>
                                            <p className="text-4xl font-black tracking-tight text-blue-600">{average9} <span className="text-xs text-slate-500 font-bold">등급</span></p>
                                            <p className="text-[10px] text-slate-400 font-bold">이수단위: {totalUnits9} 단위</p>
                                        </div>
                                        <div className="w-12 h-12 rounded-full flex items-center justify-center bg-blue-50 dark:bg-blue-950/30">
                                            <span className="text-lg font-black text-blue-600">9</span>
                                        </div>
                                    </div>

                                    <div className="p-6 rounded-2xl flex items-center justify-between bg-white dark:bg-slate-900/50 shadow-sm border border-slate-100 dark:border-slate-800">
                                        <div className="space-y-1">
                                            <span className="text-[11px] font-extrabold flex items-center gap-1 text-slate-500 dark:text-slate-400">
                                                <SvgIcon name="award" className="w-3.5 h-3.5 text-indigo-500" />
                                                고교학점제 내신 변환 (5등급제)
                                            </span>
                                            <p className="text-4xl font-black tracking-tight text-indigo-600">{average5} <span className="text-xs text-slate-500 font-bold">등급</span></p>
                                            <p className="text-[10px] text-slate-400 font-bold">이수단위: {totalUnits5} 단위</p>
                                        </div>
                                        <div className="w-12 h-12 rounded-full flex items-center justify-center bg-indigo-50 dark:bg-indigo-950/30">
                                            <span className="text-lg font-black text-indigo-600">5</span>
                                        </div>
                                    </div>
                                </div>

                                <div className="rounded-3xl border p-6 bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 shadow-sm">
                                    <div className="mb-6">
                                        <h3 className="text-base font-extrabold flex items-center gap-2 text-slate-800 dark:text-white">
                                            <SvgIcon name="activity" className="w-5 h-5 text-blue-500" />
                                            연도별 내신 등급 변화 추이 (1~3학년)
                                        </h3>
                                    </div>

                                    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 items-center">
                                        <div className="lg:col-span-2 flex justify-center bg-slate-50/50 dark:bg-slate-950/40 p-4 rounded-2xl border border-slate-100 dark:border-slate-800/60">
                                            <canvas ref={canvasRef} className="max-w-full h-auto" />
                                        </div>

                                        <div className={`p-5 rounded-2xl border h-full flex flex-col justify-center ${
                                            trendAnalysis.icon === 'up' 
                                                ? 'bg-green-500/5 border-green-200/50' 
                                                : trendAnalysis.icon === 'down' 
                                                ? 'bg-red-500/5 border-red-200/50' 
                                                : 'bg-slate-500/5 border-slate-200/50 dark:border-slate-800'
                                        }`}>
                                            <p className="text-[11px] leading-relaxed text-slate-600 dark:text-slate-400 font-bold whitespace-pre-line">
                                                {trendAnalysis.text}
                                            </p>
                                        </div>
                                    </div>
                                </div>

                                <div className="rounded-3xl border p-6 bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 shadow-sm">
                                    <h3 className="text-base font-extrabold mb-4 pb-3 border-b text-slate-800 dark:text-white">과목별 석차 및 응시자 수 등록</h3>
                                    <div className="overflow-x-auto">
                                        <table className="w-full text-left border-collapse min-w-[700px]">
                                            <thead>
                                                <tr className="border-b text-xs font-black text-slate-400 border-slate-100 dark:border-slate-800">
                                                    <th className="py-3 px-2">과목명</th>
                                                    <th className="py-3 px-2 w-20">이수단위</th>
                                                    <th className="py-3 px-2 w-28">총 응시자 (명)</th>
                                                    <th className="py-3 px-2 w-24">본인 석차 (등)</th>
                                                    <th className="py-3 px-2 w-24">동석차 수 (명)</th>
                                                    <th className="py-3 px-2 w-24 text-center">9등급제</th>
                                                    <th className="py-3 px-2 w-24 text-center">고교학점제</th>
                                                </tr>
                                            </thead>
                                            <tbody>
                                                {gradesTabSubjects
                                                    .filter(sub => !isLiberalArtSubject(sub.name))
                                                    .map((sub) => {
                                                        const subGrade = grades[sub.id] || { units: '', rank: '', total: '', joint: 1 };
                                                        const { grade9, grade5, percentage } = calculateGradeFromRank(
                                                            subGrade.rank, subGrade.total, subGrade.joint || 1
                                                        );

                                                        return (
                                                            <tr key={sub.id} className="border-b text-sm transition-colors border-slate-50 dark:border-slate-800/80 hover:bg-slate-50/50 dark:hover:bg-slate-800/40">
                                                                <td className="py-4 px-2 font-bold">{sub.name}</td>
                                                                <td className="py-4 px-2">
                                                                    <input 
                                                                        type="number" 
                                                                        value={subGrade.units || ''} 
                                                                        onChange={(e) => updateGradeData(sub.id, 'units', e.target.value)}
                                                                        placeholder="단위"
                                                                        className="w-16 px-2 py-1.5 text-xs font-bold rounded-lg border bg-slate-50 dark:bg-slate-800 border-slate-200 dark:border-slate-700 text-center"
                                                                    />
                                                                </td>
                                                                <td className="py-4 px-2">
                                                                    <input 
                                                                        type="number" 
                                                                        value={subGrade.total || ''} 
                                                                        onChange={(e) => updateGradeData(sub.id, 'total', e.target.value)}
                                                                        placeholder="총원"
                                                                        className="w-24 px-2 py-1.5 text-xs font-bold rounded-lg border bg-slate-50 dark:bg-slate-800 border-slate-200 dark:border-slate-700 text-center"
                                                                    />
                                                                </td>
                                                                <td className="py-4 px-2">
                                                                    <input 
                                                                        type="number" 
                                                                        value={subGrade.rank || ''} 
                                                                        onChange={(e) => updateGradeData(sub.id, 'rank', e.target.value)}
                                                                        placeholder="등수"
                                                                        className="w-20 px-2 py-1.5 text-xs font-bold rounded-lg border bg-slate-50 dark:bg-slate-800 border-slate-200 dark:border-slate-700 text-center"
                                                                    />
                                                                </td>
                                                                <td className="py-4 px-2">
                                                                    <input 
                                                                        type="number" 
                                                                        value={subGrade.joint || ''} 
                                                                        onChange={(e) => updateGradeData(sub.id, 'joint', e.target.value)}
                                                                        placeholder="1"
                                                                        className="w-16 px-2 py-1.5 text-xs font-bold rounded-lg border bg-slate-50 dark:bg-slate-800 border-slate-200 dark:border-slate-700 text-center"
                                                                    />
                                                                </td>
                                                                <td className="py-4 px-2 text-center">
                                                                    {grade9 !== null ? (
                                                                        <div className="inline-block px-2.5 py-1 rounded-lg bg-blue-50 dark:bg-blue-950/30 text-blue-650 dark:text-blue-400 font-extrabold text-xs">
                                                                            {grade9}등급
                                                                            <span className="block text-[8px] font-semibold text-slate-400 mt-0.5">{percentage.toFixed(1)}%</span>
                                                                        </div>
                                                                    ) : '-'}
                                                                </td>
                                                                <td className="py-4 px-2 text-center">
                                                                    {grade5 !== null ? (
                                                                        <div className="inline-block px-2.5 py-1 rounded-lg bg-indigo-50 dark:bg-indigo-950/30 text-indigo-600 dark:text-indigo-400 font-extrabold text-xs">
                                                                            {grade5}등급
                                                                        </div>
                                                                    ) : '-'}
                                                                </td>
                                                            </tr>
                                                        );
                                                    })}
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                        )}

                        {/* ==========================================
                            [TAB 3: FEEDBACK]
                        ========================================== */}
                        {activeTab === 'feedback' && (
                            <div className="space-y-8">
                                <div className="p-6 rounded-3xl border bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 shadow-sm">
                                    <h3 className="text-base font-extrabold flex items-center gap-2 text-slate-800 dark:text-white mb-2">
                                        <SvgIcon name="graduation-cap" className="w-5 h-5 text-blue-500" />
                                        목표 대학 전공학과 기재
                                    </h3>
                                    <input
                                        type="text"
                                        value={selectedDept}
                                        onChange={(e) => handleDepartmentChange(e.target.value)}
                                        placeholder="목표 학과명 입력 (예: 컴퓨터공학과, 생명과학부 등)"
                                        className="w-full px-4 py-3 border rounded-xl font-bold text-xs bg-slate-50 dark:bg-slate-800 border-slate-200 dark:border-slate-700 text-slate-900 dark:text-white outline-none"
                                    />
                                </div>

                                <div className="p-8 rounded-3xl border bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 shadow-sm">
                                    <div className="flex flex-col md:flex-row md:items-center justify-between gap-6 mb-6">
                                        <div className="flex items-center gap-3">
                                            <div className="w-10 h-10 bg-gradient-to-tr from-blue-600 to-indigo-600 rounded-xl flex items-center justify-center text-white shrink-0">
                                                <SvgIcon name="brain-circuit" className="w-5 h-5 animate-pulse" />
                                            </div>
                                            <div>
                                                <h3 className="text-base font-extrabold text-slate-800 dark:text-white">AI 전공 적합성 피드백</h3>
                                            </div>
                                        </div>
                                        <button
                                            onClick={handleGenerateAIFeedback}
                                            disabled={isGeneratingFeedback || !selectedDept.trim()}
                                            className="flex items-center justify-center gap-2 px-6 py-3 bg-gradient-to-r from-blue-600 to-indigo-600 text-white text-xs font-black rounded-xl shadow-md disabled:opacity-50"
                                        >
                                            {isGeneratingFeedback ? '대입 분석 보고서 생성 중...' : 'AI 피드백 리포트 작성'}
                                        </button>
                                    </div>

                                    {isGeneratingFeedback ? (
                                        <div className="py-14 text-center">분석 엔진이 작동 중입니다...</div>
                                    ) : aiFeedbackText ? (
                                        <div className="p-6 rounded-2xl border text-xs leading-relaxed whitespace-pre-wrap bg-slate-50 dark:bg-slate-950/40 border-slate-100 dark:border-slate-800">
                                            {aiFeedbackText}
                                        </div>
                                    ) : (
                                        <div className="py-10 text-center border border-dashed rounded-xl text-slate-400">학과를 기입하고 리포트를 생성해보세요.</div>
                                    )}
                                </div>

                                <div className="p-8 rounded-3xl border bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 shadow-sm space-y-6">
                                    <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 pb-4 border-b">
                                        <h3 className="text-base font-extrabold flex items-center gap-2 text-slate-800 dark:text-white">
                                            <SvgIcon name="spell-check" className="w-5 h-5 text-amber-500" />
                                            생기부 맞춤법 및 불용 특수기호 교정
                                        </h3>
                                        <div className="flex items-center gap-2">
                                            <button onClick={handleImportFromRecords} className="px-3 py-1.5 bg-slate-100 dark:bg-slate-800 rounded-lg text-xs font-bold">기록장 전체 가져오기</button>
                                            <button onClick={handleCheckSpellAndStyle} disabled={isCheckingSpell || !checkText.trim()} className="px-3 py-1.5 bg-amber-500 text-white rounded-lg text-xs font-bold disabled:opacity-55">맞춤법 검증 시작</button>
                                        </div>
                                    </div>

                                    {detectedForbiddenChars.length > 0 && (
                                        <div className="p-4 rounded-xl bg-amber-500/10 border border-amber-500/30 text-xs text-amber-600 font-bold">
                                            ⚠️ NEIS 입력 불가 특수문자 검출: <span className="text-red-500">{detectedForbiddenChars.join(', ')}</span> (정리 권장)
                                        </div>
                                    )}

                                    <div className="grid grid-cols-1 lg:grid-cols-5 gap-6">
                                        <div className="lg:col-span-3">
                                            <textarea
                                                value={checkText}
                                                onChange={(e) => handleCheckTextChange(e.target.value)}
                                                placeholder="검사할 문장을 입력하세요."
                                                className="w-full h-64 p-4 border rounded-xl bg-slate-50 dark:bg-slate-900 focus:outline-none"
                                            />
                                        </div>
                                        <div className="lg:col-span-2">
                                            {isCheckingSpell ? (
                                                <div className="text-center py-20 text-xs">교정 제안을 분석하고 있습니다...</div>
                                            ) : (
                                                <div className="p-4 border rounded-xl bg-slate-50/50 dark:bg-slate-900/50 h-64 overflow-y-auto text-xs leading-relaxed whitespace-pre-wrap">
                                                    {spellCheckResult || '제안된 분석 내용이 표시됩니다.'}
                                                </div>
                                            )}
                                        </div>
                                    </div>
                                </div>
                            </div>
                        )}

                        {/* ==========================================
                            [TAB 4: GROUP]
                        ========================================== */}
                        {activeTab === 'group' && (
                            <div className="space-y-8">
                                {!activeRoom ? (
                                    <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
                                        <div className="p-6 rounded-3xl border bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 shadow-sm">
                                            <h3 className="text-base font-black text-slate-850 dark:text-white mb-4">새로운 그룹방 만들기</h3>
                                            <form onSubmit={handleCreateRoom} className="space-y-4">
                                                <input 
                                                    type="text" 
                                                    value={groupCategory} 
                                                    onChange={e => setGroupCategory(e.target.value)} 
                                                    placeholder="학교이름 기입" 
                                                    className="w-full px-3 py-2 border rounded-xl text-xs bg-slate-50 dark:bg-slate-800" 
                                                    required 
                                                />
                                                <input 
                                                    type="text" 
                                                    value={groupName} 
                                                    onChange={e => setGroupName(e.target.value)} 
                                                    placeholder="방 이름 기입" 
                                                    className="w-full px-3 py-2 border rounded-xl text-xs bg-slate-50 dark:bg-slate-800" 
                                                    required 
                                                />
                                                <input 
                                                    type="password" 
                                                    value={groupPassword} 
                                                    onChange={e => setGroupPassword(e.target.value)} 
                                                    placeholder="참여 비밀번호" 
                                                    className="w-full px-3 py-2 border rounded-xl text-xs bg-slate-50 dark:bg-slate-800" 
                                                    required 
                                                />
                                                <button type="submit" className="w-full py-3 bg-blue-600 text-white rounded-xl text-xs font-bold">생성 후 자동 입장</button>
                                            </form>
                                        </div>

                                        <div className="p-6 rounded-3xl border bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 shadow-sm">
                                            <h3 className="text-base font-black text-slate-850 dark:text-white mb-4">개설 그룹방 탐색</h3>
                                            <div className="flex gap-2 mb-4">
                                                <input 
                                                    type="text" 
                                                    value={searchCategory} 
                                                    onChange={e => setSearchCategory(e.target.value)} 
                                                    placeholder="학교 이름 검색" 
                                                    className="flex-1 px-3 py-2 border rounded-xl text-xs bg-slate-50 dark:bg-slate-800" 
                                                />
                                                <button onClick={handleSearchRooms} className="px-4 py-2 bg-indigo-600 text-white rounded-xl text-xs font-bold">검색</button>
                                            </div>

                                            <div className="space-y-2">
                                                {matchingRooms.map(room => (
                                                    <div key={room.id} className="p-3 border rounded-xl flex items-center justify-between bg-slate-50/50 dark:bg-slate-800/20">
                                                        <div>
                                                            <div className="text-xs font-bold">{room.name}</div>
                                                            <div className="text-[10px] text-slate-400">개설: {room.creatorName}</div>
                                                        </div>
                                                        <button 
                                                            onClick={() => setPasswordModal({ show: true, targetRoom: room, typedPassword: '' })}
                                                            className="px-3 py-1.5 bg-slate-900 dark:bg-slate-800 text-white text-[10px] rounded-lg"
                                                        >
                                                            입장
                                                        </button>
                                                    </div>
                                                ))}
                                            </div>
                                        </div>
                                    </div>
                                ) : (
                                    <div className="space-y-6">
                                        <div className="p-6 rounded-3xl border bg-gradient-to-r from-blue-600 to-indigo-600 text-white flex flex-col md:flex-row md:items-center justify-between gap-4">
                                            <div>
                                                <span className="text-[10px] bg-white/20 px-2 py-0.5 rounded font-black">{activeRoom.category}</span>
                                                <h3 className="text-xl font-black mt-1">{activeRoom.name}</h3>
                                            </div>
                                            <div className="flex gap-2">
                                                {activeRoom.creatorId === user?.uid && (
                                                    <button onClick={triggerDeleteRoomConfirm} className="px-3 py-1.5 bg-red-650 hover:bg-red-700 rounded-lg text-xs font-bold text-white">방 삭제</button>
                                                )}
                                                <button onClick={handleExitActiveRoom} className="px-3 py-1.5 bg-white/10 rounded-lg text-xs font-bold">퇴장</button>
                                            </div>
                                        </div>

                                        <div className="grid grid-cols-1 lg:grid-cols-5 gap-6">
                                            <div className="lg:col-span-3 p-6 rounded-3xl border bg-white dark:bg-slate-900 shadow-sm">
                                                <h4 className="text-sm font-black border-b pb-3 mb-4 text-slate-800 dark:text-white">실시간 그룹 성적표</h4>
                                                <div className="overflow-x-auto">
                                                    <table className="w-full text-left min-w-[450px]">
                                                        <thead>
                                                            <tr className="border-b text-[10px] font-black text-slate-400">
                                                                <th className="py-2">사용자 ID</th>
                                                                <th className="py-2 text-center">고1 GPA</th>
                                                                <th className="py-2 text-center">고2 GPA</th>
                                                                <th className="py-2 text-center">고3 GPA</th>
                                                            </tr>
                                                        </thead>
                                                        <tbody>
                                                            {roomUsersData
                                                                .filter(m => m.joinedRoomId === activeRoom.id)
                                                                .map((member, idx) => (
                                                                    <tr key={idx} className="border-b text-xs">
                                                                        <td className="py-3 font-bold">{member.customId}</td>
                                                                        <td className="py-3 text-center text-blue-600">{member.gpa1 ? member.gpa1.toFixed(2) : '-'}</td>
                                                                        <td className="py-3 text-center text-blue-600">{member.gpa2 ? member.gpa2.toFixed(2) : '-'}</td>
                                                                        <td className="py-3 text-center text-blue-600">{member.gpa3 ? member.gpa3.toFixed(2) : '-'}</td>
                                                                    </tr>
                                                                ))}
                                                        </tbody>
                                                    </table>
                                                </div>
                                            </div>

                                            <div className="lg:col-span-2 p-6 rounded-3xl border bg-white dark:bg-slate-900 shadow-sm flex flex-col justify-between h-[450px]">
                                                <div className="overflow-y-auto space-y-3.5 flex-1 mb-4">
                                                    {chatMessages.map(msg => {
                                                        const isMe = msg.senderId === user?.uid;
                                                        return (
                                                            <div key={msg.id} className={`flex flex-col max-w-[85%] ${isMe ? 'ml-auto items-end' : 'mr-auto items-start'}`}>
                                                                <span className="text-[9px] text-slate-400 mb-0.5">{msg.senderName}</span>
                                                                <div className={`p-2.5 rounded-xl text-xs ${isMe ? 'bg-blue-600 text-white' : 'bg-slate-100 dark:bg-slate-800'}`}>
                                                                    {msg.text}
                                                                </div>
                                                            </div>
                                                        );
                                                    })}
                                                    <div ref={chatEndRef} />
                                                </div>
                                                <form onSubmit={handleSendChatMessage} className="flex gap-2">
                                                    <input 
                                                        type="text" 
                                                        value={inputChat} 
                                                        onChange={e => setInputChat(e.target.value)} 
                                                        placeholder="토론 메시지 기입" 
                                                        className="flex-1 px-3 py-2 border rounded-xl text-xs bg-slate-50 dark:bg-slate-800" 
                                                    />
                                                    <button type="submit" className="p-2 bg-blue-600 text-white rounded-xl"><SvgIcon name="send" className="w-4 h-4" /></button>
                                                </form>
                                            </div>
                                        </div>
                                    </div>
                                )}
                            </div>
                        )}

                        <footer className="mt-16 pt-8 pb-4 border-t border-slate-200/20 text-center">
                            <p className="text-[10px] md:text-[11px] font-black tracking-[0.25em] text-slate-400 uppercase">
                                용남고등학교 CATCH POINT
                            </p>
                        </footer>
                    </main>

                    {/* 설정창 */}
                    {isSettingsOpen && (
                        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm animate-in fade-in duration-200">
                            <div className="rounded-2xl border p-6 max-w-md w-full bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 text-slate-900 dark:text-white shadow-2xl">
                                <div className="flex items-center justify-between pb-4 border-b">
                                    <h3 className="text-base font-extrabold flex items-center gap-2">
                                        <SvgIcon name="settings" className="w-5 h-5 text-blue-500" />
                                        기록 매니저 프로필 및 설정
                                    </h3>
                                    <button onClick={() => setIsSettingsOpen(false)}><SvgIcon name="x" className="w-4.5 h-4.5" /></button>
                                </div>

                                <div className="py-6 space-y-6">
                                    <div className="space-y-2">
                                        <h4 className="text-sm font-bold">내 아이디(닉네임)</h4>
                                        <input 
                                            type="text" 
                                            value={customId} 
                                            onChange={e => { setCustomId(e.target.value); saveProfileSettings(e.target.value, shareGrades); }}
                                            className="w-full px-3 py-2 border rounded-xl text-xs bg-slate-50 dark:bg-slate-800" 
                                        />
                                    </div>

                                    <div className="flex items-center justify-between pb-2 border-b">
                                        <div>
                                            <h4 className="text-sm font-bold">그룹 내에 나의 내신성적 실시간 공개</h4>
                                        </div>
                                        <button 
                                            onClick={() => { const toggled = !shareGrades; setShareGrades(toggled); saveProfileSettings(customId, toggled); }}
                                            className={`relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${shareGrades ? 'bg-green-600' : 'bg-slate-300'}`}
                                        >
                                            <span className={`inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${shareGrades ? 'translate-x-6' : 'translate-x-1'}`} />
                                        </button>
                                    </div>

                                    <div className="flex items-center justify-between">
                                        <h4 className="text-sm font-bold">다크 모드 적용</h4>
                                        <button 
                                            onClick={() => setIsDarkMode(!isDarkMode)}
                                            className={`relative inline-flex h-6 w-11 items-center rounded-full transition-colors ${isDarkMode ? 'bg-blue-600' : 'bg-slate-300'}`}
                                        >
                                            <span className={`inline-block h-4 w-4 transform rounded-full bg-white transition-transform ${isDarkMode ? 'translate-x-6' : 'translate-x-1'}`} />
                                        </button>
                                    </div>

                                    <div className="space-y-2">
                                        <h4 className="text-sm font-bold">나이스 글자 수 연산 규칙</h4>
                                        <div className="grid grid-cols-2 gap-2 p-1 rounded-xl bg-slate-100 dark:bg-slate-800">
                                            <button onClick={() => setByteCalcMode('char')} className={`py-1.5 text-xs rounded-lg ${byteCalcMode === 'char' ? 'bg-white dark:bg-slate-750 text-blue-600 shadow-sm' : 'text-slate-400'}`}>일반 글자수</button>
                                            <button onClick={() => setByteCalcMode('byte')} className={`py-1.5 text-xs rounded-lg ${byteCalcMode === 'byte' ? 'bg-white dark:bg-slate-750 text-blue-600 shadow-sm' : 'text-slate-400'}`}>나이스 바이트</button>
                                        </div>
                                    </div>
                                </div>

                                <div className="pt-4 border-t flex justify-end">
                                    <button onClick={() => setIsSettingsOpen(false)} className="px-5 py-2 bg-blue-600 text-white rounded-xl text-xs font-bold">설정 완료</button>
                                </div>
                            </div>
                        </div>
                    )}

                    {/* 비밀번호 입력 창 */}
                    {passwordModal.show && (
                        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm animate-in fade-in duration-200">
                            <div className="rounded-2xl border p-6 max-w-sm w-full bg-white dark:bg-slate-900 border-slate-150 dark:border-slate-800 text-slate-900 dark:text-white shadow-2xl">
                                <div className="flex items-center gap-2 pb-3 border-b border-slate-100">
                                    <SvgIcon name="lock-keyhole" className="w-5 h-5 text-indigo-500 shrink-0" />
                                    <h4 className="text-sm font-black">비밀번호 기입</h4>
                                </div>
                                <p className="text-xs text-slate-400 mt-3 font-semibold">
                                    [ <span className="text-indigo-500 font-bold">{passwordModal.targetRoom?.name}</span> ] 에 들어가기 위한 비밀번호를 입력해주세요.
                                </p>
                                <div className="mt-4">
                                    <input 
                                        type="password"
                                        value={passwordModal.typedPassword}
                                        onChange={(e) => setPasswordModal(prev => ({ ...prev, typedPassword: e.target.value }))}
                                        placeholder="비밀번호 입력"
                                        className="w-full px-3 py-2 border rounded-xl text-xs bg-slate-50 dark:bg-slate-800"
                                        autoFocus
                                    />
                                </div>
                                <div className="flex gap-2 mt-5">
                                    <button 
                                        onClick={() => {
                                            handleJoinRoom(passwordModal.targetRoom, passwordModal.typedPassword);
                                            setPasswordModal({ show: false, targetRoom: null, typedPassword: '' });
                                        }}
                                        disabled={!passwordModal.typedPassword.trim()}
                                        className="flex-1 py-2 bg-indigo-600 text-white rounded-lg text-xs font-bold disabled:opacity-50"
                                    >
                                        입장하기
                                    </button>
                                    <button onClick={() => setPasswordModal({ show: false, targetRoom: null, typedPassword: '' })} className="flex-1 py-2 bg-slate-100 dark:bg-slate-800 text-slate-400 rounded-lg text-xs font-bold">취소</button>
                                </div>
                            </div>
                        </div>
                    )}

                    {/* 커스텀 얼럿창 */}
                    {modal.show && (
                        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/40 backdrop-blur-sm animate-in fade-in duration-200">
                            <div className="rounded-2xl shadow-xl border p-6 max-w-sm w-full bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-800 text-slate-900 dark:text-white">
                                <h4 className="text-sm font-extrabold flex items-center gap-1.5">
                                    <SvgIcon name="alert-circle" className="w-4 h-4 text-red-500" />
                                    {modal.title}
                                </h4>
                                <p className="text-xs mt-2.5 font-semibold text-slate-500">{modal.message}</p>
                                <div className="flex gap-2 mt-5">
                                    <button onClick={modal.onConfirm} className="flex-1 py-2 bg-red-500 text-white rounded-lg font-bold text-xs">확인</button>
                                    <button onClick={() => setModal({ show: false, title: '', message: '', onConfirm: null })} className="flex-1 py-2 rounded-lg font-bold text-xs bg-slate-100 dark:bg-slate-800 text-slate-400">취소</button>
                                </div>
                            </div>
                        </div>
                    )}
                </div>
            );
        }

        // React DOM 렌더링 시작
        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
