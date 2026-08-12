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
- 部署狀態：**尚未部署**。`LICENSE_CHECK_URL` 目前是空字串，序號驗證會 fail-closed 顯示「尚未設定授權伺服器網址」並保持鎖定畫面。需要使用者依 `SETUP-授權伺服器設定.md` 建立自己的 Google Sheet 並部署 Apps Script 才能實際使用。

## 頂部共用跑馬燈

`#marqueeBar` 內容抓自工作區既有的共用授權伺服器（`https://script.google.com/macros/s/AKfycbwKX0.../exec`，與 `Prompt/index.html`、`ai-video-studio` 系列共用同一個 Google Sheet：<https://docs.google.com/spreadsheets/d/1sSBXW2dAc-4u0j21Q72MzNEBIhDccShhr1iJcAdG0UE/edit>），做法完全比照 `Prompt/index.html` 的獨立跑馬燈邏輯——**跟本工具自己的序號授權後端是兩個互不相干的系統**：頁面載入時直接 POST 一個空序號給共用端點（`doPost` 不論序號是否有效都會附上 `marquee` 陣列），`localStorage` key `promptGenMarquee`，每 20 分鐘背景重抓一次。改跑馬燈內容直接編輯共用 Sheet 即可，不需要重新部署任何 Apps Script。

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

## 本次未做（後續視需要再處理）

- 桌面版 exe 打包（`launcher.py` + PyInstaller，比照 `icap-generator/icap/`、`Prompt/Prompt_Eng/` 的做法；若之後要做，記得先查工作區其他 CLAUDE.md 目前已佔用的埠號清單，取一個未使用的埠）
- 根目錄 `專案目錄.docx` 尚未加入本專案的列
