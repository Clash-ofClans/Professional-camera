<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>كاميرا فورية (Direct Camera)</title>
    <!-- تحميل مكتبة Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* تخصيص الخط الداكن والخلفية والتنظيم العام */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #000000;
            color: #ffffff;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            overflow: hidden;
        }

        /* حاوية الكاميرا الرئيسية - لمحاكاة شاشة الهاتف */
        #camera-container {
            width: 95%; 
            max-width: 450px; 
            height: 90vh; 
            max-height: 900px;
            background-color: #111;
            border-radius: 2rem; 
            box-shadow: 0 0 0 8px #222, 0 15px 40px rgba(0, 0, 0, 0.9);
            position: relative;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            transition: all 0.3s; 
        }

        /* شاشة عرض الفيديو - تطبيق التحويلات عليها */
        #video-feed {
            width: 100%;
            flex-grow: 1; 
            object-fit: cover; 
            /* تطبيق التحويل الأولي: قلب الصورة للكاميرا الأمامية وتطبيق الزوم الأساسي */
            transform: scaleX(-1) scale(1.0); 
            transition: filter 0.3s ease-out, transform 0.3s ease-out; 
            display: none; 
            transform-origin: center center; 
        }
        
        /* تأثير زر الالتقاط (شبيه بأبل) */
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

        /* رسالة الخطأ أو التحميل */
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
        
        /* رسالة التحذير (بدلاً من alert) */
        .alert-message {
            animation: fadeInOut 3s ease-in-out forwards;
        }

        @keyframes fadeInOut {
            0% { opacity: 0; transform: translateY(-20px); }
            10% { opacity: 1; transform: translateY(0); }
            90% { opacity: 1; transform: translateY(0); }
            100% { opacity: 0; transform: translateY(-20px); }
        }
        
        /* شريط الإعدادات الاحترافية الجانبي */
        #settings-drawer {
            position: absolute;
            top: 0;
            left: 0;
            width: 280px; 
            max-width: 90%;
            height: 100%;
            background-color: rgba(30, 30, 30, 0.98);
            backdrop-filter: blur(12px);
            transform: translateX(-100%);
            transition: transform 0.3s ease-in-out;
            padding: 1rem;
            z-index: 60;
            border-right: 1px solid #444;
            box-shadow: 5px 0 15px rgba(0, 0, 0, 0.5);
            overflow-y: auto; 
        }
        #settings-drawer.open {
            transform: translateX(0);
        }

        /* شريط التحكم السفلي المدمج الجديد */
        #control-bar {
            background-color: rgba(0, 0, 0, 0.7); 
            backdrop-filter: blur(5px);
            padding: 1rem;
            position: relative;
            z-index: 10;
        }
    </style>
    <!-- أيقونات Lucide -->
    <script src="https://unpkg.com/lucide@latest"></script>
