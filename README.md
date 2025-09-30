<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>📷 الكاميرا الفورية</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- تحميل أيقونات Lucide -->
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
      /* يعكس الصورة لتبدو مثل المرآة (للكاميرا الأمامية) */
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
    <!-- شاشة العد التنازلي للمؤقت الذاتي -->
    <div id="countdown-overlay" class="absolute inset-0 flex items-center justify-center z-40 bg-black/30 hidden">
        <span id="countdown-text" class="text-[8rem] sm:text-[10rem] font-extrabold text-white opacity-90 animate-pulse drop-shadow-2xl">5</span>
    </div>

    <!-- شاشة الحالة (تظهر عند الإطلاق أو الخطأ) -->
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
      <!-- زر المؤقت الذاتي -->
      <button id="timer-btn" onclick="toggleTimer()" class="w-10 h-10 rounded-full flex items-center justify-center cursor-pointer hover:scale-105 transition border-2 border-white/50 text-white">
        <i data-lucide="timer" class="w-5 h-5"></i>
      </button>

      <!-- زر المعاينة للصور الملتقطة -->
      <div id="image-preview-btn" onclick="showCapturedImage()" class="w-10 h-10 bg-gray-600 rounded-full border-2 border-white/50 flex items-center justify-center cursor-pointer hover:scale-105 transition">
        <i data-lucide="image" class="w-5 h-5 text-white"></i>
      </div>
      
      <!-- زر التصوير الرئيسي (يبدأ العد التنازلي إذا كان المؤقت مفعّلاً) -->
      <button id="capture-btn" onclick="startCaptureSequence()" class="rounded-full focus:outline-none"></button>
      
      <div class="w-10 h-10"></div> <!-- للحفاظ على التوازن -->
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
    // تهيئة أيقونات Lucide
    lucide.createIcons();

    const videoFeed = document.getElementById('video-feed');
    const statusOverlay = document.getElementById('status-overlay');
    const statusText = document.getElementById('status-text');
    const photoCanvas = document.getElementById('photo-canvas');
    const previewScreen = document.getElementById('preview-screen');
    const capturedImageDisplay = document.getElementById('captured-image');
    const countdownOverlay = document.getElementById('countdown-overlay');
    const countdownText = document.getElementById('countdown-text');
    const timerBtn = document.getElementById('timer-btn');
    const captureBtn = document.getElementById('capture-btn');
    const imagePreviewBtn = document.getElementById('image-preview-btn');

    let lastCapturedImageData = null;
    let isTimerActive = false;
    let countdownInterval = null;

    // معالج أخطاء عام
    window.onerror = (msg) => {
      showError(`حدث خطأ عام: ${msg}`);
    };

    function showError(msg) {
      statusText.innerHTML = `<p class="text-red-400 font-bold">${msg}</p>`;
      statusOverlay.classList.remove('hidden');
      videoFeed.style.display = 'none';
    }

    // بدء تشغيل الكاميرا
    async function startCamera() {
      // إجبار على استخدام HTTPS أو localhost لأسباب أمنية
      if (location.protocol !== 'https:' && location.hostname !== 'localhost') {
        showError('⚠️ يجب فتح الصفحة عبر HTTPS أو localhost ليعمل الكاميرا.');
        return;
      }

      try {
        // طلب بث الفيديو من الكاميرا الأمامية
        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' } });
        videoFeed.srcObject = stream;
        videoFeed.onloadedmetadata = () => {
          // إخفاء شاشة الحالة عند بدء بث الفيديو بنجاح
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
    
    // التبديل بين وضع المؤقت الذاتي والإيقاف
    function toggleTimer() {
        if (countdownInterval) return; // لا يمكن التبديل أثناء العد
        
        isTimerActive = !isTimerActive;
        // تحديث مظهر زر المؤقت
        timerBtn.classList.toggle('bg-teal-600', isTimerActive);
        timerBtn.classList.toggle('border-teal-600', isTimerActive);
        timerBtn.classList.toggle('bg-transparent', !isTimerActive);
        timerBtn.classList.toggle('border-white/50', !isTimerActive);
    }

    // المنطق الرئيسي لبدء التصوير (إما فوري أو بعد مؤقت)
    function startCaptureSequence() {
      if (!videoFeed.srcObject) return;

      if (isTimerActive) {
        startCountdown(5); // بدء العد التنازلي من 5 ثوان
      } else {
        takePhoto(); // التقاط فوري
      }
    }

    // دالة العد التنازلي
    function startCountdown(seconds) {
      let count = seconds;
      
      // إخفاء أدوات التحكم وتعطيل زر التصوير أثناء العد
      captureBtn.disabled = true;
      timerBtn.disabled = true;
      imagePreviewBtn.style.opacity = '0.5';
      captureBtn.style.opacity = '0.5';
      
      countdownText.textContent = count;
      countdownOverlay.classList.remove('hidden');

      countdownInterval = setInterval(() => {
        count -= 1;
        countdownText.textContent = count;
        countdownText.classList.remove('animate-pulse'); // إعادة تفعيل الأنماط
        void countdownText.offsetWidth; // إعادة تشغيل الحركة
        countdownText.classList.add('animate-pulse');

        if (count <= 0) {
          clearInterval(countdownInterval);
          countdownInterval = null;
          countdownOverlay.classList.add('hidden');
          
          // إعادة تفعيل الأزرار والتقاط الصورة
          captureBtn.disabled = false;
          timerBtn.disabled = false;
          imagePreviewBtn.style.opacity = '1';
          captureBtn.style.opacity = '1';
          takePhoto(); 
        }
      }, 1000);
    }

    function takePhoto() {
      if (!videoFeed.srcObject) return;
      
      // إظهار وميض شاشة التصوير
      const flash = document.createElement('div');
      flash.className = 'absolute inset-0 bg-white z-40 opacity-0 transition-opacity duration-75';
      document.getElementById('camera-container').appendChild(flash);
      
      setTimeout(() => { flash.style.opacity = '0.9'; }, 10);
      setTimeout(() => { 
        flash.style.opacity = '0'; 
        setTimeout(() => flash.remove(), 100);
      }, 100);


      photoCanvas.width = videoFeed.videoWidth;
      photoCanvas.height = videoFeed.videoHeight;
      const ctx = photoCanvas.getContext('2d');
      // عكس الصورة
      ctx.translate(photoCanvas.width, 0);
      ctx.scale(-1, 1);
      ctx.drawImage(videoFeed, 0, 0, photoCanvas.width, photoCanvas.height);
      
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

    // بدء تشغيل التطبيق
    startCamera();
  </script>
</body>
</html>
