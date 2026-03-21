# 文件轉換腳本

這個目錄包含一些工具腳本，用來為 **MkDocs** 建置流程準備文件。

## 目的

為了同時在 GitHub 與託管文件站上提供良好的閱讀體驗，我們以 **GitHub-flavored Markdown** 作為主要來源格式。這個腳本會在建置流程中，把 GitHub 原生語法轉換成 **MkDocs 相容語法**（特別是 `pymdown-extensions` 所需格式）。

## 支援的轉換（單向）

這個腳本執行的是單向轉換：**GitHub Markdown → MkDocs Syntax**。

### Alert / Admonition 轉換

- GitHub 對 alerts 使用的是以 blockquote 為基礎的語法。
- MkDocs 則需要 `!!!` 或 `???` 語法，才能渲染彩色提示框。

## 執行轉換

這個轉換會作為建置流程的一部分自動執行，不需要額外步驟。如果你需要手動執行，可以在倉庫根目錄執行 `convert_docs.py` 腳本。

```bash
python docs/scripts/convert_docs.py
```

### 範例

- **來源（GitHub-flavored Markdown）：**
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
