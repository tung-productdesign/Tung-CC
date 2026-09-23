# Briefing 偏好設定

## 內容權重（2026-09-23 使用者回饋）
- 提高 🛠️ 新產品/新工具/新服務類項目的數量與占比。
- 🤖 AI 模型與技術更新維持精簡摘要，不擴充篇幅。
- （此為既有規則的加強版，非反轉：AI 模型類別本就要求精簡，此次使用者進一步要求 🛠️ 類別實際增加篇數。）

## Email 版面/配色偏好（2026-09-23 使用者提供範例，採用其風格）
使用者提供另一份 AI 產出的每日報（風格類似 Codex 產出），明確表示喜歡其排版、配色與字級，要求後續套用：

- **整體風格**：深色主題（非白底卡片），背景近黑色，文字淺色，簡潔無邊框卡片感。
- **色票**：
  - 頁面背景：`#0f0f13`
  - 標題文字：`#f5f5f7`（近白）
  - 內文文字：`#c7c7cf`（淺灰）
  - 次要/meta 文字：`#8a8a93`（灰）
  - 連結文字：`#b9b6f5`（淺紫藍）
  - 「今日 3 大重點」卡片背景：`#2a2015`（暖棕），左邊框 4px `#e08a4c`（橘）
  - 分隔線：`#2a2a30` 細線
- **字級**：標題約 19-20px、分類標題約 19-20px、內文約 15-15.5px、💡 PM 意義約 14.5px、來源/分數/連結 meta 行約 13.5px。
- **每則格式**：標題（粗體）→ 摘要段落 → 「💡 對 PM 的意義：」（emoji + 粗體前綴 + 內文同段）→ 單行 meta「來源 · 分數/100 · 閱讀原文 ↗」（不要像舊版拆成多行）。
- **分類標籤命名**（採用使用者範例中的命名）：
  - 🛠️ AI 工具 & 產品
  - 📊 產品經理方法與趨勢
  - 💰 產業動態 & 融資
  - 🤖 AI 模型 / 技術進展
- **分隔線**：每則項目之間用細線分隔，不使用大量留白色塊。
- 「今日內容較少」等提示用素色文字說明，不要用醒目的黃色警示框。

## 技術定案：配色改為「跟隨裝置淺色/深色模式自動切換」（2026-09-23 最終結論）
歷經三次錯誤嘗試才定案，記錄如下避免重蹈覆轍：
1. 第一版純深色 inline style（無 meta tag）→ 手機 Gmail App 深色模式下背景被自動反轉成白底。
2. 加上 `<meta name="color-scheme" content="only light">` 想鎖定深色 → 使用者確認**這個 meta tag 本身就是造成錯誤顯示的原因**，拿掉後也沒有變好。
3. 拿掉 meta tag 回到純 inline dark style → 手機深色模式下仍是白底/淺色（證明問題出在 Gmail 自己的深色模式引擎會忽略/覆蓋 inline dark 顏色，不是 meta tag 的問題），且桌面網頁淺色模式下卻整封變黑底，深淺邏輯完全不一致。

**使用者最終確認的正確方向：不要試圖強制固定深色，而是讓信件「跟隨 Gmail/裝置目前的淺色或深色模式」自動切換**（淺色模式 = 白底深字，深色模式 = 深底淺字）。技術做法：
- `<head>` 加入 `<meta name="color-scheme" content="light dark">` 與 `<meta name="supported-color-schemes" content="light dark">`。
- 每個需要隨模式變色的元素同時給：(a) inline style 寫淺色版數值（預設/fallback），(b) 一個對應的 class（如 `bgpage`、`bgcard`、`txttitle`、`txtbody`、`txtmuted`、`txtinsight`、`txtlink`、`txthighlight`、`txtfooter`、`divider`）。
- `<head>` 內用 `<style>` 區塊，在 `@media (prefers-color-scheme: dark) { ... }` 裡對每個 class 用 `!important` 覆寫成深色版數值。
- 額外加上 Gmail 專屬的深色模式判斷選擇器 `[data-ogsc]`（Gmail 會在自己判定要套用深色模式時，在元素上標記這個屬性），同樣針對每個 class 覆寫成深色版數值，當作 Gmail 自己引擎的雙重保險（純 `prefers-color-scheme` media query 在部分 Gmail 介面不一定會生效）。

**配色數值（淺色 / 深色）：**
- 頁面背景 `bgpage`：`#ffffff` / `#0f0f13`
- 重點框背景 `bgcard`：`#fdf1e2` / `#2a2015`（左邊框固定 `#e08a4c` 兩種模式都適用，不需切換）
- 標題文字 `txttitle`：`#1a1a2e` / `#f5f5f7`
- 內文文字 `txtbody`：`#444444` / `#c7c7cf`
- 次要/meta 文字 `txtmuted`：`#888888` / `#8a8a93`
- 💡 PM 意義文字 `txtinsight`：`#6b4a2a` / `#e8dcc8`
- 連結文字 `txtlink`：`#3b5bdb` / `#b9b6f5`
- 重點框內文字 `txthighlight`：`#4a3520` / `#f0e6da`
- 頁尾註記文字 `txtfooter`：`#999999` / `#6b6b73`
- 分隔線 `divider`（border-top）：`#e5e5e5` / `#2a2a30`

往後產生 Email HTML 一律採用此「淺色 inline 預設 + class + media query/data-ogsc 深色覆寫」架構，不要再嘗試「只做深色、強制不隨模式變化」的方向（已證實在 Gmail 各介面上不可靠）。

若使用者之後回報仍有顯示異常，先問清楚是哪個裝置/App、當下系統是淺色還是深色模式、看到的實際顏色，不要再自行盲猜第四種修法。

_最後更新：2026-09-23_
