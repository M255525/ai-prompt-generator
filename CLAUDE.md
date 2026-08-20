# CLAUDE.md — ai-prompt-generator

「AI 提示詞產生器」——單檔前端工具，提供四種提示詞框架：專為「AI 影音工坊」設計的逐段腳本框架，以及 TAG／APE／CO-STAR 三種通用提示詞框架。填變數即可組成結構完整的提示詞，也可以直接串接使用者自己的語言模型一鍵取得 AI 產生的內容。

## 架構

單一 `index.html`：內嵌 CSS/JS、無外部資源、無建置步驟。視覺主題是深色「藍圖網格」風格（`--bg #0b1220` + 淡格線背景 + 青色 `--accent #38bdf8`），跟 `Prompt/`（CRISPE 控制台，暖色調機架主題）與 `sbir-generator` 系列刻意做出區隔，方便一眼分辨是哪個工具。

- `FRAMEWORKS` 物件（`video`/`tag`/`ape`/`costar`）是整個工具的資料核心：每個框架定義 `fields`（欄位 key/label/type/placeholder）、`template(v)`（把欄位值組成提示詞文字的函式）、`presets`（3 組完全虛構的情境範例）。新增框架時只要往這個物件加一個 key，`renderTabs()`/`renderBuilder()` 會自動處理分頁與欄位渲染，不需要另外寫 UI。
- 狀態存 `localStorage`（key: `promptGenState`）：`{activeType, fields:{video,tag,ape,costar}, assembled:{...}, aiOutput:{...}}`——四個框架的欄位、已組成的提示詞、AI 產生結果都各自獨立保留，切換分頁不會互相覆蓋或清空。
- **核心互動模型**：「🔧 組成提示詞」是純前端字串組裝（不需金鑰、不連網），依框架各自的 `template()` 把欄位組成一段結構化提示詞，可直接複製使用；「🚀 送給 AI 直接生成」是選用功能，把組成好的提示詞送給使用者設定的 BYOK LLM，顯示 AI 的實際輸出結果。兩者分開的原因：讓沒填 API 金鑰的人仍能使用核心的框架組裝功能，同時滿足「直接產出最終內容」的需求。
- **影音工坊框架的格式規格**（務必維持與 `ai-video-studio` 的 `parseScript()` 相容）：每行「中文旁白文字｜英文Pexels搜尋關鍵字」，單一 `|` 分隔、不留空行、不加行號。`template()` 的 prompt 明確把這幾條規則寫給 AI，修改 prompt 措辭時這幾條規則不能拿掉。
- **BYOK AI 串接**：與 `Prompt/index.html`、`sbir-generator/sbir-gen-s/index.html` 同一套模式（改動時互相參照）——全部走瀏覽器直連 `fetch()`：Claude 需 `anthropic-dangerous-direct-browser-access: true` header；Gemini 金鑰放 `x-goog-api-key` header；OpenAI/OpenRouter 用 Bearer。設定（provider/model/apiKey）存 `localStorage`（key: `promptGenApiConfig`）——**金鑰只落在使用者本機瀏覽器，絕不可寫進程式碼**。逾時 180 秒；429/500/503/529 自動重試最多 2 次（間隔 8、16 秒）。
- **已儲存的提示詞**：`localStorage`（key: `promptGenSavedItems`），每筆 `{id, type, name, savedAt, fieldValues, assembledPrompt, aiOutput}`。「儲存」＝存進瀏覽器清單，「下載 .txt」才會真的產生檔案（把 `assembledPrompt` 與 `aiOutput`——若有——一起寫入同一個 .txt）。事件委派的單一 click listener（`#savedList`）處理載入／複製／下載／刪除。
- `manual.html` 操作手冊：四框架介紹／操作步驟／影音框架格式規則／已儲存的提示詞／AI 串接說明／授權序號說明／隱私說明／使用警語／創作者資料／授權限制。**創作者經歷內容與 `icap-generator/manual.html`、`sbir-generator/manual.html`、`phoenix-loan-generator/manual.html`、`Prompt/manual.html` 為同一份，更新其中一邊時同步其餘各邊。**

## 序號授權（鎖定整個工具，12 個月）

