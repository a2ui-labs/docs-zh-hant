# A2UI Composer

透過 **A2UI Composer**，以互動方式建構 A2UI 元件。
![A2UI Composer 工作區](../assets/composer_workspace.png)

1. 開啟 [A2UI Composer](https://a2ui-project.github.io/composer/)

2. 預設情況下，你使用的是由 Angular renderer 提供的 Basic catalog

3. 用 Gemini 聊天開始建構 A2UI 介面吧！（**注意**：這需要 Gemini API
   金鑰，參見[下文](#gemini-api)。）

## 使用 Composer 介面

Composer 工作區分為多個可互動的面板，用來協助開發與偵錯 A2UI surface：

- **Gemini 助理：** 由 Gemini 驅動的聊天介面。你可以用自然語言提示詞，請 Gemini
  產生新的版面配置、調整現有 JSON，或修改視覺屬性。
- 點擊迴紋針圖示
  ![迴紋針圖示](../assets/composer_paperclip.png){style="width:30px;height:30px;display:inline;vertical-align:middle;"}
  可以**上傳附件**（例如一張設計稿）。
- 點擊相機圖示
  ![相機圖示](../assets/composer_camera.png){style="width:30px;height:30px;display:inline;vertical-align:middle;"}
  可以**附上目前 A2UI 介面的螢幕截圖**。

- **渲染後的 A2UI 預覽：** 即時顯示目前元件的視覺化預覽。

- **A2UI JSON 編輯器：** 顯示定義 UI 結構與元件階層的原始 JSON payload。你可以直接
  在這裡編輯，預覽會立即更新。
- 編輯器支援**[滑鼠停留提示](../assets/composer_editor_tooltip.png)**，會顯示對應
  A2UI 元素的說明。
- 如果你從其他來源取得 A2UI JSON，可以直接貼到這個面板。需要的話，按右鍵即可
  格式化 JSON。

- **偵錯與檢查頁籤（底部）：**
    - **Data Model：** 檢查並修改繫結到 UI 元件上的執行期狀態／資料值。在這裡的
      變更會傳播到預覽中，而預覽中的使用者輸入也會更新這個模型。
    - **Events：** 記錄已渲染元件所派送的使用者互動事件（例如點擊、輸入或選取）。
    - **Errors：** 顯示渲染、JSON 解析或 API 呼叫失敗所產生的錯誤。
    - **Raw Messages：** 顯示 Composer 與 renderer 之間的通訊，以及與 Gemini 的
      互動。（詳見[下文](#raw-messages)。）

## 元件庫

![展示某個元件的 A2UI 用法與屬性的螢幕截圖](../assets/composer_components_gallery.png)

元件庫讓你可以瀏覽目前 A2UI catalog 中提供的所有元件。每個元件都有一個渲染範例，
以及產生該渲染結果的 A2UI JSON。點擊 Usage 面板頂端的複製圖示
![複製圖示](../assets/composer_copy.png){style="width:30px;height:30px;display:inline;vertical-align:middle;"}
即可複製該元件完整的 A2UI JSON。

頁面底部有一張表格，列出該元件的所有屬性，包含說明、型別，以及該屬性是否必填。

## 設定

### renderer 應用程式

在設定頁面可以更換所使用的 renderer 應用程式。目前預先載入了三個 renderer
應用程式：

- Angular Basic Catalog
- Lit Basic Catalog
- React Basic Catalog

如果你正在開發其他 renderer 應用程式，可以在下拉選單中選擇「Custom」，並在文字
方塊中填入 URL。（關於如何建構 renderer 應用程式，參見
[A2UI Composer 整合](./composer_renderer_integration.md)。）

### Gemini API 金鑰 {#gemini-api}

同樣在這個頁面可以填入 Gemini API 金鑰，以啟用 Gemini 聊天功能。

取得 API 金鑰的步驟：

1. 前往 [Google AI Studio](https://aistudio.google.com/api-keys) 並以你的 Google
   帳號登入。
2. 點擊 Create API key。
3. 依提示選擇或新建一個 Google Cloud 專案，然後點擊 Create key。
4. 請把金鑰保存在安全的地方！

請注意，A2UI Composer 會加密該金鑰，並使用
[Web Crypto API](https://developer.mozilla.org/zh-TW/docs/Web/API/Web_Crypto_API)
把它儲存在瀏覽器本機的安全資料庫中。包含 Google 在內，任何人都無法存取該金鑰。

## 進行中的工作

A2UI 團隊正在積極推動以下改進：

- **降低延遲：** 改善 Gemini 輔助工作流程中的延遲。
- **視覺理解：** 改進 Gemini 輔助工作流程，提升其對已渲染 surface 的視覺理解能力。

## Raw Messages

Composer 與 renderer 之間往來的訊息有助於排查問題，也能幫助你更了解內部發生了
什麼事。這些訊息包括：

- **RENDERER_READY**：renderer 完成啟動程序後送出。
- **A2UI_CATALOG**：由 renderer 送出（應 Composer 的要求），包含該 renderer 支援
  的完整 A2UI catalog。
- **COMPONENT_USAGES**：由 renderer 送出（應 Composer 的要求），包含支撐元件庫
  頁面所需的資料。

每當 A2UI 元件的 data model 發生變更時，都會記錄一則 **DATA_MODEL_CHANGE** 訊息，
顯示將要回傳給 agent 的 `updateDataModel` 訊息。

此外，在使用 Gemini 聊天功能時：

- **LLM_REQUEST**：顯示送往 LLM 的完整請求（含系統提示詞）。
- **LLM_RESPONSE**：顯示從 LLM 收到的完整回應。
