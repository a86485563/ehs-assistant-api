# OpenAPI Governance

## 文件定位

本文件是 API governance 的 OpenAPI 專門分支，定義所有 MCP API 的 OpenAPI 3.0、Swagger 與 Agent 可用性細項。資料擷取與爬蟲結果驗證規則另見 `docs/quality/data-validation.md`。

### 來源與適用層級

- 原始簡報為本機參考資料，未納入版本控制；本文件與下方摘要是可共享的整理結果。
- 整理摘要：[AI-Ready-OpenAPI-Agent規範摘要.md](source/AI-Ready-OpenAPI-Agent%E8%A6%8F%E7%AF%84%E6%91%98%E8%A6%81.md)。
- 本分支將來源中已確認且適合本專案的規則列為 API 撰寫要求；來源明確標示為選填、進階或待確認的內容，不得自行升級為所有 API 的強制要求。
- 每支 API 完成時，應依本文件的「API 撰寫自我檢查」逐項確認，並在契約測試或審查紀錄中留下結果。

## OpenAPI 版本與文件

1. 每一隻 MCP API 必須提供 OpenAPI 3.0.x 文件。
2. OpenAPI 文件必須可以被標準解析器成功解析。
3. 每一隻 MCP API 必須提供可使用的 Swagger UI 或等效 Swagger 文件檢視方式。
4. Swagger 顯示內容、OpenAPI 文件、API 實作與測試必須保持一致。
5. API 行為或資料結構變更時，必須同步更新 OpenAPI 文件與相關測試。

## Operation ID

每個 operation 必須具有唯一的 `operationId`，並符合以下規則：

- 使用 lowerCamelCase。
- 採用「動詞 + 名詞或名詞片語」結構。
- 清楚表達實際用途。
- 不得使用模糊名稱，例如 `processData` 或 `handleRequest`。

### Agent 業務動詞分類

`operationId` 的動詞應從業務動作出發，並對應下列分類：

| 分類 | 適用情境 | 動詞示例 |
| --- | --- | --- |
| Acquire | 尋找並取得資訊 | `search`、`query`、`check`、`retrieve`、`scan` |
| Compute | 執行運算或規則驗證 | `summarize`、`validate`、`calculate`、`predict` |
| Transact | 觸發狀態、資金或資源變更 | `submit`、`register`、`cancel`、`authorize` |
| Integrate | 連接系統或同步資料 | `merge`、`sync`、`link`、`import`、`connect` |
| Orchestrate | 編排多步驟任務 | `schedule`、`chain`、`retry`、`route` |
| Notify | 傳遞進度、異常或結果 | `notify`、`alert`、`broadcast`、`report` |

名詞應完整、可理解，除公司內部專有名詞外不應縮寫。爬蟲讀取資訊可依用途歸入 Acquire；此分類是依業務定義推論，不代表 PPT 已直接定義爬蟲或非同步爬取 API 的命名。

## Description

每個 operation 必須提供清楚的 `description`。移除空白、Tab 與換行後，字元數不得超過 1,000。

Description 應說明：

- API 用途。
- 主要輸入條件。
- 成功結果。
- 重要限制與可能的失敗情況。

對供 Agent 使用的 MCP API，建議依下列 Markdown 結構撰寫；其中用途、使用時機與輸出重點為必要內容，輸入／回傳範例及注意事項依 API 複雜度與來源規則判定：

```markdown
# **動詞(分類)** 功能描述
## 使用時機
### 輸入內容重點說明
### 輸出內容重點說明
### 輸入格式範例
### 回傳格式範例
### 注意
```

重大影響、不可重試或容易混淆的 API，必須在「注意」中說明限制。Description 應說明關鍵輸入、輸出欄位的型別與意義；若 APIM MCP 轉換無法保留 Object、String Array 或 Output Schema 的描述，必須在 operation description 補充。

## Schema 與 `$ref`

1. Request body、response body 與巢狀 object schema 必須定義於 `components/schemas`。
2. Operation、request、response 或其他 schema 不得直接定義 inline object schema。
3. Request 與 response 應透過 `$ref` 引用共用 schema。
4. 字串、數字、布林值等單純型別參數可以直接定義。
5. 解析全部 `$ref` 後，object 巢狀深度不得超過三層；最外層 object 為第一層。

若資料天然超過三層，應改用扁平結構、識別碼引用或拆分 API，並在 `plan` 說明理由。

### 來源差異待確認

