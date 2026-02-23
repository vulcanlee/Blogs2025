# Blazor 元件生命週期事件深入探討

在 Blazor 開發框架內，Razor 元件的生命週期事件是非常重要的一部分，這些事件會在 Razor 元件的不同階段被觸發，開發者可以在這些事件內撰寫程式碼來實現特定的邏輯，例如在元件初始化時載入資料，在參數變更時更新畫面等。

如何充分掌握與理解這些 Razor 元件的生命週期運作方式與原理，將會有助於寫出更加高效率與好用的 Blazor 網頁系統，因此，在這篇文章中，將會深入探討 Blazor 元件的生命週期事件，並且透過一個範例專案來觀察這些事件的觸發情況，以及在不同的情況下，哪些事件會被觸發。

在這裡將會新設計一個 [Child] 元件，並且在父元件 [Home] 中來參考這個子元件，在這個範例專案中，將會在 Home 元件內宣告一些參數，並且將這些參數傳遞給 Child 元件，然後在 Home 元件和 Child 元件內複寫各種生命週期事件的方法，並且在這些方法內輸出一些訊息，以便在後續的文章中觀察這些事件方法的觸發情況，以及觸發的順序和時間點，這些訊息將有助於理解 Blazor 元件的生命週期事件是如何運作的，以及在不同的情況下，哪些事件會被觸發。

## 建立 Blazor 專案
* 開啟 Visual Studio 2026
* 選擇[建立新專案]
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇[Blazor Web 應用程式] 項目
* 然後點擊右下方[下一步]按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 "csBlazorLifeCycleEvent"
* 然後點擊右下方[下一步]按鈕
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
* 然後點擊右下方[建立]按鈕
* 現在，已經完成了這個 Blazor 專案的建立

## 建立 Home 元件的 Code-behind 程式碼

* 在專案根目錄下，滑鼠右擊 [Components] > [Pages] 資料夾
* 從右鍵選單中，選擇[加入]>[新增項目]
* 這裡將會使用 [顯示精簡檢視] 的方式來輸入要新增的項目
* 在 [新增項目] 對話窗內的文字輸入盒內，輸入 [Home.razor.cs]
* 然後點擊[加入]按鈕
* 現在，已經成功建立了 Home.razor.cs 類別檔案
* 接著，打開 Home.razor.cs 類別檔案，並將以下程式碼複製貼上到該檔案中：

```csharp
using Microsoft.AspNetCore.Components;

namespace csBlazorLifeCycleEvent.Components.Pages;

public partial class Home
{
    public string Name { get; set; } = "Vulcan";

    public int Age { get; set; } = 25;

    public Address Address { get; set; } = new Address();

    public override async Task SetParametersAsync(ParameterView parameters)
    {
        var dict = parameters.ToDictionary();
        var json = System.Text.Json.JsonSerializer.Serialize(dict);
        Console.WriteLine($"H1 - Home SetParametersAsync: {DateTime.Now} , {json}");
        await base.SetParametersAsync(parameters);
    }
    override protected void OnInitialized()
    {
        Console.WriteLine($"H2 - Home OnInitialized: {DateTime.Now}");
    }
    override protected async Task OnInitializedAsync()
    {
        Console.WriteLine($"H3 - Home OnInitializedAsync: {DateTime.Now}");
    }
    override protected void OnParametersSet()
    {
        Console.WriteLine($"H4 - Home OnParametersSet: {DateTime.Now}");
    }
    override protected async Task OnParametersSetAsync()
    {
        Console.WriteLine($"H5 - Home OnParametersSetAsync: {DateTime.Now}");
    }
    override protected void OnAfterRender(bool firstRender)
    {
        Console.WriteLine($"H6 - Home OnAfterRender: {DateTime.Now}, firstRender: {firstRender}");
    }
    override protected async Task OnAfterRenderAsync(bool firstRender)
    {
        Console.WriteLine($"H7 - Home OnAfterRenderAsync: {DateTime.Now}, firstRender: {firstRender}");
    }
    protected override bool ShouldRender()
    {
        Console.WriteLine($"H8 - Home ShouldRender: {DateTime.Now}");
        return base.ShouldRender();
    }
    void OnButtonNothingClick()
    {
        Console.WriteLine();
        Console.WriteLine();
    }
    void OnButtonChange1ParameterClick()
    {
        Console.WriteLine();
        Console.WriteLine();
        Name = "Spock";
    }
    void OnButtonChange2ParameterClick()
    {
        Console.WriteLine();
        Console.WriteLine();
        Name = "Tom";
        Age = 35;
    }
    void OnButtonChangeObjectPropertyParameterClick()
    {
        Console.WriteLine();
        Console.WriteLine();
        Address.Country = "USA";
        Address.City = "New York";
    }
}
```

