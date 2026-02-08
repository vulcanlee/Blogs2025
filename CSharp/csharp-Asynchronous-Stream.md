# 使用非同步數據流 Asynchronous Stream 來處理迭代工作

C# 異步數據流（Async Streams）是 C# 8.0 引入的強大特性，利用 IAsyncEnumerable<T> 和 await foreach 語句，允許以非阻塞方式產生和消費異步序列。它非常適合處理分頁 API、感測器數據或長時間運行的任務，能在資料產生時即時處理，而無需等待全部資料載入。 
關鍵概念與實作

* 返回類型 (IAsyncEnumerable<T>)：異步數據流的方法返回 IAsyncEnumerable<T>，這允許方法使用 yield return 異步產生資料。
* 產生資料 (await foreach)：使用 async 和 yield return 的方法可逐步產生數據。
* 消耗資料 (await foreach)：呼叫者使用 await foreach 遍歷序列，在讀取下一個元素時暫停執行。 

## 建立測試專案

請依照底下的操作，建立起這篇文章需要用到的練習專案

* 打開 Visual Studio 2026 IDE 應用程式
* 從 [Visual Studio 2026] 對話窗中，點選右下方的 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗右半部
  * 切換 [所有語言 (L)] 下拉選單控制項為 [C#]
  * 切換 [所有專案類型 (T)] 下拉選單控制項為 [主控台]
* 在中間的專案範本清單中，找到並且點選 [主控台應用程式] 專案範本選項
  > 專案，用於建立可在 Windows、Linux 及 macOS 於 .NET 執行的命令列應用程式
* 點選右下角的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗
* 找到 [專案名稱] 欄位，輸入 `csAsynchronousStream` 作為專案名稱
* 在剛剛輸入的 [專案名稱] 欄位下方，確認沒有勾選 [將解決方案與專案至於相同目錄中] 這個檢查盒控制項
* 點選右下角的 [下一步] 按鈕
* 現在將會看到 [其他資訊] 對話窗
* 在 [架構] 欄位中，請選擇最新的開發框架，這裡選擇的 [架構] 是 : `.NET 10.0 (長期支援)`
* 在這個練習中，需要去勾選 [不要使用最上層陳述式(T)] 這個檢查盒控制項
  > 這裡的這個操作，可以由讀者自行決定是否要勾選這個檢查盒控制項
* 請點選右下角的 [建立] 按鈕

稍微等候一下，這個 背景工作服務 專案將會建立完成

## 修改 Program.cs 類別內容

* 在專案中找到並且打開 [Program.cs] 檔案
* 將底下的程式碼取代掉 `Program.cs` 檔案中內容

```csharp
using System.Diagnostics;

namespace csAsynchronousStream;

internal class Program
{
    static async Task Main(string[] args)
    {
        var sw = Stopwatch.StartNew();

        Log("=== 非同步串流 Asynchronous Stream 的比較展示 ===");

        await Demo_Stream(sw);
        Console.WriteLine();
        Console.WriteLine();
        Console.WriteLine();
        await Demo_Batch(sw);

        Log("=== 展示結束 ===");
    }

    static void Log(string msg) => Console.WriteLine(msg);

    static void Log(Stopwatch sw, string msg)
        => Console.WriteLine($"{sw.ElapsedMilliseconds,5}ms | {msg}");

    static async Task Demo_Stream(Stopwatch sw)
    {
        Log(sw, "[Stream 串流] 使用 IAsyncEnumerable");

        await foreach (var i in RangeAsync(start: 1, count: 5, delayMs: 300, sw))
        {
            Log(sw, $"[Stream 串流] 接收到 {i} 迭代請求 -> 開始進行處理 ...");
            await Task.Delay(200); // 模擬呼叫端處理耗時
            Log(sw, $"[Stream 串流] 已經處理完成 {i} 請求 -> 請求下一筆");
        }

        Log(sw, "[Stream 串流] 結束");
    }

    static async Task Demo_Batch(Stopwatch sw)
    {
        Log(sw, "[Batch 批次] 開始進行等候 Task<List<int>> (需要等待所有的迭代都完成後，才會繼續往下處理)");

        var list = await RangeTaskAsync(start: 1, count: 5, delayMs: 300, sw);

        Log(sw, $"[Batch 批次] 準備進行處理所有的迭代工作 (count={list.Count}) -> 開始進行處理所有工作...");
        foreach (var i in list)
        {
            Log(sw, $"[Batch 批次] 正在處理 {i} 個工作...");
            await Task.Delay(200); // 模擬呼叫端處理耗時
            Log(sw, $"[Batch 批次] 已經處理完成 {i} 個工作");
        }

        Log(sw, "[Batch 批次] 結束");
    }

    static async IAsyncEnumerable<int> RangeAsync(int start, int count, int delayMs, Stopwatch sw)
    {
        for (int i = start; i < start + count; i++)
        {
            Log(sw, $"[Stream 串流] 準備要產生迭代工作 {i}...");
            await Task.Delay(delayMs);                 // 模擬「取得下一筆資料」耗時
            Log(sw, $"[Stream 串流] 產生出結果給呼叫端 {i}");
            yield return i;                             // 交付給呼叫端，呼叫端可立刻處理
        }
    }

    static async Task<List<int>> RangeTaskAsync(int start, int count, int delayMs, Stopwatch sw)
    {
        var data = new List<int>();

        for (int i = start; i < start + count; i++)
        {
            Log(sw, $"[Batch 批次] 準備要產生迭代工作 {i}...");
            await Task.Delay(delayMs);                 // 模擬「取得下一筆資料」耗時
            data.Add(i);                               // 先收集起來
            Log(sw, $"[Batch 批次] 產生結果集合 {i} (此時將還不會回傳)");
        }

        Log(sw, "[Batch 批次] 全部都處理完成，並且回傳結果");
        return data;
    }
}
```



## 執行程式碼

* 按下 `F5` 鍵，開始執行這個程式
* 程式將會開始執行，並且在主控台視窗內，將會看到類似下圖的輸出結果

```
=== 非同步串流 Asynchronous Stream 的比較展示 ===
    5ms | [Stream 串流] 使用 IAsyncEnumerable
    6ms | [Stream 串流] 準備要產生迭代工作 1...
  316ms | [Stream 串流] 產生出結果給呼叫端 1
  316ms | [Stream 串流] 接收到 1 迭代請求 -> 開始進行處理 ...
  517ms | [Stream 串流] 已經處理完成 1 請求 -> 請求下一筆
  517ms | [Stream 串流] 準備要產生迭代工作 2...
  818ms | [Stream 串流] 產生出結果給呼叫端 2
  818ms | [Stream 串流] 接收到 2 迭代請求 -> 開始進行處理 ...
 1018ms | [Stream 串流] 已經處理完成 2 請求 -> 請求下一筆
 1018ms | [Stream 串流] 準備要產生迭代工作 3...
 1318ms | [Stream 串流] 產生出結果給呼叫端 3
 1318ms | [Stream 串流] 接收到 3 迭代請求 -> 開始進行處理 ...
 1534ms | [Stream 串流] 已經處理完成 3 請求 -> 請求下一筆
 1534ms | [Stream 串流] 準備要產生迭代工作 4...
 1835ms | [Stream 串流] 產生出結果給呼叫端 4
 1836ms | [Stream 串流] 接收到 4 迭代請求 -> 開始進行處理 ...
 2036ms | [Stream 串流] 已經處理完成 4 請求 -> 請求下一筆
 2036ms | [Stream 串流] 準備要產生迭代工作 5...
 2352ms | [Stream 串流] 產生出結果給呼叫端 5
 2352ms | [Stream 串流] 接收到 5 迭代請求 -> 開始進行處理 ...
 2569ms | [Stream 串流] 已經處理完成 5 請求 -> 請求下一筆
 2571ms | [Stream 串流] 結束



 2574ms | [Batch 批次] 開始進行等候 Task<List<int>> (需要等待所有的迭代都完成後，才會繼續往下處理)
 2575ms | [Batch 批次] 準備要產生迭代工作 1...
 2886ms | [Batch 批次] 產生結果集合 1 (此時將還不會回傳)
 2886ms | [Batch 批次] 準備要產生迭代工作 2...
 3187ms | [Batch 批次] 產生結果集合 2 (此時將還不會回傳)
 3187ms | [Batch 批次] 準備要產生迭代工作 3...
 3503ms | [Batch 批次] 產生結果集合 3 (此時將還不會回傳)
 3503ms | [Batch 批次] 準備要產生迭代工作 4...
 3805ms | [Batch 批次] 產生結果集合 4 (此時將還不會回傳)
 3805ms | [Batch 批次] 準備要產生迭代工作 5...
 4122ms | [Batch 批次] 產生結果集合 5 (此時將還不會回傳)
 4123ms | [Batch 批次] 全部都處理完成，並且回傳結果
 4125ms | [Batch 批次] 準備進行處理所有的迭代工作 (count=5) -> 開始進行處理所有工作...
 4126ms | [Batch 批次] 正在處理 1 個工作...
 4337ms | [Batch 批次] 已經處理完成 1 個工作
 4337ms | [Batch 批次] 正在處理 2 個工作...
 4538ms | [Batch 批次] 已經處理完成 2 個工作
 4539ms | [Batch 批次] 正在處理 3 個工作...
 4751ms | [Batch 批次] 已經處理完成 3 個工作
 4752ms | [Batch 批次] 正在處理 4 個工作...
 4953ms | [Batch 批次] 已經處理完成 4 個工作
 4954ms | [Batch 批次] 正在處理 5 個工作...
 5156ms | [Batch 批次] 已經處理完成 5 個工作
 5157ms | [Batch 批次] 結束
=== 展示結束 ===
```
