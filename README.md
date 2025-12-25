#如果不想下载，复制以下代码到记事本保存，后缀名改为.html即可
<!DOCTYPE html>
<html lang="zh-Hans">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>专业洞洞谱 - 全功能修复版</title>
    <style>
        :root {
            --color-low: #777; --color-mid: #000; --color-high: #b00;
            --bg-wood: #fff9e6; --border-wood: #e0d5b1;
            --active-blue: #2196f3; --insert-orange: #ff9800;
            --a4-width: 210mm; --a4-height: 297mm;
        }
        body { font-family: "Microsoft YaHei", sans-serif; background: #525659; margin: 0; display: flex; height: 100vh; overflow: hidden; }
        
        #p_container { flex: 1; height: 100%; overflow-y: auto; display: flex; flex-direction: column; align-items: center; padding: 20px; box-sizing: border-box; gap: 20px; }

        .page { width: var(--a4-width); min-height: var(--a4-height); background: white; padding: 15mm 10mm; box-shadow: 0 0 10px rgba(0,0,0,0.5); box-sizing: border-box; position: relative; display: flex; flex-direction: column; flex-shrink: 0; }

        .editor-panel { width: 380px; background: #fff; border-left: 1px solid #ccc; padding: 20px; overflow-y: auto; z-index: 100; }
        
        .preview-grid { display: flex; flex-wrap: wrap; gap: 30px 3px; align-content: flex-start; }
        .note-unit { display: flex; flex-direction: column; align-items: center; width: 36px; cursor: pointer; border-radius: 4px; transition: all 0.1s; padding: 2px 0; position: relative; }
        
        .note-unit:hover { background-color: #f0f0f0; }
        .note-unit.selected { background-color: #e3f2fd !important; outline: 1px solid var(--active-blue); z-index: 10; }
        .note-unit.selected.insert-mode-active { outline: 2px solid var(--insert-orange); background-color: #fff3e0 !important; }
        
        .note-unit.selected.insert-mode-active::before {
            content: ""; position: absolute; left: -2px; top: 0; bottom: 0; width: 3px; background: var(--insert-orange); border-radius: 2px;
        }

        .setting-group { display: flex; flex-direction: column; gap: 8px; margin-bottom: 20px; }
        .setting-group input { padding: 8px; border: 1px solid #ddd; border-radius: 4px; }
        
        .key-btn { min-width: 40px; height: 38px; border: 1px solid #ddd; border-radius: 4px; cursor: pointer; font-size: 14px; font-weight: bold; background: white; }
        .btn-low { color: var(--color-low); }
        .btn-high { color: var(--color-high); }
        .func { background: #f8f9fa; font-size: 12px; flex: 1; }

        .note-num-area { position: relative; display: flex; justify-content: center; align-items: center; height: 30px; width: 100%; margin-bottom: 12px; }
        .note-num { font-size: 20px; font-weight: bold; position: relative; pointer-events: none; display: flex; align-items: center; }

        /* 节奏符号渲染 */
        .dot-after { margin-left: 2px; font-size: 16px; position: absolute; right: -8px; top: 0; }
        .underline-1 { position: absolute; bottom: -4px; left: 10%; right: 10%; height: 1.5px; background: #000; }
        .underline-2 { position: absolute; bottom: -7px; left: 10%; right: 10%; height: 1.5px; background: #000; }

        /* 高低音点精准对齐 */
        .is-high .note-num::before { content: "˙"; position: absolute; top: -14px; left: 50%; transform: translateX(-50%); font-size: 24px; font-weight: bold; }
        .is-low .note-num::after { content: "."; position: absolute; bottom: -14px; left: 50%; transform: translateX(-50%); font-size: 20px; font-weight: bold; }

        .lyric-input { width: 100%; border: none; border-bottom: 1px solid transparent; text-align: center; font-size: 14px; font-weight: bold; padding: 4px 0; margin-bottom: 8px; background: transparent; outline: none; color: #333; }
        
        /* 指法管外框：延时线共享此高度 */
        .fingering-body { background-color: var(--bg-wood); border: 1.5px solid var(--border-wood); border-radius: 6px; padding: 4px 1px; display: flex; flex-direction: column; align-items: center; pointer-events: none; min-height: 125px; box-sizing: border-box; }
        .fingering-img { display: flex; flex-direction: column; align-items: center; font-size: 22px; line-height: 0.8; width: 26px; }
        .fingering-img span { display: flex; justify-content: center; align-items: center; height: 20px; width: 100%; }

        /* ◐ 符号专项修复：防倾斜与对齐 */
        .half-dot-wrapper { display: flex; justify-content: center; align-items: center; width: 100%; height: 20px; overflow: visible; }
        .half-dot { 
            font-family: Arial, sans-serif !important; 
            font-style: normal !important; 
            display: inline-block; 
            transform: scale(0.6); 
            position: relative; 
            left: 0.8px; 
            margin-top: -1px; 
            line-height: 1; 
        }

        .mode-toggle { display: flex; align-items: center; gap: 10px; padding: 10px; background: #f0f7ff; border-radius: 6px; margin-bottom: 15px; border: 1px solid #d0e0f0; }
        .mode-toggle.orange { background: #fff3e0; border-color: #ffe0b2; }

        @media print {
            body { background: white; overflow: visible; }
            .editor-panel { display: none; }
            #p_container { padding: 0; overflow: visible; display: block; }
            .page { box-shadow: none; margin: 0; page-break-after: always; }
            .note-unit.selected { outline: none; background: none; }
        }
    </style>
</head>
<body>

<div id="p_container" onclick="deselectAll(event)"></div>

<div class="editor-panel">
    <h3 id="mode-status">录入模式</h3>
    
    <div id="mode-ui" class="mode-toggle">
        <input type="checkbox" id="insert-toggle" style="width:18px; height:18px; cursor:pointer;" onchange="updateUI()">
        <label for="insert-toggle" style="font-size:14px; cursor:pointer; font-weight:bold;">开启“插入模式”</label>
    </div>

    <div class="setting-group">
        <input type="text" id="title" placeholder="歌曲标题" oninput="updateUI()">
        <input type="text" id="subtitle" placeholder="曲目信息 (1=D 4/4)" oninput="updateUI()">
        <select id="type" onchange="changeType()" style="padding:8px;">
            <option value="5">筒音作5</option>
            <option value="1">筒音作1</option>
        </select>
    </div>

    <div style="margin-bottom:15px; display:grid; grid-template-columns: 1fr 1fr 1fr; gap:4px;">
        <button class="key-btn func" onclick="modifyRhythm('line', 1)">减时线+</button>
        <button class="key-btn func" onclick="modifyRhythm('line', -1)">减时线-</button>
        <button class="key-btn func" onclick="modifyRhythm('dot', 1)">附点</button>
        <button class="key-btn func" onclick="copySelected()">复制</button>
        <button class="key-btn func" onclick="pasteNotes()">粘贴</button>
        <button class="key-btn func" onclick="handleInput('-')">延时线(-)</button>
    </div>

    <div class="keyboard-area" style="display:flex; flex-direction:column; gap:8px;">
        <div id="row-low" style="display:flex; gap:3px; flex-wrap:wrap;"></div>
        <div id="row-mid" style="display:flex; gap:3px; flex-wrap:wrap;"></div>
        <div id="row-high" style="display:flex; gap:3px; flex-wrap:wrap;"></div>
    </div>

    <div style="display:flex; gap:4px; margin-top:15px; flex-wrap: wrap;">
        <button class="key-btn func" onclick="handleInput('BAR')">小节线</button>
        <button class="key-btn func" onclick="handleInput('SPACE')">间隔</button>
        <button class="key-btn func" onclick="backspace()" style="color:red">删除选中</button>
    </div>

    <div style="display:grid; grid-template-columns: 1fr 1fr; gap:5px; margin-top:15px;">
        <button class="key-btn func" onclick="exportProject()">导出存档</button>
        <button class="key-btn func" onclick="document.getElementById('fileInput').click()">读取存档</button>
        <input type="file" id="fileInput" style="display:none" accept=".json" onchange="importProject(event)">
        <button class="key-btn func" style="background:#2196f3; color:white; border:none; grid-column: span 2;" onclick="window.print()">保存为 PDF</button>
        <button class="key-btn func" style="grid-column: span 2;" onclick="clearAll()">清空全谱</button>
    </div>
</div>

<script>
    const c_b = '●', c_w = '○', c_h = '◐';
    const one_map = { '1':'bbbbbb', '2':'bbbbbw', '3':'bbbbww', '4':'bbbwww', '5':'bbwwww', '6':'bwwwww', '7':'wwwwww', '[1]':'wbbbbb', '[2]':'bbbbbw', '[3]':'bbbbww', '[4]':'bbbwww', '[5]':'bbwwww' };
    const five_map = { '(5)':'bbbbbb', '(6)':'bbbbbw', '(7)':'bbbbww', '1':'bbbwww', '2':'bbwwww', '3':'bwwwww', '4':'wbbwww', '5':'wbbbbb', '6':'bbbbbw', '7':'bbbbww', '[1]':'bbbwww', '[2]':'bbwwww', '[3]':'bwwwww', '[4]':'hwwwww', '[5]':'wbbbbb' };

    let inputArr = [];
    let selectedIndices = [];
    let clipboard = [];

    function renderKeyboard() {
        const type = document.getElementById('type').value;
        const layout = type === '5' ? 
            { low: ['(5)', '(6)', '(7)'], mid: ['1', '2', '3', '4', '5', '6', '7'], high: ['[1]', '[2]', '[3]', '[4]', '[5]'] } :
            { low: [], mid: ['1', '2', '3', '4', '5', '6', '7'], high: ['[1]', '[2]', '[3]', '[4]', '[5]'] };
        
        const draw = (id, notes, cls) => {
            const el = document.getElementById(id); el.innerHTML = '';
            notes.forEach(n => {
                const b = document.createElement('button');
                b.className = `key-btn ${cls}`; b.textContent = n;
                b.onclick = (e) => { e.stopPropagation(); handleInput(n); }; 
                el.appendChild(b);
            });
        };
        draw('row-low', layout.low, 'btn-low'); draw('row-mid', layout.mid, 'btn-mid'); draw('row-high', layout.high, 'btn-high');
    }

    function handleInput(n) {
        const isInsertMode = document.getElementById('insert-toggle').checked;
        const newItem = { note: n, lyric: '', lines: 0, dot: false };
        if (selectedIndices.length === 1) {
            let idx = selectedIndices[0];
            if (isInsertMode) {
                inputArr.splice(idx, 0, newItem);
                selectedIndices = [idx + 1];
            } else {
                inputArr[idx].note = n;
                selectedIndices = [];
            }
        } else {
            inputArr.push(newItem);
        }
        updateUI();
    }

    function modifyRhythm(type, val) {
        selectedIndices.forEach(idx => {
            if (inputArr[idx].note === 'BAR' || inputArr[idx].note === 'SPACE') return;
            if (type === 'line') inputArr[idx].lines = Math.max(0, Math.min(2, (inputArr[idx].lines || 0) + val));
            if (type === 'dot') inputArr[idx].dot = !inputArr[idx].dot;
        });
        updateUI();
    }

    function selectNote(index, event) {
        event.stopPropagation();
        if (event.shiftKey && selectedIndices.length > 0) {
            const start = Math.min(selectedIndices[0], index);
            const end = Math.max(selectedIndices[0], index);
            selectedIndices = Array.from({length: end - start + 1}, (_, i) => start + i);
        } else {
            selectedIndices = [index];
        }
        updateUI();
    }

    function deselectAll(e) { if(e.target.id === "p_container" || e.target.className === "page") { selectedIndices = []; updateUI(); } }
    function backspace() {
        if (selectedIndices.length > 0) {
            selectedIndices.sort((a,b)=>b-a).forEach(idx => inputArr.splice(idx, 1));
            selectedIndices = [];
        } else { inputArr.pop(); }
        updateUI();
    }

    function copySelected() { if (selectedIndices.length > 0) clipboard = selectedIndices.sort((a,b)=>a-b).map(idx => ({...inputArr[idx]})); }
    function pasteNotes() {
        if (clipboard.length > 0) {
            const targetIdx = selectedIndices.length > 0 ? Math.min(...selectedIndices) : inputArr.length;
            inputArr.splice(targetIdx, 0, ...clipboard.map(item => ({...item})));
            updateUI();
        }
    }

    function exportProject() {
        const data = { title: document.getElementById('title').value, subtitle: document.getElementById('subtitle').value, type: document.getElementById('type').value, inputArr };
        const blob = new Blob([JSON.stringify(data)], {type: 'application/json'});
        const a = document.createElement('a'); a.href = URL.createObjectURL(blob);
        a.download = (data.title || "乐谱存档") + ".json"; a.click();
    }

    function importProject(e) {
        const file = e.target.files[0]; if (!file) return;
        const reader = new FileReader();
        reader.onload = (event) => {
            const data = JSON.parse(event.target.result);
            document.getElementById('title').value = data.title || "";
            document.getElementById('subtitle').value = data.subtitle || "";
            document.getElementById('type').value = data.type || "5";
            inputArr = data.inputArr || []; renderKeyboard(); updateUI();
        };
        reader.readAsText(file);
    }

    function updateUI() {
        const container = document.getElementById('p_container');
        const isInsertMode = document.getElementById('insert-toggle').checked;
        const currentScrollTop = container.scrollTop;
        container.innerHTML = ''; 

        let currentPage = createPage(1);
        container.appendChild(currentPage);
        let grid = currentPage.querySelector('.preview-grid');
        const currentMap = document.getElementById('type').value === '1' ? one_map : five_map;

        inputArr.forEach((item, index) => {
            const isSelected = selectedIndices.includes(index);
            const unit = document.createElement('div');
            unit.className = 'note-unit' + (isSelected ? ' selected' : '');
            if(isSelected && isInsertMode && selectedIndices.length === 1) unit.classList.add('insert-mode-active');
            unit.onclick = (e) => selectNote(index, e);

            if (item.note === 'BAR') {
                unit.style.width = "10px";
                unit.innerHTML = `<div style="height:150px; border-right:2px solid #ddd; margin: 0 4px;"></div>`;
            } else if (item.note === 'SPACE') {
                unit.style.width = "15px";
                unit.innerHTML = `<div style="height:150px;"></div>`;
            } else {
                let colorClass = 'is-mid';
                let displayNum = item.note;
                if (item.note.includes('(')) { colorClass = 'is-low'; displayNum = item.note.replace(/[()]/g,''); }
                else if (item.note.includes('[')) { colorClass = 'is-high'; displayNum = item.note.replace(/[\[\]]/g,''); }

                // --- 延时线特殊处理：不生成孔位 ---
                let fingerHtml = '';
                if (item.note !== '-') {
                    const fingerStr = currentMap[item.note] || 'wwwwww';
                    for(let char of fingerStr) {
                        if(char === 'h') fingerHtml += `<div class="half-dot-wrapper"><i class="half-dot">◐</i></div>`;
                        else fingerHtml += `<span>${char === 'b' ? c_b : c_w}</span>`;
                    }
                }

                unit.classList.add(colorClass);
                unit.innerHTML = `
                    <div class="note-num-area">
                        <span class="note-num">${displayNum}${item.dot?'<span class="dot-after">.</span>':''}
                            ${item.lines >= 1 ? '<div class="underline-1"></div>' : ''}
                            ${item.lines >= 2 ? '<div class="underline-2"></div>' : ''}
                        </span>
                    </div>
                    <input type="text" class="lyric-input" value="${item.lyric}" placeholder="-" onclick="event.stopPropagation()" oninput="inputArr[${index}].lyric=this.value">
                    <div class="fingering-body"><div class="fingering-img">${fingerHtml}</div></div>
                `;
            }

            grid.appendChild(unit);
            if (grid.offsetHeight > 920) {
                currentPage = createPage(container.children.length + 1, true);
                container.appendChild(currentPage);
                grid = currentPage.querySelector('.preview-grid');
                grid.appendChild(unit);
            }
        });
        container.scrollTop = currentScrollTop;
    }

    function createPage(num, hideHeader = false) {
        const div = document.createElement('div');
        div.className = 'page';
        const titleText = document.getElementById('title').value || '未命名乐谱';
        const subText = document.getElementById('subtitle').value || '1=D 4/4';
        div.innerHTML = `
            <div class="page-header" style="${hideHeader ? 'display:none' : ''}">
                <p style="text-align:center; font-size:26px; font-weight:bold; margin:0;">${titleText}</p>
                <p style="text-align:center; font-size:13px; margin-top:5px; margin-bottom:10px;">${subText}</p>
            </div>
            <div class="preview-grid"></div>
            <div style="position:absolute; bottom:8mm; left:50%; transform:translateX(-50%); font-size:11px; color:#999;">- 第 ${num} 页 -</div>
        `;
        return div;
    }

    function changeType() { renderKeyboard(); inputArr = []; updateUI(); }
    function clearAll() { if(confirm("清空全谱？")) { inputArr = []; updateUI(); } }
    renderKeyboard();
    updateUI();
</script>
</body>
</html>