* 在這個 Code-behind 程式碼中，宣告了三個屬性，分別是 Name、Age 和 Address，在這篇文章中，將僅會把前兩個屬性傳遞給子元件 ChildView，Address 屬性將不會被傳遞給子元件 ChildView(這是因為前兩個屬性為基本資料類別型，而 Address 屬性為複雜資料類型，這樣做的目的是為了在後續的文章中，觀察在父元件 Home 的參數變更時，子元件 ChildView 的生命週期事件觸發情況)
* 接下來將會複寫 Razor 元件內的各種生命週期的事件方法，並在這些方法內輸出一些訊息，以便在後續的文章中觀察這些事件方法的觸發情況
* 第一個事件是 [SetParametersAsync] 這個事件會在 Razor 元件接收到參數時被觸發，無論是父元件傳遞的參數變更，還是使用者互動導致的參數變更，都會觸發這個事件，在這裡會將收到 [ParameterView] 物件轉換成字典，並序列化成 JSON 字串，然後輸出到控制台，以便觀察接收到的參數內容
* 第二個事件是 [OnInitialized] 這個事件會在 Razor 元件被初始化時被觸發，這個事件只會觸發一次，在這裡會輸出一條訊息到控制台，以便觀察這個事件的觸發情況
* 第三個事件是 [OnInitializedAsync] 這個事件也是在 Razor 元件被初始化時被觸發，但這個事件是非同步的，這個事件也只會觸發一次，在這裡同樣會輸出一條訊息到控制台，以便觀察這個事件的觸發情況(對於 OnInitialized 和 OnInitializedAsync 這兩個事件，從輸出訊息中，將可以看出觸發順序，以及觸發的時間點)
* 第四個事件是 [OnParametersSet] 這個事件會在 Razor 元件接收到參數變更時被觸發，無論是父元件傳遞的參數變更，還是使用者互動導致的參數變更，都會觸發這個事件，在這裡會輸出一條訊息到控制台，以便觀察這個事件的觸發情況，這個事件將不會接收到任何的參數內容，這個事件的觸發時機是在 SetParametersAsync 事件之後，並且在 Razor 元件重新渲染之前
* 第五個事件是 [OnParametersSetAsync] 這個事件也是在 Razor 元件接收到參數變更時被觸發，但這個事件是非同步的，這個事件的觸發時機也是在 SetParametersAsync 事件之後，並且在 Razor 元件重新渲染之前，在這裡同樣會輸出一條訊息到控制台，以便觀察這個事件的觸發情況
* 第六個事件是 [OnAfterRender] 這個事件會在 Razor 元件完成渲染後被觸發，這個事件會接收一個布林值參數 [firstRender]，用來表示這是否是 Razor 元件第一次完成渲染，在這裡會輸出一條訊息到控制台，以便觀察這個事件的觸發情況，以及是否為第一次渲染
* 第七個事件是 [OnAfterRenderAsync] 這個事件也是在 Razor 元件完成渲染後被觸發，但這個事件是非同步的，這個事件同樣會接收一個布林值參數 [firstRender]，用來表示這是否是 Razor 元件第一次完成渲染，在這裡同樣會輸出一條訊息到控制台，以便觀察這個事件的觸發情況，以及是否為第一次渲染
* 第八個事件是 [ShouldRender] 這個事件會在 Razor 元件接收到參數變更，或者內部狀態變更時被觸發，這個事件會決定 Razor 元件是否需要重新渲染，在這裡會輸出一條訊息到控制台，以便觀察這個事件的觸發情況，並且會回傳 base.ShouldRender() 的結果，這樣就會使用預設的邏輯來決定是否重新渲染 Razor 元件
* 接下來，宣告了四個方法，分別是：
* [OnButtonNothingClick] 將會當使用者按下這個按鈕後所觸發的委派方法，這個方法內不會對 Razor 元件的參數進行任何變更，這樣就可以觀察在使用者互動但沒有參數變更的情況下，Razor 元件的生命週期事件的觸發情況
* [OnButtonChange1ParameterClick] 將會當使用者按下這個按鈕後所觸發的委派方法，這個方法內會將 Razor 元件的 Name 參數變更為 "Spock"，這樣就可以觀察在使用者互動並且有一個參數變更的情況下，Razor 元件的生命週期事件的觸發情況
* [OnButtonChange2ParameterClick] 將會當使用者按下這個按鈕後所觸發的委派方法，這個方法內會將 Razor 元件的 Name 參數變更為 "Tom"，並且將 Age 參數變更為 35，這樣就可以觀察在使用者互動並且有多個參數變更的情況下，Razor 元件的生命週期事件的觸發情況
* [OnButtonChangeObjectPropertyParameterClick] 將會當使用者按下這個按鈕後所觸發的委派方法，這個方法內會將 Razor 元件的 Address 參數內部物件的 Country 屬性變更為 "USA"，並且將 City 屬性變更為 "New York"，這樣就可以觀察在使用者互動並且有複雜資料類型參數內部屬性變更的情況下，Razor 元件的生命週期事件的觸發情況

