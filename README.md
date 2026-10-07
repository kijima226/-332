<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>マイ学習管理 & 単語帳システム</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: sans-serif; }
        body { 
            background-color: #f5f7fa; 
            background-size: cover;
            background-position: center;
            background-attachment: fixed;
            color: #333; 
            padding: 30px; 
            min-height: 100vh;
        }
        .container { max-width: 900px; margin: 0 auto; display: flex; flex-direction: column; gap: 25px; }
        
        /* ヘッダー */
        .header-panel { 
            background: rgba(255, 255, 255, 0.95); 
            padding: 20px; 
            border-radius: 12px; 
            display: flex; 
            align-items: center; 
            justify-content: space-between;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }
        .profile-area { display: flex; align-items: center; gap: 15px; }
        .user-icon { width: 60px; height: 60px; border-radius: 50%; border: 2px solid #1a73e8; object-fit: cover; background: #eee; }
        .user-name { font-size: 1.3rem; font-weight: bold; }
        .settings-btn { background: none; border: none; font-size: 1.8rem; cursor: pointer; padding: 5px; }
        
        /* パネル共通 */
        .panel { 
            background: rgba(255, 255, 255, 0.95); 
            padding: 25px; 
            border-radius: 12px; 
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }
        h2 { font-size: 1.3rem; margin-bottom: 15px; color: #1a73e8; border-bottom: 2px solid #1a73e8; padding-bottom: 5px; }
        
        /* 学習管理 */
        .dashboard { display: flex; gap: 20px; margin-bottom: 20px; background: #e8f0fe; padding: 15px; border-radius: 8px; }
        .stat-box { flex: 1; text-align: center; }
        .stat-val { font-size: 1.8rem; font-weight: bold; color: #1a73e8; }
        .stat-lbl { font-size: 0.85rem; color: #555; }
        .course-list { list-style: none; }
        .course-item { display: flex; align-items: center; justify-content: space-between; padding: 12px; border: 1px solid #e0e0e0; border-radius: 6px; margin-bottom: 8px; background: #fff; }
        .course-item.completed { background-color: #f0fdf4; border-color: #bbf7d0; }
        .action-btn { background: #1a73e8; color: white; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; font-weight: bold; }
        .action-btn.undo { background: #dc2626; }
        
        /* 単語帳 */
        .deck-create-area { display: flex; gap: 10px; margin-bottom: 15px; }
        .deck-input { flex: 1; padding: 8px; border: 1px solid #ccc; border-radius: 4px; }
        .primary-btn { background: #1a73e8; color: white; border: none; padding: 8px 16px; border-radius: 4px; cursor: pointer; font-weight: bold; }
        .deck-tabs { display: flex; gap: 8px; margin-bottom: 15px; overflow-x: auto; padding-bottom: 5px; }
        .deck-tab { padding: 8px 16px; border: 1px solid #ccc; background: #fff; border-radius: 20px; cursor: pointer; white-space: nowrap; }
        .deck-tab.active { background: #1a73e8; color: white; border-color: #1a73e8; }
        .card-add-area { display: flex; gap: 10px; margin-bottom: 15px; background: #f0f4f8; padding: 10px; border-radius: 6px; }
        .card-list { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .word-card { background: #fff; border: 1px solid #e0e0e0; padding: 15px; border-radius: 6px; position: relative; min-height: 80px; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; cursor: pointer; font-weight: bold; }
        .word-card .back { display: none; color: #dc2626; }
        .word-card.flipped .front { display: none; }
        .word-card.flipped .back { display: block; }
        .card-del-btn { position: absolute; top: 4px; right: 6px; background: none; border: none; color: #ccc; cursor: pointer; font-size: 0.8rem; }
        .card-del-btn:hover { color: red; }

        /* 【重要】はっきり見える設定画面（モーダル）の修正 */
        .modal { 
            display: none; 
            position: fixed; 
            top: 0; 
            left: 0; 
            width: 100%; 
            height: 100%; 
            background: rgba(0, 0, 0, 0.6) !important; /* 背景をしっかり暗くする */
            justify-content: center; 
            align-items: center; 
            z-index: 9999 !important; /* 一番手前に持ってくる */
        }
        .modal-content { 
            background: #ffffff !important; /* 完全な白色背景 */
            padding: 30px; 
            border-radius: 16px; 
            max-width: 450px; 
            width: 90%; 
            position: relative; 
            box-shadow: 0 10px 25px rgba(0,0,0,0.3); /* 影をつけて立体的に */
            color: #333333;
        }
        .close-btn { position: absolute; top: 15px; right: 20px; font-size: 1.8rem; cursor: pointer; border: none; background: none; color: #666; }
        .setting-group { margin-bottom: 18px; text-align: left; }
        .setting-group label { display: block; font-size: 0.95rem; margin-bottom: 6px; font-weight: bold; color: #333; }
        .setting-group input[type="text"], .setting-group input[type="file"] { width: 100%; padding: 10px; border: 2px solid #ddd; border-radius: 6px; font-size: 0.95rem; }
    </style>
</head>
<body>

<div class="container">
    <!-- ヘッダー -->
    <div class="header-panel">
        <div class="profile-area">
            <img id="display-icon" class="user-icon" src="data:image/svg+xml;utf8,<svg xmlns='http://w3.org' viewBox='0 0 24 24' fill='%23ccc'><path d='M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z'/></svg>" alt="Icon">
            <div id="display-name" class="user-name">受講生マイページ</div>
        </div>
        <!-- ⚙️ボタン -->
        <button class="settings-btn" onclick="openModal()">⚙️</button>
    </div>

    <!-- 学習管理 -->
    <div class="panel">
        <h2>学習進捗管理</h2>
        <div class="dashboard">
            <div class="stat-box">
                <div id="progress-rate" class="stat-val">0%</div>
                <div class="stat-lbl">全体の進捗率</div>
            </div>
            <div class="stat-box">
                <div id="completed-count" class="stat-val">0 / 3</div>
                <div class="stat-lbl">完了済みの講義数</div>
            </div>
        </div>
        <ul id="course-list" class="course-list"></ul>
    </div>

    <!-- 単語帳 -->
    <div class="panel">
        <h2>自作カスタム単語帳</h2>
        <div class="deck-create-area">
            <input type="text" id="new-deck-title" class="deck-input" placeholder="新しく追加する教科名（例：社会、理科、英語など）">
            <button class="primary-btn" onclick="createDeck()">教科を追加</button>
        </div>
        <div id="deck-tabs" class="deck-tabs"></div>
        <div id="card-input-zone" class="card-add-area" style="display: none;">
            <input type="text" id="card-front" class="deck-input" placeholder="表（単語・問題）">
            <input type="text" id="card-back" class="deck-input" placeholder="裏（意味・答え）">
            <button class="primary-btn" onclick="addCard()">カードを追加</button>
        </div>
        <div id="card-list" class="card-list"></div>
    </div>
</div>

<!-- 【ポップアップする設定画面】 -->
<div id="settings-modal" class="modal">
    <div class="modal-content">
        <button class="close-btn" onclick="closeModal()">×</button>
        <h3 style="font-size: 1.3rem; margin-bottom: 15px; color:#1a73e8;">デザイン設定</h3>
        
        <div class="setting-group">
            <label>名前の変更</label>
            <input type="text" id="input-name" placeholder="新しい名前を入力">
        </div>
        <div class="setting-group">
            <label>マイアイコン画像（写真を選ぶ）</label>
            <input type="file" id="input-icon" accept="image/*" onchange="updateIcon(this)">
        </div>
        <div class="setting-group">
            <label>ホームの背景画像（写真を選ぶ）</label>
            <input type="file" id="input-bg" accept="image/*" onchange="updateBackground(this)">
        </div>
        <button class="primary-btn" style="width:100%; padding:12px; margin-top:10px; font-size:1rem;" onclick="saveProfileSettings()">設定を保存して閉じる</button>
    </div>
</div>

<script>
    const lessonsMaster = [
        { id: "l1", title: "第1講：基礎単元（第1章）" },
        { id: "l2", title: "第2講：応用問題（第2章）" },
        { id: "l3", title: "第3講：総仕上げ確認テスト" }
    ];
    let userProgress = JSON.parse(localStorage.getItem('user_lms_progress_v5')) || {};

    let wordDecks = JSON.parse(localStorage.getItem('user_word_decks_v5')) || {
        "社会": [ { f: "本能寺の変", b: "1582年" } ],
        "英語": [ { f: "apple", b: "りんご" } ],
        "理科": [ { f: "光合成", b: "二酸化炭素と水から酸素を作る" } ]
    };
    let currentDeckName = "社会";

    function init() {
        loadProfileSettings();
        renderLMS();
        renderDecks();
    }

    function renderLMS() {
        const list = document.getElementById('course-list');
        if(!list) return;
        list.innerHTML = '';
        let done = 0;

        lessonsMaster.forEach(lesson => {
            const isDone = userProgress[lesson.id] || false;
            if (isDone) done++;

            const li = document.createElement('li');
            li.className = `course-item ${isDone ? 'completed' : ''}`;
            li.innerHTML = `
                <div>${isDone ? '✅' : '⏳'} ${lesson.title}</div>
                <button class="action-btn ${isDone ? 'undo' : ''}" onclick="toggleLMS('${lesson.id}')">
                    ${isDone ? '未完了に戻す' : '完了にする'}
                </button>
            `;
            list.appendChild(li);
        });

        document.getElementById('progress-rate').innerText = `${Math.round((done / lessonsMaster.length) * 100)}%`;
        document.getElementById('completed-count').innerText = `${done} / ${lessonsMaster.length}`;
    }

    function toggleLMS(id) {


