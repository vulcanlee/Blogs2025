# Blazor 使用 Ant Design Blazor 側邊欄範例



## 建立 Blazor 專案
* 開啟 Visual Studio 2026
* 選擇「建立新專案」
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇「Blazor Web 應用程式」 項目
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 [csBlazorSiderBar]
* 然後點擊右下方「下一步」按鈕
* 接著會看到 [其他資訊] 對話窗
* 在這個對話窗內，確認使用底下的選項
    * 架構：.NET 10.0 (或更新版本)
    * 驗證類型：無
    * 勾選 針對 HTTPS 進行設定
    * 互動式轉譯模式：伺服器
    * 互動功能位置：全球
    * 勾選 包和範例頁面
    * 勾選 不要使用最上層陳述式 (這是我的個人習慣)
    * 不要勾選 在應用程式 URL 中使用 .dev.localhost TLD
    * 不要勾選 在 .NET Aspire 協調流程中登錄
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 Blazor 專案的建立

## 安裝 AntDesign 套件

套件 AntDesign 是一個流行的 UI 組件庫，提供了許多現成的 UI 元件，可以幫助開發者快速建立美觀且功能豐富的使用者介面。在這個專案中，我們將使用 AntDesign Blazor 這個套件來實現自訂主題的功能。

使用底下方式進行安裝此套件

* 在 Visual Studio 的「方案總管」視窗中，右鍵點擊專案名稱
* 從右鍵選單中，選擇「管理 NuGet 套件」
* 在 NuGet 套件管理器視窗中，切換到「瀏覽」標籤頁
* 在搜尋框中，輸入 "AntDesign" 並按下 Enter 鍵
* 從搜尋結果中，找到 "AntDesign" 套件 並點擊它
* 在這裡的範例中，使用該套件的版本為 1.5.0
* 在右側的詳細資訊面板中，點擊「安裝」按鈕

## 設定 AntDesign 

* 在專案根目錄下，找到 [App.razor] 檔案，並打開它
* 在 [App.razor] 檔案中
* 找到 `</head>` 加入以下程式碼：

```html
<link href="_content/AntDesign/css/ant-design-blazor.css" rel="stylesheet">
```

* 在 [App.razor] 檔案中
* 找到 `</body>` 加入以下程式碼：

```html
<script src="_content/AntDesign/js/ant-design-blazor.js"></script> 
<script antblazor-js></script>
```

## 建立新的版面配置頁面

* 在專案根目錄下，找到 [Components] > [Layout] 目錄下
* 滑鼠右擊 [Layout] 目錄
* 從右鍵選單中，選擇「新增項目」
* 找到並且打開 [SilderBarLayout.razor] 檔案
* 在 [SilderBarLayout.razor] 檔案中，將原有的內容全，使用底下程式碼取代：

```razor
@inherits LayoutComponentBase

<Layout>
    <Sider @bind-Collapsed=@collapsed NoTrigger OnCollapse="OnCollapse">
        <div class="logo" />
        <Menu Theme="MenuTheme.Dark" Mode="MenuMode.Inline" DefaultSelectedKeys=@(new[] { "1" })>
            <MenuItem Key="1">
                <Icon Type="@IconType.Outline.User" />
                <span>nav 1</span>
            </MenuItem>
            <MenuItem Key="2">
                <Icon Type="@IconType.Outline.VideoCamera" />
                <span>nav 2</span>
            </MenuItem>
            <MenuItem Key="3">
                <Icon Type="@IconType.Outline.Upload" />
                <span>nav 3</span>
            </MenuItem>
        </Menu>
    </Sider>
    <Layout Class="site-layout">
        <Header Class="site-layout-background" Style="padding: 0;">
            @if (collapsed)
            {
                <Icon Type="@IconType.Outline.MenuUnfold" Class="trigger" OnClick="toggle" />
            }
            else
            {
                <Icon Type="@IconType.Outline.MenuFold" Class="trigger" OnClick="toggle" />
            }
        </Header>
        <Content Class="site-layout-background" Style="margin: 24px 16px;padding: 24px;min-height: 280px;">
            @Body
        </Content>
    </Layout>
</Layout>

<style>
    #components-layout-demo-custom-trigger .trigger {
        font-size: 18px;
        line-height: 64px;
        padding: 0 24px;
        cursor: pointer;
        transition: color 0.3s;
    }

        #components-layout-demo-custom-trigger .trigger:hover {
            color: #1890ff;
        }

    #components-layout-demo-custom-trigger .logo {
        height: 32px;
        background: rgba(255, 255, 255, 0.2);
        margin: 16px;
    }

    .site-layout .site-layout-background {
        background: #fff;
    }
</style>


@code {
    bool collapsed;

    void toggle()
    {
        collapsed = !collapsed;
    }

    void OnCollapse(bool isCollapsed)
    {
        Console.WriteLine($"Collapsed: {isCollapsed}");
    }

}
```

## 修正 天氣預報 頁面

* 在專案根目錄下，找到 [Components] > [Pages] 目錄下
* 找到並且打開 [Weather.razor] 檔案
* 在 [Weather.razor] 檔案中，將原有的內容全，使用底下程式碼取代：

```razor
@page "/weather"
@layout SilderBarLayout

<PageTitle>Weather</PageTitle>

<h1>Weather</h1>

<p>This component demonstrates showing data.</p>

@if (forecasts == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <table class="table">
        <thead>
            <tr>
                <th>Date</th>
                <th aria-label="Temperature in Celsius">Temp. (C)</th>
                <th aria-label="Temperature in Fahrenheit">Temp. (F)</th>
                <th>Summary</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var forecast in forecasts)
            {
                <tr>
                    <td>@forecast.Date.ToShortDateString()</td>
                    <td>@forecast.TemperatureC</td>
                    <td>@forecast.TemperatureF</td>
                    <td>@forecast.Summary</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {
    private WeatherForecast[]? forecasts;

    protected override async Task OnInitializedAsync()
    {
        // Simulate asynchronous loading to demonstrate a loading indicator
        await Task.Delay(500);

        var startDate = DateOnly.FromDateTime(DateTime.Now);
        var summaries = new[] { "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching" };
        forecasts = Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = startDate.AddDays(index),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = summaries[Random.Shared.Next(summaries.Length)]
        }).ToArray();
    }

    private class WeatherForecast
    {
        public DateOnly Date { get; set; }
        public int TemperatureC { get; set; }
        public string? Summary { get; set; }
        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
    }
}
```

## 執行程式

首先先來看這個專案的執行結果：

* 按下 F5 鍵或點擊「開始」按鈕來執行程式
* 現在將會看到底下畫面
![](../Images/cs2025-891.png)
* 這個左側功能表清單，也是可以收合起來的
* 點擊左上角的「三」圖示按鈕，將會看到底下畫面
![](../Images/cs2025-890.png)