## 修正 Home.razor 元件的網頁標記
* 打開 Home.razor 元件，並將以下程式碼複製貼上到該元件中：

```razor
@page "/"

<PageTitle>Home</PageTitle>

<h1>Hello, world!</h1>

Welcome to your new app.

<div class="p-3">
    <button class="btn btn-primary" @onclick="OnButtonNothingClick">僅觸發按鈕事件</button>
    <button class="btn btn-secondary" @onclick="OnButtonChange1ParameterClick">觸發按鈕事件並變更 1 個參數</button>
    <button class="btn btn-success" @onclick="OnButtonChange2ParameterClick">觸發按鈕事件並變更 2 個參數</button>
    <button class="btn btn-success" @onclick="OnButtonChangeObjectPropertyParameterClick">觸發按鈕事件並變更參數的物件屬性值</button>
</div>

<div class="p-3">
    <div class="card">
        <div class="card-body">
            <p>This is the parent view.</p>
            <p>Name : @Name</p>
            <p>Age : @Age</p>
            <p>Address Hash : @Address.GetHashCode()</p>
            <p>Country : @Address.Country</p>
            <p>City : @Address.City</p>
        </div>
    </div>
</div>

<div class="p-3">
    <ChildView Name="@Name" Age="@Age" />
</div>
```

* 從 `<ChildView Name="@Name" Age="@Age" />` 這行程式碼中，可以看到將 Home 元件的 Name 和 Age 參數傳遞給了 ChildView 元件，而 Address 參數則沒有被傳遞給 ChildView 元件

## 建立 ChildView 元件
* 在專案根目錄下，滑鼠右擊 [Components] > [Pages] 資料夾
* 從右鍵選單中，選擇[加入]>[Razor 元件]
* 在 [新增項目] 對話窗內的文字輸入盒內，輸入 [ChildView.razor]
* 然後點擊[加入]按鈕
* 現在，已經成功建立了 ChildView.razor 類別檔案
* 接著，打開 ChildView.razor 類別檔案，並將以下程式碼複製貼上到該檔案中：

## 建立 ChildView 元件的 Code-behind 程式碼

* 在專案根目錄下，滑鼠右擊 [Components] > [Pages] 資料夾
* 從右鍵選單中，選擇[加入]>[新增項目]
* 這裡將會使用 [顯示精簡檢視] 的方式來輸入要新增的項目
* 在 [新增項目] 對話窗內的文字輸入盒內，輸入 [ChildView.razor.cs]
* 然後點擊[加入]按鈕
* 現在，已經成功建立了 ChildView.razor.cs 類別檔案
* 接著，打開 ChildView.razor.cs 類別檔案，並將以下程式碼複製貼上到該檔案中：

