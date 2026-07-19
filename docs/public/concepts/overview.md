# 核心概念

本節介紹 A2UI 的基礎架構。理解這些概念，將幫助你建構高品質的智慧體驅動介面。

關鍵術語的簡短定義請參見[詞彙表](glossary.md)。

## 全局視角

A2UI 建立在三個核心理念之上：

1. **串流訊息**：UI 更新以 JSON 訊息序列的方式，從智慧體流向客戶端
2. **宣告式元件**：UI 以資料描述，而不是以程式碼編寫
3. **資料繫結**：UI 結構與應用狀態彼此分離，從而支援反應式更新

## 關鍵主題

### [資料流](data-flow.md)
說明訊息如何從智慧體流向最終渲染的 UI。內容包含餐廳預約流程的完整生命週期示例、傳輸方式（SSE、WebSockets、A2A）、漸進式渲染，以及錯誤處理。

### [元件結構](components.md)
介紹 A2UI 用來表示元件階層的 **adjacency list model**。你將了解為什麼扁平列表比巢狀樹更適合這類場景、如何使用靜態與動態子元件，以及增量更新的最佳實務。

### [資料繫結](data-binding.md)
說明元件如何透過 JSON Pointer 路徑連接到應用狀態。內容涵蓋反應式元件、動態列表、輸入繫結，以及讓 A2UI 強大的「結構與狀態分離」設計。

## 訊息類型

=== "v0.9 (Stable)"

    - **`createSurface`**：建立新的 surface，並指定其 catalog
    - **`updateComponents`**：在 surface 中新增或更新 UI 元件
    - **`updateDataModel`**：更新應用狀態
    - **`deleteSurface`**：刪除一個 UI surface

    v0.9 明確將 surface 建立與渲染過程分離，`createSurface` 同時取代了 `beginRendering`，以及 `surfaceUpdate` 中隱含的 surface 建立邏輯。所有訊息都包含 `version` 欄位。

=== "v1.0 (Candidate)"

    Version 1.0 使用以下訊息類型：

    - **`createSurface`**：建立新的 surface，並指定其 catalog
    - **`updateComponents`**：在 surface 中新增或更新 UI 元件
    - **`updateDataModel`**：更新應用狀態
    - **`deleteSurface`**：刪除一個 UI surface
    - **`actionResponse`**：回應由客戶端發起的 action

    v1.0 新增了 `actionResponse` 訊息類型，讓客戶端到伺服端的同步 RPC 能力更加穩健。

=== "v0.8 (Legacy)"

    Version 0.8 使用以下訊息類型：

    - **`surfaceUpdate`**：定義或更新 UI 元件
    - **`dataModelUpdate`**：更新應用狀態
    - **`beginRendering`**：通知客戶端開始渲染
    - **`deleteSurface`**：刪除一個 UI surface

完整技術細節請參閱 [訊息參考](../reference/messages.md)。