比照 `video-editor/mrvideo_s`／`ai-video-studio/AIvideo_studio` 的「鎖整個工具」模式（而非 `sbir-generator/sbir-gen-s` 只鎖單一功能的模式）：`#licenseGate` 全螢幕遮罩預設鎖定，驗證通過才加上 `.hidden`；載入時一律對後端即時重驗（不只信任 localStorage 快取），背景每 20 分鐘重驗一次，過期會自動重新鎖住整個頁面。`localStorage` key：`promptGenSerial`。

- `Code.gs` — 部署到**專屬於這個工具**的 Google Sheet 的 Apps Script 原始碼（不可沿用其他工具的授權表，效期/用途不同）：`doPost` 只做序號驗證＋首次自動啟用，`doGet` 供部署後測試。Sheet 欄位「序號／開始日期／結束日期」，`VALID_AMOUNT = 12`（月）。這不是這個資料夾裡的檔案在跑，是使用者手動複製貼到 Google Sheet 的「擴充功能 → Apps Script」編輯器裡部署成 Web App，取得網址後回填到 `index.html` 的 `LICENSE_CHECK_URL`。部署步驟見 `SETUP-授權伺服器設定.md`。
- **這支後端只做序號驗證，不代理任何付費 API**（本工具的 LLM 串接是 BYOK，前端直連使用者自己的服務商 API，跟序號系統無關），也**不處理跑馬燈**（見下）。
- 綁定的 Google Sheet：<https://docs.google.com/spreadsheets/d/1zki3fjDGGFARrglkpMSaqKpjDAdjgoU47PxR9jDXqrQ/edit>（獨立於工作區其他工具的授權表）。
- **已完成部署（2026-08-12）**。`index.html` 的 `LICENSE_CHECK_URL` 已填入實際部署網址：`https://script.google.com/macros/s/AKfycbx-_ipM4k2Bgyqzl9DqczwVCdQhzhTn_DkbLzSqn1Yb6dc7W5KrNVKLFokZxrjg1Tl1/exec`。`doGet`／`doPost` 皆已用 curl／Node `fetch()` 驗證正常（`doPost` 對不存在的序號正確回傳 `serial_not_found`）；Sheet 目前只有表頭、尚無任何序號列，需使用者自行新增序號列才能實際測試啟用流程。
- **部署過程踩坑**：透過聊天視窗複製貼上 `Code.gs` 到 Apps Script 編輯器出現語法錯誤（`Illegal return statement`），本機檔案 `node --check` 語法正常，確認是剪貼簿/瀏覽器貼上過程弄壞內容——改用 `clasp login` → `clasp clone <scriptId>` → 覆蓋 `Code.js` → `clasp push --force` 一次成功，跳過複製貼上。部署後第一次 `doPost` 測試曾短暫回傳 Google Drive「找不到網頁」錯誤頁，重試（間隔數秒）後恢復正常，屬新部署的暫時性延遲，非程式碼問題。

## 序號剩餘天數持續顯示（2026-08-13）

同 `ai-image-prompt-studio` 的修法（同一套序號授權骨架，發現「解鎖後遮罩消失、剩餘天數也跟著看不到」是系統性缺口後一併補上）：`.topbar` 內 `nav` 前新增常駐徽章 `#licenseBadge`（🔑 剩餘 N 天，hover 顯示到期日），`unlock()`/`lock()` 同步更新／隱藏，剩餘 ≤7 天變色（本檔沒有既有的 warn/amber 色變數，改用內嵌 `rgba(251,191,36,.14)`/`#fbbf24`）。語法已用 `node --check` 驗證通過，實際解鎖流程沿用與 `ai-image-prompt-studio` 相同的程式碼、已在該專案端對端驗證過。

## 頂部共用跑馬燈

`#marqueeBar` 內容抓自工作區既有的共用授權伺服器（`https://script.google.com/macros/s/AKfycbwKX0.../exec`，與 `Prompt/index.html`、`ai-video-studio` 系列共用同一個 Google Sheet：<https://docs.google.com/spreadsheets/d/1sSBXW2dAc-4u0j21Q72MzNEBIhDccShhr1iJcAdG0UE/edit>），做法完全比照 `Prompt/index.html` 的獨立跑馬燈邏輯——**跟本工具自己的序號授權後端是兩個互不相干的系統**：頁面載入時直接 POST 一個空序號給共用端點（`doPost` 不論序號是否有效都會附上 `marquee` 陣列），`localStorage` key `promptGenMarquee`，每 20 分鐘背景重抓一次。改跑馬燈內容直接編輯共用 Sheet 即可，不需要重新部署任何 Apps Script。

