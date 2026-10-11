<!--
Sync Impact Report
版本：1.0.0 → 2.1.0
原因：依已確認的憲法討論草案重整主幹與分支邊界，移除不應位於憲法的 API、工具及短期實作細節。
追加修訂：補入專門規範與目前 feature spec 的直接承接索引，不改變核心原則。
保留：專案使命、責任邊界、資料治理、來源合規、品質驗證、效能可靠性、程式品質與文件治理。
下放：OpenAPI/Swagger/Azure MCP 的欄位細節、資料驗證細項、timeout/retry 參數、Python 執行指令、功能欄位與網站查詢規則。
正式憲章未裁定：資料疑義的隔離政策、來源規範衝突的最終核准角色、契約審查先後門檻、修訂核准角色。
受影響文件：docs/api/api-governance.md、docs/api/openapi-governance.md、docs/quality/data-validation.md、
docs/quality/performance-reliability.md、docs/quality/python-code-quality.md，以及各功能 spec/plan/tasks。
-->

# 環安小幫手專案憲章

## Project Mission and Domain Boundary

### 專案使命

環安小幫手為哥倫布 Agent 提供針對指定法規資料庫網站的 MCP 爬蟲 API。系統依 Agent 指定的法規種類與月份搜尋資料，忠實呈現來源內容及必要的來源資訊，讓 Agent 能在可信資料上進行後續分類工作。

### 系統責任

本專案負責：

- 依功能規格查詢指定法規網站、月份與種類。
- 取得、解析並依契約呈現來源資料。
- 提供來源、查詢範圍、資料總數與逐筆追溯資訊。
- 在連線失敗、格式異常、資料不完整或無法確認時明確回報異常。

目前規劃為一個 MCP API 對應一個法規資料庫網站。Agent 可指定法規種類與月份；月份的實際起訖日、時區、種類值與查詢參數由該網站的功能 `spec.md` 固定。例如查詢 9 月資料時，應依該網站規則涵蓋 9 月 1 日至 10 月 1 日前一天的資料。

所有網站 API 優先使用共通資料模型表達 `date`、`sourceWebsite`、`category`、`title`、`lawUrl`、`ref` 與資料總數；網站特有資料放入 `metadata`。各網站可在 `spec.md` 定義欄位細節，但不得省略來源、總數與逐筆追溯能力。

### 非責任與非目標

本專案不負責：

- 法規分類、分類規則執行或取代哥倫布 Agent 的判斷。
- 法律意見、法規效力判定、個案法律結論或法律責任判定。
- 使用者介面；使用者互動由哥倫布 Agent 承接，本專案專注於 MCP API。
- 翻寫、摘要、推論、補全或改變來源原文語意。
- 為尚未確認的需求預先建立不必要的功能、抽象或整合。

## Core Principles

### 一、技術與架構標準

系統應採清楚的責任分層，使輸入驗證、應用流程、網站擷取解析與基礎設施各自負責，避免網站特定規則污染共通契約或其他網站實作。

所有對外 API 必須遵守經批准、可解析、可驗證且與實作一致的介面契約。契約、文件、實作與測試的變更必須同步審查；破壞性變更必須說明影響與相容性處理。

OpenAPI、Swagger、Azure MCP、共通資料模型、schema 及 API 命名等可執行細節，由 [API Governance](../../docs/api/api-governance.md) 及其分支承接。單一網站的查詢參數、欄位映射與特殊規則由功能 `spec.md` 定義。

#### 現階段技術基準

以下是目前已確認的專案技術基準，不是單一功能的暫時偏好：

- Python `3.12.2`，由根目錄 `.python-version` 指定。
- FastAPI：非同步 API、Controller、OpenAPI 3.0 與 Swagger UI。
- Pydantic：請求與回應模型驗證。
- httpx：非同步外部 HTTP 連線與 timeout 控制。
- selectolax：HTML 解析與資料擷取。
- pytest：parser、資料正確性與 API 契約測試。
- Ruff 與 Ruff format：Python lint 與格式檢查。
- Pyright：Python 型別檢查。

