# 定義你自己的 Catalog

雖然 [Basic Catalog](../../specification/v0_9/catalogs/basic/catalog.json) 很適合入門和引導應用啟動，但多數生產應用都會定義自己的 catalog，以反映自身的設計系統。

透過定義自己的 catalog，你可以把智慧體限制在應用中真實存在的元件和視覺語言范圍內，而不是讓它使用通用輸入框或按鈕。

## 為什么要定義自己的 Catalog？

每個 A2UI surface 都由一個 **Catalog** 驱動。Catalog 本質上是一個 JSON Schema 檔案，用來告訴智慧體可以使用哪些元件、函式和主題。

定義自己的 catalog 有以下好處：

- **與設計系統對齊**：限制智慧體只使用你的應用中真實存在的元件和視覺語言。
- **安全性與類型安全**：你在客戶端應用中注冊完整 catalog，確保只有可信元件會被渲染。
- **不需要 Mapper**：建議建構直接反映客戶端設計系統的 catalog，而不是嘗試透過適配器把通用 catalog（如 Basic Catalog）對映過去。

Basic Catalog 只是一個範例，並且刻意維持精簡，以便不同 renderer 都能輕松實作。

## 工作方式

1.  **定義 Catalog**：建立 catalog 定義（JSON Schema），列出應用支援的元件、函式和樣式。
2.  **注冊 Catalog**：在客戶端應用中注冊 catalog 及其對應的元件實作（renderer）。
3.  **宣告支援能力**：客戶端告知智慧體它支援哪些 catalog（透過 `supportedCatalogIds`）。
4.  **智慧體選擇 Catalog**：智慧體為給定 UI surface 選擇一個 catalog（透過建立訊息中的 `catalogId`，例如 `createSurface`）。
5.  **智慧體生成 UI**：智慧體使用該 catalog 中按名稱定義的元件來生成元件訊息。

## 實作指南

建議建立能直接對映到現有元件庫的 catalog。

=== "Web（Lit / Angular / React）"

    要在 Web 上實作自己的 catalog：

    - 建立包含元件定義的 JSON Schema。
    - 在所選 Web renderer 中建立自己的 `Component` 物件和 `Catalog` 物件。
        - 將 schema 或引用 ID 提供給智慧體。

    _各框架的詳細指南即將推出。_

=== "Flutter"

    要在 Flutter 中實作自己的 catalog：

    - 定義描述 widget 屬性的 JSON Schema。
    - 使用自訂 renderer 將 schema 對映到 Flutter widget。

    *詳細 Flutter 集成指南即將推出。*

## 安全注意事項

定義和注冊 catalog 時：

1.  **元件白名單**：只在 catalog 定義中注冊你信任的元件。除非有嚴格控制，否則不要暴露會提供危險能力的元件（例如執行任意腳本）。
2.  **驗證屬性**：始終驗證來自智慧體訊息的元件屬性，確保它們符合預期類型約束。
3.  **清理文本**：除非已經建立安全邊界，否則避免渲染未經清理的智慧體提供內容。

## 下一步

- **[主題與樣式](theming.md)**：自訂元件的外觀和體驗。
- **[元件參考](../reference/components.md)**：探索可能可複用的標準類型。
- **[Agent 開發](agent-development.md)**：建構會與你的 Catalog 互動的智慧體。