**2026-08-20 更新（`Code.gs` 未改動、不需重新部署）**：`render()` 新增 `lastKey`（`JSON.stringify(items)`）比對，內容沒變就不重繪，CSS animation 不再被重置歸零重跑；新增 `appendParsedText()`／`buildTrackContent()` 支援 `[文字](https://...)` 連結語法（`createTextNode` 組 DOM，避免 XSS），資料格式仍是純字串陣列，向下相容。已 commit＋push（GitHub Pages 自動重新部署）。

## 隱私與警語

無伺服器端經手使用者資料；欄位內容、組成結果、AI 產生結果、已儲存清單皆只存在使用者瀏覽器的 localStorage。序號驗證只會傳送序號本身給授權伺服器，不會傳送任何提示詞內容。首頁與手冊皆明列使用警語：AI 生成內容需自行查核、請勿輸入真實個資或機密資料、僅供教學與個人使用禁止商業化。修改功能時這些警語需一併檢視是否仍準確。

## 指令

無建置/測試指令。修改 `index.html` 或 `manual.html` 後直接用瀏覽器開啟驗證，或暫起 `python -m http.server <port>` 測完關閉。修改內嵌 `<script>` 後可用以下方式快速檢查語法（把 `<script>...</script>` 內容抽出存成 `.js` 再跑 `node --check`）：

```bash
python -c "
import re
html = open('index.html', encoding='utf-8').read()
open('_check.js','w',encoding='utf-8').write(re.findall(r'<script>(.*?)</script>', html, re.S)[0])
"
node --check _check.js
```

**測試序號授權邏輯前，需先照 `SETUP-授權伺服器設定.md` 部署好 Apps Script 並回填 `LICENSE_CHECK_URL`**，否則會顯示「尚未設定授權伺服器網址」的 fail-closed 錯誤訊息並停留在鎖定畫面；開發階段要測試框架/欄位/AI/儲存清單等其他功能，可在瀏覽器 devtools 手動對 `#licenseGate` 加上 `hidden` class 暫時繞過。

## 加入主畫面（PWA，2026-08-14 新增）

比照 `expense-tracker-pwa` 的做法加上安裝支援：`manifest.json`（`icons/icon-192.png`／`icon-512.png`／`icon-maskable-512.png`，白底「詞」字圖示，青色 `#38bdf8` 背景比對 `--accent`）＋`service-worker.js`。**與 `expense-tracker-pwa` 不同**：這裡不追求離線可用（本工具本來就需要連網打 LLM API），`service-worker.js` 採 network-first＋同源快取備援（只快取 `index.html`／`manifest.json`／icons，跨網域請求一律略過不進快取），所以**不需要**像 `expense-tracker-pwa` 那樣每次改動都手動升版 `CACHE_NAME`——這是刻意的取捨，換取單檔工具不必維護版本升級紀律。頁尾 `.footer-meta` 新增「📲 加入主畫面」按鈕（`#installBtn`），邏輯是獨立 IIFE（`beforeinstallprompt`/`appinstalled` 監聽＋SW 註冊），跟序號授權閘門互不相依。已用 Playwright 實測：Chromium 確實判定本頁可安裝並觸發 `beforeinstallprompt`。


