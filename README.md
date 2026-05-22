```html
<!DOCTYPE html>
<html lang="ko" class="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>생기부 & 성적 포트폴리오 매니저</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          fontFamily: {
            sans: ['Pretendard', '-apple-system', 'BlinkMacSystemFont', 'system-ui', 'Roboto', 'sans-serif'],
          }
        }
      }
    }
  </script>
  <!-- Lucide Icons CDN -->
  <script src="https://unpkg.com/lucide@latest"></script>
  <style>
    /* 스크롤바 커스텀 */
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: transparent;
    }
    ::-webkit-scrollbar-thumb {
      background: #cbd5e1;
      border-radius: 3px;
    }
    .dark ::-webkit-scrollbar-thumb {
      background: #475569;
    }
  </style>
</head>
<body class="bg-slate-50 text-slate-900 dark:bg-slate-950 dark:text-slate-100 min-h-screen transition-colors duration-200">

  <!-- ==========================================
       [GLOBAL MODAL ALERTS]
       ========================================== -->
  <div id="custom-modal" class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4 hidden">
    <div class="w-full max-w-sm rounded-xl bg-white p-6 shadow-2xl dark:bg-slate-900 border border-slate-100 dark:border-slate-800">
      <h3 id="modal-title" class="text-base font-bold text-slate-800 dark:text-white"></h3>
      <p id="modal-message" class="text-xs text-slate-500 dark:text-slate-400 mt-2 leading-relaxed"></p>
      <div class="mt-5 flex justify-end gap-2">
        <button id="modal-cancel-btn" class="rounded-lg bg-slate-100 dark:bg-slate-800 px-4 py-2 text-xs font-medium hover:bg-slate-200 transition">취소</button>
        <button id="modal-confirm-btn" class="rounded-lg bg-red-600 text-white px-4 py-2 text-xs font-semibold hover:bg-red-700 transition">확인</button>
      </div>
    </div>
  </div>

  <!-- ==========================================
       [AUTH SECTION - LOGIN / SIGNUP / RESET]
       ========================================== -->
  <div id="auth-section" class="flex min-h-screen items-center justify-center p-4">
    <div class="w-full max-w-md rounded-2xl bg-white p-8 shadow-xl dark:bg-slate-900 border border-slate-100 dark:border-slate-800">
      <div class="flex flex-col items-center mb-6">
        <div class="flex h-12 w-12 items-center justify-center rounded-xl bg-blue-100 dark:bg-blue-900/40 text-blue-600 dark:text-blue-400 mb-3">
          <i data-lucide="graduation-cap" class="h-7 w-7"></i>
        </div>
        <h2 id="auth-title" class="text-2xl font-extrabold text-slate-800 dark:text-white">학생부 & 성적 통합 매니저</h2>
        <p id="auth-subtitle" class="text-sm text-slate-500 dark:text-slate-400 mt-1 text-center">스마트한 대입 생기부와 성적 추이 분석 관리</p>
      </div>

      <!-- 에러/성공 피드백 메시지 바 -->
      <div id="auth-alert" class="flex items-start gap-2 rounded-lg p-3 text-xs mb-4 hidden">
        <i data-lucide="alert-circle" class="h-4 w-4 shrink-0 mt-0.5"></i>
        <span id="auth-alert-text"></span>
      </div>

      <form id="auth-form" class="space-y-4">
        <div>
          <label class="block text-xs font-semibold text-slate-600 dark:text-slate-400 mb-1">이메일 주소</label>
          <div class="relative">
            <i data-lucide="mail" class="absolute left-3 top-2.5 h-4 w-4 text-slate-400"></i>
            <input type="text" id="auth-email" placeholder="name@school.com (또는 'admin')" class="w-full rounded-lg border border-slate-200 bg-white py-2 pl-10 pr-4 text-sm outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-900 dark:text-white" required>
          </div>
        </div>

        <div id="password-field-wrapper">
          <label class="block text-xs font-semibold text-slate-600 dark:text-slate-400 mb-1">비밀번호</label>
          <div class="relative">
            <i data-lucide="lock" class="absolute left-3 top-2.5 h-4 w-4 text-slate-400"></i>
            <input type="password" id="auth-password" placeholder="최소 6자 이상" class="w-full rounded-lg border border-slate-200 bg-white py-2 pl-10 pr-10 text-sm outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-900 dark:text-white">
            <button type="button" id="toggle-password-visibility" class="absolute right-3 top-2.5 text-slate-400 hover:text-slate-600">
              <i data-lucide="eye" class="h-4 w-4"></i>
            </button>
          </div>
        </div>

        <div id="remember-me-wrapper" class="flex items-center justify-between">
          <label class="flex items-center gap-2 cursor-pointer">
            <input type="checkbox" id="auth-remember" class="h-4 w-4 rounded text-blue-600 border-slate-300 focus:ring-blue-500">
            <span class="text-xs text-slate-500 dark:text-slate-400 select-none">자동 로그인 및 정보 저장</span>
          </label>
          <button type="button" id="go-reset-btn" class="text-xs text-blue-500 hover:underline">암호 분실?</button>
        </div>

        <button type="submit" id="submit-auth-btn" class="w-full rounded-lg bg-blue-600 py-2.5 text-sm font-semibold text-white hover:bg-blue-700 transition shadow-md shadow-blue-500/10">로그인</button>
      </form>

      <div id="demo-auth-divider" class="relative my-5">
        <div class="absolute inset-0 flex items-center"><div class="w-full border-t border-slate-200 dark:border-slate-800"></div></div>
        <div class="relative flex justify-center text-xs"><span class="bg-white px-2 text-slate-400 dark:bg-slate-900">또는 편리하게 시작</span></div>
      </div>

      <button id="demo-login-btn" class="w-full flex items-center justify-center gap-2 rounded-lg border border-slate-200 bg-slate-50 dark:bg-slate-900 dark:border-slate-800 py-2.5 text-xs font-bold text-slate-700 dark:text-slate-300 hover:bg-slate-100 transition">
        <i data-lucide="sparkles" class="h-4 w-4 text-amber-500"></i>
        체험용 데모 계정으로 즉시 로그인
      </button>

      <div class="mt-6 text-center text-xs">
        <p id="switch-view-text" class="text-slate-500 dark:text-slate-400">
          아직 계정이 없으신가요? <button id="switch-auth-mode" class="text-blue-500 font-bold hover:underline">회원가입</button>
        </p>
      </div>
    </div>
  </div>

  <!-- ==========================================
       [MAIN APPLICATION DASHBOARD]
       ========================================== -->
  <div id="main-dashboard" class="hidden">
    <!-- GLOBAL HEADER -->
    <header class="sticky top-0 z-40 border-b border-slate-200/80 bg-white/95 px-6 py-4 dark:border-slate-800/80 dark:bg-slate-900/95 backdrop-blur-md">
      <div class="mx-auto flex max-w-7xl items-center justify-between">
        <div class="flex items-center gap-3">
          <div class="flex h-10 w-10 items-center justify-center rounded-xl bg-gradient-to-tr from-blue-600 to-indigo-500 text-white shadow-md shadow-blue-500/20">
            <i data-lucide="brain-circuit" class="h-6 w-6 animate-pulse"></i>
          </div>
          <div>
            <h1 class="text-base font-extrabold text-slate-900 dark:text-white">생기부 & 성적 포트폴리오</h1>
            <p class="text-[10px] text-slate-500 dark:text-slate-400">2022 개정 고교 상시 피드백 설계 시스템</p>
          </div>
        </div>

        <div class="flex items-center gap-2 sm:gap-4">
          <!-- 학년 선택 필터 -->
          <div class="flex rounded-lg bg-slate-100 p-1 dark:bg-slate-800 border border-slate-200/50 dark:border-slate-700/50">
            <button onclick="setGlobalGrade('1')" id="grade-btn-1" class="rounded-md px-3 py-1 text-xs font-bold transition-all">1학년</button>
            <button onclick="setGlobalGrade('2')" id="grade-btn-2" class="rounded-md px-3 py-1 text-xs font-bold transition-all">2학년</button>
            <button onclick="setGlobalGrade('3')" id="grade-btn-3" class="rounded-md px-3 py-1 text-xs font-bold transition-all">3학년</button>
          </div>

          <!-- 다크모드/라이트모드 토글 -->
          <button id="theme-toggle-btn" class="h-8 w-8 flex items-center justify-center rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-600 dark:bg-slate-800 dark:hover:bg-slate-700 dark:text-slate-300 transition">
            <i data-lucide="moon" class="h-4 w-4" id="theme-icon"></i>
          </button>

          <!-- 설정창 버튼 -->
          <button id="settings-open-btn" class="h-8 w-8 flex items-center justify-center rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-600 dark:bg-slate-800 dark:hover:bg-slate-700 dark:text-slate-300 transition">
            <i data-lucide="settings" class="h-4 w-4"></i>
          </button>

          <!-- 로그아웃 -->
          <button id="logout-btn" class="flex items-center gap-1.5 rounded-lg border border-red-100 bg-red-50 dark:bg-red-950/20 dark:border-red-900/30 px-3 py-1.5 text-xs font-bold text-red-600 dark:text-red-400 hover:bg-red-100 dark:hover:bg-red-900/40 transition">
            <i data-lucide="log-out" class="h-3.5 w-3.5"></i>
            <span class="hidden sm:inline">로그아웃</span>
          </button>
        </div>
      </div>
    </header>

    <!-- SETTINGS SLIDE DRAWER -->
    <div id="settings-drawer" class="fixed inset-0 z-50 flex justify-end bg-black/40 backdrop-blur-sm hidden">
      <div class="w-full max-w-sm bg-white p-6 shadow-2xl dark:bg-slate-900 flex flex-col justify-between border-l border-slate-100 dark:border-slate-800">
        <div>
          <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-4 mb-5">
            <div class="flex items-center gap-2">
              <i data-lucide="user-check" class="h-5 w-5 text-blue-500"></i>
              <h3 class="font-bold text-slate-800 dark:text-white">프로필 & 정보 공개 옵션</h3>
            </div>
            <button id="settings-close-btn" class="text-slate-400 hover:text-slate-600"><i data-lucide="x" class="h-5 w-5"></i></button>
          </div>

          <div class="space-y-4">
            <div>
              <label class="block text-xs font-bold text-slate-600 dark:text-slate-400 mb-1">식별용 학생 ID 설정</label>
              <input type="text" id="custom-id-input" placeholder="예: 홍길동 (입력시 자동저장)" class="w-full rounded-lg border border-slate-200 bg-white p-2.5 text-xs outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-900 dark:text-white">
              <p class="text-[10px] text-slate-400 mt-1">그룹 멤버들과 대화 시 표시되는 필명 또는 본명입니다.</p>
            </div>

            <div class="rounded-xl border border-slate-100 dark:border-slate-800 p-4 bg-slate-50 dark:bg-slate-950">
              <label class="flex items-start gap-3 cursor-pointer">
                <input type="checkbox" id="share-grades-checkbox" class="mt-1 h-4 w-4 text-blue-600 border-slate-300 focus:ring-blue-500">
                <div>
                  <span class="text-xs font-bold text-slate-800 dark:text-slate-200 block">그룹 내 종합 내신정보 공개</span>
                  <span class="text-[10px] text-slate-400 leading-normal block mt-1">동의 시 그룹 멤버 간 학업 성취 추세 통계에 가명화된 본인의 학년별 내신 평점 정보가 공유되어 서로 비교 분석할 수 있습니다.</span>
                </div>
              </label>
            </div>
          </div>
        </div>

        <button id="settings-save-btn" class="w-full rounded-lg bg-blue-600 text-white font-bold py-2 text-xs hover:bg-blue-700 transition">닫기</button>
      </div>
    </div>

    <!-- SUB-DASHBOARD BAR -->
    <div class="bg-white border-b border-slate-200/60 dark:bg-slate-900 dark:border-slate-800">
      <div class="mx-auto flex max-w-7xl flex-wrap items-center justify-between px-6 py-3">
        <div class="flex items-center gap-2 text-xs">
          <span class="font-semibold text-slate-500 dark:text-slate-400">진로 희망학과 설정:</span>
          <input type="text" id="target-dept-input" placeholder="예: 컴퓨터공학과" class="rounded-lg border border-slate-200 bg-slate-50 px-3 py-1 text-xs font-bold text-blue-600 dark:text-blue-400 focus:border-blue-500 dark:border-slate-800 dark:bg-slate-950 outline-none max-w-[150px]">
        </div>

        <div class="flex items-center gap-3 mt-2 sm:mt-0">
          <div class="flex items-center gap-1.5 text-xs text-slate-500 dark:text-slate-400">
            <i data-lucide="clipboard-check" class="h-4 w-4 text-green-500"></i>
            <span>실시간 9등급제 내신평점:</span>
            <strong id="gpa9-badge" class="text-blue-600 dark:text-blue-400 font-extrabold">-</strong>
          </div>
          <div class="h-3 w-[1px] bg-slate-200 dark:bg-slate-800"></div>
          <div class="flex items-center gap-1.5 text-xs text-slate-500 dark:text-slate-400">
            <i data-lucide="award" class="h-4 w-4 text-purple-500"></i>
            <span>5등급제 내신평점:</span>
            <strong id="gpa5-badge" class="text-purple-600 dark:text-purple-400 font-extrabold">-</strong>
          </div>
        </div>
      </div>
    </div>

    <!-- MAIN BODY CONTAINER -->
    <div class="mx-auto max-w-7xl px-4 py-6 sm:px-6">
      <div class="grid grid-cols-1 gap-6 lg:grid-cols-4">
        
        <!-- SIDEBAR NAVIGATION -->
        <div class="lg:col-span-1 space-y-4">
          <div class="rounded-2xl border border-slate-200/70 bg-white p-4 dark:border-slate-800 dark:bg-slate-900 shadow-sm">
            <div class="mb-4">
              <span class="text-[10px] font-bold text-slate-400 uppercase tracking-widest block">기능 카테고리</span>
              <h4 class="text-xs font-extrabold text-slate-800 dark:text-slate-200 mt-0.5">대시보드 주요 메뉴</h4>
            </div>

            <div class="space-y-1">
              <button onclick="switchTab('record')" id="tab-btn-record" class="flex w-full items-center justify-between rounded-xl px-4 py-3 text-xs font-bold transition">
                <div class="flex items-center gap-2">
                  <i data-lucide="file-text" class="h-4 w-4"></i>
                  <span>생기부 활동 기록</span>
                </div>
                <i data-lucide="chevron-right" class="h-3 w-3"></i>
              </button>

              <button onclick="switchTab('grades')" id="tab-btn-grades" class="flex w-full items-center justify-between rounded-xl px-4 py-3 text-xs font-bold transition">
                <div class="flex items-center gap-2">
                  <i data-lucide="bar-chart-3" class="h-4 w-4"></i>
                  <span>내신 성적 및 추세 관리</span>
                </div>
                <i data-lucide="chevron-right" class="h-3 w-3"></i>
              </button>

              <button onclick="switchTab('feedback')" id="tab-btn-feedback" class="flex w-full items-center justify-between rounded-xl px-4 py-3 text-xs font-bold transition">
                <div class="flex items-center gap-2">
                  <i data-lucide="spell-check" class="h-4 w-4"></i>
                  <span>맞춤법 & 특수문자 교정</span>
                </div>
                <i data-lucide="chevron-right" class="h-3 w-3"></i>
              </button>

              <button onclick="switchTab('group')" id="tab-btn-group" class="flex w-full items-center justify-between rounded-xl px-4 py-3 text-xs font-bold transition">
                <div class="flex items-center gap-2">
                  <i data-lucide="users" class="h-4 w-4"></i>
                  <span>목표 고교 커뮤니티</span>
                </div>
                <i data-lucide="chevron-right" class="h-3 w-3"></i>
              </button>
            </div>
          </div>

          <!-- QUICK AI STATUS CARD -->
          <div class="rounded-2xl border border-blue-100 bg-gradient-to-tr from-blue-50/50 to-indigo-50/30 p-4 dark:border-blue-900/20 dark:from-blue-950/20 dark:to-transparent shadow-sm">
            <div class="flex items-center gap-2 mb-2">
              <i data-lucide="sparkles" class="h-4 w-4 text-amber-500"></i>
              <h5 class="text-xs font-extrabold text-blue-950 dark:text-blue-300">AI 컨설턴트 한마디</h5>
            </div>
            <p class="text-[11px] text-blue-900/80 dark:text-blue-400 leading-relaxed font-medium">
              목표 학과 <strong id="sidebar-dept-text">"컴퓨터공학과"</strong>와 관련된 연계 심화 탐구는 선택과목과 자율, 진로 세특 간의 긴밀성에서 드러납니다. 학생부 기록 탭에서 분석이 필요한 항목의 <strong>[AI 맞춤 피드백]</strong> 버튼을 이용해 보세요.
            </p>
          </div>

          <!-- SAVING STATUS CONTROL FLOATER -->
          <div class="rounded-2xl border border-slate-200 bg-white p-4 dark:border-slate-800 dark:bg-slate-900 shadow-sm flex flex-col gap-3">
            <div class="flex items-center justify-between">
              <span class="text-[10px] font-bold text-slate-400 uppercase tracking-wider">서버 동기화</span>
              <span class="text-[10px] font-bold text-emerald-500 flex items-center gap-1">
                <span class="h-2 w-2 rounded-full bg-emerald-500 animate-ping"></span>
                Connected
              </span>
            </div>
            
            <button id="save-cloud-btn" class="w-full flex items-center justify-center gap-2 rounded-xl bg-blue-600 text-white hover:bg-blue-700 font-bold py-2.5 text-xs shadow-md shadow-blue-500/15 transition">
              <i data-lucide="save" class="h-3.5 w-3.5" id="save-btn-icon"></i>
              <span id="save-btn-text">현재 학년 데이터 저장</span>
            </button>
            <p id="save-status-msg" class="text-[10px] text-center font-bold hidden"></p>
          </div>
        </div>

        <!-- CONTENT AREA (TAB SWITCHABLE) -->
        <div class="lg:col-span-3 space-y-6">

          <!-- ==========================================
               [TAB: RECORD]
               ========================================== -->
          <div id="view-record" class="tab-view space-y-6 hidden">
            <!-- 고정 기입 항목 -->
            <div class="rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm">
              <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-4 mb-5">
                <div class="flex items-center gap-2">
                  <i data-lucide="file-text" class="h-5 w-5 text-blue-500"></i>
                  <h3 class="font-extrabold text-slate-900 dark:text-white"><span class="grade-label">1</span>학년 공통 기재활동 기록</h3>
                </div>

                <div class="flex items-center gap-2">
                  <span class="text-xs font-semibold text-slate-500">글자수 계산 단위:</span>
                  <select id="byte-calc-select" class="rounded-lg border border-slate-200 bg-slate-50 p-1 text-xs font-bold text-slate-700 outline-none dark:border-slate-800 dark:bg-slate-950 dark:text-slate-300">
                    <option value="char">한글/영문 모두 1자</option>
                    <option value="byte">NEIS 기준 바이트 (한글3byte)</option>
                  </select>
                </div>
              </div>

              <!-- 고정 기재 항목 동적 주입 -->
              <div id="fixed-fields-container" class="space-y-6"></div>
            </div>

            <!-- 교과선택 과목 세특 -->
            <div class="rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm">
              <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-4 mb-5">
                <div class="flex items-center gap-2">
                  <i data-lucide="book-open" class="h-5 w-5 text-blue-500"></i>
                  <div>
                    <h3 class="font-extrabold text-slate-900 dark:text-white"><span class="grade-label">1</span>학년 교과선택 과목 세특</h3>
                    <p class="text-[10px] text-slate-400 mt-0.5">대입 핵심 전공 세부능력 및 특기사항 연계</p>
                  </div>
                </div>

                <button id="add-subject-toggle-btn" class="flex items-center gap-1 rounded-lg bg-blue-600 px-3 py-1.5 text-xs font-bold text-white hover:bg-blue-700 transition">
                  <i data-lucide="plus" class="h-3.5 w-3.5"></i>
                  교과 과목 추가
                </button>
              </div>

              <!-- 과목 추가 영역 (Toggled) -->
              <div id="add-subject-wrapper" class="rounded-xl border border-dashed border-slate-200 bg-slate-50 p-4 dark:border-slate-800 dark:bg-slate-950 mb-5 space-y-3 hidden">
                <div class="flex items-center justify-between">
                  <span class="text-xs font-bold">새 교과 과목 개설</span>
                  <button id="add-subject-close-btn" class="text-slate-400 hover:text-slate-600"><i data-lucide="x" class="h-4 w-4"></i></button>
                </div>

                <div id="recommend-subject-pool" class="hidden">
                  <span class="text-[10px] text-slate-400 block mb-1.5">추천 선택교과 목록:</span>
                  <div id="recommend-subject-list" class="flex flex-wrap gap-1.5"></div>
                </div>

                <div class="flex items-center gap-2">
                  <input type="text" id="custom-subject-input" placeholder="수동 입력 (예: 고급 물리)" class="flex-1 rounded-lg border border-slate-200 bg-white p-2 text-xs outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-900">
                  <button id="add-subject-btn" class="rounded-lg bg-slate-800 text-white dark:bg-slate-700 hover:bg-slate-900 px-4 py-2 text-xs font-bold transition">개설하기</button>
                </div>
              </div>

              <!-- 과목별 세특 텍스트에리어 컨테이너 -->
              <div id="subjects-container" class="space-y-6"></div>
            </div>

            <!-- AI 피드백 뷰포트 -->
            <div id="ai-feedback-container" class="rounded-2xl border border-indigo-100 bg-indigo-50/20 p-6 dark:border-indigo-900/30 dark:bg-indigo-950/10 shadow-sm space-y-4 hidden">
              <div class="flex items-center justify-between border-b border-indigo-100 dark:border-indigo-900/40 pb-3">
                <div class="flex items-center gap-2">
                  <i data-lucide="brain-circuit" class="h-5 w-5 text-indigo-500"></i>
                  <h4 class="font-extrabold text-indigo-900 dark:text-indigo-300">AI 컨설팅 결과 보고서</h4>
                </div>
                <button id="ai-feedback-close-btn" class="text-slate-400 hover:text-slate-600"><i data-lucide="x" class="h-4 w-4"></i></button>
              </div>

              <div id="ai-feedback-loading" class="flex flex-col items-center justify-center py-6 gap-2 hidden">
                <i data-lucide="refresh-cw" class="h-6 w-6 animate-spin text-indigo-500"></i>
                <span class="text-xs text-indigo-600 dark:text-indigo-400 font-bold">대입 컨설팅 AI 분석 중... 최대 15초가 소요될 수 있습니다.</span>
              </div>
              <div id="ai-feedback-content" class="text-xs text-slate-700 dark:text-slate-300 leading-relaxed whitespace-pre-wrap font-medium"></div>
            </div>
          </div>

          <!-- ==========================================
               [TAB: GRADES]
               ========================================== -->
          <div id="view-grades" class="tab-view space-y-6 hidden">
            <!-- 내신성적 수동기입 테이블 -->
            <div class="rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm">
              <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-4 mb-5">
                <div class="flex items-center gap-2">
                  <i data-lucide="bar-chart-3" class="h-5 w-5 text-blue-500"></i>
                  <h3 class="font-extrabold text-slate-900 dark:text-white"><span class="grade-label">1</span>학년 학기별 석차 등급 환산</h3>
                </div>
                <span class="text-[10px] text-slate-400">교양/예체능 등 석차 미산출 교과는 평점계산에서 자동 배제됩니다.</span>
              </div>

              <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse min-w-[600px]">
                  <thead>
                    <tr class="border-b border-slate-100 dark:border-slate-800 text-xs text-slate-400 font-bold">
                      <th class="pb-3 w-[25%]">과목명</th>
                      <th class="pb-3 w-[15%]">단위수</th>
                      <th class="pb-3 w-[15%]">석차 등수</th>
                      <th class="pb-3 w-[15%]">동석차 수</th>
                      <th class="pb-3 w-[15%]">수강자 수</th>
                      <th class="pb-3 text-right w-[15%]">예측 등급</th>
                    </tr>
                  </thead>
                  <tbody id="grades-table-body" class="divide-y divide-slate-50 dark:divide-slate-800/50 text-xs"></tbody>
                </table>
              </div>
            </div>

            <!-- 시각화 차트 및 트렌드 정성평가 분석 -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
              <!-- Canvas 차트 -->
              <div class="md:col-span-2 rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm flex flex-col justify-between">
                <div class="mb-4">
                  <h4 class="font-extrabold text-xs text-slate-800 dark:text-slate-200">학년별 내신 성적 변동 변위선</h4>
                  <div class="flex items-center gap-4 mt-1">
                    <span class="flex items-center gap-1 text-[10px] text-slate-400">
                      <span class="h-1.5 w-3 bg-blue-500 rounded"></span> 9등급제 추이
                    </span>
                    <span class="flex items-center gap-1 text-[10px] text-slate-400">
                      <span class="h-1.5 w-3 bg-purple-500 rounded"></span> 5등급제 추이
                    </span>
                  </div>
                </div>

                <div class="relative w-full overflow-hidden flex justify-center bg-slate-50 dark:bg-slate-950 rounded-xl p-2 border border-slate-100 dark:border-slate-800">
                  <canvas id="grades-trend-canvas" class="max-w-full h-auto object-contain"></canvas>
                </div>
              </div>

              <!-- 정성분석 카드 -->
              <div class="md:col-span-1 rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm flex flex-col justify-between">
                <div>
                  <div class="flex items-center gap-1.5 border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                    <i data-lucide="activity" class="h-4.5 w-4.5 text-blue-500"></i>
                    <h4 class="font-extrabold text-xs text-slate-800 dark:text-slate-200">내신 추이 컨설팅 보고</h4>
                  </div>

                  <div class="space-y-4">
                    <div class="flex justify-center py-2" id="trend-icon-wrapper">
                      <!-- 동적 트렌드 아이콘 삽입 -->
                    </div>
                    <p id="trend-analysis-text" class="text-xs text-slate-600 dark:text-slate-300 leading-relaxed font-semibold"></p>
                  </div>
                </div>

                <div class="rounded-xl bg-slate-50 dark:bg-slate-950 p-3 mt-4 text-[10px] text-slate-400 leading-normal border border-slate-100 dark:border-slate-800">
                  정량적 원점수 분석 외에도 이수한 과목들의 표준편차, 세특 기재 완성도가 높아야 정성적 학생부 종합전형 1차 서류 통과 가능성을 높일 수 있습니다.
                </div>
              </div>
            </div>
          </div>

          <!-- ==========================================
               [TAB: FEEDBACK]
               ========================================== -->
          <div id="view-feedback" class="tab-view space-y-6 hidden">
            <div class="rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm">
              <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 pb-4 mb-5">
                <div class="flex items-center gap-2">
                  <i data-lucide="spell-check" class="h-5 w-5 text-blue-500"></i>
                  <div>
                    <h3 class="font-extrabold text-slate-900 dark:text-white">실시간 맞춤법 & NEIS 특수문자 검사</h3>
                    <p class="text-[10px] text-slate-400 mt-0.5">교육행정정보시스템 기재 불가 특수기호 필터 및 윤문 분석</p>
                  </div>
                </div>

                <button id="spellcheck-clear-btn" class="rounded-lg border border-slate-200 dark:border-slate-800 px-3 py-1 text-xs hover:bg-slate-50 dark:hover:bg-slate-950 text-slate-600 dark:text-slate-300 flex items-center gap-1 transition">
                  <i data-lucide="rotate-ccw" class="h-3 w-3"></i>
                  초기화
                </button>
              </div>

              <div class="space-y-4">
                <div>
                  <label class="block text-xs font-bold text-slate-600 dark:text-slate-400 mb-1">검사 대상 문장 기입</label>
                  <textarea id="spellcheck-textarea" rows="6" placeholder="이곳에 맞춤법 검사 및 NEIS 특수문자 오기입 여부를 진단하고 싶은 단락을 입력하세요." class="w-full rounded-xl border border-slate-200 bg-white p-3.5 text-xs leading-relaxed outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-950 dark:text-slate-100 transition resize-none"></textarea>
                </div>

                <button id="spellcheck-analyze-btn" class="w-full rounded-lg bg-blue-600 text-white hover:bg-blue-700 py-2 text-xs font-bold shadow transition">맞춤법 및 NEIS 기재 규격 정밀 진단</button>
              </div>
            </div>

            <!-- 분석 결과 피드백 창 -->
            <div id="spellcheck-results-wrapper" class="grid grid-cols-1 md:grid-cols-3 gap-6 hidden">
              <!-- NEIS 금지 문자 리포팅 -->
              <div class="md:col-span-1 rounded-2xl border border-red-100 bg-red-50/20 p-6 dark:border-red-950/20 dark:bg-red-950/5 space-y-4">
                <div class="flex items-center gap-1.5 border-b border-red-100 dark:border-red-900/20 pb-2">
                  <i data-lucide="alert-circle" class="h-4.5 w-4.5 text-red-500"></i>
                  <h4 class="font-extrabold text-xs text-red-800 dark:text-red-400">NEIS 기재규제 분석</h4>
                </div>

                <div id="neis-violation-status"></div>
                
                <p class="text-[9px] text-slate-400 leading-relaxed mt-2">
                  ★, ○, ▣, ◆ 등의 특수기호는 대입 전송용 학교생활기록부 입력 및 서류 평가 과정에서 전산 오류를 초래할 수 있어 입력이 금지됩니다.
                </p>
              </div>

              <!-- 맞춤법 및 AI 정밀 문체 수정 보고서 -->
              <div class="md:col-span-2 rounded-2xl border border-indigo-100 bg-indigo-50/20 p-6 dark:border-indigo-950/20 dark:bg-indigo-950/5 space-y-4">
                <div class="flex items-center gap-1.5 border-b border-indigo-100 dark:border-indigo-900/20 pb-2">
                  <i data-lucide="sparkles" class="h-4.5 w-4.5 text-indigo-500"></i>
                  <h4 class="font-extrabold text-xs text-indigo-800 dark:text-indigo-400">맞춤법 & 문체 교정 분석 리포트</h4>
                </div>

                <div id="spell-ai-output" class="text-xs text-slate-700 dark:text-slate-300 leading-relaxed whitespace-pre-wrap font-medium max-h-[350px] overflow-y-auto pr-2"></div>
              </div>
            </div>
          </div>

          <!-- ==========================================
               [TAB: GROUP/COMMUNITY]
               ========================================== -->
          <div id="view-group" class="tab-view space-y-6 hidden">
            <!-- 룸 목록 / 검색 / 생성 (룸 비활성화 시) -->
            <div id="group-search-wrapper" class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <!-- 룸 생성 폼 -->
              <div class="rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm flex flex-col justify-between">
                <div>
                  <div class="flex items-center gap-2 border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                    <i data-lucide="plus" class="h-5 w-5 text-blue-500"></i>
                    <h3 class="font-extrabold text-slate-900 dark:text-white">학습 & 입시 스터디룸 개설</h3>
                  </div>

                  <form id="create-room-form" class="space-y-3">
                    <div>
                      <label class="block text-[10px] font-bold text-slate-400 mb-1">학교명 또는 그룹 분류 (예: 서울고등학교)</label>
                      <input type="text" id="create-room-category" placeholder="분류할 대표 고교 또는 그룹 기입" class="w-full rounded-lg border border-slate-200 bg-white p-2 text-xs outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-950" required>
                    </div>

                    <div>
                      <label class="block text-[10px] font-bold text-slate-400 mb-1">스터디방 이름</label>
                      <input type="text" id="create-room-name" placeholder="예: 컴퓨터공학과 학생부 전형 준비단" class="w-full rounded-lg border border-slate-200 bg-white p-2 text-xs outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-950" required>
                    </div>

                    <div>
                      <label class="block text-[10px] font-bold text-slate-400 mb-1">방 암호 설정</label>
                      <input type="password" id="create-room-password" placeholder="비밀번호 설정" class="w-full rounded-lg border border-slate-200 bg-white p-2 text-xs outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-950" required>
                    </div>

                    <button type="submit" class="w-full rounded-lg bg-blue-600 text-white hover:bg-blue-700 py-2 text-xs font-bold shadow-md shadow-blue-500/10 transition">스터디방 개설하기</button>
                  </form>
                </div>
              </div>

              <!-- 룸 검색 리스트 -->
              <div class="rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm flex flex-col justify-between">
                <div>
                  <div class="flex items-center gap-2 border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                    <i data-lucide="search" class="h-5 w-5 text-blue-500"></i>
                    <h3 class="font-extrabold text-slate-900 dark:text-white">기존 스터디룸 검색 참여</h3>
                  </div>

                  <div class="flex gap-2 mb-4">
                    <input type="text" id="search-room-category" placeholder="개설된 학교명 입력" class="flex-1 rounded-lg border border-slate-200 bg-white p-2 text-xs outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-950">
                    <button id="search-room-btn" class="rounded-lg bg-slate-800 hover:bg-slate-950 dark:bg-slate-700 dark:hover:bg-slate-600 text-white px-4 py-2 text-xs font-bold transition">방 찾기</button>
                  </div>

                  <div id="room-search-loading" class="flex justify-center py-4 hidden">
                    <i data-lucide="refresh-cw" class="h-5 w-5 animate-spin text-blue-500"></i>
                  </div>

                  <div id="room-search-empty" class="text-xs text-slate-400 text-center py-4 hidden">해당 학교로 개설된 스터디방이 존재하지 않습니다.</div>

                  <div id="matching-rooms-container" class="space-y-2 max-h-[220px] overflow-y-auto pr-1"></div>
                </div>
              </div>
            </div>

            <!-- 액티브 룸 활성화 시 메신저 패널 -->
            <div id="active-chat-wrapper" class="grid grid-cols-1 md:grid-cols-3 gap-6 hidden">
              <!-- 실시간 대화창 -->
              <div class="md:col-span-2 rounded-2xl border border-slate-200/80 bg-white dark:border-slate-800 dark:bg-slate-900 shadow-sm flex flex-col h-[520px]">
                <div class="flex items-center justify-between border-b border-slate-100 dark:border-slate-800 p-4 bg-slate-50/50 dark:bg-slate-950/20 rounded-t-2xl">
                  <div class="flex items-center gap-2">
                    <i data-lucide="message-square" class="h-5 w-5 text-blue-500"></i>
                    <div>
                      <span id="active-room-category-badge" class="rounded bg-blue-100 text-blue-600 px-1.5 py-0.5 text-[8px] font-bold dark:bg-blue-950/40 dark:text-blue-400"></span>
                      <h3 id="active-room-name-text" class="font-extrabold text-xs text-slate-900 dark:text-white mt-1"></h3>
                    </div>
                  </div>

                  <button id="exit-chat-room-btn" class="rounded-lg border border-slate-200 dark:border-slate-800 px-3 py-1 text-xs hover:bg-slate-50 dark:hover:bg-slate-950 font-bold transition">방 나가기</button>
                </div>

                <!-- 메시지 스크롤 컨테이너 -->
                <div id="chat-messages-scroll-area" class="flex-1 overflow-y-auto p-4 space-y-3"></div>

                <!-- 채팅 입력창 -->
                <form id="chat-send-form" class="border-t border-slate-100 dark:border-slate-800 p-3 flex gap-2">
                  <input type="text" id="chat-message-input" placeholder="대화 메세지를 입력하세요." class="flex-1 rounded-lg border border-slate-200 bg-white px-3 py-1.5 text-xs outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-950" required>
                  <button type="submit" class="rounded-lg bg-blue-600 text-white px-4 py-1.5 text-xs font-bold hover:bg-blue-700 transition flex items-center gap-1">
                    <i data-lucide="send" class="h-3 w-3"></i>
                    전송
                  </button>
                </form>
              </div>

              <!-- 가입 멤버 등급 비교표 -->
              <div class="md:col-span-1 rounded-2xl border border-slate-200/80 bg-white p-6 dark:border-slate-800 dark:bg-slate-900 shadow-sm flex flex-col h-[520px] justify-between">
                <div>
                  <div class="flex items-center gap-1.5 border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                    <i data-lucide="bar-chart-3" class="h-4.5 w-4.5 text-blue-500"></i>
                    <h4 class="font-extrabold text-xs text-slate-800 dark:text-slate-200">성적 공유 동의자 3개년 등급</h4>
                  </div>

                  <div id="member-grades-scroll-area" class="space-y-3 max-h-[380px] overflow-y-auto pr-1"></div>
                </div>

                <p class="text-[9px] text-slate-400 leading-normal border-t border-slate-100 dark:border-slate-800 pt-3">
                  본인 설정에서 "종합 내신정보 공개"에 동의하셔야만 가명화 통계 데이터베이스에 공유되며 멤버들의 정보도 함께 조회할 수 있습니다.
                </p>
              </div>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>

  <!-- ==========================================
       [FIREBASE IMPLEMENTATION & CORE JAVASCRIPT]
       ========================================== -->
  <script type="module">
    import { initializeApp, getApps } from 'https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js';
    import { 
      getAuth, 
      createUserWithEmailAndPassword, 
      signInWithEmailAndPassword, 
      sendPasswordResetEmail, 
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

    // Firebase Configurations
    const firebaseConfig = {
      apiKey: "AIzaSyCNncvAHxoiL4HZE4ephrST0THGPmbLfnc",
      authDomain: "app123-9a3c9.firebaseapp.com",
      projectId: "app123-9a3c9",
      storageBucket: "app123-9a3c9.firebasestorage.app",
      messagingSenderId: "933256403505",
      appId: "1:933256403505:web:a2ad048181197cc6139310",
      measurementId: "G-LJ6GL7YWVX"
    };

    // Firebase Initializer
    const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0];
    const auth = getAuth(app);
    const db = getFirestore(app);
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'student-record-app';

    // 2022 개정 교육과정 고정 기입 레이아웃 상수
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
        tip: "담임 교사가 작성하는 부분인 만큼, 객관적이고 긍정적인 핵심 에피소드 위주로 초안을 정리합니다."
      }
    };

    // 학년별 기본 탑재 교과목 데이터베이스
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

    // 추가 가능한 선택 과목 리스트
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

    // GLOBAL APPLICATION STATE
    let state = {
      user: null,
      view: 'login', // login, signup, forgotPassword, main
      activeTab: 'record', // record, grades, feedback, group
      grade: '1',
      byteCalcMode: 'char',
      customId: '',
      shareGrades: false,
      rememberMe: true,
      selectedDept: '컴퓨터공학과',
      allGradesData: {
        '1': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' },
        '2': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' },
        '3': { fixed: {}, subjects: [], grades: {}, targetDept: '컴퓨터공학과' }
      },
      // 그룹/메신저 임시 보드
      activeRoom: null,
      matchingRooms: [],
      chatMessages: [],
      roomUsersData: []
    };

    // ==========================================
    // [STATE MUTATION & RENDER ENGINE]
    // ==========================================
    function getByteLength(str) {
      if (!str) return 0;
      let byteLength = 0;
      for (let i = 0; i < str.length; i++) {
        const code = str.charCodeAt(i);
        if (code === 10) byteLength += 2;
        else if (code > 127) byteLength += 3;
        else byteLength += 1;
      }
      return byteLength;
    }

    function calculateGradeFromRank(rank, total, jointCount = 1) {
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
    }

    function isLiberalArtSubject(name) {
      if (!name) return false;
      return EXCLUDED_LIBERAL_ARTS.some(art => name.toLowerCase().includes(art));
    }

    function calculateGPAByGrade(targetGrade) {
      const currentGrades = state.allGradesData[targetGrade]?.grades || {};
      const activeSubjects = state.allGradesData[targetGrade]?.subjects || [];

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
          const units = Number(subGrade.units) || 0;
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
    }

    function getGPAForBadge() {
      const data = calculateGPAByGrade(state.grade);
      return {
        average9: data.average9 !== null ? data.average9 : '-',
        average5: data.average5 !== null ? data.average5 : '-'
      };
    }

    // ==========================================
    // [UI STATE SYNCHRONIZATION (DOM)]
    // ==========================================
    window.setGlobalGrade = function(selectedGrade) {
      state.grade = selectedGrade;
      // UI 배지/선택 업데이트
      document.querySelectorAll('[id^="grade-btn-"]').forEach(btn => {
        btn.classList.remove('bg-white', 'text-blue-600', 'shadow-sm', 'dark:bg-slate-700', 'dark:text-blue-400');
        btn.classList.add('text-slate-500', 'dark:text-slate-400', 'hover:text-slate-800');
      });
      const activeBtn = document.getElementById(`grade-btn-${selectedGrade}`);
      activeBtn.classList.add('bg-white', 'text-blue-600', 'shadow-sm', 'dark:bg-slate-700', 'dark:text-blue-400');

      document.querySelectorAll('.grade-label').forEach(span => span.textContent = selectedGrade);
      
      const gradeData = state.allGradesData[selectedGrade];
      if (gradeData) {
        state.selectedDept = gradeData.targetDept || '컴퓨터공학과';
        document.getElementById('target-dept-input').value = state.selectedDept;
        document.getElementById('sidebar-dept-text').textContent = state.selectedDept;
      }

      updateGPABadges();
      renderFixedFields();
      renderSubjectsList();
      renderGradesTable();
      drawChart();
    };

    window.switchTab = function(targetTab) {
      state.activeTab = targetTab;
      document.querySelectorAll('[id^="tab-btn-"]').forEach(btn => {
        btn.classList.remove('bg-blue-600', 'text-white', 'shadow-md', 'shadow-blue-500/10');
        btn.classList.add('text-slate-600', 'dark:text-slate-300', 'hover:bg-slate-100', 'dark:hover:bg-slate-800/50');
      });
      const activeBtn = document.getElementById(`tab-btn-${targetTab}`);
      activeBtn.classList.add('bg-blue-600', 'text-white', 'shadow-md', 'shadow-blue-500/10');

      document.querySelectorAll('.tab-view').forEach(view => view.classList.add('hidden'));
      document.getElementById(`view-${targetTab}`).classList.remove('hidden');

      if (targetTab === 'grades') {
        renderGradesTable();
        setTimeout(drawChart, 100);
      }
    };

    function updateGPABadges() {
      const gpa = getGPAForBadge();
      document.getElementById('gpa9-badge').textContent = gpa.average9;
      document.getElementById('gpa5-badge').textContent = gpa.average5;
    }

    // ==========================================
    // [RENDER FIXED FIELDS]
    // ==========================================
    function renderFixedFields() {
      const container = document.getElementById('fixed-fields-container');
      container.innerHTML = '';
      
      const fixedData = state.allGradesData[state.grade]?.fixed || {};

      Object.entries(FIXED_STRUCTURE).forEach(([key, item]) => {
        const textVal = fixedData[key] || '';
        const count = state.byteCalcMode === 'char' ? textVal.length : getByteLength(textVal);
        
        // 기입 불가 또는 경고 클래스 계산
        let indicatorClass = 'text-blue-600 dark:text-blue-400';
        if (count > item.limit) indicatorClass = 'text-red-500 font-bold';
        else if (count >= item.limit - 50) indicatorClass = 'text-orange-500 font-medium';

        const rawHtml = `
          <div class="group relative border-b border-slate-100 dark:border-slate-800/80 pb-5 last:border-none last:pb-0">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-1 mb-2">
              <span class="text-xs font-extrabold text-slate-800 dark:text-slate-200 flex items-center gap-1">
                <span class="h-2 w-2 rounded-full bg-blue-500"></span>
                ${item.label}
              </span>
              <div class="flex items-center gap-3">
                <span class="text-[10px] text-slate-400">${item.tip}</span>
                <span class="text-[10px] ${indicatorClass}">
                  ${count} / ${item.limit} ${state.byteCalcMode === 'char' ? '자' : 'Byte'}
                </span>
              </div>
            </div>
            <textarea id="fixed-${key}" placeholder="${item.placeholder}" rows="4" class="w-full rounded-xl border border-slate-200 bg-white p-3.5 text-xs leading-relaxed outline-none focus:border-blue-500 focus:ring-1 focus:ring-blue-500 dark:border-slate-800 dark:bg-slate-950 dark:text-slate-100 transition resize-none">${textVal}</textarea>
            <div class="mt-2 flex justify-end gap-2">
              <button onclick="sendToSpellchecker('fixed-${key}')" class="rounded-lg border border-slate-200 dark:border-slate-800 hover:bg-slate-50 dark:hover:bg-slate-950 px-3 py-1.5 text-[10px] font-bold flex items-center gap-1 text-slate-600 dark:text-slate-300 transition">
                <i data-lucide="spell-check" class="h-3 w-3 text-emerald-500"></i>
                맞춤법 정밀검사 보내기
              </button>
              <button onclick="triggerAIFeedback('${key}')" class="rounded-lg bg-blue-50 hover:bg-blue-100/80 dark:bg-blue-950/30 dark:hover:bg-blue-950/50 px-3 py-1.5 text-[10px] font-bold text-blue-600 dark:text-blue-400 flex items-center gap-1 transition">
                <i data-lucide="sparkles" class="h-3 w-3 text-blue-500"></i>
                AI 사정관 분석 받기
              </button>
            </div>
          </div>
        `;
        container.insertAdjacentHTML('beforeend', rawHtml);

        // 이벤트 리스너 마운트
        const textarea = document.getElementById(`fixed-${key}`);
        textarea.addEventListener('input', (e) => {
          let val = e.target.value;
          if (val.length > item.limit && state.byteCalcMode === 'char') {
            val = val.substring(0, item.limit);
            e.target.value = val;
          }
          state.allGradesData[state.grade].fixed[key] = val;
          renderFixedFields(); // 카운터 갱신을 위한 지연 렌더
          // 포커스 복원
          const restoredEl = document.getElementById(`fixed-${key}`);
          restoredEl.focus();
          restoredEl.setSelectionRange(val.length, val.length);
        });
      });
      lucide.createIcons();
    }

    // ==========================================
    // [RENDER SUBJECTS LIST]
    // ==========================================
    function renderSubjectsList() {
      const container = document.getElementById('subjects-container');
      container.innerHTML = '';

      const subjects = state.allGradesData[state.grade]?.subjects || [];

      if (subjects.length === 0) {
        container.innerHTML = `<div class="py-8 text-center text-xs text-slate-400">등록된 과목이 없습니다. 과목 추가 버튼으로 과목을 개설해 주세요.</div>`;
        return;
      }

      subjects.forEach(sub => {
        const textVal = sub.content || '';
        const limit = sub.limit || 500;
        const count = state.byteCalcMode === 'char' ? textVal.length : getByteLength(textVal);

        let indicatorClass = 'text-blue-600 dark:text-blue-400';
        if (count > limit) indicatorClass = 'text-red-500 font-bold';
        else if (count >= limit - 50) indicatorClass = 'text-orange-500 font-medium';

        const isExLiberal = isLiberalArtSubject(sub.name);
        const badgeHtml = isExLiberal ? `<span class="rounded bg-orange-50 text-orange-600 dark:bg-orange-950/30 dark:text-orange-400 px-1.5 py-0.5 text-[9px] font-bold">석차미산출(교양/예체능)</span>` : '';

        const rawHtml = `
          <div class="border-b border-slate-100 dark:border-slate-800/80 pb-5 last:border-none last:pb-0">
            <div class="flex items-center justify-between mb-2">
              <span class="text-xs font-extrabold text-slate-800 dark:text-slate-200 flex items-center gap-1.5">
                <i data-lucide="book-marked" class="h-4 w-4 text-slate-500"></i>
                ${sub.name}
                ${badgeHtml}
              </span>
              <div class="flex items-center gap-2">
                <span class="text-[10px] ${indicatorClass}">
                  ${count} / ${limit} ${state.byteCalcMode === 'char' ? '자' : 'Byte'}
                </span>
                <button onclick="deleteSubject('${sub.id}', '${sub.name}')" class="h-6 w-6 rounded hover:bg-red-50 text-slate-400 hover:text-red-500 flex items-center justify-center transition">
                  <i data-lucide="trash-2" class="h-3.5 w-3.5"></i>
                </button>
              </div>
            </div>
            <textarea id="sub-${sub.id}" placeholder="[${sub.name}] 교과 수업에서의 성취 및 탐구활동, 심화 독서 등 세부 능력 특기사항을 기재해 주세요." rows="3" class="w-full rounded-xl border border-slate-200 bg-white p-3 text-xs leading-relaxed outline-none focus:border-blue-500 dark:border-slate-800 dark:bg-slate-950 dark:text-slate-100 transition resize-none">${textVal}</textarea>
            <div class="mt-2 flex justify-end gap-2">
              <button onclick="sendToSpellchecker('sub-${sub.id}')" class="rounded-lg border border-slate-200 dark:border-slate-800 hover:bg-slate-50 dark:hover:bg-slate-950 px-2.5 py-1.5 text-[10px] font-bold flex items-center gap-1 text-slate-600 dark:text-slate-300 transition">
                <i data-lucide="spell-check" class="h-3 w-3 text-emerald-500"></i>
                맞춤법 검사
              </button>
              <button onclick="triggerAISubjectFeedback('${sub.id}', '${sub.name}')" class="rounded-lg bg-purple-50 hover:bg-purple-100/80 dark:bg-purple-950/30 dark:hover:bg-purple-950/50 px-2.5 py-1.5 text-[10px] font-bold text-purple-600 dark:text-purple-400 flex items-center gap-1 transition">
                <i data-lucide="sparkles" class="h-3 w-3 text-purple-500"></i>
                AI 세특 탐구 컨설팅
              </button>
            </div>
          </div>
        `;
        container.insertAdjacentHTML('beforeend', rawHtml);

        const textarea = document.getElementById(`sub-${sub.id}`);
        textarea.addEventListener('input', (e) => {
          let val = e.target.value;
          if (val.length > limit && state.byteCalcMode === 'char') {
            val = val.substring(0, limit);
            e.target.value = val;
          }
          sub.content = val;
          renderSubjectsList();
          const restoredEl = document.getElementById(`sub-${sub.id}`);
          restoredEl.focus();
          restoredEl.setSelectionRange(val.length, val.length);
        });
      });
      lucide.createIcons();
    }

    // ==========================================
    // [RENDER GRADES TABLE]
    // ==========================================
    function getGradesTabSubjects() {
      const activeSubjects = state.allGradesData[state.grade]?.subjects || [];
      if (state.grade !== '1') return activeSubjects;

      const splitSubjects = [];
      activeSubjects.forEach(sub => {
        const targetNamesToSplit = ["공통국어", "공통수학", "공통영어", "통합사회", "통합과학"];
        if (targetNamesToSplit.includes(sub.name)) {
          splitSubjects.push({ id: `${sub.id}_1`, name: `${sub.name}1`, isSplit: true });
          splitSubjects.push({ id: `${sub.id}_2`, name: `${sub.name}2`, isSplit: true });
        } else {
          splitSubjects.push(sub);
        }
      });
      return splitSubjects;
    }

    function renderGradesTable() {
      const tbody = document.getElementById('grades-table-body');
      tbody.innerHTML = '';

      const subjects = getGradesTabSubjects();
      if (subjects.length === 0) {
        tbody.innerHTML = `<tr><td colspan="6" class="py-8 text-center text-slate-400">성적을 산출할 교과 정보가 없습니다. 생기부 기록 탭에서 과목을 먼저 구성해 주세요.</td></tr>`;
        return;
      }

      subjects.forEach(sub => {
        const subGrade = state.allGradesData[state.grade]?.grades?.[sub.id] || { units: '', rank: '', total: '', joint: 1 };
        const isTarget = !isLiberalArtSubject(sub.name);

        const rowHtml = `
          <tr class="hover:bg-slate-50/50 dark:hover:bg-slate-900/50">
            <td class="py-3 font-semibold text-slate-800 dark:text-slate-200">${sub.name}</td>
            <td class="py-2">
              <input type="number" ${isTarget ? '' : 'disabled'} id="grade-units-${sub.id}" value="${subGrade.units || ''}" placeholder="단위수" class="w-[80px] rounded border border-slate-200 bg-white p-1 outline-none text-xs dark:border-slate-800 dark:bg-slate-950 disabled:bg-slate-100">
            </td>
            <td class="py-2">
              <input type="number" ${isTarget ? '' : 'disabled'} id="grade-rank-${sub.id}" value="${subGrade.rank || ''}" placeholder="석차" class="w-[80px] rounded border border-slate-200 bg-white p-1 outline-none text-xs dark:border-slate-800 dark:bg-slate-950 disabled:bg-slate-100">
            </td>
            <td class="py-2">
              <input type="number" ${isTarget ? '' : 'disabled'} id="grade-joint-${sub.id}" value="${subGrade.joint || ''}" placeholder="동점자" class="w-[80px] rounded border border-slate-200 bg-white p-1 outline-none text-xs dark:border-slate-800 dark:bg-slate-950 disabled:bg-slate-100">
            </td>
            <td class="py-2">
              <input type="number" ${isTarget ? '' : 'disabled'} id="grade-total-${sub.id}" value="${subGrade.total || ''}" placeholder="총인원" class="w-[80px] rounded border border-slate-200 bg-white p-1 outline-none text-xs dark:border-slate-800 dark:bg-slate-950 disabled:bg-slate-100">
            </td>
            <td class="py-2 text-right">
              ${!isTarget ? `
                <span class="text-slate-400">P/F 과목</span>
              ` : `
                <div class="flex flex-col items-end">
                  <span class="font-extrabold text-blue-600 dark:text-blue-400" id="pred-g9-${sub.id}">-</span>
                  <span class="text-[10px] text-purple-500" id="pred-g5-${sub.id}">-</span>
                </div>
              `}
            </td>
          </tr>
        `;
        tbody.insertAdjacentHTML('beforeend', rowHtml);

        if (isTarget) {
          updatePredictionBadges(sub.id, subGrade);

          // 입력 이벤트 리스너 바인딩
          ['units', 'rank', 'joint', 'total'].forEach(field => {
            const input = document.getElementById(`grade-${field}-${sub.id}`);
            input.addEventListener('input', (e) => {
              const val = Number(e.target.value);
              if (!state.allGradesData[state.grade].grades[sub.id]) {
                state.allGradesData[state.grade].grades[sub.id] = { units: 0, rank: 0, total: 0, joint: 1 };
              }
              state.allGradesData[state.grade].grades[sub.id][field] = val;
              
              // 즉시 로컬 연산 및 계산 배지 갱신
              const currentObj = state.allGradesData[state.grade].grades[sub.id];
              const { grade9, grade5 } = calculateGradeFromRank(currentObj.rank, currentObj.total, currentObj.joint || 1);
              currentObj.grade9 = grade9;
              currentObj.grade5 = grade5;

              updatePredictionBadges(sub.id, currentObj);
              updateGPABadges();
              drawChart();
            });
          });
        }
      });
    }

    function updatePredictionBadges(id, subGrade) {
      const g9El = document.getElementById(`pred-g9-${id}`);
      const g5El = document.getElementById(`pred-g5-${id}`);
      if (g9El && g5El) {
        g9El.textContent = subGrade.grade9 ? `${subGrade.grade9}등급 (9등급)` : '-';
        g5El.textContent = subGrade.grade5 ? `${subGrade.grade5}등급 (5등급)` : '';
      }
    }

    // ==========================================
    // [CANVAS GRADES CHART & ANALYSIS]
    // ==========================================
    function drawChart() {
      const canvas = document.getElementById('grades-trend-canvas');
      if (!canvas) return;

      const ctx = canvas.getContext('2d');
      canvas.width = 600;
      canvas.height = 300;

      const data1 = calculateGPAByGrade('1');
      const data2 = calculateGPAByGrade('2');
      const data3 = calculateGPAByGrade('3');

      const points9 = [
        { label: '고1', val: data1.average9 },
        { label: '고2', val: data2.average9 },
        { label: '고3', val: data3.average9 }
      ];

      const points5 = [
        { label: '고1', val: data1.average5 },
        { label: '고2', val: data2.average5 },
        { label: '고3', val: data3.average5 }
      ];

      const isDark = document.documentElement.classList.contains('dark');
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Y축 등급 격자선 그리기 (1~9등급)
      ctx.strokeStyle = isDark ? '#1e293b' : '#f1f5f9';
      ctx.lineWidth = 1;
      for (let i = 1; i < 10; i++) {
        const y = 20 + (i - 1) * 25;
        ctx.beginPath();
        ctx.moveTo(60, y);
        ctx.lineTo(550, y);
        ctx.stroke();

        ctx.fillStyle = isDark ? '#94a3b8' : '#64748b';
        ctx.font = 'bold 10px sans-serif';
        ctx.fillText(`${i}등급`, 25, y + 3);
      }

      const xCoords = [120, 300, 480];
      ctx.strokeStyle = isDark ? '#334155' : '#cbd5e1';
      ctx.beginPath();
      ctx.moveTo(60, 245);
      ctx.lineTo(550, 245);
      ctx.stroke();

      // 학년 라벨 출력
      points9.forEach((p, idx) => {
        ctx.fillStyle = isDark ? '#cbd5e1' : '#334155';
        ctx.font = 'bold 12px sans-serif';
        ctx.textAlign = 'center';
        ctx.fillText(p.label, xCoords[idx], 270);
      });

      // 추세선 라인 렌더링 함수
      const drawTrendLine = (points, strokeColor, fillColor) => {
        const validPoints = points
          .map((p, idx) => ({ ...p, x: xCoords[idx] }))
          .filter(p => p.val !== null && !isNaN(p.val));

        if (validPoints.length === 0) return;

        ctx.strokeStyle = strokeColor;
        ctx.lineWidth = 3;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';
        ctx.beginPath();
        
        validPoints.forEach((p, idx) => {
          const y = 20 + (p.val - 1) * 25; 
          if (idx === 0) ctx.moveTo(p.x, y);
          else ctx.lineTo(p.x, y);
        });
        ctx.stroke();

        validPoints.forEach(p => {
          const y = 20 + (p.val - 1) * 25;
          ctx.fillStyle = strokeColor;
          ctx.beginPath();
          ctx.arc(p.x, y, 6, 0, 2 * Math.PI);
          ctx.fill();

          ctx.fillStyle = '#ffffff';
          ctx.beginPath();
          ctx.arc(p.x, y, 3, 0, 2 * Math.PI);
          ctx.fill();

          ctx.fillStyle = isDark ? '#f8fafc' : '#0f172a';
          ctx.font = 'bold 11px sans-serif';
          ctx.textAlign = 'center';
          ctx.fillText(`${Number(p.val).toFixed(2)}`, p.x, y - 12);
        });
      };

      drawTrendLine(points9, '#3b82f6', '#60a5fa');
      drawTrendLine(points5, '#8b5cf6', '#a78bfa');

      // 정성 평가 리포트 업데이트
      updateTrendAnalysisReport(points9);
    }

    function updateTrendAnalysisReport(points) {
      const validPoints = points.filter(p => p.val !== null);
      const iconWrapper = document.getElementById('trend-icon-wrapper');
      const textEl = document.getElementById('trend-analysis-text');

      if (validPoints.length < 2) {
        iconWrapper.innerHTML = `<div class="flex h-12 w-12 items-center justify-center rounded-full bg-slate-50 text-slate-400 dark:bg-slate-900"><i data-lucide="info" class="h-7 w-7"></i></div>`;
        textEl.textContent = "분석을 위한 성적 데이터가 부족합니다.";
        lucide.createIcons();
        return;
      }

      const first = validPoints[0];
      const last = validPoints[validPoints.length - 1];
      const diff = first.val - last.val;

      if (diff > 0.15) {
        iconWrapper.innerHTML = `<div class="flex h-12 w-12 items-center justify-center rounded-full bg-emerald-50 text-emerald-500 dark:bg-emerald-950/40"><i data-lucide="trending-up" class="h-7 w-7"></i></div>`;
        textEl.textContent = "학년이 갈수록 등급이 낮아지는(성적이 상승하는) 대표적인 '지속 우상향(학업 성취도 개선형)' 흐름입니다. 대입 학생부 종합전형에서 학업 주도성과 발전 가능성 부문에서 매우 긍정적인 정성 평가를 받을 확률이 높습니다.";
      } else if (diff < -0.15) {
        iconWrapper.innerHTML = `<div class="flex h-12 w-12 items-center justify-center rounded-full bg-red-50 text-red-500 dark:bg-red-950/40"><i data-lucide="trending-down" class="h-7 w-7"></i></div>`;
        textEl.textContent = "학년이 올라갈수록 등급 수치가 올라가는 '우하향(학업 관리 보완 필요형)' 흐름이 감지되었습니다. 3학년 1학기 최종 내신 및 주력 선택과목들의 성적 방어가 필요하며, 학생부 기재 내용(세특)을 통해 학업적 성실성을 추가 보강하는 보완 전략이 요구됩니다.";
      } else {
        iconWrapper.innerHTML = `<div class="flex h-12 w-12 items-center justify-center rounded-full bg-blue-50 text-blue-500 dark:bg-blue-950/40"><i data-lucide="award" class="h-7 w-7"></i></div>`;
        textEl.textContent = "일정한 내신 등급 구간을 유지하는 '성적 안정형(지속 유지형)' 흐름입니다. 급격한 편차 없이 성실함을 꾸준히 증명하고 있으므로, 전공 핵심 권장 과목의 이수 여부와 성취도 수준이 입시 성공의 핵심 열쇠가 될 것입니다.";
      }
      lucide.createIcons();
    }

    // ==========================================
    // [AI INTEGRATION LOGIC]
    // ==========================================
    window.triggerAIFeedback = async function(fixedFieldKey) {
      const textVal = state.allGradesData[state.grade]?.fixed?.[fixedFieldKey] || '';
      if (!textVal.trim()) {
        showCustomModal('작성 확인', '작성된 기록 내용이 없습니다. 먼저 내용을 입력하고 진행해 주세요.');
        return;
      }

      const activeLabel = FIXED_STRUCTURE[fixedFieldKey].label;
      const loader = document.getElementById('ai-feedback-loading');
      const content = document.getElementById('ai-feedback-content');
      const box = document.getElementById('ai-feedback-container');

      box.classList.remove('hidden');
      loader.classList.remove('hidden');
      content.textContent = '';

      const prompt = `
        당신은 대입 학생부 종합전형의 베테랑 사정관이자 학교생활기록부 컨설턴트입니다.
        학생이 작성한 고등학교 생활기록부 [${activeLabel}] 항목의 기록 초안을 읽고 피드백을 주어야 합니다.

        [작성 항목]
        - 학년: ${state.grade}학년
        - 타겟학과: ${state.selectedDept}
        - 항목유형: ${activeLabel}
        - 입력내용: "${textVal}"

        [답변 조건 및 가이드라인]
        1. 전문적이고 친절한 격려의 어조로 답변하세요. (한국어로 출력)
        2. 강점 분석: 현재 기록에 드러난 학생의 주도성과 역량을 짚어주세요.
        3. 보완 전략: 타겟학과인 [${state.selectedDept}]와 관련하여, 전공적합성 및 탐구의 심화도를 높이기 위해 구체적으로 어떤 단어, 추가 도서, 혹은 확장 탐구 주제를 덧붙여 수정하면 좋을지 예시 문장과 함께 명확히 처방을 제안하세요.
        4. 글자수 최적화 및 구조 제안을 제공해 주세요.
      `;

      try {
        const feedback = await callGeminiFeedbackAPI(textVal, prompt);
        content.textContent = feedback;
      } catch (err) {
        content.textContent = "죄송합니다. 인공지능 분석 연동 도중 에러가 발생했습니다. 다시 시도해 주세요.";
      } finally {
        loader.classList.add('hidden');
      }
    };

    window.triggerAISubjectFeedback = async function(subId, name) {
      const subjects = state.allGradesData[state.grade]?.subjects || [];
      const sub = subjects.find(s => s.id === subId);
      if (!sub || !sub.content.trim()) {
        showCustomModal('작성 확인', '과목 세특에 분석할 기록 내용이 존재하지 않습니다.');
        return;
      }

      const loader = document.getElementById('ai-feedback-loading');
      const content = document.getElementById('ai-feedback-content');
      const box = document.getElementById('ai-feedback-container');

      box.classList.remove('hidden');
      loader.classList.remove('hidden');
      content.textContent = '';

      const prompt = `
        당신은 최고의 고교 진학지도 교사입니다. 학생의 선택과목 세특 구성을 평가해 주십시오.

        [과목 정보]
        - 교과목명: ${name}
        - 타겟학과: ${state.selectedDept}
        - 입력 세특: "${sub.content}"

        [답변 조건]
        1. 학업 역량과 주도적 탐구 태도가 드러나는지 진단하세요.
        2. 전공적합성 관점: ${state.selectedDept}와 해당 교과가 어떤 연결고리로 입시 사정관에게 매력적으로 보일 수 있는지 분석하고, 세특에 녹여낼 수 있는 심화 탐구주제, 독서 연계, 추가 개념을 명확히 제시하세요.
        3. 세특 예문 수정 제안을 제공하세요.
      `;

      try {
        const feedback = await callGeminiFeedbackAPI(sub.content, prompt);
        content.textContent = feedback;
      } catch (err) {
        content.textContent = "교과 세특 AI 연동 중 문제가 발생했습니다.";
      } finally {
        loader.classList.add('hidden');
      }
    };

    // ==========================================
    // [SPELLCHECKER ENGINE]
    // ==========================================
    window.sendToSpellchecker = function(id) {
      const el = document.getElementById(id);
      if (el) {
        document.getElementById('spellcheck-textarea').value = el.value;
        switchTab('feedback');
        handleSpellCheck();
      }
    };

    async function handleSpellCheck() {
      const text = document.getElementById('spellcheck-textarea').value;
      if (!text.trim()) {
        showCustomModal('확인', '검사할 문장을 입력해 주세요.');
        return;
      }

      const wrapper = document.getElementById('spellcheck-results-wrapper');
      const neisStatus = document.getElementById('neis-violation-status');
      const spellAiOutput = document.getElementById('spell-ai-output');

      wrapper.classList.remove('hidden');
      neisStatus.innerHTML = '';
      spellAiOutput.textContent = '정밀 진단 분석을 불러오고 있습니다...';

      // 1. NEIS 제한 특수기호 검출
      const matches = text.match(NEIS_FORBIDDEN_CHARS_REGEX) || [];
      const uniqueForbidden = [...new Set(matches)];

      if (uniqueForbidden.length > 0) {
        neisStatus.innerHTML = `
          <div class="space-y-2">
            <p class="text-[11px] text-red-700 dark:text-red-300 leading-normal font-semibold">
              NEIS 입력 불가 특수문자가 감지되었습니다. 생활기록부 등록 전에 반드시 소거하시기 바랍니다.
            </p>
            <div class="flex flex-wrap gap-1">
              ${uniqueForbidden.map(char => `<span class="rounded bg-red-100 dark:bg-red-900/50 text-red-600 dark:text-red-300 font-extrabold px-2 py-1 text-xs">${char}</span>`).join('')}
            </div>
          </div>
        `;
      } else {
        neisStatus.innerHTML = `
          <div class="flex flex-col items-center py-4 text-center">
            <i data-lucide="check-circle-2" class="h-8 w-8 text-green-500"></i>
            <span class="text-[11px] text-green-600 dark:text-green-400 font-bold mt-2">안전: 금지 기호 무결</span>
          </div>
        `;
        lucide.createIcons();
      }

      // 2. AI 교정 진단 API 호출
      const prompt = `
        당신은 한국어 맞춤법 및 띄어쓰기 전문가이자 생기부 전문 에디터입니다.
        사용자가 제시한 다음 문장의 맞춤법, 띄어쓰기, 문맥상 비문 등을 꼼꼼히 확인하여 완벽한 문장으로 고쳐주어야 합니다.

        [검사 대상 문장]
        "${text}"

        [답변 조건]
        1. 교정 전후 비교 표 및 어떤 부분이 틀렸는지 간결한 원인 설명을 포함하세요.
        2. 완성된 최종 전체 수정본 문장을 따로 복사할 수 있도록 한 단락에 온전히 정리해서 하단에 배치하세요.
        3. 생기부 입력 조건에 어긋나는 과장되거나 상투적인 수식어(예: 매우 훌륭한, 완벽한)가 있다면 자연스럽게 정성적/사실적 묘사로 윤문해 주세요.
      `;

      try {
        const feedback = await callGeminiFeedbackAPI(text, prompt);
        spellAiOutput.textContent = feedback;
      } catch (err) {
        spellAiOutput.textContent = "서버 장애로 인해 분석 결과를 불러오지 못했습니다.";
      }
    }

    window.clearSpellChecker = function() {
      document.getElementById('spellcheck-textarea').value = '';
      document.getElementById('spellcheck-results-wrapper').classList.add('hidden');
    };

    // ==========================================
    // [SUBJECT MANAGEMENT LOGIC]
    // ==========================================
    window.deleteSubject = function(id, name) {
      showConfirmModal(
        '과목 삭제 확인',
        `정말 '${name}' 과목 기록을 삭제하시겠습니까? 삭제한 정보는 복구할 수 없습니다.`,
        () => {
          const subjects = state.allGradesData[state.grade].subjects || [];
          state.allGradesData[state.grade].subjects = subjects.filter(s => s.id !== id);
          
          if (state.grade === '1') {
            delete state.allGradesData[state.grade].grades[`${id}_1`];
            delete state.allGradesData[state.grade].grades[`${id}_2`];
          }
          delete state.allGradesData[state.grade].grades[id];

          renderSubjectsList();
          renderGradesTable();
          updateGPABadges();
          drawChart();
        }
      );
    };

    function addSelectedSubjectName(subjectName) {
      const currentSubjects = state.allGradesData[state.grade]?.subjects || [];
      if (currentSubjects.some(sub => sub.name === subjectName)) {
        showCustomModal('중복 경고', `이미 '${subjectName}' 과목이 리스트에 존재합니다.`);
        return;
      }

      const newSub = {
        id: `custom_${Date.now()}`,
        name: subjectName,
        content: '',
        limit: 500
      };

      state.allGradesData[state.grade].subjects.push(newSub);
      document.getElementById('add-subject-wrapper').classList.add('hidden');
      renderSubjectsList();
      renderGradesTable();
    }

    // ==========================================
    // [GROUP STUDY COMMUNITY / CHAT ENGINE]
    // ==========================================
    let chatUnsub = null;
    let membersUnsub = null;

    async function handleCreateGroupRoom(e) {
      e.preventDefault();
      const category = document.getElementById('create-room-category').value.trim();
      const name = document.getElementById('create-room-name').value.trim();
      const password = document.getElementById('create-room-password').value;

      if (!state.user) return;

      try {
        const publicCollectionRef = collection(db, 'artifacts', appId, 'public', 'data', 'study_rooms');
        await addDoc(publicCollectionRef, {
          category,
          roomName: name,
          password,
          creatorId: state.user.uid,
          createdAt: new Date().toISOString()
        });

        document.getElementById('create-room-category').value = '';
        document.getElementById('create-room-name').value = '';
        document.getElementById('create-room-password').value = '';

        showCustomModal('성공', '방이 성공적으로 개설되었습니다! 검색하여 참가하세요.');
        handleSearchRooms(category);
      } catch (error) {
        showCustomModal('오류', '방 개설에 실패했습니다.');
      }
    }

    async function handleSearchRooms(keyword) {
      const searchInput = keyword || document.getElementById('search-room-category').value.trim();
      if (!searchInput) {
        showCustomModal('검색어 오류', '학교명을 입력해 주세요.');
        return;
      }

      const loader = document.getElementById('room-search-loading');
      const emptyMsg = document.getElementById('room-search-empty');
      const container = document.getElementById('matching-rooms-container');

      loader.classList.remove('hidden');
      emptyMsg.classList.add('hidden');
      container.innerHTML = '';

      try {
        const publicCollectionRef = collection(db, 'artifacts', appId, 'public', 'data', 'study_rooms');
        const snapshot = await getDocs(publicCollectionRef);

        const rooms = [];
        snapshot.forEach(docSnap => {
          const d = docSnap.data();
          if (d.category.toLowerCase().includes(searchInput.toLowerCase())) {
            rooms.push({ id: docSnap.id, ...d });
          }
        });

        loader.classList.add('hidden');
        if (rooms.length === 0) {
          emptyMsg.classList.remove('hidden');
          return;
        }

        rooms.forEach(room => {
          const deleteBtn = state.user && state.user.uid === room.creatorId ? `
            <button onclick="deleteGroupRoom('${room.id}', '${room.creatorId}')" class="rounded-lg bg-red-50 hover:bg-red-100 text-red-500 p-1.5 transition">
              <i data-lucide="trash-2" class="h-3.5 w-3.5"></i>
            </button>
          ` : '';

          const roomHtml = `
            <div class="flex items-center justify-between rounded-xl bg-slate-50 dark:bg-slate-950 p-3 border border-slate-100 dark:border-slate-800">
              <div>
                <span class="rounded bg-blue-100 text-blue-600 px-1.5 py-0.5 text-[8px] font-bold dark:bg-blue-950/40 dark:text-blue-400">${room.category}</span>
                <h4 class="text-xs font-bold text-slate-800 dark:text-slate-200 mt-1">${room.roomName}</h4>
              </div>
              <div class="flex items-center gap-1.5">
                <button onclick="requestJoinRoom('${room.id}')" class="rounded-lg bg-blue-600 hover:bg-blue-700 text-white px-3 py-1.5 text-[10px] font-bold transition">참가</button>
                ${deleteBtn}
              </div>
            </div>
          `;
          container.insertAdjacentHTML('beforeend', roomHtml);
        });
        lucide.createIcons();
      } catch (err) {
        loader.classList.add('hidden');
      }
    }

    window.requestJoinRoom = async function(roomId) {
      try {
        const roomRef = doc(db, 'artifacts', appId, 'public', 'data', 'study_rooms', roomId);
        const snapshot = await getDocs(query(collection(db, 'artifacts', appId, 'public', 'data', 'study_rooms')));
        let targetRoom = null;
        snapshot.forEach(docSnap => {
          if (docSnap.id === roomId) targetRoom = { id: docSnap.id, ...docSnap.data() };
        });

        if (!targetRoom) return;

        const enteredPw = prompt('스터디방 비밀번호를 입력해 주세요:');
        if (enteredPw === null) return;

        if (targetRoom.password && targetRoom.password !== enteredPw) {
          showCustomModal('오류', '비밀번호가 방 정보와 일치하지 않습니다.');
          return;
        }

        state.activeRoom = targetRoom;
        
        // UI 분기 전환
        document.getElementById('group-search-wrapper').classList.add('hidden');
        document.getElementById('active-chat-wrapper').classList.remove('hidden');

        document.getElementById('active-room-category-badge').textContent = targetRoom.category;
        document.getElementById('active-room-name-text').textContent = targetRoom.roomName;

        // 실시간 채팅 바인딩
        initChatListener(roomId);
        initRoomMembersListener();
      } catch (err) {
        showCustomModal('오류', '참가에 실패했습니다.');
      }
    };

    function initChatListener(roomId) {
      if (chatUnsub) chatUnsub();

      const msgCollection = collection(db, 'artifacts', appId, 'public', 'data', `chat_${roomId}`);
      chatUnsub = onSnapshot(msgCollection, (snapshot) => {
        const msgs = [];
        snapshot.forEach(docSnap => {
          msgs.push({ id: docSnap.id, ...docSnap.data() });
        });
        msgs.sort((a, b) => new Date(a.time) - new Date(b.time));

        const scrollArea = document.getElementById('chat-messages-scroll-area');
        scrollArea.innerHTML = '';

        msgs.forEach(msg => {
          const isMe = msg.senderId === state.user.uid;
          const msgHtml = `
            <div class="flex ${isMe ? 'justify-end' : 'justify-start'}">
              <div class="max-w-[70%] rounded-xl px-3 py-2 text-xs leading-relaxed ${isMe ? 'bg-blue-600 text-white rounded-br-none' : 'bg-slate-100 text-slate-800 dark:bg-slate-800 dark:text-slate-200 rounded-bl-none'}">
                <div class="text-[9px] opacity-75 font-semibold mb-0.5">${msg.senderName}</div>
                <div>${msg.text}</div>
              </div>
            </div>
          `;
          scrollArea.insertAdjacentHTML('beforeend', msgHtml);
        });

        scrollArea.scrollTop = scrollArea.scrollHeight;
      }, (err) => {
        console.error("채팅 메시지 바인딩 오류:", err);
      });
    }

    function initRoomMembersListener() {
      if (membersUnsub) membersUnsub();

      const membersMetaRef = collection(db, 'artifacts', appId, 'public', 'data', 'all_member_grades');
      membersUnsub = onSnapshot(membersMetaRef, (snapshot) => {
        const list = [];
        snapshot.forEach(docSnap => {
          const d = docSnap.data();
          if (d.shareGrades) list.push(d);
        });

        const container = document.getElementById('member-grades-scroll-area');
        container.innerHTML = '';

        if (list.length === 0) {
          container.innerHTML = `<p class="text-[10px] text-slate-400 text-center py-8">공유 동의를 완료한 멤버가 없습니다.</p>`;
          return;
        }

        list.forEach(member => {
          const memberHtml = `
            <div class="rounded-xl border border-slate-100 dark:border-slate-800 bg-slate-50 dark:bg-slate-950 p-3">
              <div class="flex justify-between items-center mb-1">
                <span class="text-xs font-bold text-slate-800 dark:text-slate-200">${member.customId || '익명'}</span>
                <span class="text-[9px] text-slate-400">UID: ${member.userId.substring(0, 6)}</span>
              </div>
              <div class="grid grid-cols-3 gap-1 text-[10px] text-center font-bold">
                <div class="bg-white dark:bg-slate-900 rounded p-1">
                  <div class="text-[8px] text-slate-400">1학년</div>
                  <div class="text-blue-500">${member.gpa1 ? `${member.gpa1}평점` : '미기재'}</div>
                </div>
                <div class="bg-white dark:bg-slate-900 rounded p-1">
                  <div class="text-[8px] text-slate-400">2학년</div>
                  <div class="text-blue-500">${member.gpa2 ? `${member.gpa2}평점` : '미기재'}</div>
                </div>
                <div class="bg-white dark:bg-slate-900 rounded p-1">
                  <div class="text-[8px] text-slate-400">3학년</div>
                  <div class="text-blue-500">${member.gpa3 ? `${member.gpa3}평점` : '미기재'}</div>
                </div>
              </div>
            </div>
          `;
          container.insertAdjacentHTML('beforeend', memberHtml);
        });
      }, (err) => {
        console.error("멤버 데이터 연동 오류:", err);
      });
    }

    async function handleSendChatMessage(e) {
      e.preventDefault();
      const input = document.getElementById('chat-message-input');
      const text = input.value.trim();
      if (!text || !state.activeRoom) return;

      try {
        const msgCollection = collection(db, 'artifacts', appId, 'public', 'data', `chat_${state.activeRoom.id}`);
        await addDoc(msgCollection, {
          senderId: state.user.uid,
          senderName: state.customId || (state.user.email ? state.user.email.split('@')[0] : '익명학생'),
          text,
          time: new Date().toISOString()
        });
        input.value = '';
      } catch (err) {
        console.error("메시지 전송 오류:", err);
      }
    }

    window.deleteGroupRoom = async function(roomId, creatorId) {
      if (state.user.uid !== creatorId) {
        showCustomModal('오류', '개설자만 해당 방을 영구 폐쇄할 수 있습니다.');
        return;
      }

      showConfirmModal(
        '방 영구 폐쇄',
        '이 방을 삭제하시겠습니까? 관련 모든 대화와 룸 메타가 일괄 제거됩니다.',
        async () => {
          try {
            const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'study_rooms', roomId);
            await deleteDoc(docRef);
            state.activeRoom = null;
            document.getElementById('active-chat-wrapper').classList.add('hidden');
            document.getElementById('group-search-wrapper').classList.remove('hidden');
            handleSearchRooms();
          } catch (err) {
            console.error("방 삭제 오류:", err);
          }
        }
      );
    };

    // ==========================================
    // [FIRESTORE PERSISTENT CLOUD DATA SYNC]
    // ==========================================
    async function saveToCloud() {
      if (!state.user) return;

      const saveBtn = document.getElementById('save-cloud-btn');
      const saveIcon = document.getElementById('save-btn-icon');
      const saveText = document.getElementById('save-btn-text');
      const saveMsg = document.getElementById('save-status-msg');

      saveBtn.disabled = true;
      saveIcon.classList.add('animate-spin');
      saveText.textContent = '클라우드에 저장 중...';

      try {
        const docRef = doc(db, 'artifacts', appId, 'users', state.user.uid, 'records', `grade_${state.grade}`);
        const dataToSave = state.allGradesData[state.grade];

        await setDoc(docRef, {
          fixed: dataToSave.fixed || {},
          subjects: dataToSave.subjects || [],
          grades: dataToSave.grades || {},
          targetDept: state.selectedDept,
          lastUpdated: new Date().toISOString()
        });

        // 타 학생 비교용 메타 파일 백업 동기화
        const metaRef = doc(db, 'artifacts', appId, 'public', 'data', 'all_member_grades', state.user.uid);
        await setDoc(metaRef, {
          userId: state.user.uid,
          customId: state.customId || (state.user.email ? state.user.email.split('@')[0] : '익명학생'),
          shareGrades: state.shareGrades,
          gpa1: calculateGPAByGrade('1').average9,
          gpa2: calculateGPAByGrade('2').average9,
          gpa3: calculateGPAByGrade('3').average9,
          lastUpdated: new Date().toISOString()
        }, { merge: true });

        saveMsg.classList.remove('hidden', 'text-red-500');
        saveMsg.classList.add('text-emerald-500');
        saveMsg.textContent = '성공적으로 클라우드 백업이 완료되었습니다.';
      } catch (err) {
        console.error("데이터 백업 오류:", err);
        saveMsg.classList.remove('hidden', 'text-emerald-500');
        saveMsg.classList.add('text-red-500');
        saveMsg.textContent = '네트워크 불안정 또는 권한 에러 발생.';
      } finally {
        saveBtn.disabled = false;
        saveIcon.classList.remove('animate-spin');
        saveText.textContent = '현재 학년 데이터 저장';
        setTimeout(() => saveMsg.classList.add('hidden'), 3500);
      }
    }

    // ==========================================
    // [INITIAL SYNC LISTENER (MAIN STATE)]
    // ==========================================
    function initCloudSyncListeners(currentUser) {
      if (!currentUser) return;

      // 1. 프로필 세팅 동기화
      const profileRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'profile', 'settings');
      onSnapshot(profileRef, (snap) => {
        if (snap.exists()) {
          const data = snap.data();
          state.customId = data.customId || '';
          state.shareGrades = !!data.shareGrades;
        } else {
          state.customId = currentUser.email ? currentUser.email.split('@')[0] : '익명학생';
          state.shareGrades = false;
        }
        document.getElementById('custom-id-input').value = state.customId;
        document.getElementById('share-grades-checkbox').checked = state.shareGrades;
      });

      // 2. 3개 학년 데이터 실시간 결속 리스너
      ['1', '2', '3'].forEach(g => {
        const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'records', `grade_${g}`);
        onSnapshot(docRef, (docSnap) => {
          if (docSnap.exists()) {
            const data = docSnap.data();
            state.allGradesData[g] = {
              fixed: data.fixed || {},
              subjects: data.subjects || [],
              grades: data.grades || {},
              targetDept: data.targetDept || '컴퓨터공학과'
            };
          } else {
            // 이니셜 구조 매핑
            const initialFixed = {};
            Object.keys(FIXED_STRUCTURE).forEach(key => initialFixed[key] = "");

            const defaultSubjects = DEFAULT_GRADE_SUBJECTS[g].map((sub, index) => ({
              id: `default_${g}_${index}_${Date.now()}`,
              name: sub.name,
              content: sub.content,
              limit: sub.limit
            }));

            state.allGradesData[g] = {
              fixed: initialFixed,
              subjects: defaultSubjects,
              grades: {},
              targetDept: '컴퓨터공학과'
            };
          }
          if (g === state.grade) {
            setGlobalGrade(state.grade);
          }
        });
      });
    }

    // ==========================================
    // [USER AUTH & INTERACTION CONTROLLER]
    // ==========================================
    async function handleAuthFormSubmit(e) {
      e.preventDefault();
      const emailInput = document.getElementById('auth-email').value;
      const passwordInput = document.getElementById('auth-password').value;
      const rememberCheckbox = document.getElementById('auth-remember').checked;

      hideAuthAlert();

      const finalEmail = emailInput.trim().toLowerCase() === 'admin' ? 'admin@admin.com' : emailInput.trim();

      if (!/\S+@\S+\.\S+/.test(finalEmail)) {
        showAuthAlert('error', '정확한 이메일 형식을 기입해 주세요.');
        return;
      }

      state.rememberMe = rememberCheckbox;
      if (typeof window !== 'undefined') {
        localStorage.setItem('remember_me', rememberCheckbox ? 'true' : 'false');
        if (rememberCheckbox) localStorage.setItem('saved_email', emailInput);
        else localStorage.removeItem('saved_email');
      }

      try {
        const persistenceType = rememberCheckbox ? browserLocalPersistence : browserSessionPersistence;
        await setPersistence(auth, persistenceType);

        if (state.view === 'login') {
          await signInWithEmailAndPassword(auth, finalEmail, passwordInput);
        } else if (state.view === 'signup') {
          if (passwordInput.length < 6) {
            showAuthAlert('error', '비밀번호는 최소 6자 이상이어야 합니다.');
            return;
          }
          await createUserWithEmailAndPassword(auth, finalEmail, passwordInput);
          showAuthAlert('success', '성공적으로 가입되었습니다! 로그인 중...');
        } else if (state.view === 'forgotPassword') {
          await sendPasswordResetEmail(auth, finalEmail);
          showAuthAlert('success', '비밀번호 재설정 링크가 발송되었습니다.');
        }
      } catch (err) {
        if (state.view === 'login' && (err.code === 'auth/user-not-found' || err.code === 'auth/invalid-credential' || err.code === 'auth/wrong-password')) {
          if (emailInput.trim() === 'admin') {
            try {
              showAuthAlert('success', '데모 어드민 계정을 자동 생성 후 로그인 중...');
              await createUserWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
              await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
              return;
            } catch (signupErr) {}
          }
        }
        showAuthAlert('error', getErrorMessage(err.code));
      }
    }

    async function handleDemoLogin() {
      hideAuthAlert();
      document.getElementById('auth-email').value = 'admin';
      document.getElementById('auth-password').value = 'admin09';
      
      try {
        const persistenceType = state.rememberMe ? browserLocalPersistence : browserSessionPersistence;
        await setPersistence(auth, persistenceType);
        await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
      } catch (err) {
        try {
          await createUserWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
          await signInWithEmailAndPassword(auth, 'admin@admin.com', 'admin09');
        } catch (createErr) {
          showAuthAlert('error', '데모 계정 자동 구성 오류: ' + createErr.message);
        }
      }
    }

    async function handleLogout() {
      try {
        if (chatUnsub) chatUnsub();
        if (membersUnsub) membersUnsub();
        
        await signOut(auth);
        state.user = null;
        state.activeRoom = null;
        state.view = 'login';

        document.getElementById('main-dashboard').classList.add('hidden');
        document.getElementById('auth-section').classList.remove('hidden');
        switchAuthLayout('login');
      } catch (err) {
        console.error("로그아웃 실패:", err);
      }
    }

    // ==========================================
    // [UI RE-ARRANGE HELPERS (VANILLA JAVASCRIPT)]
    // ==========================================
    function showAuthAlert(type, text) {
      const alertBox = document.getElementById('auth-alert');
      const alertText = document.getElementById('auth-alert-text');
      alertBox.classList.remove('hidden', 'bg-red-50', 'text-red-600', 'dark:bg-red-950/30', 'dark:text-red-400', 'border-red-100', 'bg-green-50', 'text-green-600', 'dark:bg-green-950/30', 'dark:text-green-400', 'border-green-100');

      if (type === 'error') {
        alertBox.classList.add('bg-red-50', 'text-red-600', 'dark:bg-red-950/30', 'dark:text-red-400', 'border-red-100', 'border');
      } else {
        alertBox.classList.add('bg-green-50', 'text-green-600', 'dark:bg-green-950/30', 'dark:text-green-400', 'border-green-100', 'border');
      }
      alertText.textContent = text;
      alertBox.classList.remove('hidden');
    }

    function hideAuthAlert() {
      document.getElementById('auth-alert').classList.add('hidden');
    }

    function switchAuthLayout(targetView) {
      state.view = targetView;
      const title = document.getElementById('auth-title');
      const subtitle = document.getElementById('auth-subtitle');
      const passWrapper = document.getElementById('password-field-wrapper');
      const rememberWrapper = document.getElementById('remember-me-wrapper');
      const demoDivider = document.getElementById('demo-auth-divider');
      const demoBtn = document.getElementById('demo-login-btn');
      const submitBtn = document.getElementById('submit-auth-btn');
      const switchText = document.getElementById('switch-view-text');

      hideAuthAlert();

      if (targetView === 'login') {
        title.textContent = '학생부 & 성적 통합 매니저';
        subtitle.textContent = '스마트한 대입 생기부와 성적 추이 분석 관리';
        passWrapper.classList.remove('hidden');
        rememberWrapper.classList.remove('hidden');
        demoDivider.classList.remove('hidden');
        demoBtn.classList.remove('hidden');
        submitBtn.textContent = '로그인';
        switchText.innerHTML = `아직 계정이 없으신가요? <button id="switch-auth-mode" class="text-blue-500 font-bold hover:underline" type="button">회원가입</button>`;
      } else if (targetView === 'signup') {
        title.textContent = '통합 매니저 회원가입';
        subtitle.textContent = '고교 3개년의 체계적 준비를 시작하세요';
        passWrapper.classList.remove('hidden');
        rememberWrapper.classList.add('hidden');
        demoDivider.classList.add('hidden');
        demoBtn.classList.add('hidden');
        submitBtn.textContent = '회원가입 완료';
        switchText.innerHTML = `기존 계정으로 돌아가기 <button id="switch-auth-mode" class="text-blue-500 font-bold hover:underline" type="button">로그인 하기</button>`;
      } else if (targetView === 'forgotPassword') {
        title.textContent = '비밀번호 찾기';
        subtitle.textContent = '가입한 이메일로 암호 초기화 링크를 발송합니다';
        passWrapper.classList.add('hidden');
        rememberWrapper.classList.add('hidden');
        demoDivider.classList.add('hidden');
        demoBtn.classList.add('hidden');
        submitBtn.textContent = '재설정 메일 발송';
        switchText.innerHTML = `기존 계정으로 돌아가기 <button id="switch-auth-mode" class="text-blue-500 font-bold hover:underline" type="button">로그인 하기</button>`;
      }

      // 이벤트 위임 처리 바인딩 유지
      document.getElementById('switch-auth-mode').addEventListener('click', () => {
        if (state.view === 'login') switchAuthLayout('signup');
        else switchAuthLayout('login');
      });
    }

    // ==========================================
    // [MODAL SYSTEM (REPLACES WINDOW ALERTS)]
    // ==========================================
    let modalCallback = null;

    function showCustomModal(title, message) {
      document.getElementById('modal-title').textContent = title;
      document.getElementById('modal-message').textContent = message;
      document.getElementById('modal-cancel-btn').classList.add('hidden'); // 단순 알림용 취소 숨김
      document.getElementById('custom-modal').classList.remove('hidden');
      modalCallback = null;
    }

    function showConfirmModal(title, message, onConfirm) {
      document.getElementById('modal-title').textContent = title;
      document.getElementById('modal-message').textContent = message;
      document.getElementById('modal-cancel-btn').classList.remove('hidden');
      document.getElementById('custom-modal').classList.remove('hidden');
      modalCallback = onConfirm;
    }

    // ==========================================
    // [GEMINI API CLIENT MODULE]
    // ==========================================
    const callGeminiFeedbackAPI = async (userInput, systemPrompt) => {
      const apiKey = ""; // Canvas 환경 지원 자동 인젝션용 빈 주소 유지
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

    // ==========================================
    // [EVENT LISTENER AND SYSTEM MOUNT]
    // ==========================================
    document.addEventListener('DOMContentLoaded', () => {
      // 1. 저장된 이메일 체크 및 자동 보정
      if (typeof window !== 'undefined') {
        const saved = localStorage.getItem('saved_email');
        if (saved) document.getElementById('auth-email').value = saved;
      }

      // 2. 인증 및 UI 핸들러 마운트
      document.getElementById('auth-form').addEventListener('submit', handleAuthFormSubmit);
      document.getElementById('demo-login-btn').addEventListener('click', handleDemoLogin);
      document.getElementById('logout-btn').addEventListener('click', handleLogout);
      document.getElementById('theme-toggle-btn').addEventListener('click', toggleTheme);
      document.getElementById('settings-open-btn').addEventListener('click', () => document.getElementById('settings-drawer').classList.remove('hidden'));
      document.getElementById('settings-close-btn').addEventListener('click', () => document.getElementById('settings-drawer').classList.add('hidden'));
      document.getElementById('settings-save-btn').addEventListener('click', () => document.getElementById('settings-drawer').classList.add('hidden'));
      document.getElementById('save-cloud-btn').addEventListener('click', saveToCloud);
      document.getElementById('go-reset-btn').addEventListener('click', () => switchAuthLayout('forgotPassword'));
      
      // 모달 단일 버블링 바인딩
      document.getElementById('modal-confirm-btn').addEventListener('click', () => {
        document.getElementById('custom-modal').classList.add('hidden');
        if (modalCallback) modalCallback();
      });
      document.getElementById('modal-cancel-btn').addEventListener('click', () => {
        document.getElementById('custom-modal').classList.add('hidden');
        modalCallback = null;
      });

      // 맞춤법 검사기 마운트
      document.getElementById('spellcheck-analyze-btn').addEventListener('click', handleSpellCheck);
      document.getElementById('spellcheck-clear-btn').addEventListener('click', clearSpellChecker);

      // 교과 과목 개설 토글 마운트
      document.getElementById('add-subject-toggle-btn').addEventListener('click', () => {
        const wrapper = document.getElementById('add-subject-wrapper');
        wrapper.classList.toggle('hidden');
        
        // 추천 선택교과 동적 갱신
        const recommendWrapper = document.getElementById('recommend-subject-pool');
        const recommendList = document.getElementById('recommend-subject-list');
        recommendList.innerHTML = '';

        const recommendationList = SELECTABLE_OPTIONAL_SUBJECTS[state.grade] || [];
        if (recommendationList.length > 0) {
          recommendWrapper.classList.remove('hidden');
          recommendationList.forEach(name => {
            const btn = document.createElement('button');
            btn.className = 'rounded bg-white border border-slate-200 hover:border-blue-500 px-2.5 py-1 text-[10px] font-semibold dark:bg-slate-900 dark:border-slate-800';
            btn.textContent = name;
            btn.type = 'button';
            btn.onclick = () => addSelectedSubjectName(name);
            recommendList.appendChild(btn);
          });
        } else {
          recommendWrapper.classList.add('hidden');
        }
      });

      document.getElementById('add-subject-close-btn').addEventListener('click', () => document.getElementById('add-subject-wrapper').classList.add('hidden'));
      document.getElementById('add-subject-btn').addEventListener('click', () => {
        const input = document.getElementById('custom-subject-input');
        const value = input.value.trim();
        if (value) {
          addSelectedSubjectName(value);
          input.value = '';
        }
      });

      // 맞춤법 검사 에어 닫기
      document.getElementById('ai-feedback-close-btn').addEventListener('click', () => {
        document.getElementById('ai-feedback-container').classList.add('hidden');
      });

      // 희망 학과 실시간 동기화
      document.getElementById('target-dept-input').addEventListener('input', (e) => {
        const value = e.target.value;
        state.selectedDept = value;
        document.getElementById('sidebar-dept-text').textContent = value;
        state.allGradesData[state.grade].targetDept = value;
      });

      // 식별용 ID 설정 및 내신 공유 동적 동기화
      document.getElementById('custom-id-input').addEventListener('input', (e) => {
        state.customId = e.target.value;
        setDoc(doc(db, 'artifacts', appId, 'users', state.user.uid, 'profile', 'settings'), {
          customId: state.customId,
          shareGrades: state.shareGrades,
          userId: state.user.uid,
          lastUpdated: new Date().toISOString()
        }, { merge: true });
      });

      document.getElementById('share-grades-checkbox').addEventListener('change', (e) => {
        state.shareGrades = e.target.checked;
        setDoc(doc(db, 'artifacts', appId, 'users', state.user.uid, 'profile', 'settings'), {
          customId: state.customId,
          shareGrades: state.shareGrades,
          userId: state.user.uid,
          lastUpdated: new Date().toISOString()
        }, { merge: true });
      });

      // 바이트 단위 변경 대응 리렌더
      document.getElementById('byte-calc-select').addEventListener('change', (e) => {
        state.byteCalcMode = e.target.value;
        renderFixedFields();
        renderSubjectsList();
      });

      // 커뮤니티 엔진 연동 마운트
      document.getElementById('create-room-form').addEventListener('submit', handleCreateGroupRoom);
      document.getElementById('search-room-btn').addEventListener('click', () => handleSearchRooms());
      document.getElementById('chat-send-form').addEventListener('submit', handleSendChatMessage);
      document.getElementById('exit-chat-room-btn').addEventListener('click', () => {
        if (chatUnsub) chatUnsub();
        if (membersUnsub) membersUnsub();
        state.activeRoom = null;
        document.getElementById('active-chat-wrapper').classList.add('hidden');
        document.getElementById('group-search-wrapper').classList.remove('hidden');
      });

      // 3. 비밀번호 보임/안보임 이펙터
      document.getElementById('toggle-password-visibility').addEventListener('click', () => {
        const passInput = document.getElementById('auth-password');
        const icon = document.querySelector('#toggle-password-visibility i');
        if (passInput.type === 'password') {
          passInput.type = 'text';
          icon.setAttribute('data-lucide', 'eye-off');
        } else {
          passInput.type = 'password';
          icon.setAttribute('data-lucide', 'eye');
        }
        lucide.createIcons();
      });

      // 4. 모바일 화면 리사이즈 시 차트 재배열 동기화
      window.addEventListener('resize', () => {
        if (state.activeTab === 'grades') {
          drawChart();
        }
      });

      // 5. 초기 인증 상태 체크
      onAuthStateChanged(auth, (currentUser) => {
        if (currentUser) {
          state.user = currentUser;
          document.getElementById('auth-section').classList.add('hidden');
          document.getElementById('main-dashboard').classList.remove('hidden');
          
          initCloudSyncListeners(currentUser);
          switchTab('record');
        } else {
          document.getElementById('main-dashboard').classList.add('hidden');
          document.getElementById('auth-section').classList.remove('hidden');
          switchAuthLayout('login');
        }
      });

      // 아이콘 드로잉 트리거
      lucide.createIcons();
    });

    // 테마 전환 이펙트
    function toggleTheme() {
      const html = document.documentElement;
      const themeIcon = document.getElementById('theme-icon');
      if (html.classList.contains('dark')) {
        html.classList.remove('dark');
        themeIcon.setAttribute('data-lucide', 'moon');
        localStorage.setItem('theme', 'light');
      } else {
        html.classList.add('dark');
        themeIcon.setAttribute('data-lucide', 'sun');
        localStorage.setItem('theme', 'dark');
      }
      lucide.createIcons();
      if (state.activeTab === 'grades') {
        drawChart();
      }
    }

    // 초기 테마 체크 마운트
    (function initTheme() {
      const savedTheme = localStorage.getItem('theme');
      const html = document.documentElement;
      const themeIcon = document.getElementById('theme-icon');
      if (savedTheme === 'dark' || (!savedTheme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
        html.classList.add('dark');
        if (themeIcon) themeIcon.setAttribute('data-lucide', 'sun');
      } else {
        html.classList.remove('dark');
        if (themeIcon) themeIcon.setAttribute('data-lucide', 'moon');
      }
    })();
  </script>
</body>
</html>

```
