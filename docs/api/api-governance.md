# API Governance

## 文件定位

本文件是專案憲法的 API 治理入口，承接憲法中關於資料正確性、可追溯性、來源合規、API 契約與可驗證性的核心原則。

API 治理分成兩個專門分支：

- [OpenAPI Governance](openapi-governance.md)：OpenAPI 3.0、Swagger、schema、命名、相容性與 Azure MCP 規範。
- [Crawler Data Validation](../quality/data-validation.md)：跨 API 與 crawler 的查詢範圍、總筆數、分頁、逐筆 `ref`、原文一致性與失敗判定；其主要治理來源是憲法的原有方向七「測試與驗證」。

單一法規網站的查詢參數、HTML 結構、欄位映射與來源特殊規則，應由各功能 `spec.md` 定義，不得直接假設所有網站都相同。

## 共通治理要求

1. 每個 MCP API 都必須遵守經批准且可驗證的 API 契約。
2. 每個 request 的輸入參數都必須驗證；網站必須來自支援網站清單。
3. 成功結果必須使用共通資料模型表達來源網站、日期、種類、標題、法規連結、`ref` 與資料總數。
4. API 契約、實作、文件與測試必須保持一致。
5. 無法確認資料正確性、完整性或來源參照時，不得回傳為正常成功結果。
6. 每個 request 都必須留下可備查的執行紀錄，包括查詢條件、來源端點、月份、結果狀態、資料筆數與失敗原因。
7. Log 不得洩漏秘密或不必要的敏感資料；目前不建立資料庫，但紀錄格式應保留未來持久化的可能。

## 架構承接

共通 API 應維持清楚的分層：

- Presentation：Controller 負責輸入驗證與契約格式的 response。
- Business/Application：Service 控制查詢流程；crawler/parser 負責網站擷取與解析。
- Infrastructure：HTTP client、logging、設定與外部技術依賴。
- Persistence/Repository：目前不建立，只有在需求確認後才導入。

網站特有的解析規則應封裝在對應 crawler/parser，不得污染共通 API 契約或其他網站的實作。
