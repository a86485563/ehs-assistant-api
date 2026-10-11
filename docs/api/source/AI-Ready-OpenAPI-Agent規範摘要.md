# AI-Ready OpenAPI 撰寫規範（依 PPT 整理）

## 來源與使用方式

來源為本機參考簡報（未納入版本控制），共 56 頁；本檔保留可共享的整理摘要。
`AI-Ready API 檢查清單.pptx` 因加密尚未讀取，未納入此版。

本文件僅整理 PPT 的規則與範例，不新增一般 OpenAPI 最佳實務。各節標示來源頁碼。「禁止／避免／建議」「必填／選填」依 PPT 保留，不自行升級為更嚴格要求。進階章節依第 46 頁標示為可選擇引用。PPT 不一致之處另列待確認，不由 Agent 自行裁定。

PPT 的示例值用於解釋寫法，不代表所有 API 都採相同格式、上限、預設值或錯誤碼。撰寫個別 API 時，不得把示例值當成該 API 已確認的行為。

## 1. OperationId 命名（第 11、47 頁）

規則：動詞（業務分類）＋名詞（業務物件）。

| 分類 | PPT 定義的情境 | PPT 列出的動詞 |
| --- | --- | --- |
| Acquire（獲取） | 主動從系統或資料庫中尋找並獲取所需的資訊 | search / query / check / retrieve / scan |
| Compute（運算） | 執行邏輯處理、數值運算或業務規則驗證 | summarize / validate / calculate / predict |
| Transact（交易） | 觸發具有狀態變更、涉及資金或資源流轉的動作 | submit / register / cancel / authorize |
| Integrate（整合） | 將不同系統、平台或服務進行連接與數據同步 | merge / sync / link / import / connect |
| Orchestrate（編排） | 執行複雜的多步驟任務，透過串聯多個 API 或工具達成大目標 | schedule / chain / retry / route |
| Notify（通知） | 將任務進度、異常狀況或結果主動傳遞給人類或其他系統 | notify / alert / broadcast / report |

- 動詞從業務動作出發，避免技術導向或多含意動詞。
- PPT 的不良動詞示例：`do`、`process`、`post`、`get`。
- PPT 的良好動詞示例：`submit`、`calculate`、`query`、`find`。
- 名詞不縮寫，應明確、直觀；公司內部專有名詞例外。
- 不良命名：`searchInv`、`getPO`、`searchData`。
- 良好命名：`searchInvoice`、`findPurchaseOrder`、`queryGoodsReceiptRecord`。

### 爬蟲 API 的分類說明（依定義推論，非 PPT 原例）

若用途是抓取網頁資訊供呼叫者讀取，對應 Acquire 的「獲取資訊」，因此可歸為 Acquire。`retrieveWebPageContent` 是依該規則推導的名稱，不是 PPT 提供的名稱。

PPT 沒有直接定義爬蟲 API、建立爬取工作或非同步爬取的分類，不應把這些情境額外寫成既定分類規則。

## 2. Description 結構（第 13–17、32 頁）

使用 Markdown，全文 1,000 字以內。

| 區段 | PPT 要求 | 撰寫內容 |
| --- | --- | --- |
| `# **動詞(分類)** 功能描述` | 必填 | 明確說明工具做什麼，採固定格式 |
| `## 使用時機` | 必填 | 說明何時用、何時不該用（負向情境） |
| `### 輸入內容重點說明` | 基礎結構列選填；特殊輸入另有補充要求 | Array、巢狀物件等特殊輸入說明，詳見第 3 節 |
| `### 輸出內容重點說明` | 必填 | 說明關鍵回傳欄位的型別與意義 |
| `### 輸入格式範例` | 主體列選填；第 32 頁另檢查 Request Body Array 是否提供 | Request Body 範例，協助 Agent 正確輸入 |
| `### 回傳格式範例` | 主體列選填 | 輸出多層嵌套或複雜時可提供，協助 Agent 解讀 |
| `### 注意` | 第 17 頁列選填；第 32 頁針對重大影響或易混淆 API 要求檢查 | 邊界條件、限制、副作用，重要限制用 `**` 強調 |

第 32 頁要求檢查重大影響或容易混淆的 API 是否在「注意」表述，例如不可 Retry、某情境禁止使用工具。

