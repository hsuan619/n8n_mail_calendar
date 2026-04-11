# Gmail 郵件自動寫入 Google Calendar

這是一個基於 n8n 的自動化工作流，利用 **Gemini AI** 讀取 Gmail 郵件，自動分析內容是否包含重要行程，並將其分類後寫入 Google 日曆中。

## 📖 工作流簡介

本流程旨在解決手動從郵件記錄行程的繁瑣過程。

workflow架構

<img width="1430" height="413" alt="image" src="https://github.com/user-attachments/assets/29618417-8dc6-4874-9cbc-5870914b18e5" />

寫入測試

<img width="871" height="547" alt="image" src="https://github.com/user-attachments/assets/33eea8eb-52c1-432c-91fd-92ff9614fdc3" />

---

## 🚀 主要功能

* **自動觸發與過濾**：
    * 每分鐘自動檢查 Gmail 新郵件。
    * 透過 Google Sheets 維護的「黑名單」自動過濾不必要的寄件者（如 Uber、銀行通知）。
* **AI 深度分析 (Gemini)**：
    * 使用 `Gemini` 模型進行語意分析。
    * 自動判定重要性：會議、面試或特定重要人物的來信將優先處理。
    * 自動分類：補助申請、參加活動、通知、會議、面試或其他。
* **智能排程寫入**：
    * **全天事件**：若信中僅提及日期而無具體時間，則建立全天活動。
    * **時段事件**：若信中有明確時間，則自動對接至 Google 日曆的特定時段。

---

## 🛠 節點結構說明

1.  **數據輸入**：`Gmail 新郵件觸發` -> `Get row(s) in sheet (黑名單)`。
2.  **邏輯篩選**：透過 `JavaScript` 檢查寄件者是否符合黑名單條件。
3.  **AI 解析層**：
    * `郵件重要性分析` (AI Agent) 負責提取標題、日期、摘要與時間。
    * `結構化輸出解析器` 確保 AI 輸出穩定的 JSON 格式。
4.  **分流處理**：
    * `Split Out1`：處理多事件情況。
    * `重要性分流1`：根據 `time` 欄位是否為空來決定建立「全天」或「特定時間」事件。
5.  **終點**：將結果寫入 `Google Calendar`。

---

## ⚙️ 環境設定需求

在導入此 JSON 檔案後，請確保以下憑證 (Credentials) 已正確設定：

* **Gmail OAuth2 API**：讀取郵件權限。
* **Google Sheets OAuth2 API**：讀取黑名單試算表。
* **Google Gemini (PaLM) API**：用於執行 AI 分析任務。
* **Google Calendar OAuth2 API**：寫入日曆權限。

> [!NOTE]
> **預設時區說明**：目前工作流在寫入日曆時，預設使用 `+08:00` (台灣時區)。如需調整，請修改「建立高重要事件1」節點中的時間格式參數。

---

## 📊 AI 輸出格式參考

AI 處理後的結構如下：
| 欄位名稱 | 描述 |
| :--- | :--- |
| `category` | 事件分類（如：面試、補助） |
| `should_create_event` | 布林值，判定是否寫入日曆 |
| `title` | 優化後的行程標題 |
| `date` | YYYY-MM-DD 格式日期 |
| `time` | HH:mm 格式時間（若有） |

---
