# AI 提示詞產生器

四種提示詞框架：專為「AI 影音工坊」設計的逐段腳本框架，以及 TAG／APE／CO-STAR 三種通用提示詞框架。填變數即可組成結構完整的提示詞，也可以直接串接你自己的語言模型一鍵取得 AI 產生的內容。

🔗 **線上使用**：<https://m255525.github.io/ai-prompt-generator/>

⚠️ **需要授權序號才能使用**（見下方「授權序號」一節）。

## 這是什麼

- **AI 影音工坊框架**：填「主題／幾段」，組成的提示詞會要求 AI 產出逐行的「中文旁白｜英文搜尋關鍵字」腳本，格式與 [ai-video-studio](https://m255525.github.io/ai-video-studio-play/) 的腳本解析規則完全對應，AI 產生的結果可以直接複製貼過去使用。
- **TAG 框架**（Task／Action／Goal）：把「要做什麼任務、具體怎麼做、最終想達成什麼目標」講清楚，適合交辦型的單一任務。
- **APE 框架**（Action／Purpose／Expectation）：把「要做的事、為什麼要做、期待的輸出長什麼樣子」講清楚，適合明確、可執行的單一請求。
- **CO-STAR 框架**：Context 背景、Objective 目標、Style 風格、Tone 語氣、Audience 受眾、Response 輸出格式，六個維度最完整，適合需要精準控制文字風格與格式的內容產出。

每種框架都有「🔧 組成提示詞」（純前端字串組裝，不需金鑰、可複製）與「🚀 送給 AI 直接生成」（選用，需自備 API 金鑰，直接取得 AI 產生的最終內容）兩種用法。

## 功能

- **四個框架分頁**，各自獨立保留欄位內容、組成結果與 AI 產生結果，切換分頁不會互相覆蓋
- **內建範例**：每個框架各 3 組完全虛構的情境範例，一鍵套用快速上手
- **BYOK**：支援 Claude／OpenAI／Gemini／OpenRouter 四選一，API 金鑰只存在瀏覽器 localStorage，不經過任何後端伺服器
- **已儲存的提示詞**：可將組成的提示詞（連同 AI 產生結果）存成有名字的紀錄，之後載入、複製、下載 .txt 或刪除
- **授權序號閘門**：整個工具需輸入有效序號才能使用，效期 12 個月

## 怎麼用

1. 開啟 <https://m255525.github.io/ai-prompt-generator/>，輸入授權序號
2. 選一個框架分頁，套用範例或自行填寫欄位
3. 按「🔧 組成提示詞」取得可複製的提示詞文字；或展開「API 連線設定」貼上你自己的金鑰，按「🚀 送給 AI 直接生成」直接取得 AI 產生的內容
4. 滿意的結果可在「已儲存的提示詞」取名儲存

詳細操作說明見 [manual.html](https://m255525.github.io/ai-prompt-generator/manual.html)。

### API 金鑰申請網址

| 服務商 | 申請網址 |
|---|---|
| Claude（Anthropic） | <https://console.anthropic.com/> |
| OpenAI | <https://platform.openai.com/api-keys> |
| Gemini（Google AI Studio） | <https://aistudio.google.com/apikey> |
| OpenRouter | <https://openrouter.ai/keys> |

## 授權序號

本工具需先輸入授權序號並驗證通過，才能使用整個工具，效期自第一次驗證起算 12 個月。序號請向工具提供者索取；沒有序號請透過 [manual.html](https://m255525.github.io/ai-prompt-generator/manual.html) 上的聯絡方式洽詢。

## 技術架構

純前端單檔工具，**沒有任何建置流程、框架、npm 依賴**：

| 項目 | 做法 |
|---|---|
| 提示詞組裝 | 純前端字串模板，不連網 |
| AI 直接生成 | 瀏覽器直接 `fetch` 你選擇的 LLM 服務商官方 API（無後端代理） |
| 金鑰儲存 | `localStorage`，只在使用者自己的瀏覽器裡 |
| 授權序號驗證 | Google Sheet + Google Apps Script（免費輕量後端），只做序號驗證，不經手提示詞內容 |
| 頂部跑馬燈 | 與工作區其他工具共用同一個公告來源（可選、失敗不影響主功能） |

## 本機開發

不需要任何建置工具或安裝依賴，純靜態檔案：

```bash
git clone https://github.com/M255525/ai-prompt-generator.git
cd ai-prompt-generator
python -m http.server 8000
```

開啟 `http://localhost:8000`（本機測試序號閘門需先部署自己的 Apps Script 後端，見 `SETUP-授權伺服器設定.md`）。

## 檔案結構

```
index.html                     主程式（跑馬燈 + 序號閘門 + 四框架 + AI 生成 + 儲存清單）
manual.html                    操作手冊
Code.gs                        授權序號驗證用 Apps Script 後端原始碼
SETUP-授權伺服器設定.md         授權後端部署步驟
CLAUDE.md                      開發筆記／架構決策紀錄
```

## 隱私與資料

本 repo 公開的只有程式碼。你填寫的欄位內容、組成的提示詞、AI 產生結果只存在自己瀏覽器的 localStorage；按下「送給 AI 直接生成」時，這些內容會直接連線送到你選擇的 AI 服務商，不經過本工具作者或任何第三方伺服器。授權序號驗證只會傳送序號本身給驗證伺服器，不會傳送任何提示詞內容。

## 授權/用途

僅供教學與個人使用，禁止未經授權公開發布、販售或商業化使用。