依賴集中記錄於根目錄 `requirements.txt`，執行環境使用專案虛擬環境。若要替換 Python 版本、核心框架或主要套件，必須在 `plan.md` 說明原因、影響與相容性驗證，並同步更新專案設定與品質文件。

#### 架構與持久化邊界

- Presentation：Controller 接收 MCP/API 請求、驗證輸入並依契約回傳結果。
- Business/Application：Service 協調查詢流程；不得負責法規分類、摘要或法律判斷。
- Crawler/Parser：負責特定網站的資料取得與解析，不負責 API 路由。
- Infrastructure：提供 HTTP client、logging、設定與外部技術依賴。
- Persistence/Repository：目前不建立資料庫或 Repository；未來若需要保存查詢紀錄、執行狀態或結果，必須先經需求、`spec.md` 與 `plan.md` 確認。

API 與爬蟲初期採非同步處理，但目前不建立 Job API。若未來查詢時間、資料量或使用情境需要 Job 資源，必須另行定義契約與生命週期，不得由實作自行加入。

### 二、安全與資料保護

所有外部輸入都必須驗證。可查詢的網站必須來自專案明確支援與核准的來源，不得由輸入任意指定未支援或未核准的網站。

每次處理都應留下足以追蹤狀態的紀錄；紀錄不得洩漏秘密或不必要的敏感資料。若未來導入 token、認證資訊或持久化儲存，必須補充適當的遮罩、存取控制與保護措施。

具體輸入欄位、認證方式、log 欄位與異常分類由 API governance、品質分支及功能 `spec.md` 承接，不得降低本原則要求。

### 三、效能、可靠性與可觀測性

外部網站存取與資料處理必須具備適當的資源控制、失敗可追蹤性與可觀測性。系統不得因重試、非同步處理或擴充需求而違反來源規範、掩蓋資料異常或回報未完成結果為成功。

每次 MCP request 都應能追蹤查詢條件、處理狀態、成功結果與失敗原因。長時間或不穩定的工作不得阻塞一般請求；其處理方式、timeout、retry、頻率、backoff 與未來 Job 或持久化方案由 [效能可靠性分支](../../docs/quality/performance-reliability.md) 承接。

錯誤資訊應足以支援查核與除錯，但不得洩漏秘密或不必要的敏感資料。未驗證、失敗或不完整的結果不得以正常成功狀態提供。

### 四、程式品質與測試

新增或修改的行為必須有相應的驗證。法規資料只有在正確性、完整性、來源參照與契約一致性都可被驗證時，才能視為成功；驗證失敗或尚未執行時，必須如實記錄。

程式碼應遵守一致、可檢查且可維護的命名、格式、分層與責任邊界。品質檢查未通過或未執行時，不得宣稱變更完成。

跨 API、crawler/parser、資料模型與功能規格的驗證由 [資料驗證分支](../../docs/quality/data-validation.md) 承接；Python 命名、格式、型別、測試與維護性由 [Python 品質分支](../../docs/quality/python-code-quality.md) 承接。具體測試案例與驗收條件由功能 `spec.md`、`plan.md` 與 `tasks.md` 定義。

### 五、資料治理與合規

資料擷取、整理、轉換與輸出必須忠實保留來源意義，不得捏造、任意遺漏、扭曲、摘要或以推測取代來源資料。符合 API 契約不等同於資料內容正確；各功能必須定義可驗證的正確性與完整性條件。無法確認資料完整性時，不得將結果視為正常成功。

對外提供的資料必須能追溯至來源、取得時間與取得範圍；來源有版本或更新識別資訊時，也應保留該資訊。查詢結果至少應能辨識來源網站、查詢期間、資料總數及逐筆來源參照；具體欄位與 `ref` 格式由 API contract 與功能 `spec.md` 定義。

