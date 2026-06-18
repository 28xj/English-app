<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>GM 的極簡單字卡</title>
    <style>
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background: #f0f0f0; margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center; min-height: 100vh; color: #333; position: relative; overflow-x: hidden; }
        .container { width: 100%; max-width: 400px; display: flex; flex-direction: column; gap: 20px; z-index: 1; }
        .nav { display: flex; gap: 10px; }
        .nav button { flex: 1; padding: 12px; border: 2px solid #ccc; border-radius: 12px; background: white; font-size: 16px; font-weight: bold; color: #555; cursor: pointer; }
        .nav button.active { background: #333; color: white; border-color: #333; }
        .card-view, .input-view { background: white; border-radius: 24px; padding: 25px; box-shadow: 0 4px 12px rgba(0,0,0,0.05); min-height: 420px; display: flex; flex-direction: column; justify-content: space-between; border: 1px solid #eee; position: relative; }
        .hidden { display: none !important; }
        
        /* 複習字卡排版 */
        .flashcard { flex: 1; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; gap: 12px; margin-bottom: 20px; }
        .card-row { width: 100%; padding: 12px; background: #fafafa; border-radius: 10px; border: 1px solid #eaeaea; }
        .label-text { font-size: 11px; color: #999; text-transform: uppercase; margin-bottom: 4px; font-weight: bold; }
        .word-present { font-size: 26px; font-weight: bold; color: #1a1a1a; word-break: break-word; }
        .word-pronounce { font-size: 20px; font-weight: bold; color: #555; word-break: break-word; }
        .word-trans { font-size: 20px; color: #444; word-break: break-word; }
        
        /* 按鈕 */
        .actions { display: flex; gap: 15px; }
        .btn-action { flex: 1; padding: 15px; border: 2px solid #ccc; border-radius: 16px; font-size: 18px; font-weight: bold; cursor: pointer; display: flex; align-items: center; justify-content: center; }
        .btn-delta { background: #eee; color: #333; border-color: #ccc; }
        .btn-circle { background: #fff; color: #333; border-color: #333; }
        
        /* 表單 */
        .form-group { display: flex; flex-direction: column; gap: 8px; margin-bottom: 15px; }
        .form-group label { font-size: 14px; font-weight: bold; color: #555; }
        .form-group input { padding: 12px; border: 2px solid #ddd; border-radius: 12px; font-size: 16px; outline: none; color: #333; }
        .form-group input:focus { border-color: #999; }
        .btn-submit { background: #333; color: white; border: none; padding: 15px; border-radius: 12px; font-size: 16px; font-weight: bold; cursor: pointer; width: 100%; }
        .stats { font-size: 12px; color: #888; text-align: center; }

        /* 動態彈出動畫樣式 */
        .toast-animation {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-250%, -50%) scale(0.5);
            background: rgba(30, 30, 30, 0.95);
            color: #fff;
            padding: 18px 30px;
            border-radius: 50px;
            font-size: 22px;
            font-weight: bold;
            box-shadow: 0 10px 25px rgba(0,0,0,0.2);
            pointer-events: none;
            opacity: 0;
            white-space: nowrap;
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            z-index: 999;
        }
        .toast-animation.show {
            opacity: 1;
            transform: translate(-50%, -50%) scale(1);
        }
    </style>
</head>
<body>

<!-- 動態文字彈出層 -->
<div id="toast" class="toast-animation"></div>

<div class="container">
    <div class="nav">
        <button id="nav-review" class="active" onclick="switchView('review')">複習單字</button>
        <button id="nav-add" onclick="switchView('add')">新增單字</button>
    </div>

    <!-- 複習模式 -->
    <div id="view-review" class="card-view">
        <div class="stats" id="stats-text">剩餘單字載入中...</div>
        <div class="flashcard">
            <!-- 單字/句子格子 -->
            <div class="card-row">
                <div class="label-text">單字 / 句子</div>
                <div class="word-present" id="card-word">載入中...</div>
            </div>
            <!-- 空耳唸法 -->
            <div class="card-row">
                <div class="label-text">空耳唸法</div>
                <div class="word-pronounce" id="card-pronounce">載入中...</div>
            </div>
            <!-- 中文意思格子 -->
            <div class="card-row">
                <div class="label-text">中文意思</div>
                <div class="word-trans" id="card-trans">載入中...</div>
            </div>
        </div>
        <div class="actions">
            <button class="btn-action btn-delta" onclick="triggerScore('delta')">不熟</button>
            <button class="btn-action btn-circle" onclick="triggerScore('circle')">懂了</button>
        </div>
    </div>

    <!-- 輸入模式 -->
    <div id="view-add" class="input-view hidden">
        <div style="flex: 1;">
            <div class="form-group">
                <label>第一個框：單字/句子</label>
                <input type="text" id="in-word" placeholder="請輸入單字或句子">
            </div>
            <div class="form-group">
                <label>第二個框：空耳唸法</label>
                <input type="text" id="in-pronounce" placeholder="請輸入空耳諧音唸法">
            </div>
            <div class="form-group">
                <label>第三個框：中文意思</label>
                <input type="text" id="in-trans" placeholder="請輸入中文意思">
            </div>
        </div>
        <button class="btn-submit" onclick="saveWord()">儲存單字</button>
    </div>
</div>

<script>
    let words = JSON.parse(localStorage.getItem('gm_words')) || [];
    let pool = [];
    let currentWord = null;

    function saveToStorage() {
        localStorage.setItem('gm_words', JSON.stringify(words));
    }

    function switchView(view) {
        if (view === 'review') {
            document.getElementById('view-review').classList.remove('hidden');
            document.getElementById('view-add').classList.add('hidden');
            document.getElementById('nav-review').classList.add('active');
            document.getElementById('nav-add').classList.remove('active');
            buildPool();
            nextCard();
        } else {
            document.getElementById('view-review').classList.add('hidden');
            document.getElementById('view-add').classList.remove('hidden');
            document.getElementById('nav-review').classList.remove('active');
            document.getElementById('nav-add').classList.add('active');
        }
    }

    function buildPool() {
        let circles = words.filter(w => w.status === 'circle');
        let deltas = words.filter(w => w.status === 'delta');
        let news = words.filter(w => w.status === 'new');

        pool = [];

        deltas.forEach(w => {
            for(let i=0; i<3; i++) pool.push({...w});
        });

        news.forEach(w => pool.push({...w}));

        if (pool.length === 0 && circles.length > 0) {
            circles.forEach(w => {
                w.status = 'new';
                pool.push({...w});
            });
            saveToStorage();
        }
        
        if(pool.length === 0 && words.length > 0){
            words.forEach(w => w.status = 'new');
            words.forEach(w => pool.push({...w}));
            saveToStorage();
        }

        pool.sort(() => Math.random() - 0.5);
        updateStats();
    }

    function updateStats() {
        document.getElementById('stats-text').innerText = `總單字量: ${words.length} | 本輪剩餘: ${pool.length} 張`;
    }

    function nextCard() {
        if (pool.length === 0) {
            buildPool();
        }

        if (pool.length === 0) {
            document.getElementById('card-word').innerText = "目前沒有單字，快去新增吧！";
            document.getElementById('card-pronounce').innerText = "";
            document.getElementById('card-trans').innerText = "";
            currentWord = null;
            return;
        }

        currentWord = pool.shift();
        
        document.getElementById('card-word').innerText = currentWord.word;
        document.getElementById('card-pronounce').innerText = currentWord.pronounce || "未填寫";
        document.getElementById('card-trans').innerText = currentWord.trans;
        
        updateStats();
    }

    // 處理分數並觸發文字動畫
    function triggerScore(type) {
        if (!currentWord) return;

        const toast = document.getElementById('toast');
        if (type === 'circle') {
            toast.innerText = "你太棒了！";
        } else {
            toast.innerText = "加油！你可以的";
        }
        
        // 顯示動畫
        toast.classList.add('show');
        
        // 0.8秒後自動隱藏
        setTimeout(() => {
            toast.classList.remove('show');
        }, 800);

        // 改狀態並跳下一題
        let target = words.find(w => w.id === currentWord.id);
        if (target) {
            target.status = type;
            saveToStorage();
        }

        nextCard();
    }

    function saveWord() {
        const word = document.getElementById('in-word').value.trim();
        const pronounce = document.getElementById('in-pronounce').value.trim();
        const trans = document.getElementById('in-trans').value.trim();

        if (!word || !pronounce || !trans) {
            alert('三個框框都是必填的喔！');
            return;
        }

        const newW = {
            id: Date.now(),
            word: word,
            pronounce: pronounce,
            trans: trans,
            status: 'new'
        };

        words.push(newW);
        saveToStorage();

        document.getElementById('in-word').value = '';
        document.getElementById('in-pronounce').value = '';
        document.getElementById('in-trans').value = '';

        alert('新增成功！');
        switchView('review');
    }

    switchView('review');
</script>

</body>
</html>