```csharp
using Microsoft.AspNetCore.Components;

namespace csBlazorLifeCycleEvent.Components.Pages;

public partial class ChildView
{
    [Parameter]
    public string Name { get; set; } = string.Empty;

    [Parameter]
    public int Age { get; set; }
    public Address Address { get; set; } = new();
    public override async Task SetParametersAsync(ParameterView parameters)
    {
        var dict = parameters.ToDictionary();
        var json = System.Text.Json.JsonSerializer.Serialize(dict);
        Console.WriteLine($"C1 - Child SetParametersAsync: {DateTime.Now} , {json}");
        if (parameters.TryGetValue<string>("Name", out var newName))
        {
            if (Name != newName)
                Console.WriteLine($"     收到新的 Name: {newName}");
        }
        if (parameters.TryGetValue<int>("Age", out var newAge))
        {
            if (Age != newAge)
                Console.WriteLine($"     收到新的 Age: {newAge}");
        }
        await base.SetParametersAsync(parameters);
    }
    override protected void OnInitialized()
    {
        Console.WriteLine($"C2 - Child OnInitialized: {DateTime.Now}");
    }
    override protected async Task OnInitializedAsync()
    {
        Console.WriteLine($"C3 - Child OnInitializedAsync: {DateTime.Now}");
    }
    override protected void OnParametersSet()
    {
        Console.WriteLine($"C4 - Child OnParametersSet: {DateTime.Now}");
    }
    override protected async Task OnParametersSetAsync()
    {
        Console.WriteLine($"C5 - Child OnParametersSetAsync: {DateTime.Now}");
    }
    override protected void OnAfterRender(bool firstRender)
    {
        Console.WriteLine($"C6 - Child OnAfterRender: {DateTime.Now}, firstRender: {firstRender}");
    }
    override protected async Task OnAfterRenderAsync(bool firstRender)
    {
        Console.WriteLine($"C7 - Child OnAfterRenderAsync: {DateTime.Now}, firstRender: {firstRender}");
    }
    protected override bool ShouldRender()
    {
        Console.WriteLine($"C8 - Child ShouldRender: {DateTime.Now}");
        return base.ShouldRender();
    }
}
```

* 在這個 Code-behind 程式碼中，宣告了兩個參數屬性，使用了 `[Parameter]`，分別是 Name 和 Age，這兩個參數將會從父元件 Home 傳遞過來，在這裡同樣複寫了 Razor 元件內的各種生命週期的事件方法，並在這些方法內輸出一些訊息，以便在後續的文章中觀察這些事件方法的觸發情況，這些事件方法與 Home 元件內的事件方法相同，只是在輸出的訊息中，將 Home 改為 Child，以便區分是 Home 元件的事件觸發，還是 ChildView 元件的事件觸發

## 修正 ChildView.razor 元件的網頁標記
* 打開 ChildView.razor 元件，並將以下程式碼複製貼上到該元件中：

```razor
<div class="card">
    <div class="card-body">
        <p>This is the child view.</p>
        <p>Name : @Name</p>
        <p>Age : @Age</p>
        <p>Address Hash : @Address.GetHashCode()</p>
        <p>Country : @Address.Country</p>
        <p>City : @Address.City</p>
    </div>
</div>
```

## 取消預先渲覽功能
* 在專案根目錄下的[Components] 資料夾
* 找到並且打開 [App.razor] 元件
* 搜尋出程式碼 `<HeadOutlet @rendermode="InteractiveServer" />` 將其改寫成為 `<HeadOutlet @rendermode="@(new InteractiveServerRenderMode(prerender: false))" />`，以取消預先渲覽功能
* 
* 搜尋出程式碼 `<Routes @rendermode="InteractiveServer" />` 將其改寫成為 `<Routes @rendermode="@(new InteractiveServerRenderMode(prerender: false))" />`，以取消預先渲覽功能

# 執行程式

首先先來看這個專案的執行結果：

* 按下 F5 鍵或點擊[開始]按鈕來執行程式
* 在瀏覽器中，將會看到 Home 元件的內容，以及 ChildView 元件的內容，如下圖
![](../Images/cs2025-858.png)
* 在最上方出現的四個按鈕
* 接著出現的卡片內容，則是在 [Home] 這個元件內加入的標記內容
* 然後在下方看到的卡片內容，則是在 [ChildView] 這個元件內加入的標記內容
* 底下為在 [Console] 視窗內看到的資訊