第 48、52 頁的進階規範將輸入／回傳範例列為必填且禁止佔位符。此要求保留在進階章節，不自行改成所有 API 的基礎要求。

## 3. 輸入重點與 APIM MCP 限制（第 14–17、23、32、55–56 頁）

以下為 PPT 對環境限制的記載，不延伸為所有版本的 MCP 通用規則。

### 3.1 Object 描述補充（第 14–15 頁）

Azure APIM MCP Server 無法將 Object 類型物件的 Description 帶到 MCP 的 Input Schema 中，因此需要在最上層 Operation Description 的「輸入內容重點說明」說明該參數的用法。

### 3.2 String Array 描述補充（第 14、16、32 頁）

Array 類型若存放 String，Array 與其中 String 的 Description 不會被完整帶入，因此需要在最上層補充輸入重點。

第 32 頁要求：Request Body 有 Array 類型參數時，檢查是否提供「輸入內容重點說明」及「輸入格式範例」。

PPT 第 14 頁的輸入說明示例：

```markdown
### 輸入內容重點說明:
- purchaseOrderNoList: string array，採購單號清單，採購單號格式: PO-YYYYMMDD-XXXXX
```

### 3.3 輸出描述補充（第 17 頁）

PPT 記載 APIM MCP Server 未提供 Output Schema 轉制，因此在「輸出內容重點說明」描述回傳欄位的型別及意義。

### 3.4 巢狀物件及 Array 的環境規則差異

- 第 23 頁：Nested Object 以 `$ref` 定義，不使用 inline，轉成 MCP 後才會正常顯示。
- 第 55 頁：Request Body 以一層為限，禁止多層嵌套。
- 第 56 頁：Request Body 目前僅開放 String Array，不提供 Object Array 作為 MCP Input；Array 需有輸入重點及範例。

前述頁面對巢狀輸入的支援範圍存在差異。本文件不自行指定哪一頁優先，也不新增「通過測試即可放寬」等規則。正式採用前需確認適用版本或頁面。

## 4. 參數設計 Best Practice（僅依 PPT）

### 4.1 命名具可讀性（第 20 頁）

| 等級 | PPT 示例 | 原因 |
| --- | --- | --- |
| 禁止 | `ref1`、`u`、`q`、`dt` | 無法辨識意義，Agent 容易填錯 |
| 避免 | `orderAmt`、`procDt`、`usrId` | 可猜但不確定，仍有誤解風險 |
| 建議 | `orderAmount`、`processDatetime`、`userId` | 語義完整，容易理解 |

不將 PPT 示例的大小寫風格額外提升為強制命名規則。

### 4.2 每個參數都有 description 與 type（第 21 頁）

PPT 將僅有 type、沒有 description 列為禁止；將描述過短且沒有範例列為避免；建議說明語義、範例及轉換指引。

禁止示例：

```yaml
site:
  type: string
```

避免示例：

```yaml
site:
  type: string
  description: 廠區
```

建議示例（PPT 內容）：

```yaml
site:
  type: string
  description: |
    廠區縮寫代碼，為英文與數字的組合。例：WZS（中山廠），若出現其他代碼請與用戶確認
```

### 4.3 有格式要求時，提供格式與範例（第 21–22 頁）

PPT 示例：

```yaml
createDatetime:
  type: string
  description: |
    發出Alert的時間點，套用格式: yyyy-MM-dd HH:mm:ss.sss。例如: 2026-03-03 12:51:21.100。
```

這是 PPT 的示例格式，不能自行改寫成 PPT 未要求的時區格式，也不能推定所有時間參數都使用此格式。

### 4.4 有限定值時，列出值、轉換邏輯及邊界處理（第 5、22 頁）

- 清楚描述該參數應填什麼，可提供範例。
- 有限定值時列出可用值，提供對應轉換邏輯。
- 說明不在清單內時應如何處置。
- 第 32 頁檢查表另要求有限定值的參數列出完整範例清單。

PPT 第 5 頁的示例：

> 訂單的篩選狀態。僅接受 'pending'（待處理）、'shipped'（已出貨）或 'cancelled'（已取消）。若用戶提到「還沒寄出的東西」，請帶入 'pending'。

此例示範如何把使用者語句對應到 API 接受的值。

### 4.5 清單過長時，可另提供轉換 API（第 22 頁）

