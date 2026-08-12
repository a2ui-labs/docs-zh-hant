# A2UI Composer 整合手冊

## 背景

A2UI Composer 並不了解、也不與任何特定的 catalog 或 renderer 技術堆疊整合。為了看到
A2UI JSON 被渲染出來的結果，它依賴一個「renderer 應用程式」。A2UI Composer 會把
renderer 應用程式放在 iframe 中託管，並透過 **postMessage** 與其通訊。託管 renderer
應用程式的 iframe 帶有 `sandbox='allow-scripts allow-same-origin allow-forms'`，
對單純渲染 A2UI JSON 而言不會造成問題。

renderer 應用程式負責接收 A2UI JSON，並用自己的 renderer 把結果顯示出來。

## 橋接層（Bridge）

為了簡化與 A2UI Composer 的整合工作，我們提供了一個**橋接層**。它是一小段
JavaScript 程式碼，由 renderer 應用程式引入，負責協調 A2UI Composer 與 renderer
應用程式之間的所有通訊。

為了讓整合更加簡單，我們還提供了針對不同框架的封裝。

## 範例

可以參考這些範例 renderer 應用程式：

- [Angular](https://github.com/a2ui-project/composer/tree/main/samples/ng-basic-catalog)
- [Lit](https://github.com/a2ui-project/composer/tree/main/samples/lit-basic-catalog)
- [React](https://github.com/a2ui-project/composer/tree/main/samples/react-basic-catalog)

它們提供的都是同一版本的 Basic catalog。

執行託管版的 [A2UI Composer](https://a2ui-project.github.io/composer/) 時，進入設定
頁面並點擊 Renderer 下拉選單，就能看到這些 renderer 應用程式的實際運作效果。

### 使用 Angular

以下以 Angular 為例，示範如何建立一個 renderer 應用程式。

### 加入相依套件

首先，把核心整合套件加入專案的相依列表：

```
yarn add a2ui-bridge @a2ui/web_core @a2ui/angular
```

當然，如果你使用的是其他套件管理器，依相應方式操作即可。

#### 建立包裝元件

建立一個如下所示的新元件：

```ts
import {Component, inject} from '@angular/core';
import {SurfaceComponent} from '@a2ui/angular/v0_9';
import {A2uiSandboxConnection} from 'a2ui-bridge/angular';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [SurfaceComponent],
  template: `
    <main class="sandbox-shell">
      @if (sandbox.surfaceId()) {
        <a2ui-v09-surface [surfaceId]="sandbox.surfaceId()" />
      } @else {
        <p style="padding: 24px; color: #666; font-family: sans-serif; text-align: center;">
          Waiting for RENDER_A2UI payloads...
        </p>
      }
    </main>
  `,
})
export class AppComponent {
  protected sandbox = inject(A2uiSandboxConnection);
}
```

注意，`@else` 中的內容可以依需求修改，但範本的其餘部分應維持原樣。

#### 啟動你的 renderer 應用程式

設定標準的 Angular 啟動進入點檔案（`src/main.ts`），把你的 catalog 類別動態傳給
sandbox 的 provider 對應：

```ts
import {bootstrapApplication} from '@angular/platform-browser';
import {AppComponent} from './app/app.component';
import {provideA2uiSandbox} from 'a2ui-bridge/angular';
import {BasicCatalog} from '@a2ui/angular/v0_9';

bootstrapApplication(AppComponent, {
  providers: [
    provideA2uiSandbox([BasicCatalog]), // Injects and exposes dynamic catalogs
  ],
}).catch((err) => console.error('A2UI Sandbox Bootstrap Failed:', err));
```

記得把 `BasicCatalog` 換成你自己的 catalog。

> 關於變更偵測相容性的說明：`provideA2uiSandbox` 輔助函式開箱即用地 100% 相容
> 標準的 Zone 變更偵測（使用 `zone.js`）與無 Zone 的變更偵測
> （`provideZonelessChangeDetection()`）。
