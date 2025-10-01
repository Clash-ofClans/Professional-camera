<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>كاميرا احترافية</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        /* إعدادات عامة */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #000;
            color: #fff;
            overflow: hidden;
            position: relative;
        }

        /* تطبيق الكاميرا */
        .camera-app {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        #video {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transform: scaleX(-1);
            transition: filter 0.3s ease, transform 0.2s ease;
        }

        #canvas {
            display: none;
        }

        /* شبكة الخطوط */
        .gridlines {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: none;
            pointer-events: none;
            opacity: 0.3;
            background-image: 
                linear-gradient(rgba(255,255,255,0.2) 1px, transparent 1px),
                linear-gradient(90deg, rgba(255,255,255,0.2) 1px, transparent 1px);
            background-size: 33.33% 33.33%;
        }

        body.show-grid .gridlines {
            display: block;
        }

        /* الأشرطة والتحكمات */
        .top-bar, .bottom-bar {
            position: absolute;
            left: 0;
            right: 0;
            display: flex;
            align-items: center;
            padding: 15px 20px;
            background: linear-gradient(to bottom, rgba(0,0,0,0.7), rgba(0,0,0,0.3));
            backdrop-filter: blur(15px);
            z-index: 10;
        }

        .top-bar {
            top: 0;
            justify-content: space-between;
        }

        .bottom-bar {
            bottom: 0;
            background: linear-gradient(to top, rgba(0,0,0,0.7), rgba(0,0,0,0.3));
        }

        /* مجموعات الأزرار */
        .top-bar-left, .top-bar-right {
            display: flex;
            gap: 20px;
        }
        
        .bottom-bar-left, .bottom-bar-center, .bottom-bar-right {
            display: flex;
            align-items: center;
            gap: 20px;
        }

        .bottom-bar-center {
            flex-grow: 1;
            justify-content: center;
        }

        /* الأزرار */
        .icon-btn {
            background: rgba(255, 255, 255, 0.1);
            border: none;
            color: #fff;
            font-size: 1.5rem;
            cursor: pointer;
            padding: 12px;
            transition: all 0.3s ease;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .icon-btn:hover {
            background: rgba(255, 255, 255, 0.2);
            transform: scale(1.05);
        }

        .icon-btn.active {
            background: rgba(255, 255, 255, 0.3);
            color: #f39c12;
        }

        .shutter-btn {
            width: 80px;
            height: 80px;
            background: linear-gradient(135deg, #ffffff, #e0e0e0);
            border: 5px solid #333;
            border-radius: 50%;
            cursor: pointer;
            transition: all 0.2s ease;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
        }

        .shutter-btn:hover {
            transform: scale(1.05);
        }

        .shutter-btn:active {
            background: linear-gradient(135deg, #e0e0e0, #c0c0c0);
            transform: scale(0.95);
        }

        /* محدد الأوضاع في الأسفل */
        .mode-selector {
            display: flex;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 25px;
            padding: 5px;
            backdrop-filter: blur(10px);
        }

        .mode-btn {
            background: none;
            border: none;
            color: #fff;
            padding: 10px 20px;
            cursor: pointer;
            border-radius: 20px;
            transition: all 0.3s ease;
            font-size: 0.9rem;
        }

        .mode-btn.active {
            background: linear-gradient(135deg, #f39c12, #e67e22);
            box-shadow: 0 2px 10px rgba(243, 156, 18, 0.4);
        }

        /* عناصر الشريط السفلي */
        .gallery-preview {
            width: 50px;
            height: 50px;
            border: 2px solid #fff;
            border-radius: 10px;
            overflow: hidden;
            cursor: pointer;
            transition: transform 0.3s ease;
        }

        .gallery-preview:hover {
            transform: scale(1.05);
        }

        .gallery-preview img {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        .zoom-control {
            display: flex;
            align-items: center;
            gap: 10px;
            background: rgba(255, 255, 255, 0.1);
            padding: 8px 15px;
            border-radius: 25px;
            backdrop-filter: blur(10px);
        }

        #zoomSlider {
            width: 120px;
            -webkit-appearance: none;
            appearance: none;
            height: 5px;
            background: rgba(255, 255, 255, 0.3);
            border-radius: 5px;
            outline: none;
        }

        #zoomSlider::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 15px;
            height: 15px;
            background: #fff;
            border-radius: 50%;
            cursor: pointer;
        }

        /* لوحة الإعدادات المتقدمة */
        .advanced-settings {
            position: absolute;
            right: -320px;
            top: 0;
            width: 320px;
            height: 100%;
            background: linear-gradient(to left, rgba(0, 0, 0, 0.9), rgba(0, 0, 0, 0.8));
            backdrop-filter: blur(20px);
            padding: 20px;
            transition: right 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            z-index: 20;
            overflow-y: auto;
            box-shadow: -5px 0 20px rgba(0, 0, 0, 0.5);
        }

        .advanced-settings.open {
            right: 0;
        }

        .advanced-settings h3 {
            text-align: center;
            margin-bottom: 20px;
            font-size: 1.3rem;
            color: #f39c12;
        }

        .setting-group {
            margin-bottom: 20px;
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        .setting-group label {
            font-size: 0.9rem;
            color: #aaa;
        }

        .setting-group input[type="range"] {
            width: 100%;
            -webkit-appearance: none;
            appearance: none;
            height: 6px;
            background: rgba(255, 255, 255, 0.2);
            border-radius: 5px;
            outline: none;
        }

        .setting-group input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 18px;
            height: 18px;
            background: #f39c12;
            border-radius: 50%;
            cursor: pointer;
        }

        .setting-group select {
            padding: 10px;
            background: #333;
            color: #fff;
            border: 1px solid #555;
            border-radius: 4px;
        }

        .filter-selector {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
        }

        .filter-btn {
            padding: 12px;
            background: rgba(255, 255, 255, 0.1);
            color: #fff;
            border: none;
            border-radius: 20px;
            cursor: pointer;
            transition: all 0.3s ease;
            font-size: 0.9rem;
        }

        .filter-btn:hover {
            background: rgba(255, 255, 255, 0.2);
        }

        .filter-btn.active {
            background: linear-gradient(135deg, #3498db, #2980b9);
            box-shadow: 0 2px 10px rgba(52, 152, 219, 0.4);
        }

        .mode-presets {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-top: 20px;
        }

        .mode-presets button {
            padding: 15px;
            background: linear-gradient(135deg, #3498db, #2980b9);
            color: #fff;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s ease;
            font-size: 1rem;
            font-weight: 500;
        }

        .mode-presets button:hover {
            background: linear-gradient(135deg, #2980b9, #21618c);
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(52, 152, 219, 0.3);
        }

        /* مؤشر التركيز */
        .focus-indicator {
            position: absolute;
            width: 80px;
            height: 80px;
            border: 2px solid rgba(255, 255, 255, 0.8);
            border-radius: 50%;
            pointer-events: none;
            opacity: 0;
            transform: translate(-50%, -50%) scale(0.5);
            transition: opacity 0.3s, transform 0.3s;
        }

        .focus-indicator.active {
            opacity: 1;
            transform: translate(-50%, -50%) scale(1);
        }

        .focus-indicator::before,
        .focus-indicator::after {
            content: '';
            position: absolute;
            background: rgba(255, 255, 255, 0.8);
        }

        .focus-indicator::before {
            width: 20px;
            height: 2px;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }

        .focus-indicator::after {
            width: 2px;
            height: 20px;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }

        /* تأثير الفلاش */
        .flash-effect {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.8);
            opacity: 0;
            pointer-events: none;
            z-index: 100;
        }

        .flash-effect.active {
            animation: flash 0.3s ease-out;
        }

        @keyframes flash {
            0% { opacity: 0; }
            50% { opacity: 0.8; }
            100% { opacity: 0; }
        }

        /* مؤشر التسجيل */
        .recording-indicator {
            position: absolute;
            top: 80px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 15px;
            background: rgba(255, 0, 0, 0.7);
            border-radius: 20px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .recording-indicator.active {
            opacity: 1;
        }

        .recording-dot {
            width: 12px;
            height: 12px;
            background: #fff;
            border-radius: 50%;
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }

        .recording-time {
            font-weight: 500;
        }

        /* عدسة التكبير */
        .zoom-lens {
            position: absolute;
            width: 150px;
            height: 150px;
            border: 3px solid #f39c12;
            border-radius: 50%;
            overflow: hidden;
            pointer-events: none;
            opacity: 0;
            transform: translate(-50%, -50%) scale(0.8);
            transition: opacity 0.3s, transform 0.3s;
            box-shadow: 0 0 20px rgba(243, 156, 18, 0.5);
        }

        .zoom-lens.active {
            opacity: 1;
            transform: translate(-50%, -50%) scale(1);
        }

        .zoom-lens-content {
            width: 100%;
            height: 100%;
            transform: scaleX(-1);
        }

        /* إشعارات */
        .notification {
            position: absolute;
            top: 100px;
            left: 50%;
            transform: translateX(-50%);
            padding: 12px 20px;
            background: rgba(0, 0, 0, 0.8);
            border-radius: 25px;
            font-size: 0.9rem;
            opacity: 0;
            transition: opacity 0.3s, transform 0.3s;
            z-index: 100;
        }

        .notification.show {
            opacity: 1;
            transform: translateX(-50%) translateY(10px);
        }

        /* زر التبديل السريع للفلاتر */
        .filter-quick-toggle {
            position: absolute;
            right: 20px;
            top: 50%;
            transform: translateY(-50%);
            display: flex;
            flex-direction: column;
            gap: 15px;
            opacity: 0;
            transition: opacity 0.3s;
            z-index: 15;
        }

        .filter-quick-toggle.active {
            opacity: 1;
        }

        .filter-quick-btn {
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.1);
            border: none;
            color: #fff;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.2rem;
        }

        .filter-quick-btn:hover {
            background: rgba(255, 255, 255, 0.2);
            transform: scale(1.1);
        }

        .filter-quick-btn.active {
            background: rgba(243, 156, 18, 0.5);
        }

        /* شريط المعلومات */
        .info-bar {
            position: absolute;
            left: 20px;
            top: 80px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            font-size: 0.85rem;
            color: rgba(255, 255, 255, 0.7);
        }

        .info-item {
            display: flex;
            align-items: center;
            gap: 5px;
        }

        .info-item i {
            font-size: 1rem;
            color: #f39c12;
        }

        /* شريط الأدوات السريع */
        .quick-tools {
            position: absolute;
            left: 20px;
            bottom: 100px;
            display: flex;
            flex-direction: column;
            gap: 15px;
            opacity: 0;
            transition: opacity 0.3s;
            z-index: 15;
        }

        .quick-tools.active {
            opacity: 1;
        }

        .quick-tool-btn {
            width: 45px;
            height: 45px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.1);
            border: none;
            color: #fff;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.3rem;
        }

        .quick-tool-btn:hover {
            background: rgba(255, 255, 255, 0.2);
            transform: scale(1.1);
        }

        .quick-tool-btn.active {
            background: rgba(243, 156, 18, 0.5);
        }

        /* محدد الألوان */
        .color-picker {
            position: absolute;
            bottom: 100px;
            right: 20px;
            display: flex;
            gap: 10px;
            opacity: 0;
            transition: opacity 0.3s;
            z-index: 15;
        }

        .color-picker.active {
            opacity: 1;
        }

        .color-option {
            width: 35px;
            height: 35px;
            border-radius: 50%;
            border: 2px solid rgba(255, 255, 255, 0.3);
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .color-option:hover {
            transform: scale(1.1);
            border-color: rgba(255, 255, 255, 0.8);
        }

        .color-option.active {
            border-color: #fff;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
        }

        /* تأثيرات خاصة */
        .special-effects {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 5;
        }

        .particles {
            position: absolute;
            width: 100%;
            height: 100%;
            overflow: hidden;
        }

        .particle {
            position: absolute;
            width: 4px;
            height: 4px;
            background: rgba(255, 255, 255, 0.8);
            border-radius: 50%;
            pointer-events: none;
        }

        /* مؤشرات الأوضاع */
        .mode-indicator {
            position: absolute;
            top: 80px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 15px;
            border-radius: 20px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .mode-indicator.active {
            opacity: 1;
        }

        .mode-indicator i {
            font-size: 1.2rem;
        }

        .mode-indicator span {
            font-weight: 500;
        }

        /* مؤشر البطارية */
        .battery-indicator {
            display: flex;
            align-items: center;
            gap: 5px;
            font-size: 0.9rem;
        }

        .battery-icon {
            font-size: 1.2rem;
        }

        .battery-level {
            font-weight: 500;
        }

        /* مؤشر جودة الصورة */
        .quality-indicator {
            display: flex;
            align-items: center;
            gap: 5px;
            font-size: 0.9rem;
        }

        .quality-dots {
            display: flex;
            gap: 3px;
        }

        .quality-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.3);
        }

        .quality-dot.active {
            background: #f39c12;
        }

        /* تأثيرات الوضع الليلي */
        .night-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 50, 0.2);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .night-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع البورتريه */
        .portrait-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 40%, rgba(0, 0, 0, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .portrait-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الطبيعة */
        .nature-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 100, 0, 0.1), rgba(0, 50, 0, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .nature-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الطعام */
        .food-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 200, 0, 0.1), rgba(255, 150, 0, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .food-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الماكرو */
        .macro-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.1) 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .macro-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الأفلام */
        .cinema-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 0, 0, 0.2), rgba(0, 0, 0, 0.4));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .cinema-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الوثائق */
        .document-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(200, 200, 200, 0.1), rgba(150, 150, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .document-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الرياضة */
        .sports-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 0, 0, 0.1), rgba(150, 0, 0, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .sports-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع المعمار */
        .architecture-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 0, 255, 0.1), rgba(0, 0, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .architecture-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الفن */
        .art-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 0, 255, 0.1), rgba(150, 0, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .art-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الموسيقى */
        .music-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 255, 255, 0.1), rgba(0, 150, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .music-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الموضة */
        .fashion-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 0, 150, 0.1), rgba(150, 0, 100, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .fashion-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع السيارات */
        .cars-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(100, 100, 100, 0.1), rgba(50, 50, 50, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .cars-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الطيران */
        .aerial-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 150, 255, 0.1), rgba(0, 100, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .aerial-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الماء */
        .water-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 200, 255, 0.1), rgba(0, 150, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .water-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع النباتات */
        .plants-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 255, 0, 0.1), rgba(0, 200, 0, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .plants-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الحشرات */
        .insects-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 20%, rgba(255, 255, 0, 0.1) 50%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .insects-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الطيور */
        .birds-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(135, 206, 235, 0.1), rgba(70, 130, 180, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .birds-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع السماء */
        .sky-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(135, 206, 250, 0.1), rgba(70, 130, 180, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .sky-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الغيوم */
        .clouds-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 255, 255, 0.1), rgba(200, 200, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .clouds-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع المطر */
        .rain-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(100, 100, 150, 0.1), rgba(50, 50, 100, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .rain-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الضباب */
        .fog-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(200, 200, 200, 0.2), rgba(150, 150, 150, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .fog-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشمس */
        .sun-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 30% 30%, rgba(255, 255, 0, 0.2), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .sun-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع القمر */
        .moon-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 70% 30%, rgba(200, 200, 255, 0.2), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .moon-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع النجوم */
        .stars-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(0, 0, 50, 0.3) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .stars-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الألعاب النارية */
        .fireworks-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 40%, rgba(255, 100, 0, 0.2) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .fireworks-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الحفلات */
        .party-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(255, 0, 0, 0.1), rgba(0, 255, 0, 0.1), rgba(0, 0, 255, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .party-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الأطفال */
        .kids-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 200, 200, 0.1), rgba(255, 150, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .kids-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الحيوانات الأليفة */
        .pets-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(200, 150, 100, 0.1), rgba(150, 100, 50, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .pets-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الطعام المتقدم */
        .food-pro-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 200, 0, 0.2), rgba(255, 150, 0, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .food-pro-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الزهور */
        .flowers-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 100, 200, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .flowers-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشاطئ */
        .beach-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 255, 200, 0.1), rgba(200, 200, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .beach-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الثلج */
        .snow-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(230, 230, 255, 0.2), rgba(200, 200, 230, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .snow-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الغروب */
        .sunset-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 150, 0, 0.2), rgba(255, 100, 0, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .sunset-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخلفية */
        .backlight-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 40%, rgba(255, 255, 200, 0.3) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .backlight-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنخفضة */
        .low-light-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(50, 50, 100, 0.2), rgba(30, 30, 70, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .low-light-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع اللقطة البانورامية */
        .panorama-guide {
            position: absolute;
            top: 50%;
            left: 0;
            width: 100%;
            height: 60%;
            transform: translateY(-50%);
            display: none;
            pointer-events: none;
            z-index: 7;
        }

        .panorama-guide.active {
            display: block;
        }

        .panorama-line {
            position: absolute;
            top: 0;
            width: 2px;
            height: 100%;
            background: rgba(243, 156, 18, 0.8);
        }

        .panorama-line.left {
            left: 20%;
        }

        .panorama-line.right {
            right: 20%;
        }

        .panorama-arrow {
            position: absolute;
            top: 50%;
            transform: translateY(-50%);
            font-size: 2rem;
            color: rgba(243, 156, 18, 0.8);
        }

        .panorama-arrow.left {
            left: 10%;
        }

        .panorama-arrow.right {
            right: 10%;
        }

        /* تأثيرات الصورة المتحركة */
        .gif-indicator {
            position: absolute;
            top: 80px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 15px;
            background: rgba(155, 89, 182, 0.7);
            border-radius: 20px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .gif-indicator.active {
            opacity: 1;
        }

        .gif-icon {
            font-size: 1.2rem;
        }

        .gif-info {
            font-weight: 500;
        }

        /* تأثيرات الوقت البطيء */
        .time-lapse-indicator {
            position: absolute;
            top: 80px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 15px;
            background: rgba(52, 152, 219, 0.7);
            border-radius: 20px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .time-lapse-indicator.active {
            opacity: 1;
        }

        .time-lapse-icon {
            font-size: 1.2rem;
        }

        .time-lapse-info {
            font-weight: 500;
        }

        /* تأثيرات الحركة البطيئة */
        .slow-motion-effect {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.1);
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .slow-motion-effect.active {
            opacity: 1;
        }

        /* تأثيرات اللقطة المزدوجة */
        .dual-shot-preview {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 80%;
            height: 60%;
            display: none;
            pointer-events: none;
            z-index: 8;
        }

        .dual-shot-preview.active {
            display: flex;
            gap: 10px;
        }

        .dual-shot-frame {
            flex: 1;
            border: 2px solid rgba(255, 255, 255, 0.5);
            border-radius: 10px;
            overflow: hidden;
        }

        .dual-shot-frame img {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        /* تأثيرات الصورة البانورامية 360 */
        .panorama-360-indicator {
            position: absolute;
            top: 80px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 15px;
            background: rgba(46, 204, 113, 0.7);
            border-radius: 20px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .panorama-360-indicator.active {
            opacity: 1;
        }

        .panorama-360-icon {
            font-size: 1.2rem;
        }

        .panorama-360-info {
            font-weight: 500;
        }

        /* تأثيرات وضع العلامات المائية */
        .watermark-mode-indicator {
            position: absolute;
            top: 80px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 15px;
            background: rgba(41, 128, 185, 0.7);
            border-radius: 20px;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .watermark-mode-indicator.active {
            opacity: 1;
        }

        .watermark-mode-icon {
            font-size: 1.2rem;
        }

        .watermark-mode-info {
            font-weight: 500;
        }

        /* تأثيرات وضع قوس قزح */
        .rainbow-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, 
                rgba(255, 0, 0, 0.1), 
                rgba(255, 127, 0, 0.1), 
                rgba(255, 255, 0, 0.1), 
                rgba(0, 255, 0, 0.1), 
                rgba(0, 0, 255, 0.1), 
                rgba(75, 0, 130, 0.1), 
                rgba(148, 0, 211, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .rainbow-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع البرق */
        .lightning-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 255, 200, 0.1), rgba(255, 255, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .lightning-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشفق القطبي */
        .aurora-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 255, 100, 0.1), rgba(0, 200, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .aurora-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الثلج المتساقط */
        .falling-snow-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(230, 230, 255, 0.2), rgba(200, 200, 230, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .falling-snow-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الأوراق المتساقطة */
        .falling-leaves-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 200, 100, 0.1), rgba(200, 150, 50, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .falling-leaves-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الزهور المتفتحة */
        .blooming-flowers-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 100, 200, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .blooming-flowers-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع النجوم المتحركة */
        .moving-stars-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(0, 0, 50, 0.3) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .moving-stars-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الليلة المظلمة */
        .dark-night-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 0, 30, 0.3), rgba(0, 0, 20, 0.5));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dark-night-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع النهار المشرق */
        .bright-day-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 255, 200, 0.1), rgba(255, 255, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .bright-day-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الغروب الذهبي */
        .golden-sunset-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 200, 0, 0.2), rgba(255, 150, 0, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .golden-sunset-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الفجر البنفسجي */
        .purple-dawn-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(150, 100, 200, 0.2), rgba(100, 50, 150, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .purple-dawn-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع السماء الزرقاء */
        .blue-sky-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(135, 206, 250, 0.1), rgba(70, 130, 180, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .blue-sky-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الغيوم البيضاء */
        .white-clouds-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 255, 255, 0.1), rgba(230, 230, 230, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .white-clouds-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الغيوم الرمادية */
        .gray-clouds-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(200, 200, 200, 0.2), rgba(150, 150, 150, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .gray-clouds-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع المطر الخفيف */
        .light-rain-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(150, 150, 200, 0.1), rgba(100, 100, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .light-rain-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع المطر الغزير */
        .heavy-rain-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(100, 100, 150, 0.2), rgba(50, 50, 100, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .heavy-rain-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الضباب الكثيف */
        .thick-fog-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(200, 200, 200, 0.3), rgba(150, 150, 150, 0.4));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .thick-fog-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الضباب الخفيف */
        .light-fog-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(200, 200, 200, 0.1), rgba(150, 150, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .light-fog-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشمس المشرقة */
        .rising-sun-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 30% 70%, rgba(255, 200, 0, 0.2), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .rising-sun-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشمس الغاربة */
        .setting-sun-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 70% 30%, rgba(255, 150, 0, 0.2), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .setting-sun-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع القمر المكتمل */
        .full-moon-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 70% 30%, rgba(200, 200, 255, 0.2), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .full-moon-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الهلال */
        .crescent-moon-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 70% 30%, rgba(200, 200, 255, 0.1), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .crescent-moon-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع النجوم الساطعة */
        .bright-stars-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(0, 0, 50, 0.2) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .bright-stars-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع النجوم الخافتة */
        .dim-stars-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(0, 0, 50, 0.1) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dim-stars-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع مجرة درب التبانة */
        .milky-way-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 20%, rgba(100, 100, 200, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .milky-way-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشهب */
        .shooting-stars-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(0, 0, 50, 0.2) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .shooting-stars-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع المذنبات */
        .comets-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(0, 0, 50, 0.2) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .comets-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الكسوف الشمسي */
        .solar-eclipse-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 50% 50%, rgba(0, 0, 0, 0.5) 30%, transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .solar-eclipse-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الخسوف القمري */
        .lunar-eclipse-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 70% 30%, rgba(150, 50, 50, 0.2), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .lunar-eclipse-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشفق القطبي الشمالي */
        .northern-lights-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(0, 255, 100, 0.1), rgba(0, 200, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .northern-lights-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشفق القطبي الجنوبي */
        .southern-lights-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(100, 0, 255, 0.1), rgba(50, 0, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .southern-lights-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع قوس قزح المزدوج */
        .double-rainbow-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, 
                rgba(255, 0, 0, 0.1), 
                rgba(255, 127, 0, 0.1), 
                rgba(255, 255, 0, 0.1), 
                rgba(0, 255, 0, 0.1), 
                rgba(0, 0, 255, 0.1), 
                rgba(75, 0, 130, 0.1), 
                rgba(148, 0, 211, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .double-rainbow-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الهالة الشمسية */
        .sun-halo-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 50% 50%, transparent 20%, rgba(255, 255, 200, 0.2) 40%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .sun-halo-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الهالة القمرية */
        .moon-halo-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 70% 30%, transparent 20%, rgba(200, 200, 255, 0.2) 40%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .moon-halo-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الشفق الذهبي */
        .golden-hour-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 200, 100, 0.2), rgba(255, 150, 50, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .golden-hour-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الساعة الزرقاء */
        .blue-hour-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(50, 100, 200, 0.2), rgba(30, 70, 150, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .blue-hour-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الظلال الطويلة */
        .long-shadows-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(0, 0, 0, 0.1), rgba(0, 0, 0, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .long-shadows-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الجانبية */
        .side-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, rgba(0, 0, 0, 0.2), rgba(255, 255, 255, 0.1), rgba(0, 0, 0, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .side-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخلفية القوية */
        .strong-backlight-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 40%, rgba(255, 255, 200, 0.4) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .strong-backlight-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخلفية الضعيفة */
        .weak-backlight-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 40%, rgba(255, 255, 200, 0.2) 80%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .weak-backlight-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المباشرة */
        .direct-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .direct-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة غير المباشرة */
        .indirect-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 200, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .indirect-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الطبيعية */
        .natural-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 255, 200, 0.1), rgba(200, 200, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .natural-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الصناعية */
        .artificial-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 200, 100, 0.1), rgba(200, 150, 50, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .artificial-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الدافئة */
        .warm-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(255, 200, 100, 0.2), rgba(200, 150, 50, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .warm-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الباردة */
        .cool-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(100, 200, 255, 0.2), rgba(50, 150, 200, 0.3));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .cool-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المحايدة */
        .neutral-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(200, 200, 200, 0.1), rgba(150, 150, 150, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .neutral-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الملونة */
        .colored-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, 
                rgba(255, 0, 0, 0.1), 
                rgba(0, 255, 0, 0.1), 
                rgba(0, 0, 255, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .colored-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتغيرة */
        .changing-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, 
                rgba(255, 0, 0, 0.1), 
                rgba(255, 127, 0, 0.1), 
                rgba(255, 255, 0, 0.1), 
                rgba(0, 255, 0, 0.1), 
                rgba(0, 0, 255, 0.1), 
                rgba(75, 0, 130, 0.1), 
                rgba(148, 0, 211, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .changing-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الوامضة */
        .flashing-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.1);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .flashing-lighting-mode-overlay.active {
            opacity: 1;
            animation: flash 1s infinite;
        }

        /* تأثيرات وضع الإضاءة المتقطعة */
        .strobe-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.2);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .strobe-lighting-mode-overlay.active {
            opacity: 1;
            animation: flash 0.5s infinite;
        }

        /* تأثيرات وضع الإضاءة الخافتة */
        .dim-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.2);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dim-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الساطعة */
        .bright-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.2);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .bright-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المعتدلة */
        .moderate-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(200, 200, 200, 0.1);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .moderate-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة القوية */
        .strong-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.3);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .strong-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الضعيفة */
        .weak-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.1);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .weak-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتوسطة */
        .medium-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(150, 150, 150, 0.1);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .medium-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة العالية */
        .high-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.4);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .high-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنخفضة */
        .low-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.3);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .low-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المرتفعة */
        .elevated-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.5);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .elevated-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنخفضة جداً */
        .very-low-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.4);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .very-low-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المرتفعة جداً */
        .very-high-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.6);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .very-high-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة القصوى */
        .extreme-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.7);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .extreme-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الدنيا */
        .minimal-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .minimal-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المثلى */
        .optimal-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(200, 200, 200, 0.2);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .optimal-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المثالية */
        .perfect-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(220, 220, 220, 0.2);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .perfect-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الممتازة */
        .excellent-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(240, 240, 240, 0.2);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .excellent-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الرائعة */
        .amazing-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(250, 250, 250, 0.2);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .amazing-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخارقة */
        .superb-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.3);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .superb-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة السحرية */
        .magical-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, 
                rgba(255, 0, 0, 0.1), 
                rgba(255, 127, 0, 0.1), 
                rgba(255, 255, 0, 0.1), 
                rgba(0, 255, 0, 0.1), 
                rgba(0, 0, 255, 0.1), 
                rgba(75, 0, 130, 0.1), 
                rgba(148, 0, 211, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .magical-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخيالية */
        .fantasy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, 
                rgba(255, 0, 0, 0.2), 
                rgba(255, 127, 0, 0.2), 
                rgba(255, 255, 0, 0.2), 
                rgba(0, 255, 0, 0.2), 
                rgba(0, 0, 255, 0.2), 
                rgba(75, 0, 130, 0.2), 
                rgba(148, 0, 211, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .fantasy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الحالمة */
        .dreamy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 200, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dreamy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الرومانسية */
        .romantic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 150, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .romantic-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الدرامية */
        .dramatic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(0, 0, 0, 0.3), rgba(255, 255, 255, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dramatic-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الغامضة */
        .mysterious-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(0, 0, 50, 0.3), rgba(50, 0, 50, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .mysterious-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المرعبة */
        .scary-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(50, 0, 0, 0.3), rgba(0, 0, 50, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .scary-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المريحة */
        .comforting-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 200, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .comforting-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنعشة */
        .refreshing-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 255, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .refreshing-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنشطة */
        .energizing-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .energizing-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المهدئة */
        .calming-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 200, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .calming-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المبهجة */
        .cheerful-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .cheerful-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الحزينة */
        .sad-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 150, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .sad-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الغاضبة */
        .angry-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 150, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .angry-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة السعيدة */
        .happy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .happy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الحزينة */
        .melancholy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(100, 100, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .melancholy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتفائلة */
        .optimistic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .optimistic-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتشائمة */
        .pessimistic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(100, 100, 100, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .pessimistic-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الواثقة */
        .confident-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .confident-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المترددة */
        .hesitant-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .hesitant-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الجريئة */
        .bold-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 200, 100, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .bold-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخجولة */
        .shy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 200, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .shy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة القوية */
        .strong-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .strong-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الضعيفة */
        .weak-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(100, 100, 100, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .weak-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الناعمة */
        .soft-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .soft-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخشنة */
        .harsh-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.5) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .harsh-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المشرقة */
        .bright-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .bright-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المعتمة */
        .dark-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(0, 0, 0, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dark-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة النظيفة */
        .clean-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .clean-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتسخة */
        .dirty-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 150, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dirty-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة النقية */
        .pure-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .pure-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الملوثة */
        .polluted-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(100, 100, 100, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .polluted-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الطبيعية */
        .natural-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .natural-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الصناعية */
        .artificial-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .artificial-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الشمسية */
        .solar-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 30% 30%, rgba(255, 255, 0, 0.3), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .solar-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة القمرية */
        .lunar-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 70% 30%, rgba(200, 200, 255, 0.3), transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .lunar-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة النجمية */
        .stellar-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 255, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .stellar-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المجرة */
        .galaxy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 20%, rgba(100, 100, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .galaxy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الكونية */
        .cosmic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 20%, rgba(50, 50, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .cosmic-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة السماوية */
        .celestial-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 20%, rgba(150, 150, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .celestial-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الأرضية */
        .terrestrial-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 200, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .terrestrial-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المائية */
        .aquatic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 200, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .aquatic-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الهوائية */
        .aerial-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .aerial-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة النارية */
        .fiery-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 150, 100, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .fiery-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الجليدية */
        .icy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 230, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .icy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الصخرية */
        .rocky-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 150, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .rocky-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الرملية */
        .sandy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 230, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .sandy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة العشبية */
        .grassy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 255, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .grassy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخشبية */
        .woody-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 150, 100, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .woody-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المعدنية */
        .metallic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .metallic-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة البلاستيكية */
        .plastic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .plastic-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الزجاجية */
        .glassy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 230, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .glassy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المرآوية */
        .mirrored-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .mirrored-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الشفافة */
        .transparent-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .transparent-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المعتمة */
        .opaque-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 200, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .opaque-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة اللامعة */
        .shiny-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.5) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .shiny-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الباهتة */
        .dull-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 150, 150, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dull-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة اللامعة */
        .glossy-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .glossy-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة غير اللامعة */
        .matte-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 200, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .matte-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الناعمة */
        .smooth-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(255, 255, 255, 0.3) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .smooth-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الخشنة */
        .rough-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(150, 150, 150, 0.4) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .rough-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المستوية */
        .flat-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 30%, rgba(200, 200, 200, 0.2) 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .flat-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنحنية */
        .curved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 20%, rgba(200, 200, 200, 0.3) 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .curved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المستقيمة */
        .straight-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .straight-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المائلة */
        .slanted-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .slanted-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة العمودية */
        .vertical-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .vertical-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الأفقية */
        .horizontal-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .horizontal-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة القطرية */
        .diagonal-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .diagonal-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتقاطعة */
        .cross-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2)),
                        linear-gradient(-45deg, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .cross-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتوازية */
        .parallel-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2)),
                        linear-gradient(90deg, rgba(200, 200, 200, 0.1), rgba(255, 255, 255, 0.2), rgba(200, 200, 200, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .parallel-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتعامدة */
        .perpendicular-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, rgba(200, 200, 200, 0.2), rgba(255, 255, 255, 0.3), rgba(200, 200, 200, 0.2)),
                        linear-gradient(to bottom, rgba(200, 200, 200, 0.1), rgba(255, 255, 255, 0.2), rgba(200, 200, 200, 0.1));
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .perpendicular-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتقاربة */
        .converging-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, transparent 10%, rgba(255, 255, 255, 0.3) 50%, transparent 90%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .converging-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتباعدة */
        .diverging-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.3) 10%, transparent 50%, rgba(255, 255, 255, 0.3) 90%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .diverging-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتمركزة */
        .centered-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.4) 10%, transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .centered-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الموزعة */
        .distributed-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 30% 30%, rgba(255, 255, 255, 0.3) 10%, transparent 40%),
                        radial-gradient(circle at 70% 30%, rgba(255, 255, 255, 0.3) 10%, transparent 40%),
                        radial-gradient(circle at 30% 70%, rgba(255, 255, 255, 0.3) 10%, transparent 40%),
                        radial-gradient(circle at 70% 70%, rgba(255, 255, 255, 0.3) 10%, transparent 40%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .distributed-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المركزة */
        .focused-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.5) 10%, transparent 50%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .focused-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنتشرة */
        .diffused-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.2) 10%, transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .diffused-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المركزة */
        .concentrated-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.6) 5%, transparent 30%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .concentrated-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتفرقة */
        .scattered-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 20% 20%, rgba(255, 255, 255, 0.2) 5%, transparent 20%),
                        radial-gradient(circle at 80% 20%, rgba(255, 255, 255, 0.2) 5%, transparent 20%),
                        radial-gradient(circle at 20% 80%, rgba(255, 255, 255, 0.2) 5%, transparent 20%),
                        radial-gradient(circle at 80% 80%, rgba(255, 255, 255, 0.2) 5%, transparent 20%),
                        radial-gradient(circle at 50% 50%, rgba(255, 255, 255, 0.2) 5%, transparent 20%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .scattered-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المجمعة */
        .gathered-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.4) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .gathered-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المبعثرة */
        .dispersed-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 10% 10%, rgba(255, 255, 255, 0.2) 5%, transparent 15%),
                        radial-gradient(circle at 90% 10%, rgba(255, 255, 255, 0.2) 5%, transparent 15%),
                        radial-gradient(circle at 10% 90%, rgba(255, 255, 255, 0.2) 5%, transparent 15%),
                        radial-gradient(circle at 90% 90%, rgba(255, 255, 255, 0.2) 5%, transparent 15%),
                        radial-gradient(circle at 50% 10%, rgba(255, 255, 255, 0.2) 5%, transparent 15%),
                        radial-gradient(circle at 10% 50%, rgba(255, 255, 255, 0.2) 5%, transparent 15%),
                        radial-gradient(circle at 90% 50%, rgba(255, 255, 255, 0.2) 5%, transparent 15%),
                        radial-gradient(circle at 50% 90%, rgba(255, 255, 255, 0.2) 5%, transparent 15%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dispersed-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتجمعة */
        .clustered-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 40% 40%, rgba(255, 255, 255, 0.3) 10%, transparent 30%),
                        radial-gradient(circle at 60% 40%, rgba(255, 255, 255, 0.3) 10%, transparent 30%),
                        radial-gradient(circle at 40% 60%, rgba(255, 255, 255, 0.3) 10%, transparent 30%),
                        radial-gradient(circle at 60% 60%, rgba(255, 255, 255, 0.3) 10%, transparent 30%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .clustered-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتفرقة */
        .separated-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 20% 20%, rgba(255, 255, 255, 0.3) 10%, transparent 25%),
                        radial-gradient(circle at 80% 20%, rgba(255, 255, 255, 0.3) 10%, transparent 25%),
                        radial-gradient(circle at 20% 80%, rgba(255, 255, 255, 0.3) 10%, transparent 25%),
                        radial-gradient(circle at 80% 80%, rgba(255, 255, 255, 0.3) 10%, transparent 25%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .separated-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتحدة */
        .unified-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.3) 20%, transparent 70%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unified-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المقسمة */
        .divided-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, rgba(255, 255, 255, 0.3) 50%, transparent 50%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .divided-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المدمجة */
        .merged-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0.3) 50%, rgba(255, 255, 255, 0.2) 100%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .merged-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنفصلة */
        .detached-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 20% 20%, rgba(255, 255, 255, 0.3) 10%, transparent 20%),
                        radial-gradient(circle at 80% 80%, rgba(255, 255, 255, 0.3) 10%, transparent 20%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .detached-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتصلة */
        .connected-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0.3) 50%, rgba(255, 255, 255, 0.2) 100%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .connected-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المنعزلة */
        .isolated-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.4) 10%, transparent 30%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .isolated-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتكاملة */
        .integrated-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.3) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .integrated-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المستقلة */
        .independent-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at 30% 30%, rgba(255, 255, 255, 0.3) 10%, transparent 25%),
                        radial-gradient(circle at 70% 70%, rgba(255, 255, 255, 0.3) 10%, transparent 25%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .independent-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتفاعلة */
        .interactive-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.3) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .interactive-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة الثابتة */
        .static-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.3) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .static-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتحركة */
        .dynamic-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.3) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .dynamic-lighting-mode-overlay.active {
            opacity: 1;
            animation: pulse 2s infinite;
        }

        /* تأثيرات وضع الإضاءة المتغيرة */
        .changing-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.3) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .changing-lighting-mode-overlay.active {
            opacity: 1;
            animation: pulse 3s infinite;
        }

        /* تأثيرات وضع الإضاءة المتطورة */
        .evolving-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.3) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .evolving-lighting-mode-overlay.active {
            opacity: 1;
            animation: pulse 4s infinite;
        }

        /* تأثيرات وضع الإضاءة المتقدمة */
        .advanced-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.4) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .advanced-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة جداً */
        .highly-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.5) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .highly-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة للغاية */
        .extremely-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.6) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .extremely-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل مطلق */
        .absolutely-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.7) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .absolutely-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل نهائي */
        .finally-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.8) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .finally-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل كامل */
        .completely-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 0.9) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .completely-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل تام */
        .totally-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .totally-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل كلي */
        .fully-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .fully-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل شامل */
        .comprehensively-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .comprehensively-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متكامل */
        .integrally-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .integrally-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل موحد */
        .uniformly-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .uniformly-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متناسق */
        .coherently-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .coherently-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متجانس */
        .homogeneously-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .homogeneously-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متسق */
        .consistently-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .consistently-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل منتظم */
        .regularly-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .regularly-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متكرر */
        .repeatedly-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .repeatedly-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل مستمر */
        .continuously-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .continuously-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متواصل */
        .successively-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .successively-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متعاقب */
        .alternately-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .alternately-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متناوب */
        .alternating-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .alternating-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متبادل */
        .reciprocal-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .reciprocal-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل معاكس */
        .inverse-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .inverse-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل معكوس */
        .reversed-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .reversed-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متناقض */
        .contrasting-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .contrasting-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متضاد */
        .opposing-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .opposing-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متعارض */
        .conflicting-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .conflicting-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متصارع */
        .clashing-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .clashing-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متضارب */
        .discrepant-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .discrepant-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل متباين */
        .disparate-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .disparate-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل مختلف */
        .different-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .different-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل مميز */
        .distinct-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .distinct-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل فريد */
        .unique-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unique-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل خاص */
        .special-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .special-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل استثنائي */
        .exceptional-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .exceptional-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل غير عادي */
        .unusual-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unusual-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل نادر */
        .rare-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .rare-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل فريد من نوعه */
        .one-of-a-kind-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .one-of-a-kind-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا مثيل له */
        .unparalleled-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unparalleled-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يضاهى */
        .unrivaled-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unrivaled-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا مثيل له */
        .unmatched-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unmatched-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يضاهى */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .invincible-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يهزم */
        .undefeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .undefeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .unconquerable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unconquerable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يغلب */
        .unbeatable-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, rgba(255, 255, 255, 1) 20%, transparent 60%);
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 6;
        }

        .unbeatable-evolved-lighting-mode-overlay.active {
            opacity: 1;
        }

        /* تأثيرات وضع الإضاءة المتطورة بشكل لا يقهر */
        .invincible-evolved-lighting-mode-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient
