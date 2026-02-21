# 在 ASP.NET Core 框架下，升級 NLog 5.5 到 NLog 6.1.1 版本需要做的修正紀錄

在上一篇文章， [在 ASP.NET Core 框架下，如何使用 NLog 5.5 進行日誌記錄](https://csharpkh.blogspot.com/2026/02/csharp-asp-net-core-blazor-nlog-template.html) 採用的 NLog 5.5 版本的套件，現在已經升級到 NLog 6.1.1 ，這裡可以參考 [https://nlog-project.org/2025/04/29/nlog-6-0-major-changes.html](https://nlog-project.org/2025/04/29/nlog-6-0-major-changes.html) 的說明，說明最新版本多了那些功能。

若之後開發或者想要將之前採用 NLog 6.x 之前的專案，升級到 NLog 6.x 套件下來使用，可以參考這篇文章，這裡會說明從 NLog 5.5 升級到 NLog 6.1.1 版本需要做的修正紀錄，包含套件版本升級、`nlog.config` 的異動說明，讓讀者可以清楚知道在升級過程中需要注意的事項。

## 套件版本升級

在這個專案內，`csNlog55.csproj` 的 [NLog.Web.AspNetCore] 套件版本使用的 `5.5.0`，現在需要將 [NLog.Web.AspNetCore] 使其升級到 `6.1.1` 版本，升級後會自動帶入相依套件 `NLog` (核心) 6.1.0、`NLog.Extensions.Logging` 6.1.1、`NLog.Web.AspNetCore` 6.1.1（含 `net10.0` 專屬 TFM 組件）。

| 套件 | 版本 |
|---|---|
| `NLog` (核心) | 6.1.0 |
| `NLog.Extensions.Logging` | 6.1.1 |
| `NLog.Web.AspNetCore` | 6.1.1（含 `net10.0` 專屬 TFM 組件）|

* 滑鼠右擊這個專案節點
* 將會看到這個專案定義檔案 [csNlog55.csproj] 
* 將這個 NuGet 套件參考， `<PackageReference Include="NLog.Web.AspNetCore" Version="5.5.0" />` 升級到 `<PackageReference Include="NLog.Web.AspNetCore" Version="6.1.1" />` 即可。

## nlog.config 異動

* 修正 XML Schema 命名空間 URL 改為 HTTPS
* 在專案根目錄下找到並且打開 [nlog.config]，將 XML Schema 命名空間 URL 從 `<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"` 改為 `<nlog xmlns="https://nlog-project.org/schemas/NLog.xsd"`，這是因為 NLog 6.x 官方將 Schema URL 從 `http://www.nlog-project.org/` 正式遷移至 `https://nlog-project.org/`（去掉 `www.`、改為 HTTPS）。舊 URL 已廢棄。此設定影響 IDE（如 Visual Studio）的 XML 自動補全與驗證功能，使用正確 URL 才能取得最新的屬性提示。

NLog 6.x 官方將 Schema URL 從 `http://www.nlog-project.org/` 正式遷移至 `https://nlog-project.org/`（去掉 `www.`、改為 HTTPS）。舊 URL 已廢棄。此設定影響 IDE（如 Visual Studio）的 XML 自動補全與驗證功能，使用正確 URL 才能取得最新的屬性提示。

* 在 NLog 5.x 就已將 [AsyncWrapper] 的 [optimizeBufferReuse] 標記為 [Obsolete]，NLog 6.0 正式從類別定義中刪除此屬性。原本的用途是讓各 Target 在 render layout 時重複利用同一塊記憶體緩衝區，以降低 GC 壓力。此優化已內建於引擎層，**自動啟用，不需任何設定**。
* 在 [nlog.config] 中，將 [AsyncWrapper] 的 [optimizeBufferReuse="true"] 移除即可。也就是這個標記 `optimizeBufferReuse="true"` 需要從 [AsyncWrapper] 的 target 定義中刪除。

## 異動 2 — 移除 `AsyncWrapper` 的 `optimizeBufferReuse`

```xml
<!-- 之前 -->
<target xsi:type="AsyncWrapper" name="file_async"
        overflowAction="Discard"
        queueLimit="10000"
        batchSize="1"
        timeToSleepBetweenBatches="50"
        optimizeBufferReuse="true"      ← 移除
        fullBatchSizeWriteLimit="5">

<!-- 之後 -->
<target xsi:type="AsyncWrapper" name="file_async"
        overflowAction="Discard"
        queueLimit="10000"
        batchSize="1"
        timeToSleepBetweenBatches="50"
        fullBatchSizeWriteLimit="5">
```

* 在 [nlog.config] 中，找到 [FileTarget] 的 target 定義，將 [concurrentWrites] 與 [enableArchiveFileCompression] 這兩個屬性移除即可，也就是找到 `concurrentWrites="false"` & `` 這兩個敘述，從 [nlog.config] 中刪除。NLog 6.0 重新設計了 [FileTarget] 的內部 Appender 架構，採用兩個明確分工的實作：
| Appender | 觸發條件 | 行為 |
|---|---|---|
| `ExclusiveFileLockingAppender` | `keepFileOpen="true"`（現為預設） | 程序獨佔開啟檔案，效能最高 |
| `MinimalFileLockingAppender` | `keepFileOpen="false"` | 每次寫入才開啟／關閉，允許外部工具同時讀取 |

* [concurrentWrites] 原本是用來啟用「多程序同時寫入同一個 log 檔」的 Mutex 鎖定機制。但這與 `keepFileOpen="true"` 本質矛盾（獨佔開啟就不可能讓其他程序同時寫入），本專案採用 `keepFileOpen="true"` 是正確選擇（單程序、高效能），移除 `concurrentWrites` 後行為完全不變。
，因此 NLog 6.0 直接廢除此屬性。
NLog 6.1 下要允許多程序共寫的做法：

```xml
<!-- 改用 keepFileOpen="false"，由 MinimalFileLockingAppender 處理 -->
<target name="file" xsi:type="File"
        keepFileOpen="false"
        fileName="..." />
```

* [enableArchiveFileCompression] 的移除原因是 NLog 6.0 將封存壓縮功能完整移除，理由有三：
1. `.NET 6+` 已提供完整的 `System.IO.Compression` API，NLog 不需重複實作
2. 壓縮操作屬於 CPU 密集型 I/O，放在 logging 路徑內會影響應用程式效能
3. 原本實作有跨平台相容性問題（Windows/Linux 行為不一致）

* 底下為修正前後的差異說明

```xml
<!-- 之前 -->
<target name="file" xsi:type="File"
        keepFileOpen="true"
        concurrentWrites="false"            ← 移除
        openFileCacheTimeout="30"
        ...
        archiveAboveSize="104857600"
        enableArchiveFileCompression="true" ← 移除 />

<!-- 之後 -->
<target name="file" xsi:type="File"
        keepFileOpen="true"
        openFileCacheTimeout="30"
        ...
        archiveAboveSize="104857600"/>
```

## 異動總覽對照表

| 項目 | NLog 5.5 | NLog 6.1 | 影響 |
|---|---|---|---|
| 套件版本 | `5.5.0` | `6.1.1` | 核心升級 |
| Schema URL | `http://www.nlog-...` | `https://nlog-...` | IDE 驗證／補全 |
| `optimizeBufferReuse` | 需手動設定 | **自動內建** | 無功能損失 |
| `concurrentWrites` | 可設定 | **由 `keepFileOpen` 決定** | 行為不變（本專案獨佔模式）|
| `enableArchiveFileCompression` | 內建 | **移除** | 需改用外部壓縮方案 |