```plain
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:7019
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5121
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: C:\Vulcan\Github\CSharp2025\csBlazorLifeCycleEvent\csBlazorLifeCycleEvent
H1 - Home SetParametersAsync: 2026/2/23 上午 10:38:35 , {}
H2 - Home OnInitialized: 2026/2/23 上午 10:38:35
H3 - Home OnInitializedAsync: 2026/2/23 上午 10:38:35
H4 - Home OnParametersSet: 2026/2/23 上午 10:38:35
H5 - Home OnParametersSetAsync: 2026/2/23 上午 10:38:35
C1 - Child SetParametersAsync: 2026/2/23 上午 10:38:35 , {"Name":"Vulcan","Age":25}
     收到新的 Name: Vulcan
     收到新的 Age: 25
C2 - Child OnInitialized: 2026/2/23 上午 10:38:35
C3 - Child OnInitializedAsync: 2026/2/23 上午 10:38:35
C4 - Child OnParametersSet: 2026/2/23 上午 10:38:35
C5 - Child OnParametersSetAsync: 2026/2/23 上午 10:38:35
H6 - Home OnAfterRender: 2026/2/23 上午 10:38:35, firstRender: True
H7 - Home OnAfterRenderAsync: 2026/2/23 上午 10:38:35, firstRender: True
C6 - Child OnAfterRender: 2026/2/23 上午 10:38:35, firstRender: True
C7 - Child OnAfterRenderAsync: 2026/2/23 上午 10:38:35, firstRender: True
```

* 從這些輸出的訊息中，可以觀察到在 Home 元件和 ChildView 元件的各種生命週期事件的觸發情況，以及觸發的順序和時間點，這些訊息將有助於理解 Blazor 元件的生命週期事件是如何運作的，以及在不同的情況下，哪些事件會被觸發
* 這裡將會說明這些事件觸發的順序和時間點，首先在 Home 元件中，當 Razor 元件被初始化時，會先觸發 [SetParametersAsync] 事件，然後觸發 [OnInitialized] 事件，接著觸發 [OnInitializedAsync] 事件，然後觸發 [OnParametersSet] 事件，最後觸發 [OnParametersSetAsync] 事件，在 ChildView 元件中，當 Razor 元件被初始化時，同樣會先觸發 [SetParametersAsync] 事件，然後觸發 [OnInitialized] 事件，接著觸發 [OnInitializedAsync] 事件，然後觸發 [OnParametersSet] 事件，最後觸發 [OnParametersSetAsync] 事件，在 Home 元件和 ChildView 元件的這些事件之後，才會觸發 [OnAfterRender] 和 [OnAfterRenderAsync] 這兩個事件，這些事件的觸發順序和時間點是固定的，無論是在哪個元件中，這些事件的觸發順序和時間點都是相同的，這些事件的觸發順序和時間點是由 Blazor 框架所定義的，開發者無法改變這些事件的觸發順序和時間點，這些事件的觸發順序和時間點是 Blazor 元件生命週期的一部分，理解這些事件的觸發順序和時間點，對於開發 Blazor 元件是非常重要的，這些事件的觸發順序和時間點，將會影響到元件的行為和性能，開發者需要根據這些事件的觸發順序和時間點，來設計元件的邏輯和行為，以達到最佳的性能和使用者體驗

* 接下來，將會在 Home 元件內，點擊 [僅觸發按鈕事件] 按鈕，這個按鈕所綁定的事件，並沒有做任何事情，觀察當這個按鈕被點擊之後，這些事件會如何被觸發，當點擊這個按鈕之後，將會在 Console 視窗內看到以下的輸出訊息：

```plain
H8 - Home ShouldRender: 2026/2/23 上午 10:48:17
H6 - Home OnAfterRender: 2026/2/23 上午 10:48:17, firstRender: False
H7 - Home OnAfterRenderAsync: 2026/2/23 上午 10:48:17, firstRender: False
```

* 由於該按鈕並沒有做任何事情，而是觸發了重新渲染 Home 元件的事件，所以在 Console 視窗內，將會看到 Home 元件的 [ShouldRender] 事件被觸發，然後是 [OnAfterRender] 和 [OnAfterRenderAsync] 這兩個事件被觸發，而由於傳遞到子元件內的參數並沒有改變，所以 ChildView 元件的生命週期事件不會被觸發

