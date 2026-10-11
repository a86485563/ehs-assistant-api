# Python Code Quality

## 文件定位

本文件是憲法「程式設計慣例與可維護性」方向的專門分支，承接 Python 命名、格式、lint、型別檢查、分層依賴與程式碼維護規範。

憲法只保留上位要求：程式碼應遵守一致且可檢查的慣例，保持清楚責任、單一目的與可維護性；本文件定義初期可執行的工具與規則。

## 工具標準

初期使用以下工具：

- **Ruff**：Python lint 與規則檢查。
- **Ruff format**：Python 程式碼格式化。
- **Pyright**：Python 型別檢查。
- **pytest**：行為、parser、資料品質與 API 契約測試。

工具版本由根目錄 `requirements.txt` 管理。若要替換工具，必須在 `plan` 中說明原因、影響與替代驗證方式。

## Python 命名慣例

- 函式與變數使用 `snake_case`。
- 類別使用 `PascalCase`。
- 常數使用 `UPPER_SNAKE_CASE`。
- 模組與套件名稱使用小寫，必要時使用底線。
- API 對外欄位依 OpenAPI/API governance 分支規範，不因 Python 內部命名慣例而任意改寫契約。
- 命名應表達實際責任，不使用模糊名稱掩蓋用途。

## 架構與依賴方向

- Controller 負責輸入驗證、流程呼叫與 response 轉換，不直接解析 HTML。
- Service 負責查詢流程協調，不直接承擔單一網站的 HTML 細節。
- Crawler/Parser 負責特定網站的資料取得與解析，不負責 API 路由或法規分類。
- 共用資料模型不得依賴單一網站 parser 的實作細節。
- 模組應維持清楚責任，避免為尚未確認的需求建立抽象。

## 可維護性要求

- 函式與模組應維持單一主要責任。
- 網站特殊規則集中於對應的 crawler/parser。
- 修改行為時必須同步更新相關測試與文件。
- lint、格式或型別檢查失敗時，不得宣稱變更完成。
- 不以註解掩蓋難以理解的設計；必要時應改善命名、分層或流程。
- 未執行或無法完成的品質檢查，必須如實記錄原因與影響。

## 驗證方式

變更完成前，至少應依適用範圍執行：

```text
ruff check .
ruff format --check .
pyright
pytest
```

各功能的 `plan` 與 `tasks` 可以依風險縮小或增加範圍，但不得省略必要驗證而宣稱完成。