PPT 第 23 頁要求巢狀物件以 `$ref` 定義，第 55 頁則限制 Request Body 為一層，第 56 頁記載 MCP Input 目前僅開放 String Array。這些規則的適用版本與優先順序尚未統一；在確認前，不得自行放寬或加嚴為新的全域規則。Request Body 出現 Array 時，仍應在 operation description 提供輸入重點與格式範例。

## 共通 MCP 契約

MCP request 至少應能表達：

- 支援的法規網站識別值。
- 搜尋月份。

法規種類與其他查詢條件由各網站 `spec.md` 定義。

成功 response 應能表達：

- `date`
- `sourceWebsite`
- `category`
- `title`
- `lawUrl`
- `ref`
- `metadata`
- 查詢結果總筆數

欄位的型別、required、格式與錯誤 schema 必須在實際 OpenAPI contract 中明確定義。

## 相容性與 Azure MCP

1. API 變更必須檢查既有 Agent 的相容性。
2. 破壞性變更必須有影響說明、遷移方案與測試。
3. 發布前必須依當時適用的 Azure MCP 規範檢查工具介面、輸入輸出與錯誤行為。
4. Azure MCP 規範若變更，應更新本文件、OpenAPI contract、功能 `spec`、測試與實作。

## 參數、Response 與錯誤

- 每個參數都必須明確提供 `type` 與 `description`，並區分 required 與 optional。
- 參數有格式要求時，description 必須提供該 API 實際採用的格式與範例；有限定值時，列出完整值、語意、轉換邏輯與超出範圍的處理方式。
- Object 與 String Array 的用途、欄位語意及輸入方式，除 schema description 外，應依 APIM MCP 限制在 operation description 的輸入重點補充。
- Response 欄位應使用清楚的業務語意，保持扁平且不超過三層；不得加入與單一任務無關的輸出。
- 錯誤回應除錯誤碼外，應包含自然語言訊息與可執行的修正方向。是否採用統一的 `code`、`msg`、`detail`、`data` 結構，屬進階規範，尚未列為所有 API 的強制要求。
- 每支 API 應聚焦單一業務任務；不要以 `action` 在同一 API 中切換多個獨立的建立、更新或刪除任務。

## 進階規範（選用）

以下內容來自 PPT 第 46–52 頁，目前是可選引用，不得視為所有 API 的預設契約：

- 輸入／回傳使用可驗證的真實測試資料，且禁止佔位符。
- 非同步操作說明 `jobId`、狀態查詢 API 與狀態判讀方式。
- 分頁說明參數、下一頁指示、取完資料的迴圈邏輯與單次上限。
- 寫入操作在 Description 中明確標示警示。

其中輪詢間隔、逾時分鐘數、分頁筆數等 PPT 示例值，不得直接當成專案既有行為。

## OpenAPI 發布檢查

發布前至少確認：

- OpenAPI 文件可以解析。
- Swagger UI 可以正常顯示所有 API。
- 每個 operation 有唯一且合規的 `operationId`。
- 每個 operation 有符合長度限制的 `description`。
- 沒有不符合規則的 inline object schema。
- object 巢狀深度不超過三層。
- 實際輸入輸出與 OpenAPI contract 一致。

## API 撰寫自我檢查

每支 API 的 OpenAPI contract、operation description 與 MCP 轉換結果，至少確認：

- [ ] `operationId` 唯一、使用 lowerCamelCase，且動詞符合六大業務分類。
- [ ] 名詞完整可理解，沒有不必要縮寫或模糊動詞。
- [ ] Description 使用 Markdown，移除空白、Tab 與換行後不超過 1,000 字。
- [ ] Description 說明用途、使用時機與不適用情境。
- [ ] Description 說明關鍵輸出欄位的型別與意義。
- [ ] 每個參數都有 `type`、`description`，並明確區分必填與選填。
- [ ] 格式或限定值已提供實際格式、完整值清單、範例與邊界處理。
- [ ] Request Body 有 Array、Object 或其他容易遺失描述的輸入時，已補充輸入重點；需要時提供輸入格式範例。
- [ ] Response 欄位語意清楚，結構不超過三層，且只包含此業務任務必要資料。
- [ ] 錯誤訊息提供自然語言與修正方向。
- [ ] 重大影響、禁止 Retry 或容易誤用的限制已寫入「注意」。
- [ ] 已解析 OpenAPI `$ref`、檢查 Swagger 顯示，並確認實作、契約與測試一致。
- [ ] 已檢查轉換出的 MCP Input Schema；可用時以 MCP Inspector 或 Postman 驗證。

來源差異（巢狀 Request Body、Object Array、進階範例要求）在正式確認前，應於審查紀錄標記「待確認」，不得以自我檢查清單自行裁定。