* 接下來，將會在 Home 元件內，點擊 [觸發按鈕事件並變更 1 個參數] 按鈕，這個按鈕所綁定的事件，會將 Razor 元件的 Name 參數變更為 "Spock"，觀察當這個按鈕被點擊之後，這些事件會如何被觸發，當點擊這個按鈕之後，將會在 Console 視窗內看到以下的輸出訊息：

```plain
H8 - Home ShouldRender: 2026/2/23 上午 11:32:05
C1 - Child SetParametersAsync: 2026/2/23 上午 11:32:05 , {"Name":"Spock","Age":25}
     收到新的 Name: Spock
C4 - Child OnParametersSet: 2026/2/23 上午 11:32:05
C5 - Child OnParametersSetAsync: 2026/2/23 上午 11:32:05
C8 - Child ShouldRender: 2026/2/23 上午 11:32:05
H6 - Home OnAfterRender: 2026/2/23 上午 11:32:05, firstRender: False
H7 - Home OnAfterRenderAsync: 2026/2/23 上午 11:32:05, firstRender: False
C6 - Child OnAfterRender: 2026/2/23 上午 11:32:05, firstRender: False
C7 - Child OnAfterRenderAsync: 2026/2/23 上午 11:32:05, firstRender: False
```

* 這裡是網頁畫面截圖
![](../Images/cs2025-857.png)
* 由於該按鈕觸發了參數變更，所以在 Console 視窗內，將會看到 Home 元件的 [ShouldRender] 事件被觸發，然後 ChildView 元件的 [SetParametersAsync] 事件被觸發，並且在該事件內，接收到新的 Name 參數值為 "Spock"，接著是 ChildView 元件的 [OnParametersSet] 和 [OnParametersSetAsync] 這兩個事件被觸發，然後是 ChildView 元件的 [ShouldRender] 事件被觸發，最後是 Home 元件和 ChildView 元件的 [OnAfterRender] 和 [OnAfterRenderAsync] 這四個事件被觸發，而由於傳遞到子元件內的參數有改變，所以 ChildView 元件的生命週期事件也會被觸發
* 接下來，將會在 Home 元件內，點擊 [觸發按鈕事件並變更 2 個參數] 按鈕，這個按鈕所綁定的事件，會將 Razor 元件的 Name 參數變更為 "Tom"，並且將 Age 參數變更為 35，觀察當這個按鈕被點擊之後，這些事件會如何被觸發，當點擊這個按鈕之後，將會在 Console 視窗內看到以下的輸出訊息：

```plain
H8 - Home ShouldRender: 2026/2/23 上午 11:34:33
C1 - Child SetParametersAsync: 2026/2/23 上午 11:34:33 , {"Name":"Tom","Age":35}
     收到新的 Name: Tom
     收到新的 Age: 35
C4 - Child OnParametersSet: 2026/2/23 上午 11:34:33
C5 - Child OnParametersSetAsync: 2026/2/23 上午 11:34:33
C8 - Child ShouldRender: 2026/2/23 上午 11:34:33
H6 - Home OnAfterRender: 2026/2/23 上午 11:34:33, firstRender: False
H7 - Home OnAfterRenderAsync: 2026/2/23 上午 11:34:33, firstRender: False
C6 - Child OnAfterRender: 2026/2/23 上午 11:34:33, firstRender: False
C7 - Child OnAfterRenderAsync: 2026/2/23 上午 11:34:33, firstRender: False
```

* 這裡是網頁畫面截圖
![](../Images/cs2025-856.png)

* 由於該按鈕觸發了參數變更，所以在 Console 視窗內，將會看到 Home 元件的 [ShouldRender] 事件被觸發，然後 ChildView 元件的 [SetParametersAsync] 事件被觸發，並且在該事件內，接收到新的 Name 參數值為 "Tom"，以及新的 Age 參數值為 35，接著是 ChildView 元件的 [OnParametersSet] 和 [OnParametersSetAsync] 這兩個事件被觸發，然後是 ChildView 元件的 [ShouldRender] 事件被觸發，最後是 Home 元件和 ChildView 元件的 [OnAfterRender] 和 [OnAfterRenderAsync] 這四個事件被觸發，而由於傳遞到子元件內的參數有改變，所以 ChildView 元件的生命週期事件也會被觸發
