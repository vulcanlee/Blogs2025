# React API 04 : 呼叫一個 Get 方法 API，傳送與取得 Header 的數值

在 React 系列的第一篇文章中 [React API 01 : 前端使用 React，後端使用 ASP.NET Core 的測試標準專案](https://csharpkh.blogspot.com/2026/02/csharp-React-Web-Api.html) & [React API 02 : 呼叫一個 Get 方法 API，取得查詢字串內容並將結果渲染到網頁上](https://csharpkh.blogspot.com/2026/02/csharp-React-Get-Query-String.html) ，這裡透過建立一個 ASP.NET Core Web API 專案，來示範如何建立一個 GET 方法的 API 端點，在第一篇文章中，單純使用 HTTP GET 方法來呼叫 API，第二篇文章，雖然也是使用 HTTP GET 方法，但是透過查詢字串的方式，接收來自於前端 App 的參數，透過這些參數，進行相關客製處理，另外建立一個 React 專案，並且在前端的 React 應用程式中呼叫這個 API 端點，將查詢字串參數傳送給後端，並且將後端回傳的資料渲染到網頁上。

現在，我們來看看第三篇文章的內容，這裡同樣是建立一個 ASP.NET Core Web API 專案，並且在其中定義一個 GET 方法的 API 端點，不過這次我們將不會使用查詢字串參數的方式來接收來自於前端 App 的參數，而是使用路由方式來傳遞同樣的參數，然後在前端的 React 應用程式中呼叫這個 API 端點，將路由參數傳送給後端，並且將後端回傳的資料渲染到網頁上。透過這樣的說明，可以同時讓前端與後端的開發者知道如何使用路由參數來進行設計。

接下來就來嘗試看看這樣的需求如何做到吧！

## 建立 ASP.NET Core Web API 專案
* 開啟 Visual Studio 2026
* 選擇「建立新專案」
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇「ASP.NET Core Web API」 項目
  >此專案範本可用於使用 ASP.NET Core 控制器或最小 API 建立 RESTful Web API，並可選擇地支援 OpenAPI和驗證
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 [WebApiDemo]
* 然後點擊右下方「下一步」按鈕
* 接著會看到 [其他資訊] 對話窗
* 在這個對話窗內，確認使用底下的選項
    * 架構：.NET 10.0 (或更新版本)
    * 驗證類型：無
    * 勾選 針對 HTTPS 進行設定
    * 啟用 OpenAPI 支援
    * 勾選 不要使用最上層陳述式 (這是我的個人習慣)
    * 使用控制器
    * 不要勾選 在 .NET Aspire 協調流程中登錄
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 ASP.NET Core Web API 專案的建立

## 修改 API 端點的程式碼
* 在方案總管內，找到並展開 [Controllers] 資料夾
* 找到並打開 [WeatherForecastController.cs] 檔案
* 將該檔案內的程式碼全部刪除，然後將底下的程式碼貼上到該檔案內

```csharp
using Microsoft.AspNetCore.Mvc;

namespace WebApiDemo.Controllers;

[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [HttpGet("GetWeatherByLocationAndDate/{location}/{date}")]
    public ActionResult<IEnumerable<WeatherForecast>> 
        GetWeatherByLocationAndDateRoute([FromRoute] string location, [FromRoute] DateOnly date)
    {
        // 這裡可以根據地點與日期來產生不同的天氣預報
        return Ok(Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = date.AddDays(index),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)],
            Location = location
        }).ToArray());
    }
}
```

這裡定義了一個新的 API 端點 [GetWeatherByLocationAndDate]，這個端點使用了 `[HttpGet]` 的屬性來指定它是一個 GET 方法的 API，並且在路由中指定了 [GetWeatherByLocationAndDate] 的路徑。這個 API 端點接受兩個路由參數，分別是 [location] 與 [date]，並且使用 `[FromRoute]` 的屬性來告訴 ASP.NET Core 從路由中取得這些參數的值。

## 建立 React 專案
* 滑鼠右擊解決方案 [WebApiDemo]，選擇「加入」>「新增專案」
* 在 [加入新專案] 視窗中，在右方清單內，找到並選擇「React 個應用程式」 項目
  >請注意選擇具有底下的說明項目的專案範本
  >
  >TypeScript React 專案範本，透過執行 npx 的全域安裝來進行啟動載入
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 [reactdemo]
* 然後點擊右下方「下一步」按鈕
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 React 個應用程式 專案的建立
* 在方案總管內，將會看到有兩個專案建立起來

## 修改 React 專案的程式碼
* 在方案總管內，找到並展開 [reactdemo] 專案
* 找到並打開 [src] 資料夾內的 [App.tsx] 檔案
* 將該檔案內的程式碼全部刪除，然後將底下的程式碼貼上到該檔案內

```tsx
import { useState } from 'react'
import reactLogo from './assets/react.svg'
import viteLogo from '/vite.svg'
import './App.css'

// 定義 WeatherForecast 型別，對應後端的資料結構
interface WeatherForecast {
    date: string;
    temperatureC: number;
    temperatureF: number;
    summary: string;
    location: string;
}

function App() {
    const [count, setCount] = useState(0)
    const [forecasts, setForecasts] = useState<WeatherForecast[]>([])
    const [loading, setLoading] = useState(false)
    const [error, setError] = useState('')

    // 使用者輸入地點與日期
    const [location, setLocation] = useState('Kaohsiung')
    const [date, setDate] = useState(() => new Date().toISOString().split('T')[0]) // YYYY-MM-DD

    const apiBase = 'https://localhost:7074'

    // 呼叫後端 GetWeatherByLocationAndDate API
    const fetchWeatherByLocationAndDate = async () => {
        setLoading(true)
        setError('')
        setForecasts([])

        try {
            if (!location.trim()) {
                throw new Error('地點不可為空')
            }
            if (!/^\d{4}-\d{2}-\d{2}$/.test(date)) {
                throw new Error('日期格式需為 YYYY-MM-DD')
            }

            const query = new URLSearchParams({
                location: location.trim(),
                date: date
            }).toString()

            const response = await fetch(`${apiBase}/weatherforecast/GetWeatherByLocationAndDate/${location.trim()}/${date}`)

            if (!response.ok) {
                throw new Error(`API 請求失敗: ${response.status}`)
            }

            const data = await response.json() as WeatherForecast[]
            setForecasts(data)
        } catch (err) {
            setError(err instanceof Error ? err.message : '取得資料時發生錯誤')
            console.error('GetWeatherByLocationAndDate 呼叫失敗:', err)
        } finally {
            setLoading(false)
        }
    }

    return (
        <>
            <div>
                <a href="https://vite.dev" target="_blank">
                    <img src={viteLogo} className="logo" alt="Vite logo" />
                </a>
                <a href="https://react.dev" target="_blank">
                    <img src={reactLogo} className="logo react" alt="React logo" />
                </a>
            </div>
            <h1>Vite + React</h1>
            <div className="card">
                <button onClick={() => setCount((count) => count + 1)}>
                    count is {count}
                </button>
                <p>
                    Edit <code>src/App.tsx</code> and save to test HMR
                </p>
            </div>

            {/* 自訂查詢天氣預報 */}
            <div className="card" style={{ marginTop: '20px', width: '100%', maxWidth: 800 }}>
                <h2>查詢指定地點與日期的天氣預報</h2>

                <div style={{ display: 'flex', gap: '12px', flexWrap: 'wrap', marginBottom: '12px' }}>
                    <div style={{ display: 'flex', flexDirection: 'column' }}>
                        <label htmlFor="location">地點</label>
                        <input
                            id="location"
                            type="text"
                            value={location}
                            onChange={e => setLocation(e.target.value)}
                            placeholder="輸入地點"
                            style={{ padding: '6px 8px' }}
                        />
                    </div>
                    <div style={{ display: 'flex', flexDirection: 'column' }}>
                        <label htmlFor="date">日期 (YYYY-MM-DD)</label>
                        <input
                            id="date"
                            type="date"
                            value={date}
                            onChange={e => setDate(e.target.value)}
                            style={{ padding: '6px 8px' }}
                        />
                    </div>
                    <div style={{ alignSelf: 'flex-end' }}>
                        <button
                            onClick={fetchWeatherByLocationAndDate}
                            disabled={loading || !location.trim() || !/^\d{4}-\d{2}-\d{2}$/.test(date)}
                            style={{ padding: '8px 14px' }}
                        >
                            {loading ? '查詢中...' : '取得天氣'}
                        </button>
                    </div>
                </div>

                {error && <p style={{ color: 'red' }}>{error}</p>}

                {forecasts.length > 0 && (
                    <div style={{ marginTop: '12px', overflowX: 'auto' }}>
                        <table style={{ width: '100%', borderCollapse: 'collapse', minWidth: 600 }}>
                            <thead>
                                <tr>
                                    <th style={{ border: '1px solid #ddd', padding: '8px' }}>日期</th>
                                    <th style={{ border: '1px solid #ddd', padding: '8px' }}>地點</th>
                                    <th style={{ border: '1px solid #ddd', padding: '8px' }}>溫度 (C)</th>
                                    <th style={{ border: '1px solid #ddd', padding: '8px' }}>溫度 (F)</th>
                                    <th style={{ border: '1px solid #ddd', padding: '8px' }}>概況</th>
                                </tr>
                            </thead>
                            <tbody>
                                {forecasts.map((forecast, index) => (
                                    <tr key={index}>
                                        <td style={{ border: '1px solid #ddd', padding: '8px' }}>{forecast.date}</td>
                                        <td style={{ border: '1px solid #ddd', padding: '8px' }}>{forecast.location}</td>
                                        <td style={{ border: '1px solid #ddd', padding: '8px' }}>{forecast.temperatureC}</td>
                                        <td style={{ border: '1px solid #ddd', padding: '8px' }}>{forecast.temperatureF}</td>
                                        <td style={{ border: '1px solid #ddd', padding: '8px' }}>{forecast.summary}</td>
                                    </tr>
                                ))}
                            </tbody>
                        </table>
                    </div>
                )}

                {forecasts.length === 0 && !loading && !error && (
                    <p style={{ marginTop: '8px', color: '#666' }}>請輸入地點與日期後點擊「取得天氣」。</p>
                )}
            </div>

            <p className="read-the-docs">
                Click on the Vite and React logos to learn more
            </p>
        </>
    )
}

export default App
```


在這個 [App.tsx] 的程式碼中，定義了一個 `WeatherForecast` 的 TypeScript 介面，來對應後端 API 回傳的資料結構。接著，使用 React 的 `useState` Hook 來管理天氣預報資料、載入狀態和錯誤訊息。

這裡設定 [apiBase] 變數為後端 API 的基本 URL，這樣在呼叫 API 的時候就可以使用這個變數來組合完整的 API 端點 URL。另外，使用了 const response = await fetch(`${apiBase}/weatherforecast/GetWeatherByLocationAndDate/${location.trim()}/${date}`) 來呼叫後端的 API 端點，並且將地點與日期作為路由參數傳送給後端。

另外，定義了一個 `fetchWeatherForecast` 的非同步函數，來呼叫後端的 API 端點，並將回傳的資料存到 `forecasts` 的狀態中。如果在呼叫 API 的過程中發生錯誤，會將錯誤訊息存到 `error` 的狀態中。在 JSX 的部分，建立了一個按鈕，當使用者點擊時會呼叫 `fetchWeatherForecast` 函數來獲取天氣預報資料。當資料成功獲取後，會將資料以表格的形式顯示在畫面上。如果在獲取資料的過程中發生錯誤，我們會在畫面上顯示錯誤訊息。

另外，對於使用者要輸入的欄位，這裡使用了兩個 `useState` 來分別管理地點與日期的值，並且在按鈕被點擊時，將這些值作為查詢字串參數傳送給後端的 API 端點。接著，也做了輸入防呆檢查，確保地點不可為空，日期的格式必須為 YYYY-MM-DD，只有在這些條件都滿足的情況下，按鈕才會被啟用。

## 設定同時啟動多個專案
* 在這個方案內，擁有兩個專案，分別是 [WebApiDemo] 與 [reactdemo]
* 前者是 ASP.NET Core Web API 專案，後者是 React 的前端專案
* 因此，我們需要設定 Visual Studio 來同時啟動這兩個專案，才能在開發過程中同時測試前後端的功能
* 在方案總管內，右擊方案 [WebApiDemo]，選擇 [設定啟動專案]
* 在 [方案 'ReactWebApi' 屬性頁] 對話窗內，選擇 [多個啟動專案] 的選項
* 在下方的專案列表內，找到 [WebApiDemo] 與 [reactdemo] 這兩個專案
* 將這兩個專案的 [動作] 欄位都設定為 [啟動]
* 然後點擊右下方的 [確定] 按鈕，來儲存這個設定
![](../Images/cs2025-882.png)

## 修正 CORS 的問題
* 這個錯誤訊息表示了，因為 CORS 的政策限制，導致前端的 React 應用程式無法成功呼叫後端的 API 端點
* 為了修正這個問題，我們需要在後端的 ASP.NET Core Web API 專案中，加入對 CORS 的支援
* 打開 [Program.cs] 檔案
* 在該檔案內，找到 `builder.Services.AddOpenApi();` 這一行
* 在這一行的下方，加入底下的程式碼，來設定 CORS 的政策，允許來自 `http://localhost:49158` 的請求

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowReactApp", policy =>
    {
        policy.WithOrigins("http://localhost:49158") // React 應用運行在此端口
                .AllowAnyHeader()
                .AllowAnyMethod();
    });
});
```

* 這裡使用了 `AddCors` 方法來加入 CORS 的服務，並定義了一個名為 "AllowReactApp" 的政策，該政策允許來自 `http://localhost:49158` 的請求，並且允許任何標頭和方法(這裡使用這些方法 [AllowAnyHeader()] & [AllowAnyMethod()] 是為了簡化測試，實際上在生產環境中，建議根據需求來限制允許的標頭和方法，以增強安全性)
* 接著找到 `app.UseHttpsRedirection();` 這一行
* 在這一行的下方，加入底下的程式碼，來啟用 CORS 的中介軟體，並指定使用剛剛設定的 CORS 政策

```csharp
#region 使用 CORS 中介軟體 - 必須放在管道的早期位置
app.UseCors("AllowReactApp");
#endregion
```

* 這裡使用了 `UseCors` 方法來啟用 CORS 的中介軟體，並指定使用 "AllowReactApp" 這個政策
* 儲存 [Program.cs] 的修改

## 執行程式

* 按下 F5 鍵或點擊「開始」按鈕來執行程式
* 此時，會出現 React 設計的網頁，如下圖所示
![](../Images/cs2025-879.png)
* 在網頁最下方，可以看到兩個要輸入的欄位
* 地點與日期，請隨意輸入任何值到這兩個欄位中
* 例如，地點輸入「Taipei」，日期輸入「2024-06-01」
* 然後點擊 [取得天氣] 的按鈕
* 此時，React 的前端應用程式會呼叫後端的 API 端點，並將地點與日期作為查詢字串參數傳送給後端
* 後端的 API 端點會根據接收到的地點與日期來產生對應的天氣預報資料，並將這些資料以 JSON 格式回傳給前端
* 前端接收到這些資料後，會將它們渲染到網頁上，以表格的形式顯示出來
![](../Images/cs2025-878.png)