</head>
<body class="antialiased">

    <div id="camera-container" class="relative">

        <!-- رسالة الخطأ والتحميل -->
        <div id="status-overlay">
            <svg class="animate-spin h-10 w-10 text-yellow-400 mb-4" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <p id="status-text">جاري محاولة الوصول للكاميرا...</p>
        </div>
        
        <!-- أيقونات الحالة العلوية (شريط حالة الهاتف) -->
        <div class="absolute top-0 left-0 right-0 z-10 p-4 flex justify-between items-center text-xs font-semibold bg-gradient-to-b from-black/80 to-transparent">
            <span id="current-time">10:00</span>
            <div class="flex items-center space-x-1 space-x-reverse">
                <span class="text-green-500">5G</span>
                <i data-lucide="battery-charging" class="w-4 h-4"></i>
            </div>
        </div>

        <!-- 1. شاشة عرض الفيديو الحية -->
        <video id="video-feed" autoplay playsinline></video>

        <!-- 2. واجهة التحكم السفلية المدمجة -->
        <div id="control-bar" class="flex flex-col items-center">
            
            <!-- تم إزالة شريط أوضاع الكاميرا (Normal/Portrait) للتبسيط -->
            
            <!-- صف الأزرار الرئيسية (Gallery - Shutter - Pro Settings) -->
            <div class="flex justify-between items-center w-full px-8 pt-4"> 
                
                <!-- 1. زر المعرض/الصور الملتقطة (الزر الأيسر) -->
                <div onclick="showCapturedImage()" class="w-10 h-10 bg-gray-600 rounded-full border-2 border-white/50 flex items-center justify-center text-sm font-bold cursor-pointer transition hover:scale-105">
                    <i data-lucide="image" class="w-5 h-5 text-white"></i>
                </div>

                <!-- 2. زر الالتقاط الرئيسي (الوسط) -->
                <button id="capture-btn" onclick="takePhoto()" class="rounded-full focus:outline-none"></button>
                
                <!-- 3. زر الإعدادات الاحترافية (Pro Settings) (الزر الأيمن) -->
                <button onclick="toggleSettingsPanel()" class="p-3 rounded-full text-white/80 hover:text-white transition duration-200">
                    <i data-lucide="settings" class="w-6 h-6"></i>
                </button>
            </div>
        </div>
        
        <!-- 3. شاشة عرض الصورة الملتقطة (مخفية) -->
        <div id="preview-screen" class="absolute inset-0 bg-black z-30 hidden p-4 flex-col justify-between">
            <h3 class="text-center text-xl font-bold mb-4">الصورة الملتقطة</h3>
            <img id="captured-image" class="w-full h-auto max-h-[80%] object-contain rounded-xl shadow-lg mb-6 mx-auto flex-grow" alt="الصورة الملتقطة">
            <div class="flex justify-center space-x-4 space-x-reverse">
                <button onclick="hideCapturedImage()" class="bg-teal-600 hover:bg-teal-500 text-white font-bold py-2 px-6 rounded-lg transition duration-300">
                    العودة للكاميرا
                </button>
                <!-- حذف الصورة يتم من الذاكرة المحلية فقط الآن -->
                <button onclick="deletePhotoFromPreview()" class="bg-red-600 hover:bg-red-500 text-white font-bold py-2 px-6 rounded-lg transition duration-300">
                    حذف الصورة
                </button>
            </div>
        </div>

        <!-- 4. لوحة الإعدادات الاحترافية (Pro Settings Drawer) -->
        <div id="settings-drawer" class="">
            <div class="flex justify-between items-center mb-6">
                 <h4 class="text-lg font-bold text-yellow-400">إعدادات التصوير المتقدمة (Pro Max)</h4>
                 <button onclick="toggleSettingsPanel()" class="text-white hover:text-gray-400 transition"><i data-lucide="x" class="w-6 h-6"></i></button>
            </div>
            
            <hr class="border-gray-700 my-4">

            <!-- ============================== -->
            <!-- 1. قِسم الإضاءة (Light Control) -->
            <!-- ============================== -->
            <h5 class="text-sm font-semibold text-green-400 mb-3 flex items-center">
                <i data-lucide="sun" class="w-4 h-4 ml-2"></i>
                التحكم بالإضاءة (Exposure/ISO)
            </h5>

            <!-- التحكم بالتعريض (Exposure) -->
            <div class="mb-6">
                <label for="exposure" class="block text-sm font-medium mb-2">التعريض (EV) <span id="exposure-value" class="text-yellow-400">0.0</span></label>
                <input type="range" id="exposure" min="-5" max="5" step="0.5" value="0" oninput="updateFilter('exposure', this.value)" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer range-lg">
            </div>
            
            <!-- محاكاة ISO / التشويش الرقمي -->
            <div class="mb-6">
                <label for="iso" class="block text-sm font-medium mb-2">محاكاة ISO / التشويش الليلي <span id="iso-value" class="text-yellow-400">0%</span></label>
                <input type="range" id="iso" min="0" max="100" step="10" value="0" oninput="updateFilter('iso', this.value)" class="w-full h-2 bg-gray-700 range-lg">
            </div>
            
            <hr class="border-gray-700 my-4">

            <!-- ============================== -->
            <!-- 2. قِسم الألوان والتباين (Color & Contrast) -->
            <!-- ============================== -->
            <h5 class="text-sm font-semibold text-yellow-400 mb-3 flex items-center">
                <i data-lucide="palette" class="w-4 h-4 ml-2"></i>
                تخصيص الألوان والتباين
            </h5>

            <!-- التحكم بدرجة حرارة اللون (Color Temp) - White Balance -->
             <div class="mb-6">
                <label for="colorTemp" class="block text-sm font-medium mb-2">درجة حرارة اللون (Warmth) <span id="colorTemp-value" class="text-yellow-400">0%</span></label>
                <input type="range" id="colorTemp" min="0" max="100" step="5" value="0" oninput="updateFilter('colorTemp', this.value)" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer range-lg">
            </div>
            
            <!-- إزاحة اللون (Hue Rotation) -->
            <div class="mb-6">
                <label for="hue" class="block text-sm font-medium mb-2">إزاحة اللون (Hue Shift) <span id="hue-value" class="text-yellow-400">$0^{\circ}$</span></label>
                <input type="range" id="hue" min="0" max="360" step="10" value="0" oninput="updateFilter('hue', this.value)" class="w-full h-2 bg-gray-700 range-lg">
            </div>

            <!-- التحكم بالتشبع (Saturation) -->
             <div class="mb-6">
                <label for="saturation" class="block text-sm font-medium mb-2">التشبع (Saturation) <span id="saturation-value" class="text-yellow-400">100%</span></label>
                <input type="range" id="saturation" min="0" max="200" step="5" value="100" oninput="updateFilter('saturation', this.value)" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer range-lg">
            </div>
            
            <!-- التحكم بالتباين (Contrast) -->
             <div class="mb-6">
                <label for="contrast" class="block text-sm font-medium mb-2">التباين (Contrast) <span id="contrast-value" class="text-yellow-400">100%</span></label>
                <input type="range" id="contrast" min="50" max="200" step="5" value="100" oninput="updateFilter('contrast', this.value)" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer range-lg">
            </div>

            <hr class="border-gray-700 my-4">

            <!-- ============================== -->
            <!-- 3. قِسم الأوضاع الخاصة (Special Modes) -->
            <!-- ============================== -->
            <h5 class="text-sm font-semibold text-blue-400 mb-3 flex items-center">
                 <i data-lucide="telescope" class="w-4 h-4 ml-2"></i>
                إعدادات التصوير المتخصص
            </h5>

            <!-- وضع تصوير القمر (Moon Mode) -->
            <div class="mb-6 flex justify-between items-center p-3 bg-gray-900 rounded-lg">
                <label class="block text-sm font-medium">
                    <i data-lucide="moon" class="w-5 h-5 inline-block align-middle ml-2 text-blue-400"></i>
                    وضع القمر (Night/Moon Mode)
                </label>
                <button id="moon-mode-toggle" onclick="toggleMoonMode()" class="px-4 py-1 bg-gray-700 rounded-full text-xs transition duration-200">
                    إيقاف
                </button>
            </div>
            
            <!-- التحكم بالزوم (Zoom) -->
            <div class="mb-6">
                <label for="zoom" class="block text-sm font-medium mb-2">الزوم (Zoom) <span id="zoom-value" class="text-yellow-400">1.0X</span></label>
                <input type="range" id="zoom" min="100" max="2000" step="10" value="100" oninput="updateFilter('zoom', this.value)" class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer range-lg">
            </div>
            
        </div>

    </div>

    <!-- عنصر Canvas مخفي لالتقاط الإطار -->
    <canvas id="photo-canvas" style="display: none;"></canvas>

    <!-- JavaScript لتطبيق الكاميرا (تم حذف Firebase بالكامل) -->
    <script>
        // دمج أيقونات Lucide
        lucide.createIcons();

        const videoFeed = document.getElementById('video-feed');
        const photoCanvas = document.getElementById('photo-canvas');
        const statusOverlay = document.getElementById('status-overlay');
        const statusText = document.getElementById('status-text');
        const previewScreen = document.getElementById('preview-screen');
        const settingsDrawer = document.getElementById('settings-drawer');
        const moonModeToggle = document.getElementById('moon-mode-toggle');
        const capturedImageDisplay = document.getElementById('captured-image');

        let isMoonMode = false; 
        let lastCapturedImageData = null; // لتخزين آخر صورة ملتقطة محلياً

        // حالة الفلاتر الاحترافية
        let currentFilters = {
            exposure: 0, 
            iso: 0, 
            contrast: 100,
            saturation: 100,
            hue: 0, 
            colorTemp: 0, 
            zoom: 100, 
        };

        // --- وظيفة تحديث الوقت ---
        function updateTime() {
            const now = new Date();
            // تنسيق الوقت للعرض (مثال: 08:15 PM)
            document.getElementById('current-time').textContent = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', hour12: true });
        }
        setInterval(updateTime, 60000); 

        // --- وظيفة عرض رسالة للمستخدم (بدلاً من alert) ---
        window.alertUser = function(message) {
             const messageBox = document.createElement('div');
             messageBox.className = 'alert-message fixed top-4 right-4 bg-yellow-600 text-white p-4 rounded-lg shadow-xl z-50 transition transform duration-300';
             messageBox.textContent = message;
             document.body.appendChild(messageBox);
             setTimeout(() => {
                 messageBox.remove();
             }, 3000);
        };

        // --- وظيفة تطبيق الفلاتر البسيطة والمتقدمة ---
        function applyFilters() {
            let combinedFilter = '';
            const zoomScale = currentFilters.zoom / 100; 
            
            if (isMoonMode) {
                // وضع القمر: تباين عالٍ جداً، سطوع منخفض، أسود وأبيض تقريباً
                combinedFilter = ' contrast(400%) brightness(30%) grayscale(10%)'; 
            } else {
                
                // تطبيق الإعدادات الاحترافية للتصوير الطبيعي
                const brightnessFactor = (currentFilters.exposure / 5 * 0.5) + 1; 
                combinedFilter += ` brightness(${brightnessFactor * 100}%)`;
                
                // ISO Simulation (محاكاة التشويش)
                const isoNoise = currentFilters.iso / 100 * 0.4; 
                combinedFilter += ` grayscale(${isoNoise}%) `; 
                if (currentFilters.iso > 50) {
                     // تطبيق ضبابية بسيطة لزيادة الإحساس بالتشويش في الإضاءة المنخفضة
                     combinedFilter += ` blur(${isoNoise * 1.5}px) `; // تخفيف الضبابية قليلاً
                }

                combinedFilter += ` contrast(${currentFilters.contrast}%)`;
                combinedFilter += ` saturation(${currentFilters.saturation}%)`;
                
                // تطبيق درجة حرارة اللون (Warmth) وإزاحة اللون (Hue)
                combinedFilter += ` sepia(${currentFilters.colorTemp}%) hue-rotate(${currentFilters.hue}deg)`;
            }

            videoFeed.style.filter = combinedFilter.trim();
            // تطبيق الزوم ودمجه مع قلب الصورة
            videoFeed.style.transform = `scaleX(-1) scale(${zoomScale})`;
        }

        // --- وظيفة التحكم بإعدادات الـ Pro Max ---
        window.updateFilter = function(key, value) {
            currentFilters[key] = parseFloat(value);
            
            // تحديث قيم العرض
            if (key === 'exposure') {
                document.getElementById('exposure-value').textContent = parseFloat(value).toFixed(1);
            } else if (key === 'zoom') {
                document.getElementById('zoom-value').textContent = `${(parseFloat(value) / 100).toFixed(1)}X`;
            } else if (key === 'colorTemp') { 
                 document.getElementById('colorTemp-value').textContent = `${value}%`;
            } else if (key === 'iso') { 
                 document.getElementById('iso-value').textContent = `${value}%`;
            } else if (key === 'hue') { 
                 document.getElementById('hue-value').textContent = `${value}°`;
            } else {
                document.getElementById(`${key}-value`).textContent = `${value}%`;
            }

            // تطبيق الفلتر
            applyFilters();
        }
        
        // --- وظيفة إظهار/إخفاء لوحة الإعدادات الاحترافية ---
        window.toggleSettingsPanel = function() {
            settingsDrawer.classList.toggle('open');
            lucide.createIcons();
        }

        // --- وظيفة تبديل وضع القمر ---
        window.toggleMoonMode = function() {
            isMoonMode = !isMoonMode;

            if (isMoonMode) {
                moonModeToggle.textContent = 'تشغيل';
                moonModeToggle.classList.remove('bg-gray-700');
                moonModeToggle.classList.add('bg-blue-600', 'text-white');
            } else {
                moonModeToggle.textContent = 'إيقاف';
                moonModeToggle.classList.remove('bg-blue-600', 'text-white');
                moonModeToggle.classList.add('bg-gray-700');
            }
            applyFilters();
        }


        // --- وظيفة بدء تشغيل الكاميرا (مع معالجة الأخطاء المحسّنة) ---
        async function startCamera() {
            updateTime();
            try {
                // التحقق من دعم المتصفح
                if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                    statusText.innerHTML = `
                        <i data-lucide="alert-triangle" class="w-8 h-8 text-red-500 mb-4 mx-auto"></i>
                        <p class="font-bold mb-2">خطأ فادح:</p>
                        هذا المتصفح لا يدعم الوصول إلى الكاميرا.
                    `;
                    lucide.createIcons();
                    return;
                }

                statusText.textContent = "جاري فتح الكاميرا...";
                // طلب الوصول إلى الكاميرا الأمامية ('user')
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
                
                videoFeed.srcObject = stream;
                
                videoFeed.onloadedmetadata = () => {
                    // إخفاء التحميل وإظهار الفيديو فوراً
                    statusOverlay.classList.add('hidden');
                    videoFeed.style.display = 'block'; 
                    applyFilters(); 
                };

            } catch (error) {
                console.error("Error accessing camera:", error);
                
                let errorMessage = "حدث خطأ غير معروف أثناء محاولة الوصول للكاميرا.";

                // معالجة أخطاء الرفض (السبب الأكثر شيوعاً)
                if (error.name === 'NotAllowedError' || error.name === 'SecurityError') {
                    errorMessage = `
                        <i data-lucide="camera-off" class="w-8 h-8 text-red-500 mb-4 mx-auto"></i>
                        <p class="font-bold mb-2 text-yellow-400">**تم رفض الوصول للكاميرا**</p>
                        لفتح الكاميرا، يجب عليك **السماح** بالوصول عبر النافذة المنبثقة التي تظهر في متصفحك.
                        <br/><span class="text-sm text-gray-400"> (عادةً ما تكون في الزاوية العلوية).</span>
                    `;
                } else if (error.name === 'NotFoundError') {
                    errorMessage = `
                        <i data-lucide="scan-face" class="w-8 h-8 text-red-500 mb-4 mx-auto"></i>
                        <p class="font-bold mb-2">لم يتم العثور على كاميرا</p>
                        يرجى التأكد من أن جهازك يحتوي على كاميرا.
                    `;
                } else {
                     errorMessage = `
                        <i data-lucide="bug" class="w-8 h-8 text-red-500 mb-4 mx-auto"></i>
                        <p class="font-bold mb-2">خطأ غير متوقع:</p>
                        ${error.name}: ${error.message}
                    `;
                }

                statusText.innerHTML = errorMessage;
                lucide.createIcons(); 
            }
        }
        window.startCamera = startCamera; 
        
        // --- وظيفة التقاط الصورة (Snap) ---
        window.takePhoto = function() {
            if (!videoFeed.srcObject) {
                alertUser("الكاميرا ليست جاهزة بعد.");
                return;
            }

            // 1. إعداد الـ Canvas والـ Context
            const zoomScale = currentFilters.zoom / 100;
            
            photoCanvas.width = videoFeed.videoWidth;
            photoCanvas.height = videoFeed.videoHeight;
            const context = photoCanvas.getContext('2d');
            
            // 2. تطبيق فلاتر اللون والتباين والتعريض على الـ Canvas
            let filterToApply = videoFeed.style.filter.replace(/blur\(\d+px\)/g, '').trim();
            context.filter = filterToApply; 
            
            // لحساب منطقة الرسم بعد الزوم (عن طريق اقتصاص الفيديو)
            const scaledWidth = photoCanvas.width / zoomScale;
            const scaledHeight = photoCanvas.height / zoomScale;
            const offsetX = (photoCanvas.width - scaledWidth) / 2;
            const offsetY = (photoCanvas.height - scaledHeight) / 2;

            // 3. تطبيق "القلب" (Flip) للتعويض عن عكس الكاميرا الأمامية
            context.translate(photoCanvas.width, 0);
            context.scale(-1, 1);
            
            // 4. رسم الإطار مع تطبيق الزوم الرقمي
            context.drawImage(
                videoFeed, 
                offsetX, 
                offsetY, 
                scaledWidth, 
                scaledHeight,
                0, 
                0, 
                photoCanvas.width, 
                photoCanvas.height
            );

            // 5. إعادة تعيين التحويلات لإعادة استخدام الـ context
            context.setTransform(1, 0, 0, 1, 0, 0);
            context.filter = 'none'; 

            // 6. تحويل الصورة إلى Base64 بجودة عالية
            const dataUrl = photoCanvas.toDataURL('image/jpeg', 1.0); 
            
            // 7. حفظ البيانات محلياً وعرضها (بدون Firestore)
            lastCapturedImageData = dataUrl;
            capturedImageDisplay.src = dataUrl;
            alertUser("تم التقاط الصورة بنجاح!");
            showCapturedImage();


            // 8. تأثير بصري
            const captureBtn = document.getElementById('capture-btn');
            captureBtn.style.backgroundColor = '#fca5a5';
            setTimeout(() => { captureBtn.style.backgroundColor = 'white'; }, 100);
        }

        // --- عرض شاشة المعاينة ---
        window.showCapturedImage = function() {
             if (lastCapturedImageData) {
                previewScreen.classList.remove('hidden');
                previewScreen.classList.add('flex');
            } else {
                alertUser('لم تلتقط أي صورة بعد!');
            }
        }
        
        // --- إخفاء شاشة المعاينة ---
        window.hideCapturedImage = function() {
            previewScreen.classList.add('hidden');
            previewScreen.classList.remove('flex');
        }

        // --- حذف الصورة من الذاكرة المحلية (الهدف الشخصي) ---
        window.deletePhotoFromPreview = function() {
            lastCapturedImageData = null;
            capturedImageDisplay.src = "https://placehold.co/400x300/1e293b/94a3b8?text=لا+صور";
            alertUser("تم مسح الصورة من الذاكرة المحلية.");
            window.hideCapturedImage();
        };


        // بدء تشغيل الكاميرا فوراً
        window.startCamera();
    </script>
</body>
</html>
