# 文檔轉換腳本

此目錄包含用於為 **MkDocs** 建構流程準備文檔的工具腳本。

## 用途

為了同時保證 GitHub 與托管站點上的閱讀體驗，專案使用 **GitHub-flavored Markdown** 作為主要事實來源。該腳本會在建構流水線中把 GitHub 原生語法轉換為 **MkDocs 相容語法**（特別是面向 `pymdown-extensions`）。

## 支援的轉換（單向）

該腳本執行單向轉換：**GitHub Markdown → MkDocs Syntax**。

### Alert/Admonition 轉換

腳本會處理以下轉換：

- GitHub 使用基於 blockquote 的語法來表示 alert。
- MkDocs 需要 `!!!` 或 `???` 語法來渲染彩色提示框。

## 執行轉換

轉換會作為建構流水線的一部分執行，不需要額外步驟。如果需要手動執行轉換，可以在倉庫根目錄執行 `convert_docs.py` 腳本。

```bash
python docs/scripts/convert_docs.py
```

### 範例

- **源檔案（GitHub-flavored Markdown）：**

    ```markdown
    > ⚠️ **Attention**
    >
    > This is an alert.
    ```

- **目標（MkDocs Syntax）：**

    ```markdown
    !!! warning "Attention"
    This is an alert.
    ```
