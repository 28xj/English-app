<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>GM 單字卡</title>
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <style>
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        
        body { 
            background: #f8f9fa; 
            margin: 0; 
            /* 關鍵修正：幫手機頂部與底部留出安全精準的呼吸空間，不和動態島打架 */
            padding: calc(env(safe-area-inset-top) + 20px) 20px calc(env(safe-area-inset-bottom) + 20px) 20px; 
            display: flex; 
            flex-direction: column; 
            align-items: center; 
            min-height: 100vh; 
            color: #1a1a1a; 
            user-select: none; 
        }
        
        .container { width: 100%; max-width: 400px; display: flex; flex-direction: column; gap: 16px; }
        .nav { display: flex; gap: 6px; background: #e9ecef; padding: 4px; border-radius: 12px; }
        .nav button { flex: 1; padding: 12px; border: none; border-radius: 8px; background: transparent; font-size: 14px; font-weight: bold; color: #6c757d; cursor: pointer; }
        .nav button.active { background: #ffffff; color: #1a1a1a; box-shadow: 0 2px 6px rgba(0,0,0,0.06); }
        .card-view, .input-view, .list-view { background: #ffffff; border-radius: 20px; padding: 24px; box-shadow: 0 8px 24px rgba(0,0,0,0.03); min-height: 440px; display: flex; flex-direction: column; justify-content: space-between; border: 1px solid #f1f3f5; position: relative; }
        .hidden { display: none !important; }
        .flashcard { flex: 1; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; gap: 12px; margin-bottom: 20px; }
        .card-row { width: 100%; padding: 16px; background: #fafafa; border-radius: 14px; border: 1px solid #eee; }
        .label-text { font-size: 10px; color: #adb5bd; text-transform: uppercase; margin-bottom: 4px; font-weight: bold; letter-spacing: 0.5px; }
        .word-main { font-size: 26px; font-weight: 800; color: #1a1a1a; word-break: break-word; }
        .word-pronounce { font-size: 18px; font-weight: bold; color: #495057; word-break: break-word; }
        .word-trans { font-size: 18px; color: #212529; word-break: break-word; }
        .actions { display: flex; gap: 12px; }
        .btn-action { flex: 1; padding: 16px; border: 2px solid #ced4da; border-radius: 16px; font-size: 16px; font-weight: bold; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: all 0.15s ease; background: #f8f9fa; color: #495057; }
        .btn-action:active { transform: scale(0.97); }
        .btn-delta { background: #e9ecef; color: #212529; border-color: #dee2e6; }
        .btn-circle { background: #1a1a1a; color: #ffffff; border-color: #1a1a1a; }
        .form-container { display: flex; flex-direction: column; gap: 12px; flex: 1; }
        .form-group { display: flex; flex-direction: column; gap: 4px; }
        .form-group label { font-size: 12px; font-weight: bold; color: #495057; }
        .form-group input { padding: 14px; border: 2px solid #e9ecef; border-radius: 12px; font-size: 15px; outline: none; color: #212529; background: #f8f9fa; transition: all 0.2s ease; }
        .form-group input:focus { border-color: #1a1a1a; background: #ffffff; }
        .btn-submit { background: #1a1a1a; color: #ffffff; border: none; padding: 16px; border-radius: 14px; font-size: 16px; font-weight: bold; cursor: pointer; }
        .stats { font-size: 11px; color: #868e96; text-align: center; font-weight: 500; }
        .list-container { flex: 1; overflow-y: auto; max-height: 280px; margin-bottom: 12px; display: flex; flex-direction: column; gap: 8px; }
        .list-item { display: flex; justify-content: space-between; align-items: center; padding: 12px; background: #f8f9fa; border-radius: 12px; border: 1px solid #e9ecef; }
        .list-info { display: flex; flex-direction: column; gap: 2px; max-width: 75%; }
        .list-word { font-weight: bold; font-size: 15px; color: #1a1a1a; }
        .list-details { font-size: 11px; color: #6c757d; }
        .btn-delete { background: #e63946; color: white; border: none; padding: 6px 12px; border-radius: 8px; font-size: 11px; font-weight: bold; cursor: pointer; }
        .backup-section { display: flex; gap: 8px; margin-top: 8px; }
        .btn-secondary { flex: 1; background: #e9ecef; color: #495057; border: none; padding: 10px; border-radius: 10px; font-size: 12px; font-weight: bold; cursor: pointer; }
        .toast-animation { position: fixed; top: 45%; left: 50%; transform: translate(-50%, -50%) scale(0.8); background: rgba(26, 26, 26, 0.95); color: #ffffff; padding: 14px 28px; border-radius: 50px; font-size: 20px; font-weight: bold; box-shadow: 0 10px 30px rgba(0,0,0,0.15); pointer-events: none; opacity: 0; white-space: nowrap; transition: all 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275); z-index: 9999; }
        .toast-animation.show { opacity: 1; transform: translate(-50%, -50%) scale(1); }
        .custom-modal { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.4); display: flex; justify-content: center; align-items: center; z-index: 10000; opacity: 0; pointer-events: none; transition: opacity 0.2s ease; padding: 20px; }
        .custom-modal.show { opacity: 1; pointer-events: auto; }
        .modal-content { background: white; border-radius: 20px; padding: 24px; width: 100%; max-width: 320px; text-align: center; box-shadow: 0 10px 30px rgba(0,0,0,0.1); transform: scale(0.9); transition: transform 0.2s ease; }
        .custom-modal.show .modal-content { transform: scale(1); }
        .modal-title { font-size: 18px; font-weight: bold; margin-bottom: 12px; }
        .modal-text { font-size: 14px; color: #6c757d; margin-bottom: 20px; line-height: 1.5; word-break: break-all; }
        .modal-buttons { display: flex; gap: 10px; }
        .modal-btn { flex: 1; padding: 12px; border-radius: 10px; font-weight: bold; font-size: 14px; cursor: pointer; border: none; }
        .modal-btn-confirm { background: #1a1a1a; color: white; }
        .modal-btn-cancel { background: #e9ecef; color: #495057; }
    </style>
</head>
<body>
<div id="toast" class="toast-animation"></div>
<div id="custom-modal" class="custom-modal"><div class="modal-content"><div class="modal-title" id="modal-title">提示</div><div class="modal-text" id="modal-text">內容</div><div class="modal-buttons" id="modal-buttons"><button class="modal-btn modal-btn-confirm" onclick="closeModal()">確認</button></div></div></div>
<div class="container">
    <div class="nav"><button id="nav-review" class="active" onclick="switchView('review')">複習單字</button><button id="nav-add" onclick="switchView('add')">新增單字</button><button id="nav-list" onclick="switchView('list')">單字清單</button></div>
    <div id="view-review" class="card-view">
        <div class="stats" id="stats-text">載入中...</div>
        <div class="flashcard">
            <div class="card-row"><div class="label-text">單字 / 句子</div><div class="word-main" id="card-word">載入中...</div></div>
            <div class="card-row"><div class="label-text">空耳唸法</div><div class="word-pronounce" id="card-pronounce">載入中...</div></div>
            <div class="card-row"><div class="label-text">中文意思</div><div class="word-trans" id="card-trans">載入中...</div></div>
        </div>
        <div class="actions"><button class="btn-action btn-delta" onclick="triggerScore('delta')">不熟</button><button class="btn-action btn-circle" onclick="triggerScore('circle')">懂了</button></div>
    </div>
    <div id="view-add" class="input-view hidden">
        <div class="form-container">
            <div class="form-group"><label>第一個框：單字/句子</label><input type="text" id="in-word" placeholder="例如: Work" autocomplete="off"></div>
            <div class="form-group"><label>第二個框：空耳唸法</label><input type="text" id="in-pronounce" placeholder="例如: 沃克" autocomplete="off"></div>
            <div class="form-group"><label>第三個框：中文意思</label><input type="text" id="in-trans" placeholder="例如: 工作" autocomplete="off"></div>
        </div>
        <button class="btn-submit" onclick="saveWord()">儲存單字</button>
    </div>
    <div id="view-list" class="list-view hidden">
        <div style="display: flex; flex-direction: column; flex: 1;"><div class="stats" style="margin-bottom: 10px; text-align: left;" id="list-count-text">目前無單字</div><div class="list-container" id="list-container"></div></div>
        <div class="backup-section"><button class="btn-secondary" onclick="exportData()">備份數據</button><button class="btn-secondary" onclick="importData()">匯入數據</button></div>
    </div>
</div>
<script>
    const defaultWords = [{ id: 1, word: "Work", pronounce: "沃克", trans: "工作", status: "new" }];
    let words = JSON.parse(localStorage.getItem('gm_words')) || defaultWords;
    let pool = []; let currentWord = null; let confirmCallback = null;
    function showCustomAlert(title, text) { document.getElementById('modal-title').innerText = title; document.getElementById('modal-text').innerText = text; document.getElementById('modal-buttons').innerHTML = `<button class="modal-btn modal-btn-confirm" onclick="closeModal()">好</button>`; document.getElementById('custom-modal').classList.add('show'); }
    function showCustomConfirm(title, text, callback) { document.getElementById('modal-title').innerText = title; document.getElementById('modal-text').innerText = text; confirmCallback = callback; document.getElementById('modal-buttons').innerHTML = `<button class="modal-btn modal-btn-cancel" onclick="closeModal()">取消</button><button class="modal-btn modal-btn-confirm" onclick="handleConfirm()">確認</button>`; document.getElementById('custom-modal').classList.add('show'); }
    function closeModal() { document.getElementById('custom-modal').classList.remove('show'); }
    function handleConfirm() { closeModal(); if (confirmCallback) confirmCallback(); }
    function saveToStorage() { localStorage.setItem('gm_words', JSON.stringify(words)); }
    function switchView(view) {
        document.getElementById('nav-review').classList.remove('active'); document.getElementById('nav-add').classList.remove('active'); document.getElementById('nav-list').classList.remove('active');
        document.getElementById('view-review').classList.add('hidden'); document.getElementById('view-add').classList.add('hidden'); document.getElementById('view-list').classList.add('hidden');
        if (view === 'review') { document.getElementById('view-review').classList.remove('hidden'); document.getElementById('nav-review').classList.add('active'); buildPool(); nextCard(); }
        else if (view === 'add') { document.getElementById('view-add').classList.remove('hidden'); document.getElementById('nav-add').classList.add('active'); }
        else if (view === 'list') { document.getElementById('view-list').classList.remove('hidden'); document.getElementById('nav-list').classList.add('active'); renderList(); }
    }
    function buildPool() {
        let circles = words.filter(w => w.status === 'circle'); let deltas = words.filter(w => w.status === 'delta'); let news = words.filter(w => w.status === 'new');
        pool = [];
        deltas.forEach(w => { for(let i = 0; i < 4; i++) pool.push({...w}); });
        news.forEach(w => { pool.push({...w}); });
        if (pool.length === 0 && circles.length > 0) { circles.forEach(w => { w.status = 'new'; pool.push({...w}); }); saveToStorage(); }
        if (pool.length === 0 && words.length > 0) { words.forEach(w => { w.status = 'new'; pool.push({...w}); }); saveToStorage(); }
        pool.sort(() => Math.random() - 0.5); updateStats();
    }
    function updateStats() { document.getElementById('stats-text').innerText = `總單字量: ${words.length} | 本輪剩餘: ${pool.length} 張`; }
    function nextCard() {
        if (pool.length === 0) buildPool();
        if (pool.length === 0) { document.getElementById('card-word').innerText = "目前沒有單字，快去新增吧！"; document.getElementById('card-pronounce').innerText = "-"; document.getElementById('card-trans').innerText = "-"; currentWord = null; return; }
        currentWord = pool.shift(); document.getElementById('card-word').innerText = currentWord.word; document.getElementById('card-pronounce').innerText = currentWord.pronounce; document.getElementById('card-trans').innerText = currentWord.trans; updateStats();
    }
    function triggerScore(type) {
        if (!currentWord) return;
        const toast = document.getElementById('toast'); toast.innerText = type === 'circle' ? "你太棒了！" : "加油！你可以的"; toast.classList.add('show'); setTimeout(() => toast.classList.remove('show'), 800);
        let target = words.find(w => w.id === currentWord.id); if (target) { target.status = type; saveToStorage(); }
        nextCard();
    }
    function saveWord() {
        const word = document.getElementById('in-word').value.trim(); const pronounce = document.getElementById('in-pronounce').value.trim(); const trans = document.getElementById('in-trans').value.trim();
        if (!word || !pronounce || !trans) { showCustomAlert('提示', '三個框框都要填滿唷！'); return; }
        const newW = { id: Date.now(), word: word, pronounce: pronounce, trans: trans, status: 'new' };
        words.push(newW); saveToStorage();
        document.getElementById('in-word').value = ''; document.getElementById('in-pronounce').value = ''; document.getElementById('in-trans').value = '';
        const toast = document.getElementById('toast'); toast.innerText = "新增成功！"; toast.classList.add('show'); setTimeout(() => toast.classList.remove('show'), 800);
        switchView('review');
    }
    function renderList() {
        const container = document.getElementById('list-container'); const countText = document.getElementById('list-count-text'); container.innerHTML = '';
        countText.innerText = `目前單字庫共有: ${words.length} 個單字`;
        if (words.length === 0) { container.innerHTML = '<div style="color: #999; text-align: center; padding: 20px; font-size: 13px;">目前清單是空的</div>'; return; }
        const displayList = [...words].reverse();
        displayList.forEach(w => {
            const item = document.createElement('div'); item.className = 'list-item';
            let statusBadge = w.status === 'circle' ? '● 懂了' : (w.status === 'delta' ? '▲ 不熟' : '新加入');
            item.innerHTML = `<div class="list-info"><div class="list-word">${w.word}</div><div class="list-details">唸法: ${w.pronounce} | 中文: ${
