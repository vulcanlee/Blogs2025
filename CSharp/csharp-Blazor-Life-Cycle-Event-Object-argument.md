# Blazor 元件生命週期事件深入探討 2 - 使用基本型別與類別引數的差異

在上一篇 [ASP.NET Core Blazor 元件生命週期事件深入探討](https://csharpkh.blogspot.com/2026/02/csharp-Blazor-Life-Cycle-Event.html) 文章中，已經說明了 Blazor 元件的各種生命週期事件，以及在不同的情況下，哪些事件會被觸發，在這篇文章中，將會繼續深入探討 Blazor 元件的生命週期事件，並且透過一個範例專案來觀察在使用基本型別參數與類別型別參數時，Blazor 元件的生命週期事件的觸發情況，以及在不同的情況下，哪些事件會被觸發。

不過，當時使用的是 [Primitive Type] 資料型別，在這篇文章中，將會自行建立一個 [Address] 類別，並且在 Home 元件內宣告一個 Address 型別的參數，然後將這個參數傳遞給 Child 元件，這樣就可以觀察在使用類別型別參數時，Blazor 元件的生命週期事件的觸發情況，以及在不同的情況下，哪些事件會被觸發。

## 修正 Home.razor 元件的網頁標記
* 打開 Home.razor 元件，並將以下程式碼複製貼上到該元件中：
* 找到 `<ChildView Name="@Name" Age="@Age" />`
* 將其改寫成 `<ChildView Name="@Name" Age="@Age" Address="@Address" />`，這裡將會新增一個 Address 參數，並且將 Home 元件內的 Address 參數傳遞給 ChildView 元件


## 修正 ChildView.razor.cs 元件的 Code-behind 程式碼
* 打開 ChildView.razor.cs 元件，並將以下程式碼
* 找到 `public Address Address { get; set; } = new();`
* 將其改寫成 `[Parameter] public Address Address { get; set; } = new();`，這裡將會新增一個 Address 參數，並且將 Home 元件內的 Address 參數傳遞給 ChildView 元件

# 執行程式

首先先來看這個專案的執行結果：

* 按下 F5 鍵或點擊[開始]按鈕來執行程式
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
H1 - Home SetParametersAsync: 2026/2/23 下午 01:08:27 , {}
H2 - Home OnInitialized: 2026/2/23 下午 01:08:27
H3 - Home OnInitializedAsync: 2026/2/23 下午 01:08:27
H4 - Home OnParametersSet: 2026/2/23 下午 01:08:27
H5 - Home OnParametersSetAsync: 2026/2/23 下午 01:08:27
C1 - Child SetParametersAsync: 2026/2/23 下午 01:08:27 , {"Name":"Vulcan","Age":25,"Address":{"Country":"","City":""}}
     收到新的 Name: Vulcan
     收到新的 Age: 25
C2 - Child OnInitialized: 2026/2/23 下午 01:08:27
C3 - Child OnInitializedAsync: 2026/2/23 下午 01:08:27
C4 - Child OnParametersSet: 2026/2/23 下午 01:08:27
C5 - Child OnParametersSetAsync: 2026/2/23 下午 01:08:27
H6 - Home OnAfterRender: 2026/2/23 下午 01:08:27, firstRender: True
H7 - Home OnAfterRenderAsync: 2026/2/23 下午 01:08:27, firstRender: True
C6 - Child OnAfterRender: 2026/2/23 下午 01:08:27, firstRender: True
C7 - Child OnAfterRenderAsync: 2026/2/23 下午 01:08:27, firstRender: True
```

* 這裡看到的結果，與上一篇文章看到的差異在於，在 ChildView 元件的 [SetParametersAsync] 事件內，接收到的參數值中，除了 Name 和 Age 這兩個基本型別參數之外，還多了一個 Address 這個類別型別參數，並且在該事件內，接收到的 Address 參數值為一個新的 Address 物件，該物件的 Country 和 City 屬性值都是空字串，這是因為在 Home 元件內宣告的 Address 參數初始值為一個新的 Address 物件，而該物件的 Country 和 City 屬性值都是空字串，所以在 ChildView 元件內接收到的 Address 參數值也是一個新的 Address 物件，而該物件的 Country 和 City 屬性值也是空字串。
* 接下來，將會在 Home 元件內，點擊 [僅觸發按鈕事件] 按鈕，這個按鈕所綁定的事件，並沒有做任何事情，觀察當這個按鈕被點擊之後，這些事件會如何被觸發，當點擊這個按鈕之後，將會在 Console 視窗內看到以下的輸出訊息：

```plain
H8 - Home ShouldRender: 2026/2/23 下午 01:12:28
C1 - Child SetParametersAsync: 2026/2/23 下午 01:12:28 , {"Name":"Vulcan","Age":25,"Address":{"Country":"","City":""}}
C4 - Child OnParametersSet: 2026/2/23 下午 01:12:28
C5 - Child OnParametersSetAsync: 2026/2/23 下午 01:12:28
C8 - Child ShouldRender: 2026/2/23 下午 01:12:28
H6 - Home OnAfterRender: 2026/2/23 下午 01:12:28, firstRender: False
H7 - Home OnAfterRenderAsync: 2026/2/23 下午 01:12:28, firstRender: False
C6 - Child OnAfterRender: 2026/2/23 下午 01:12:28, firstRender: False
C7 - Child OnAfterRenderAsync: 2026/2/23 下午 01:12:28, firstRender: False
```

* 底下的內容將會是上一篇文章中看到的執行結果

```plain
H8 - Home ShouldRender: 2026/2/23 上午 10:48:17
H6 - Home OnAfterRender: 2026/2/23 上午 10:48:17, firstRender: False
H7 - Home OnAfterRenderAsync: 2026/2/23 上午 10:48:17, firstRender: False
```

* 比較這兩篇文章的執行結果，在這篇中，有使用一個自訂的 [Address] 類別型別參數，並且將其傳遞給 ChildView 元件，而在上一篇文章中，沒有使用這個類別型別參數，所以在這篇文章中，在 Home 元件內點擊 [僅觸發按鈕事件] 按鈕之後，ChildView 元件的 [SetParametersAsync] 事件會被觸發，並且在該事件內接收到的參數值中，包含了 Name、Age 和 Address 這三個參數值，而在上一篇文章中，在 Home 元件內點擊 [僅觸發按鈕事件] 按鈕之後，ChildView 元件的 [SetParametersAsync] 事件不會被觸發，因為沒有任何參數值被傳遞給 ChildView 元件，所以在該事件內也不會接收到任何參數值，這就是使用基本型別參數與類別型別參數的差異，在使用基本型別參數時，如果沒有改變參數值，則不會觸發子元件的生命週期事件，而在使用類別型別參數時，即使沒有改變參數值，由於傳遞的是物件的參考，所以仍然會觸發子元件的生命週期事件，並且在該事件內接收到的參數值仍然是同一個物件的參考，這就是使用基本型別參數與類別型別參數的差異，這也是在 Blazor 開發中需要注意的一點，當使用類別型別參數時，即使沒有改變參數值，也會觸發子元件的生命週期事件，並且在該事件內接收到的參數值仍然是同一個物件的參考，所以在撰寫 Blazor 元件時，需要注意這一點，以避免不必要的事件觸發和效能問題。
* 接下來，將會在 Home 元件內，點擊 [觸發按鈕事件並變更 1 個參數] 按鈕，這個按鈕所綁定的事件，會將 Razor 元件的 Name 參數變更為 "Spock"，觀察當這個按鈕被點擊之後，這些事件會如何被觸發，當點擊這個按鈕之後，將會在 Console 視窗內看到以下的輸出訊息：

```plain
H8 - Home ShouldRender: 2026/2/23 下午 01:16:28
C1 - Child SetParametersAsync: 2026/2/23 下午 01:16:28 , {"Name":"Spock","Age":25,"Address":{"Country":"","City":""}}
     收到新的 Name: Spock
C4 - Child OnParametersSet: 2026/2/23 下午 01:16:28
C5 - Child OnParametersSetAsync: 2026/2/23 下午 01:16:28
C8 - Child ShouldRender: 2026/2/23 下午 01:16:28
H6 - Home OnAfterRender: 2026/2/23 下午 01:16:28, firstRender: False
H7 - Home OnAfterRenderAsync: 2026/2/23 下午 01:16:28, firstRender: False
C6 - Child OnAfterRender: 2026/2/23 下午 01:16:28, firstRender: False
C7 - Child OnAfterRenderAsync: 2026/2/23 下午 01:16:28, firstRender: False
```

* 底下是上一篇文章的同一個按鈕的執行效果

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

* 比較這兩篇文章的執行結果，差異在於，在這篇文章中，在 Home 元件內點擊 [觸發按鈕事件並變更 1 個參數] 按鈕之後，ChildView 元件的 [SetParametersAsync] 事件會被觸發，並且在該事件內接收到的參數值中，包含了 Name、Age 和 Address 這三個參數值，而在上一篇文章中，在 Home 元件內點擊 [觸發按鈕事件並變更 1 個參數] 按鈕之後，ChildView 元件的 [SetParametersAsync] 事件同樣會被觸發，並且在該事件內接收到的參數值中，包含了 Name 和 Age 這兩個參數值，但沒有 Address 這個類別型別參數值，這就是使用基本型別參數與類別型別參數的差異，在使用基本型別參數時，如果沒有改變參數值，則不會觸發子元件的生命週期事件，而在使用類別型別參數時，即使沒有改變參數值，由於傳遞的是物件的參考，所以仍然會觸發子元件的生命週期事件，並且在該事件內接收到的參數值仍然是同一個物件的參考，這就是使用基本型別參數與類別型別參數的差異，這也是在 Blazor 開發中需要注意的一點，當使用類別型別參數時，即使沒有改變參數值，也會觸發子元件的生命週期事件，並且在該事件內接收到的參數值仍然是同一個物件的參考，所以在撰寫 Blazor 元件時，需要注意這一點，以避免不必要的事件觸發和效能問題。
* 接下來，將會在 Home 元件內，點擊 [觸發按鈕事件並變更參數的物件屬性值] 按鈕，這個按鈕所綁定的事件，會將 Razor 元件的 Address 參數的 Country 屬性值變更為 "USA"，觀察當這個按鈕被點擊之後，這些事件會如何被觸發，當點擊這個按鈕之後，將會在 Console 視窗內看到以下的輸出訊息：

```plain
H8 - Home ShouldRender: 2026/2/23 下午 01:19:38
C1 - Child SetParametersAsync: 2026/2/23 下午 01:19:38 , {"Name":"Spock","Age":25,"Address":{"Country":"USA","City":"New York"}}
C4 - Child OnParametersSet: 2026/2/23 下午 01:19:38
C5 - Child OnParametersSetAsync: 2026/2/23 下午 01:19:38
C8 - Child ShouldRender: 2026/2/23 下午 01:19:38
H6 - Home OnAfterRender: 2026/2/23 下午 01:19:38, firstRender: False
H7 - Home OnAfterRenderAsync: 2026/2/23 下午 01:19:38, firstRender: False
C6 - Child OnAfterRender: 2026/2/23 下午 01:19:38, firstRender: False
C7 - Child OnAfterRenderAsync: 2026/2/23 下午 01:19:38, firstRender: False
```

* 這裡看到的執行結果為，在 Home 元件內點擊 [觸發按鈕事件並變更參數的物件屬性值] 按鈕之後，ChildView 元件的 [SetParametersAsync] 事件會被觸發，並且在該事件內接收到的參數值中，包含了 Name、Age 和 Address 這三個參數值，而在該事件內接收到的 Address 參數值中，Country 屬性值已經變更為 "USA"，City 屬性值仍然是 "New York"，這是因為在 Home 元件內宣告的 Address 參數初始值為一個新的 Address 物件，而該物件的 Country 和 City 屬性值都是空字串，所以在 ChildView 元件內接收到的 Address 參數值也是一個新的 Address 物件，而該物件的 Country 和 City 屬性值也是空字串，但當點擊 [觸發按鈕事件並變更參數的物件屬性值] 按鈕之後，Home 元件內宣告的 Address 參數所參考的物件的 Country 屬性值被改變為 "USA"，所以在 ChildView 元件內接收到的 Address 參數值中，Country 屬性值也被改變為 "USA"，而 City 屬性值仍然是 "New York"，這就是使用類別型別參數時，即使沒有改變參數值，由於傳遞的是物件的參考，所以仍然會觸發子元件的生命週期事件，並且在該事件內接收到的參數值仍然是同一個物件的參考，所以當改變了該物件的屬性值之後，在子元件內接收到的該物件的屬性值也會被改變，這就是使用類別型別參數時需要注意的一點，以避免不必要的事件觸發和效能問題。