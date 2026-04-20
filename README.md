<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WordMaster - 全方位管理版</title>
    <style>
        :root { --primary: #4a90e2; --success: #34c759; --danger: #ff3b30; --bg: #f8f9fa; --card-bg: #ffffff; }
        body { font-family: -apple-system, sans-serif; background-color: var(--bg); display: flex; flex-direction: column; align-items: center; min-height: 100vh; padding: 40px 20px; color: #333; margin: 0; box-sizing: border-box; }
        .hidden { display: none !important; }
        
        #login-interface, #study-interface, #quiz-interface, #result-interface { width: 100%; max-width: 360px; display: flex; flex-direction: column; align-items: center; text-align: center; }

        .card { width: 100%; min-height: 280px; background: var(--card-bg); border-radius: 24px; box-shadow: 0 10px 30px rgba(0,0,0,0.08); display: flex; flex-direction: column; justify-content: center; align-items: center; padding: 25px; position: relative; box-sizing: border-box; cursor: pointer; transition: 0.3s; }
        
        input, select, textarea { width: 100%; padding: 12px; border-radius: 10px; border: 1px solid #ddd; box-sizing: border-box; margin-bottom: 12px; font-size: 1rem; font-family: inherit; }
        .primary-btn { padding: 12px 30px; background: var(--primary); color: white; border: none; border-radius: 50px; cursor: pointer; font-weight: bold; width: 100%; margin-top: 5px; }
        .btn-circle { width: 60px; height: 60px; border-radius: 50%; border: none; background: white; box-shadow: 0 4px 10px rgba(0,0,0,0.1); cursor: pointer; font-size: 20px; display: flex; align-items: center; justify-content: center; }
        .chip { padding: 8px 18px; border-radius: 20px; background: #e0e0e0; cursor: pointer; font-size: 14px; border: none; margin: 4px; }
        .chip.active { background: var(--primary); color: white; }

        .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.4); justify-content: center; align-items: center; z-index: 100; backdrop-filter: blur(4px); }
        .modal-content { background: white; padding: 25px; border-radius: 24px; width: 320px; box-sizing: border-box; max-height: 90vh; overflow-y: auto; text-align: left; }
        
        .opt-btn { width: 100%; padding: 12px; border-radius: 12px; border: 1px solid #ddd; background: white; cursor: pointer; margin-bottom: 10px; }
        .opt-btn.correct { background: var(--success) !important; color: white; }
        .opt-btn.wrong { background: var(--danger) !important; color: white; }

        #toast { visibility: hidden; background: #333; color: #fff; padding: 12px 24px; border-radius: 50px; position: fixed; bottom: 50px; left: 50%; transform: translateX(-50%); opacity: 0; transition: 0.3s; z-index: 2000; }
        #toast.show { visibility: visible; opacity: 1; bottom: 70px; }
        
        .list-item-row { display: flex; justify-content: space-between; align-items: center; padding: 8px 12px; background: #f9f9f9; border-radius: 10px; margin-bottom: 8px; font-size: 0.9rem; }
        .del-list-btn { color: var(--danger); cursor: pointer; border: none; background: none; font-size: 1.1rem; }
    </style>
</head>
<body>

    <div id="login-interface">
        <h2 style="color: var(--primary);">WordMaster</h2>
        <input type="text" id="user-name-input" placeholder="請輸入使用者名字">
        <button class="primary-btn" onclick="login()">進入系統</button>
    </div>

    <div id="study-interface" class="hidden">
        <div style="display: flex; justify-content: space-between; width: 100%; align-items: center; margin-bottom: 10px;">
            <span id="user-display" style="font-weight: bold; color: var(--primary);"></span>
            <button onclick="logout()" style="background:none; border:none; color:#999; text-decoration:underline; cursor:pointer;">登出</button>
        </div>
        
        <div style="display:flex; gap:10px; width:100%; margin-bottom:15px;">
            <select id="unit-nav-selector" onchange="setUnitMode(this.value)" style="margin-bottom:0; flex-grow: 1;"></select>
            <button onclick="openAddModal()" style="padding:0 15px; border-radius:10px; border:none; background:var(--success); color:white; cursor:pointer; font-weight:bold;">+</button>
        </div>

        <div class="card" onclick="flipCard()">
            <div style="position: absolute; top: 20px; right: 20px; font-size: 24px; color: #ccc;" onclick="openManageModal(event)">⋯</div>
            <h1 id="word-display">載入中...</h1>
            <div id="detail-display" style="display: none; text-align: center; width: 100%;">
                <h2 style="margin-top:0;"><span id="trans-display"></span> <small id="pos-display" style="color:var(--primary); font-size: 0.9rem;"></small></h2>
                <p id="example-display" style="background: #f3f4f6; padding: 12px; border-radius: 10px; font-size: 0.85rem; margin-bottom:10px;"></p>
                <div id="note-display" style="color: #666; font-size: 0.8rem; border-top: 1px dashed #ddd; padding-top: 8px; font-style: italic;"></div>
            </div>
        </div>

        <div style="display: flex; gap: 20px; margin-top: 20px;">
            <button class="btn-circle" onclick="changeWord(-1)">←</button>
            <button id="star-btn" class="btn-circle" onclick="toggleStar()">☆</button>
            <button class="btn-circle" onclick="changeWord(1)">→</button>
        </div>

        <button class="primary-btn" onclick="startQuiz()">開始測驗</button>
        <div id="custom-folder-bar" style="margin-top: 20px; display: flex; flex-wrap: wrap; justify-content: center;"></div>
    </div>

    <div id="quiz-interface" class="hidden">
        <div style="margin-bottom: 15px; font-weight: bold; color: var(--primary);">測驗：<span id="quiz-progress"></span></div>
        <div class="card" style="cursor: default;"><div id="quiz-question-text" style="font-size: 1.3rem; font-weight: bold; margin-bottom: 15px;"></div><div id="quiz-options-area" style="width:100%"></div></div>
    </div>
    <div id="result-interface" class="hidden">
        <h2>測驗結束</h2><p id="score-text" style="font-weight:bold;"></p><div id="wrong-words-list" style="width:100%;"></div><button class="primary-btn" onclick="exitQuiz()">返回學習</button>
    </div>

    <div id="wordEditModal" class="modal">
        <div class="modal-content">
            <h3 id="modal-title" style="margin-top:0">單字資訊</h3>
            <label style="font-size:0.75rem; color:#666;">單元</label>
            <input list="units-datalist" id="edit-word-unit">
            <datalist id="units-datalist"></datalist>
            <input type="text" id="edit-word-eng" placeholder="英文單字">
            <select id="edit-word-pos">
                <option value="n.">n. (名詞)</option><option value="v.">v. (動詞)</option><option value="adj.">adj. (形容詞)</option><option value="adv.">adv. (副詞)</option>
                <option value="prep.">prep. (介係詞)</option><option value="conj.">conj. (連接詞)</option><option value="pron.">pron. (代名詞)</option>
                <option value="phr.">phr. (短語/片語)</option>
            </select>
            <input type="text" id="edit-word-chi" placeholder="中文翻譯">
            <textarea id="edit-word-ex" placeholder="例句" rows="2"></textarea>
            <textarea id="edit-word-note" placeholder="備註 / 補充資料 (選填)" rows="2" style="border-color: #4a90e2;"></textarea>
            
            <button class="primary-btn" style="background:var(--success)" onclick="saveWordChange()">儲存變更</button>
            <button id="delete-btn" class="primary-btn" style="background:var(--danger); display:none;" onclick="deleteWord()">刪除此字卡</button>
            <button class="primary-btn" style="background:#888" onclick="closeEditModal()">取消</button>
        </div>
    </div>

    <div id="manageModal" class="modal">
        <div class="modal-content">
            <h3 style="margin-top:0">管理與清單</h3>
            <button class="primary-btn" style="background:#f39c12; margin-bottom:15px;" onclick="openEditFromManage()">✏️ 編輯單字內容</button>
            <div style="border-top:1px solid #eee; padding-top:15px;">
                <p style="font-size:0.8rem; color:#666; margin-bottom:10px;">我的清單：</p>
                <div id="existing-lists-area" style="max-height:150px; overflow-y:auto; margin-bottom:10px;"></div>
                <input type="text" id="new-list-name" placeholder="建立新清單...">
                <button class="primary-btn" onclick="handleCreateAndAdd()">建立並加入</button>
            </div>
            <button class="primary-btn" style="background:#888" onclick="closeManageModal()">關閉</button>
        </div>
    </div>

    <div id="toast"></div>

    <script>
        let currentUser = "", vocabulary = [], customLists = [];
        let currentUnit = "all", currentCustomList = "none", currentIndex = 0, isFlipped = false;
        let quizQueue = [], quizIdx = 0, wrongWords = [], editingId = null;

        // --- 核心與存檔 ---
        function login() {
            const name = document.getElementById('user-name-input').value.trim();
            if (!name) return alert("請輸入名字");
            currentUser = name;
            loadUserData();
            document.getElementById('login-interface').classList.add('hidden');
            document.getElementById('study-interface').classList.remove('hidden');
            document.getElementById('user-display').innerText = "👤 " + currentUser;
            updateUnitSelector();
            updateUI();
        }

        function saveUserData() {
            localStorage.setItem("wordmaster_" + currentUser, JSON.stringify({ vocabulary, customLists }));
        }

        function loadUserData() {
            const savedData = localStorage.getItem("wordmaster_" + currentUser);
            if (savedData) {
                const parsed = JSON.parse(savedData);
                vocabulary = parsed.vocabulary;
                customLists = parsed.customLists;
            } else {
                vocabulary = [{ id: 1, unit: "預設單元", word: "Study", pos: "v.", trans: "學習", ex: "I study English.", note: "這是範例備註", starred: false, myLists: [] }];
                customLists = [];
                saveUserData();
            }
        }

        function logout() { location.reload(); }

        // --- 編輯/新增 (含備註) ---
        function openAddModal() {
            editingId = null;
            document.getElementById('modal-title').innerText = "新增字卡";
            document.getElementById('delete-btn').style.display = 'none';
            document.getElementById('edit-word-unit').value = currentUnit === "all" ? "" : currentUnit;
            document.getElementById('edit-word-eng').value = "";
            document.getElementById('edit-word-chi').value = "";
            document.getElementById('edit-word-ex').value = "";
            document.getElementById('edit-word-note').value = "";
            document.getElementById('wordEditModal').style.display = 'flex';
        }

        function openEditFromManage() {
            const item = getFilteredList()[currentIndex % getFilteredList().length];
            if (!item) return;
            editingId = item.id;
            document.getElementById('modal-title').innerText = "編輯字卡";
            document.getElementById('delete-btn').style.display = 'block';
            document.getElementById('edit-word-unit').value = item.unit;
            document.getElementById('edit-word-eng').value = item.word;
            document.getElementById('edit-word-pos').value = item.pos || "n.";
            document.getElementById('edit-word-chi').value = item.trans;
            document.getElementById('edit-word-ex').value = item.ex || "";
            document.getElementById('edit-word-note').value = item.note || "";
            closeManageModal();
            document.getElementById('wordEditModal').style.display = 'flex';
        }

        function saveWordChange() {
            const unit = document.getElementById('edit-word-unit').value.trim() || "未分類";
            const word = document.getElementById('edit-word-eng').value.trim();
            const pos = document.getElementById('edit-word-pos').value;
            const trans = document.getElementById('edit-word-chi').value.trim();
            const ex = document.getElementById('edit-word-ex').value.trim();
            const note = document.getElementById('edit-word-note').value.trim();

            if (!word || !trans) return showToast("請填寫單字與翻譯");

            if (editingId) {
                const idx = vocabulary.findIndex(v => v.id === editingId);
                if (idx !== -1) vocabulary[idx] = { ...vocabulary[idx], unit, word, pos, trans, ex, note };
            } else {
                vocabulary.push({ id: Date.now(), unit, word, pos, trans, ex, note, starred: false, myLists: [] });
            }

            saveUserData();
            updateUnitSelector();
            closeEditModal();
            updateUI();
            showToast("儲存成功");
        }

        function deleteWord() {
            if (!confirm("確定刪除此字卡？")) return;
            vocabulary = vocabulary.filter(v => v.id !== editingId);
            saveUserData();
            updateUnitSelector();
            closeEditModal();
            currentIndex = 0;
            updateUI();
        }

        function closeEditModal() { document.getElementById('wordEditModal').style.display = 'none'; }

        // --- 清單管理 (刪除清單) ---
        function deleteCustomList(listName, e) {
            e.stopPropagation();
            if (!confirm(`確定要刪除「${listName}」清單嗎？\n(單字不會被刪除，僅移除此分類標籤)`)) return;
            customLists = customLists.filter(l => l !== listName);
            vocabulary.forEach(v => {
                v.myLists = v.myLists.filter(l => l !== listName);
            });
            if (currentCustomList === listName) currentCustomList = "none";
            saveUserData();
            openManageModal({stopPropagation:()=>{}}); // 刷新列表
            updateUI();
        }

        function openManageModal(e) {
            e.stopPropagation();
            const area = document.getElementById('existing-lists-area');
            area.innerHTML = "";
            customLists.forEach(name => {
                const row = document.createElement('div');
                row.className = 'list-item-row';
                row.innerHTML = `
                    <span style="cursor:pointer; flex-grow:1;">+ ${name}</span>
                    <button class="del-list-btn" onclick="deleteCustomList('${name}', event)">🗑️</button>
                `;
                row.querySelector('span').onclick = () => {
                    const currentWord = getFilteredList()[currentIndex % getFilteredList().length];
                    if (currentWord && !currentWord.myLists.includes(name)) {
                        currentWord.myLists.push(name);
                        saveUserData();
                        showToast(`已加入「${name}」`);
                    }
                    closeManageModal();
                };
                area.appendChild(row);
            });
            document.getElementById('manageModal').style.display = 'flex';
        }

        function closeManageModal() { document.getElementById('manageModal').style.display = 'none'; }

        function handleCreateAndAdd() {
            const name = document.getElementById('new-list-name').value.trim();
            if (name) {
                if (!customLists.includes(name)) { customLists.push(name); saveUserData(); }
                const currentWord = getFilteredList()[currentIndex % getFilteredList().length];
                if (currentWord && !currentWord.myLists.includes(name)) { currentWord.myLists.push(name); saveUserData(); }
                document.getElementById('new-list-name').value = "";
                closeManageModal();
                updateUI();
            }
        }

        // --- UI 更新與基礎功能 ---
        function updateUI() {
            const list = getFilteredList();
            const wordDisplay = document.getElementById('word-display');
            const detailDisplay = document.getElementById('detail-display');
            if (list.length === 0) {
                wordDisplay.innerText = "尚無單字";
                wordDisplay.style.display = 'block';
                detailDisplay.style.display = 'none';
                return;
            }
            const item = list[currentIndex % list.length];
            wordDisplay.innerText = item.word;
            document.getElementById('trans-display').innerText = item.trans;
            document.getElementById('pos-display').innerText = `(${item.pos || 'n.'})`;
            document.getElementById('example-display').innerText = item.ex || "(無例句)";
            const noteArea = document.getElementById('note-display');
            noteArea.innerText = item.note || "";
            noteArea.style.display = item.note ? "block" : "none";
            
            document.getElementById('star-btn').innerText = item.starred ? "★" : "☆";
            document.getElementById('star-btn').style.color = item.starred ? "#ffd700" : "#333";
            renderFolderBar();
        }

        function getFilteredList() {
            let list = vocabulary;
            if (currentUnit !== "all") list = list.filter(v => v.unit === currentUnit);
            if (currentCustomList !== "none") list = list.filter(v => v.myLists.includes(currentCustomList));
            return list;
        }

        function updateUnitSelector() {
            const selector = document.getElementById('unit-nav-selector');
            const units = [...new Set(vocabulary.map(v => v.unit))];
            selector.innerHTML = '<option value="all">📚 所有單元</option>';
            units.forEach(u => selector.innerHTML += `<option value="${u}">📖 ${u}</option>`);
            selector.value = currentUnit;
            const datalist = document.getElementById('units-datalist');
            datalist.innerHTML = "";
            units.forEach(u => datalist.innerHTML += `<option value="${u}">`);
        }

        function flipCard() { if(getFilteredList().length===0) return; isFlipped = !isFlipped; document.getElementById('detail-display').style.display = isFlipped ? 'block' : 'none'; document.getElementById('word-display').style.display = isFlipped ? 'none' : 'block'; }
        function changeWord(n) { currentIndex += n; if(currentIndex < 0) currentIndex = 0; isFlipped = false; updateUI(); }
        function setUnitMode(u) { currentUnit = u; currentIndex = 0; updateUI(); }
        function setListMode(l) { currentCustomList = l; currentIndex = 0; updateUI(); }
        function toggleStar() { const item = getFilteredList()[currentIndex % getFilteredList().length]; if (item) { item.starred = !item.starred; saveUserData(); updateUI(); } }
        function showToast(m) { const t = document.getElementById("toast"); t.innerText = m; t.className = "show"; setTimeout(() => t.className = "", 2500); }
        function renderFolderBar() { const bar = document.getElementById('custom-folder-bar'); bar.innerHTML = `<button class="chip ${currentCustomList==='none'?'active':''}" onclick="setListMode('none')">全部</button>`; customLists.forEach(l => { bar.innerHTML += `<button class="chip ${currentCustomList===l?'active':''}" onclick="setListMode('${l}')">${l}</button>`; }); }

        // --- 測驗功能 (維持原樣) ---
        function startQuiz() { quizQueue = [...getFilteredList()]; if (quizQueue.length < 1) return showToast("無單字"); quizQueue.sort(() => Math.random() - 0.5); quizIdx = 0; wrongWords = []; document.getElementById('study-interface').classList.add('hidden'); document.getElementById('quiz-interface').classList.remove('hidden'); loadQuestion(); }
        function loadQuestion() {
            const item = quizQueue[quizIdx];
            document.getElementById('quiz-progress').innerText = `${quizIdx+1}/${quizQueue.length}`;
            const area = document.getElementById('quiz-options-area'); area.innerHTML = "";
            const isEng = Math.random() > 0.5;
            document.getElementById('quiz-question-text').innerText = isEng ? item.word : `${item.trans} (${item.pos})`;
            const correctText = isEng ? `${item.trans} (${item.pos})` : item.word;
            let opts = [correctText];
            let pool = vocabulary.filter(v => v.id !== item.id);
            pool.sort(() => Math.random() - 0.5);
            opts.push(...pool.slice(0, 3).map(v => isEng ? `${v.trans} (${v.pos})` : v.word));
            opts.sort(() => Math.random() - 0.5);
            opts.forEach(o => {
                const b = document.createElement('button'); b.className = 'opt-btn'; b.innerText = o;
                b.onclick = () => {
                    if(o===correctText) b.classList.add('correct');
                    else { b.classList.add('wrong'); wrongWords.push(item); document.querySelectorAll('.opt-btn').forEach(x => { if(x.innerText===correctText) x.classList.add('correct'); }); }
                    setTimeout(() => { quizIdx++; if(quizIdx < quizQueue.length) loadQuestion(); else showResults(); }, 1200);
                }; area.appendChild(b);
            });
        }
        function showResults() {
            document.getElementById('quiz-interface').classList.add('hidden');
            document.getElementById('result-interface').classList.remove('hidden');
            document.getElementById('score-text').innerText = `分數：${quizQueue.length - wrongWords.length}/${quizQueue.length}`;
            const area = document.getElementById('wrong-words-list'); area.innerHTML = "";
            wrongWords.forEach(w => {
                const div = document.createElement('div'); div.style = "display:flex; justify-content:space-between; background:white; padding:12px; border-radius:10px; margin-bottom:8px; align-items:center;";
                div.innerHTML = `<div style='text-align:left;'><strong>${w.word}</strong><br><small>${w.trans} (${w.pos})</small></div><span style="color:var(--primary); cursor:pointer;" onclick="openManageFromRes('${w.word}')">⋯</span>`;
                area.appendChild(div);
            });
        }
        function openManageFromRes(w) { const item = vocabulary.find(v => v.word === w); if(item) { currentIndex = getFilteredList().findIndex(v => v.id === item.id); openManageModal({stopPropagation:()=>{}}); } }
        function exitQuiz() { document.getElementById('result-interface').classList.add('hidden'); document.getElementById('study-interface').classList.remove('hidden'); updateUI(); }
    </script>
</body>
</html>
