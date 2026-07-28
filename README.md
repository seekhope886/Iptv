# Iptv
A high-performance IPTV Player extension for MIT AI2

# [擴充功能] 萬能 IPTV 與 MPEG-TS 串流播放器 (com.luckyh9h.iptv)

本組件是專為 MIT App Inventor 2、Kodular 與 Niotron 開發的高效能 IPTV 電視直播擴充功能。採用**「智慧雙引擎自適應架構」**，能自動辨識串流格式：當偵測到 `.m3u8` (HLS) 或 `.mp4` 長影片時，會直接調用 Android 系統底層晶片進行原生硬體解碼；當接收到傳統 `.ts` 廣播串流時，則無縫切換至基於 `mpegts.js` 的 MSE 網頁轉碼核心。

主程式內建記憶體動態重組技術與 `autoCleanupSourceBuffer` 機制，**徹底粉碎了同類擴充功能播放 `.ts` 源時「5秒必卡死凍結」的技術通病**，並完美開通了無視 Android 系統鎖定的滿版全螢幕控制通道。

## 🛠️ 基本資訊
* **組件包名（Package Name）：** `com.luckyh9h.iptv`
* **內部類別名（Class Name）：** `Iptv`
* **元件版本：** `2.0`
* **組件分類：** Extension（擴充功能）
* **非可視組件：** 是（設為 Non-visible 確保在 AI2 網頁編輯器中拖拉 100% 絕不跳出 UmbrellaException 崩潰）
* **最低支援系統：** Android API 19+ (KitKat) 以上

## 📦 內嵌資產檔案（Assets）
本擴充功能在編譯時已自動封裝以下檔案至沙盒中（Java 會在執行時自動於記憶體重組，100% 繞過 Android 10+ 本地 file:/// 路徑跨域安全性封鎖）：
* `player.html` - 內嵌 H5 播放視窗主體與 Console 雙向狀態同步通訊機制。
* `mpegts.min.js` - MPEG-TS 串流核心解碼引擎。

---

## 🧩 設計師屬性 (Designer Properties)
無。（完全改由 Blocks 積木動態配置，提供 App 最彈性的適配空間）。

---

## 🚀 方法功能積木 (SimpleFunctions)

### 🔴 `CreatePlayer`
初始化 HTML5 視訊環境，並動態將播放器 WebView 注入到 AI2 的介面容器中。
* **參數：** 
  * `layout` (*HVArrangement*) - 畫面上用來當作電視螢幕的「水平配置」或「垂直配置」容器。（播放器寬高會自動填滿該容器）。

### 🔴 `SetStreamUrl`
【預載預載機制】將 App 初始化取得的網址先灌入播放器暫存，此時畫面會優雅地蓋上預設封面海報且不發出聲音。直到使用者**用手指親自點擊手機上的播放器畫面**，影片才會在瞬間觸發開播。
* **參數：** 
  * `streamUrl` (*String*) - 電視台直播源網址（支援 `.m3u8`、`.ts` 或 `.mp4`）。

### 🔴 `PlayStream`
【強制立刻開播】擊碎一切海報與暫存狀態，強制播放器以最高優先權立刻解碼並自動撥放出聲音與畫面。
* **參數：** 
  * `streamUrl` (*String*) - 電視台直播源網址（支援 `.m3u8`、`.ts` 或 `.mp4`）。

### 🔴 `Stop`
停止目前的串流播放，並強制清空手機內的緩衝快取，以節省使用者手機流量。

### 🔴 `Destroy`
關閉並還原全螢幕狀態，銷毀底層網頁實例並徹底釋放影音晶片運存。在 App 關閉或使用者換台、返回時**務必呼叫**，能徹底杜絕記憶體洩漏與背景耗電。

---

## ⚡ 事件監聽積木 (SimpleEvents)

### 🟢 `OnReady`
當播放器核心網頁在背景編織、重組完成，準備好接收影音網址的瞬間觸發。

### 🟢 `OnPlayStarted`
當影片穿透防盜鏈，正式在螢幕上渲染出第一個直播畫面的瞬間觸發。

### 🟢 `OnPlayPaused`
當使用者點擊網頁原生控制條的「暫停」按鈕時觸發。

### 🟢 `OnFullscreenChanged`
當使用者點擊播放器右下角放大鈕，進入或退出手機滿版全螢幕時連動觸發。
* **參數：** 
  * `isFullscreen` (*Boolean*) - 進入全螢幕回傳 `true`，縮小還原回傳 `false`。

### 🟢 `OnTimeUpdated`
高頻率進度同步事件。影片播放時，每 0.2 秒會自動回傳目前播到了第幾秒，可用於製作精準的進度條。
* **參數：** 
  * `seconds` (*String*) - 目前播放的時間秒數（例如：`12.5`）。

### 🟢 `OnError`
當串流死鏈、連線逾時、伺服器跨域阻擋（CORS）或解碼器卡死時觸發，會主動回報真兇訊息。
* **參數：** 
  * `errorMessage` (*String*) - 手機底層噴出的具體英文錯誤日誌。

---

## 🧱 官方標準積木拼接範例 (Blocks Workflow)