使用外部法規網站前，必須確認並遵守適用的 `robots.txt`、使用條款、官方 API、頻率限制及其他存取政策，不得以技術方式規避禁止或限制。來源規範不明、互相衝突或無法確認時，不得自行猜測為允許，應回報異常或交由適當角色確認。

## Development Workflow and Quality Gates

需求、規格、技術方案、任務、實作與驗證應依 Spec Kit 的文件分層與流程執行，且每項工作都必須能回溯至本憲章、功能 `spec.md`、技術 `plan.md`、任務 `tasks.md` 或明確的治理分支。

功能完成前至少必須確認：

- 資料符合功能定義的正確性與完整性條件。
- 資料可追溯至來源、查詢範圍與必要處理紀錄。
- 爬取行為符合來源規範與採用的限制。
- API contract、文件、實作與測試一致，且通過所屬 API governance 的檢查。
- 失敗、未完成、資料不完整或尚未執行的驗證都已如實記錄。
- 受影響的品質、效能、可觀測性與程式品質檢查已執行，或明確記錄未執行原因。

尚未執行或無法完成的驗證不得宣告為已通過。每次執行 task 時，應回報執行摘要與 task 數量，至少列出已完成、進行中、未完成或失敗的數量，並使回報與 `tasks.md` 保持一致。

## Governance

本憲章優先於個別功能規格、技術計畫、任務與實作。下游文件可以將主幹原則轉化為更具體、可執行及可驗證的規則，但不得降低或違反本憲章。

文件治理採「憲法主幹、專門規範分支、feature 文件下游」：

- 本憲章定義使命、責任邊界、長期原則與治理底線。
- API governance、品質與其他專門文件承接領域規則。
- `spec.md`、`plan.md`、`tasks.md`、OpenAPI contract 與測試承接功能需求、方案、工作與具體契約。
- 若下游規範與憲法衝突，必須先處理憲法或提出正式例外，不得由實作自行解釋。

### 分支承接索引

本憲章目前直接承接以下專門規範與功能文件：

- [API Governance](../../docs/api/api-governance.md)：API 共通契約、資料模型、輸入驗證與 API 分層。
- [OpenAPI Governance](../../docs/api/openapi-governance.md)：OpenAPI、Swagger、schema、命名、Azure MCP 與 API 自我檢查。
- [Crawler Data Validation](../../docs/quality/data-validation.md)：查詢完整性、總筆數、逐筆 `ref`、原文一致性與失敗判定。
- [Performance, Reliability and Scalability](../../docs/quality/performance-reliability.md)：timeout、retry、非同步處理、來源頻率與未來持久化。
- [Python Code Quality](../../docs/quality/python-code-quality.md)：Python 命名、格式、lint、型別檢查、測試與可維護性。
- [Monthly Law News Feature Spec](../../specs/001-query-monthly-law-news/spec.md)：月度法規查詢的功能範圍、來源條件與驗收要求。

各分支可再連結至其下游的 contract、plan、tasks 與測試；若文件尚未建立，不得在憲章中宣稱其已存在。

憲章修訂必須記錄變更內容、原因、受影響文件與驗證結果。尚未指定修訂核准角色前，不得假定 AI 可自行核准憲章修訂；每次修訂仍須由專案提出者明確授權。

當憲章內容開始包含單一 API、單一網站、短期工具或實作參數時，應先提出切分理由，將細節移至適當分支，並保留憲章與分支的引用關係。

憲章採語意化版本：

- MAJOR：移除或重新定義原則，造成不相容的治理要求。
- MINOR：新增原則、章節或實質擴充要求。
- PATCH：不改變規則意義的澄清、文字或錯字修正。

**Version**: 2.1.1 | **Ratified**: 2026-10-11 | **Last Amended**: 2026-10-11
