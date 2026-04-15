# 🚀 Rocket Shooter - Space Learning Game

โปรเจกต์เกมยิงจรวดในอวกาศ พร้อมสอดแทรกความรู้เรื่องดาวเคราะห์ในระบบสุริยะ พัฒนาด้วย HTML5, Tailwind CSS และ JavaScript

## 🎮 ฟีเจอร์ของเกม
* **ระบบล็อกอิน:** รองรับการสร้างบัญชีและเข้าสู่ระบบ
* **ระดับความยาก:** เลือกเล่นได้ 4 ระดับ (Easy, Normal, Hard, และ Boss Mode)
* **สารานุกรมอวกาศ:** หน้าเรียนรู้ข้อมูลดาวเคราะห์ทั้ง 8 ดวงในระบบสุริยะ
* **ลีดเดอร์บอร์ด:** ระบบแสดงลำดับคะแนนผู้เล่นสูงสุด

## 💻 Source Code (HTML/JavaScript)

```html
<!DOCTYPE html>
<html lang="th" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Rocket Shooter</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="/_sdk/data_sdk.js"></script>
  <style>
    body {
      box-sizing: border-box;
      touch-action: none;
      overflow: hidden;
    }
    @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap');
    
    .game-font {
      font-family: 'Orbitron', sans-serif;
    }
    
    @keyframes pulse-glow {
      0%, 100% { filter: drop-shadow(0 0 10px currentColor); }
      50% { filter: drop-shadow(0 0 20px currentColor); }
    }
    
    @keyframes shake {
      0%, 100% { transform: translateX(0); }
      25% { transform: translateX(-5px); }
      75% { transform: translateX(5px); }
    }
    
    .shake {
      animation: shake 0.1s ease-in-out;
    }
    
    @keyframes star-twinkle {
      0%, 100% { opacity: 0.3; }
      50% { opacity: 1; }
    }
    
    .star {
      animation: star-twinkle 2s ease-in-out infinite;
    }
    
    @keyframes explosion {
      0% { transform: scale(0); opacity: 1; }
      100% { transform: scale(2); opacity: 0; }
    }
    
    .explosion {
      animation: explosion 0.3s ease-out forwards;
    }

    @keyframes slideIn {
      from { opacity: 0; transform: translateY(20px); }
      to { opacity: 1; transform: translateY(0); }
    }
    
    .slide-in {
      animation: slideIn 0.3s ease-out;
    }

    @keyframes bossShake {
      0%, 100% { transform: translateX(0) translateY(0); }
      25% { transform: translateX(-3px) translateY(2px); }
      50% { transform: translateX(3px) translateY(-2px); }
      75% { transform: translateX(-2px) translateY(-3px); }
    }
    
    .boss-shake {
      animation: bossShake 0.15s ease-in-out infinite;
    }
  </style>
</head>
<body class="h-full m-0 p-0 bg-gray-900">
  <div id="game-container" class="relative w-full h-full overflow-hidden">
    <div id="stars" class="absolute inset-0 pointer-events-none"></div>
    
    <canvas id="gameCanvas" class="absolute inset-0 w-full h-full"></canvas>
    
    <div id="ui-overlay" class="absolute top-0 left-0 right-0 p-3 pointer-events-none">
      <div class="flex justify-between items-start gap-2">
        <div class="flex flex-col gap-2">
          <div class="bg-black/50 rounded-lg px-3 py-2 backdrop-blur-sm pointer-events-auto cursor-pointer hover:bg-black/70 transition-all" id="player-name-badge">
            <div class="flex items-center gap-2">
              <span class="text-2xl">🚀</span>
              <span id="current-player-name" class="game-font text-cyan-400 text-sm font-bold">ผู้เล่น</span>
            </div>
          </div>
          
          <div class="bg-black/50 rounded-lg p-2 backdrop-blur-sm">
            <div class="flex items-center gap-2">
              <span class="text-red-400 text-lg">❤️</span>
              <div class="w-24 sm:w-32 h-4 bg-gray-700 rounded-full overflow-hidden">
                <div id="health-bar" class="h-full bg-gradient-to-r from-red-500 to-pink-500 transition-all duration-300" style="width: 100%"></div>
              </div>
              <span id="health-text" class="game-font text-white text-xs sm:text-sm">100</span>
            </div>
          </div>
          
          <div class="bg-black/50 rounded-lg p-2 backdrop-blur-sm">
            <div class="flex items-center gap-2">
              <span class="text-yellow-400 text-lg">⚡</span>
              <span class="game-font text-yellow-400 text-xs sm:text-sm">LV.<span id="gun-level">1</span></span>
            </div>
          </div>

          <div class="bg-black/50 rounded-lg p-2 backdrop-blur-sm">
            <div class="flex items-center gap-2">
              <span id="difficulty-icon" class="text-lg">🟢</span>
              <span id="difficulty-name" class="game-font text-white text-xs sm:text-sm">ง่าย</span>
            </div>
          </div>
        </div>
        
        <div class="bg-black/50 rounded-lg px-4 py-2 backdrop-blur-sm">
          <div class="game-font text-center">
            <div class="text-cyan-400 text-xs">STAGE</div>
            <div id="stage-number" class="text-white text-xl sm:text-2xl font-bold">1</div>
          </div>
        </div>
        
        <div class="bg-black/50 rounded-lg p-2 backdrop-blur-sm">
          <div class="flex items-center gap-2">
            <span class="text-purple-400 text-lg">👾</span>
            <span id="enemy-count" class="game-font text-purple-400 text-xs sm:text-sm">0/5</span>
          </div>
        </div>
      </div>
      
      <div class="absolute top-16 left-1/2 -translate-x-1/2">
        <div class="game-font text-yellow-300 text-sm sm:text-lg">
          SCORE: <span id="score">0</span>
        </div>
      </div>
      
      <div id="boss-round-indicator" class="absolute top-24 left-1/2 -translate-x-1/2 bg-red-900/80 rounded-lg px-4 py-2 backdrop-blur-sm hidden">
        <div class="game-font text-center">
          <div class="text-red-300 text-sm">BOSS ROUND</div>
          <div id="boss-round-number" class="text-white text-xl font-bold">1/5</div>
        </div>
      </div>
      
      <div class="absolute bottom-4 right-4 pointer-events-auto flex gap-3">
        <button id="learning-btn" class="game-font bg-gradient-to-r from-purple-600 to-indigo-600 hover:from-purple-500 hover:to-indigo-500 text-white px-6 py-3 rounded-full text-base font-bold shadow-lg shadow-purple-500/50 transform hover:scale-105 transition-all flex items-center gap-2">
          🌍 เรียนรู้
        </button>
        <button id="leaderboard-btn" class="game-font bg-gradient-to-r from-yellow-600 to-amber-600 hover:from-yellow-500 hover:to-amber-500 text-white px-6 py-3 rounded-full text-base font-bold shadow-lg shadow-yellow-500/50 transform hover:scale-105 transition-all flex items-center gap-2">
          🏆 อันดับ
        </button>
      </div>
    </div>
    
    <div id="login-screen" class="absolute inset-0 flex flex-col items-center justify-center bg-gradient-to-b from-purple-900/95 to-blue-900/95 backdrop-blur-sm z-40">
      <div class="bg-black/60 rounded-2xl p-8 max-w-md w-full mx-4 backdrop-blur-md border-2 border-cyan-500/30">
        <div class="text-center mb-6">
          <div class="text-6xl mb-4">🚀</div>
          <h1 id="login-game-title" class="game-font text-4xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-pink-500 to-yellow-400 mb-2">
            ROCKET SHOOTER
          </h1>
          <p class="text-gray-400 text-sm">ยินดีต้อนรับสู่เกม!</p>
        </div>
        
        <div id="login-form" class="space-y-4">
          <h2 class="game-font text-2xl text-cyan-400 text-center mb-4">เข้าสู่ระบบ</h2>
          <div>
            <label for="login-username" class="block text-gray-300 text-sm mb-2 game-font">ชื่อผู้ใช้</label>
            <input type="text" id="login-username" class="w-full px-4 py-3 bg-gray-800/50 border-2 border-cyan-500/30 rounded-lg text-white focus:border-cyan-400 focus:outline-none game-font" placeholder="ใส่ชื่อผู้ใช้" maxlength="20">
          </div>
          <div id="login-error" class="hidden bg-red-500/20 border border-red-500 rounded-lg p-3 text-red-300 text-sm text-center"></div>
          <button id="login-btn" class="w-full game-font bg-gradient-to-r from-cyan-500 to-blue-600 hover:from-cyan-400 hover:to-blue-500 text-white px-6 py-3 rounded-lg text-lg font-bold shadow-lg transform hover:scale-105 transition-all">
            เข้าสู่ระบบ
          </button>
          <div class="text-center">
            <button id="show-register-btn" class="text-cyan-400 hover:text-cyan-300 text-sm game-font">
              ยังไม่มีบัญชี? <span class="underline">สร้างบัญชีใหม่</span>
            </button>
          </div>
        </div>
      </div>
    </div>
    </div>
</body>
</html>
```

---
*โปรเจกต์นี้สร้างขึ้นเพื่อการศึกษาเท่านั้น*
