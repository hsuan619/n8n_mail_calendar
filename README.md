# Gmail 郵件自動寫入 Google Calendar

這是一個以 **n8n** 建立的自動化工作流程，主要功能是：

1. 監聽 Gmail 新郵件。
2. 透過黑名單過濾不需要處理的寄件對象。
3. 使用 Gemini 模型分析郵件內容，判斷是否需要建立行事曆事件。
4. 將符合條件的郵件轉換為 Google Calendar 事件。
5. 依照郵件是否包含明確時間，分流建立「全天事件」或「指定時間事件」。

workflow架構
<img width="1430" height="413" alt="image" src="https://github.com/user-attachments/assets/29618417-8dc6-4874-9cbc-5870914b18e5" />
寫入測試
<img width="869" height="544" alt="測試img" src="https://github.com/user-attachments/assets/2d95ed49-3901-4f60-8157-969b0e8e433c" />


---

## 專案目的

許多重要通知會以 Email 形式送達同時不會自動加入行事曆，例如：

- 會議邀請
- 面試通知
- 補助申請
- 繳費或截止日期提醒
- 學校事務通知

這個 workflow 的目的，是將這類郵件自動轉換成日曆事件，避免使用者漏看重要時程。

---

## 流程概覽

### 1. Gmail 新郵件觸發
Workflow 以 Gmail Trigger 作為入口，每分鐘檢查一次新郵件。

### 2. 黑名單篩選
先從 Google Sheets 讀取黑名單，並透過 JavaScript Code 比對收件者地址。

若郵件收件者命中黑名單關鍵字，則直接排除；未命中才進入後續分析。

### 3. 初步排除不必要郵件
再用 Filter 節點排除明顯不需要建立事件的郵件，例如 Uber、銀行通知等類型。

### 4. Gemini 郵件分類與事件抽取
將郵件主旨與內文送入 Gemini 模型，並搭配結構化輸出解析器，要求模型回傳固定 JSON 格式，包含：

- category
- should_create_event
- title
- date
- time
- summary
- subject
- sender

### 5. 只保留需要建立的事件
若 `should_create_event = true`，才會繼續往下執行。

### 6. 依時間資訊分流
使用 Switch 節點判斷 `time` 是否為空：

- **time 為空**：建立全天事件
- **time 不為空**：建立指定時間事件

### 7. 寫入 Google Calendar
最後將事件寫入指定的 Google Calendar。

---

## 節點說明

### Gmail 新郵件觸發
- 類型：`gmailTrigger`
- 功能：監聽新郵件
- 觸發頻率：每分鐘

### Get row(s) in sheet
- 類型：`googleSheets`
- 功能：讀取黑名單資料
- 資料來源：Google Sheets 中的黑名單工作表

### Code in JavaScript
- 類型：`code`
- 功能：將 Gmail 收件者與黑名單比對
- 輸出：`proceed: true/false`

### 初步將不必要的信件篩除(e.g. uber, 銀行)
- 類型：`filter`
- 功能：排除不需要進入 AI 分析的信件

### 郵件重要性分析
- 類型：`@n8n/n8n-nodes-langchain.agent`
- 功能：使用 Gemini 分析郵件內容
- 目的：判斷是否要建立事件，並抽取日期、時間與標題

### 結構化輸出解析器
- 類型：`outputParserStructured`
- 功能：將模型輸出限制為固定 JSON 結構

### 僅保留須建立的事件
- 類型：`filter`
- 功能：僅放行需要建立日曆事件的郵件

### 重要性分流1
- 類型：`switch`
- 功能：依據是否有時間分成兩條路徑

### 建立高重要事件
- 類型：`googleCalendar`
- 功能：建立全天事件

### 建立高重要事件1
- 類型：`googleCalendar`
- 功能：建立指定時間事件

---

## 事件建立規則

### 會建立事件的內容
- 會議
- 面試
- 補助申請
- 截止日期
- 繳費提醒
- 學校事務
- 特定重要寄件者的郵件**(未來加上重要人員名單)**

### 不建立事件的內容
- 活動宣傳
- 明顯的行銷郵件
- 已知無需處理的服務通知


---

## 使用前提

- 已設定 Gmail OAuth2
- 已設定 Google Sheets OAuth2
- 已設定 Google Calendar OAuth2
- 已啟用 Gemini API (model: Gemma4 26B)
- Google Sheets 黑名單資料已建立

---

## 注意事項

1. 目前日期欄位使用的是 `{{ $now.year }}-MM-DD` 的格式設計，實際落地時需確認模型輸出的日期是否完整且可解析。
2. `time` 若缺失，會預設為全天事件。
3. workflow 中的日曆描述欄位可再優化，目前字串模板仍有可調整空間。
4. 黑名單是依收件者地址進行比對，適合做第一層粗篩，但不等於完整內容判斷。

---

## 適用情境

這個 workflow 很適合：

- 個人信箱自動整理
- 學生處理課程、面試、補助通知
- 行政或專案信件自動排程
- 需要快速把重要郵件轉成待辦/日曆事件的情境

