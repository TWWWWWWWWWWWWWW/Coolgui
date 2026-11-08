(function () {

'use strict';

// Ensure mobile viewport sizing when running from a mobile browser.
if (!document.querySelector('meta[name=viewport]')) {
    const viewport = document.createElement('meta');
    viewport.name = 'viewport';
    viewport.content = 'width=device-width, initial-scale=1, maximum-scale=1, viewport-fit=cover';
    document.head.appendChild(viewport);
}

// =========================
// 狀態
// =========================

let guiVisible = false;
let activeCleanup = null;

const state = {

    darkMode:false,
    blurMode:false,
    rotateMode:false,
    rainbowMode:false,
    imagesHidden:false,
    autoScroll:false,
    borderMode:false,
    rgbMode:false,
    transparentMode:false,
    spinImages:false,
    freezeAnimations:false,
    fpsEnabled:false,
    zoomMode:false,
    superMode:false,
    godMode:false,
    textHidden:false
};

let rainbowInterval = null;
let autoScrollInterval = null;
let freezeStyle = null;

// =========================
// CSS
// =========================

const coolguiStyle = document.createElement('style');
coolguiStyle.textContent = `

@keyframes coolrgb {
    0% { filter: hue-rotate(0deg); }
    100% { filter: hue-rotate(360deg); }
}

@keyframes coolspin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

.coolgui-btn{
position:fixed;
top:20px;
right:20px;
width:150px;
height:60px;
background:black;
color:white;
border:3px solid red;
border-radius:14px;
z-index:999999;
cursor:pointer;
font-size:18px;
font-weight:bold;
box-shadow:0 0 20px red;
}

.coolgui-menu{
position:fixed;
top:50%;
left:50%;
transform:translate(-50%,-50%);
width:min(1200px,95vw);
max-height:85vh;
overflow:auto;
background:#000;
color:white;
border-radius:18px;
padding:20px;
z-index:1000000;
box-shadow:0 0 40px red;
}

.coolgui-title{
font-size:30px;
font-weight:bold;
text-align:center;
margin-bottom:15px;
}

.coolgui-search{
width:100%;
padding:12px;
margin-bottom:15px;
background:#111;
color:white;
border:2px solid red;
border-radius:12px;
box-sizing:border-box;
}

.coolgui-actions{
display:flex;
flex-wrap:wrap;
gap:10px;
justify-content:center;
}

.coolgui-action{
width:170px;
height:60px;
background:black;
color:white;
border:2px solid red;
border-radius:12px;
cursor:pointer;
font-size:14px;
font-weight:bold;
transition:.2s;
}

.coolgui-action:hover{
background:red;
transform:scale(1.05);
}

.coolgui-notify{
position:fixed;
right:20px;
bottom:20px;
background:black;
color:white;
border:2px solid red;
border-radius:12px;
padding:10px;
z-index:9999999;
}

.coolgui-rotate{
transform:rotate(180deg);
transition:1s;
}

.coolgui-blur{
filter:blur(5px);
}

.coolgui-rgb{
animation:coolrgb 2s linear infinite;
}

.coolgui-border *{
outline:1px solid red !important;
}

.coolgui-spin img{
animation:coolspin 2s linear infinite;
}

.coolgui-transparent{
opacity:0.5;
}

.coolgui-dark{
background:black !important;
color:white !important;
}

.coolgui-super{
filter:contrast(150%) saturate(200%);
transform:scale(1.03);
}

.coolgui-god *{
border-radius:20px !important;
box-shadow:0 0 10px red !important;
}


/* Mobile / touch layout */
* { -webkit-tap-highlight-color: transparent; }
.coolgui-menu {
  box-sizing: border-box;
  width: min(100vw - 16px, 760px);
  max-width: calc(100vw - 16px);
  max-height: calc(100dvh - 16px);
  max-height: calc(100vh - 16px);
  top: 8px; left: 8px; transform: none;
  padding: 14px;
  padding-bottom: max(14px, env(safe-area-inset-bottom));
  overflow-y: auto; overscroll-behavior: contain;
  touch-action: pan-y;
}
.coolgui-title { font-size: 23px; margin: 4px 0 12px; }
.coolgui-search { min-height: 48px; font-size: 16px; }
.coolgui-actions {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 9px;
}
.coolgui-action {
  box-sizing: border-box;
  width: 100%; min-width: 0; min-height: 52px; height: auto;
  padding: 10px 6px; font-size: 14px;
  touch-action: manipulation; user-select: none;
}
.coolgui-action:active { transform: scale(.97); background: #500; }
.coolgui-notify {
  right: 10px; left: 10px; bottom: max(10px, env(safe-area-inset-bottom));
  text-align: center; font-size: 14px; overflow-wrap: anywhere;
}
.coolgui-mobile-controls {
  display:flex; gap:8px; justify-content:center; margin:0 0 12px;
}
.coolgui-mobile-controls button {
  flex:1; min-height:46px; background:#171717; color:white;
  border:2px solid red; border-radius:10px; font-size:15px;
  touch-action:manipulation;
}
@media (max-width:360px) {
  .coolgui-actions { grid-template-columns:1fr; }
  .coolgui-title { font-size:20px; }
}

`;
document.head.appendChild(coolguiStyle);

// =========================
// UI
// =========================

const button = document.createElement('button');

button.className = 'coolgui-btn';
button.textContent = 'COOL GUI';

button.style.display = 'none';

document.body.appendChild(button);

const menu = document.createElement('div');

menu.className = 'coolgui-menu';
menu.style.display = 'none';

document.body.appendChild(menu);

const title = document.createElement('div');

title.className = 'coolgui-title';
title.textContent = 'COOL GUI';

menu.appendChild(title);

const mobileControls = document.createElement('div');
mobileControls.className = 'coolgui-mobile-controls';

const cancelButton = document.createElement('button');
cancelButton.textContent = '停止目前操作';
cancelButton.addEventListener('click', () => cancelCurrentMode());

const minimizeButton = document.createElement('button');
minimizeButton.textContent = '－ 最小化';
minimizeButton.addEventListener('click', () => {
    menu.style.display = 'none';

    button.textContent = 'COOL';
    button.style.cssText =
        'position:fixed;right:14px;bottom:20px;top:auto;left:auto;' +
        'width:62px;height:62px;border-radius:50%;' +
        'background:#111;color:white;border:2px solid red;' +
        'z-index:9999999;font-weight:bold;display:block;';
});

const closeButton = document.createElement('button');
closeButton.textContent = '× 永久關閉';
closeButton.addEventListener('click', () => {
    cancelCurrentMode();
    clearInterval(rainbowInterval);
    clearInterval(autoScrollInterval);

    menu.remove();
    button.remove();
    fpsBox.remove();
    coolguiStyle.remove();
});

mobileControls.append(cancelButton, minimizeButton, closeButton);
menu.appendChild(mobileControls);

const searchBox = document.createElement('input');

searchBox.className = 'coolgui-search';
searchBox.placeholder = '搜尋功能...';

menu.appendChild(searchBox);

const actionsContainer = document.createElement('div');

actionsContainer.className = 'coolgui-actions';

menu.appendChild(actionsContainer);

// =========================
// FPS
// =========================

const fpsBox = document.createElement('div');

fpsBox.style.position = 'fixed';
fpsBox.style.left = '10px';
fpsBox.style.bottom = '10px';
fpsBox.style.background = 'black';
fpsBox.style.color = 'lime';
fpsBox.style.padding = '8px';
fpsBox.style.zIndex = '999999';
fpsBox.style.display = 'none';

document.body.appendChild(fpsBox);

let lastFrame = performance.now();

function fpsLoop(){

    const now = performance.now();

    fpsBox.textContent =
        'FPS: ' +
        Math.round(1000 / (now - lastFrame));

    lastFrame = now;

    requestAnimationFrame(fpsLoop);
}

fpsLoop();

// =========================
// 通知
// =========================

function notify(text){

    const n = document.createElement('div');

    n.className = 'coolgui-notify';

    n.textContent = text;

    document.body.appendChild(n);

    setTimeout(() => {

        n.remove();

    },3000);
}

// =========================
// createAction
// =========================

function createAction(text, handler){

    const btn = document.createElement('button');

    btn.className = 'coolgui-action';
    btn.dataset.action = text;
    btn.textContent = '開啟：' + text;
    btn.onclick = handler;

    actionsContainer.appendChild(btn);
    return btn;
}

function setActionMode(text, enabled){
    const btn = Array.from(actionsContainer.querySelectorAll('.coolgui-action')).find(item => item.dataset.action === text);
    if (btn) btn.textContent = (enabled ? '取消：' : '開啟：') + text;
}

// =========================
// 搜尋
// =========================

searchBox.addEventListener('input', () => {

    const value =
        searchBox.value.toLowerCase();

    actionsContainer
    .querySelectorAll('.coolgui-action')
    .forEach(btn => {

        btn.style.display =
            btn.dataset.action
            .toLowerCase()
            .includes(value)
            ? 'block'
            : 'none';
    });
});

// =========================
// ESC
// =========================

let activeToolName = null;

function cancelCurrentMode(){
    if(activeCleanup){
        const cleanup = activeCleanup;
        const toolName = activeToolName;
        activeCleanup = null;
        activeToolName = null;
        cleanup();
        if (toolName) setActionMode(toolName, false);
        notify('已取消目前模式');
    }
}

document.addEventListener('keydown', e => {

    if(e.key === 'Escape'){

        cancelCurrentMode();

        menu.style.display = 'none';
    }
});

// =========================
// 崩網：有限時效能測試，不會永久鎖死瀏覽器
// =========================

createAction('崩網', () => {
    cancelCurrentMode();
    let stopped = false;
    const deadline = performance.now() + 2500;
    function stopStress() {
        stopped = true;
        if (activeCleanup === stopStress) activeCleanup = null;
        notify('崩網測試已停止');
    }
    activeCleanup = stopStress;
    notify('崩網測試開始（最多 2.5 秒）');
    function runBatch() {
        if (stopped || performance.now() >= deadline) {
            if (!stopped) {
                if (activeCleanup === stopStress) activeCleanup = null;
                notify('崩網測試完成');
            }
            return;
        }
        const until = performance.now() + 8;
        let n = 0;
        while (performance.now() < until) n += Math.sqrt(Math.random() * 10000);
        void n;
        setTimeout(runBatch, 0);
    }
    runBatch();
});

// =========================
// 功能
// =========================

createAction('暗黑模式', () => {

    state.darkMode = !state.darkMode;
    setActionMode('暗黑模式', state.darkMode);

    document.documentElement
    .classList.toggle('coolgui-dark');
});

createAction('模糊網頁', () => {

    state.blurMode = !state.blurMode;
    setActionMode('模糊網頁', state.blurMode);

    document.body
    .classList.toggle('coolgui-blur');
});

createAction('旋轉網頁', () => {

    state.rotateMode = !state.rotateMode;
    setActionMode('旋轉網頁', state.rotateMode);

    document.body
    .classList.toggle('coolgui-rotate');
});

createAction('RGB 模式', () => {

    state.rgbMode = !state.rgbMode;
    setActionMode('RGB 模式', state.rgbMode);

    document.body
    .classList.toggle('coolgui-rgb');
});

createAction('彩虹模式', () => {

    state.rainbowMode = !state.rainbowMode;
    setActionMode('彩虹模式', state.rainbowMode);

if(state.rainbowMode){

    rainbowInterval =
        setInterval(() => {

            document.body.style.background =
                `hsl(${Math.random() * 360}, 100%, 50%)`;

        }, 200);

}else{

        clearInterval(rainbowInterval);

        document.body.style.background = '';
    }
});

createAction('SUPER MODE', () => {

    state.superMode = !state.superMode;
    setActionMode('SUPER MODE', state.superMode);

    document.body
    .classList.toggle('coolgui-super');
});

createAction('GOD MODE', () => {

    state.godMode = !state.godMode;
    setActionMode('GOD MODE', state.godMode);

    document.body
    .classList.toggle('coolgui-god');
});

createAction('隱藏圖片', () => {

    state.imagesHidden = !state.imagesHidden;
    setActionMode('隱藏圖片', state.imagesHidden);

    if (state.imagesHidden) {
        window.__coolguiImageDisplay = Array.from(document.querySelectorAll('img'))
            .map(img => [img, img.style.display]);
        window.__coolguiImageDisplay.forEach(([img]) => { img.style.display = 'none'; });
    } else {
        (window.__coolguiImageDisplay || []).forEach(([img, value]) => {
            if (img.isConnected) img.style.display = value;
        });
        window.__coolguiImageDisplay = null;
    }
});

createAction('圖片旋轉', () => {

    state.spinImages = !state.spinImages;
    setActionMode('圖片旋轉', state.spinImages);

    document.body
    .classList.toggle('coolgui-spin');
});

createAction('透明模式', () => {

    state.transparentMode = !state.transparentMode;
    setActionMode('透明模式', state.transparentMode);

    document.body
    .classList.toggle('coolgui-transparent');
});

createAction('凍結動畫', () => {

    state.freezeAnimations = !state.freezeAnimations;
    setActionMode('凍結動畫', state.freezeAnimations);

    if(state.freezeAnimations){

        freezeStyle =
            document.createElement('style');

        freezeStyle.textContent =

`
*{
    animation:none !important;
    transition:none !important;
}
`
;

        document.head.appendChild(
            freezeStyle
        );

    }else{

        if(freezeStyle){

            freezeStyle.remove();
        }
    }
});

createAction('元素邊框', () => {

    state.borderMode = !state.borderMode;
    setActionMode('元素邊框', state.borderMode);

    document.body
    .classList.toggle('coolgui-border');
});

createAction('FPS', () => {

    state.fpsEnabled = !state.fpsEnabled;
    setActionMode('FPS', state.fpsEnabled);

    fpsBox.style.display =
        state.fpsEnabled
        ? 'block'
        : 'none';
});

let fontScaleRestore = null;
let fontScaleMode = null;

function toggleFontScale(mode, factor, label){
    if (fontScaleMode === mode) {
        if (fontScaleRestore) fontScaleRestore();
        fontScaleRestore = null;
        fontScaleMode = null;
        setActionMode(label, false);
        notify('已還原文字大小');
        return;
    }
    if (fontScaleRestore) fontScaleRestore();
    const elements = Array.from(document.querySelectorAll('body *'));
    const originals = elements.map(el => [el, el.style.fontSize]);
    elements.forEach(el => {
        const size = parseFloat(getComputedStyle(el).fontSize);
        if (Number.isFinite(size)) el.style.fontSize = (size * factor) + 'px';
    });
    fontScaleRestore = () => originals.forEach(([el, value]) => {
        if (el.isConnected) el.style.fontSize = value;
    });
    fontScaleMode = mode;
    setActionMode(label, true);
}

createAction('放大文字', () => toggleFontScale('large', 1.2, '放大文字'));
createAction('縮小文字', () => toggleFontScale('small', 0.8, '縮小文字'));

createAction('隱藏文字', () => {

    state.textHidden = !state.textHidden;
    setActionMode('隱藏文字', state.textHidden);

    if (state.textHidden) {
        window.__coolguiTextVisibility = Array.from(document.querySelectorAll('p,span,h1,h2,h3,h4,h5,h6'))
            .map(el => [el, el.style.visibility]);
        window.__coolguiTextVisibility.forEach(([el]) => { el.style.visibility = 'hidden'; });
    } else {
        (window.__coolguiTextVisibility || []).forEach(([el, value]) => {
            if (el.isConnected) el.style.visibility = value;
        });
        window.__coolguiTextVisibility = null;
    }
});

createAction('網頁縮放', () => {

    state.zoomMode = !state.zoomMode;
    setActionMode('網頁縮放', state.zoomMode);

    document.body.style.zoom =
        state.zoomMode
        ? '150%'
        : '100%';
});

createAction('自動滾動', () => {

    state.autoScroll = !state.autoScroll;
    setActionMode('自動滾動', state.autoScroll);

    if(state.autoScroll){

        autoScrollInterval =
            setInterval(() => {

            window.scrollBy(0,5);

        },10);

    }else{

        clearInterval(
            autoScrollInterval
        );
    }
});

createAction('回到頂部', () => {

    window.scrollTo({

        top:0,
        behavior:'smooth'
    });
});

createAction('回到底部', () => {

    window.scrollTo({

        top:document.body.scrollHeight,
        behavior:'smooth'
    });
});

createAction('全螢幕', () => {

    if(!document.fullscreenElement){

        if (!document.documentElement.requestFullscreen) {
            notify('此手機瀏覽器不支援全螢幕 API');
            return;
        }
        document.documentElement.requestFullscreen().catch(() => notify('全螢幕請求被瀏覽器拒絕'));

    }else{

        document.exitFullscreen();
    }
});

createAction('顯示密碼', () => {

    if (!window.__coolguiPasswordsShown) {
        window.__coolguiPasswordTypes = Array.from(document.querySelectorAll('input[type="password"]'))
            .map(el => [el, el.type]);
        window.__coolguiPasswordTypes.forEach(([el]) => { el.type = 'text'; });
        window.__coolguiPasswordsShown = true;
        setActionMode('顯示密碼', true);
        notify('密碼已顯示，再點一次可隱藏');
    } else {
        (window.__coolguiPasswordTypes || []).forEach(([el, type]) => {
            if (el.isConnected) el.type = type;
        });
        window.__coolguiPasswordTypes = null;
        window.__coolguiPasswordsShown = false;
        setActionMode('顯示密碼', false);
        notify('密碼欄位已還原');
    }
});

createAction('隱藏廣告', () => {

    const selector = '[class*=ad],[id*=ad],iframe';
    if (!window.__coolguiAdsHidden) {
        window.__coolguiAdsDisplay = Array.from(document.querySelectorAll(selector))
            .map(el => [el, el.style.display]);
        window.__coolguiAdsDisplay.forEach(([el]) => { el.style.display = 'none'; });
        window.__coolguiAdsHidden = true;
        setActionMode('隱藏廣告', true);
        notify('廣告已隱藏，再點一次可還原');
    } else {
        (window.__coolguiAdsDisplay || []).forEach(([el, value]) => {
            if (el.isConnected) el.style.display = value;
        });
        window.__coolguiAdsDisplay = null;
        window.__coolguiAdsHidden = false;
        setActionMode('隱藏廣告', false);
        notify('廣告顯示已還原');
    }
});

createAction('顯示所有圖片', () => {

    const urls =
        Array.from(document.images)
        .map(img => img.src)
        .join('\n');

    alert(urls || '沒有圖片');
});

createAction('顯示所有連結', () => {

    const links =
        Array.from(
            document.querySelectorAll('a')
        )
        .map(a => a.href)
        .join('\n');

    alert(links || '沒有連結');
});

createAction('複製全部文字', async () => {

    try {
        await navigator.clipboard.writeText(document.body.innerText);
        notify('已複製');
    } catch (err) {
        notify('複製失敗：此瀏覽器可能不允許剪貼簿存取');
    }
});

createAction('執行 JS', () => {

    const code =
        prompt('輸入 JS');

    if(!code) return;

    try{

        eval(code);

        notify('JS 執行成功');

    }catch(err){

        alert(err);
    }
});

createAction('注入 CSS', () => {

    if (window.__coolguiInjectedStyle) {
        window.__coolguiInjectedStyle.remove();
        window.__coolguiInjectedStyle = null;
        setActionMode('注入 CSS', false);
        notify('已移除上次注入的 CSS');
        return;
    }

    const css = prompt('輸入 CSS');
    if (!css) return;

    const injectedStyle = document.createElement('style');
    injectedStyle.textContent = css;
    document.head.appendChild(injectedStyle);
    window.__coolguiInjectedStyle = injectedStyle;
    setActionMode('注入 CSS', true);
    notify('CSS 已注入，再點一次可移除');
});

createAction('元素檢查器', () => {

    if (activeCleanup && activeToolName === '元素檢查器') { cancelCurrentMode(); return; }
    cancelCurrentMode();
    activeToolName = '元素檢查器';
    setActionMode('元素檢查器', true);

    notify('點元素查看資訊');

   function inspect(e){

    if ((menu.contains(e.target) || button.contains(e.target))) return;
    e.preventDefault();
    e.stopPropagation();

    const el = e.target;

    alert(`
TAG: ${el.tagName}

ID: ${el.id}

CLASS: ${el.className}
`);

    cleanup();
}

    function cleanup(){

        document.removeEventListener(
            'click',
            inspect,
            true
        );

        activeCleanup = null;
        activeToolName = null;
        setActionMode('元素檢查器', false);
    }

    activeCleanup = cleanup;

    document.addEventListener(
        'click',
        inspect,
        true
    );
});

createAction('刪除元素', () => {

    if (activeCleanup && activeToolName === '刪除元素') { cancelCurrentMode(); return; }
    cancelCurrentMode();
    activeToolName = '刪除元素';
    setActionMode('刪除元素', true);

    notify('點元素刪除');

    function removeHandler(e){

        if ((menu.contains(e.target) || button.contains(e.target))) return;
        e.preventDefault();

        e.stopPropagation();

        const el = e.target;

        if(
            el === menu ||
            menu.contains(el) ||
            el === button
        ){
            return;
        }

        el.remove();

        cleanup();

        notify('元素已刪除');
    }

    function cleanup(){

        document.removeEventListener(
            'click',
            removeHandler,
            true
        );

        activeCleanup = null;
        activeToolName = null;
        setActionMode('刪除元素', false);
    }

    activeCleanup = cleanup;

    document.addEventListener(
        'click',
        removeHandler,
        true
    );
});

createAction('直接改字', () => {

    if (activeCleanup && activeToolName === '直接改字') { cancelCurrentMode(); return; }
    cancelCurrentMode();
    activeToolName = '直接改字';
    setActionMode('直接改字', true);

    notify('點文字開始修改（ESC 隨時退出）');

    let currentEditable = null;

    function finishEdit() {

        if (!currentEditable) return;

        currentEditable.contentEditable = 'false';
        currentEditable.style.outline = '';
        currentEditable.blur();

        currentEditable = null;
    }

    function editHandler(e) {

        // 忽略 GUI
        if (
            e.target === menu ||
            menu.contains(e.target) ||
            e.target === button
        ) return;

        const el = e.target;

        // 過濾危險/無意義元素
        if (
            el === document.body ||
            el === document.documentElement ||
            ['SCRIPT', 'STYLE', 'INPUT', 'TEXTAREA'].includes(el.tagName)
        ) return;

        e.preventDefault();
        e.stopPropagation();

        finishEdit();

        currentEditable = el;

        el.contentEditable = 'true';
        el.style.outline = '2px solid red';
        el.focus();

        notify('正在編輯文字（ESC 結束）');

        // 游標到最後
        const range = document.createRange();
        range.selectNodeContents(el);
        range.collapse(false);

        const sel = window.getSelection();
        sel.removeAllRanges();
        sel.addRange(range);
    }

    function keyHandler(e) {

        if (!currentEditable) return;

        if (e.key === 'Escape') {
            e.preventDefault();
            finishEdit();
            notify('已退出編輯');
        }

        if (e.key === 'Enter') {

            // DIV / P 才允許換行
            if (!['DIV', 'P'].includes(currentEditable.tagName)) {
                e.preventDefault();
                finishEdit();
                notify('已完成編輯');
            }
        }
    }

    function outsideClickHandler(e) {

        if (!currentEditable) return;

        if (
            e.target !== currentEditable &&
            !currentEditable.contains(e.target)
        ) {
            finishEdit();
        }
    }

    function forceGlobalExit() {
        finishEdit();
    }

    function cleanup() {

        forceGlobalExit();

        document.removeEventListener('click', editHandler, true);
        document.removeEventListener('keydown', keyHandler, true);
        document.removeEventListener('pointerdown', outsideClickHandler, true);

        activeCleanup = null;
        activeToolName = null;
        setActionMode('直接改字', false);
    }

    activeCleanup = cleanup;

    document.addEventListener('click', editHandler, true);
    document.addEventListener('keydown', keyHandler, true);
    document.addEventListener('pointerdown', outsideClickHandler, true);

});
// =========================
// GUI
// =========================

button.addEventListener('click', () => {
    menu.style.display = 'block';
    button.style.display = 'none';
});

// =========================
// F12 / 手機 Console：載入後直接開啟
// =========================

button.style.display = 'none';
menu.style.display = 'block';
guiVisible = true;

// =========================
// 初始化
// =========================

notify(
'COOL GUI 已載入：模式按鈕可再次點擊取消' 
);

})();
