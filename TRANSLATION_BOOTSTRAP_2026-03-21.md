# 繁體中文翻譯完成記錄 2026-03-21

## 本次已完成

1. 建立 `docs-zh-hant/` 目錄，並從最新 `A2UI/docs` 複製文件骨架與資源檔。
2. 完整翻譯 `README.md`、`mkdocs.yaml` 與 `docs/**` 下的全部 Markdown 文件。
3. 保持與上游最新文件結構一致，包括 `concepts`、`guides`、`reference`、`ecosystem`、`specification` 與 `docs/scripts/README.md`。
4. 移除了從上游繼承的 `docs/CNAME`，避免未來部署時誤用 `a2ui.org` 網域。

## 說明

- 某些頁面保留了上游原文中的 `TODO`、實驗性說明與外部連結，這些內容只做語言翻譯，不改變其技術語意。
- `docs/specification/*.md` 仍沿用上游 `--8<--` include 形式；若後續要獨立建站，還需要補齊對應 include 檔來源。
- 如果後續要單獨發布，建議把此目錄初始化為獨立 Git 倉庫，並補上對應部署流程。