若需陳列的清單超過百項，造成 Description 過長，可撰寫另一支 API，把使用者輸入轉換成合法輸入，再呼叫原工具。

PPT 用語為「可以」，本文件不升級為必須。

### 4.6 必填與選填明確區分（第 32 頁）

參數必填與選填需明確區分。PPT 此檢查項未進一步規定 default、nullable、空字串、空陣列或不同 Parameter Object 的寫法，因此不在此補成 PPT 的最佳實務。

### 4.7 Object／Array 依輸入重點規則補充（第 14–16、23、32 頁）

參數本身撰寫說明之外，依本文件第 3 節補充 Operation 層級的輸入重點與範例；巢狀物件的適用限制仍需確認。

## 5. Response 與錯誤處理（第 24–26 頁）

- 回應結構保持扁平，至多三層巢狀；避免不必要封裝及 Object 中相同參數命名造成混淆。
- 回應欄位與內容語義化，不使用難以理解的縮寫。
- 錯誤訊息包含自然語言及修正方向，不只提供錯誤碼或 `INVALID_PARAM`。

PPT 的錯誤修正方向示例：

```json
{
  "code": 40001,
  "detail": "site 代碼 WZQ不存在，有效代碼為 WZS/WHQ/WIH，請確認後重試"
}
```

第 29 頁補充：在流程允許的前提下，Agent 可以嘗試依錯誤訊息調整參數並重新呼叫。

## 6. API 設計原則（第 30 頁）

- 以業務邏輯為出發點，避免過多與任務無關的輸出。
- 以單一任務為導向，避免一個 API 包含過多功能。
- PPT 示例：將以 action 切換 Update／CREATE／DELETE 的 `manageUserProfile`，拆成 `updateUserProfile`、`createUserProfile`、`deleteUserProfile`。

## 7. 進階規範（第 46–52 頁，可選擇引用）

本節不自動納入基礎必填規範。

- 第 48 頁：輸入／回傳範例必填且禁止佔位符，使用測試環境中真實存在、可驗證的資料值。
- 第 49 頁：統一 `code`、`msg`、`detail`、`data` 回傳結構及業務錯誤碼；detail 使用自然語言與修正方向。
- 第 50 頁：非同步操作說明 jobId、狀態查詢 API 及 PENDING／RUNNING／SUCCESS／FAILED 的判讀方式。每 10 秒、超過 10 分鐘為該頁建議設定，不能當成既有 API 已實作的行為。
- 第 51 頁：分頁說明參數名稱及用法、下一頁指示欄位、取得全部資料的迴圈邏輯、單次回傳上限。50／200 等數字屬頁面示例。
- 第 52 頁：進階檢查表另列寫入操作在 Description 有警示標示。

## 8. 自我檢查（依第 32、37–38 頁）

- [ ] OperationId 動詞對應六大分類，名詞不縮寫（專有名詞除外），名稱可理解用途。
- [ ] Description 有 Markdown 結構，全文 1,000 字以內。
- [ ] 使用時機說明何時用、何時不用。
- [ ] 輸出重點說明關鍵欄位。
- [ ] Request Body 有 Array 時，提供輸入重點及輸入範例。
- [ ] 每個參數有 type 與 description。
- [ ] 限定值有完整範例清單；格式要求有格式與範例。
- [ ] 明確區分必填與選填。
- [ ] Response 不超過三層，欄位語義清楚。
- [ ] 錯誤訊息使用自然語言並提供修正方向。
- [ ] 多個 Object 嵌套的 $ref 寫法及其環境適用限制已釐清。
- [ ] 重大影響或容易混淆的 API 在「注意」表述限制。
- [ ] 檢查轉換出的 MCP Input Schema 是否符合預期。
- [ ] 使用 Postman 或 MCP Inspector 檢查 MCP 能否正常運行。

## 9. 尚未統一的來源差異

1. 輸入／回傳範例：主體為選填，進階為必填；Array 有額外要求。
2. 巢狀物件：第 23 頁要求 $ref，第 55 頁限制 Request Body 一層。
3. Array：第 56 頁僅開放 String Array，需確認其適用範圍。
4. 第二份加密檢查清單尚未納入。

以上保留為來源差異，不自行選定更嚴格版本，也不使用外部最佳實務補成內部規範。
