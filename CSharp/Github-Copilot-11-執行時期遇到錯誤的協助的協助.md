# Github Copilot 11 : 執行時期遇到錯誤的協助的協助


當修改完成程式之後，進入到執行階段，突然拋出如下例外異常通知，遇到這樣的問題，通常是所有的程式設計師都會感到頭痛與絕望無助的時候，例如，遇到底下的狀況

![](../Images/cs2025-920.png)

這裡遇到的 [使用者未處理的例外狀況] 詳細資訊如下

```
System.ArgumentNullException
  HResult=0x80004003
  Message=Value cannot be null. (Parameter 'stringToEscape')
  Source=System.Private.CoreLib
  StackTrace: 
   於 System.ArgumentNullException.Throw(String paramName)
   於 System.ArgumentNullException.ThrowIfNull(Object argument, String paramName)
   於 System.UriHelper.EscapeString(String stringToEscape, Boolean checkExistingEscaped, SearchValues`1 noEscape)
   於 SmartEHRLaunch1.Components.Pages.Launch.<GetAuthorizeUrlAsync>d__18.MoveNext() 在 C:\Vulcan\Projects\SmartEHRLaunch1\SmartEHRLaunch1\Components\Pages\Launch.razor.cs:行 108 中
   於 SmartEHRLaunch1.Components.Pages.Launch.<OnAfterRenderAsync>d__15.MoveNext() 在 C:\Vulcan\Projects\SmartEHRLaunch1\SmartEHRLaunch1\Components\Pages\Launch.razor.cs:行 27 中
   於 Microsoft.AspNetCore.Components.RenderTree.Renderer.<GetErrorHandledTask>d__101.MoveNext()
```

此時，可以點選該例外狀況對話窗中的 [使用 Copilot 分析] 按鈕，看看會有甚麼結果

![](../Images/cs2025-919.png)

從 Copilot 所提供的建議內容看到 `使用 #debugger，分析 'System.ArgumentNullException' 例外狀況` 這樣的訊息，表示此時 Copilot 正在抓取 VS 2026 內的相關資訊來進行分析，發現是因為在組成授權網址的時候，某個參數值是 null，因此導致這個例外狀況的發生

```
錯誤是 System.ArgumentNullException: Value cannot be null. (Parameter 'stringToEscape')，來源是 Uri.EscapeDataString(...) 被傳入了 null。
從除錯資料可知：
•	RedirectUrl 為 null
•	FhirServerUrl 有值
•	AuthorizeUrl 有值
因此在這行發生錯誤：

$"&redirect_uri={Uri.EscapeDataString(SmartAppSettingService.Data.RedirectUrl)}"
```