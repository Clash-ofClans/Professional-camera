<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>📷 الكاميرا الفورية</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/lucide@latest"></script>
  <style>
    body {
      font-family: 'Inter', sans-serif;
      background-color: #000;
      color: #fff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
      overflow: hidden;
    }
    #camera-container {
      width: 95%;
      max-width: 450px;
      height: 90vh;
      background-color: #111;
      border-radius: 2rem;
      box-shadow: 0 0 0 8px #222, 0 15px 40px rgba(0,0,0,0.9);
      position: relative;
      overflow: hidden;
      display: flex;
      flex-direction: column;
    }
    #video-feed {
      width: 100%;
      height: 100%;
      object-fit: cover;
      transform: scaleX(-1);
      display: none;
    }
    #status-overlay {
      position: absolute;
      inset: 0;
      background-color: rgba(0, 0, 0, 0.95);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      z-index: 50;
      text-align: center;
      font-size: 1.25rem;
      padding: 1rem;
    }
    #capture-btn {
      background-color: white;
      border: 8px solid rgba(255, 255, 255, 0.3);
      width: 65px;
      height: 65px;
      transition: transform 0.1s;
    }
    #capture-btn:active {
      transform: scale(0.85);
      border-color: rgba(255, 255, 255, 0.5);
    }
    #preview-screen {
      position: absolute;
      inset: 0;
      background-color: #000;
      z-index: 30;
      padding: 1rem;
    }
  </style>
</head>
<body>
  <div id="camera-container">
    <!-- شاشة الحالة -->
    <div id="status-overlay">
      <svg class="animate-spin h-10 w-10 text-yellow-400 mb-4" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
      </svg>
      <p id="status-text" class="text-xl font-bold">جاري فتح الكاميرا...</p>
    </div>

    <!-- بث الكاميرا -->
    <video id="video-feed" autoplay playsinline></video>

    <!-- أدوات التصوير -->
    <div id="control-bar" class="absolute bottom-0 w-full bg-black/60 backdrop-blur-md p-4 flex justify-center items-center space-x-6 space-x-reverse">
      <div onclick="showCapturedImage()" class="w-10 h-10 bg-gray-600 rounded-full border-2 border-white/50 flex items-center justify-center cursor-pointer hover:scale-105 transition">
        <i data-lucide="image" class="w-5 h-5 text-white"></i>
      </div>
      <button id="capture-btn" onclick="takePhoto()" class="rounded-full focus:outline-none"></button>
      <div class="w-10 h-10"></div>
    </div>

    <!-- شاشة المعاينة -->
    <div id="preview-screen" class="hidden flex-col justify-between">
      <h3 class="text-center text-xl font-bold mt-4 mb-4 text-gray-200">الصورة الملتقطة</h3>
      <img id="captured-image" class="w-full h-auto max-h-[70%] object-contain rounded-xl shadow-lg mb-6 mx-auto flex-grow" alt="الصورة الملتقطة">
      <div class="flex justify-center space-x-4 space-x-reverse">
        <button onclick="hideCapturedImage()" class="bg-teal-600 hover:bg-teal-500 text-white font-bold py-2 px-6 rounded-lg transition">العودة للكاميرا</button>
        <button onclick="deletePhotoFromPreview()" class="bg-red-600 hover:bg-red-500 text-white font-bold py-2 px-6 rounded-lg transition">حذف الصورة</button>
      </div>
    </div>
  </div>

  <canvas id="photo-canvas" style="display:none;"></canvas>

  <script>
    lucide.createIcons();

    const videoFeed = document.getElementById('video-feed');
    const statusOverlay = document.getElementById('status-overlay');
    const statusText = document.getElementById('status-text');
    const photoCanvas = document.getElementById('photo-canvas');
    const previewScreen = document.getElementById('preview-screen');
    const capturedImageDisplay = document.getElementById('captured-image');

    let lastCapturedImageData = null;

    window.onerror = (msg) => {
      statusText.innerHTML = `<p class="text-red-400 font-bold">${msg}</p>`;
    };

    function showError(msg) {
      statusText.innerHTML = `<p class="text-red-400 font-bold">${msg}</p>`;
    }

    async function startCamera() {
      if (location.protocol !== 'https:' && location.hostname !== 'localhost') {
        showError('⚠️ يجب فتح الصفحة عبر HTTPS أو localhost ليعمل الكاميرا.');
        return;
      }

      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' } });
        videoFeed.srcObject = stream;
        videoFeed.onloadedmetadata = () => {
          videoFeed.style.display = 'block';
          statusOverlay.classList.add('hidden');
        };
      } catch (error) {
        console.error(error);
        if (error.name === 'NotAllowedError') showError('❌ تم رفض إذن الكاميرا.');
        else if (error.name === 'NotFoundError') showError('📷 لا توجد كاميرا متصلة.');
        else showError(`❌ خطأ: ${error.message}`);
      }
    }

    function takePhoto() {
      if (!videoFeed.srcObject) return;
      photoCanvas.width = videoFeed.videoWidth;
      photoCanvas.height = videoFeed.videoHeight;
      const ctx = photoCanvas.getContext('2d');
      ctx.translate(photoCanvas.width, 0);
      ctx.scale(-1, 1);
      ctx.drawImage(videoFeed, 0, 0);
      const dataUrl = photoCanvas.toDataURL('image/jpeg', 1.0);
      lastCapturedImageData = dataUrl;
      capturedImageDisplay.src = dataUrl;
      showCapturedImage();
    }

    function showCapturedImage() {
      if (!lastCapturedImageData) return;
      previewScreen.classList.remove('hidden');
      previewScreen.classList.add('flex');
    }

    function hideCapturedImage() {
      previewScreen.classList.add('hidden');
      previewScreen.classList.remove('flex');
    }

    function deletePhotoFromPreview() {
      lastCapturedImageData = null;
      capturedImageDisplay.src = "";
      hideCapturedImage();
    }

    startCamera();
  </script>
</body>
</html>