```text
// 1. App 開屏時動態蓋出電視機並預載待命
當 Screen1.Initialize 執行時：
    呼叫 Iptv1.CreatePlayer(layout: HorizontalArrangement1)
    呼叫 Iptv1.SetStreamUrl(streamUrl: "https://example.com")

// 2. 當使用者在選台清單 ListView 點選頻道時，立刻強制轉台開播
當 list_channels.AfterPicking 執行時：
    呼叫 Iptv1.PlayStream(streamUrl: list_channels.Selection)

// 3. 按下返回鍵時徹底清理記憶體防背景偷跑
當 Screen1.BackPressed 執行時：
    呼叫 Iptv1.Destroy
    呼叫 關閉此畫面 (close screen)
```

---

# [Extension] IPTV & MPEG-TS Stream Player (com.luckyh9h.iptv)

A high-performance IPTV Player extension for MIT App Inventor 2, Kodular, and Niotron. Built with an **Adaptive Dual-Engine Architecture**, it seamlessly switches between Native Android Hardware Decoding for `.m3u8` (HLS) / `.mp4` streams and JavaScript-based MSE decoding via `mpegts.js` for traditional `.ts` live broadcasts. 

It completely resolves the notorious "5-second freezing bug" by integrating a real-time memory buffer auto-cleanup loop and features a native implementation for immersive immersive fullscreen mode.

## 🛠️ Information
* **Package Name:** `com.luckyh9h.iptv`
* **Class Name:** `Iptv`
* **Version:** `2.0`
* **Category:** Extension
* **Non-visible Component:** Yes (Safe from GWT `UmbrellaException` in Designer)
* **Minimum Android API:** 19+ (KitKat)

## 📦 Assets Required
The extension automatically cross-compiles and bundles the following assets in its sandbox:
* `player.html` - The web-based player container with state sync handlers.
* `mpegts.min.js` - The core MPEG-TS decoder library (automatically injected into memory via Java at runtime to bypass strict Android 10+ local file origin CORS security restrictions).

---

## 🧩 Designer Properties
None. (Configured dynamically via blocks to ensure absolute flexibility).

---

## 🚀 SimpleFunctions (Methods)

### 🔴 `CreatePlayer`
Initializes the HTML5 playback environment and dynamically injects the video engine container into an AI2 layout.
* **Parameters:** 
  * `layout` (*HVArrangement*) - The Horizontal or Vertical arrangement component serving as the TV screen container. (Will automatically scale to `MATCH_PARENT`).

### 🔴 `SetStreamUrl`
Pre-loads an IPTV stream URL into the player's memory without starting playback immediately. It will overlay a default placeholder/poster image. Playback triggers the exact second the user touches/taps the player container screen.
* **Parameters:** 
  * `streamUrl` (*String*) - The stream link (`.m3u8`, `.ts`, or `.mp4`).

### 🔴 `PlayStream`
Instantly bypasses all overlays and forces the player to start streaming the specified media asset immediately with absolute priority.
* **Parameters:** 
  * `streamUrl` (*String*) - The stream link (`.m3u8`, `.ts`, or `.mp4`).

### 🔴 `Stop`
Stops the current media playback and clears the active streaming cache from memory to save bandwidth.

### 🔴 `Destroy`
Gracefully exits fullscreen mode, destroys the internal WebView instance, and releases all video hardware resources to prevent memory leaks during channel surfing.

---

## ⚡ SimpleEvents

### 🟢 `OnReady`
Triggered the exact moment the internal HTML5 container is fully compiled in memory and ready to receive stream assets.

### 🟢 `OnPlayStarted`
Triggered when the video stream officially starts rendering frames on the screen.

### 🟢 `OnPlayPaused`
Triggered when the user pauses the playback via the native media controller overlay.

### 🟢 `OnFullscreenChanged`
Triggered when the screen enters or exits the native full-bleed fullscreen view.
* **Parameters:** 
  * `isFullscreen` (*Boolean*) - Returns `true` if full-bleed full-screen mode is enabled; otherwise `false`.

### 🟢 `OnTimeUpdated`
High-frequency sync event that returns the current playback progress of the video asset.
* **Parameters:** 
  * `seconds` (*String*) - The current elapsed time formatted to one decimal place (e.g., `12.5`).

### 🟢 `OnError`
Triggered when an unexpected infrastructure anomaly occurs (e.g., dead streams, CORS blocks, or engine initialization failures).
* **Parameters:** 
  * `errorMessage` (*String*) - The exact error logs generated by the player.

---

## 🧱 Best Practice Block Implementation (Example Workflow)

```text
// 1. Setup the TV Screen and Preload on App Startup
When Screen1.Initialize do:
    Call Iptv1.CreatePlayer(layout: HorizontalArrangement1)
    Call Iptv1.SetStreamUrl(streamUrl: "https://example.com")

// 2. Immediate Channel Surfing via ListView
When list_channels.AfterPicking do:
    Call Iptv1.PlayStream(streamUrl: list_channels.Selection)

// 3. Memory Cleanup on Exit
When Screen1.BackPressed do:
    Call Iptv1.Destroy
    Close Screen
```