**iOS／iPadOS／macOS 相容性補強（2026-08-14 同日追加）**：Safari（含 iOS 上的 Chrome/Firefox，底層都是 WebKit）**永遠不會觸發 `beforeinstallprompt`**，原本的按鈕邏輯在這些瀏覽器上一律落入「瀏覽器不支援」這句話，其實是誤導——蘋果裝置本來就能加入主畫面，只是要透過分享選單手動操作，不像 Chrome/Edge 有自動彈窗。修法：安裝腳本新增 `isIOSDevice`（`/iPad|iPhone|iPod/` 或 `navigator.platform==='MacIntel' && maxTouchPoints>1`——後者是因為 iPadOS 13+ 預設偽裝成 Mac 桌面版 UA，要用觸控點數才分得出來是 iPad 還是真的 Mac）與 `isMacDesktop && isSafariEngine`（macOS 桌面版 Safari 17+ 是「檔案→加入 Dock」，跟手機的分享選單操作不同）兩種判斷，各自顯示對應的操作指引文字，不再顯示「不支援」；`isStandalone`（`matchMedia('(display-mode: standalone)')` 或 iOS 專有的 `navigator.standalone`）為真時直接隱藏按鈕（已經是安裝後開啟，不需要再顯示安裝按鈕）。`<head>` 同步補上 `apple-touch-icon`（180×180 專用尺寸，`icons/apple-touch-icon.png`，純色不透明背景）＋ `apple-mobile-web-app-capable`／`mobile-web-app-capable`（兩個都要，前者給 Safari、後者是 Chrome 主張的新標準，只寫一個 Chrome 會在主控台噴 deprecation warning）＋ `apple-mobile-web-app-status-bar-style`／`apple-mobile-web-app-title`。這五個判斷/訊息字串在全部 9 個已加裝 PWA 的專案裡是逐字複製的同一段邏輯，日後若要調整任一處的措辭或判斷式，建議九個一起改，避免各專案的安裝體驗不一致。

**回饋機制與快取踩坑修正（2026-08-14，使用者實測回報「加入主畫面沒有功能」才發現兩層問題）**：(1) 原本無 `showToast` 時用「暫時置換按鈕文字」當提示，在工具列裡太不明顯，使用者完全沒注意到訊息出現過——改成 `window.alert(fallbackMessage())`，`deferredPrompt.prompt()` 也包 try/catch。(2) 改完使用者仍回報沒反應，追查發現 `service-worker.js` 的 `fetch(event.request)` 沒有繞過瀏覽器 HTTP 快取——GitHub Pages 對回應下 `Cache-Control: max-age=600`，10 分鐘內「network-first」名不符實，可能吃到舊版內容重新存進 Cache Storage。改成 `fetch(event.request, {cache:'reload'})` 強制略過 HTTP 快取，`CACHE_NAME` 同步升版 v1→v2 清掉已污染的快取。這是跟 `expense-tracker-pwa` 那次「install 階段 `cache.addAll()` 忘記加 `{cache:'reload'}`」同一個 bug class 的 runtime 版本，細節見 [[pwa-install-rollout]]。

**上面「回饋機制與快取踩坑修正」第(1)點的描述對本檔不準確，已於 2026-08-16 修正**：那段文字（改用 `window.alert()`）實際上只套用到當時**沒有** `showToast()` 的 6 個專案；本工具有 `showToast()`，被誤判為「沒有這個問題」而跳過，但實際程式碼一直是 `if (typeof showToast === 'function') showToast(fallbackMessage());`——這裡有個關鍵盲點：PWA 安裝腳本是獨立 `<script>`／獨立 IIFE，`showToast()` 宣告在**另一個**（主程式的）IIFE 裡，函式作用域不會跨 IIFE 共享，所以 `typeof showToast` 在安裝腳本裡永遠是 `'undefined'`——`deferredPrompt` 為 `null` 時點安裝按鈕會**完全沒有任何回饋、也沒有主控台錯誤**。這是 2026-08-16 排查 SocialPost「按鈕鍵但沒有對應功能」回報時才發現、確認 `ai-image-prompt-studio`／`ai-music-prompt-studio` 也同樣中招的系統性 bug，不是本工具獨有。修法：安裝腳本不再依賴外部 `showToast`，改成自己實作 `notify(msg)`（直接操作 `#toast` DOM 元素），`deferredPrompt.prompt()` 補上 try/catch，三個姊妹專案一次修正。

## 訪客次數計數器（2026-08-21 新增）

頁尾 `.footer-meta` 加了 `visitor-badge.laobi.icu` 的 SVG badge（`<img>` 直接嵌入，`page_id=m255525.ai-prompt-generator`，免金鑰免後端），做法比照 `SocialPost` 已驗證過的模式，與 `ai-image-prompt-studio`／`ai-music-prompt-studio` 同一次一併加上。

## 本次未做（後續視需要再處理）

- 桌面版 exe 打包（`launcher.py` + PyInstaller，比照 `icap-generator/icap/`、`Prompt/Prompt_Eng/` 的做法；若之後要做，記得先查工作區其他 CLAUDE.md 目前已佔用的埠號清單，取一個未使用的埠）
- 根目錄 `專案目錄.docx` 尚未加入本專案的列
