<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>সুপার মাইনার অ্যাপ</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
    <!-- Telegram Mini App SDK যোগ করা হয়েছে -->
    <script src="https://telegram.org/js/telegram-web-app.js"></script> 
    <style>
        body {
            font-family: 'Inter', sans-serif;
            /* থিম ডেটা লোড হওয়ার পর বডি কালার জাভাস্ক্রিপ্ট দ্বারা সেট করা হবে */
            background-color: #1a1a2e; 
            color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 0;
            overflow-x: hidden;
            transition: background-color 0.3s;
        }
        .container {
            width: 100%;
            max-width: 480px; 
            background-color: #0f0f1d; /* ডিফল্ট কন্টেইনার কালার */
            border-radius: 1.5rem;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            padding-bottom: 20px;
        }
        /* ট্যাপ করার কয়েন স্টাইল */
        #mine-button {
            transition: transform 0.1s ease-out;
            cursor: pointer;
            box-shadow: 0 0 50px rgba(255, 255, 0, 0.4), 0 0 20px rgba(255, 165, 0, 0.8);
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            border: none;
            outline: none;
        }
        #mine-button:active:not(:disabled) {
            transform: scale(0.95);
        }
        /* Disabled state for button */
        #mine-button:disabled {
            cursor: not-allowed;
            opacity: 0.5;
            box-shadow: none;
        }
        /* ফ্লোটিং টেক্সট স্টাইল */
        .floating-text {
            position: absolute;
            font-size: 2rem;
            font-weight: 800;
            color: #ffcc00;
            pointer-events: none;
            animation: floatUp 1s ease-out forwards;
            opacity: 0;
            z-index: 10; 
        }
        @keyframes floatUp {
            0% { transform: translateY(0) scale(1); opacity: 1; }
            100% { transform: translateY(-100px) scale(0.8); opacity: 0; }
        }
        .upgrade-btn {
            background: linear-gradient(145deg, #ffcc00, #ffa343);
            color: #1a1a2e;
            font-weight: 700;
            box-shadow: 0 4px 15px rgba(255, 165, 0, 0.5);
            transition: all 0.2s;
        }
        .upgrade-btn:disabled {
            background: #4b5563; /* ডিজেবল হলে ধূসর রং */
            box-shadow: none;
            cursor: not-allowed;
            color: #9ca3af;
        }
        .upgrade-btn:hover:not(:disabled) {
            box-shadow: 0 6px 20px rgba(255, 165, 0, 0.7);
            transform: translateY(-2px);
        }
    </style>
</head>
<body>
    <div class="container">
        <header class="p-6 text-center">
            <h1 class="text-3xl font-bold text-yellow-400">সুপার মাইনার</h1>
            <p id="auth-status" class="text-sm text-gray-400 mt-1">অ্যাক্সেস অনুমোদিত হচ্ছে...</p>
        </header>

        <!-- স্কোরবোর্ড -->
        <div class="text-center p-4">
            <p class="text-2xl font-semibold text-gray-300">মোট কয়েন</p>
            <p id="coin-count" class="text-6xl font-extrabold text-white mb-2">0</p>
            <p class="text-md font-medium text-green-400">
                <span id="tap-power-display">১</span> ট্যাপ/ক্লিক |
                <span id="passive-rate-display">০</span> কয়েন/সেকেন্ড
            </p>
        </div>

        <!-- মাইনিং এরিয়া -->
        <div class="relative flex justify-center items-center py-10" id="mining-area">
            <!-- Changed to a semantic button for accessibility & proper disabled support -->
            <button id="mine-button" type="button" aria-label="মাইন করুন" class="w-48 h-48 bg-yellow-400 rounded-full flex justify-center items-center transform transition duration-150 ease-out" disabled>
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-24 h-24 text-[#0f0f1d]">
                    <path fill-rule="evenodd" d="M12 2.25c-5.385 0-9.75 4.365-9.75 9.75s4.365 9.75 9.75 9.75 9.75-4.365 9.75-9.75S17.385 2.25 12 2.25ZM12.75 6a.75.75 0 0 0-1.5 0v6c0 .414.336.75.75.75h4.5a.75.75 0 0 0 0-1.5h-3.75V6Z" clip-rule="evenodd" />
                </svg>
            </button>
        </div>

        <!-- আপগ্রেড সেকশন -->
        <div class="p-4 space-y-4">
            <h2 class="text-xl font-bold text-yellow-400 border-b border-gray-700 pb-2">আপগ্রেড সেন্টার</h2>

            <!-- ট্যাপ পাওয়ার আপগ্রেড -->
            <div class="bg-[#1a1a2e] p-3 rounded-xl flex justify-between items-center shadow-md">
                <div>
                    <p class="font-bold text-lg">ট্যাপ পাওয়ার বাড়ান</p>
                    <p class="text-sm text-gray-400">প্রতি ক্লিকে আরও কয়েন</p>
                </div>
                <button id="upgrade-tap-btn" class="upgrade-btn px-4 py-2 rounded-lg text-sm" onclick="buyUpgrade('tap')">
                    কিনে নিন: <span id="tap-cost">১০০</span>
                </button>
            </div>

            <!-- প্যাসিভ রেট আপগ্রেড -->
            <div class="bg-[#1a1a2e] p-3 rounded-xl flex justify-between items-center shadow-md">
                <div>
                    <p class="font-bold text-lg">প্যাসিভ মাইনিং বাড়ান</p>
                    <p class="text-sm text-gray-400">অ্যাপ বন্ধ থাকলেও ইনকাম</p>
                </div>
                <button id="upgrade-passive-btn" class="upgrade-btn px-4 py-2 rounded-lg text-sm" onclick="buyUpgrade('passive')">
                    কিনে নিন: <span id="passive-cost">৫০০</span>
                </button>
            </div>

            <div id="status-message" class="text-center text-sm mt-4 hidden p-2 rounded-lg bg-red-800 text-white"></div>
        </div>

        <!-- ইউজার আইডি ডিসপ্লে -->
        <footer class="p-4 text-center text-xs text-gray-500 border-t border-gray-700 mt-4">
            <p>আপনার ইউজার আইডি (শেয়ার করতে পারেন): <span id="user-id-display" class="font-mono text-yellow-500" aria-live="polite">Loading...</span></p>
            <p id="tma-status" class="text-xs text-blue-400 mt-1">TMA SDK: সংযোগ হচ্ছে...</p>
        </footer>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        // Firebase মডিউল আমদানি
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, onSnapshot, setDoc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // ফায়ারবেস কনফিগারেশন এবং গ্লোবাল ভ্যারিয়েবলস
        setLogLevel && setLogLevel('Debug');
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let db, auth;
        let userId = 'anon';
        let isAuthReady = false;
        let gameDataRef = null;
        let lastSaveAt = 0; // timestamp to throttle saves (ms)

        // গেম স্টেট
        window.gameState = {
            coins: 0,
            tapPower: 1,
            passiveRate: 0, // কয়েন প্রতি সেকেন্ড
            lastUpdateTime: Date.now()
        };

        const BASE_COSTS = {
            tap: 100,
            passive: 500
        };

        // DOM এলিমেন্ট
        const coinCountEl = document.getElementById('coin-count');
        const tapPowerEl = document.getElementById('tap-power-display');
        const passiveRateEl = document.getElementById('passive-rate-display');
        const userIdDisplayEl = document.getElementById('user-id-display');
        const authStatusEl = document.getElementById('auth-status');
        const tapCostEl = document.getElementById('tap-cost');
        const passiveCostEl = document.getElementById('passive-cost');
        const mineButton = document.getElementById('mine-button');
        const miningArea = document.getElementById('mining-area');
        const tmaStatusEl = document.getElementById('tma-status');

        /* TMA স্পেসিফিক ফাংশনালিটি */

        function initTelegramWebApp() {
            if (window.Telegram && window.Telegram.WebApp) {
                const WebApp = window.Telegram.WebApp;
                WebApp.ready();
                tmaStatusEl.textContent = "TMA SDK: প্রস্তুত";

                // টেলিগ্রাম থিম কালার ব্যবহার করে UI আপডেট করা
                document.body.style.backgroundColor = WebApp.themeParams.bg_color || '#1a1a2e';
                document.querySelector('.container').style.backgroundColor = WebApp.themeParams.secondary_bg_color || '#0f0f1d';
                
                // TMA চলাকালীন ব্যবহারকারী যখন অ্যাপ থেকে বেরিয়ে যাবে, তখন ডেটা সেভ করা নিশ্চিত করা
                WebApp.onEvent('mainButtonClicked', () => saveGameData(window.gameState, { immediate: true }));
                WebApp.onEvent('viewportChanged', () => saveGameData(window.gameState));
                
            } else {
                tmaStatusEl.textContent = "TMA SDK: লোড হয়নি (সাধারণ ব্রাউজার)";
            }
        }

        // ফাংশন: ফ্লোটিং টেক্সট ডিসপ্লে
        function showFloatingText(amount, x, y) {
            const el = document.createElement('div');
            el.className = 'floating-text';
            el.textContent = `+${amount}`;
            
            // ক্লিক বা টাচের অবস্থানের কাছাকাছি ফ্লোটিং টেক্সট দেখানো
            const rect = miningArea.getBoundingClientRect();
            el.style.left = `${x - rect.left - 20}px`; 
            el.style.top = `${y - rect.top - 40}px`; 
            miningArea.appendChild(el);

            // অ্যানিমেশন শেষ হলে এলিমেন্ট সরিয়ে ফেলা
            el.addEventListener('animationend', () => el.remove());
        }

        // ফাংশন: গেম স্টেট আপডেট করা
        function updateUI() {
            coinCountEl.textContent = Math.floor(window.gameState.coins).toLocaleString('bn-IN');
            tapPowerEl.textContent = window.gameState.tapPower.toLocaleString('bn-IN');
            passiveRateEl.textContent = window.gameState.passiveRate.toLocaleString('bn-IN');

            // Tap Power Cost Calculation: Doubling cost for each level (1, 2, 4, 8, ...)
            const tapLevel = Math.log2(window.gameState.tapPower) || 0; 
            const nextTapCost = BASE_COSTS.tap * Math.pow(2, tapLevel);
            
            // Passive Rate Cost Calculation: Linear increase (500, 1000, 1500, ...)
            const passiveLevel = window.gameState.passiveRate; 
            const nextPassiveCost = BASE_COSTS.passive * (passiveLevel + 1); 

            tapCostEl.textContent = nextTapCost.toLocaleString('bn-IN');
            passiveCostEl.textContent = nextPassiveCost.toLocaleString('bn-IN');

            // বাটন ডিসেবল/এনাবল করা
            document.getElementById('upgrade-tap-btn').disabled = window.gameState.coins < nextTapCost;
            document.getElementById('upgrade-passive-btn').disabled = window.gameState.coins < nextPassiveCost;
        }

        // ফাংশন: কয়েন ট্যাপ
        function handleTap(event) {
            if (!isAuthReady) return;

            window.gameState.coins += window.gameState.tapPower;

            let clickX, clickY;

            if (event.touches && event.touches.length > 0) {
                // টাচ ইভেন্টের জন্য
                clickX = event.touches[0].clientX;
                clickY = event.touches[0].clientY;
            } else {
                // মাউস ইভেন্টের জন্য
                clickX = event.clientX;
                clickY = event.clientY;
            }

            // কয়েনের উপরে ফ্লোটিং টেক্সট দেখানো
            showFloatingText(window.gameState.tapPower, clickX, clickY);

            updateUI();
            saveGameData(window.gameState); 
        }

        // টাচ এবং মাউস ইভেন্ট যুক্ত করা
        mineButton.addEventListener('click', handleTap);
        mineButton.addEventListener('touchstart', (e) => {
            e.preventDefault(); 
            handleTap(e);
        });

        // ফাংশন: প্যাসিভ মাইনিং
        function passiveMining() {
            if (!isAuthReady) return;

            const currentTime = Date.now();
            // গত আপডেটের সময় থেকে কত সেকেন্ড অতিবাহিত হয়েছে
            const elapsedTime = (currentTime - window.gameState.lastUpdateTime) / 1000; 

            if (elapsedTime > 0) {
                const minedCoins = window.gameState.passiveRate * elapsedTime;
                window.gameState.coins += minedCoins;
                window.gameState.lastUpdateTime = currentTime;
                
                if (minedCoins >= 1) { 
                    updateUI();
                    // প্রতি 10 সেকেন্ডে বা যখন উল্লেখযোগ্য পরিবর্তন হবে তখনই সেভ করা
                    if (elapsedTime > 10) saveGameData(window.gameState);
                }
            }
        }
        
        // প্রতি সেকেন্ডে প্যাসিভ মাইনিং চেক করা
        setInterval(passiveMining, 1000); 

        // ফাংশন: আপগ্রেড কেনা
        window.buyUpgrade = async (type) => {
            if (!isAuthReady) {
                showMessage("অনুগ্রহ করে লগইন হওয়ার জন্য অপেক্ষা করুন।", true);
                return;
            }

            let cost = 0;
            if (type === 'tap') {
                const tapLevel = Math.log2(window.gameState.tapPower) || 0;
                cost = BASE_COSTS.tap * Math.pow(2, tapLevel);
            } else if (type === 'passive') {
                const passiveLevel = window.gameState.passiveRate;
                cost = BASE_COSTS.passive * (passiveLevel + 1);
            }

            if (window.gameState.coins >= cost) {
                window.gameState.coins -= cost;
                
                if (type === 'tap') {
                    window.gameState.tapPower *= 2; // ট্যাপ পাওয়ার দ্বিগুণ
                    showMessage("ট্যাপ পাওয়ার আপগ্রেড সফল!", false);
                } else if (type === 'passive') {
                    window.gameState.passiveRate += 1; // প্যাসিভ রেট প্রতি সেকেন্ডে ১ কয়েন বৃদ্ধি
                    showMessage("প্যাসিভ মাইনিং আপগ্রেড সফল!", false);
                }

                updateUI();
                await saveGameData(window.gameState, { immediate: true });
            } else {
                showMessage("কয়েন যথেষ্ট নয়!", true);
                
                // TMA ফিডব্যাক (যখন যথেষ্ট কয়েন না থাকে)
                if(window.Telegram && window.Telegram.WebApp) {
                    window.Telegram.WebApp.HapticFeedback.notificationOccurred('error');
                }
            }
        };

        // ফাংশন: মেসেজ দেখানো
        function showMessage(message, isError) {
            const statusEl = document.getElementById('status-message');
            statusEl.textContent = message;
            statusEl.classList.remove('hidden', 'bg-red-800', 'bg-green-800');
            statusEl.classList.add(isError ? 'bg-red-800' : 'bg-green-800');
            
            setTimeout(() => {
                statusEl.classList.add('hidden');
            }, 3000);
        }

        // ফাংশন: Firestore-এ ডেটা লোড করা
        function loadGameData() {
            if (!gameDataRef) return;
            
            authStatusEl.textContent = `ইউজার: ${userId}`;
            userIdDisplayEl.textContent = userId;

            // onSnapshot দিয়ে রিয়েল-টাইম লিসেনার সেট করা
            const unsubscribe = onSnapshot(gameDataRef, (docSnap) => {
                if (docSnap.exists()) {
                    const savedData = docSnap.data();
                    console.log("Firestore ডেটা লোড করা হয়েছে:", savedData);
                    
                    // ডেটা লোড করে গেম স্টেট আপডেট করা
                    window.gameState = {
                        coins: savedData.coins || 0,
                        tapPower: savedData.tapPower || 1,
                        passiveRate: savedData.passiveRate || 0,
                        // Firestore Timestamp কে JavaScript Date এ রূপান্তর করা
                        lastUpdateTime: savedData.lastUpdateTime instanceof Date ? savedData.lastUpdateTime.getTime() : savedData.lastUpdateTime || Date.now()
                    };

                    // অফলাইন প্যাসিভ ইনকাম ক্যালকুলেশন
                    passiveMining(); 
                } else {
                    console.log("Firestore-এ কোনো গেম ডেটা পাওয়া যায়নি। নতুন গেম শুরু হচ্ছে।");
                    saveGameData(window.gameState, { immediate: true });
                }
                updateUI();
                isAuthReady = true;
                mineButton.disabled = false;
            }, (error) => {
                console.error("Firestore লিসেনার ত্রুটি:", error);
                showMessage("ডেটা লোড করতে সমস্যা হচ্ছে। ইন্টারনেট চেক করুন।", true);
            });
        }

        // ফাংশন: Firestore-এ ডেটা সেভ করা (থ্রটল করা: সর্বোচ্চ 1 save প্রতি 2 সেকেন্ড)
        async function saveGameData(data, options = { immediate: false }) {
            if (!gameDataRef || !isAuthReady) return;
            const now = Date.now();

            // যদি immediate=true করা হয়, তখন অবিলম্বে সেভ কর
            if (!options.immediate && (now - lastSaveAt) < 2000) {
                // থ্রটলিং ইফেক্ট: খুব ঘন সেভ কল এড়ানো
                return;
            }

            try {
                await setDoc(gameDataRef, {
                    coins: data.coins,
                    tapPower: data.tapPower,
                    passiveRate: data.passiveRate,
                    lastUpdateTime: Date.now() 
                }, { merge: true });
                lastSaveAt = Date.now();
            } catch (e) {
                console.error("ডেটা সেভ করতে ত্রুটি:", e);
                // Mini App এ একটি সতর্কতা দেখানো
                if (window.Telegram && window.Telegram.WebApp) {
                    window.Telegram.WebApp.showAlert("ডেটা সেভ করতে ব্যর্থ! ইন্টারনেট চেক করুন।");
                }
            }
        }

        // ফাংশন: ফায়ারবেস ইনিশিয়ালাইজেশন ও অথেন্টিকেশন
        async function initializeFirebase() {
            if (Object.keys(firebaseConfig).length === 0) {
                authStatusEl.textContent = "Firebase কনফিগারেশন অনুপস্থিত। ডেটা সেভ হবে না।";
                isAuthReady = true;
                mineButton.disabled = false;
                userIdDisplayEl.textContent = "স্থানীয় মোড";
                return;
            }

            const app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);

            mineButton.disabled = true;

            try {
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                    authStatusEl.textContent = "ব্যবহারকারী অনুমোদিত।";
                } else {
                    await signInAnonymously(auth);
                    authStatusEl.textContent = "নামবিহীনভাবে লগইন করা হয়েছে।";
                }

                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        // ব্যক্তিগত ডেটা সংরক্ষণের জন্য পাথ: /artifacts/{appId}/users/{userId}/game_data/hamster_mine
                        gameDataRef = doc(db, "artifacts", appId, "users", userId, "game_data", "hamster_mine");
                        loadGameData();
                    } else {
                        console.log("অথেন্টিকেশন ব্যর্থ বা ইউজার লগআউট করেছেন।");
                        userIdDisplayEl.textContent = "লগইন প্রয়োজন";
                        authStatusEl.textContent = "লগইন ব্যর্থ।";
                    }
                });

            } catch (error) {
                console.error("অথেন্টিকেশনে ত্রুটি:", error);
                authStatusEl.textContent = "লগইন করতে সমস্যা হয়েছে।";
                showMessage(`লগইন ত্রুটি: ${error.message}`, true);
            }
        }
        
        // পেজ লোড হলে Firebase এবং TMA SDK উভয়ই শুরু করা হবে
        window.onload = () => {
            initTelegramWebApp();
            initializeFirebase();
        };
    </script>
</body>
</html>
