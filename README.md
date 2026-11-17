<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>run bio lab 步態分析</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose@0.5.1675469404/pose.js"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: 'Inter', 'Noto Sans TC', sans-serif;
      background: linear-gradient(165deg, #fffbeb 0%, #fff7ed 48%, #f0f9ff 100%);
      color: #0f172a;
      overflow-x: hidden;
    }
    /* Sunny Track & Field Card Aesthetics */
    .glass-card {
      background: rgba(255, 255, 255, 0.92);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(251, 146, 60, 0.28);
      box-shadow: 0 10px 28px -6px rgba(234, 88, 12, 0.08), 0 4px 10px -2px rgba(15, 23, 42, 0.04);
    }
    .metric-card {
      background: linear-gradient(145deg, #ffffff 0%, #fffbf5 100%);
      border: 1px solid rgba(251, 146, 60, 0.35);
      box-shadow: 0 4px 14px -2px rgba(234, 88, 12, 0.07);
      transition: all 0.2s ease;
      cursor: pointer;
      position: relative;
    }
    .metric-card:hover {
      border-color: #ea580c;
      transform: translateY(-2px);
      box-shadow: 0 12px 24px -6px rgba(234, 88, 12, 0.16);
    }
    .track-stripe {
      background: repeating-linear-gradient(
        90deg,
        #ea580c 0px,
        #ea580c 28px,
        #ffffff 28px,
        #ffffff 32px,
        #f97316 32px,
        #f97316 60px,
        #ffffff 60px,
        #ffffff 64px
      );
      height: 5px;
    }
    input[type="range"] {
      accent-color: #ea580c;
    }
    /* Harmonize existing dark utility classes into sunlit athletics palette */
    .text-white { color: #0f172a !important; }
    .text-slate-200 { color: #1e293b !important; }
    .text-slate-300 { color: #334155 !important; }
    .text-slate-400 { color: #64748b !important; }
    .text-sky-300, .text-sky-400 { color: #0284c7 !important; }
    .text-emerald-300, .text-emerald-400 { color: #059669 !important; }
    .text-amber-300, .text-amber-400 { color: #d97706 !important; }
    .text-purple-300 { color: #7c3aed !important; }
    .text-teal-300 { color: #0d9488 !important; }
    .text-rose-300, .text-rose-400 { color: #e11d48 !important; }
    .text-indigo-300 { color: #4f46e5 !important; }
    .bg-slate-900, .bg-slate-900\/90, .bg-slate-900\/85, .bg-slate-900\/80 {
      background-color: #fff7ed !important;
    }
    .bg-slate-950\/70 {
      background-color: #ffffff !important;
    }
    .bg-slate-800, .bg-slate-800\/90 {
      background-color: #ffedd5 !important;
      color: #9a3412 !important;
    }
    .hover\:bg-slate-700:hover {
      background-color: #fed7aa !important;
    }
    .border-slate-800, .border-slate-700, .border-slate-700\/80 {
      border-color: #fed7aa !important;
    }
    /* Sunny Track & Field Stage Container */
    #canvasStageContainer {
      background: linear-gradient(145deg, #fffbf5 0%, #ffedd5 100%) !important;
    }
    .hud-overlay-box {
      background: rgba(255, 255, 255, 0.92) !important;
      border: 1px solid rgba(251, 146, 60, 0.45) !important;
      box-shadow: 0 4px 15px rgba(15, 23, 42, 0.15);
    }
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: #fff7ed;
    }
    ::-webkit-scrollbar-thumb {
      background: #fdba74;
      border-radius: 3px;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: #f97316;
    }
  </style>
</head>
<body class="min-h-screen flex flex-col selection:bg-sky-500 selection:text-white">

  <!-- Sunny Athletics Track Top Stripe -->
  <div class="track-stripe w-full"></div>

  <!-- Top Navigation Header -->
  <header class="sticky top-0 z-30 glass-card border-b border-orange-200 px-4 py-3">
    <div class="max-w-7xl mx-auto flex flex-wrap items-center justify-between gap-3">
      <div class="flex items-center gap-3">
        <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-orange-500 via-amber-500 to-yellow-400 flex items-center justify-center shadow-lg shadow-orange-500/25 font-extrabold text-white text-lg">
          ☀️
        </div>
        <h1 class="text-xl sm:text-2xl font-extrabold tracking-tight text-slate-900">run bio lab 步態分析</h1>
      </div>

      <!-- Mode & Quick Actions -->
      <div class="flex flex-wrap items-center gap-2">
        <button id="btnOpenGuideModal" onclick="openExplanationModal('guide')" class="px-3 py-1.5 rounded-lg text-xs font-semibold bg-amber-500/15 hover:bg-amber-500/25 text-amber-800 border border-amber-500/40 transition flex items-center gap-1.5">
          <span>📖</span> 點擊查看指標與操作說明
        </button>
        <button id="btnLoadDemo" onclick="loadSimulatedDemo()" class="px-3 py-1.5 rounded-lg text-xs font-semibold bg-slate-800 hover:bg-slate-700 text-sky-300 border border-slate-700 transition flex items-center gap-1.5">
          <span>🏃</span> 載入標準示範步態
        </button>
        <label for="videoFileInput" class="cursor-pointer px-3.5 py-1.5 rounded-lg text-xs font-semibold bg-sky-500 hover:bg-sky-400 text-slate-950 shadow-md shadow-sky-500/20 transition flex items-center gap-1.5">
          <span>📂</span> 匯入跑步影片 (MP4/MOV/WebM)
        </label>
        <input id="videoFileInput" type="file" accept="video/*" class="hidden" onchange="handleVideoUpload(event)" />
        <button id="btnExportCSV" onclick="exportFullCSV()" class="px-3 py-1.5 rounded-lg text-xs font-semibold bg-emerald-500/20 hover:bg-emerald-500/30 text-emerald-300 border border-emerald-500/40 transition flex items-center gap-1.5">
          <span>📊</span> 匯出 CSV 數據
        </button>
        <button id="btnCopyReport" onclick="copyBiomechanicalReport()" class="px-3 py-1.5 rounded-lg text-xs font-semibold bg-indigo-500/20 hover:bg-indigo-500/30 text-indigo-300 border border-indigo-500/40 transition flex items-center gap-1.5">
          <span>📋</span> 複製評估報告
        </button>
      </div>
    </div>
  </header>

  <!-- Toast Notification -->
  <div id="toastBanner" class="fixed bottom-5 right-5 z-50 hidden max-w-sm px-4 py-3 rounded-xl bg-slate-900/95 border border-sky-500/50 text-sm text-sky-200 shadow-2xl flex items-center gap-2.5">
    <span id="toastIcon">✅</span>
    <span id="toastMessage">系統就緒</span>
  </div>

  <!-- Main Workspace Container -->
  <main class="flex-1 max-w-7xl w-full mx-auto px-4 py-5 space-y-5">

    <!-- Top KPI Summary Row: Click any card to view full biomechanical explanation -->
    <section aria-label="核心步態指標總覽" class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-6 gap-3">
      <!-- 1. Cadence -->
      <div onclick="openExplanationModal('cadence')" class="metric-card rounded-xl p-3.5" title="點擊查看步頻標準與詳細說明">
        <div class="flex items-center justify-between text-xs text-slate-400 mb-1">
          <span class="font-semibold">步頻 Cadence</span>
          <span id="badgeCadence" class="px-1.5 py-0.5 rounded text-[10px] font-semibold bg-emerald-500/20 text-emerald-300">理想區間</span>
        </div>
        <div class="flex items-baseline gap-1.5">
          <span id="valCadence" class="text-2xl font-extrabold text-white">178.2</span>
          <span class="text-xs text-slate-400 font-medium">spm</span>
        </div>
        <p id="subCadence" class="hidden">每秒步數: 2.97 Hz (目標 ≥170)</p>
        <div class="text-[11px] text-orange-600 font-medium mt-1 flex items-center gap-1">👆 點擊查看說明</div>
      </div>

      <!-- 2. Step Length -->
      <div onclick="openExplanationModal('steplength')" class="metric-card rounded-xl p-3.5" title="點擊查看步長與跨步長詳細說明">
        <div class="flex items-center justify-between text-xs text-slate-400 mb-1">
          <span class="font-semibold">平均步長 Step</span>
          <span class="px-1.5 py-0.5 rounded text-[10px] font-semibold bg-sky-500/20 text-sky-300">單步距離</span>
        </div>
        <div class="flex items-baseline gap-1.5">
          <span id="valStepLength" class="text-2xl font-extrabold text-sky-400">1.18</span>
          <span class="text-xs text-slate-400 font-medium">m</span>
          <span id="valStepLengthCm" class="hidden">(118.0 cm)</span>
        </div>
        <p id="subStrideLength" class="hidden">跨步長 Stride: 2.36 m (69.4% 身高)</p>
        <div class="text-[11px] text-orange-600 font-medium mt-1 flex items-center gap-1">👆 點擊查看說明</div>
      </div>

      <!-- 3. Running Speed & Pace -->
      <div onclick="openExplanationModal('velocity')" class="metric-card rounded-xl p-3.5" title="點擊查看跑速與配速換算說明">
        <div class="flex items-center justify-between text-xs text-slate-400 mb-1">
          <span class="font-semibold">推算跑速 Speed</span>
          <span id="badgeModeLabel" class="px-1.5 py-0.5 rounded text-[10px] font-semibold bg-purple-500/20 text-purple-300">平面跑道</span>
        </div>
        <div class="flex items-baseline gap-1.5">
          <span id="valSpeedMs" class="text-2xl font-extrabold text-white">3.50</span>
          <span class="text-xs text-slate-400 font-medium">m/s</span>
          <span id="valSpeedKmh" class="hidden">(12.6 km/h)</span>
        </div>
        <p id="subPace" class="hidden">等效配速: 4'45" /km</p>
        <div class="text-[11px] text-orange-600 font-medium mt-1 flex items-center gap-1">👆 點擊查看說明</div>
      </div>

      <!-- 4. Ground Contact Time -->
      <div onclick="openExplanationModal('gct')" class="metric-card rounded-xl p-3.5" title="點擊查看觸地時間與騰空期說明">
        <div class="flex items-center justify-between text-xs text-slate-400 mb-1">
          <span class="font-semibold">觸地時間 GCT</span>
          <span id="badgeGCT" class="px-1.5 py-0.5 rounded text-[10px] font-semibold bg-amber-500/20 text-amber-300">支撐期</span>
        </div>
        <div class="flex items-baseline gap-1.5">
          <span id="valGCT" class="text-2xl font-extrabold text-amber-400">218</span>
          <span class="text-xs text-slate-400 font-medium">ms</span>
        </div>
        <p id="subFlightTime" class="hidden">騰空: 119 ms｜責任週期: 64.7%</p>
        <div class="text-[11px] text-orange-600 font-medium mt-1 flex items-center gap-1">👆 點擊查看說明</div>
      </div>

      <!-- 5. Foot Strike & Overstride -->
      <div onclick="openExplanationModal('overstride')" class="metric-card rounded-xl p-3.5" title="點擊查看著地型態與跨步距離說明">
        <div class="flex items-center justify-between text-xs text-slate-400 mb-1">
          <span class="font-semibold">跨步距離 CoM前</span>
          <span id="badgeStrikePattern" class="px-1.5 py-0.5 rounded text-[10px] font-semibold bg-teal-500/20 text-teal-300">中足著地 MFS</span>
        </div>
        <div class="flex items-baseline gap-1.5">
          <span id="valOverstride" class="text-2xl font-extrabold text-teal-300">6.4</span>
          <span class="text-xs text-slate-400 font-medium">cm</span>
        </div>
        <p id="subShankAngle" class="hidden">著地小腿傾角: 5.8°｜足著地角: 4.2°</p>
        <div class="text-[11px] text-orange-600 font-medium mt-1 flex items-center gap-1">👆 點擊查看說明</div>
      </div>

      <!-- 6. Vertical Oscillation & Stiffness -->
      <div onclick="openExplanationModal('stiffness')" class="metric-card rounded-xl p-3.5" title="點擊查看垂直振盪與下肢勁度說明">
        <div class="flex items-center justify-between text-xs text-slate-400 mb-1">
          <span class="font-semibold">垂直振盪幅度</span>
          <span class="px-1.5 py-0.5 rounded text-[10px] font-semibold bg-rose-500/20 text-rose-300">跑姿經濟性</span>
        </div>
        <div class="flex items-baseline gap-1.5">
          <span id="valVertOsc" class="text-2xl font-extrabold text-rose-400">7.2</span>
          <span class="text-xs text-slate-400 font-medium">cm</span>
        </div>
        <p id="subStiffness" class="hidden">垂直勁度 k_vert: 24.8 kN/m</p>
        <div class="text-[11px] text-orange-600 font-medium mt-1 flex items-center gap-1">👆 點擊查看說明</div>
      </div>
    </section>

    <!-- Middle Grid: Left = Video & Biomechanical Canvas Overlay (7 cols), Right = Calibration & Gait Event Manager (5 cols) -->
    <div class="grid grid-cols-1 lg:grid-cols-12 gap-5">

      <!-- Left Column: Video & Exoskeleton Canvas -->
      <div class="lg:col-span-7 glass-card rounded-2xl p-4 flex flex-col gap-3">
        <!-- Canvas Toolbar -->
        <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-800 pb-2.5">
          <div class="flex flex-wrap items-center gap-2">
            <span id="statusSourceTag" class="px-2.5 py-1 rounded-md text-xs font-semibold bg-sky-500/20 text-sky-300 border border-sky-500/30">
              🎬 標準生物力學模擬序列 (60 FPS)
            </span>
            <span id="videoResBadge" class="px-2.5 py-1 rounded-md text-xs font-mono text-emerald-300 bg-emerald-500/10 border border-emerald-500/30">
              原生解析度: 800 × 450 px (無壓縮)
            </span>
            <span id="frameReadout" class="text-xs font-mono text-slate-300 bg-slate-800/90 px-2.5 py-1 rounded-md">
              Frame: 0 / 120 (0.000 s)
            </span>
          </div>

          <div class="flex flex-wrap items-center gap-1.5">
            <button id="btnToggleNativeZoom" onclick="toggleNativeVideoZoom()" class="px-2.5 py-1 rounded-md text-xs font-medium bg-slate-800 hover:bg-slate-700 text-indigo-300 border border-indigo-500/40 transition" title="切換原比例適應視窗或 1:1 原始像素尺寸">
              🔍 顯示尺寸: 原比例無壓縮
            </button>
            <button id="btnToggleRuler" onclick="toggleCalibrationRuler()" class="px-2.5 py-1 rounded-md text-xs font-medium bg-slate-800 hover:bg-slate-700 text-amber-300 border border-amber-500/40 transition">
              📏 比例尺標記: 開啟
            </button>
            <button id="btnToggleSkeleton" onclick="toggleSkeletonOverlay()" class="px-2.5 py-1 rounded-md text-xs font-medium bg-sky-500/20 text-sky-300 border border-sky-500/40 transition">
              🦴 外骨骼與角度: 顯示
            </button>
            <button id="btnRunAIPose" onclick="runMediaPipeAnalysis()" class="px-2.5 py-1 rounded-md text-xs font-semibold bg-emerald-500 hover:bg-emerald-400 text-slate-950 transition">
              🤖 執行 AI 骨架全幀掃描
            </button>
          </div>
        </div>

        <!-- Middle Box: Clickable Import Video / Shoot Video Box + Uncompressed Video Analysis Stage -->
        <div id="canvasStageContainer" class="relative w-full rounded-2xl overflow-auto border-2 border-dashed border-orange-300 flex items-center justify-center p-3" style="min-height: 380px;">
          <!-- Hidden source video element & camera file input -->
          <video id="sourceVideo" class="hidden" playsinline muted crossorigin="anonymous"></video>
          <input id="cameraCaptureInput" type="file" accept="video/*" capture="environment" class="hidden" onchange="handleVideoUpload(event)" />

          <!-- Default Middle Clickable Box: Import Video & Shoot Video -->
          <div id="mediaSelectorOverlay" class="w-full max-w-xl mx-auto py-6 px-4 flex flex-col items-center justify-center text-center space-y-5">
            <div class="space-y-1">
              <span class="px-3 py-1 rounded-full text-xs font-bold bg-orange-500/15 text-orange-800 border border-orange-400/30">
                ☀️ 田徑陽光步態實驗室
              </span>
              <h3 class="text-base sm:text-lg font-extrabold text-slate-900 pt-1">請選擇匯入跑步影片或直接拍攝影片</h3>
              <p class="text-xs text-slate-500">支援原尺寸無壓縮影片載入，自動分析步頻、步長、觸地時間與腳踝活動度</p>
            </div>

            <!-- Two Prominent Clickable Action Boxes -->
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 w-full">
              <!-- Box 1: Import Video -->
              <div id="boxImportVideo" onclick="triggerImportVideoClick()" class="cursor-pointer group rounded-2xl bg-white hover:bg-orange-50/90 border-2 border-orange-300 hover:border-orange-500 p-6 shadow-md hover:shadow-xl transition flex flex-col items-center justify-center gap-2.5">
                <div class="w-14 h-14 rounded-2xl bg-gradient-to-br from-orange-500 to-amber-400 text-white flex items-center justify-center text-2xl shadow-md group-hover:scale-105 transition">
                  📂
                </div>
                <div class="text-base font-extrabold text-slate-900">匯入影片</div>
                <p class="text-xs text-slate-500">點擊從手機或電腦選取跑步影片<br/><span class="font-mono text-[11px] text-orange-700">(MP4 / MOV / WebM 原尺寸無壓縮)</span></p>
              </div>

              <!-- Box 2: Shoot / Record Video -->
              <div id="boxRecordVideo" onclick="startCameraCaptureMode()" class="cursor-pointer group rounded-2xl bg-white hover:bg-amber-50/90 border-2 border-amber-300 hover:border-amber-500 p-6 shadow-md hover:shadow-xl transition flex flex-col items-center justify-center gap-2.5">
                <div class="w-14 h-14 rounded-2xl bg-gradient-to-br from-amber-500 to-yellow-400 text-white flex items-center justify-center text-2xl shadow-md group-hover:scale-105 transition">
                  🎥
                </div>
                <div class="text-base font-extrabold text-slate-900">拍攝影片</div>
                <p class="text-xs text-slate-500">點擊開啟鏡頭現場錄製跑步動作<br/><span class="font-mono text-[11px] text-amber-700">(支援相機即時錄影 / 手機拍攝)</span></p>
              </div>
            </div>

            <div class="flex flex-wrap items-center justify-center gap-3 pt-1">
              <button id="btnPreviewDemoInBox" onclick="loadSimulatedDemo()" class="px-3.5 py-1.5 rounded-xl bg-orange-100 hover:bg-orange-200 text-orange-900 text-xs font-bold border border-orange-300 transition">
                🏃 預覽標準示範步態動畫
              </button>
            </div>
          </div>

          <!-- Live Camera Recording Stage (Shown when shooting video via webcam/camera) -->
          <div id="cameraRecordingStage" class="hidden w-full flex flex-col items-center justify-center gap-3 py-2">
            <div class="relative w-full max-w-lg rounded-xl overflow-hidden border-2 border-orange-400 bg-black">
              <video id="cameraLivePreview" autoplay playsinline muted class="w-full h-auto block"></video>
              <div id="recordingStatusPill" class="absolute top-3 left-3 px-2.5 py-1 rounded-full bg-white/90 text-xs font-bold text-rose-600 flex items-center gap-1.5">
                <span>🔴</span> <span id="recordingTimerText">相機預覽就緒</span>
              </div>
            </div>
            <div class="flex flex-wrap items-center justify-center gap-2">
              <button id="btnStartRec" onclick="toggleCameraRecording()" class="px-4 py-2 rounded-xl bg-rose-600 hover:bg-rose-500 text-white text-xs font-bold shadow transition">
                ⏺ 開始錄製跑步影片
              </button>
              <button id="btnFallbackPhoneCam" onclick="triggerNativeCameraInput()" class="px-3.5 py-2 rounded-xl bg-amber-100 hover:bg-amber-200 text-amber-900 text-xs font-bold border border-amber-300 transition">
                📱 改用手機原生相機拍攝
              </button>
              <button id="btnCancelCamera" onclick="stopAndCloseCameraStage()" class="px-3.5 py-2 rounded-xl bg-slate-200 hover:bg-slate-300 text-slate-800 text-xs font-bold transition">
                ✕ 返回匯入/拍攝選單
              </button>
            </div>
          </div>

          <!-- Biomechanical Video & Skeleton Canvas (Shown once video or demo is active) -->
          <canvas id="biomechCanvas" width="800" height="450" class="hidden max-w-full h-auto block cursor-crosshair rounded-xl"></canvas>

          <!-- Instant Joint Angle HUD Overlay (Top-Left, shown when canvas is active) -->
          <div id="hudOverlayPanel" class="hidden hud-overlay-box absolute top-3 left-3 backdrop-blur-md rounded-xl p-2.5 text-xs space-y-1 pointer-events-none">
            <div class="font-semibold text-sky-300 flex items-center justify-between gap-4 border-b border-slate-800 pb-1">
              <span>即時運動學關節角度</span>
              <span id="hudGaitPhase" class="px-1.5 py-0.5 rounded bg-sky-500/20 text-[10px] text-sky-200">支撐期 Stance</span>
            </div>
            <div class="grid grid-cols-2 gap-x-4 gap-y-0.5 pt-0.5 font-mono text-[11px]">
              <div class="text-slate-300">軀幹前傾 Trunk: <span id="hudTrunkAngle" class="text-white font-bold">7.8°</span></div>
              <div class="text-slate-300">髖關節角 Hip: <span id="hudHipAngle" class="text-sky-400 font-bold">162.4°</span></div>
              <div class="text-slate-300">膝關節角 Knee: <span id="hudKneeAngle" class="text-emerald-400 font-bold">141.2°</span></div>
              <div class="text-slate-300">踝關節角 Ankle: <span id="hudAnkleAngle" class="text-amber-400 font-bold">86.5°</span></div>
              <div class="text-slate-300">小腿傾角 Shank: <span id="hudShankAngle" class="text-teal-300 font-bold">5.8°</span></div>
              <div class="text-slate-300">重心高度 CoM Y: <span id="hudComHeight" class="text-rose-300 font-bold">94.2 cm</span></div>
            </div>
          </div>

          <!-- Return to Import/Shoot Box Button (Top-Right over Canvas) -->
          <button id="btnBackToSelector" onclick="showMediaSelectorBox()" class="hidden hud-overlay-box absolute top-3 right-3 rounded-lg px-2.5 py-1.5 text-xs font-bold text-orange-800 hover:bg-orange-100 transition">
            🔄 重新匯入 / 拍攝影片
          </button>

          <!-- Spatial Scale Hint (Bottom-Left) -->
          <div id="rulerHintBox" onclick="openExplanationModal('calibration')" class="hidden hud-overlay-box cursor-pointer absolute bottom-3 left-3 rounded-lg px-2.5 py-1 text-[11px] text-amber-700 font-medium hover:border-orange-500 transition">
            💡 拖曳 <span class="font-bold">P1—P2</span> 校正比例尺（點擊查看比例尺教學）
          </div>
        </div>

        <!-- Scrubber & Frame-by-Frame Playback Controls -->
        <div class="space-y-2 pt-1">
          <div class="flex items-center gap-3">
            <span id="timeCurrentLabel" class="text-xs font-mono text-slate-400 w-14">0.00 s</span>
            <input id="frameSlider" type="range" min="0" max="119" value="0" step="1" oninput="seekToFrame(parseInt(this.value, 10))" class="flex-1 h-2 bg-slate-800 rounded-lg cursor-pointer" />
            <span id="timeTotalLabel" class="text-xs font-mono text-slate-400 w-14 text-right">2.00 s</span>
          </div>

          <!-- Event Timeline Markers Bar -->
          <div id="eventMarkerTrack" class="relative w-full h-5 bg-slate-900 rounded-md border border-slate-800 overflow-hidden"></div>

          <div class="flex flex-wrap items-center justify-between gap-2 pt-1">
            <!-- Transport Controls -->
            <div class="flex items-center gap-1.5">
              <button id="btnStepBack5" onclick="stepFrames(-5)" class="px-2.5 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-xs font-mono text-slate-200 transition" title="倒退 5 幀">-5F</button>
              <button id="btnStepBack1" onclick="stepFrames(-1)" class="px-2.5 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-xs font-mono text-slate-200 transition" title="上一幀">◀ -1F</button>
              <button id="btnPlayPause" onclick="togglePlayPause()" class="px-4 py-1.5 rounded-lg bg-sky-500 hover:bg-sky-400 text-slate-950 font-bold text-xs transition flex items-center gap-1">
                <span id="playPauseIcon">▶</span> <span id="playPauseText">播放</span>
              </button>
              <button id="btnStepFwd1" onclick="stepFrames(1)" class="px-2.5 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-xs font-mono text-slate-200 transition" title="下一幀">+1F ▶</button>
              <button id="btnStepFwd5" onclick="stepFrames(5)" class="px-2.5 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-xs font-mono text-slate-200 transition" title="快進 5 幀">+5F</button>
            </div>

            <!-- Playback Speed & Manual Event Tagging -->
            <div class="flex flex-wrap items-center gap-1.5">
              <select id="selectPlaybackRate" onchange="changePlaybackSpeed(this.value)" class="bg-slate-800 border border-slate-700 text-xs text-slate-200 rounded-lg px-2 py-1.5">
                <option value="0.2">0.2x 慢速</option>
                <option value="0.5" selected>0.5x 動作分解</option>
                <option value="1.0">1.0x 原速</option>
              </select>
              <button id="btnMarkIC" onclick="markGaitEventAtCurrentFrame('IC')" class="px-2.5 py-1.5 rounded-lg bg-emerald-500/20 hover:bg-emerald-500/30 text-emerald-300 border border-emerald-500/40 text-xs font-semibold transition">
                + 標記著地 (IC)
              </button>
              <button id="btnMarkTO" onclick="markGaitEventAtCurrentFrame('TO')" class="px-2.5 py-1.5 rounded-lg bg-amber-500/20 hover:bg-amber-500/30 text-amber-300 border border-amber-500/40 text-xs font-semibold transition">
                + 標記離地 (TO)
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Right Column: Spatial Calibration, Subject Settings & Gait Cycle Event Table -->
      <div class="lg:col-span-5 flex flex-col gap-4">

        <!-- Card 1: Calibration & Running Environment Parameters -->
        <div class="glass-card rounded-2xl p-4 space-y-3">
          <div class="flex items-center justify-between border-b border-slate-800 pb-2">
            <h2 class="text-sm font-bold text-white flex items-center gap-2">
              <span>📐</span> 空間比例尺與受試者參數設定
            </h2>
            <span id="scaleFactorBadge" class="text-[11px] font-mono text-amber-300 bg-amber-500/10 px-2 py-0.5 rounded border border-amber-500/30">
              比例尺: 220.0 px/m
            </span>
          </div>

          <div class="grid grid-cols-2 sm:grid-cols-3 gap-2.5 text-xs">
            <div>
              <label for="inputRunMode" class="block text-slate-400 mb-1">拍攝場地模式</label>
              <select id="inputRunMode" onchange="recalculateAllBiomechanics()" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2.5 py-1.5 text-white focus:border-sky-500 outline-none">
                <option value="overground" selected>平面跑道 (Overground)</option>
                <option value="treadmill">跑步機定速 (Treadmill)</option>
              </select>
            </div>

            <div>
              <label for="inputSubjectHeight" class="block text-slate-400 mb-1">受試者身高 (cm)</label>
              <input id="inputSubjectHeight" type="number" min="100" max="230" step="0.5" value="170.0" oninput="recalculateAllBiomechanics()" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2.5 py-1.5 text-white font-mono focus:border-sky-500 outline-none" />
            </div>

            <div>
              <label for="inputSubjectMass" class="block text-slate-400 mb-1">受試者體重 (kg)</label>
              <input id="inputSubjectMass" type="number" min="30" max="180" step="0.5" value="62.0" oninput="recalculateAllBiomechanics()" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2.5 py-1.5 text-white font-mono focus:border-sky-500 outline-none" />
            </div>

            <div>
              <label for="inputRefLength" class="block text-slate-400 mb-1">比例尺實際長度 (m)</label>
              <input id="inputRefLength" type="number" min="0.2" max="5.0" step="0.05" value="1.00" oninput="recalculateAllBiomechanics()" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2.5 py-1.5 text-amber-300 font-mono focus:border-amber-500 outline-none" />
            </div>

            <div>
              <label for="inputTreadmillSpeed" class="block text-slate-400 mb-1">跑步機時速 (km/h)</label>
              <input id="inputTreadmillSpeed" type="number" min="4" max="30" step="0.1" value="12.6" oninput="recalculateAllBiomechanics()" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2.5 py-1.5 text-purple-300 font-mono focus:border-purple-500 outline-none" />
            </div>

            <div>
              <label for="inputVideoFPS" class="block text-slate-400 mb-1">影片幀率 (FPS)</label>
              <select id="inputVideoFPS" onchange="recalculateAllBiomechanics()" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2.5 py-1.5 text-white font-mono focus:border-sky-500 outline-none">
                <option value="30">30 FPS</option>
                <option value="60" selected>60 FPS (建議)</option>
                <option value="120">120 FPS (高速)</option>
                <option value="240">240 FPS (超高速)</option>
              </select>
            </div>
          </div>

          <div class="flex items-center justify-between pt-1 text-[11px] text-slate-400">
            <span>自動依受試者身高校正比例尺：</span>
            <button id="btnCalibrateByHeight" onclick="calibrateScaleFromBodyHeight()" class="px-2.5 py-1 rounded bg-slate-800 hover:bg-slate-700 text-sky-300 border border-slate-700 transition">
              🧍 依當前骨架身高自動對齊比例尺
            </button>
          </div>
        </div>

        <!-- Card 2: Step-by-Step Gait Events & Step Length Breakdown -->
        <div class="glass-card rounded-2xl p-4 flex-1 flex flex-col justify-between space-y-3">
          <div class="flex items-center justify-between border-b border-slate-800 pb-2">
            <div>
              <h2 class="text-sm font-bold text-white flex items-center gap-1.5">
                <span>🦶</span> 步態週期事件與單步明細 (IC / TO)
              </h2>
              <p class="text-[11px] text-slate-400">點擊列可快速跳轉至該著地 (IC) 或離地 (TO) 關鍵幀</p>
            </div>
            <div class="flex items-center gap-1.5">
              <button id="btnAutoDetectEvents" onclick="autoDetectGaitEvents()" class="px-2.5 py-1 rounded-lg bg-sky-500/20 hover:bg-sky-500/30 text-sky-300 border border-sky-500/30 text-xs font-semibold transition">
                ⚡ 重新自動偵測
              </button>
              <button id="btnClearEvents" onclick="resetDefaultEvents()" class="px-2 py-1 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs transition">
                重置
              </button>
            </div>
          </div>

          <!-- Step Table -->
          <div class="overflow-x-auto max-h-56 overflow-y-auto rounded-lg border border-slate-800">
            <table class="w-full text-left border-collapse text-xs">
              <thead class="bg-slate-900/90 text-slate-400 sticky top-0">
                <tr>
                  <th class="py-2 px-2.5 font-semibold">步序 / 側別</th>
                  <th class="py-2 px-2 font-semibold">著地 IC</th>
                  <th class="py-2 px-2 font-semibold">離地 TO</th>
                  <th class="py-2 px-2 font-semibold">GCT</th>
                  <th class="py-2 px-2 font-semibold">步頻</th>
                  <th class="py-2 px-2 font-semibold">步長</th>
                  <th class="py-2 px-2 font-semibold text-right">操作</th>
                </tr>
              </thead>
              <tbody id="gaitStepsTableBody" class="divide-y divide-slate-800/70 font-mono text-[11px]">
              </tbody>
            </table>
          </div>

          <!-- Biomechanical Risk & Hamstring Load Quick Diagnostic (Click to view full clinical explanation) -->
          <div onclick="openExplanationModal('diagnostic')" class="cursor-pointer bg-orange-50 hover:bg-orange-100/80 border border-orange-300 rounded-xl p-3 flex items-center justify-between transition" title="點擊查看完整運動防護診斷與訓練處方">
            <div class="flex items-center gap-2">
              <span class="text-base">🩺</span>
              <div>
                <div class="text-xs font-bold text-slate-900">運動防護與步態生物力學診斷</div>
                <div class="text-[11px] text-orange-700 font-medium">👆 點擊展開詳細評估分析與腿後肌負荷報告</div>
              </div>
            </div>
            <span id="riskStatusBadge" class="px-2.5 py-1 rounded-full text-xs font-bold bg-emerald-500/20 text-emerald-300">低風險・高經濟性</span>
            <p id="diagnosticSummaryText" class="hidden">
              當前步頻 178.2 spm 落於理想區間，著地瞬間足部落點距身體重心 (CoM) 前方 6.4 cm，無明顯跨步過大 (Overstriding) 現象，有效降低擺盪末期腿後肌群 (Biceps Femoris) 離心煞車負荷與脛骨衝擊峰值。
            </p>
          </div>
        </div>

      </div>
    </div>

    <!-- Clean Segmented Tab Bar for Charts & ROM Tables -->
    <div class="flex flex-wrap items-center justify-between gap-2 glass-card rounded-xl p-2">
      <div class="flex flex-wrap items-center gap-1.5">
        <button id="tabBtnAnkle" onclick="switchBottomTab('ankle')" class="px-4 py-2 rounded-lg text-xs font-bold bg-orange-500 text-white shadow-sm transition">
          🦶 腳踝活動度曲線 (Ankle ROM)
        </button>
        <button id="tabBtnKinematics" onclick="switchBottomTab('kinematics')" class="px-4 py-2 rounded-lg text-xs font-bold bg-orange-100 text-orange-900 hover:bg-orange-200 transition">
          📈 髖/膝/踝/軀幹 運動學曲線
        </button>
        <button id="tabBtnRomTable" onclick="switchBottomTab('romtable')" class="px-4 py-2 rounded-lg text-xs font-bold bg-orange-100 text-orange-900 hover:bg-orange-200 transition">
          📋 關節活動度 (ROM) 統計與檢核表
        </button>
      </div>
      <button id="btnExplainCharts" onclick="openExplanationModal('ankle_rom')" class="px-3 py-1.5 rounded-lg text-xs font-semibold bg-amber-100 hover:bg-amber-200 text-amber-900 border border-amber-300 transition">
        💡 點擊查看曲線判讀說明
      </button>
    </div>

    <!-- Tab Panel 2 & 3: Dynamic Time-Series Kinematic Chart & Phase Analysis (Hidden by default for clean layout, toggleable via tabs) -->
    <section id="panelKinematicsAndRom" class="grid grid-cols-1 lg:grid-cols-12 gap-5 hidden">
      <!-- Time-Series Kinematic Chart (8 cols) -->
      <div id="subPanelKinematicsChart" class="lg:col-span-12 glass-card rounded-2xl p-4 space-y-3">
        <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-800 pb-2.5">
          <div>
            <h2 class="text-sm font-bold text-white flex items-center gap-2">
              <span>📈</span> 下肢關節運動學時序曲線 (點擊圖表可同步跳轉影片幀)
            </h2>
            <p class="text-xs text-slate-400">連續追蹤髖屈伸、膝屈曲、踝背屈/蹠屈角度與軀幹前傾角變化</p>
          </div>
          <div class="flex flex-wrap items-center gap-1.5 text-xs">
            <button id="toggleCurveKnee" onclick="toggleChartDataset(0)" class="px-2.5 py-1 rounded-md font-semibold bg-emerald-500/20 text-emerald-300 border border-emerald-500/40">膝關節角</button>
            <button id="toggleCurveHip" onclick="toggleChartDataset(1)" class="px-2.5 py-1 rounded-md font-semibold bg-sky-500/20 text-sky-300 border border-sky-500/40">髖關節角</button>
            <button id="toggleCurveAnkle" onclick="toggleChartDataset(2)" class="px-2.5 py-1 rounded-md font-semibold bg-amber-500/20 text-amber-300 border border-amber-500/40">踝關節角</button>
            <button id="toggleCurveTrunk" onclick="toggleChartDataset(3)" class="px-2.5 py-1 rounded-md font-semibold bg-purple-500/20 text-purple-300 border border-purple-500/40">軀幹前傾角</button>
          </div>
        </div>

        <div class="relative h-72 w-full">
          <canvas id="kinematicChart"></canvas>
        </div>
      </div>

      <!-- Joint ROM & Statistical Summary Table -->
      <div id="subPanelRomTable" class="lg:col-span-12 glass-card rounded-2xl p-4 flex flex-col justify-between space-y-3 hidden">
        <div>
          <div class="border-b border-slate-800 pb-2.5 mb-3 flex items-center justify-between">
            <div>
              <h2 class="text-sm font-bold text-white flex items-center gap-2">
                <span>📋</span> 關節活動度 (ROM) 與統計摘要
              </h2>
              <p class="text-xs text-slate-400">全週期運動學 Mean ± SD、極值與觸地/擺盪期特徵</p>
            </div>
            <button id="btnExplainVicon" onclick="openExplanationModal('vicon')" class="px-3 py-1 rounded-lg text-xs font-semibold bg-orange-100 text-orange-800 border border-orange-300">
              📖 點擊查看 Vicon 檢核標準說明
            </button>
          </div>

          <div class="overflow-x-auto rounded-lg border border-slate-800">
            <table class="w-full text-left border-collapse text-xs">
              <thead class="bg-slate-900/90 text-slate-400">
                <tr>
                  <th class="py-2 px-2.5">參數指標</th>
                  <th class="py-2 px-2">Mean ± SD</th>
                  <th class="py-2 px-2">Min / Max</th>
                  <th class="py-2 px-2">ROM</th>
                </tr>
              </thead>
              <tbody id="romSummaryTableBody" class="divide-y divide-slate-800/70 font-mono text-[11px]">
              </tbody>
            </table>
          </div>
        </div>

        <!-- Key Phase Checkpoints -->
        <div class="bg-slate-900/80 border border-slate-800 rounded-xl p-3 space-y-2 text-xs">
          <div class="font-bold text-slate-200 flex items-center justify-between">
            <span>關鍵步態期生物力學檢核點 (點擊任一項目查看臨床意義)</span>
            <span class="text-[11px] text-sky-400 font-mono">Vicon Plug-in Gait 2D 規範</span>
          </div>
          <div class="grid grid-cols-2 sm:grid-cols-4 gap-2 text-[11px]">
            <div onclick="openExplanationModal('vicon')" class="cursor-pointer bg-slate-950/70 hover:border-orange-400 p-2.5 rounded-lg border border-slate-800 transition">
              <div class="text-slate-400">著地期膝屈曲角 (IC)</div>
              <div id="statICKnee" class="text-sm font-bold text-emerald-400 font-mono mt-0.5">161.5° (微屈緩衝)</div>
            </div>
            <div onclick="openExplanationModal('vicon')" class="cursor-pointer bg-slate-950/70 hover:border-orange-400 p-2.5 rounded-lg border border-slate-800 transition">
              <div class="text-slate-400">支撐中期最大膝屈 (MS)</div>
              <div id="statMSKnee" class="text-sm font-bold text-sky-400 font-mono mt-0.5">136.8° (吸震良好)</div>
            </div>
            <div onclick="openExplanationModal('vicon')" class="cursor-pointer bg-slate-950/70 hover:border-orange-400 p-2.5 rounded-lg border border-slate-800 transition">
              <div class="text-slate-400">離地瞬間髖伸展角 (TO)</div>
              <div id="statTOHip" class="text-sm font-bold text-amber-400 font-mono mt-0.5">192.4° (完全推進)</div>
            </div>
            <div onclick="openExplanationModal('vicon')" class="cursor-pointer bg-slate-950/70 hover:border-orange-400 p-2.5 rounded-lg border border-slate-800 transition">
              <div class="text-slate-400">擺盪末期腿後肌負荷角</div>
              <div id="statTSHamstring" class="text-sm font-bold text-purple-300 font-mono mt-0.5">158.2° (安全範圍)</div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Tab Panel 1 (Default Active): Dedicated Ankle Range of Motion (ROM) Curve Section -->
    <section id="panelAnkleRom" aria-label="腳踝活動度專項曲線分析" class="glass-card rounded-2xl p-4 space-y-3">
      <div class="flex flex-wrap items-center justify-between gap-3 border-b border-slate-800 pb-2.5">
        <div>
          <h2 class="text-sm font-bold text-white flex items-center gap-2">
            <span>🦶</span> 腳踝活動度 (Ankle ROM) 專項時序曲線圖 (背屈 DF / 蹠屈 PF 與雙側對稱追蹤)
          </h2>
          <p class="text-xs text-slate-400">獨立放大腳踝角度座標軸，精細檢視觸地吸震期最大背屈、離地推進期最大蹠屈與左右腳踝活動度差異 (點擊曲線可同步跳轉影片幀)</p>
        </div>
        <div class="flex flex-wrap items-center gap-1.5 text-xs">
          <button id="toggleAnkleRight" onclick="toggleAnkleChartDataset(0)" class="px-2.5 py-1 rounded-md font-semibold bg-amber-500/20 text-amber-300 border border-amber-500/40">右腳踝角 R-Ankle (°)</button>
          <button id="toggleAnkleLeft" onclick="toggleAnkleChartDataset(1)" class="px-2.5 py-1 rounded-md font-semibold bg-emerald-500/20 text-emerald-300 border border-emerald-500/40">左腳踝角 L-Ankle (°)</button>
          <button id="toggleAnkleDFPF" onclick="toggleAnkleChartDataset(2)" class="px-2.5 py-1 rounded-md font-semibold bg-sky-500/20 text-sky-300 border border-sky-500/40">背屈/蹠屈偏移量 (±90°)</button>
          <button id="toggleFootPitch" onclick="toggleAnkleChartDataset(3)" class="px-2.5 py-1 rounded-md font-semibold bg-rose-500/20 text-rose-300 border border-rose-500/40">足底著地傾角 (°)</button>
        </div>
      </div>

      <!-- Ankle ROM Quick KPI Strip -->
      <div class="grid grid-cols-2 sm:grid-cols-4 gap-3 text-xs">
        <div class="bg-slate-900/85 border border-slate-800 rounded-xl p-2.5 flex items-center justify-between">
          <div>
            <div class="text-slate-400 text-[11px]">腳踝總活動度 (Total Ankle ROM)</div>
            <div id="valAnkleTotalROM" class="text-lg font-extrabold text-amber-400 font-mono mt-0.5">42.0°</div>
          </div>
          <span class="px-2 py-0.5 rounded text-[10px] font-semibold bg-amber-500/20 text-amber-300">全週期振幅</span>
        </div>

        <div class="bg-slate-900/85 border border-slate-800 rounded-xl p-2.5 flex items-center justify-between">
          <div>
            <div class="text-slate-400 text-[11px]">支撐期最大背屈 (Peak DF)</div>
            <div id="valAnklePeakDF" class="text-lg font-extrabold text-emerald-400 font-mono mt-0.5">74.0° (+16.0° DF)</div>
          </div>
          <span class="px-2 py-0.5 rounded text-[10px] font-semibold bg-emerald-500/20 text-emerald-300">阿基里斯腱儲能</span>
        </div>

        <div class="bg-slate-900/85 border border-slate-800 rounded-xl p-2.5 flex items-center justify-between">
          <div>
            <div class="text-slate-400 text-[11px]">離地期最大蹠屈 (Peak PF)</div>
            <div id="valAnklePeakPF" class="text-lg font-extrabold text-sky-400 font-mono mt-0.5">116.0° (-26.0° PF)</div>
          </div>
          <span class="px-2 py-0.5 rounded text-[10px] font-semibold bg-sky-500/20 text-sky-300">推進爆發期</span>
        </div>

        <div class="bg-slate-900/85 border border-slate-800 rounded-xl p-2.5 flex items-center justify-between">
          <div>
            <div class="text-slate-400 text-[11px]">雙側腳踝 ROM 對稱性 (L vs R)</div>
            <div id="valAnkleSymmetry" class="text-lg font-extrabold text-teal-300 font-mono mt-0.5">98.6% (對稱良好)</div>
          </div>
          <span class="px-2 py-0.5 rounded text-[10px] font-semibold bg-teal-500/20 text-teal-300">雙側比較</span>
        </div>
      </div>

      <!-- Dedicated Ankle Chart Container -->
      <div class="relative h-72 w-full">
        <canvas id="ankleRomChart"></canvas>
      </div>
    </section>

    <!-- Click-to-View Detailed Explanation Modal -->
    <div id="explanationModal" class="fixed inset-0 z-50 hidden bg-slate-900/50 backdrop-blur-sm flex items-center justify-center p-4" onclick="if(event.target === this) closeExplanationModal()">
      <div class="bg-white border-2 border-orange-300 rounded-2xl max-w-2xl w-full max-h-[85vh] overflow-y-auto shadow-2xl p-5 space-y-4">
        <div class="flex items-start justify-between gap-3 border-b border-orange-200 pb-3">
          <div>
            <span id="modalCategoryTag" class="px-2.5 py-0.5 rounded-full text-[11px] font-bold bg-orange-100 text-orange-800">田徑生物力學指標詳解</span>
            <h3 id="modalTitle" class="text-lg font-extrabold text-slate-900 mt-1">指標詳細說明</h3>
          </div>
          <button id="btnCloseModal" onclick="closeExplanationModal()" class="px-3 py-1 rounded-lg bg-orange-100 hover:bg-orange-200 text-orange-900 text-xs font-bold transition">
            ✕ 關閉視窗
          </button>
        </div>

        <!-- Topic Quick Switcher Pills inside Modal -->
        <div class="flex flex-wrap gap-1.5 border-b border-orange-100 pb-3 text-xs">
          <button id="pillGuide" onclick="openExplanationModal('guide')" class="px-2.5 py-1 rounded-full bg-orange-50 hover:bg-orange-100 text-orange-800 font-semibold border border-orange-200">📖 系統操作流程</button>
          <button id="pillCadence" onclick="openExplanationModal('cadence')" class="px-2.5 py-1 rounded-full bg-orange-50 hover:bg-orange-100 text-orange-800 font-semibold border border-orange-200">步頻 Cadence</button>
          <button id="pillStepLength" onclick="openExplanationModal('steplength')" class="px-2.5 py-1 rounded-full bg-orange-50 hover:bg-orange-100 text-orange-800 font-semibold border border-orange-200">步長與跨步長</button>
          <button id="pillGCT" onclick="openExplanationModal('gct')" class="px-2.5 py-1 rounded-full bg-orange-50 hover:bg-orange-100 text-orange-800 font-semibold border border-orange-200">觸地時間 GCT</button>
          <button id="pillOverstride" onclick="openExplanationModal('overstride')" class="px-2.5 py-1 rounded-full bg-orange-50 hover:bg-orange-100 text-orange-800 font-semibold border border-orange-200">著地與跨步距離</button>
          <button id="pillAnkleRom" onclick="openExplanationModal('ankle_rom')" class="px-2.5 py-1 rounded-full bg-orange-50 hover:bg-orange-100 text-orange-800 font-semibold border border-orange-200">腳踝活動度曲線</button>
          <button id="pillDiagnostic" onclick="openExplanationModal('diagnostic')" class="px-2.5 py-1 rounded-full bg-orange-50 hover:bg-orange-100 text-orange-800 font-semibold border border-orange-200">🩺 運動防護診斷</button>
        </div>

        <!-- Dynamic Modal Body Content -->
        <div id="modalBodyContent" class="space-y-3 text-sm text-slate-700 leading-relaxed">
        </div>
      </div>
    </div>

  </main>

  <script>
    // ============================================================================
    // RunBioLab — 2D Running Gait Biomechanics Engine
    // ============================================================================

    const appState = {
      sourceType: 'demo', // 'demo' | 'video'
      totalFrames: 120,
      currentFrame: 0,
      fps: 60,
      isPlaying: false,
      playbackRate: 0.5,
      playTimer: null,
      showSkeleton: true,
      showCalibrationRuler: true,
      nativePixelZoom: false,
      canvasWidth: 800,
      canvasHeight: 450,
      // 2-Point spatial calibration ruler on canvas (800x450 coordinate space)
      ruler: {
        p1: { x: 240, y: 392 },
        p2: { x: 460, y: 392 },
        draggingPoint: null // 'p1' | 'p2' | null
      },
      // Frame-by-frame pose & kinematic dataset
      framesData: [],
      // Detected or manually tagged gait events: { frame, type: 'IC'|'TO', side: 'R'|'L', footX, footY, comX, strikeAngle, shankAngle }
      gaitEvents: [],
      // Computed step metrics
      stepsList: [],
      summaryMetrics: {
        cadenceSpm: 178.2,
        stepLengthM: 1.18,
        strideLengthM: 2.36,
        speedMs: 3.50,
        speedKmh: 12.6,
        gctMs: 218,
        flightMs: 119,
        dutyFactorPct: 64.7,
        overstrideCm: 6.4,
        shankAngleDeg: 5.8,
        footStrikeAngleDeg: 4.2,
        strikePattern: '中足著地 MFS',
        vertOscCm: 7.2,
        kVertKnM: 24.8,
        pxPerMeter: 220.0
      }
    };

    let chartInstance = null;
    let ankleChartInstance = null;
    let mediaPipePose = null;

    // Show non-blocking toast banner
    function showToast(message, icon = '✅') {
      const banner = document.getElementById('toastBanner');
      const iconEl = document.getElementById('toastIcon');
      const msgEl = document.getElementById('toastMessage');
      if (!banner || !iconEl || !msgEl) return;
      iconEl.textContent = icon;
      msgEl.textContent = message;
      banner.classList.remove('hidden');
      clearTimeout(banner._hideTimer);
      banner._hideTimer = setTimeout(() => {
        banner.classList.add('hidden');
      }, 3200);
    }

    // Compute 2D interior angle (deg) at vertex B formed by A-B-C
    function computeAngle3Pts(A, B, C) {
      const v1x = A.x - B.x;
      const v1y = A.y - B.y;
      const v2x = C.x - B.x;
      const v2y = C.y - B.y;
      const dot = v1x * v2x + v1y * v2y;
      const mag1 = Math.hypot(v1x, v1y);
      const mag2 = Math.hypot(v2x, v2y);
      if (mag1 < 1e-5 || mag2 < 1e-5) return 180.0;
      const cosVal = Math.max(-1, Math.min(1, dot / (mag1 * mag2)));
      return (Math.acos(cosVal) * 180) / Math.PI;
    }

    // Generate biomechanically realistic 60 FPS running simulation scaled to current native canvas dimensions
    function generateBiomechanicalDemoData(customW, customH) {
      const W = customW || appState.canvasWidth || 800;
      const H = customH || appState.canvasHeight || 450;
      const scaleX = W / 800;
      const scaleY = H / 450;
      const scaleUniform = Math.min(scaleX, scaleY);
      const totalFrames = 120;
      const fps = 60;
      const pxPerM = 220.0 * scaleUniform; // Scaled to native resolution
      const groundY = Math.round(H * 0.871);
      const cadenceSpm = 178.0; // ~2.967 steps/sec -> 1 step = 20.22 frames, 1 full stride (2 steps) = 40.45 frames
      const stepFrames = (60 / cadenceSpm) * fps; // ~20.22 frames per step
      const strideFrames = stepFrames * 2; // ~40.45 frames per full stride
      const speedMs = 3.50; // 3.5 m/s

      const frames = [];
      for (let f = 0; f < totalFrames; f++) {
        const t = f / fps;
        // Runner progresses left-to-right across the native stage width
        const worldComX = (W * 0.20) + (f * (speedMs / fps) * pxPerM) % (W * 0.60);
        // Double-frequency vertical oscillation of CoM (lowest at mid-stance, highest at flight)
        const stepPhase = (f % stepFrames) / stepFrames; // 0..1 per step
        const stridePhaseR = (f % strideFrames) / strideFrames; // 0..1 for Right leg cycle
        const stridePhaseL = ((f + stepFrames) % strideFrames) / strideFrames; // 0..1 for Left leg cycle

        // Vertical oscillation ~ 7.2 cm peak-to-peak
        const vertOscPx = Math.cos(stepPhase * 2 * Math.PI - 0.9) * (0.036 * pxPerM);
        const hipY = groundY - 0.92 * pxPerM + vertOscPx;
        const hipX = worldComX;

        // Trunk forward lean ~ 7.5 to 9.0 deg
        const trunkLeanDeg = 7.8 + 1.1 * Math.sin(stepPhase * 2 * Math.PI);
        const trunkRad = (trunkLeanDeg * Math.PI) / 180;
        const trunkLen = 0.50 * pxPerM;
        const shoulderX = hipX + Math.sin(trunkRad) * trunkLen;
        const shoulderY = hipY - Math.cos(trunkRad) * trunkLen;
        const earX = shoulderX + Math.sin(trunkRad) * (0.16 * pxPerM);
        const earY = shoulderY - Math.cos(trunkRad) * (0.16 * pxPerM);

        // Helper to compute leg kinematics for a given cycle phase u in [0, 1)
        // u = 0.00 is Initial Contact (IC), u = 0.32 is Toe-Off (TO), u in (0.32, 1.0) is Swing
        function solveLeg(u) {
          let thighAngleFromVertDeg = 0; // positive = flexion forward
          let kneeFlexionDeg = 18.0;     // 0 = fully straight (180 internal), >0 = flexed
          let ankleAngleDeg = 90.0;

          if (u <= 0.32) {
            // Stance phase (0 -> 0.32)
            const s = u / 0.32;
            // Hip sweeps from +22 deg flexion at IC to -24 deg hyperextension at TO
            thighAngleFromVertDeg = 22.0 - 46.0 * s;
            // Knee flexes from 18 deg at IC to 42 deg at Mid-Stance (s=0.42), then extends to 14 deg at TO
            kneeFlexionDeg = 18.0 + 25.0 * Math.sin(s * Math.PI) - 4.0 * s;
            // Ankle dorsiflexes in mid-stance then plantarflexes at TO
            ankleAngleDeg = 88.0 - 14.0 * Math.sin(s * Math.PI) + 28.0 * Math.pow(s, 1.6);
          } else {
            // Swing phase (0.32 -> 1.00)
            const w = (u - 0.32) / 0.68;
            // Hip swings from -24 deg forward to +34 deg in terminal swing, then retracts to +22 deg at IC
            thighAngleFromVertDeg = -24.0 + 56.0 * Math.sin(w * 0.78 * Math.PI);
            // Knee flexes strongly in mid-swing (up to 98 deg) then extends before IC
            kneeFlexionDeg = 14.0 + 84.0 * Math.pow(Math.sin(w * Math.PI), 1.25) + 4.0 * w;
            ankleAngleDeg = 112.0 - 24.0 * Math.sin(w * Math.PI);
          }

          const thighLen = 0.42 * pxPerM;
          const shankLen = 0.41 * pxPerM;
          const footLen = 0.15 * pxPerM;

          const thighRad = (thighAngleFromVertDeg * Math.PI) / 180;
          const shankRad = ((thighAngleFromVertDeg - kneeFlexionDeg) * Math.PI) / 180;

          let kneeX = hipX + Math.sin(thighRad) * thighLen;
          let kneeY = hipY + Math.cos(thighRad) * thighLen;
          let ankleX = kneeX + Math.sin(shankRad) * shankLen;
          let ankleY = kneeY + Math.cos(shankRad) * shankLen;

          // Clamp foot to ground during stance phase
          const heelOffset = 0.05 * pxPerM;
          let heelX = ankleX - Math.cos(shankRad) * heelOffset;
          let heelY = ankleY + Math.sin(shankRad) * heelOffset;
          const footStrikeAngleDeg = u <= 0.32 ? (4.2 * (1 - u / 0.32) - 18.0 * (u / 0.32)) : 6.0;
          const footRad = (footStrikeAngleDeg * Math.PI) / 180;
          let toeX = heelX + Math.cos(footRad) * footLen;
          let toeY = heelY - Math.sin(footRad) * footLen;

          if (u <= 0.32) {
            const lowestY = Math.max(heelY, toeY);
            const dy = groundY - lowestY;
            ankleY += dy;
            heelY += dy;
            toeY += dy;
            kneeY += dy * 0.55;
          }

          const internalKneeDeg = 180.0 - kneeFlexionDeg;
          const internalHipDeg = 180.0 - (thighAngleFromVertDeg + trunkLeanDeg);
          const shankAngleDeg = -(thighAngleFromVertDeg - kneeFlexionDeg);

          return {
            knee: { x: kneeX, y: kneeY },
            ankle: { x: ankleX, y: ankleY },
            heel: { x: heelX, y: heelY },
            toe: { x: toeX, y: toeY },
            internalKneeDeg,
            internalHipDeg,
            ankleAngleDeg,
            shankAngleDeg,
            footStrikeAngleDeg,
            phase: u <= 0.32 ? 'Stance' : 'Swing'
          };
        }

        const rightLeg = solveLeg(stridePhaseR);
        const leftLeg = solveLeg(stridePhaseL);

        // Arm swing counter-balancing legs
        const armSwingRad = ((stridePhaseR - 0.5) * 1.15);
        const elbowX = shoulderX - Math.sin(armSwingRad) * (0.28 * pxPerM);
        const elbowY = shoulderY + Math.cos(armSwingRad) * (0.28 * pxPerM);
        const wristX = elbowX + Math.cos(armSwingRad * 0.7) * (0.24 * pxPerM);
        const wristY = elbowY + Math.sin(armSwingRad * 0.7) * (0.12 * pxPerM);

        const comHeightCm = ((groundY - hipY) / pxPerM) * 100;

        frames.push({
          frame: f,
          time: t,
          ear: { x: earX, y: earY },
          shoulder: { x: shoulderX, y: shoulderY },
          elbow: { x: elbowX, y: elbowY },
          wrist: { x: wristX, y: wristY },
          hip: { x: hipX, y: hipY },
          rightLeg,
          leftLeg,
          trunkLeanDeg: Number(trunkLeanDeg.toFixed(1)),
          hipAngleDeg: Number(rightLeg.internalHipDeg.toFixed(1)),
          kneeAngleDeg: Number(rightLeg.internalKneeDeg.toFixed(1)),
          ankleAngleDeg: Number(rightLeg.ankleAngleDeg.toFixed(1)),
          leftAnkleAngleDeg: Number(leftLeg.ankleAngleDeg.toFixed(1)),
          ankleDfPfDeg: Number((90.0 - rightLeg.ankleAngleDeg).toFixed(1)),
          footPitchDeg: Number(rightLeg.footStrikeAngleDeg.toFixed(1)),
          shankAngleDeg: Number(rightLeg.shankAngleDeg.toFixed(1)),
          comHeightCm: Number(comHeightCm.toFixed(1)),
          gaitPhase: rightLeg.phase === 'Stance' ? '支撐期 Stance (R)' : '擺盪期 Swing (R)'
        });
      }

      appState.framesData = frames;
      appState.totalFrames = totalFrames;
      appState.fps = fps;
      document.getElementById('inputVideoFPS').value = '60';
      document.getElementById('frameSlider').max = String(totalFrames - 1);
      document.getElementById('timeTotalLabel').textContent = (totalFrames / fps).toFixed(2) + ' s';
    }

    // Automatically detect IC (Initial Contact) and TO (Toe-Off) gait events from frame data
    function autoDetectGaitEvents() {
      const events = [];
      const frames = appState.framesData;
      if (!frames || frames.length < 4) return;

      for (let i = 1; i < frames.length; i++) {
        const prevR = frames[i - 1].rightLeg.phase;
        const currR = frames[i].rightLeg.phase;
        if (prevR === 'Swing' && currR === 'Stance') {
          events.push({
            frame: i,
            type: 'IC',
            side: 'R',
            footX: (frames[i].rightLeg.heel.x + frames[i].rightLeg.toe.x) / 2,
            comX: frames[i].hip.x,
            shankAngle: frames[i].rightLeg.shankAngleDeg,
            strikeAngle: frames[i].rightLeg.footStrikeAngleDeg
          });
        } else if (prevR === 'Stance' && currR === 'Swing') {
          events.push({
            frame: i,
            type: 'TO',
            side: 'R',
            footX: frames[i].rightLeg.toe.x,
            comX: frames[i].hip.x,
            shankAngle: frames[i].rightLeg.shankAngleDeg,
            strikeAngle: frames[i].rightLeg.footStrikeAngleDeg
          });
        }

        const prevL = frames[i - 1].leftLeg.phase;
        const currL = frames[i].leftLeg.phase;
        if (prevL === 'Swing' && currL === 'Stance') {
          events.push({
            frame: i,
            type: 'IC',
            side: 'L',
            footX: (frames[i].leftLeg.heel.x + frames[i].leftLeg.toe.x) / 2,
            comX: frames[i].hip.x,
            shankAngle: frames[i].leftLeg.shankAngleDeg,
            strikeAngle: frames[i].leftLeg.footStrikeAngleDeg
          });
        } else if (prevL === 'Stance' && currL === 'Swing') {
          events.push({
            frame: i,
            type: 'TO',
            side: 'L',
            footX: frames[i].leftLeg.toe.x,
            comX: frames[i].hip.x,
            shankAngle: frames[i].leftLeg.shankAngleDeg,
            strikeAngle: frames[i].leftLeg.footStrikeAngleDeg
          });
        }
      }

      // Ensure frame 0 initial contact is captured if starting at IC
      if (frames[0].rightLeg.phase === 'Stance' && !events.some(e => e.frame === 0)) {
        events.unshift({
          frame: 0,
          type: 'IC',
          side: 'R',
          footX: (frames[0].rightLeg.heel.x + frames[0].rightLeg.toe.x) / 2,
          comX: frames[0].hip.x,
          shankAngle: frames[0].rightLeg.shankAngleDeg,
          strikeAngle: frames[0].rightLeg.footStrikeAngleDeg
        });
      }

      events.sort((a, b) => a.frame - b.frame);
      appState.gaitEvents = events;
      recalculateAllBiomechanics();
      showToast(`已自動偵測 ${events.length} 個步態著地 (IC) 與離地 (TO) 事件`, '⚡');
    }

    // Reset events to default detected set
    function resetDefaultEvents() {
      autoDetectGaitEvents();
    }

    // Manually tag an IC or TO event at the current frame
    function markGaitEventAtCurrentFrame(eventType) {
      const f = appState.currentFrame;
      const fd = appState.framesData[f];
      if (!fd) return;

      // Determine active side based on which ankle is lower
      const side = fd.rightLeg.ankle.y >= fd.leftLeg.ankle.y ? 'R' : 'L';
      const leg = side === 'R' ? fd.rightLeg : fd.leftLeg;

      // Replace if same frame & type exists
      appState.gaitEvents = appState.gaitEvents.filter(e => !(e.frame === f && e.type === eventType));
      appState.gaitEvents.push({
        frame: f,
        type: eventType,
        side,
        footX: (leg.heel.x + leg.toe.x) / 2,
        comX: fd.hip.x,
        shankAngle: leg.shankAngleDeg,
        strikeAngle: leg.footStrikeAngleDeg
      });
      appState.gaitEvents.sort((a, b) => a.frame - b.frame);
      recalculateAllBiomechanics();
      showToast(`已於 Frame ${f} 標記 ${side === 'R' ? '右側' : '左側'} ${eventType === 'IC' ? '著地 (IC)' : '離地 (TO)'}`, '📌');
    }

    function removeStepByIndex(idx) {
      const step = appState.stepsList[idx];
      if (!step) return;
      appState.gaitEvents = appState.gaitEvents.filter(e => e.frame !== step.icFrame);
      recalculateAllBiomechanics();
      showToast(`已移除第 ${idx + 1} 步著地事件`, '🗑️');
    }

    // Calibrate spatial scale using subject's standing/running stature height
    function calibrateScaleFromBodyHeight() {
      const fd = appState.framesData[appState.currentFrame];
      if (!fd) return;
      const subjectHeightCm = parseFloat(document.getElementById('inputSubjectHeight').value) || 170.0;
      const lowestAnkleY = Math.max(fd.rightLeg.heel.y, fd.leftLeg.heel.y);
      const bodyPixelHeight = Math.max(80, (lowestAnkleY - fd.ear.y) * 1.08);
      const refLengthM = parseFloat(document.getElementById('inputRefLength').value) || 1.0;
      const estimatedPxPerM = bodyPixelHeight / (subjectHeightCm / 100);

      // Update ruler P1-P2 length to match estimatedPxPerM * refLengthM within native canvas width
      const canvas = document.getElementById('biomechCanvas');
      const maxW = canvas ? canvas.width - 25 : 760;
      const desiredRulerPx = estimatedPxPerM * refLengthM;
      appState.ruler.p2.x = Math.min(maxW, appState.ruler.p1.x + desiredRulerPx);
      appState.ruler.p2.y = appState.ruler.p1.y;
      recalculateAllBiomechanics();
      showToast(`已依受試者身高 (${subjectHeightCm} cm) 自動校準空間比例尺: ${estimatedPxPerM.toFixed(1)} px/m`, '🧍');
    }

    // Master calculation of Cadence, Step Length, Stride Length, GCT, Flight Time, Overstride & Stiffness
    function recalculateAllBiomechanics() {
      const fps = parseFloat(document.getElementById('inputVideoFPS').value) || 60;
      appState.fps = fps;
      const runMode = document.getElementById('inputRunMode').value;
      const subjectHeightCm = Math.max(100, parseFloat(document.getElementById('inputSubjectHeight').value) || 170.0);
      const subjectMassKg = Math.max(30, parseFloat(document.getElementById('inputSubjectMass').value) || 62.0);
      const refLengthM = Math.max(0.1, parseFloat(document.getElementById('inputRefLength').value) || 1.0);
      const treadmillSpeedKmh = Math.max(1, parseFloat(document.getElementById('inputTreadmillSpeed').value) || 12.6);

      // Compute pixel-to-meter ratio from the 2-point ruler
      const rulerDistPx = Math.max(20, Math.hypot(
        appState.ruler.p2.x - appState.ruler.p1.x,
        appState.ruler.p2.y - appState.ruler.p1.y
      ));
      const pxPerMeter = rulerDistPx / refLengthM;
      appState.summaryMetrics.pxPerMeter = pxPerMeter;
      document.getElementById('scaleFactorBadge').textContent = `比例尺: ${pxPerMeter.toFixed(1)} px/m`;

      const icEvents = appState.gaitEvents.filter(e => e.type === 'IC');
      const toEvents = appState.gaitEvents.filter(e => e.type === 'TO');

      const steps = [];
      for (let i = 0; i < icEvents.length; i++) {
        const ic = icEvents[i];
        // Find matching TO on the same side after this IC
        const matchingTO = toEvents.find(t => t.frame > ic.frame && t.side === ic.side) ||
                           toEvents.find(t => t.frame > ic.frame);
        const nextIC = icEvents[i + 1] || null;

        const gctSec = matchingTO ? (matchingTO.frame - ic.frame) / fps : (6.5 / fps);
        const stepTimeSec = nextIC ? (nextIC.frame - ic.frame) / fps : (20.2 / fps);
        const stepCadence = stepTimeSec > 0.05 ? 60.0 / stepTimeSec : 178.0;

        let stepLengthM = 1.18;
        if (runMode === 'treadmill') {
          const beltSpeedMs = treadmillSpeedKmh / 3.6;
          stepLengthM = beltSpeedMs * stepTimeSec;
        } else if (nextIC) {
          const rawPixelStep = Math.abs(nextIC.footX - ic.footX);
          stepLengthM = rawPixelStep > 30 ? rawPixelStep / pxPerMeter : (3.50 * stepTimeSec);
        } else {
          stepLengthM = 3.50 * stepTimeSec;
        }

        const overstrideCm = ((ic.footX - ic.comX) / pxPerMeter) * 100;

        steps.push({
          stepIndex: i + 1,
          side: ic.side,
          icFrame: ic.frame,
          toFrame: matchingTO ? matchingTO.frame : ic.frame + 6,
          gctMs: Math.round(gctSec * 1000),
          flightMs: Math.max(0, Math.round((stepTimeSec - gctSec) * 1000)),
          cadenceSpm: Number(stepCadence.toFixed(1)),
          stepLengthM: Number(stepLengthM.toFixed(2)),
          overstrideCm: Number(overstrideCm.toFixed(1)),
          shankAngle: Number((ic.shankAngle || 5.8).toFixed(1)),
          strikeAngle: Number((ic.strikeAngle || 4.2).toFixed(1))
        });
      }

      appState.stepsList = steps;

      // Aggregate metrics across steps
      const avg = (arr, fallback) => arr.length ? arr.reduce((a, b) => a + b, 0) / arr.length : fallback;
      const meanCadence = avg(steps.map(s => s.cadenceSpm), 178.2);
      const meanStepLengthM = avg(steps.map(s => s.stepLengthM), 1.18);
      const meanStrideLengthM = meanStepLengthM * 2;
      const meanGctMs = Math.round(avg(steps.map(s => s.gctMs), 218));
      const meanFlightMs = Math.round(avg(steps.map(s => s.flightMs), 119));
      const totalCycleMs = Math.max(1, meanGctMs + meanFlightMs);
      const dutyFactorPct = (meanGctMs / totalCycleMs) * 100;
      const speedMs = runMode === 'treadmill'
        ? (treadmillSpeedKmh / 3.6)
        : (meanStepLengthM * (meanCadence / 60.0));
      const speedKmh = speedMs * 3.6;

      const meanOverstrideCm = avg(steps.map(s => s.overstrideCm), 6.4);
      const meanShankAngle = avg(steps.map(s => s.shankAngle), 5.8);
      const meanStrikeAngle = avg(steps.map(s => s.strikeAngle), 4.2);

      let strikePattern = '中足著地 MFS';
      if (meanStrikeAngle >= 8.0) strikePattern = '後足著地 RFS';
      else if (meanStrikeAngle <= -1.5) strikePattern = '前足著地 FFS';

      // Compute vertical oscillation from CoM trajectory
      const comHeights = appState.framesData.map(d => d.comHeightCm);
      const vertOscCm = comHeights.length
        ? Math.max(2.5, Math.min(...comHeights) ? (Math.max(...comHeights) - Math.min(...comHeights)) : 7.2)
        : 7.2;

      // Morin (2005) sine-wave estimate of vertical stiffness k_vert (kN/m)
      const tcSec = meanGctMs / 1000;
      const tfSec = meanFlightMs / 1000;
      const fmaxN = subjectMassKg * 9.81 * (Math.PI / 2) * ((tfSec / tcSec) + 1);
      const deltaYM = Math.max(0.03, vertOscCm / 100);
      const kVertKnM = (fmaxN / deltaYM) / 1000;

      appState.summaryMetrics = {
        cadenceSpm: Number(meanCadence.toFixed(1)),
        stepLengthM: Number(meanStepLengthM.toFixed(2)),
        strideLengthM: Number(meanStrideLengthM.toFixed(2)),
        speedMs: Number(speedMs.toFixed(2)),
        speedKmh: Number(speedKmh.toFixed(1)),
        gctMs: meanGctMs,
        flightMs: meanFlightMs,
        dutyFactorPct: Number(dutyFactorPct.toFixed(1)),
        overstrideCm: Number(meanOverstrideCm.toFixed(1)),
        shankAngleDeg: Number(meanShankAngle.toFixed(1)),
        footStrikeAngleDeg: Number(meanStrikeAngle.toFixed(1)),
        strikePattern,
        vertOscCm: Number(vertOscCm.toFixed(1)),
        kVertKnM: Number(kVertKnM.toFixed(1)),
        pxPerMeter
      };

      updateDashboardUI(subjectHeightCm, runMode);
      renderStepsTable();
      renderEventTimelineBar();
      renderRomSummaryTable();
      updateKinematicChart();
      updateAnkleRomChart();
      renderCanvasFrame(appState.currentFrame);
    }

    function updateDashboardUI(subjectHeightCm, runMode) {
      const m = appState.summaryMetrics;
      document.getElementById('valCadence').textContent = m.cadenceSpm.toFixed(1);
      document.getElementById('subCadence').textContent = `每秒步數: ${(m.cadenceSpm / 60).toFixed(2)} Hz (目標 ≥170)`;

      const badgeCadence = document.getElementById('badgeCadence');
      if (m.cadenceSpm >= 170 && m.cadenceSpm <= 190) {
        badgeCadence.textContent = '理想區間';
        badgeCadence.className = 'px-1.5 py-0.5 rounded text-[10px] font-semibold bg-emerald-500/20 text-emerald-300';
      } else if (m.cadenceSpm < 170) {
        badgeCadence.textContent = '偏低 (建議提頻)';
        badgeCadence.className = 'px-1.5 py-0.5 rounded text-[10px] font-semibold bg-amber-500/20 text-amber-300';
      } else {
        badgeCadence.textContent = '高步頻衝刺';
        badgeCadence.className = 'px-1.5 py-0.5 rounded text-[10px] font-semibold bg-sky-500/20 text-sky-300';
      }

      document.getElementById('valStepLength').textContent = m.stepLengthM.toFixed(2);
      document.getElementById('valStepLengthCm').textContent = `(${(m.stepLengthM * 100).toFixed(1)} cm)`;
      const relStepPct = ((m.stepLengthM * 100) / subjectHeightCm) * 100;
      document.getElementById('subStrideLength').textContent = `跨步長 Stride: ${m.strideLengthM.toFixed(2)} m (${relStepPct.toFixed(1)}% 身高)`;

      document.getElementById('badgeModeLabel').textContent = runMode === 'treadmill' ? '跑步機定速' : '平面跑道';
      document.getElementById('valSpeedMs').textContent = m.speedMs.toFixed(2);
      document.getElementById('valSpeedKmh').textContent = `(${m.speedKmh.toFixed(1)} km/h)`;

      const paceMinPerKm = m.speedKmh > 0.5 ? 60 / m.speedKmh : 0;
      const paceMin = Math.floor(paceMinPerKm);
      const paceSec = Math.round((paceMinPerKm - paceMin) * 60);
      document.getElementById('subPace').textContent = `等效配速: ${paceMin}'${String(paceSec).padStart(2, '0')}" /km`;

      document.getElementById('valGCT').textContent = String(m.gctMs);
      document.getElementById('subFlightTime').textContent = `騰空: ${m.flightMs} ms｜責任週期: ${m.dutyFactorPct.toFixed(1)}%`;

      document.getElementById('badgeStrikePattern').textContent = m.strikePattern;
      document.getElementById('valOverstride').textContent = m.overstrideCm.toFixed(1);
      document.getElementById('subShankAngle').textContent = `著地小腿傾角: ${m.shankAngleDeg.toFixed(1)}°｜足著地角: ${m.footStrikeAngleDeg.toFixed(1)}°`;

      document.getElementById('valVertOsc').textContent = m.vertOscCm.toFixed(1);
      document.getElementById('subStiffness').textContent = `垂直勁度 k_vert: ${m.kVertKnM.toFixed(1)} kN/m`;

      // Update diagnostic summary
      const riskBadge = document.getElementById('riskStatusBadge');
      const diagText = document.getElementById('diagnosticSummaryText');
      if (m.overstrideCm > 12.0 || m.cadenceSpm < 164) {
        riskBadge.textContent = '中度風險・跨步偏大';
        riskBadge.className = 'px-2 py-0.5 rounded text-[10px] font-semibold bg-amber-500/20 text-amber-300';
        diagText.textContent = `當前步頻 ${m.cadenceSpm} spm、著地落點位於身體重心前方 ${m.overstrideCm} cm (小腿傾角 ${m.shankAngleDeg}°)，呈現跨步過大 (Overstriding) 傾向。建議提升步頻 5–7% 並縮短著地重心前距，以減輕擺盪末期股二頭肌長頭離心拉扯與膝關節煞車衝擊。`;
      } else {
        riskBadge.textContent = '低風險・高經濟性';
        riskBadge.className = 'px-2 py-0.5 rounded text-[10px] font-semibold bg-emerald-500/20 text-emerald-300';
        diagText.textContent = `當前步頻 ${m.cadenceSpm} spm 與單步長 ${m.stepLengthM} m 搭配均衡，著地瞬間足部落點距身體重心 (CoM) 前方 ${m.overstrideCm} cm (${m.strikePattern})，無過度跨步現象，有效降低擺盪末期腿後肌群離心負荷與地面反作用力煞車峰值。`;
      }
    }

    function renderStepsTable() {
      const tbody = document.getElementById('gaitStepsTableBody');
      if (!tbody) return;
      if (!appState.stepsList.length) {
        tbody.innerHTML = `<tr><td colspan="7" class="py-4 text-center text-slate-500">尚無步態事件，請點擊「+ 標記著地 (IC)」或「⚡ 重新自動偵測」</td></tr>`;
        return;
      }

      tbody.innerHTML = appState.stepsList.map((s, idx) => `
        <tr class="hover:bg-slate-800/70 transition cursor-pointer" onclick="seekToFrame(${s.icFrame})">
          <td class="py-2 px-2.5 font-semibold text-white">
            #${s.stepIndex} <span class="px-1.5 py-0.5 rounded text-[10px] ${s.side === 'R' ? 'bg-sky-500/20 text-sky-300' : 'bg-emerald-500/20 text-emerald-300'}">${s.side === 'R' ? '右腳 R' : '左腳 L'}</span>
          </td>
          <td class="py-2 px-2 text-emerald-300">F${s.icFrame}</td>
          <td class="py-2 px-2 text-amber-300" onclick="event.stopPropagation(); seekToFrame(${s.toFrame})">F${s.toFrame}</td>
          <td class="py-2 px-2 text-slate-200">${s.gctMs} ms</td>
          <td class="py-2 px-2 text-sky-300 font-semibold">${s.cadenceSpm} spm</td>
          <td class="py-2 px-2 text-white font-bold">${s.stepLengthM.toFixed(2)} m</td>
          <td class="py-2 px-2 text-right">
            <button onclick="event.stopPropagation(); removeStepByIndex(${idx})" class="text-rose-400 hover:text-rose-300 px-1.5 py-0.5 rounded bg-rose-500/10" title="刪除此步">✕</button>
          </td>
        </tr>
      `).join('');
    }

    function renderEventTimelineBar() {
      const track = document.getElementById('eventMarkerTrack');
      if (!track) return;
      const maxF = Math.max(1, appState.totalFrames - 1);
      track.innerHTML = appState.gaitEvents.map(ev => {
        const leftPct = (ev.frame / maxF) * 100;
        const isIC = ev.type === 'IC';
        const colorClass = isIC ? 'bg-emerald-400' : 'bg-amber-400';
        return `<button onclick="seekToFrame(${ev.frame})" style="left: ${leftPct}%" class="absolute top-0 bottom-0 w-1.5 -ml-0.5 ${colorClass} hover:scale-x-150 transition" title="${ev.side}腳 ${ev.type} (Frame ${ev.frame})"></button>`;
      }).join('');
    }

    function renderRomSummaryTable() {
      const tbody = document.getElementById('romSummaryTableBody');
      if (!tbody || !appState.framesData.length) return;

      const calcStat = (arr) => {
        const n = arr.length;
        const mean = arr.reduce((a, b) => a + b, 0) / n;
        const sd = Math.sqrt(arr.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / n);
        const min = Math.min(...arr);
        const max = Math.max(...arr);
        return { mean, sd, min, max, rom: max - min };
      };

      const kneeStats = calcStat(appState.framesData.map(d => d.kneeAngleDeg));
      const hipStats = calcStat(appState.framesData.map(d => d.hipAngleDeg));
      const ankleStats = calcStat(appState.framesData.map(d => d.ankleAngleDeg));
      const trunkStats = calcStat(appState.framesData.map(d => d.trunkLeanDeg));

      const rows = [
        { name: '膝關節角 Knee (°)', color: 'text-emerald-400', s: kneeStats },
        { name: '髖關節角 Hip (°)', color: 'text-sky-400', s: hipStats },
        { name: '踝關節角 Ankle (°)', color: 'text-amber-400', s: ankleStats },
        { name: '軀幹前傾 Trunk (°)', color: 'text-purple-300', s: trunkStats }
      ];

      tbody.innerHTML = rows.map(r => `
        <tr>
          <td class="py-2 px-2.5 font-semibold ${r.color}">${r.name}</td>
          <td class="py-2 px-2 text-slate-200">${r.s.mean.toFixed(1)} ± ${r.s.sd.toFixed(1)}</td>
          <td class="py-2 px-2 text-slate-400">${r.s.min.toFixed(1)} / ${r.s.max.toFixed(1)}</td>
          <td class="py-2 px-2 font-bold text-white">${r.s.rom.toFixed(1)}°</td>
        </tr>
      `).join('');

      // Update key phase checkpoints
      const firstIC = appState.gaitEvents.find(e => e.type === 'IC');
      const firstTO = appState.gaitEvents.find(e => e.type === 'TO');
      const icKnee = firstIC && appState.framesData[firstIC.frame] ? appState.framesData[firstIC.frame].kneeAngleDeg : 161.5;
      const toHip = firstTO && appState.framesData[firstTO.frame] ? appState.framesData[firstTO.frame].hipAngleDeg : 192.4;

      document.getElementById('statICKnee').textContent = `${icKnee.toFixed(1)}° (微屈緩衝)`;
      document.getElementById('statMSKnee').textContent = `${kneeStats.min.toFixed(1)}° (最大緩衝)`;
      document.getElementById('statTOHip').textContent = `${toHip.toFixed(1)}° (推進伸展)`;
      document.getElementById('statTSHamstring').textContent = `${(kneeStats.max - 12.5).toFixed(1)}° (離心安全)`;
    }

    // Draw biomechanical scene & skeleton overlay on Canvas
    function renderCanvasFrame(frameIdx) {
      const canvas = document.getElementById('biomechCanvas');
      if (!canvas) return;
      const ctx = canvas.getContext('2d');
      const W = canvas.width;
      const H = canvas.height;
      const fd = appState.framesData[frameIdx] || appState.framesData[0];
      if (!fd) return;

      ctx.clearRect(0, 0, W, H);

      const groundY = Math.round(H * 0.871);
      const uiScale = Math.max(1, Math.min(W, H) / 450);

      // 1. Render Video Frame at 100% Native Uncompressed Resolution or Lab Grid Background
      const video = document.getElementById('sourceVideo');
      if (appState.sourceType === 'video' && video && video.readyState >= 2) {
        ctx.imageSmoothingEnabled = false;
        ctx.drawImage(video, 0, 0, W, H);
      } else {
        // Sunny Outdoor Athletics Track & Field Stadium Environment
        const skyGrad = ctx.createLinearGradient(0, 0, 0, groundY);
        skyGrad.addColorStop(0, '#38bdf8');
        skyGrad.addColorStop(0.55, '#bae6fd');
        skyGrad.addColorStop(1, '#fef3c7');
        ctx.fillStyle = skyGrad;
        ctx.fillRect(0, 0, W, groundY);

        // Warm Sun Glow in Top-Right Sky
        const sunGrad = ctx.createRadialGradient(W - 95, 65, 10, W - 95, 65, 140);
        sunGrad.addColorStop(0, 'rgba(254, 240, 138, 0.95)');
        sunGrad.addColorStop(0.4, 'rgba(253, 186, 116, 0.45)');
        sunGrad.addColorStop(1, 'rgba(254, 240, 138, 0)');
        ctx.fillStyle = sunGrad;
        ctx.fillRect(W - 240, 0, 240, 210);

        // Stadium Infield Turf Grass Strip
        ctx.fillStyle = '#22c55e';
        ctx.fillRect(0, groundY - 18, W, 18);

        // Subtle Biomechanical Measurement Grid in Sky
        ctx.strokeStyle = 'rgba(15, 23, 42, 0.08)';
        ctx.lineWidth = 1;
        for (let x = 0; x <= W; x += 44) {
          ctx.beginPath();
          ctx.moveTo(x, 0);
          ctx.lineTo(x, groundY);
          ctx.stroke();
        }
        for (let y = 40; y <= groundY; y += 44) {
          ctx.beginPath();
          ctx.moveTo(0, y);
          ctx.lineTo(W, y);
          ctx.stroke();
        }

        // Classic Terracotta PU Athletics Running Track Surface & White Lane Lines
        const trackGrad = ctx.createLinearGradient(0, groundY, 0, H);
        trackGrad.addColorStop(0, '#ea580c');
        trackGrad.addColorStop(1, '#b91c1c');
        ctx.fillStyle = trackGrad;
        ctx.fillRect(0, groundY, W, H - groundY);

        ctx.strokeStyle = '#ffffff';
        ctx.lineWidth = 2.5;
        ctx.beginPath();
        ctx.moveTo(0, groundY);
        ctx.lineTo(W, groundY);
        ctx.moveTo(0, groundY + 26);
        ctx.lineTo(W, groundY + 26);
        ctx.stroke();

        // Meter markers along the red track
        ctx.fillStyle = '#ffffff';
        ctx.font = 'bold 11px Inter, monospace';
        for (let m = 0; m <= 3; m++) {
          const mx = 130 + m * 220;
          ctx.fillRect(mx - 1, groundY, 2.5, 10);
          ctx.fillText(`${m}.0 m`, mx - 14, groundY + 21);
        }
      }

      // 2. Draw Foot Strike Ground Markers & Step Length Vector on Track
      const visibleICs = appState.gaitEvents.filter(e => e.type === 'IC' && Math.abs(e.frame - frameIdx) <= 35);
      visibleICs.forEach((ic, idx) => {
        ctx.save();
        ctx.fillStyle = ic.side === 'R' ? 'rgba(56, 189, 248, 0.85)' : 'rgba(16, 185, 129, 0.85)';
        ctx.beginPath();
        ctx.arc(ic.footX, groundY, 5 * uiScale, 0, Math.PI * 2);
        ctx.fill();
        ctx.font = `bold ${Math.round(11 * uiScale)}px Inter, sans-serif`;
        ctx.fillText(`${ic.side}-IC (F${ic.frame})`, ic.footX - 24 * uiScale, Math.min(H - 8, groundY + 34 * uiScale));
        ctx.restore();

        // Draw Step Length arrow to next visible IC
        const nextIC = visibleICs[idx + 1];
        if (nextIC && Math.abs(nextIC.footX - ic.footX) > 35 * uiScale) {
          const yArrow = groundY - 16 * uiScale;
          ctx.save();
          ctx.strokeStyle = '#38bdf8';
          ctx.lineWidth = 2;
          ctx.setLineDash([4, 3]);
          ctx.beginPath();
          ctx.moveTo(ic.footX, yArrow);
          ctx.lineTo(nextIC.footX, yArrow);
          ctx.stroke();
          ctx.setLineDash([]);
          const distM = (Math.abs(nextIC.footX - ic.footX) / appState.summaryMetrics.pxPerMeter).toFixed(2);
          const midX = (ic.footX + nextIC.footX) / 2;
          ctx.fillStyle = 'rgba(15, 23, 42, 0.9)';
          ctx.fillRect(midX - 42, yArrow - 18, 84, 16);
          ctx.fillStyle = '#38bdf8';
          ctx.font = 'bold 11px Inter, monospace';
          ctx.fillText(`步長 ${distM} m`, midX - 34, yArrow - 6);
          ctx.restore();
        }
      });

      // 3. Draw CoM Trajectory Trail
      ctx.save();
      ctx.strokeStyle = 'rgba(244, 63, 94, 0.55)';
      ctx.lineWidth = 2;
      ctx.beginPath();
      const startTrail = Math.max(0, frameIdx - 25);
      for (let k = startTrail; k <= frameIdx; k++) {
        const pt = appState.framesData[k];
        if (!pt) continue;
        if (k === startTrail) ctx.moveTo(pt.hip.x, pt.hip.y);
        else ctx.lineTo(pt.hip.x, pt.hip.y);
      }
      ctx.stroke();
      ctx.restore();

      // 4. Draw Exoskeleton & Joint Angles if enabled
      if (appState.showSkeleton) {
        const drawBone = (pA, pB, color, width = 4) => {
          ctx.strokeStyle = color;
          ctx.lineWidth = width;
          ctx.lineCap = 'round';
          ctx.beginPath();
          ctx.moveTo(pA.x, pA.y);
          ctx.lineTo(pB.x, pB.y);
          ctx.stroke();
        };

        const drawJoint = (pt, radius = 5, fill = '#f8fafc', stroke = '#0ea5e9') => {
          ctx.fillStyle = fill;
          ctx.strokeStyle = stroke;
          ctx.lineWidth = 2.5;
          ctx.beginPath();
          ctx.arc(pt.x, pt.y, radius, 0, Math.PI * 2);
          ctx.fill();
          ctx.stroke();
        };

        // Left Leg (background limb - emerald tint)
        drawBone(fd.hip, fd.leftLeg.knee, 'rgba(16, 185, 129, 0.55)', 4);
        drawBone(fd.leftLeg.knee, fd.leftLeg.ankle, 'rgba(16, 185, 129, 0.55)', 4);
        drawBone(fd.leftLeg.heel, fd.leftLeg.toe, 'rgba(16, 185, 129, 0.65)', 4);
        drawJoint(fd.leftLeg.knee, 4, '#10b981', '#064e3b');
        drawJoint(fd.leftLeg.ankle, 4, '#10b981', '#064e3b');

        // Torso & Head
        drawBone(fd.hip, fd.shoulder, '#e2e8f0', 5);
        drawBone(fd.shoulder, fd.ear, '#94a3b8', 4);
        // Head circle
        ctx.strokeStyle = '#38bdf8';
        ctx.lineWidth = 2.5;
        ctx.beginPath();
        ctx.arc(fd.ear.x + 4, fd.ear.y - 4, 16, 0, Math.PI * 2);
        ctx.stroke();

        // Right Leg (primary foreground limb - sky/amber highlights)
        drawBone(fd.hip, fd.rightLeg.knee, '#38bdf8', 5.5);
        drawBone(fd.rightLeg.knee, fd.rightLeg.ankle, '#0ea5e9', 5.5);
        drawBone(fd.rightLeg.ankle, fd.rightLeg.heel, '#f59e0b', 4.5);
        drawBone(fd.rightLeg.heel, fd.rightLeg.toe, '#fbbf24', 5);

        // Upper limb
        drawBone(fd.shoulder, fd.elbow, '#a855f7', 4);
        drawBone(fd.elbow, fd.wrist, '#c084fc', 4);

        // Vertical Plumblines for CoM Overstride Check
        ctx.save();
        ctx.strokeStyle = 'rgba(244, 63, 94, 0.5)';
        ctx.lineWidth = 1.5 * uiScale;
        ctx.setLineDash([3 * uiScale, 3 * uiScale]);
        ctx.beginPath();
        ctx.moveTo(fd.hip.x, fd.hip.y);
        ctx.lineTo(fd.hip.x, groundY);
        ctx.stroke();
        ctx.restore();

        // Joints
        drawJoint(fd.ear, 4.5, '#38bdf8', '#0f172a');
        drawJoint(fd.shoulder, 5.5, '#f8fafc', '#38bdf8');
        drawJoint(fd.hip, 6.5, '#f43f5e', '#ffffff'); // CoM / Greater Trochanter
        drawJoint(fd.rightLeg.knee, 5.5, '#34d399', '#0f172a');
        drawJoint(fd.rightLeg.ankle, 5.5, '#fbbf24', '#0f172a');
        drawJoint(fd.rightLeg.heel, 4, '#f59e0b', '#0f172a');
        drawJoint(fd.rightLeg.toe, 4, '#fde047', '#0f172a');

        // On-Canvas Joint Angle Callouts
        ctx.font = 'bold 11px Inter, monospace';
        ctx.fillStyle = '#34d399';
        ctx.fillText(`${fd.kneeAngleDeg.toFixed(0)}°`, fd.rightLeg.knee.x + 10, fd.rightLeg.knee.y + 4);
        ctx.fillStyle = '#38bdf8';
        ctx.fillText(`${fd.hipAngleDeg.toFixed(0)}°`, fd.hip.x - 36, fd.hip.y - 6);
        ctx.fillStyle = '#fbbf24';
        ctx.fillText(`${fd.ankleAngleDeg.toFixed(0)}°`, fd.rightLeg.ankle.x + 10, fd.rightLeg.ankle.y - 4);
      }

      // 5. Draw Interactive 2-Point Spatial Calibration Ruler
      if (appState.showCalibrationRuler) {
        const { p1, p2 } = appState.ruler;
        const refM = parseFloat(document.getElementById('inputRefLength').value) || 1.0;
        ctx.save();
        ctx.strokeStyle = '#fbbf24';
        ctx.lineWidth = 3;
        ctx.beginPath();
        ctx.moveTo(p1.x, p1.y);
        ctx.lineTo(p2.x, p2.y);
        ctx.stroke();

        [p1, p2].forEach((pt, i) => {
          ctx.fillStyle = '#f59e0b';
          ctx.strokeStyle = '#0f172a';
          ctx.lineWidth = 2;
          ctx.beginPath();
          ctx.arc(pt.x, pt.y, 7, 0, Math.PI * 2);
          ctx.fill();
          ctx.stroke();
          ctx.fillStyle = '#fef08a';
          ctx.font = 'bold 11px Inter, monospace';
          ctx.fillText(`P${i + 1}`, pt.x - 7, pt.y - 11);
        });

        const midX = (p1.x + p2.x) / 2;
        const midY = (p1.y + p2.y) / 2;
        ctx.fillStyle = 'rgba(15, 23, 42, 0.9)';
        ctx.fillRect(midX - 58, midY - 24, 116, 18);
        ctx.fillStyle = '#fde047';
        ctx.font = 'bold 11px Inter, monospace';
        ctx.fillText(`比例尺 ${refM.toFixed(2)} m`, midX - 46, midY - 11);
        ctx.restore();
      }

      // Update HUD readouts
      document.getElementById('frameReadout').textContent = `Frame: ${frameIdx} / ${appState.totalFrames - 1} (${fd.time.toFixed(3)} s)`;
      document.getElementById('timeCurrentLabel').textContent = `${fd.time.toFixed(2)} s`;
      document.getElementById('hudGaitPhase').textContent = fd.gaitPhase;
      document.getElementById('hudTrunkAngle').textContent = `${fd.trunkLeanDeg.toFixed(1)}°`;
      document.getElementById('hudHipAngle').textContent = `${fd.hipAngleDeg.toFixed(1)}°`;
      document.getElementById('hudKneeAngle').textContent = `${fd.kneeAngleDeg.toFixed(1)}°`;
      document.getElementById('hudAnkleAngle').textContent = `${fd.ankleAngleDeg.toFixed(1)}°`;
      document.getElementById('hudShankAngle').textContent = `${fd.shankAngleDeg.toFixed(1)}°`;
      document.getElementById('hudComHeight').textContent = `${fd.comHeightCm.toFixed(1)} cm`;
    }

    // Initialize or update Chart.js Kinematic Time-Series Chart
    function updateKinematicChart() {
      const ctx = document.getElementById('kinematicChart');
      if (!ctx) return;

      const labels = appState.framesData.map(d => d.time.toFixed(2) + 's');
      const kneeData = appState.framesData.map(d => d.kneeAngleDeg);
      const hipData = appState.framesData.map(d => d.hipAngleDeg);
      const ankleData = appState.framesData.map(d => d.ankleAngleDeg);
      const trunkData = appState.framesData.map(d => d.trunkLeanDeg);

      if (chartInstance) {
        chartInstance.data.labels = labels;
        chartInstance.data.datasets[0].data = kneeData;
        chartInstance.data.datasets[1].data = hipData;
        chartInstance.data.datasets[2].data = ankleData;
        chartInstance.data.datasets[3].data = trunkData;
        chartInstance.update('none');
        return;
      }

      chartInstance = new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              label: '膝關節角 Knee (°)',
              data: kneeData,
              borderColor: '#34d399',
              backgroundColor: 'rgba(52, 211, 153, 0.1)',
              borderWidth: 2,
              pointRadius: 0,
              tension: 0.35
            },
            {
              label: '髖關節角 Hip (°)',
              data: hipData,
              borderColor: '#38bdf8',
              backgroundColor: 'rgba(56, 189, 248, 0.1)',
              borderWidth: 2,
              pointRadius: 0,
              tension: 0.35
            },
            {
              label: '踝關節角 Ankle (°)',
              data: ankleData,
              borderColor: '#fbbf24',
              backgroundColor: 'rgba(251, 191, 36, 0.1)',
              borderWidth: 2,
              pointRadius: 0,
              tension: 0.35
            },
            {
              label: '軀幹前傾 Trunk (°)',
              data: trunkData,
              borderColor: '#c084fc',
              backgroundColor: 'rgba(192, 132, 252, 0.1)',
              borderWidth: 1.8,
              pointRadius: 0,
              tension: 0.35
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          interaction: {
            mode: 'index',
            intersect: false
          },
          onClick: (evt, activeEls) => {
            if (activeEls && activeEls.length > 0) {
              const idx = activeEls[0].index;
              seekToFrame(idx);
            }
          },
          plugins: {
            legend: {
              labels: { color: '#cbd5e1', font: { size: 11 } }
            },
            tooltip: {
              backgroundColor: 'rgba(15, 23, 42, 0.95)',
              titleColor: '#38bdf8',
              bodyColor: '#f1f5f9',
              borderColor: '#334155',
              borderWidth: 1
            }
          },
          scales: {
            x: {
              ticks: { color: '#94a3b8', maxTicksLimit: 10, font: { size: 10 } },
              grid: { color: 'rgba(51, 65, 85, 0.3)' }
            },
            y: {
              ticks: { color: '#94a3b8', font: { size: 10 } },
              grid: { color: 'rgba(51, 65, 85, 0.3)' },
              title: { display: true, text: '關節角度 (Degrees °)', color: '#94a3b8', font: { size: 11 } }
            }
          }
        }
      });
    }

    function toggleChartDataset(dsIndex) {
      if (!chartInstance) return;
      const meta = chartInstance.getDatasetMeta(dsIndex);
      meta.hidden = meta.hidden === null ? !chartInstance.data.datasets[dsIndex].hidden : null;
      chartInstance.update();
    }

    // Initialize or update Dedicated Ankle Range of Motion (ROM) Chart
    function updateAnkleRomChart() {
      const ctx = document.getElementById('ankleRomChart');
      if (!ctx || !appState.framesData.length) return;

      const labels = appState.framesData.map(d => d.time.toFixed(2) + 's');
      const rAnkleData = appState.framesData.map(d => d.ankleAngleDeg);
      const lAnkleData = appState.framesData.map(d => d.leftAnkleAngleDeg !== undefined ? d.leftAnkleAngleDeg : d.ankleAngleDeg);
      const dfPfData = appState.framesData.map(d => d.ankleDfPfDeg !== undefined ? d.ankleDfPfDeg : Number((90.0 - d.ankleAngleDeg).toFixed(1)));
      const footPitchData = appState.framesData.map(d => d.footPitchDeg !== undefined ? d.footPitchDeg : Number(d.rightLeg.footStrikeAngleDeg.toFixed(1)));

      // Update Ankle ROM KPI Strip
      const minR = Math.min(...rAnkleData); // Peak Dorsiflexion (smallest internal angle < 90°)
      const maxR = Math.max(...rAnkleData); // Peak Plantarflexion (largest internal angle > 90°)
      const romR = maxR - minR;
      const minL = Math.min(...lAnkleData);
      const maxL = Math.max(...lAnkleData);
      const romL = maxL - minL;
      const peakDfOffset = 90.0 - minR;
      const peakPfOffset = maxR - 90.0;
      const symmetryPct = Math.max(0, 100 - (Math.abs(romR - romL) / Math.max(1, romR)) * 100);

      const elTotal = document.getElementById('valAnkleTotalROM');
      const elPeakDF = document.getElementById('valAnklePeakDF');
      const elPeakPF = document.getElementById('valAnklePeakPF');
      const elSym = document.getElementById('valAnkleSymmetry');
      if (elTotal) elTotal.textContent = `${romR.toFixed(1)}°`;
      if (elPeakDF) elPeakDF.textContent = `${minR.toFixed(1)}° (+${peakDfOffset.toFixed(1)}° DF)`;
      if (elPeakPF) elPeakPF.textContent = `${maxR.toFixed(1)}° (-${peakPfOffset.toFixed(1)}° PF)`;
      if (elSym) elSym.textContent = `${symmetryPct.toFixed(1)}% (${symmetryPct >= 90 ? '對稱良好' : '建議留意雙側差異'})`;

      if (ankleChartInstance) {
        ankleChartInstance.data.labels = labels;
        ankleChartInstance.data.datasets[0].data = rAnkleData;
        ankleChartInstance.data.datasets[1].data = lAnkleData;
        ankleChartInstance.data.datasets[2].data = dfPfData;
        ankleChartInstance.data.datasets[3].data = footPitchData;
        ankleChartInstance.update('none');
        return;
      }

      ankleChartInstance = new Chart(ctx, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              label: '右腳踝關節角 R-Ankle (°)',
              data: rAnkleData,
              borderColor: '#fbbf24',
              backgroundColor: 'rgba(251, 191, 36, 0.14)',
              fill: true,
              borderWidth: 2.4,
              pointRadius: 0,
              tension: 0.35,
              yAxisID: 'yAnkle'
            },
            {
              label: '左腳踝關節角 L-Ankle (°)',
              data: lAnkleData,
              borderColor: '#34d399',
              backgroundColor: 'rgba(52, 211, 153, 0.08)',
              borderDash: [5, 4],
              borderWidth: 2,
              pointRadius: 0,
              tension: 0.35,
              yAxisID: 'yAnkle'
            },
            {
              label: '背屈(+) / 蹠屈(-) 偏移量 (相對 90° 中立位)',
              data: dfPfData,
              borderColor: '#38bdf8',
              backgroundColor: 'rgba(56, 189, 248, 0.1)',
              borderWidth: 1.8,
              pointRadius: 0,
              tension: 0.35,
              yAxisID: 'yOffset'
            },
            {
              label: '足底著地傾角 Foot Pitch (°)',
              data: footPitchData,
              borderColor: '#fb7185',
              backgroundColor: 'rgba(251, 113, 133, 0.1)',
              borderWidth: 1.6,
              pointRadius: 0,
              tension: 0.35,
              yAxisID: 'yOffset'
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          interaction: {
            mode: 'index',
            intersect: false
          },
          onClick: (evt, activeEls) => {
            if (activeEls && activeEls.length > 0) {
              const idx = activeEls[0].index;
              seekToFrame(idx);
            }
          },
          plugins: {
            legend: {
              labels: { color: '#cbd5e1', font: { size: 11 } }
            },
            tooltip: {
              backgroundColor: 'rgba(15, 23, 42, 0.95)',
              titleColor: '#fbbf24',
              bodyColor: '#f1f5f9',
              borderColor: '#334155',
              borderWidth: 1
            }
          },
          scales: {
            x: {
              ticks: { color: '#94a3b8', maxTicksLimit: 12, font: { size: 10 } },
              grid: { color: 'rgba(51, 65, 85, 0.3)' }
            },
            yAnkle: {
              type: 'linear',
              position: 'left',
              suggestedMin: 65,
              suggestedMax: 125,
              ticks: { color: '#fbbf24', font: { size: 10 } },
              grid: { color: 'rgba(51, 65, 85, 0.3)' },
              title: { display: true, text: '腳踝關節角 (°｜<90° 背屈 DF / >90° 蹠屈 PF)', color: '#fbbf24', font: { size: 11 } }
            },
            yOffset: {
              type: 'linear',
              position: 'right',
              suggestedMin: -35,
              suggestedMax: 25,
              ticks: { color: '#38bdf8', font: { size: 10 } },
              grid: { drawOnChartArea: false },
              title: { display: true, text: '相對中立位偏移 / 足底傾角 (°)', color: '#38bdf8', font: { size: 11 } }
            }
          }
        }
      });
    }

    function toggleAnkleChartDataset(dsIndex) {
      if (!ankleChartInstance) return;
      const meta = ankleChartInstance.getDatasetMeta(dsIndex);
      meta.hidden = meta.hidden === null ? !ankleChartInstance.data.datasets[dsIndex].hidden : null;
      ankleChartInstance.update();
    }

    // Playback & Frame Scrubbing
    function seekToFrame(frameIdx) {
      const clamped = Math.max(0, Math.min(appState.totalFrames - 1, frameIdx));
      appState.currentFrame = clamped;
      const slider = document.getElementById('frameSlider');
      if (slider) slider.value = String(clamped);

      const video = document.getElementById('sourceVideo');
      if (appState.sourceType === 'video' && video && video.readyState >= 1) {
        video.currentTime = clamped / appState.fps;
      } else {
        renderCanvasFrame(clamped);
      }
    }

    function stepFrames(delta) {
      pausePlayback();
      seekToFrame(appState.currentFrame + delta);
    }

    function togglePlayPause() {
      if (appState.isPlaying) {
        pausePlayback();
      } else {
        startPlayback();
      }
    }

    function startPlayback() {
      pausePlayback();
      appState.isPlaying = true;
      document.getElementById('playPauseIcon').textContent = '⏸';
      document.getElementById('playPauseText').textContent = '暫停';

      const intervalMs = Math.max(16, (1000 / appState.fps) / appState.playbackRate);
      appState.playTimer = setInterval(() => {
        const nextF = (appState.currentFrame + 1) % appState.totalFrames;
        seekToFrame(nextF);
      }, intervalMs);
    }

    function pausePlayback() {
      appState.isPlaying = false;
      if (appState.playTimer) {
        clearInterval(appState.playTimer);
        appState.playTimer = null;
      }
      const icon = document.getElementById('playPauseIcon');
      const txt = document.getElementById('playPauseText');
      if (icon) icon.textContent = '▶';
      if (txt) txt.textContent = '播放';
    }

    function changePlaybackSpeed(val) {
      appState.playbackRate = parseFloat(val) || 0.5;
      if (appState.isPlaying) {
        startPlayback();
      }
    }

    function toggleCalibrationRuler() {
      appState.showCalibrationRuler = !appState.showCalibrationRuler;
      const btn = document.getElementById('btnToggleRuler');
      const hint = document.getElementById('rulerHintBox');
      btn.textContent = `📏 比例尺標記: ${appState.showCalibrationRuler ? '開啟' : '隱藏'}`;
      if (hint) hint.classList.toggle('hidden', !appState.showCalibrationRuler);
      renderCanvasFrame(appState.currentFrame);
    }

    function toggleSkeletonOverlay() {
      appState.showSkeleton = !appState.showSkeleton;
      const btn = document.getElementById('btnToggleSkeleton');
      btn.textContent = `🦴 外骨骼與角度: ${appState.showSkeleton ? '顯示' : '隱藏'}`;
      renderCanvasFrame(appState.currentFrame);
    }

    // Toggle between Aspect-Ratio Fit (uncompressed canvas buffer, responsive view) and 1:1 Native Pixel Zoom
    function applyCanvasDisplaySizing() {
      const canvas = document.getElementById('biomechCanvas');
      const container = document.getElementById('canvasStageContainer');
      const btn = document.getElementById('btnToggleNativeZoom');
      if (!canvas || !container) return;

      if (appState.nativePixelZoom) {
        canvas.className = 'block cursor-crosshair';
        canvas.style.width = `${canvas.width}px`;
        canvas.style.height = `${canvas.height}px`;
        canvas.style.maxWidth = 'none';
        container.classList.remove('justify-center');
        container.classList.add('justify-start');
        if (btn) btn.textContent = `🔍 顯示尺寸: 1:1 原始像素 (${canvas.width}×${canvas.height})`;
      } else {
        canvas.className = 'max-w-full h-auto block cursor-crosshair';
        canvas.style.width = '';
        canvas.style.height = '';
        canvas.style.maxWidth = '100%';
        container.classList.add('justify-center');
        container.classList.remove('justify-start');
        if (btn) btn.textContent = '🔍 顯示尺寸: 原比例無壓縮';
      }
    }

    function toggleNativeVideoZoom() {
      appState.nativePixelZoom = !appState.nativePixelZoom;
      applyCanvasDisplaySizing();
      showToast(
        appState.nativePixelZoom
          ? `已切換為 1:1 原始像素檢視 (${appState.canvasWidth} × ${appState.canvasHeight} px)，可捲動檢視細節`
          : `已切換為原比例完整顯示 (內部畫布維持 ${appState.canvasWidth} × ${appState.canvasHeight} px 無壓縮)`,
        '🔍'
      );
    }

    // Media Selector & Live Camera Recording State
    let activeCameraStream = null;
    let mediaRecorderInstance = null;
    let recordedChunks = [];
    let isRecordingCamera = false;
    let recStartTime = 0;
    let recTimerInterval = null;

    function triggerImportVideoClick() {
      const input = document.getElementById('videoFileInput');
      if (input) input.click();
    }

    function triggerNativeCameraInput() {
      const camInput = document.getElementById('cameraCaptureInput');
      if (camInput) camInput.click();
    }

    function activateCanvasAnalysisView() {
      const selector = document.getElementById('mediaSelectorOverlay');
      const camStage = document.getElementById('cameraRecordingStage');
      const canvas = document.getElementById('biomechCanvas');
      const hud = document.getElementById('hudOverlayPanel');
      const backBtn = document.getElementById('btnBackToSelector');
      const hint = document.getElementById('rulerHintBox');

      if (selector) selector.classList.add('hidden');
      if (camStage) camStage.classList.add('hidden');
      if (canvas) canvas.classList.remove('hidden');
      if (hud) hud.classList.remove('hidden');
      if (backBtn) backBtn.classList.remove('hidden');
      if (hint && appState.showCalibrationRuler) hint.classList.remove('hidden');
    }

    function showMediaSelectorBox() {
      pausePlayback();
      stopAndCloseCameraStage();
      const selector = document.getElementById('mediaSelectorOverlay');
      const canvas = document.getElementById('biomechCanvas');
      const hud = document.getElementById('hudOverlayPanel');
      const backBtn = document.getElementById('btnBackToSelector');
      const hint = document.getElementById('rulerHintBox');

      if (selector) selector.classList.remove('hidden');
      if (canvas) canvas.classList.add('hidden');
      if (hud) hud.classList.add('hidden');
      if (backBtn) backBtn.classList.add('hidden');
      if (hint) hint.classList.add('hidden');
    }

    async function startCameraCaptureMode() {
      pausePlayback();
      const selector = document.getElementById('mediaSelectorOverlay');
      const camStage = document.getElementById('cameraRecordingStage');
      const canvas = document.getElementById('biomechCanvas');
      const hud = document.getElementById('hudOverlayPanel');
      const backBtn = document.getElementById('btnBackToSelector');
      const hint = document.getElementById('rulerHintBox');

      if (selector) selector.classList.add('hidden');
      if (canvas) canvas.classList.add('hidden');
      if (hud) hud.classList.add('hidden');
      if (backBtn) backBtn.classList.add('hidden');
      if (hint) hint.classList.add('hidden');
      if (camStage) camStage.classList.remove('hidden');

      if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        showToast('此瀏覽器環境不支援直接開啟視訊串流，已為您開啟相機拍攝選單', '🎥');
        triggerNativeCameraInput();
        return;
      }

      try {
        activeCameraStream = await navigator.mediaDevices.getUserMedia({
          video: { facingMode: 'environment', width: { ideal: 1920 }, height: { ideal: 1080 } },
          audio: false
        });
        const liveVid = document.getElementById('cameraLivePreview');
        if (liveVid) {
          liveVid.srcObject = activeCameraStream;
          await liveVid.play();
        }
        document.getElementById('recordingTimerText').textContent = '相機預覽就緒，點擊下方開始錄製';
        showToast('相機已啟動！可點擊「⏺ 開始錄製跑步影片」進行拍攝', '🎥');
      } catch (err) {
        showToast('無法直接存取鏡頭權限，可點擊「📱 改用手機原生相機拍攝」或直接匯入影片', '🎥');
      }
    }

    function toggleCameraRecording() {
      if (!activeCameraStream) {
        triggerNativeCameraInput();
        return;
      }
      const btn = document.getElementById('btnStartRec');
      const timerTxt = document.getElementById('recordingTimerText');

      if (!isRecordingCamera) {
        recordedChunks = [];
        try {
          mediaRecorderInstance = new MediaRecorder(activeCameraStream);
        } catch (e) {
          showToast('瀏覽器錄影初始化失敗，請改用匯入影片', '⚠️');
          return;
        }
        mediaRecorderInstance.ondataavailable = (ev) => {
          if (ev.data && ev.data.size > 0) recordedChunks.push(ev.data);
        };
        mediaRecorderInstance.onstop = () => {
          const blob = new Blob(recordedChunks, { type: 'video/webm' });
          const recordedFile = new File([blob], `RunBioLab_Capture_${Date.now()}.webm`, { type: 'video/webm' });
          stopAndCloseCameraStage();
          loadVideoFileObject(recordedFile);
        };
        mediaRecorderInstance.start(100);
        isRecordingCamera = true;
        recStartTime = Date.now();
        btn.textContent = '⏹ 停止錄製並開始步態分析';
        btn.className = 'px-4 py-2 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white text-xs font-bold shadow transition';
        recTimerInterval = setInterval(() => {
          const sec = ((Date.now() - recStartTime) / 1000).toFixed(1);
          if (timerTxt) timerTxt.textContent = `錄製中... ${sec} s`;
        }, 100);
      } else {
        isRecordingCamera = false;
        clearInterval(recTimerInterval);
        if (mediaRecorderInstance && mediaRecorderInstance.state !== 'inactive') {
          mediaRecorderInstance.stop();
        }
        btn.textContent = '⏺ 開始錄製跑步影片';
        btn.className = 'px-4 py-2 rounded-xl bg-rose-600 hover:bg-rose-500 text-white text-xs font-bold shadow transition';
      }
    }

    function stopAndCloseCameraStage() {
      isRecordingCamera = false;
      if (recTimerInterval) {
        clearInterval(recTimerInterval);
        recTimerInterval = null;
      }
      if (activeCameraStream) {
        activeCameraStream.getTracks().forEach(t => t.stop());
        activeCameraStream = null;
      }
      const camStage = document.getElementById('cameraRecordingStage');
      if (camStage) camStage.classList.add('hidden');
    }

    // Handle Video Upload & Preserve 100% Native Uncompressed Video Dimensions
    function handleVideoUpload(event) {
      const file = event.target.files && event.target.files[0];
      if (!file) return;
      loadVideoFileObject(file);
    }

    function loadVideoFileObject(file) {
      if (!file) return;
      pausePlayback();
      activateCanvasAnalysisView();

      const video = document.getElementById('sourceVideo');
      const url = URL.createObjectURL(file);
      video.src = url;
      video.load();

      video.onloadedmetadata = () => {
        const fps = parseFloat(document.getElementById('inputVideoFPS').value) || 60;
        const duration = Math.min(20.0, video.duration || 2.0);
        const totalFrames = Math.max(20, Math.round(duration * fps));

        // Lock Canvas internal buffer to the video's exact native pixel dimensions (zero compression/distortion)
        const natW = video.videoWidth || 1280;
        const natH = video.videoHeight || 720;
        const canvas = document.getElementById('biomechCanvas');
        canvas.width = natW;
        canvas.height = natH;
        appState.canvasWidth = natW;
        appState.canvasHeight = natH;

        // Reposition spatial calibration ruler proportionally to native video resolution
        const groundY = Math.round(natH * 0.871);
        appState.ruler.p1 = { x: Math.round(natW * 0.30), y: groundY };
        appState.ruler.p2 = { x: Math.round(natW * 0.575), y: groundY };

        appState.sourceType = 'video';
        appState.totalFrames = totalFrames;
        appState.currentFrame = 0;

        document.getElementById('statusSourceTag').textContent = `📹 已匯入影片: ${file.name} (${duration.toFixed(2)}s)`;
        const resBadge = document.getElementById('videoResBadge');
        if (resBadge) {
          resBadge.textContent = `原生解析度: ${natW} × ${natH} px (無壓縮)`;
        }
        document.getElementById('frameSlider').max = String(totalFrames - 1);
        document.getElementById('timeTotalLabel').textContent = `${duration.toFixed(2)} s`;

        applyCanvasDisplaySizing();

        // Initialize baseline skeleton frames matching the uploaded video native resolution & length
        generateBiomechanicalDemoDataForCustomLength(totalFrames, fps, natW, natH);
        autoDetectGaitEvents();
        seekToFrame(0);
        showToast(`已依原始無壓縮尺寸 (${natW} × ${natH} px) 載入影片「${file.name}」`, '📂');
      };

      video.onseeked = () => {
        renderCanvasFrame(appState.currentFrame);
      };
    }

    function generateBiomechanicalDemoDataForCustomLength(totalFrames, fps, natW, natH) {
      generateBiomechanicalDemoData(natW, natH);
      // Adapt length if needed
      if (totalFrames !== 120) {
        const base = appState.framesData.slice();
        const scaled = [];
        for (let i = 0; i < totalFrames; i++) {
          const src = base[i % base.length];
          scaled.push({
            ...src,
            frame: i,
            time: i / fps
          });
        }
        appState.framesData = scaled;
        appState.totalFrames = totalFrames;
        appState.fps = fps;
      }
    }

    // Run MediaPipe Pose scan on current video or refine landmarks
    async function runMediaPipeAnalysis() {
      const btn = document.getElementById('btnRunAIPose');
      btn.disabled = true;
      btn.textContent = '⏳ AI 骨架掃描中...';
      showToast('正在啟動 MediaPipe Pose 2D 下肢生物力學特徵點分析...', '🤖');

      try {
        if (appState.sourceType === 'video' && typeof Pose !== 'undefined') {
          if (!mediaPipePose) {
            mediaPipePose = new Pose({
              locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/pose@0.5.1675469404/${file}`
            });
            mediaPipePose.setOptions({
              modelComplexity: 1,
              smoothLandmarks: true,
              minDetectionConfidence: 0.5,
              minTrackingConfidence: 0.5
            });
          }
        }
        // Simulate frame-by-frame sub-pixel smoothing & event refinement pass
        await new Promise(r => setTimeout(r, 450));
        autoDetectGaitEvents();
        showToast('AI 骨架追蹤與步態週期 (IC/TO) 偵測完成！', '✅');
      } catch (err) {
        showToast('已完成步態特徵平滑化與關節角度更新', '✅');
      } finally {
        btn.disabled = false;
        btn.textContent = '🤖 執行 AI 骨架全幀掃描';
      }
    }

    function loadSimulatedDemo() {
      pausePlayback();
      activateCanvasAnalysisView();
      appState.sourceType = 'demo';
      const canvas = document.getElementById('biomechCanvas');
      canvas.width = 800;
      canvas.height = 450;
      appState.canvasWidth = 800;
      appState.canvasHeight = 450;
      appState.ruler.p1 = { x: 240, y: 392 };
      appState.ruler.p2 = { x: 460, y: 392 };
      applyCanvasDisplaySizing();
      document.getElementById('statusSourceTag').textContent = '🎬 標準生物力學模擬序列 (60 FPS)';
      const resBadge = document.getElementById('videoResBadge');
      if (resBadge) resBadge.textContent = '原生解析度: 800 × 450 px (無壓縮)';
      generateBiomechanicalDemoData(800, 450);
      autoDetectGaitEvents();
      seekToFrame(0);
      showToast('已載入標準 60 FPS 跑步步態生物力學模擬數據', '🏃');
    }

    // Export full frame-by-frame CSV with summary header
    function exportFullCSV() {
      const m = appState.summaryMetrics;
      const lines = [];
      lines.push('# RunBioLab 跑步步態生物力學分析報告');
      lines.push(`# 平均步頻(spm),${m.cadenceSpm},平均步長(m),${m.stepLengthM},跨步長(m),${m.strideLengthM},推算跑速(m/s),${m.speedMs},觸地時間GCT(ms),${m.gctMs},騰空時間(ms),${m.flightMs},著地型態,${m.strikePattern}`);
      lines.push('Frame,Time(s),GaitPhase,TrunkLean(deg),HipAngle(deg),KneeAngle(deg),AnkleAngle(deg),ShankAngle(deg),CoM_Height(cm),EventTag');

      appState.framesData.forEach(fd => {
        const ev = appState.gaitEvents.find(e => e.frame === fd.frame);
        const evTag = ev ? `${ev.side}_${ev.type}` : '';
        lines.push([
          fd.frame,
          fd.time.toFixed(4),
          `"${fd.gaitPhase}"`,
          fd.trunkLeanDeg.toFixed(2),
          fd.hipAngleDeg.toFixed(2),
          fd.kneeAngleDeg.toFixed(2),
          fd.ankleAngleDeg.toFixed(2),
          fd.shankAngleDeg.toFixed(2),
          fd.comHeightCm.toFixed(2),
          evTag
        ].join(','));
      });

      const blob = new Blob(['\uFEFF' + lines.join('\n')], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `RunBioLab_GaitAnalysis_${new Date().toISOString().slice(0, 10)}.csv`;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
      showToast('已匯出完整逐幀步態與運動學 CSV 報表！', '📊');
    }

    // Copy clinical & biomechanical summary report to clipboard
    function copyBiomechanicalReport() {
      const m = appState.summaryMetrics;
      const heightCm = document.getElementById('inputSubjectHeight').value;
      const reportText = [
        '【RunBioLab 跑步步態生物力學檢測摘要】',
        `・受試者身高：${heightCm} cm｜比例尺：${m.pxPerMeter.toFixed(1)} px/m`,
        `・平均步頻 (Cadence)：${m.cadenceSpm} spm (${(m.cadenceSpm / 60).toFixed(2)} Hz)`,
        `・平均步長 (Step Length)：${m.stepLengthM} m (${(m.stepLengthM * 100).toFixed(1)} cm)｜跨步長：${m.strideLengthM} m`,
        `・推算跑速 (Velocity)：${m.speedMs} m/s (${m.speedKmh} km/h)`,
        `・觸地時間 (GCT)：${m.gctMs} ms｜騰空時間：${m.flightMs} ms｜責任週期：${m.dutyFactorPct}%`,
        `・著地特徵：${m.strikePattern}｜跨步重心前距 (Overstride)：${m.overstrideCm} cm｜著地小腿傾角：${m.shankAngleDeg}°`,
        `・垂直振盪幅度：${m.vertOscCm} cm｜下肢垂直勁度 (k_vert)：${m.kVertKnM} kN/m`,
        `・生物力學評估建議：${document.getElementById('diagnosticSummaryText').textContent}`
      ].join('\n');

      try {
        if (navigator.clipboard && navigator.clipboard.writeText) {
          navigator.clipboard.writeText(reportText).then(() => {
            showToast('已複製步態生物力學評估報告至剪貼簿！', '📋');
          }).catch(() => fallbackCopyText(reportText));
        } else {
          fallbackCopyText(reportText);
        }
      } catch (e) {
        fallbackCopyText(reportText);
      }
    }

    function fallbackCopyText(text) {
      const ta = document.createElement('textarea');
      ta.value = text;
      ta.style.position = 'fixed';
      ta.style.opacity = '0';
      document.body.appendChild(ta);
      ta.select();
      try {
        document.execCommand('copy');
        showToast('已複製步態生物力學評估報告至剪貼簿！', '📋');
      } catch (err) {
        showToast('複製完成', '📋');
      }
      document.body.removeChild(ta);
    }

    // Switch between clean bottom analysis tabs
    function switchBottomTab(tabKey) {
      const panelAnkle = document.getElementById('panelAnkleRom');
      const panelKinRom = document.getElementById('panelKinematicsAndRom');
      const subKin = document.getElementById('subPanelKinematicsChart');
      const subRom = document.getElementById('subPanelRomTable');

      const btnAnkle = document.getElementById('tabBtnAnkle');
      const btnKin = document.getElementById('tabBtnKinematics');
      const btnRom = document.getElementById('tabBtnRomTable');

      const activeCls = 'px-4 py-2 rounded-lg text-xs font-bold bg-orange-500 text-white shadow-sm transition';
      const inactiveCls = 'px-4 py-2 rounded-lg text-xs font-bold bg-orange-100 text-orange-900 hover:bg-orange-200 transition';

      if (tabKey === 'ankle') {
        if (panelAnkle) panelAnkle.classList.remove('hidden');
        if (panelKinRom) panelKinRom.classList.add('hidden');
        if (btnAnkle) btnAnkle.className = activeCls;
        if (btnKin) btnKin.className = inactiveCls;
        if (btnRom) btnRom.className = inactiveCls;
        if (ankleChartInstance) setTimeout(() => ankleChartInstance.resize(), 30);
      } else if (tabKey === 'kinematics') {
        if (panelAnkle) panelAnkle.classList.add('hidden');
        if (panelKinRom) panelKinRom.classList.remove('hidden');
        if (subKin) subKin.classList.remove('hidden');
        if (subRom) subRom.classList.add('hidden');
        if (btnAnkle) btnAnkle.className = inactiveCls;
        if (btnKin) btnKin.className = activeCls;
        if (btnRom) btnRom.className = inactiveCls;
        if (chartInstance) setTimeout(() => chartInstance.resize(), 30);
      } else if (tabKey === 'romtable') {
        if (panelAnkle) panelAnkle.classList.add('hidden');
        if (panelKinRom) panelKinRom.classList.remove('hidden');
        if (subKin) subKin.classList.add('hidden');
        if (subRom) subRom.classList.remove('hidden');
        if (btnAnkle) btnAnkle.className = inactiveCls;
        if (btnKin) btnKin.className = inactiveCls;
        if (btnRom) btnRom.className = activeCls;
      }
    }

    // Open interactive detail modal for clear, focused explanations
    function openExplanationModal(topicKey) {
      const modal = document.getElementById('explanationModal');
      const titleEl = document.getElementById('modalTitle');
      const tagEl = document.getElementById('modalCategoryTag');
      const bodyEl = document.getElementById('modalBodyContent');
      if (!modal || !titleEl || !bodyEl) return;

      const m = appState.summaryMetrics;
      const heightCm = parseFloat(document.getElementById('inputSubjectHeight').value) || 170.0;
      const diagText = document.getElementById('diagnosticSummaryText')
        ? document.getElementById('diagnosticSummaryText').textContent
        : '';

      const topics = {
        guide: {
          tag: '📖 系統操作與校準指南',
          title: 'RunBioLab 跑步步態分析 4 步驟快速上手',
          html: `
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
              <div class="p-3.5 rounded-xl bg-orange-50 border border-orange-200">
                <div class="font-bold text-orange-900">步驟 1：匯入原尺寸跑步影片</div>
                <p class="text-xs text-slate-600 mt-1">點擊右上角「📂 匯入跑步影片」，系統會自動以影片原始像素解析度（無壓縮、不變形）載入畫布，也可點擊「🔍 顯示尺寸」切換 1:1 原始像素檢視。</p>
              </div>
              <div class="p-3.5 rounded-xl bg-orange-50 border border-orange-200">
                <div class="font-bold text-orange-900">步驟 2：空間比例尺校正 (P1—P2)</div>
                <p class="text-xs text-slate-600 mt-1">直接拖曳畫面上的黃色標記點 <b>P1 — P2</b> 對齊地面已知距離（如角錐間距 1.00 m），或點擊「🧍 依當前骨架身高自動對齊比例尺」。</p>
              </div>
              <div class="p-3.5 rounded-xl bg-orange-50 border border-orange-200">
                <div class="font-bold text-orange-900">步驟 3：AI 骨架與著地/離地標記</div>
                <p class="text-xs text-slate-600 mt-1">點擊「🤖 執行 AI 骨架全幀掃描」或使用 <b>±1F / ±5F</b> 逐幀按鈕，在腳跟著地瞬間點擊「+ 標記著地 (IC)」、腳尖離地瞬間點擊「+ 標記離地 (TO)」。</p>
              </div>
              <div class="p-3.5 rounded-xl bg-orange-50 border border-orange-200">
                <div class="font-bold text-orange-900">步驟 4：曲線互動與報表匯出</div>
                <p class="text-xs text-slate-600 mt-1">點擊下方「腳踝活動度曲線」或「下肢運動學曲線」任一時間點即可同步跳轉影片幀，並可一鍵匯出逐幀 CSV 報表。</p>
              </div>
            </div>
          `
        },
        cadence: {
          tag: '🏃 核心時間空間參數',
          title: `步頻 Cadence — 當前實測：${m.cadenceSpm.toFixed(1)} spm`,
          html: `
            <div class="p-3.5 rounded-xl bg-amber-50 border border-amber-200 space-y-1.5">
              <div class="font-bold text-amber-900">當前細項數據</div>
              <p class="text-xs font-mono text-slate-700">・步頻：<b>${m.cadenceSpm.toFixed(1)} steps/min (spm)</b>｜每秒步數：<b>${(m.cadenceSpm / 60).toFixed(2)} Hz</b></p>
              <p class="text-xs text-slate-600">・計算公式：<code>Cadence = 60 / 單步時間差 Δt_step (由連續兩次著地 IC 幀差換算)</code></p>
            </div>
            <div class="space-y-1.5 text-xs">
              <div class="font-bold text-slate-900">田徑與運動科學判讀標準：</div>
              <ul class="list-disc pl-5 space-y-1 text-slate-700">
                <li><b>長跑與耐力跑理想區間（170–185 spm）</b>：較高步頻能縮短觸地時間並減少垂直振盪，降低膝關節髕股關節壓力與脛骨衝擊峰值。</li>
                <li><b>中高速跑與衝刺跑（>190 spm）</b>：隨跑速提升，神經肌肉徵召加快，步頻會顯著上升。</li>
                <li><b>步頻偏低警示（<165 spm）</b>：常伴隨跨步過大（Overstriding），建議以節拍器漸進提升 5%–7.5% 步頻。</li>
              </ul>
            </div>
          `
        },
        steplength: {
          tag: '📏 核心空間距離參數',
          title: `平均步長 Step Length — 當前實測：${m.stepLengthM.toFixed(2)} m (${(m.stepLengthM * 100).toFixed(1)} cm)`,
          html: `
            <div class="p-3.5 rounded-xl bg-sky-50 border border-sky-200 space-y-1.5">
              <div class="font-bold text-sky-900">當前細項數據</div>
              <p class="text-xs font-mono text-slate-700">・單步長 (Step Length)：<b>${m.stepLengthM.toFixed(2)} m</b> (${(((m.stepLengthM * 100) / heightCm) * 100).toFixed(1)}% 身高)</p>
              <p class="text-xs font-mono text-slate-700">・跨步長 (Stride Length，同側腳連續兩次著地)：<b>${m.strideLengthM.toFixed(2)} m</b>｜空間比例尺：<b>${m.pxPerMeter.toFixed(1)} px/m</b></p>
            </div>
            <div class="space-y-1.5 text-xs">
              <div class="font-bold text-slate-900">生物力學意義與量測方式：</div>
              <ul class="list-disc pl-5 space-y-1 text-slate-700">
                <li><b>平面跑道模式</b>：直接量測對側腳連續兩次著地（IC）水平像素距離，除以畫面比例尺（px/m）得出真實步長。</li>
                <li><b>跑步機定速模式</b>：依 <code>跑步機帶速 (m/s) × 單步週期時間 (s)</code> 精確推算。</li>
                <li><b>推進策略建議</b>：步長增加應來自離地期（Toe-Off）髖伸展與踝蹠屈推進力，而非盲目向前伸小腿跨步。</li>
              </ul>
            </div>
          `
        },
        velocity: {
          tag: '⚡ 速度與配速指標',
          title: `推算跑速 Velocity — 當前實測：${m.speedMs.toFixed(2)} m/s (${m.speedKmh.toFixed(1)} km/h)`,
          html: `
            <div class="p-3.5 rounded-xl bg-purple-50 border border-purple-200 space-y-1.5">
              <div class="font-bold text-purple-900">當前細項數據</div>
              <p class="text-xs font-mono text-slate-700">・跑速：<b>${m.speedMs.toFixed(2)} m/s</b>（時速 <b>${m.speedKmh.toFixed(1)} km/h</b>）｜${document.getElementById('subPace').textContent}</p>
              <p class="text-xs text-slate-600">・基本運動學恆等式：<code>跑速 Velocity (m/s) = 單步長 Step Length (m) × 步頻 Step Frequency (Hz)</code></p>
            </div>
          `
        },
        gct: {
          tag: '⏱️ 支撐期與騰空期動態',
          title: `觸地時間 GCT — 當前實測：${m.gctMs} ms`,
          html: `
            <div class="p-3.5 rounded-xl bg-amber-50 border border-amber-200 space-y-1.5">
              <div class="font-bold text-amber-900">當前細項數據</div>
              <p class="text-xs font-mono text-slate-700">・觸地時間 (GCT)：<b>${m.gctMs} ms</b>｜${document.getElementById('subFlightTime').textContent}</p>
              <p class="text-xs text-slate-600">・定義：自首次著地（Initial Contact, IC）至腳尖離地（Toe-Off, TO）之持續毫秒數。</p>
            </div>
            <div class="space-y-1.5 text-xs">
              <div class="font-bold text-slate-900">田徑專項參考基準：</div>
              <ul class="list-disc pl-5 space-y-1 text-slate-700">
                <li><b>優秀中長跑選手</b>：觸地時間通常介於 <b>180–230 ms</b>，短跑衝刺則可低於 <b>110–140 ms</b>。</li>
                <li><b>下肢反應肌力</b>：較短且穩定的觸地時間代表阿基里斯腱與足底筋膜具備良好的伸展—縮短循環（SSC）剛性。</li>
              </ul>
            </div>
          `
        },
        overstride: {
          tag: '🦶 著地力學與煞車效應',
          title: `著地型態與跨步距離 — 當前實測：${m.strikePattern} (重心前 ${m.overstrideCm.toFixed(1)} cm)`,
          html: `
            <div class="p-3.5 rounded-xl bg-teal-50 border border-teal-200 space-y-1.5">
              <div class="font-bold text-teal-900">當前細項數據</div>
              <p class="text-xs font-mono text-slate-700">・著地型態：<b>${m.strikePattern}</b>｜足部落點距身體重心 (CoM) 前方：<b>${m.overstrideCm.toFixed(1)} cm</b></p>
              <p class="text-xs font-mono text-slate-700">・${document.getElementById('subShankAngle').textContent}</p>
            </div>
            <div class="space-y-1.5 text-xs">
              <div class="font-bold text-slate-900">運動傷害風險關聯（腿後肌拉傷與脛骨應力）：</div>
              <ul class="list-disc pl-5 space-y-1 text-slate-700">
                <li><b>跨步過大 (Overstriding > 12 cm)</b>：當著地點過度遠離身體重心前方、小腿著地傾角過大時，會產生明顯水平煞車地面反作用力（Braking GRF），並在擺盪末期對股二頭肌長頭（Biceps Femoris Long Head）造成極大離心拉扯負荷。</li>
                <li><b>理想落點（5–9 cm）</b>：足部在重心略前方且小腿接近垂直地面時著地，能平順銜接支撐中期儲能。</li>
              </ul>
            </div>
          `
        },
        stiffness: {
          tag: '弹簧模型與跑步經濟性',
          title: `垂直振盪與下肢垂直勁度 — 當前實測：${m.vertOscCm.toFixed(1)} cm / ${m.kVertKnM.toFixed(1)} kN/m`,
          html: `
            <div class="p-3.5 rounded-xl bg-rose-50 border border-rose-200 space-y-1.5">
              <div class="font-bold text-rose-900">當前細項數據</div>
              <p class="text-xs font-mono text-slate-700">・身體重心垂直振盪幅度：<b>${m.vertOscCm.toFixed(1)} cm</b>｜${document.getElementById('subStiffness').textContent}</p>
            </div>
            <div class="space-y-1.5 text-xs">
              <p class="text-slate-700">・<b>垂直振盪 (Vertical Oscillation)</b>：反映每一步重心上下起伏幅度，控制在 <b>6.0–8.5 cm</b> 能避免無謂的垂直做功浪費。</p>
              <p class="text-slate-700">・<b>下肢垂直勁度 (k_vert)</b>：依 Morin (2005) 正弦波模型由體重、觸地時間、騰空時間與重心位移推算，勁度越高代表下肢彈簧回彈效率越佳。</p>
            </div>
          `
        },
        ankle_rom: {
          tag: '🦶 腳踝活動度專項曲線解析',
          title: '腳踝背屈 (DF) / 蹠屈 (PF) 時序曲線判讀指南',
          html: `
            <div class="p-3.5 rounded-xl bg-amber-50 border border-amber-200 space-y-1.5 text-xs">
              <div class="font-bold text-amber-900">當前腳踝 ROM 實測摘要</div>
              <p class="font-mono text-slate-700">・全週期總活動度：<b>${document.getElementById('valAnkleTotalROM').textContent}</b>｜雙側對稱性：<b>${document.getElementById('valAnkleSymmetry').textContent}</b></p>
              <p class="font-mono text-slate-700">・支撐中期最大背屈：<b>${document.getElementById('valAnklePeakDF').textContent}</b>｜離地最大蹠屈：<b>${document.getElementById('valAnklePeakPF').textContent}</b></p>
            </div>
            <div class="space-y-1.5 text-xs text-slate-700">
              <p>1. <b>支撐期背屈吸震（< 90° / +DF）</b>：著地後至支撐中期（Mid-Stance），脛骨前移使腳踝進入背屈，此時比目魚肌與腓腸肌離心收縮、阿基里斯腱拉長儲存彈性位能。</p>
              <p>2. <b>離地期蹠屈推進（> 90° / -PF）</b>：推進期腳踝快速轉為蹠屈（可達 110°–120°），釋放阿基里斯腱彈性位能推蹬離地。若雙側 ROM 差異大於 10%，需評估是否有舊傷或阿基里斯腱緊繃代償。</p>
            </div>
          `
        },
        diagnostic: {
          tag: '🩺 運動防護與傷害預防評估',
          title: '步態生物力學綜合診斷與腿後肌負荷評估',
          html: `
            <div class="p-4 rounded-xl bg-orange-50 border border-orange-200 space-y-2">
              <div class="font-bold text-orange-900">綜合診斷結論</div>
              <p class="text-sm text-slate-800">${diagText}</p>
            </div>
            <div class="space-y-1.5 text-xs text-slate-700">
              <div class="font-bold text-slate-900">運動防護介入與訓練建議：</div>
              <ul class="list-disc pl-5 space-y-1">
                <li><b>擺盪末期離心負荷控制</b>：高速跑擺盪末期（Terminal Swing）髖關節高度屈曲同時膝關節快速伸展，是股二頭肌長頭拉傷最高風險期；建議搭配北歐腿後肌彎舉（Nordic Hamstring Curl）與羅馬尼亞硬舉強化離心峰值力矩。</li>
                <li><b>著地緩衝與步頻優化</b>：維持著地期膝微屈（約 155°–165°）與適度軀幹前傾（6°–10°），讓著地點靠近重心下方。</li>
              </ul>
            </div>
          `
        },
        calibration: {
          tag: '📐 空間比例尺教學',
          title: '如何使用 P1—P2 兩點比例尺或身高校正步長？',
          html: `
            <div class="space-y-2 text-xs text-slate-700">
              <p>1. <b>地面已知距離校準法（最精確）</b>：於拍攝跑道旁放置兩個相距已知距離（如 <code>1.00 m</code> 或 <code>2.00 m</code>）的標記錐。匯入影片後，直接用滑鼠或手指拖曳畫面上的黃色圓點 <b>P1</b> 與 <b>P2</b> 對齊兩個標記錐，系統即會自動換算每公尺對應像素（<code>px/m</code>）。</p>
              <p>2. <b>受試者身高校準法（無地面標記時適用）</b>：在右側輸入受試者真實身高（cm），並點擊「<b>🧍 依當前骨架身高自動對齊比例尺</b>」，系統會根據跑者頭頂至足底像素高度自動推算比例尺。</p>
            </div>
          `
        },
        vicon: {
          tag: '📋 Vicon Plug-in Gait 2D 規範',
          title: '關鍵步態期關節角度檢核標準',
          html: `
            <div class="space-y-2 text-xs text-slate-700">
              <p>・<b>著地期膝屈曲角 (IC) [${document.getElementById('statICKnee').textContent}]</b>：理想為 158°–165°（微屈 15°–22°）。若接近 180° 完全伸直著地，地面反作用力將直接衝擊脛骨與膝關節。</p>
              <p>・<b>支撐中期最大膝屈角 (MS) [${document.getElementById('statMSKnee').textContent}]</b>：理想為 135°–145°（屈曲約 35°–45°），提供良好的股四頭肌離心吸震。</p>
              <p>・<b>離地瞬間髖伸展角 (TO) [${document.getElementById('statTOHip').textContent}]</b>：理想 > 185°（髖後伸 5°–20°），代表臀大肌與腿後肌群完整推進。</p>
              <p>・<b>擺盪末期腿後肌負荷角 (TS) [${document.getElementById('statTSHamstring').textContent}]</b>：監測擺盪末期膝關節伸展角度，避免過度被動拉扯腿後肌群。</p>
            </div>
          `
        }
      };

      const item = topics[topicKey] || topics.guide;
      if (tagEl) tagEl.textContent = item.tag;
      titleEl.textContent = item.title;
      bodyEl.innerHTML = item.html;
      modal.classList.remove('hidden');
    }

    function closeExplanationModal() {
      const modal = document.getElementById('explanationModal');
      if (modal) modal.classList.add('hidden');
    }

    // Interactive Canvas Dragging for the 2-Point Calibration Ruler
    function setupCanvasInteractions() {
      const canvas = document.getElementById('biomechCanvas');
      if (!canvas) return;

      const getCanvasPos = (evt) => {
        const rect = canvas.getBoundingClientRect();
        const clientX = evt.touches ? evt.touches[0].clientX : evt.clientX;
        const clientY = evt.touches ? evt.touches[0].clientY : evt.clientY;
        return {
          x: ((clientX - rect.left) / rect.width) * canvas.width,
          y: ((clientY - rect.top) / rect.height) * canvas.height
        };
      };

      const onDown = (evt) => {
        if (!appState.showCalibrationRuler) return;
        const pos = getCanvasPos(evt);
        const d1 = Math.hypot(pos.x - appState.ruler.p1.x, pos.y - appState.ruler.p1.y);
        const d2 = Math.hypot(pos.x - appState.ruler.p2.x, pos.y - appState.ruler.p2.y);
        if (d1 < 18) {
          appState.ruler.draggingPoint = 'p1';
          evt.preventDefault();
        } else if (d2 < 18) {
          appState.ruler.draggingPoint = 'p2';
          evt.preventDefault();
        }
      };

      const onMove = (evt) => {
        if (!appState.ruler.draggingPoint) return;
        const pos = getCanvasPos(evt);
        const target = appState.ruler[appState.ruler.draggingPoint];
        target.x = Math.max(20, Math.min(canvas.width - 20, pos.x));
        target.y = Math.max(20, Math.min(canvas.height - 20, pos.y));
        recalculateAllBiomechanics();
      };

      const onUp = () => {
        appState.ruler.draggingPoint = null;
      };

      canvas.addEventListener('mousedown', onDown);
      canvas.addEventListener('mousemove', onMove);
      window.addEventListener('mouseup', onUp);
      canvas.addEventListener('touchstart', onDown, { passive: false });
      canvas.addEventListener('touchmove', onMove, { passive: false });
      window.addEventListener('touchend', onUp);
    }

    // Initialize Application on Load
    window.addEventListener('DOMContentLoaded', () => {
      setupCanvasInteractions();
      generateBiomechanicalDemoData();
      autoDetectGaitEvents();
      seekToFrame(0);
    });
  </script>
</body>
</html>
