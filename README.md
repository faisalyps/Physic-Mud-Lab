<!doctype html>
<html lang="en" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Physics Mud Lab</title>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body {
      box-sizing: border-box;
    }
    
    @keyframes float {
      0%, 100% { transform: translateY(0px); }
      50% { transform: translateY(-10px); }
    }
    
    @keyframes pulse-glow {
      0%, 100% { box-shadow: 0 0 20px rgba(59, 130, 246, 0.5); }
      50% { box-shadow: 0 0 40px rgba(59, 130, 246, 0.8); }
    }
    
    @keyframes ripple {
      0% { transform: scale(1); opacity: 1; }
      100% { transform: scale(1.5); opacity: 0; }
    }
    
    @keyframes walk {
      0%, 100% { transform: translateY(0px) rotate(0deg); }
      25% { transform: translateY(-5px) rotate(-2deg); }
      75% { transform: translateY(-3px) rotate(2deg); }
    }
    
    @keyframes bounce {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.05); }
    }
    
    .float-animation {
      animation: float 3s ease-in-out infinite;
    }
    
    .pulse-glow {
      animation: pulse-glow 2s ease-in-out infinite;
    }
    
    .walk-animation {
      animation: walk 0.5s ease-in-out infinite;
    }
    
    .bounce-animation {
      animation: bounce 0.6s ease-in-out infinite;
    }
    
    .mud-surface {
      background: linear-gradient(180deg, 
        #6B4423 0%,
        #5D3A1A 20%,
        #4A2F15 40%,
        #3D2510 60%,
        #2F1D0C 80%,
        #1F1308 100%
      );
      position: relative;
      box-shadow: inset 0 -20px 40px rgba(0,0,0,0.5);
    }
    
    .mud-surface::before {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      height: 50%;
      background: repeating-linear-gradient(
        90deg,
        transparent,
        transparent 3px,
        rgba(0,0,0,0.1) 3px,
        rgba(0,0,0,0.1) 6px
      );
    }
    
    .mud-surface::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      height: 3px;
      background: linear-gradient(90deg, 
        rgba(139, 69, 19, 0.8),
        rgba(101, 67, 33, 0.8),
        rgba(139, 69, 19, 0.8)
      );
      box-shadow: 0 2px 10px rgba(0,0,0,0.5);
    }
    
    .grass-texture {
      background: linear-gradient(180deg, #4CAF50 0%, #2E7D32 100%);
      position: relative;
    }
    
    .grass-texture::before {
      content: '🌿';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      font-size: 12px;
      line-height: 12px;
      opacity: 0.6;
      text-shadow: 1px 1px 2px rgba(0,0,0,0.3);
    }
    
    .sky-background {
      background: linear-gradient(180deg, #87CEEB 0%, #B0E0E6 100%);
    }
    
    .object-shadow {
      filter: drop-shadow(2px 4px 8px rgba(0,0,0,0.4));
    }
    
    .ripple-effect {
      position: absolute;
      border: 3px solid rgba(139, 69, 19, 0.6);
      border-radius: 50%;
      animation: ripple 1s ease-out;
      pointer-events: none;
    }
    
    .lock-overlay {
      background: rgba(0,0,0,0.7);
      backdrop-filter: blur(4px);
    }
    
    .checkmark {
      display: inline-block;
      animation: checkmark-pop 0.3s ease-out;
    }
    
    @keyframes checkmark-pop {
      0% { transform: scale(0); }
      50% { transform: scale(1.2); }
      100% { transform: scale(1); }
    }
    
    .fullscreen-warning {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      z-index: 9999;
      background: white;
      padding: 2rem;
      border-radius: 1rem;
      box-shadow: 0 20px 60px rgba(0,0,0,0.3);
      text-align: center;
      max-width: 90%;
    }
    
    .blur-background {
      filter: blur(10px);
      pointer-events: none;
    }
    
    .draggable {
      cursor: grab;
      user-select: none;
    }
    
    .draggable:active {
      cursor: grabbing;
    }
    
    .block-shape {
      border-radius: 8px;
      box-shadow: 
        inset -4px -4px 8px rgba(0,0,0,0.3),
        inset 4px 4px 8px rgba(255,255,255,0.1),
        0 4px 12px rgba(0,0,0,0.4);
    }
    
    .person-svg {
      filter: drop-shadow(2px 4px 6px rgba(0,0,0,0.3));
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
 </head>
 <body class="h-full overflow-auto">
  <div id="fullscreen-prompt" class="fullscreen-warning" style="display: none;">
   <div class="text-6xl mb-4">
    🔒
   </div>
   <h2 class="text-2xl font-bold mb-4">Mode Fokus Diperlukan</h2>
   <p class="text-gray-700 mb-6">Untuk pengalaman terbaik, aplikasi akan masuk mode layar penuh dan mencegah perpindahan tab.</p><button id="enter-fullscreen-btn" class="px-8 py-4 bg-blue-600 text-white rounded-lg text-xl font-semibold hover:bg-blue-700 transition-all transform hover:scale-105 active:scale-95"> Mulai Sekarang 🚀 </button>
  </div>
  <div id="app" class="w-full h-full"></div>
  <script>
    const defaultConfig = {
      main_title: "Physics Mud Lab",
      subtitle: "Explore pressure through interactive challenges",
      next_button_text: "Next",
      previous_button_text: "Previous",
      predict_button_text: "Predict",
      reset_button_text: "Reset",
      background_color: "#EFF6FF",
      surface_color: "#FFFFFF",
      text_color: "#1E293B",
      primary_action_color: "#3B82F6",
      secondary_action_color: "#64748B",
      font_family: "Inter",
      font_size: 16
    };

    let state = {
      currentSlide: 0,
      score: 0,
      activeSimulation: null,
      selectedOption: null,
      simulationState: 'idle',
      positionX: 0,
      sinkLevel: 0,
      feedback: null,
      quizAnswered: false,
      numerator: null,
      denominator: null,
      showFormulaResult: false,
      animationFrame: null,
      completedChallenges: {},
      isFullscreen: false,
      showFullscreenPrompt: true,
      interactiveAnswers: {},
      isDragging: false,
      dragObject: null,
      dragStartX: 0,
      dragCurrentX: 0
    };

    const slides = [
      { id: 0, title: "Zona 1: Tantangan Luas (A)", type: "challenge", scenarios: ['manusia', 'balok'] },
      { 
        id: 1, 
        title: "Kesimpulan: Luas", 
        type: "interactive_quiz",
        questions: [
          { q: "Semakin besar luas permukaan bidang sentuh, tekanan yang dihasilkan semakin .......", options: ['besar', 'kecil'], correct: 'kecil' },
          { q: "Semakin kecil luas permukaan bidang sentuh, tekanan yang dihasilkan semakin .......", options: ['besar', 'kecil'], correct: 'besar' }
        ]
      },
      { id: 2, title: "Zona 2: Tantangan Gaya (F)", type: "challenge", scenarios: ['kendaraan', 'muatan'] },
      { 
        id: 3, 
        title: "Kesimpulan: Gaya", 
        type: "interactive_quiz",
        questions: [
          { q: "Semakin besar gaya yang diberikan, tekanan yang dihasilkan semakin .......", options: ['besar', 'kecil'], correct: 'besar' },
          { q: "Semakin kecil gaya yang diberikan, tekanan yang dihasilkan semakin .......", options: ['besar', 'kecil'], correct: 'kecil' }
        ]
      },
      { id: 4, title: "Misi Akhir: Rumus", type: "formula" }
    ];

    const simulationData = {
      manusia: {
        title: "Misi 1: Manusia",
        options: [
          { id: 'boots', label: 'Boots', icon: 'person_boots', desc: 'Alas Luas', result: 'safe', width: 80, height: 50, visualWidth: 80, visualHeight: 100 },
          { id: 'heels', label: 'Heels', icon: 'person_heels', desc: 'Alas Sempit', result: 'sink', width: 25, height: 50, visualWidth: 80, visualHeight: 100 }
        ]
      },
      balok: {
        title: "Misi 2: Balok",
        options: [
          { id: 'wide', label: 'Balok Lebar', icon: 'wide_block', desc: 'Posisi Lebar', result: 'safe', width: 100, height: 40, visualWidth: 100, visualHeight: 40 },
          { id: 'narrow', label: 'Balok Sempit', icon: 'narrow_block', desc: 'Posisi Sempit', result: 'sink', width: 30, height: 100, visualWidth: 30, visualHeight: 100 }
        ]
      },
      kendaraan: {
        title: "Misi 3: Kendaraan",
        options: [
          { id: 'bike', label: 'Sepeda', icon: 'bike_svg', desc: 'Ringan', result: 'safe', width: 40, height: 45, visualWidth: 100, visualHeight: 80 },
          { id: 'truck', label: 'Truk', icon: 'truck_svg', desc: 'Berat', result: 'sink', width: 80, height: 50, visualWidth: 120, visualHeight: 80 }
        ]
      },
      muatan: {
        title: "Misi 4: Muatan",
        options: [
          { id: 'empty', label: 'Kosong', icon: 'cart_empty_svg', desc: 'Ringan', result: 'safe', width: 40, height: 40, visualWidth: 80, visualHeight: 80 },
          { id: 'full', label: 'Penuh', icon: 'cart_full_svg', desc: 'Berat', result: 'sink', width: 40, height: 40, visualWidth: 80, visualHeight: 80 }
        ]
      }
    };

    function renderPersonSVG(type) {
      const primaryColor = window.elementSdk?.config?.primary_action_color || defaultConfig.primary_action_color;
      
      if (type === 'person_boots') {
        return `
          <svg width="80" height="100" viewBox="0 0 80 100" class="person-svg">
            <!-- Head -->
            <circle cx="40" cy="15" r="12" fill="#FFD6A5" stroke="#333" stroke-width="2"/>
            <!-- Body -->
            <rect x="30" y="27" width="20" height="30" rx="5" fill="${primaryColor}" stroke="#333" stroke-width="2"/>
            <!-- Arms -->
            <rect x="20" y="30" width="8" height="25" rx="4" fill="#FFD6A5" stroke="#333" stroke-width="1.5"/>
            <rect x="52" y="30" width="8" height="25" rx="4" fill="#FFD6A5" stroke="#333" stroke-width="1.5"/>
            <!-- Legs -->
            <rect x="32" y="57" width="7" height="28" rx="3" fill="#2C5AA0" stroke="#333" stroke-width="2"/>
            <rect x="41" y="57" width="7" height="28" rx="3" fill="#2C5AA0" stroke="#333" stroke-width="2"/>
            <!-- Boots (Wide base) -->
            <rect x="28" y="85" width="15" height="12" rx="3" fill="#8B4513" stroke="#333" stroke-width="2"/>
            <rect x="37" y="85" width="15" height="12" rx="3" fill="#8B4513" stroke="#333" stroke-width="2"/>
            <!-- Eyes -->
            <circle cx="36" cy="14" r="1.5" fill="#333"/>
            <circle cx="44" cy="14" r="1.5" fill="#333"/>
            <!-- Smile -->
            <path d="M 36 19 Q 40 21 44 19" stroke="#333" stroke-width="1.5" fill="none" stroke-linecap="round"/>
          </svg>
        `;
      } else if (type === 'person_heels') {
        return `
          <svg width="80" height="100" viewBox="0 0 80 100" class="person-svg">
            <!-- Head -->
            <circle cx="40" cy="15" r="12" fill="#FFD6A5" stroke="#333" stroke-width="2"/>
            <!-- Body -->
            <rect x="30" y="27" width="20" height="30" rx="5" fill="#E91E63" stroke="#333" stroke-width="2"/>
            <!-- Arms -->
            <rect x="20" y="30" width="8" height="25" rx="4" fill="#FFD6A5" stroke="#333" stroke-width="1.5"/>
            <rect x="52" y="30" width="8" height="25" rx="4" fill="#FFD6A5" stroke="#333" stroke-width="1.5"/>
            <!-- Legs -->
            <rect x="33" y="57" width="6" height="30" rx="3" fill="#FFD6A5" stroke="#333" stroke-width="2"/>
            <rect x="41" y="57" width="6" height="30" rx="3" fill="#FFD6A5" stroke="#333" stroke-width="2"/>
            <!-- Heels (Narrow base) -->
            <path d="M 33 87 L 35 95 L 32 95 L 30 87 Z" fill="#FF1744" stroke="#333" stroke-width="2"/>
            <rect x="32" y="95" width="4" height="4" fill="#333"/>
            <path d="M 41 87 L 43 95 L 40 95 L 38 87 Z" fill="#FF1744" stroke="#333" stroke-width="2"/>
            <rect x="40" y="95" width="4" height="4" fill="#333"/>
            <!-- Eyes -->
            <circle cx="36" cy="14" r="1.5" fill="#333"/>
            <circle cx="44" cy="14" r="1.5" fill="#333"/>
            <!-- Smile -->
            <path d="M 36 19 Q 40 21 44 19" stroke="#333" stroke-width="1.5" fill="none" stroke-linecap="round"/>
          </svg>
        `;
      }
    }

    function renderBlockSVG(type) {
      if (type === 'wide_block') {
        return `
          <svg width="100" height="40" viewBox="0 0 100 40">
            <defs>
              <linearGradient id="blockGrad1" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" style="stop-color:#A0522D;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#8B4513;stop-opacity:1" />
              </linearGradient>
            </defs>
            <rect class="block-shape" width="100" height="40" fill="url(#blockGrad1)" stroke="#654321" stroke-width="3"/>
            <line x1="10" y1="5" x2="90" y2="5" stroke="rgba(255,255,255,0.3)" stroke-width="2"/>
            <line x1="10" y1="35" x2="90" y2="35" stroke="rgba(0,0,0,0.3)" stroke-width="2"/>
          </svg>
        `;
      } else if (type === 'narrow_block') {
        return `
          <svg width="30" height="100" viewBox="0 0 30 100">
            <defs>
              <linearGradient id="blockGrad2" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" style="stop-color:#A0522D;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#8B4513;stop-opacity:1" />
              </linearGradient>
            </defs>
            <rect class="block-shape" width="30" height="100" fill="url(#blockGrad2)" stroke="#654321" stroke-width="3"/>
            <line x1="5" y1="10" x2="5" y2="90" stroke="rgba(255,255,255,0.3)" stroke-width="2"/>
            <line x1="25" y1="10" x2="25" y2="90" stroke="rgba(0,0,0,0.3)" stroke-width="2"/>
          </svg>
        `;
      }
    }

    function renderVehicleSVG(type) {
      const primaryColor = window.elementSdk?.config?.primary_action_color || defaultConfig.primary_action_color;
      
      if (type === 'bike_svg') {
        return `
          <svg width="100" height="80" viewBox="0 0 100 80" class="object-shadow">
            <!-- Rear wheel -->
            <circle cx="25" cy="60" r="15" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="25" cy="60" r="10" fill="#555" stroke="#000" stroke-width="1"/>
            <circle cx="25" cy="60" r="3" fill="#888"/>
            <!-- Front wheel -->
            <circle cx="75" cy="60" r="15" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="75" cy="60" r="10" fill="#555" stroke="#000" stroke-width="1"/>
            <circle cx="75" cy="60" r="3" fill="#888"/>
            <!-- Frame -->
            <line x1="25" y1="60" x2="45" y2="35" stroke="${primaryColor}" stroke-width="4" stroke-linecap="round"/>
            <line x1="45" y1="35" x2="65" y2="35" stroke="${primaryColor}" stroke-width="4" stroke-linecap="round"/>
            <line x1="65" y1="35" x2="75" y2="60" stroke="${primaryColor}" stroke-width="4" stroke-linecap="round"/>
            <line x1="45" y1="35" x2="45" y2="60" stroke="${primaryColor}" stroke-width="4" stroke-linecap="round"/>
            <!-- Seat -->
            <rect x="40" y="32" width="18" height="5" rx="2" fill="#8B4513" stroke="#654321" stroke-width="1"/>
            <!-- Handlebar -->
            <line x1="65" y1="35" x2="70" y2="28" stroke="#333" stroke-width="3" stroke-linecap="round"/>
            <line x1="68" y1="28" x2="75" y2="28" stroke="#333" stroke-width="3" stroke-linecap="round"/>
            <!-- Pedals -->
            <circle cx="45" cy="60" r="4" fill="#666" stroke="#000" stroke-width="1"/>
          </svg>
        `;
      } else if (type === 'truck_svg') {
        return `
          <svg width="120" height="80" viewBox="0 0 120 80" class="object-shadow">
            <!-- Cargo container -->
            <rect x="15" y="25" width="50" height="35" rx="3" fill="#FF6B6B" stroke="#CC5555" stroke-width="3"/>
            <line x1="20" y1="30" x2="60" y2="30" stroke="rgba(255,255,255,0.3)" stroke-width="2"/>
            <!-- Cabin -->
            <path d="M 65 35 L 65 60 L 95 60 L 100 50 L 100 35 Z" fill="${primaryColor}" stroke="#2563EB" stroke-width="3"/>
            <!-- Cabin window -->
            <path d="M 70 38 L 70 52 L 88 52 L 93 45 L 93 38 Z" fill="#87CEEB" stroke="#333" stroke-width="2"/>
            <!-- Front grill -->
            <rect x="95" y="45" width="5" height="15" fill="#333"/>
            <line x1="96" y1="47" x2="96" y2="58" stroke="#666" stroke-width="1"/>
            <line x1="98" y1="47" x2="98" y2="58" stroke="#666" stroke-width="1"/>
            <!-- Rear wheels -->
            <circle cx="30" cy="65" r="10" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="30" cy="65" r="6" fill="#555"/>
            <circle cx="50" cy="65" r="10" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="50" cy="65" r="6" fill="#555"/>
            <!-- Front wheel -->
            <circle cx="85" cy="65" r="10" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="85" cy="65" r="6" fill="#555"/>
            <!-- Headlight -->
            <circle cx="100" cy="48" r="2" fill="#FFEB3B"/>
          </svg>
        `;
      } else if (type === 'cart_empty_svg') {
        return `
          <svg width="80" height="80" viewBox="0 0 80 80" class="object-shadow">
            <!-- Cart basket -->
            <path d="M 20 25 L 25 55 L 65 55 L 70 25 Z" fill="none" stroke="${primaryColor}" stroke-width="3" stroke-linejoin="round"/>
            <line x1="25" y1="55" x2="28" y2="45" stroke="${primaryColor}" stroke-width="2"/>
            <line x1="65" y1="55" x2="62" y2="45" stroke="${primaryColor}" stroke-width="2"/>
            <line x1="28" y1="45" x2="62" y2="45" stroke="${primaryColor}" stroke-width="2"/>
            <!-- Handle -->
            <path d="M 20 25 L 15 20 L 12 20" stroke="#333" stroke-width="3" stroke-linecap="round" fill="none"/>
            <!-- Wheels -->
            <circle cx="32" cy="65" r="8" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="32" cy="65" r="4" fill="#555"/>
            <circle cx="58" cy="65" r="8" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="58" cy="65" r="4" fill="#555"/>
            <!-- Cart grid -->
            <line x1="35" y1="30" x2="37" y2="50" stroke="${primaryColor}" stroke-width="1" opacity="0.5"/>
            <line x1="45" y1="28" x2="47" y2="52" stroke="${primaryColor}" stroke-width="1" opacity="0.5"/>
            <line x1="55" y1="27" x2="57" y2="51" stroke="${primaryColor}" stroke-width="1" opacity="0.5"/>
          </svg>
        `;
      } else if (type === 'cart_full_svg') {
        return `
          <svg width="80" height="80" viewBox="0 0 80 80" class="object-shadow">
            <!-- Cart basket -->
            <path d="M 20 25 L 25 55 L 65 55 L 70 25 Z" fill="none" stroke="${primaryColor}" stroke-width="3" stroke-linejoin="round"/>
            <line x1="25" y1="55" x2="28" y2="45" stroke="${primaryColor}" stroke-width="2"/>
            <line x1="65" y1="55" x2="62" y2="45" stroke="${primaryColor}" stroke-width="2"/>
            <line x1="28" y1="45" x2="62" y2="45" stroke="${primaryColor}" stroke-width="2"/>
            <!-- Handle -->
            <path d="M 20 25 L 15 20 L 12 20" stroke="#333" stroke-width="3" stroke-linecap="round" fill="none"/>
            <!-- Items in cart -->
            <rect x="30" y="28" width="12" height="12" rx="2" fill="#8B4513" stroke="#654321" stroke-width="2"/>
            <rect x="48" y="26" width="12" height="14" rx="2" fill="#FF6B6B" stroke="#CC5555" stroke-width="2"/>
            <circle cx="40" cy="45" r="5" fill="#4CAF50" stroke="#2E7D32" stroke-width="2"/>
            <circle cx="54" cy="45" r="5" fill="#FFA726" stroke="#F57C00" stroke-width="2"/>
            <rect x="32" y="50" width="8" height="4" rx="1" fill="#9C27B0" stroke="#7B1FA2" stroke-width="1"/>
            <rect x="48" y="50" width="10" height="4" rx="1" fill="#2196F3" stroke="#1976D2" stroke-width="1"/>
            <!-- Wheels -->
            <circle cx="32" cy="65" r="8" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="32" cy="65" r="4" fill="#555"/>
            <circle cx="58" cy="65" r="8" fill="#333" stroke="#000" stroke-width="2"/>
            <circle cx="58" cy="65" r="4" fill="#555"/>
          </svg>
        `;
      }
    }

    // Fullscreen and tab lock functionality
    function requestFullscreen() {
      const elem = document.documentElement;
      if (elem.requestFullscreen) {
        elem.requestFullscreen().catch(() => {});
      } else if (elem.webkitRequestFullscreen) {
        elem.webkitRequestFullscreen();
      } else if (elem.msRequestFullscreen) {
        elem.msRequestFullscreen();
      }
    }

    function setupFullscreenAndLock() {
      state.isFullscreen = true;
      state.showFullscreenPrompt = false;
      document.getElementById('fullscreen-prompt').style.display = 'none';
      document.getElementById('app').classList.remove('blur-background');
      requestFullscreen();
      render();
    }

    // Prevent tab switching
    document.addEventListener('visibilitychange', function() {
      if (state.isFullscreen && document.hidden) {
        const notification = document.createElement('div');
        notification.style.cssText = `
          position: fixed;
          top: 20px;
          left: 50%;
          transform: translateX(-50%);
          background: #EF4444;
          color: white;
          padding: 1rem 2rem;
          border-radius: 0.5rem;
          font-weight: bold;
          z-index: 10000;
          box-shadow: 0 10px 40px rgba(0,0,0,0.3);
        `;
        notification.textContent = '⚠️ Jangan keluar dari aplikasi!';
        document.body.appendChild(notification);
        setTimeout(() => notification.remove(), 3000);
      }
    });

    document.addEventListener('contextmenu', (e) => {
      if (state.isFullscreen) e.preventDefault();
    });

    document.addEventListener('keydown', (e) => {
      if (state.isFullscreen) {
        if (e.key === 'F11' || 
            (e.ctrlKey && (e.key === 'w' || e.key === 't')) ||
            e.altKey) {
          e.preventDefault();
        }
      }
    });

    const audioContext = new (window.AudioContext || window.webkitAudioContext)();
    
    // Fungsi untuk mengacak array (Fisher-Yates shuffle)
    function shuffleArray(array) {
      const shuffled = [...array];
      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
      }
      return shuffled;
    }
    
    function playSound(type) {
      const sounds = {
        'click': { freq: 800, duration: 0.1, type: 'sine' },
        'splash': { freq: 200, duration: 0.3, type: 'sawtooth' },
        'success': { freq: 600, duration: 0.2, type: 'sine' },
        'fail': { freq: 150, duration: 0.4, type: 'sawtooth' },
        'boots_walk': { freq: 350, duration: 0.08, type: 'square' },
        'heels_walk': { freq: 800, duration: 0.06, type: 'sine' },
        'block_drag': { freq: 180, duration: 0.1, type: 'sawtooth' },
        'bike_move': { freq: 500, duration: 0.12, type: 'triangle' },
        'truck_move': { freq: 120, duration: 0.2, type: 'sawtooth' },
        'cart_empty': { freq: 450, duration: 0.1, type: 'sine' },
        'cart_full': { freq: 200, duration: 0.15, type: 'square' }
      };
      
      const sound = sounds[type];
      if (!sound) return;
      
      const oscillator = audioContext.createOscillator();
      const gainNode = audioContext.createGain();
      
      oscillator.connect(gainNode);
      gainNode.connect(audioContext.destination);
      
      oscillator.frequency.value = sound.freq;
      oscillator.type = sound.type;
      
      gainNode.gain.setValueAtTime(0.25, audioContext.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + sound.duration);
      
      oscillator.start(audioContext.currentTime);
      oscillator.stop(audioContext.currentTime + sound.duration);
    }
    
    function selectOption(simType, optionId) {
      const challengeKey = `${simType}_${optionId}`;
      if (state.completedChallenges[challengeKey]) {
        return;
      }
      
      playSound('click');
      
      state.activeSimulation = simType;
      state.selectedOption = optionId;
      state.simulationState = 'selecting';
      state.feedback = null;
      state.positionX = 0;
      state.sinkLevel = 0;
      state.isDragging = false;
      state.dragCurrentX = 0;
      state.userPrediction = null;
      render();
    }

    function makePrediction(prediction) {
      playSound('click');
      state.userPrediction = prediction;
      state.simulationState = 'predicting';
      render();
    }

    function startDrag(e) {
      if (state.simulationState !== 'predicting') return;
      
      state.isDragging = true;
      state.dragStartX = e.clientX || e.touches[0].clientX;
      state.dragObject = e.currentTarget;
      
      e.currentTarget.classList.add('walk-animation');
    }

    function onDrag(e) {
      if (!state.isDragging) return;
      
      e.preventDefault();
      const currentX = e.clientX || e.touches[0].clientX;
      const deltaX = currentX - state.dragStartX;
      
      const container = document.getElementById('drag-zone');
      if (!container) return;
      
      const containerWidth = container.offsetWidth;
      const progress = Math.max(0, Math.min(100, (deltaX / containerWidth) * 100 + 5));
      
      const lastProgress = state.dragCurrentX || 0;
      if (Math.abs(progress - lastProgress) > 5) {
        const simType = state.activeSimulation;
        const optionId = state.selectedOption;
        
        if (simType === 'manusia') {
          playSound(optionId === 'boots' ? 'boots_walk' : 'heels_walk');
        } else if (simType === 'balok') {
          playSound('block_drag');
        } else if (simType === 'kendaraan') {
          playSound(optionId === 'bike' ? 'bike_move' : 'truck_move');
        } else if (simType === 'muatan') {
          playSound(optionId === 'empty' ? 'cart_empty' : 'cart_full');
        }
      }
      
      state.dragCurrentX = progress;
      
      if (state.dragObject) {
        state.dragObject.style.left = `${progress}%`;
      }
    }

    function endDrag(e) {
      if (!state.isDragging) return;
      
      state.isDragging = false;
      
      if (state.dragObject) {
        state.dragObject.classList.remove('walk-animation');
      }
      
      runSimulation('auto');
    }

    function runSimulation(prediction) {
      state.simulationState = 'running';
      
      const data = simulationData[state.activeSimulation];
      const option = data.options.find(o => o.id === state.selectedOption);
      const actuallySinks = option.result === 'sink';

      let startX = state.dragCurrentX || 0;
      let sinkStarted = false;
      
      const animate = () => {
        startX += 1.5;
        
        let currentSink = 0;
        let shouldStop = false;
        
        if (actuallySinks && startX > 25 && !sinkStarted) {
          sinkStarted = true;
          playSound('splash');
        }
        
        if (startX > 20 && startX < 80) {
          if (actuallySinks) {
            if (startX > 25) {
              const sinkProgress = (startX - 25) / 30;
              currentSink = Math.min(sinkProgress * 180, 180);
              
              if (currentSink >= 160) {
                shouldStop = true;
              }
            }
          }
        }

        state.positionX = startX;
        state.sinkLevel = currentSink;
        render();

        if (!shouldStop && startX < 95) {
          state.animationFrame = requestAnimationFrame(animate);
        } else {
          finishSimulation(prediction, option);
        }
      };
      state.animationFrame = requestAnimationFrame(animate);
      render();
    }

    function finishSimulation(prediction, option) {
      state.simulationState = 'finished';
      const actuallySinks = option.result === 'sink';
      const userPredictedSink = state.userPrediction === 'sink';
      const predictionCorrect = (userPredictedSink && actuallySinks) || (!userPredictedSink && !actuallySinks);
      
      const explanations = {
        'manusia_boots': 'Boots memiliki luas alas yang BESAR, sehingga tekanan ke tanah KECIL. Tekanan kecil membuat boots tidak tenggelam di lumpur! 👢✓',
        'manusia_heels': 'Heels memiliki luas alas yang SEMPIT, sehingga tekanan ke tanah BESAR. Tekanan besar membuat heels tenggelam dalam di lumpur! 👠✗',
        'balok_wide': 'Balok lebar memiliki luas permukaan yang BESAR, sehingga tekanan ke tanah KECIL. Balok aman tidak tenggelam! 📦✓',
        'balok_narrow': 'Balok sempit memiliki luas permukaan yang KECIL, sehingga tekanan ke tanah BESAR. Balok tenggelam dalam! 📦✗',
        'kendaraan_bike': 'Sepeda memiliki gaya (berat) yang KECIL, sehingga tekanan ke tanah KECIL. Sepeda ringan tidak tenggelam! 🚲✓',
        'kendaraan_truck': 'Truk memiliki gaya (berat) yang BESAR, sehingga tekanan ke tanah BESAR. Truk berat tenggelam dalam! 🚛✗',
        'muatan_empty': 'Keranjang kosong memiliki gaya (berat) yang KECIL, sehingga tekanan ke tanah KECIL. Keranjang ringan aman! 🛒✓',
        'muatan_full': 'Keranjang penuh memiliki gaya (berat) yang BESAR, sehingga tekanan ke tanah BESAR. Keranjang berat tenggelam! 🛒✗'
      };
      
      const challengeKey = `${state.activeSimulation}_${state.selectedOption}`;
      const explanation = explanations[challengeKey] || '';
      
      let feedbackMessage = '';
      let feedbackType = 'success';
      
      if (predictionCorrect) {
        state.score += 20;
        feedbackMessage = `✓ PREDIKSI BENAR! Kamu memprediksi objek akan ${actuallySinks ? 'TENGGELAM' : 'AMAN'}, dan memang benar ${actuallySinks ? 'tenggelam' : 'aman'}! +20 poin 🎉`;
        playSound('success');
      } else {
        state.score += 5;
        feedbackMessage = `✗ PREDIKSI SALAH. Kamu memprediksi objek akan ${userPredictedSink ? 'TENGGELAM' : 'AMAN'}, tapi sebenarnya objek ${actuallySinks ? 'TENGGELAM' : 'AMAN'}. +5 poin`;
        feedbackType = 'warning';
        playSound('fail');
      }
      
      state.feedback = { 
        type: feedbackType, 
        message: feedbackMessage,
        explanation: explanation
      };
      state.completedChallenges[challengeKey] = true;
      
      render();
    }

    function resetSimulation() {
      if (state.animationFrame) {
        cancelAnimationFrame(state.animationFrame);
      }
      state.activeSimulation = null;
      state.selectedOption = null;
      state.simulationState = 'idle';
      state.positionX = 0;
      state.sinkLevel = 0;
      state.feedback = null;
      state.dragCurrentX = 0;
      render();
    }

    function answerInteractiveQuestion(questionIndex, answer) {
      const currentSlideData = slides[state.currentSlide];
      const question = currentSlideData.questions[questionIndex];
      
      if (!state.interactiveAnswers) {
        state.interactiveAnswers = {};
      }
      
      const correct = answer === question.correct;
      state.interactiveAnswers[questionIndex] = { answer, correct };
      
      const allAnswered = currentSlideData.questions.every((_, idx) => state.interactiveAnswers[idx]);
      const allCorrect = currentSlideData.questions.every((_, idx) => 
        state.interactiveAnswers[idx]?.correct
      );
      
      if (allAnswered && allCorrect) {
        state.score += 20;
        state.feedback = { type: 'success', message: '🎉 Sempurna! Semua jawaban benar!' };
      }
      
      render();
    }

    function nextSlide() {
      if (state.currentSlide < slides.length - 1) {
        state.currentSlide++;
        state.quizAnswered = false;
        state.feedback = null;
        state.interactiveAnswers = {};
        resetSimulation();
      }
    }

    function previousSlide() {
      if (state.currentSlide > 0) {
        state.currentSlide--;
        state.quizAnswered = false;
        state.feedback = null;
        state.interactiveAnswers = {};
        resetSimulation();
      }
    }

    function setFormulaPart(part, value) {
      if (part === 'numerator') {
        state.numerator = value;
      } else {
        state.denominator = value;
      }
      
      if (state.numerator && state.denominator) {
        const correct = state.numerator === 'F' && state.denominator === 'A';
        if (correct) {
          state.score += 30;
          state.showFormulaResult = true;
          state.feedback = { type: 'success', message: '🎉 Sempurna! Rumus P = F/A benar!' };
        } else {
          state.feedback = { type: 'error', message: '✗ Belum tepat. Coba lagi!' };
          state.numerator = null;
          state.denominator = null;
        }
      }
      render();
    }

    function renderChallengeSlide(slideData) {
      const customFont = window.elementSdk?.config?.font_family || defaultConfig.font_family;
      const baseSize = window.elementSdk?.config?.font_size || defaultConfig.font_size;
      const surfaceColor = window.elementSdk?.config?.surface_color || defaultConfig.surface_color;
      const textColor = window.elementSdk?.config?.text_color || defaultConfig.text_color;
      const primaryColor = window.elementSdk?.config?.primary_action_color || defaultConfig.primary_action_color;

      return `
        <div class="space-y-6">
          <h2 style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.75}px; color: ${textColor};" class="font-bold text-center">${slideData.title}</h2>
          
          <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            ${slideData.scenarios.map(scenario => {
              const data = simulationData[scenario];
              const isActive = state.activeSimulation === scenario;
              
              const completedCount = data.options.filter(opt => 
                state.completedChallenges[`${scenario}_${opt.id}`]
              ).length;
              const allCompleted = completedCount === data.options.length;
              
              return `
                <div style="background-color: ${surfaceColor};" class="rounded-xl shadow-lg p-6 space-y-4 relative">
                  ${allCompleted ? `
                    <div class="absolute top-4 right-4 text-4xl checkmark">✅</div>
                  ` : completedCount > 0 ? `
                    <div class="absolute top-4 right-4 text-2xl checkmark">${completedCount}/${data.options.length} ✓</div>
                  ` : ''}
                  
                  ${allCompleted && !isActive ? `
                    <div class="lock-overlay absolute inset-0 rounded-xl flex items-center justify-center z-10">
                      <div class="text-center text-white">
                        <div class="text-6xl mb-2">🔒</div>
                        <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize}px;" class="font-bold">Selesai!</p>
                      </div>
                    </div>
                  ` : ''}
                  
                  <h3 style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.25}px; color: ${textColor};" class="font-semibold text-center">${data.title}</h3>
                  
                  <div class="grid grid-cols-2 gap-4">
                    ${shuffleArray([...data.options]).map(option => {
                      const optionCompleted = state.completedChallenges[`${scenario}_${option.id}`];
                      const isSelected = state.selectedOption === option.id && state.activeSimulation === scenario;
                      
                      let preview = '';
                      if (option.icon === 'person_boots' || option.icon === 'person_heels') {
                        preview = `<div class="flex justify-center mb-2">${renderPersonSVG(option.icon)}</div>`;
                      } else if (option.icon === 'wide_block' || option.icon === 'narrow_block') {
                        preview = `<div class="flex justify-center items-center mb-2" style="min-height: 100px;">${renderBlockSVG(option.icon)}</div>`;
                      } else if (option.icon.includes('_svg')) {
                        preview = `<div class="flex justify-center items-center mb-2" style="min-height: 100px;">${renderVehicleSVG(option.icon)}</div>`;
                      } else {
                        preview = `<div class="text-5xl mb-2">${option.icon}</div>`;
                      }
                      
                      return `
                        <button 
                          onclick="selectOption('${scenario}', '${option.id}')"
                          ${optionCompleted ? 'disabled' : ''}
                          style="background-color: ${isSelected ? primaryColor : surfaceColor}; 
                                 color: ${isSelected ? '#FFFFFF' : textColor};
                                 border: 3px solid ${primaryColor};
                                 font-family: ${customFont}, Arial, sans-serif;
                                 font-size: ${baseSize * 0.9}px;
                                 position: relative;"
                          class="p-4 rounded-lg hover:opacity-80 transition-all transform hover:scale-105 active:scale-95 disabled:opacity-50 disabled:cursor-not-allowed ${isSelected ? 'bounce-animation' : ''}"
                        >
                          ${optionCompleted ? '<div class="absolute top-2 right-2 text-xl z-20">✅</div>' : ''}
                          ${preview}
                          <div class="font-semibold">${option.label}</div>
                          <div class="text-sm opacity-75">${option.desc}</div>
                          <div style="font-size: ${baseSize * 0.75}px;" class="mt-2 text-xs">Lebar: ${option.width}cm</div>
                        </button>
                      `;
                    }).join('')}
                  </div>
                </div>
              `;
            }).join('')}
          </div>
          
          ${state.activeSimulation ? renderSimulationArea() : ''}
        </div>
      `;
    }

    function renderSimulationArea() {
      const customFont = window.elementSdk?.config?.font_family || defaultConfig.font_family;
      const baseSize = window.elementSdk?.config?.font_size || defaultConfig.font_size;
      const surfaceColor = window.elementSdk?.config?.surface_color || defaultConfig.surface_color;
      const textColor = window.elementSdk?.config?.text_color || defaultConfig.text_color;
      const primaryColor = window.elementSdk?.config?.primary_action_color || defaultConfig.primary_action_color;
      const secondaryColor = window.elementSdk?.config?.secondary_action_color || defaultConfig.secondary_action_color;
      const resetText = window.elementSdk?.config?.reset_button_text || defaultConfig.reset_button_text;

      const data = simulationData[state.activeSimulation];
      const option = data.options.find(o => o.id === state.selectedOption);

      let objectHTML = '';
      if (option.icon === 'person_boots' || option.icon === 'person_heels') {
        objectHTML = renderPersonSVG(option.icon);
      } else if (option.icon === 'wide_block' || option.icon === 'narrow_block') {
        objectHTML = renderBlockSVG(option.icon);
      } else if (option.icon.includes('_svg')) {
        objectHTML = renderVehicleSVG(option.icon);
      } else {
        objectHTML = `<div style="font-size: 80px; line-height: 1;">${option.icon}</div>`;
      }

      return `
        <div style="background-color: ${surfaceColor};" class="rounded-xl shadow-2xl p-6 mt-6">
          <h3 style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.25}px; color: ${textColor};" class="font-semibold text-center mb-4">
            Simulasi: ${option.label}
          </h3>
          
          ${state.simulationState === 'selecting' ? `
            <div class="text-center space-y-6 mb-6">
              <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.3}px; color: ${textColor};" class="font-bold">
                ❓ Menurut kamu, objek ini akan...
              </p>
              <div class="flex gap-4 justify-center">
                <button 
                  onclick="makePrediction('sink')"
                  style="background: linear-gradient(135deg, #EF4444 0%, #DC2626 100%); font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; box-shadow: 0 4px 15px rgba(239, 68, 68, 0.4);"
                  class="px-10 py-5 text-white rounded-xl hover:opacity-90 transition-all transform hover:scale-105 active:scale-95 font-bold"
                >
                  🌊 TENGGELAM
                </button>
                <button 
                  onclick="makePrediction('safe')"
                  style="background: linear-gradient(135deg, #10B981 0%, #059669 100%); font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; box-shadow: 0 4px 15px rgba(16, 185, 129, 0.4);"
                  class="px-10 py-5 text-white rounded-xl hover:opacity-90 transition-all transform hover:scale-105 active:scale-95 font-bold"
                >
                  ✅ AMAN
                </button>
              </div>
            </div>
          ` : ''}
          
          ${state.simulationState === 'predicting' ? `
            <div class="text-center space-y-4 mb-6">
              <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; color: ${textColor};" class="font-semibold">
                👆 Seret objek ke kanan untuk memulai simulasi!
              </p>
            </div>
          ` : ''}
          
          ${state.simulationState === 'selecting' || state.simulationState === 'predicting' || state.simulationState === 'running' || state.simulationState === 'finished' ? `
            <div id="drag-zone" class="relative h-80 rounded-xl overflow-hidden mb-6 shadow-inner" style="border: 4px solid #8B4513;">
              <div class="sky-background absolute top-0 left-0 w-full h-2/5"></div>
              <div class="grass-texture absolute left-0 w-full" style="top: 40%; height: 10%;"></div>
              <div class="mud-surface absolute bottom-0 left-0 w-full" style="height: 50%;"></div>
              
              <div 
                id="draggable-object"
                ${state.simulationState === 'predicting' ? 'class="draggable"' : ''}
                style="position: absolute; left: ${state.positionX || 5}%; bottom: calc(50% - ${state.sinkLevel}px); transform: translateX(-50%) ${state.sinkLevel > 50 ? `rotate(${Math.min(state.sinkLevel / 3, 20)}deg)` : ''}; transition: ${state.isDragging ? 'none' : 'all 0.1s'}; z-index: 5; opacity: ${state.sinkLevel > 100 ? Math.max(0.3, 1 - (state.sinkLevel - 100) / 50) : 1}; ${state.simulationState === 'selecting' ? 'pointer-events: none;' : ''}"
                ${state.simulationState === 'predicting' ? `onmousedown="startDrag(event)" ontouchstart="startDrag(event)"` : ''}
              >
                ${objectHTML}
                ${state.sinkLevel > 10 ? `
                  <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); 
                              width: ${(option.visualWidth || option.width) * 1.5}px; height: ${(option.visualWidth || option.width) * 1.5}px;"
                       class="ripple-effect"></div>
                ` : ''}
                ${state.sinkLevel > 20 ? `
                  <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); 
                              width: ${(option.visualWidth || option.width) * 2}px; height: ${(option.visualWidth || option.width) * 2}px; animation-delay: 0.3s;"
                       class="ripple-effect"></div>
                ` : ''}
              </div>
              
              <div class="absolute bottom-2 left-2 right-2">
                <div style="background-color: rgba(255,255,255,0.3); height: 8px;" class="rounded-full overflow-hidden">
                  <div style="width: ${state.positionX || 5}%; background: linear-gradient(90deg, ${primaryColor} 0%, ${secondaryColor} 100%); height: 100%; transition: width 0.1s linear;"></div>
                </div>
              </div>
            </div>
          ` : ''}
          
          ${state.feedback ? `
            <div style="background: ${state.feedback.type === 'success' ? 'linear-gradient(135deg, #D1FAE5 0%, #A7F3D0 100%)' : state.feedback.type === 'warning' ? 'linear-gradient(135deg, #FEF3C7 0%, #FDE68A 100%)' : 'linear-gradient(135deg, #FEE2E2 0%, #FECACA 100%)'}; 
                        color: ${state.feedback.type === 'success' ? '#065F46' : state.feedback.type === 'warning' ? '#92400E' : '#991B1B'};
                        font-family: ${customFont}, Arial, sans-serif;
                        font-size: ${baseSize * 1.1}px;
                        box-shadow: 0 4px 15px ${state.feedback.type === 'success' ? 'rgba(16, 185, 129, 0.3)' : state.feedback.type === 'warning' ? 'rgba(245, 158, 11, 0.3)' : 'rgba(239, 68, 68, 0.3)'};"
                 class="p-5 rounded-lg text-center font-bold mb-4">
              <div class="mb-3">${state.feedback.message}</div>
              ${state.feedback.explanation ? `
                <div style="background: rgba(255,255,255,0.3); font-size: ${baseSize * 0.95}px; border-top: 2px solid ${state.feedback.type === 'success' ? '#059669' : state.feedback.type === 'warning' ? '#D97706' : '#DC2626'};" class="mt-4 pt-4 px-4 rounded-lg text-left">
                  <div class="font-bold mb-2">📚 Penjelasan:</div>
                  <div class="font-normal leading-relaxed">${state.feedback.explanation}</div>
                </div>
              ` : ''}
            </div>
          ` : ''}
          
          ${state.simulationState === 'finished' ? `
            <div class="text-center">
              <button 
                onclick="resetSimulation()"
                style="background-color: ${secondaryColor}; font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize}px;"
                class="px-8 py-3 text-white rounded-lg hover:opacity-80 transition-all transform hover:scale-105 active:scale-95"
              >
                ${resetText}
              </button>
            </div>
          ` : ''}
        </div>
      `;
    }

    function renderInteractiveQuizSlide(slideData) {
      const customFont = window.elementSdk?.config?.font_family || defaultConfig.font_family;
      const baseSize = window.elementSdk?.config?.font_size || defaultConfig.font_size;
      const surfaceColor = window.elementSdk?.config?.surface_color || defaultConfig.surface_color;
      const textColor = window.elementSdk?.config?.text_color || defaultConfig.text_color;
      const primaryColor = window.elementSdk?.config?.primary_action_color || defaultConfig.primary_action_color;

      if (!state.interactiveAnswers) {
        state.interactiveAnswers = {};
      }

      return `
        <div style="background-color: ${surfaceColor}; box-shadow: 0 20px 60px rgba(0,0,0,0.15);" class="rounded-xl p-8 max-w-3xl mx-auto">
          <h2 style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.75}px; color: ${textColor};" class="font-bold text-center mb-8">
            ${slideData.title}
          </h2>
          
          <div style="background: linear-gradient(135deg, ${primaryColor}15 0%, ${primaryColor}25 100%); border-left: 5px solid ${primaryColor};" class="p-6 rounded-lg mb-8">
            <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; color: ${textColor};" class="font-bold mb-2 flex items-center gap-2">
              <span class="text-2xl">💡</span> Jawab pertanyaan berikut:
            </p>
          </div>
          
          <div class="space-y-8">
            ${slideData.questions.map((question, idx) => {
              const answered = state.interactiveAnswers[idx];
              return `
                <div style="background: ${answered ? (answered.correct ? 'linear-gradient(135deg, #D1FAE5 0%, #A7F3D0 100%)' : 'linear-gradient(135deg, #FEE2E2 0%, #FECACA 100%)') : '#F8FAFC'}; border: 3px solid ${answered ? (answered.correct ? '#10B981' : '#EF4444') : primaryColor};" class="p-6 rounded-xl transition-all">
                  <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.05}px; color: ${textColor};" class="font-semibold mb-4">
                    ${idx + 1}. ${question.q}
                  </p>
                  <div class="flex gap-4 justify-center">
                    ${shuffleArray([...question.options]).map(option => `
                      <button 
                        onclick="answerInteractiveQuestion(${idx}, '${option}')"
                        ${answered ? 'disabled' : ''}
                        style="background: ${answered?.answer === option ? (answered.correct ? 'linear-gradient(135deg, #10B981 0%, #059669 100%)' : 'linear-gradient(135deg, #EF4444 0%, #DC2626 100%)') : `linear-gradient(135deg, ${primaryColor} 0%, ${primaryColor}dd 100%)`}; 
                               font-family: ${customFont}, Arial, sans-serif; 
                               font-size: ${baseSize}px;
                               box-shadow: 0 4px 15px ${answered?.answer === option ? (answered.correct ? 'rgba(16, 185, 129, 0.4)' : 'rgba(239, 68, 68, 0.4)') : `${primaryColor}40`};"
                        class="px-8 py-4 text-white rounded-lg font-bold hover:opacity-90 transition-all transform hover:scale-105 active:scale-95 disabled:cursor-not-allowed disabled:transform-none"
                      >
                        ${answered?.answer === option ? (answered.correct ? '✓ ' : '✗ ') : ''}${option.toUpperCase()}
                      </button>
                    `).join('')}
                  </div>
                  ${answered && !answered.correct ? `
                    <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 0.9}px; color: #991B1B;" class="text-center mt-3 font-semibold">
                      Coba lagi!
                    </p>
                  ` : answered && answered.correct ? `
                    <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 0.9}px; color: #065F46;" class="text-center mt-3 font-semibold">
                      ✓ Benar!
                    </p>
                  ` : ''}
                </div>
              `;
            }).join('')}
          </div>
          
          ${state.feedback ? `
            <div style="background: linear-gradient(135deg, #D1FAE5 0%, #A7F3D0 100%); 
                        color: #065F46;
                        font-family: ${customFont}, Arial, sans-serif;
                        font-size: ${baseSize * 1.2}px;
                        box-shadow: 0 10px 30px rgba(16, 185, 129, 0.3);"
                 class="mt-8 p-6 rounded-xl text-center font-bold">
              ${state.feedback.message}
            </div>
          ` : ''}
        </div>
      `;
    }

    function renderFormulaSlide() {
      const customFont = window.elementSdk?.config?.font_family || defaultConfig.font_family;
      const baseSize = window.elementSdk?.config?.font_size || defaultConfig.font_size;
      const surfaceColor = window.elementSdk?.config?.surface_color || defaultConfig.surface_color;
      const textColor = window.elementSdk?.config?.text_color || defaultConfig.text_color;
      const primaryColor = window.elementSdk?.config?.primary_action_color || defaultConfig.primary_action_color;

      return `
        <div style="background-color: ${surfaceColor}; box-shadow: 0 20px 60px rgba(0,0,0,0.15);" class="rounded-xl p-8 max-w-3xl mx-auto">
          <h2 style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 2}px; color: ${textColor};" class="font-bold text-center mb-6">
            🎯 Misi Akhir: Rumus Tekanan
          </h2>
          
          <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; color: ${textColor};" class="text-center mb-8">
            Susun rumus tekanan dengan benar!
          </p>
          
          <div class="flex justify-center items-center gap-6 mb-10">
            <div style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 2.5}px; color: ${textColor};" class="font-bold">
              P =
            </div>
            
            <div style="border: 4px solid ${primaryColor}; box-shadow: 0 10px 40px ${primaryColor}30;" class="flex flex-col items-center p-6 rounded-2xl bg-white">
              <div style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 2}px; color: ${textColor};" class="min-w-[100px] min-h-[80px] flex items-center justify-center font-bold mb-3">
                ${state.numerator || '?'}
              </div>
              <div style="border-top: 4px solid ${textColor};" class="w-full"></div>
              <div style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 2}px; color: ${textColor};" class="min-w-[100px] min-h-[80px] flex items-center justify-center font-bold mt-3">
                ${state.denominator || '?'}
              </div>
            </div>
          </div>
          
          <div class="grid grid-cols-2 gap-8 mb-6">
            <div class="space-y-4">
              <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; color: ${textColor};" class="font-bold text-center">
                Pilih Pembilang:
              </p>
              <button 
                onclick="setFormulaPart('numerator', 'F')"
                style="background: ${state.numerator === 'F' ? `linear-gradient(135deg, ${primaryColor} 0%, ${primaryColor}dd 100%)` : 'white'}; 
                       color: ${state.numerator === 'F' ? 'white' : textColor};
                       border: 3px solid ${primaryColor};
                       font-family: ${customFont}, Arial, sans-serif;
                       font-size: ${baseSize * 1.5}px;
                       box-shadow: 0 4px 15px ${state.numerator === 'F' ? `${primaryColor}50` : 'rgba(0,0,0,0.1)'};"
                class="w-full p-6 rounded-xl font-bold hover:opacity-80 transition-all transform hover:scale-105 active:scale-95"
              >
                F (Gaya)
              </button>
              <button 
                onclick="setFormulaPart('numerator', 'A')"
                style="background: ${state.numerator === 'A' ? `linear-gradient(135deg, ${primaryColor} 0%, ${primaryColor}dd 100%)` : 'white'}; 
                       color: ${state.numerator === 'A' ? 'white' : textColor};
                       border: 3px solid ${primaryColor};
                       font-family: ${customFont}, Arial, sans-serif;
                       font-size: ${baseSize * 1.5}px;
                       box-shadow: 0 4px 15px ${state.numerator === 'A' ? `${primaryColor}50` : 'rgba(0,0,0,0.1)'};"
                class="w-full p-6 rounded-xl font-bold hover:opacity-80 transition-all transform hover:scale-105 active:scale-95"
              >
                A (Luas)
              </button>
            </div>
            
            <div class="space-y-4">
              <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; color: ${textColor};" class="font-bold text-center">
                Pilih Penyebut:
              </p>
              <button 
                onclick="setFormulaPart('denominator', 'F')"
                style="background: ${state.denominator === 'F' ? `linear-gradient(135deg, ${primaryColor} 0%, ${primaryColor}dd 100%)` : 'white'}; 
                       color: ${state.denominator === 'F' ? 'white' : textColor};
                       border: 3px solid ${primaryColor};
                       font-family: ${customFont}, Arial, sans-serif;
                       font-size: ${baseSize * 1.5}px;
                       box-shadow: 0 4px 15px ${state.denominator === 'F' ? `${primaryColor}50` : 'rgba(0,0,0,0.1)'};"
                class="w-full p-6 rounded-xl font-bold hover:opacity-80 transition-all transform hover:scale-105 active:scale-95"
              >
                F (Gaya)
              </button>
              <button 
                onclick="setFormulaPart('denominator', 'A')"
                style="background: ${state.denominator === 'A' ? `linear-gradient(135deg, ${primaryColor} 0%, ${primaryColor}dd 100%)` : 'white'}; 
                       color: ${state.denominator === 'A' ? 'white' : textColor};
                       border: 3px solid ${primaryColor};
                       font-family: ${customFont}, Arial, sans-serif;
                       font-size: ${baseSize * 1.5}px;
                       box-shadow: 0 4px 15px ${state.denominator === 'A' ? `${primaryColor}50` : 'rgba(0,0,0,0.1)'};"
                class="w-full p-6 rounded-xl font-bold hover:opacity-80 transition-all transform hover:scale-105 active:scale-95"
              >
                A (Luas)
              </button>
            </div>
          </div>
          
          ${state.feedback ? `
            <div style="background: ${state.feedback.type === 'success' ? 'linear-gradient(135deg, #D1FAE5 0%, #A7F3D0 100%)' : 'linear-gradient(135deg, #FEE2E2 0%, #FECACA 100%)'}; 
                        color: ${state.feedback.type === 'success' ? '#065F46' : '#991B1B'};
                        font-family: ${customFont}, Arial, sans-serif;
                        font-size: ${baseSize * 1.1}px;"
                 class="p-5 rounded-lg text-center font-bold">
              ${state.feedback.message}
            </div>
          ` : ''}
          
          ${state.showFormulaResult ? `
            <div style="background: linear-gradient(135deg, #FEF3C7 0%, #FDE68A 100%); border: 4px solid #F59E0B; font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; color: ${textColor};" class="mt-6 p-8 rounded-2xl text-center">
              <div class="text-6xl mb-4">🎓</div>
              <p class="font-bold mb-3 text-2xl">Rumus Tekanan:</p>
              <p class="text-4xl font-bold mb-4">P = F / A</p>
              <p class="text-lg font-semibold">P = Tekanan (Pascal)</p>
              <p class="text-lg font-semibold">F = Gaya (Newton)</p>
              <p class="text-lg font-semibold">A = Luas Penampang (m²)</p>
              <div class="mt-6 text-3xl">🏆 Skor Akhir: ${state.score}</div>
            </div>
          ` : ''}
        </div>
      `;
    }

    function render() {
      const customFont = window.elementSdk?.config?.font_family || defaultConfig.font_family;
      const baseSize = window.elementSdk?.config?.font_size || defaultConfig.font_size;
      const bgColor = window.elementSdk?.config?.background_color || defaultConfig.background_color;
      const surfaceColor = window.elementSdk?.config?.surface_color || defaultConfig.surface_color;
      const textColor = window.elementSdk?.config?.text_color || defaultConfig.text_color;
      const primaryColor = window.elementSdk?.config?.primary_action_color || defaultConfig.primary_action_color;
      const secondaryColor = window.elementSdk?.config?.secondary_action_color || defaultConfig.secondary_action_color;
      const mainTitle = window.elementSdk?.config?.main_title || defaultConfig.main_title;
      const subtitle = window.elementSdk?.config?.subtitle || defaultConfig.subtitle;
      const nextText = window.elementSdk?.config?.next_button_text || defaultConfig.next_button_text;
      const prevText = window.elementSdk?.config?.previous_button_text || defaultConfig.previous_button_text;

      const currentSlideData = slides[state.currentSlide];
      
      let slideContent = '';
      if (currentSlideData.type === 'challenge') {
        slideContent = renderChallengeSlide(currentSlideData);
      } else if (currentSlideData.type === 'interactive_quiz') {
        slideContent = renderInteractiveQuizSlide(currentSlideData);
      } else if (currentSlideData.type === 'formula') {
        slideContent = renderFormulaSlide();
      }

      const appElement = document.getElementById('app');
      if (state.showFullscreenPrompt) {
        appElement.classList.add('blur-background');
      }

      appElement.innerHTML = `
        <div class="max-w-6xl mx-auto p-6 h-full flex flex-col">
          <header style="background: linear-gradient(135deg, ${primaryColor} 0%, ${secondaryColor} 100%); box-shadow: 0 10px 40px ${primaryColor}40;" class="rounded-2xl p-8 mb-8 text-white text-center flex-shrink-0">
            <h1 style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 2.5}px;" class="font-bold mb-2">
              ${mainTitle}
            </h1>
            <p style="font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px;" class="opacity-90">
              ${subtitle}
            </p>
            <div class="mt-6 flex justify-center items-center gap-6">
              <div style="background-color: rgba(255,255,255,0.25); font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; box-shadow: 0 4px 15px rgba(0,0,0,0.2);" class="px-6 py-3 rounded-xl backdrop-blur-sm">
                <span class="font-bold">🏆 Skor: ${state.score}</span>
              </div>
              <div style="background-color: rgba(255,255,255,0.25); font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 1.1}px; box-shadow: 0 4px 15px rgba(0,0,0,0.2);" class="px-6 py-3 rounded-xl backdrop-blur-sm">
                <span class="font-bold">📍 Zona ${state.currentSlide + 1} / ${slides.length}</span>
              </div>
            </div>
          </header>
          
          <main class="mb-8 flex-grow overflow-auto">
            ${slideContent}
          </main>
          
          <nav class="flex justify-between items-center flex-shrink-0">
            <button 
              onclick="previousSlide()"
              ${state.currentSlide === 0 ? 'disabled' : ''}
              style="background: linear-gradient(135deg, ${secondaryColor} 0%, ${secondaryColor}dd 100%); font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize}px; box-shadow: 0 4px 15px ${secondaryColor}40;"
              class="px-8 py-4 text-white rounded-xl hover:opacity-90 transition-all disabled:opacity-30 disabled:cursor-not-allowed transform hover:scale-105 active:scale-95 font-semibold"
            >
              ← ${prevText}
            </button>
            
            <div style="background-color: ${surfaceColor}; font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize * 0.9}px; color: ${textColor}; box-shadow: 0 4px 15px rgba(0,0,0,0.1);" class="px-6 py-3 rounded-xl font-semibold">
              ${currentSlideData.title}
            </div>
            
            <button 
              onclick="nextSlide()"
              ${state.currentSlide === slides.length - 1 ? 'disabled' : ''}
              style="background: linear-gradient(135deg, ${primaryColor} 0%, ${primaryColor}dd 100%); font-family: ${customFont}, Arial, sans-serif; font-size: ${baseSize}px; box-shadow: 0 4px 15px ${primaryColor}40;"
              class="px-8 py-4 text-white rounded-xl hover:opacity-90 transition-all disabled:opacity-30 disabled:cursor-not-allowed transform hover:scale-105 active:scale-95 font-semibold"
            >
              ${nextText} →
            </button>
          </nav>
        </div>
      `;

      document.body.style.background = `linear-gradient(135deg, ${bgColor} 0%, ${primaryColor}15 100%)`;
      
      if (state.simulationState === 'predicting') {
        document.addEventListener('mousemove', onDrag);
        document.addEventListener('mouseup', endDrag);
        document.addEventListener('touchmove', onDrag);
        document.addEventListener('touchend', endDrag);
      }
    }

    async function onConfigChange(config) {
      render();
    }

    if (window.elementSdk) {
      window.elementSdk.init({
        defaultConfig,
        onConfigChange,
        mapToCapabilities: (config) => ({
          recolorables: [
            {
              get: () => config.background_color || defaultConfig.background_color,
              set: (value) => {
                if (window.elementSdk.config) {
                  window.elementSdk.config.background_color = value;
                  window.elementSdk.setConfig({ background_color: value });
                }
              }
            },
            {
              get: () => config.surface_color || defaultConfig.surface_color,
              set: (value) => {
                if (window.elementSdk.config) {
                  window.elementSdk.config.surface_color = value;
                  window.elementSdk.setConfig({ surface_color: value });
                }
              }
            },
            {
              get: () => config.text_color || defaultConfig.text_color,
              set: (value) => {
                if (window.elementSdk.config) {
                  window.elementSdk.config.text_color = value;
                  window.elementSdk.setConfig({ text_color: value });
                }
              }
            },
            {
              get: () => config.primary_action_color || defaultConfig.primary_action_color,
              set: (value) => {
                if (window.elementSdk.config) {
                  window.elementSdk.config.primary_action_color = value;
                  window.elementSdk.setConfig({ primary_action_color: value });
                }
              }
            },
            {
              get: () => config.secondary_action_color || defaultConfig.secondary_action_color,
              set: (value) => {
                if (window.elementSdk.config) {
                  window.elementSdk.config.secondary_action_color = value;
                  window.elementSdk.setConfig({ secondary_action_color: value });
                }
              }
            }
          ],
          borderables: [],
          fontEditable: {
            get: () => config.font_family || defaultConfig.font_family,
            set: (value) => {
              if (window.elementSdk.config) {
                window.elementSdk.config.font_family = value;
                window.elementSdk.setConfig({ font_family: value });
              }
            }
          },
          fontSizeable: {
            get: () => config.font_size || defaultConfig.font_size,
            set: (value) => {
              if (window.elementSdk.config) {
                window.elementSdk.config.font_size = value;
                window.elementSdk.setConfig({ font_size: value });
              }
            }
          }
        }),
        mapToEditPanelValues: (config) => new Map([
          ['main_title', config.main_title || defaultConfig.main_title],
          ['subtitle', config.subtitle || defaultConfig.subtitle],
          ['next_button_text', config.next_button_text || defaultConfig.next_button_text],
          ['previous_button_text', config.previous_button_text || defaultConfig.previous_button_text],
          ['predict_button_text', config.predict_button_text || defaultConfig.predict_button_text],
          ['reset_button_text', config.reset_button_text || defaultConfig.reset_button_text]
        ])
      });
    }

    document.getElementById('enter-fullscreen-btn').addEventListener('click', setupFullscreenAndLock);
    document.getElementById('fullscreen-prompt').style.display = 'block';
    
    render();
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9b46f0ae5783d8a3',t:'MTc2NjgxODgxOC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
