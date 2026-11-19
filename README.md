```html
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام تتبع وإعلان محطات الأتوبيس الذكي</title>
    <!-- تحميل Tailwind CSS للتصميم المتجاوب -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- تحميل مكتبة Leaflet للخرائط التفاعلية -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <!-- خط القاهرة (Cairo) لدعم اللغة العربية بشكل جمالي -->
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Cairo', sans-serif;
            background-color: #f0f4f8;
        }
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #e2e8f0;
        }
        ::-webkit-scrollbar-thumb {
            background: #0284c7;
            border-radius: 10px;
        }
        #map {
            height: 100%;
            min-height: 380px;
            z-index: 10;
        }
        .pulse-ring {
            animation: pulse-animation 2s infinite;
        }
        @keyframes pulse-animation {
            0% { box-shadow: 0 0 0 0 rgba(14, 165, 233, 0.7); }
            70% { box-shadow: 0 0 0 15px rgba(14, 165, 233, 0); }
            100% { box-shadow: 0 0 0 0 rgba(14, 165, 233, 0); }
        }
    </style>
</head>
<body class="text-slate-800 antialiased min-h-screen flex flex-col">

    <!-- الشريط العلوي (Header) -->
    <header class="bg-gradient-to-l from-blue-950 via-blue-900 to-sky-900 text-white shadow-lg border-b-4 border-amber-400">
        <div class="max-w-7xl mx-auto px-4 py-4 flex flex-col md:flex-row items-center justify-between gap-4">
            <div class="flex items-center gap-4">
                <div class="bg-white/10 p-3 rounded-2xl backdrop-blur-sm border border-white/20">
                    <svg class="w-10 h-10 text-amber-400" fill="currentColor" viewBox="0 0 24 24">
                        <path d="M4 16c0 .88.39 1.67 1 2.22V20c0 .55.45 1 1 1h1c.55 0 1-.45 1-1v-1h8v1c0 .55.45 1 1 1h1c.55 0 1-.45 1-1v-1.78c.61-.55 1-1.34 1-2.22V6c0-3.5-3.58-4-8-4s-8 .5-8 4v10zm3.5 1c-.83 0-1.5-.67-1.5-1.5S6.67 14 7.5 14s1.5.67 1.5 1.5S8.33 17 7.5 17zm9 0c-.83 0-1.5-.67-1.5-1.5s.67-1.5 1.5-1.5 1.5.67 1.5 1.5-.67 1.5-1.5 1.5zm1.5-6H6V6h12v5z"/>
                    </svg>
                </div>
                <div>
                    <h1 class="text-xl md:text-2xl font-black tracking-wide text-white">
                        خط سير أتوبيس: <span class="text-amber-400">أكتوبر ◄ كورنيش المعادي</span>
                    </h1>
                    <p class="text-xs md:text-sm text-sky-200 flex items-center gap-2 mt-1">
                        <span class="bg-sky-700/80 px-2.5 py-0.5 rounded-full border border-sky-400/30">عن طريق الطريق الدائري</span>
                        <span>• نظام الإعلان الصوتي الذكي GPS</span>
                    </p>
                </div>
            </div>

            <!-- أزرار التحكم الرئيسية -->
            <div class="flex flex-wrap items-center justify-center gap-2">
                <button id="btn-real-gps" onclick="startRealGPS()" class="flex items-center gap-2 bg-emerald-600 hover:bg-emerald-500 active:scale-95 transition text-white font-bold py-2.5 px-4 rounded-xl shadow-md text-sm">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"/><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"/></svg>
                    <span>تفعيل GPS وبث الموقع للجميع</span>
                </button>
                <button id="btn-sim-gps" onclick="toggleSimulation()" class="flex items-center gap-2 bg-amber-500 hover:bg-amber-400 active:scale-95 transition text-slate-950 font-bold py-2.5 px-4 rounded-xl shadow-md text-sm">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14.752 11.168l-3.197-2.132A1 1 0 0010 9.87v4.263a1 1 0 001.555.832l3.197-2.132a1 1 0 000-1.664z"/><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>
                    <span id="sim-btn-text">محاكاة الرحلة (تجربة)</span>
                </button>
            </div>
        </div>
    </header>

    <!-- شريط حالة البث المباشر المشترك للجميع -->
    <div class="bg-blue-950 text-sky-100 border-b border-sky-800/60 px-4 py-2 text-xs">
        <div class="max-w-7xl mx-auto flex flex-wrap items-center justify-between gap-2">
            <div class="flex items-center gap-2">
                <span id="live-broadcast-dot" class="w-2.5 h-2.5 rounded-full bg-emerald-400 animate-ping"></span>
                <span class="font-bold text-amber-300">البث المباشر للركاب:</span>
                <span id="live-viewer-status">متصل بغرفة التتبع المباشر (الجميع يرى موقع الأتوبيس الآن)</span>
            </div>
            <div class="flex items-center gap-3">
                <span id="last-update-time" class="text-sky-300 font-mono">آخر تحديث: الآن</span>
                <button onclick="setManualLocationMode()" class="bg-sky-800 hover:bg-sky-700 text-white px-2.5 py-1 rounded-lg border border-sky-600 transition font-bold">
                    📍 حدد مكاني يدوياً على الخريطة
                </button>
            </div>
        </div>
    </div>

    <!-- شريط التنبيهات المخصص -->
    <div id="toast-box" class="hidden fixed bottom-5 left-5 right-5 md:left-auto md:right-5 md:w-96 bg-slate-900 text-white p-4 rounded-2xl shadow-2xl border-r-4 border-amber-400 z-50 transition-all duration-300">
        <div class="flex items-start gap-3">
            <div class="text-amber-400 mt-0.5">
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>
            </div>
            <div class="flex-1">
                <h4 id="toast-title" class="font-bold text-sm text-amber-300">تنبيه النظام</h4>
                <p id="toast-msg" class="text-xs text-slate-200 mt-1 leading-relaxed"></p>
            </div>
            <button onclick="hideToast()" class="text-slate-400 hover:text-white">✕</button>
        </div>
    </div>

    <!-- المحتوى الرئيسي -->
    <main class="flex-1 max-w-7xl w-full mx-auto p-4 grid grid-cols-1 lg:grid-cols-12 gap-6">
        
        <!-- العمود الأيمن: شاشة العرض وقائمة المحطات -->
        <div class="lg:col-span-5 flex flex-col gap-5">
            
            <!-- شاشة الإعلان الصوتي والبيانات الحية -->
            <div class="bg-slate-900 text-white rounded-3xl p-5 shadow-xl border-2 border-slate-700 relative overflow-hidden">
                <div class="flex items-center justify-between border-b border-slate-700/80 pb-3 mb-4">
                    <div class="flex items-center gap-2">
                        <span id="status-dot" class="w-3 h-3 rounded-full bg-slate-500"></span>
                        <span id="gps-mode-label" class="text-xs font-semibold text-slate-300">في انتظار بدء الرحلة...</span>
                    </div>
                    <button onclick="announceCurrentState(true)" title="إعادة الإعلان الصوتي الآن" class="bg-sky-600/30 hover:bg-sky-600/50 text-sky-300 border border-sky-500/40 px-3 py-1 rounded-full text-xs font-bold flex items-center gap-1.5 transition">
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z"/></svg>
                        <span>نطق الإعلان</span>
                    </button>
                </div>

                <!-- المحطة الحالية والقادمة -->
                <div class="grid grid-cols-1 gap-3">
                    <div class="bg-slate-800/90 p-3.5 rounded-2xl border border-slate-700">
                        <span class="text-xs text-slate-400 block mb-1">الموقع الحالي / آخر محطة:</span>
                        <div id="current-station-name" class="text-lg font-bold text-emerald-400">نقطة الانطلاق: أكتوبر</div>
                    </div>

                    <div class="bg-gradient-to-l from-sky-950 to-slate-800 p-4 rounded-2xl border border-sky-500/40 shadow-inner">
                        <div class="flex justify-between items-center mb-1">
                            <span class="text-xs text-sky-300 font-semibold">المحطة التالية:</span>
                            <span id="next-station-badge" class="text-[11px] bg-sky-500/20 text-sky-300 px-2 py-0.5 rounded-full">محطة رقم 2</span>
                        </div>
                        <div id="next-station-name" class="text-2xl font-black text-amber-400 tracking-wide">صينية الواحات</div>
                        
                        <!-- فرق التوقيت والمسافة -->
                        <div class="grid grid-cols-2 gap-3 mt-4 pt-3 border-t border-slate-700/80">
                            <div class="bg-slate-900/70 p-2.5 rounded-xl text-center">
                                <span class="text-[11px] text-slate-400 block">فرق التوقيت المتبقي</span>
                                <span id="eta-display" class="text-lg font-black text-white">-- دقيقة</span>
                            </div>
                            <div class="bg-slate-900/70 p-2.5 rounded-xl text-center">
                                <span class="text-[11px] text-slate-400 block">المسافة المتبقية</span>
                                <span id="distance-display" class="text-lg font-black text-sky-400">-- كم</span>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- نص الإعلان الصوتي المعروض -->
                <div class="mt-4 bg-black/40 rounded-xl p-3 border border-slate-800">
                    <div class="flex items-center justify-between text-xs text-amber-400 font-bold mb-1">
                        <span>🔊 آخر إعلان صوتي:</span>
                        <span id="gps-coords-debug" class="text-[10px] font-mono text-slate-400" dir="ltr">GPS: Waiting...</span>
                    </div>
                    <p id="announcement-text" class="text-xs md:text-sm text-slate-200 leading-relaxed">
                        اضغط على "محاكاة الرحلة" للتجربة الفورية، أو "تفعيل GPS وبث الموقع للجميع" أثناء ركوب الأتوبيس.
                    </p>
                </div>

                <!-- رسالة توضيحية وحل بديل في حال حظر GPS داخل نافذة المعاينة -->
                <div id="iframe-warning" class="hidden mt-3 bg-rose-950/90 border border-rose-500/60 rounded-xl p-3 text-xs text-rose-100">
                    <p id="gps-error-detail" class="leading-relaxed mb-2"></p>
                    <button onclick="downloadStandaloneHTML()" class="w-full bg-amber-500 hover:bg-amber-400 text-slate-950 font-bold py-2 px-3 rounded-lg transition flex items-center justify-center gap-2">
                        <span>📥 تنزيل الملف لتشغيله مباشرة على الهاتف (GPS كامل)</span>
                    </button>
                </div>

                <!-- إعدادات السرعة ونطاق الالتقاط -->
                <div class="mt-4 pt-3 border-t border-slate-800 flex flex-wrap items-center justify-between gap-2 text-xs text-slate-400">
                    <div class="flex items-center gap-2">
                        <span>السرعة:</span>
                        <span id="current-speed-display" class="text-white font-bold">50 كم/س</span>
                    </div>
                    <div class="flex items-center gap-2">
                        <label for="capture-radius">نطاق المحطة:</label>
                        <select id="capture-radius" class="bg-slate-800 text-white border border-slate-700 rounded-lg px-2 py-1 text-xs">
                            <option value="0.25">250 متر</option>
                            <option value="0.45" selected>450 متر (موصى به)</option>
                            <option value="0.80">800 متر</option>
                        </select>
                    </div>
                    <div class="flex items-center gap-2">
                        <label for="sim-speed">سرعة المحاكاة:</label>
                        <select id="sim-speed" class="bg-slate-800 text-white border border-slate-700 rounded-lg px-2 py-1 text-xs">
                            <option value="1">عادي (1x)</option>
                            <option value="3" selected>سريع (3x)</option>
                            <option value="6">فائق (6x)</option>
                        </select>
                    </div>
                </div>
            </div>

            <!-- قائمة المحطات بالترتيب -->
            <div class="bg-white rounded-3xl p-5 shadow-md border border-slate-200 flex-1 flex flex-col">
                <div class="flex items-center justify-between mb-4 border-b border-slate-100 pb-3">
                    <h2 class="text-base md:text-lg font-black text-blue-950 flex items-center gap-2">
                        <span class="bg-blue-900 text-white w-7 h-7 rounded-lg flex items-center justify-center text-sm">🚌</span>
                        <span>المحطات بالترتيب (اضغط للانتقال)</span>
                    </h2>
                    <button onclick="resetTrip()" class="text-xs text-rose-600 hover:bg-rose-50 px-2.5 py-1 rounded-lg font-bold transition">
                        إعادة ضبط
                    </button>
                </div>

                <div id="stations-list" class="space-y-2.5 overflow-y-auto pr-1 max-h-[340px]">
                    <!-- يتم توليد المحطات ديناميكياً بواسطة الجافاسكريبت -->
                </div>
            </div>
        </div>

        <!-- العمود الأيسر: الخريطة التفاعلية لمسار الدائري -->
        <div class="lg:col-span-7 flex flex-col">
            <div class="bg-white rounded-3xl p-3 shadow-md border border-slate-200 flex-1 flex flex-col min-h-[450px]">
                <div class="flex items-center justify-between px-2 pb-2 mb-2 border-b border-slate-100">
                    <span class="text-xs md:text-sm font-bold text-slate-700">خريطة المسار المباشر (الطريق الدائري)</span>
                    <div class="flex items-center gap-3 text-xs">
                        <span class="flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-full bg-emerald-500 inline-block"></span> البداية</span>
                        <span class="flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-full bg-sky-600 inline-block"></span> محطة</span>
                        <span class="flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-full bg-amber-500 inline-block"></span> الأتوبيس</span>
                    </div>
                </div>
                <div id="map" class="rounded-2xl flex-1 w-full overflow-hidden border border-slate-200"></div>
            </div>
        </div>

    </main>

    <!-- وحدة البث المباشر المشترك عبر السحابة -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        window.liveBusSync = {
            enabled: false,
            isBroadcaster: false,
            db: null,
            docRef: null,
            userId: null,
            broadcast: async function(data) {
                if (!this.enabled || !this.docRef || !this.userId) return;
                try {
                    await setDoc(this.docRef, {
                        ...data,
                        broadcasterId: this.userId,
                        updatedAt: Date.now()
                    }, { merge: true });
                } catch (e) {
                    console.warn("تعذر إرسال التحديث المباشر:", e);
                }
            }
        };

        async function initLiveTracking() {
            try {
                if (typeof __firebase_config === 'undefined') return;
                const firebaseConfig = JSON.parse(__firebase_config);
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
                const app = initializeApp(firebaseConfig);
                const auth = getAuth(app);
                const db = getFirestore(app);

                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                const user = auth.currentUser;
                if (!user) return;

                window.liveBusSync.db = db;
                window.liveBusSync.userId = user.uid;
                window.liveBusSync.docRef = doc(db, 'artifacts', appId, 'public', 'data', 'bus_live_status', 'october_maadi_line1');
                window.liveBusSync.enabled = true;

                onSnapshot(window.liveBusSync.docRef, (snap) => {
                    if (!snap.exists()) return;
                    const data = snap.data();
                    if (!window.liveBusSync.isBroadcaster && data.lat && data.lng) {
                        const timeStr = new Date(data.updatedAt || Date.now()).toLocaleTimeString('ar-EG');
                        document.getElementById('last-update-time').innerText = `آخر تحديث: ${timeStr}`;
                        document.getElementById('gps-mode-label').innerText = "مباشر: يتم عرض موقع الأتوبيس الفعلي للجميع";
                        document.getElementById('status-dot').className = "w-3 h-3 rounded-full bg-emerald-500 animate-pulse";
                        if (window.receiveLiveBusUpdate) {
                            window.receiveLiveBusUpdate(data);
                        }
                    }
                }, (err) => {
                    console.warn("خطأ في استقبال البث المباشر:", err);
                });
            } catch (err) {
                console.warn("يعمل في الوضع المحلي:", err);
            }
        }

        initLiveTracking();
    </script>

    <!-- المنطق الرئيسي للتطبيق -->
    <script>
        const STATIONS = [
            { id: 1, name: "أكتوبر", lat: 29.9737, lng: 30.9438, desc: "نقطة البداية - مدينة 6 أكتوبر" },
            { id: 2, name: "صينية الواحات", lat: 29.9708, lng: 31.0265, desc: "تقاطع طريق الواحات مع الدائري" },
            { id: 3, name: "سوق الجملة", lat: 29.9565, lng: 31.0695, desc: "أمام سوق الجملة بمدينة 6 أكتوبر" },
            { id: 4, name: "زويل", lat: 29.9235, lng: 31.1210, desc: "بجوار مدينة زويل للعلوم والتكنولوجيا" },
            { id: 5, name: "مصر للطيران", lat: 29.9580, lng: 31.1585, desc: "منطقة مصر للطيران على الطريق الدائري" },
            { id: 6, name: "الكنيسة", lat: 29.9850, lng: 31.1860, desc: "نزلة الكنيسة - الجيزة" },
            { id: 7, name: "المنيب", lat: 29.9815, lng: 31.2130, desc: "ميدان وموقف المنيب" },
            { id: 8, name: "كورنيش المعادي", lat: 29.9595, lng: 31.2485, desc: "نهاية الخط - كورنيش النيل بالمعادي" }
        ];

        const ROUTE_PATH = [
            [29.9737, 30.9438], // 1. أكتوبر
            [29.9745, 30.9850],
            [29.9708, 31.0265], // 2. صينية الواحات
            [29.9640, 31.0480],
            [29.9565, 31.0695], // 3. سوق الجملة
            [29.9400, 31.0950],
            [29.9235, 31.1210], // 4. زويل
            [29.9380, 31.1420],
            [29.9580, 31.1585], // 5. مصر للطيران
            [29.9720, 31.1730],
            [29.9850, 31.1860], // 6. الكنيسة
            [29.9840, 31.2000],
            [29.9815, 31.2130], // 7. المنيب
            [29.9710, 31.2310],
            [29.9595, 31.2485]  // 8. كورنيش المعادي
        ];

        let map, busMarker, routePolyline;
        let currentStationIndex = 0;
        let nextStationIndex = 1;
        let announcedApproaching = false;
        let isSimulating = false;
        let simInterval = null;
        let watchId = null;
        let simSegmentIndex = 0;
        let simProgress = 0;
        let busPosition = { lat: STATIONS[0].lat, lng: STATIONS[0].lng };
        let currentSpeedKmh = 50;
        let isFirstRealGpsFix = true;
        let gpsCircles = [];
        let isManualPinMode = false;

        window.receiveLiveBusUpdate = function(data) {
            if (typeof data.nextStationIndex === 'number') {
                nextStationIndex = data.nextStationIndex;
            }
            if (typeof data.currentStationIndex === 'number') {
                currentStationIndex = data.currentStationIndex;
            }
            if (map) {
                map.panTo([data.lat, data.lng], { animate: true, duration: 0.4 });
            }
            updateUIWithPosition(data.lat, data.lng, data.speedKmh || 45, true);
            renderStationsList();
        };

        function setManualLocationMode() {
            isManualPinMode = !isManualPinMode;
            if (isManualPinMode) {
                showToast("وضع التحديد اليدوي نشط", "اضغط في أي مكان على الخريطة لتحديث مكانك وبثه فوراً لجميع الركاب!");
                document.getElementById('gps-mode-label').innerText = "اضغط على الخريطة لتحديد مكانك وبثه";
            } else {
                showToast("تم الإلغاء", "تم إيقاف وضع النقر على الخريطة.");
            }
        }

        window.onload = function () {
            initMap();
            renderStationsList();
            updateUIWithPosition(busPosition.lat, busPosition.lng, 50);

            const radiusSelect = document.getElementById('capture-radius');
            if (radiusSelect) {
                radiusSelect.addEventListener('change', drawStationCircles);
            }
        };

        function drawStationCircles() {
            if (!map) return;
            gpsCircles.forEach(c => map.removeLayer(c));
            gpsCircles = [];
            const radiusMeters = parseFloat(document.getElementById('capture-radius').value) * 1000;
            STATIONS.forEach(st => {
                const circle = L.circle([st.lat, st.lng], {
                    color: '#0284c7',
                    fillColor: '#38bdf8',
                    fillOpacity: 0.12,
                    weight: 1,
                    radius: radiusMeters
                }).addTo(map);
                gpsCircles.push(circle);
            });
        }

        function initMap() {
            map = L.map('map').setView([29.9600, 31.1000], 11);

            L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png', {
                attribution: '&copy; OpenStreetMap contributors',
                maxZoom: 18
            }).addTo(map);

            routePolyline = L.polyline(ROUTE_PATH, {
                color: '#0284c7',
                weight: 6,
                opacity: 0.85,
                lineJoin: 'round'
            }).addTo(map);

            STATIONS.forEach((station, idx) => {
                const isFirst = idx === 0;
                const isLast = idx === STATIONS.length - 1;
                const bgColor = isFirst ? '#16a34a' : (isLast ? '#dc2626' : '#0284c7');

                const iconHtml = `
                    <div style="background-color: ${bgColor}; color: white; width: 28px; height: 28px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; font-family: 'Cairo', sans-serif; border: 2px solid white; box-shadow: 0 2px 6px rgba(0,0,0,0.4); font-size: 13px;">
                        ${station.id}
                    </div>
                `;

                const customIcon = L.divIcon({
                    html: iconHtml,
                    className: '',
                    iconSize: [28, 28],
                    iconAnchor: [14, 14]
                });

                L.marker([station.lat, station.lng], { icon: customIcon })
                    .addTo(map)
                    .bindPopup(`<b>${station.id}. ${station.name}</b><br><span style="font-size:12px">${station.desc}</span>`);
            });

            drawStationCircles();

            const busIconHtml = `
                <div class="pulse-ring" style="background-color: #f59e0b; color: #0f172a; width: 38px; height: 38px; border-radius: 50%; display: flex; align-items: center; justify-content: center; border: 3px solid #0f172a; box-shadow: 0 4px 10px rgba(0,0,0,0.5); font-size: 18px;">
                    🚌
                </div>
            `;
            const busIcon = L.divIcon({
                html: busIconHtml,
                className: '',
                iconSize: [38, 38],
                iconAnchor: [19, 19]
            });

            busMarker = L.marker([busPosition.lat, busPosition.lng], { icon: busIcon, zIndexOffset: 1000 }).addTo(map);
            map.fitBounds(routePolyline.getBounds(), { padding: [30, 30] });

            map.on('click', function(e) {
                if (isManualPinMode) {
                    if (window.liveBusSync) window.liveBusSync.isBroadcaster = true;
                    isFirstRealGpsFix = false;
                    autoAlignToNearestStation(e.latlng.lat, e.latlng.lng, 50);
                    updateUIWithPosition(e.latlng.lat, e.latlng.lng, 50);
                    document.getElementById('gps-mode-label').innerText = "بث مباشر (تم تحديد الموقع يدوياً)";
                    document.getElementById('status-dot').className = "w-3 h-3 rounded-full bg-emerald-500";
                    isManualPinMode = false;
                    showToast("تم التحديث والبث!", "تم تحديث موقعك وبثه مباشرة لجميع الركاب.");
                }
            });
        }

        function renderStationsList() {
            const listContainer = document.getElementById('stations-list');
            if (!listContainer) return;
            listContainer.innerHTML = '';

            STATIONS.forEach((station, index) => {
                const isPassed = index < nextStationIndex;
                const isNext = index === nextStationIndex;

                let cardStyle = 'bg-slate-50 border-slate-200 text-slate-700';
                let badgeStyle = 'bg-blue-600 text-white';
                let statusBadge = '';

                if (isPassed) {
                    cardStyle = 'bg-emerald-50/70 border-emerald-200 text-emerald-900';
                    badgeStyle = 'bg-emerald-600 text-white';
                    statusBadge = '<span class="text-[11px] font-bold text-emerald-700 bg-emerald-100 px-2 py-0.5 rounded-full">تم المرور ✓</span>';
                } else if (isNext) {
                    cardStyle = 'bg-amber-50 border-amber-400 text-slate-900 ring-2 ring-amber-400/40 shadow-sm';
                    badgeStyle = 'bg-amber-500 text-slate-950';
                    statusBadge = '<span class="text-[11px] font-bold text-amber-800 bg-amber-200 px-2 py-0.5 rounded-full animate-pulse">المحطة التالية</span>';
                }

                const itemHtml = `
                    <div onclick="jumpToStation(${index})" class="cursor-pointer transition-all hover:scale-[1.01] flex items-center justify-between p-3 rounded-2xl border ${cardStyle}">
                        <div class="flex items-center gap-3">
                            <div class="w-8 h-8 rounded-full flex items-center justify-center font-black text-sm shadow-sm ${badgeStyle}">
                                ${station.id}
                            </div>
                            <div>
                                <div class="font-bold text-sm md:text-base">${station.name}</div>
                                <div class="text-[11px] text-slate-500">${station.desc}</div>
                            </div>
                        </div>
                        <div>${statusBadge}</div>
                    </div>
                `;
                listContainer.insertAdjacentHTML('beforeend', itemHtml);
            });
        }

        function calculateDistanceKm(lat1, lon1, lat2, lon2) {
            const R = 6371;
            const dLat = (lat2 - lat1) * Math.PI / 180;
            const dLon = (lon2 - lon1) * Math.PI / 180;
            const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                      Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                      Math.sin(dLon / 2) * Math.sin(dLon / 2);
            const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
            return R * c;
        }

        function updateUIWithPosition(lat, lng, speedKmh, isFromRemote = false) {
            busPosition = { lat, lng };
            if (busMarker) {
                busMarker.setLatLng([lat, lng]);
            }

            const debugEl = document.getElementById('gps-coords-debug');
            if (debugEl) {
                debugEl.innerText = `${lat.toFixed(4)}, ${lng.toFixed(4)}`;
            }

            const effectiveSpeed = (speedKmh && speedKmh > 10) ? speedKmh : 45;
            currentSpeedKmh = Math.round(effectiveSpeed);
            document.getElementById('current-speed-display').innerText = `${currentSpeedKmh} كم/س`;

            if (!isFromRemote && window.liveBusSync && window.liveBusSync.isBroadcaster) {
                const timeStr = new Date().toLocaleTimeString('ar-EG');
                document.getElementById('last-update-time').innerText = `آخر بث: ${timeStr}`;
                window.liveBusSync.broadcast({
                    lat,
                    lng,
                    speedKmh: currentSpeedKmh,
                    currentStationIndex,
                    nextStationIndex
                });
            }

            const captureRadiusKm = parseFloat(document.getElementById('capture-radius').value) || 0.45;
            const approachRadiusKm = captureRadiusKm + 0.55;

            if (!isSimulating) {
                for (let i = nextStationIndex; i < STATIONS.length; i++) {
                    const d = calculateDistanceKm(lat, lng, STATIONS[i].lat, STATIONS[i].lng);
                    if (d <= captureRadiusKm) {
                        nextStationIndex = i;
                        break;
                    }
                }
            }

            if (nextStationIndex >= STATIONS.length) {
                document.getElementById('current-station-name').innerText = "وصلنا إلى: كورنيش المعادي (نهاية الخط)";
                document.getElementById('next-station-name').innerText = "نهاية الرحلة";
                document.getElementById('eta-display').innerText = "0 دقيقة";
                document.getElementById('distance-display').innerText = "0 كم";
                return;
            }

            const targetStation = STATIONS[nextStationIndex];
            const distKm = calculateDistanceKm(lat, lng, targetStation.lat, targetStation.lng);

            let minutesRemaining = Math.ceil((distKm / effectiveSpeed) * 60);
            if (minutesRemaining < 1 && distKm > 0.08) minutesRemaining = 1;

            document.getElementById('next-station-badge').innerText = `محطة رقم ${targetStation.id}`;
            document.getElementById('next-station-name').innerText = targetStation.name;
            document.getElementById('distance-display').innerText = distKm < 1
                ? `${Math.round(distKm * 1000)} متر`
                : `${distKm.toFixed(1)} كم`;
            document.getElementById('eta-display').innerText = formatArabicTime(minutesRemaining);

            if (distKm <= approachRadiusKm && distKm > captureRadiusKm && !announcedApproaching) {
                announcedApproaching = true;
                const msg = `نقترب الآن من محطة ${targetStation.name}، فرق التوقيت المتبقي حوالي ${formatArabicTime(minutesRemaining)}.`;
                triggerAnnouncement(msg);
            }

            if (distKm <= captureRadiusKm) {
                currentStationIndex = nextStationIndex;
                document.getElementById('current-station-name').innerText = `وصلنا الآن: ${targetStation.name}`;

                if (nextStationIndex + 1 < STATIONS.length) {
                    nextStationIndex++;
                    announcedApproaching = false;

                    const upcomingStation = STATIONS[nextStationIndex];
                    const nextDistKm = calculateDistanceKm(lat, lng, upcomingStation.lat, upcomingStation.lng);
                    const nextMinutes = Math.max(1, Math.ceil((nextDistKm / effectiveSpeed) * 60));

                    const msg = `وصلنا إلى محطة ${targetStation.name}. المحطة التالية هي ${upcomingStation.name}، وفرق التوقيت بيننا وبينها حوالي ${formatArabicTime(nextMinutes)}.`;
                    triggerAnnouncement(msg);
                    renderStationsList();
                } else {
                    nextStationIndex++;
                    const msg = `وصلنا الآن إلى محطة ${targetStation.name}، وهي نهاية خط السير. نتمنى لكم رحلة سعيدة.`;
                    triggerAnnouncement(msg);
                    renderStationsList();
                    if (isSimulating) toggleSimulation();
                }
            }
        }

        function formatArabicTime(minutes) {
            if (minutes <= 0) return "أقل من دقيقة";
            if (minutes === 1) return "دقيقة واحدة";
            if (minutes === 2) return "دقيقتين";
            if (minutes >= 3 && minutes <= 10) return `${minutes} دقائق`;
            return `${minutes} دقيقة`;
        }

        function triggerAnnouncement(text) {
            document.getElementById('announcement-text').innerText = text;
            showToast("إعلان المحطة", text);

            try {
                if ('speechSynthesis' in window) {
                    window.speechSynthesis.cancel();
                    const utterance = new SpeechSynthesisUtterance(text);
                    utterance.lang = 'ar-EG';
                    utterance.rate = 0.95;
                    utterance.pitch = 1.0;

                    const voices = window.speechSynthesis.getVoices();
                    const arabicVoice = voices.find(v => v.lang.includes('ar'));
                    if (arabicVoice) {
                        utterance.voice = arabicVoice;
                    }

                    window.speechSynthesis.speak(utterance);
                }
            } catch (err) {
                console.error("خطأ في تشغيل الصوت:", err);
            }
        }

        function announceCurrentState(manual = false) {
            if (nextStationIndex >= STATIONS.length) {
                triggerAnnouncement("وصلنا إلى نهاية الخط في كورنيش المعادي.");
                return;
            }
            const nextSt = STATIONS[nextStationIndex];
            const distKm = calculateDistanceKm(busPosition.lat, busPosition.lng, nextSt.lat, nextSt.lng);
            const mins = Math.max(1, Math.ceil((distKm / currentSpeedKmh) * 60));
            const msg = `المحطة التالية هي ${nextSt.name}، وفرق التوقيت بيننا وبين المحطة التالية حوالي ${formatArabicTime(mins)}.`;
            triggerAnnouncement(msg);
        }

        function startRealGPS() {
            if (isSimulating) toggleSimulation();
            if (window.liveBusSync) window.liveBusSync.isBroadcaster = true;

            if ('speechSynthesis' in window) {
                const unlockUtterance = new SpeechSynthesisUtterance("");
                window.speechSynthesis.speak(unlockUtterance);
            }

            if (!("geolocation" in navigator)) {
                handleGPSError({ code: 0, message: "المتصفح لا يدعم خاصية تحديد المواقع." });
                return;
            }

            isFirstRealGpsFix = true;
            document.getElementById('iframe-warning').classList.add('hidden');
            document.getElementById('gps-mode-label').innerText = "جاري الاتصال بالأقمار الصناعية (GPS)...";
            document.getElementById('status-dot').className = "w-3 h-3 rounded-full bg-amber-400 animate-ping";

            if (watchId !== null) navigator.geolocation.clearWatch(watchId);

            navigator.geolocation.getCurrentPosition(
                (pos) => onGPSSuccess(pos),
                (err) => handleGPSError(err),
                {
                    enableHighAccuracy: false,
                    maximumAge: 10000,
                    timeout: 8000
                }
            );

            watchId = navigator.geolocation.watchPosition(
                (position) => onGPSSuccess(position),
                (error) => {
                    if (isFirstRealGpsFix) {
                        handleGPSError(error);
                    }
                },
                {
                    enableHighAccuracy: true,
                    maximumAge: 2000,
                    timeout: 20000
                }
            );
        }

        function onGPSSuccess(position) {
            const lat = position.coords.latitude;
            const lng = position.coords.longitude;
            const speedKmh = position.coords.speed ? (position.coords.speed * 3.6) : 45;

            document.getElementById('iframe-warning').classList.add('hidden');
            document.getElementById('gps-mode-label').innerText = "متصل بـ GPS الحقيقي (تتبع تلقائي)";
            document.getElementById('status-dot').className = "w-3 h-3 rounded-full bg-emerald-500";

            if (isFirstRealGpsFix) {
                isFirstRealGpsFix = false;
                autoAlignToNearestStation(lat, lng, speedKmh);
            }

            map.panTo([lat, lng]);
            updateUIWithPosition(lat, lng, speedKmh);
        }

        function handleGPSError(error) {
            if (watchId !== null) {
                navigator.geolocation.clearWatch(watchId);
                watchId = null;
            }

            document.getElementById('gps-mode-label').innerText = "تعذر الوصول لإشارة GPS";
            document.getElementById('status-dot').className = "w-3 h-3 rounded-full bg-rose-500";

            let reasonText = "تعذر تحديد موقعك الحالي.";
            if (error.code === 1) {
                reasonText = "تم رفض إذن الموقع (Permission Denied) لأن التطبيق يعمل داخل نافذة معاينة محمية، أو أنك لم تسمح للمتصفح بالوصول للموقع.";
            } else if (error.code === 2) {
                reasonText = "إشارة الموقع غير متاحة حالياً (تأكد من تشغيل GPS / Location في إعدادات هاتفك).";
            } else if (error.code === 3) {
                reasonText = "انتهت مهلة البحث عن الأقمار الصناعية (قد تكون داخل مبنى مغلق).";
            }

            document.getElementById('gps-error-detail').innerText = reasonText + " يمكنك الضغط على زر (📍 حدد مكاني يدوياً على الخريطة) في الأعلى لبث مكانك فوراً للجميع، أو تنزيل الملف لتشغيله على هاتفك.";
            document.getElementById('iframe-warning').classList.remove('hidden');
            showToast("حل بديل فوري", "المعاينة تمنع GPS التلقائي؛ اضغط على (حدد مكاني يدوياً على الخريطة) بالأعلى أو نزّل الملف لهاتفك.");
        }

        function downloadStandaloneHTML() {
            const htmlContent = "<!DOCTYPE html>\n" + document.documentElement.outerHTML;
            const blob = new Blob([htmlContent], { type: "text/html;charset=utf-8" });
            const url = URL.createObjectURL(blob);
            const a = document.createElement("a");
            a.href = url;
            a.download = "bus-gps-tracker.html";
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            showToast("تم التنزيل بنجاح", "افتح ملف bus-gps-tracker.html في متصفح Chrome على هاتفك ليعمل GPS الحقيقي بكامل صلاحياته.");
        }

        function autoAlignToNearestStation(lat, lng, speedKmh) {
            let nearestIdx = 0;
            let minDist = Infinity;

            STATIONS.forEach((st, idx) => {
                const d = calculateDistanceKm(lat, lng, st.lat, st.lng);
                if (d < minDist) {
                    minDist = d;
                    nearestIdx = idx;
                }
            });

            if (nearestIdx < STATIONS.length - 1) {
                currentStationIndex = nearestIdx;
                nextStationIndex = nearestIdx + 1;
            } else {
                currentStationIndex = STATIONS.length - 2;
                nextStationIndex = STATIONS.length - 1;
            }

            announcedApproaching = false;
            renderStationsList();

            const nextSt = STATIONS[nextStationIndex];
            const distToNext = calculateDistanceKm(lat, lng, nextSt.lat, nextSt.lng);
            const mins = Math.max(1, Math.ceil((distToNext / (speedKmh > 10 ? speedKmh : 45)) * 60));

            document.getElementById('current-station-name').innerText = `أقرب نقطة: ${STATIONS[currentStationIndex].name}`;
            triggerAnnouncement(`تم تحديد موقعك. المحطة التالية هي ${nextSt.name}، وفرق التوقيت حوالي ${formatArabicTime(mins)}.`);
        }

        function toggleSimulation() {
            if (watchId !== null) {
                navigator.geolocation.clearWatch(watchId);
                watchId = null;
            }
            if (window.liveBusSync) window.liveBusSync.isBroadcaster = true;

            const btnText = document.getElementById('sim-btn-text');
            const statusDot = document.getElementById('status-dot');
            const modeLabel = document.getElementById('gps-mode-label');

            if (isSimulating) {
                isSimulating = false;
                clearInterval(simInterval);
                btnText.innerText = "استئناف المحاكاة";
                statusDot.className = "w-3 h-3 rounded-full bg-amber-500";
                modeLabel.innerText = "المحاكاة متوقفة مؤقتاً";
            } else {
                isSimulating = true;
                btnText.innerText = "إيقاف المحاكاة";
                statusDot.className = "w-3 h-3 rounded-full bg-sky-400 animate-ping";
                modeLabel.innerText = "وضع المحاكاة النشط (بث مباشر للجميع)";

                if (simSegmentIndex === 0 && simProgress === 0) {
                    const firstNext = STATIONS[1];
                    const dist = calculateDistanceKm(STATIONS[0].lat, STATIONS[0].lng, firstNext.lat, firstNext.lng);
                    const mins = Math.max(1, Math.ceil((dist / 50) * 60));
                    triggerAnnouncement(`بدأت الرحلة من محطة أكتوبر. المحطة التالية هي ${firstNext.name}، وفرق التوقيت حوالي ${formatArabicTime(mins)}.`);
                }

                simInterval = setInterval(() => {
                    if (simSegmentIndex >= ROUTE_PATH.length - 1) {
                        toggleSimulation();
                        return;
                    }

                    const speedMultiplier = parseInt(document.getElementById('sim-speed').value, 10);
                    simProgress += 0.02 * speedMultiplier;

                    if (simProgress >= 1) {
                        simProgress = 0;
                        simSegmentIndex++;
                        if (simSegmentIndex >= ROUTE_PATH.length - 1) return;
                    }

                    const curP1 = ROUTE_PATH[simSegmentIndex];
                    const curP2 = ROUTE_PATH[simSegmentIndex + 1];
                    const lat = curP1[0] + (curP2[0] - curP1[0]) * simProgress;
                    const lng = curP1[1] + (curP2[1] - curP1[1]) * simProgress;

                    map.panTo([lat, lng], { animate: true, duration: 0.25 });
                    updateUIWithPosition(lat, lng, 55);
                }, 300);
            }
        }

        function jumpToStation(stationIndex) {
            if (window.liveBusSync) window.liveBusSync.isBroadcaster = true;
            const st = STATIONS[stationIndex];
            let closestSeg = 0;
            let minD = Infinity;
            ROUTE_PATH.forEach((pt, idx) => {
                const d = calculateDistanceKm(st.lat, st.lng, pt[0], pt[1]);
                if (d < minD) {
                    minD = d;
                    closestSeg = idx;
                }
            });

            simSegmentIndex = Math.min(closestSeg, ROUTE_PATH.length - 2);
            simProgress = 0;
            currentStationIndex = stationIndex;
            nextStationIndex = stationIndex;
            announcedApproaching = false;

            map.setView([st.lat, st.lng], 13);
            updateUIWithPosition(st.lat, st.lng, 50);
            renderStationsList();
        }

        function resetTrip() {
            if (isSimulating) toggleSimulation();
            simSegmentIndex = 0;
            simProgress = 0;
            currentStationIndex = 0;
            nextStationIndex = 1;
            announcedApproaching = false;

            const startSt = STATIONS[0];
            busPosition = { lat: startSt.lat, lng: startSt.lng };
            busMarker.setLatLng([startSt.lat, startSt.lng]);
            map.fitBounds(routePolyline.getBounds(), { padding: [30, 30] });

            document.getElementById('sim-btn-text').innerText = "محاكاة الرحلة (تجربة)";
            document.getElementById('gps-mode-label').innerText = "تمت إعادة الضبط - جاهز للانطلاق";
            document.getElementById('status-dot').className = "w-3 h-3 rounded-full bg-slate-500";
            document.getElementById('current-station-name').innerText = "نقطة الانطلاق: أكتوبر";

            updateUIWithPosition(startSt.lat, startSt.lng, 50);
            renderStationsList();
            showToast("إعادة ضبط", "تمت إعادة الأتوبيس إلى نقطة البداية (أكتوبر).");
        }

        let toastTimeout;
        function showToast(title, message) {
            const box = document.getElementById('toast-box');
            document.getElementById('toast-title').innerText = title;
            document.getElementById('toast-msg').innerText = message;
            box.classList.remove('hidden');

            clearTimeout(toastTimeout);
            toastTimeout = setTimeout(() => {
                box.classList.add('hidden');
            }, 6000);
        }

        function hideToast() {
            document.getElementById('toast-box').classList.add('hidden');
        }

        if ('speechSynthesis' in window) {
            window.speechSynthesis.getVoices();
        }
    </script>
</body>
</html>
```
