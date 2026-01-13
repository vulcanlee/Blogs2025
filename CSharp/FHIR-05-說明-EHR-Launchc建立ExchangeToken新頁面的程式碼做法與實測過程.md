# FHIR 05 說明 EHR Launch 建立 ExchangeToken 新頁面的程式碼做法與實測過程

在上一篇文章中 [FHIR 04 在 EHR Launch 建立新頁面來接收授權碼進而取得存取權杖並呼叫 FHIR API](https://csharpkh.blogspot.com/2026/01/FHIR-04--FHIR-API.html)  ，已經完成了相關 Exchange Token 的程式碼，透過這些新增的程式碼與頁面，透過準備新得到的存取權杖 Access Token 成功地呼叫 FHIR API 來讀取病患資料。

首先，先來執行這個專案，看看執行成果，接著，再來說明這些程式碼的做法與實測過程。

## 執行結果說明

依照底下步驟操作，完成使用 Smart On FHIR 沙盒環境來進行 EHR Launch 的授權碼取得流程：

* 開啟並且執行 [SmartEHRLaunch1]
* 開啟沙盒驗證網頁 [https://thas.mohw.gov.tw/]
* 這裡需要先進行身分驗證，點選 [登入] 文字連結
* 在登入對話窗內，依序輸入帳號、密碼、驗證碼
* 最後點選 [登入] 按鈕，完成身分驗證作業
* 點選上方的 [Sand Box] 連結，進入沙盒系統
* 現在出現了 [SAND-BOX] 對話窗
* 對於 [請選擇] 下拉選單，選擇 [EHR Launch] 項目，也就是預設值
* 在 [請輸入網址] 欄位中，輸入 `https://localhost:7108/launch`
* 其中， [https://localhost:7108/launch] 端點，就是剛剛撰寫程式碼的 Razor 元件頁面
* 點選 [完成] 按鈕，系統會導向到 EHR Launch 頁面
* 在 [Launch] 端點頁面中，將會顯示出文字 [請稍後，正在初始化中]
* 這個網址將會是 Sandbox 系統導向到我們的 EHR Launch 頁面的網址，從這裡也看到了兩個重要的參數：
  * `iss` ：代表 EHR 系統的 FHIR 伺服器位址 [https://thas.mohw.gov.tw/v/r4/fhir](https://thas.mohw.gov.tw/v/r4/fhir) ，這個將會是我們後續要與之互動的 FHIR 伺服器
  * `launch` ：代表這次啟動的 Launch 參數，也就是 EHR 的系統所產生的 Launch Context
* 接著，系統會自動將我們的瀏覽器，重新導向到授權伺服器
* 現在，將會進入到 Sand Box 的授權伺服器，要取得授權碼，當取得了授權碼之後，就會使用 redirect_uri 參數，重新導向回到我們開發的頁面端點 `https://localhost:7108/ExchangeToken` ，並且在網址列中帶入授權碼參數 code
* 在網頁上，將會看到下面的畫面截圖，準備進入到身分驗證階段
* 在這裡使用預設的帳號與密碼，點選 [Login] 按鈕
* 現在的畫面將會切換到選擇病患的畫面
* 點選任一病患項目，準備進入到下一個階段，從抵下畫面截圖可以看到，這裡選取的病人名稱為 [A DMSpatient1 A]
![](../Images/cs2025-905.png)
* 現在，將會進入到 Sand Box 的授權伺服器，要取得授權碼，當取得了授權碼之後，就會使用 redirect_uri 參數，重新導向回到我們開發的頁面端點 `https://localhost:7108/ExchangeToken` ，並且在網址列中帶入授權碼參數 code 與 state 參數
* 這時候，ExchangeToken 頁面將會被載入，並且開始執行頁面程式碼
* 之後，就會在螢幕上看到下面截圖畫面，這裡出現了從授權伺服器回傳的授權碼內容，這裡列出了 code & state 這兩個參數
* 第一個參數為授權碼，其實這裡的內容就是一個 JWT 物件，有了這個授權碼之後，便可以透過這個授權碼來交換存取權杖 Access Token，有了存取權杖之後，便可以用來呼叫 FHIR API 來讀取病患資料，會需要這樣做的原因在於，此時的 FHIR Server API 若想要成功呼叫與取得結果，必須要有存取權杖 Access Token 才行，否則，將會回傳錯誤訊息
* 底下為授權碼經過使用 Jwt 解碼之後的實際內容

```json
{
  "context": {
    "need_patient_banner": true,
    "smart_style_url": "https://thas.mohw.gov.tw/smart-style.json",
    "patient": "patient-TWEMR-DMS-123456789-A123456789"
  },
  "client_id": "smart-app",
  "redirect_uri": "https://localhost:7108/ExchangeToken",
  "scope": "openid fhirUser profile launch/patient patient/*.read patient/Encounter.read patient/MedicationRequest.read patient/ServiceRequest.read",
  "pkce": "auto",
  "client_type": "public",
  "user": "Practitioner/147533",
  "iat": 1768196295,
  "exp": 1768196595
}
```
* context 物件，也稱為 Launch Context，就是操作 EHR 系統中當時的狀態，包含了病患 ID(當前工作病人的 FHIR Patient id。此 ID 會自動套用到 patient/* scope。)、指向 EHR UI styling 設定（顏色、字型、logo 等），App 應抓取該 URL 以符合 EHR 視覺風格、是否需要顯示病患橫幅(這裡是 true，表示 EHR 要求 App 顯示 Patient Banner 病人識別區塊，App 不能隱藏病人身分。)等資訊
* OAuth Client 設定：
  * client_id：在 EHR 系統中註冊的 Client ID
  * redirect_uri：授權伺服器在授權完成之後，將會導向回來的頁面端點
  * scope：這次授權所要求的存取範圍
  * pkce：這裡使用的是自動產生 PKCE code challenge 與 code verifier 的方式
  * client_type：這是一個公開型態的 Client
* 對於 Scope 內容，這裡包含了 OpenID Connect 所需的 openid、fhirUser、profile 等基本欄位，另外，還有 launch/patient 代表這是一個以病患為中心的 App，還有 patient/*.read 代表要求讀取病患相關的所有資源的權限，另外，也要求了讀取 Encounter、MedicationRequest、ServiceRequest 等資源的權限；因為有 patient/*.read，後面三個其實是冗餘的，但某些 EHR 會要求顯式列出。
* 使用者身分：表示目前登入 EHR 並啟動 App 的人是：FHIR Practitioner 資源：Practitioner/147533
* Token 時效：從底下兩個值可以看到授權碼的有效時間，兩者相差 300 秒（5 分鐘），代表這是一個 短效 launch token / assertion，只用來換 access token。
  * iat：Issued At，表示這個授權碼的發行時間
  * exp：Expiration，表示這個授權碼的過期時間   
* 第二個參數為 state 參數，這裡將會用於要接收來自於之前頁面的狀態 Key 值，如此，在這個 ExchangeToken 頁面中，便可以取得之前頁面所儲存的相關參數值
![](../Images/cs2025-904.png)
* 接著，頁面程式碼會自動進行存取權杖的交換動作，並且使用取得的存取權杖來呼叫 FHIR API 來讀取病患資料
* 最後，頁面將會顯示出讀取到的病患資料內容，包含病患名稱與出生日期
![](../Images/cs2025-903.png)

## 剖析 Access Token 的輔助方法與模型

在這個 ExchangeToken 頁面的程式碼中，為了要能夠順利地交換取得存取權杖 Access Token 與呼叫 FHIR API 來讀取病患資料，必須要先建立兩個輔助的程式碼檔案，分別是 SmartResponse 資料模型與 JwtHelper 輔助支援類別。

### SmartResponse.cs

* 當要透過授權碼來取得存取權杖時候，將會得到如下的 JSON 物件

```json
{
   "access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzY29wZSI6Im9wZW5pZCBmaGlyVXNlciBwcm9maWxlIGxhdW5jaC9wYXRpZW50IHBhdGllbnQvKi5yZWFkIHBhdGllbnQvRW5jb3VudGVyLnJlYWQgcGF0aWVudC9NZWRpY2F0aW9uUmVxdWVzdC5yZWFkIHBhdGllbnQvU2VydmljZVJlcXVlc3QucmVhZCIsImNvbnRleHQiOnsibmVlZF9wYXRpZW50X2Jhbm5lciI6dHJ1ZSwic21hcnRfc3R5bGVfdXJsIjoiaHR0cHM6Ly90aGFzLm1vaHcuZ292LnR3L3NtYXJ0LXN0eWxlLmpzb24iLCJwYXRpZW50IjoicGF0aWVudC1UV0VNUi1ETVMtMTIzNDU2Nzg5LUExMjM0NTY3ODkifSwiY2xpZW50X2lkIjoic21hcnQtYXBwIiwiZmhpclVzZXIiOiJQcmFjdGl0aW9uZXIvMTQ3NTMzIiwiaWF0IjoxNzY4MTk3Mjk5LCJleHAiOjE3NjgyMDA4OTl9.3PzzzPfCpFFC7MuFCXuWYSa99j_yvb6DOG0nU2Jjmuw",
   "token_type":"Bearer",
   "expires_in":3600,
   "scope":"openid fhirUser profile launch/patient patient/*.read patient/Encounter.read patient/MedicationRequest.read patient/ServiceRequest.read",
   "id_token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcm9maWxlIjoiUHJhY3RpdGlvbmVyLzE0NzUzMyIsImZoaXJVc2VyIjoiUHJhY3RpdGlvbmVyLzE0NzUzMyIsImF1ZCI6InNtYXJ0LWFwcCIsInN1YiI6IjFhN2M3NjFiN2MzM2NiNGRmNGVhMDcwZTMwZmRiZjUxMzUxNjM1OWZjYzQ5YzNhZDYzYjY0MmU0NTEwOTc2NDQiLCJpc3MiOiJodHRwczovL3RoYXMubW9ody5nb3YudHcvdi9yNC9maGlyIiwiaWF0IjoxNzY4MTk3Mjk5LCJleHAiOjE3NjgyMDA4OTl9.UW3SlDfwfJZAHb4TZDwKtrVv8Gc7MPsna9B3HLiyHZgDhsO8QBvA_Ck5p988iIxz1poek13_Ul7YIMowaTkSo-N6-F7KQX2zDKcnxl_l51wMtecNgDSmwf5V6HFUAsFppTwcz6OUkquXM76JwDnz8S03yHT--sbJxVt8JNfNMkWLx-KZa2g65K3xkSE_iVLMCabnK6eb8SWe8vPqljs62wt1GjdWutPfxJKZfBZJ_Oab4w0O8xrRJZ9UIb6Wq3e_5wBw58GtDEGkZTmXlIawoQywdJ1Z7nIzxMj7bupbMPbtQOQXcB_aK2o4LpoHjX1J2oMdHK4dsDh6bBlm8S9_rA",
   "need_patient_banner":true,
   "smart_style_url":"https://thas.mohw.gov.tw/smart-style.json",
   "patient":"patient-TWEMR-DMS-123456789-A123456789"
}
```

* 該 json 物件結構符合：SMART on FHIR + OpenID Connect Hybrid
* SMART 規格允許 EHR 把 launch context 同時放在 token 內與 response body，此時，拿到的是「雙重綁定」版本（高可信）。
* 這裡取得的 access_token 經過解碼之後的內容如下，這個存取權杖的有效期先將會是1 小時

```json
{
  "scope": "openid fhirUser profile launch/patient patient/*.read patient/Encounter.read patient/MedicationRequest.read patient/ServiceRequest.read",
  "context": {
    "need_patient_banner": true,
    "smart_style_url": "https://thas.mohw.gov.tw/smart-style.json",
    "patient": "patient-TWEMR-DMS-123456789-A123456789"
  },
  "client_id": "smart-app",
  "fhirUser": "Practitioner/147533",
  "iat": 1768197299,
  "exp": 1768200899
}
```  

* 這裡取得的 id_token 經過解碼之後的內容如下，是 RS256，內容是 OpenID Connect 身分聲明

```json
{
  "profile": "Practitioner/147533",
  "fhirUser": "Practitioner/147533",
  "aud": "smart-app",
  "sub": "1a7c761b7c33cb4df4ea070e30fdbf513516359fcc49c3ad63b642e451097644",
  "iss": "https://thas.mohw.gov.tw/v/r4/fhir",
  "iat": 1768197299,
  "exp": 1768200899
}
```

* 因此，這裡需要一個 SmartResponse 資料模型來對應這些欄位。透過了反序列化程式碼 `SmartResponse smartResponse = JsonSerializer.Deserialize<SmartResponse>(json);` 便可以將上述的 JSON 內容，轉換成 SmartResponse 物件來使用。

* 在 Smart On FHIR 文件內的 [2.2.3.3 Launch context arrives with your access_token](https://build.fhir.org/ig/HL7/smart-app-launch/scopes-and-launch-context.html) 文件中，有提到授權伺服器回傳的存取權杖 Access Token 的 JSON 格式內容，這個內容需要一個資料模型來對應這些欄位，因此，我們建立了 SmartResponse.cs 這個資料模型來對應這些欄位。

| Launch context parameter | Example value | Meaning |
|-------------------------|--------------|---------|
| `patient` | `"123"` | String value with a patient id, indicating that the app was launched in the context of FHIR Patient 123. If the app has any patient-level scopes, they will be scoped to Patient 123. |
| `encounter` | `"123"` | String value with an encounter id, indicating that the app was launched in the context of FHIR Encounter 123. |
| `fhirContext` | `[{"reference": "Appointment/123"}]` | Array of objects referring to any resource type other than `"Patient"` or `"Encounter"`. See details below. |
| `need_patient_banner` | `true` or `false` (boolean) | Boolean value indicating whether the app was launched in a UX context where a patient banner is required (when `true`) or may not be required (when `false`). An app receiving a value of `false` might not need to take up screen real estate displaying a patient banner. |
| `intent` | `"reconcile-medications"` | String value describing the intent of the application launch (see notes below). |
| `smart_style_url` | `"https://ehr/styles/smart_v1.json"` | String URL where the EHR’s style parameters can be retrieved (for apps that support styling). |
| `tenant` | `"2ddd6c3a-8e9a-44c6-a305-52111ad302a2"` | String conveying an opaque identifier for the healthcare organization that is launching the app. This parameter is intended primarily to support EHR Launch scenarios. |

### JwtHelper.cs

`JwtHelper.DecodePayload` 這個方法主要做「解析並美化顯示 JWT 的 Payload」：

* 基本驗證
   - 檢查 `jwt` 字串是否為 `null`、空白。
   - 用 `.` 切割 JWT，確認至少有兩段（Header 和 Payload），否則當成格式錯誤。

* 取出並還原 Payload 的 Base64Url
   - 只拿中間那段 `parts[1]`（Payload）。
   - 把 Base64Url 格式的字元：
     - `-` 換成 `+`
     - `_` 換成 `/`
   - 依長度補上 `=` 或 `==`，讓它變成合法的 Base64 字串。

* 解碼成 JSON 字串
   - 使用 `Convert.FromBase64String` 把 Base64 轉成位元組陣列。
   - 用 `Encoding.UTF8.GetString` 把位元組轉成 JSON 字串。

* 格式化（縮排、美化）JSON
   - 用 `JsonDocument.Parse` 解析 JSON。
   - 用 `Utf8JsonWriter` 並設定 `Indented = true`，把 JSON 重新寫入到 `MemoryStream`。
   - 最後再用 `Encoding.UTF8.GetString` 從 `MemoryStream` 轉回「排版好、易讀的 JSON 字串」。

* 回傳結果
   - 回傳的是「格式化過的」JWT Payload JSON，而不是原始壓縮在 JWT 裡的 Base64Url 字串。

## ExchangeToken.razor 頁面程式碼說明
* 在這個頁面，使用了底下語法，來接收使用 Query String 參數傳遞過來的值
```razor
[SupplyParameterFromQuery(Name = "code")]
public string? Code { get; set; }
[SupplyParameterFromQuery(Name = "state")]
public string? State { get; set; }
```

* 在 OnAfterRenderAsync 方法中，這裡會先檢查是否為第一次渲染頁面
* 在第一次渲染事件發生的時候，將會執行底下的敘述

```csharp
await SetAuthCodeAsync();

StateHasChanged();
await System.Threading.Tasks.Task.Delay(5000);

SmartResponse smartResponse = await GetAccessTokenAsync();
await GetPatientAsync(smartResponse);
StateHasChanged();
```

在 `ExchangeToken.razor.cs` 這段被你標出的程式碼：

```csharp
if (firstRender)
{
    await SetAuthCodeAsync();

    StateHasChanged();
    await System.Threading.Tasks.Task.Delay(5000);

    SmartResponse smartResponse = await GetAccessTokenAsync();
    await GetPatientAsync(smartResponse);
    StateHasChanged();
}
```

* 在 Blazor 元件第一次渲染後，自動完成「授權碼處理 → 換取 token → 讀取病人資料」這一連串動作：
* await SetAuthCodeAsync()
   - 從 `OAuthStateStoreService` 載入對應 `State` 的設定物件，寫入目前的 `Code`、`State`，更新 `SmartAppSettingService`，並用 `JwtHelper` 解碼 `AuthCode`（當作 JWT）存到 `AuthorizationCodeJson`，方便頁面顯示。
* GetAccessTokenAsync()
   - 建立 `POST` 請求到 `TokenUrl`，帶上 `grant_type=authorization_code`、`code`、`redirect_uri`、`launch`，呼叫 OAuth token endpoint。
   - 回傳 JSON 反序列化成 `SmartResponse`（裡面包含 `AccessToken`、`PatientId` 等）。
* GetPatientAsync(smartResponse)
   - 透過 `FhirClient` 讀取 `Patient/{smartResponse.PatientId}`，把結果存到 `patient` 欄位，並把 `isReadPatient` 設為 `true`。

### GetAccessTokenAsync 方法說明：
* 為了要使用授權碼來交換存取權杖 Access Token，這裡將會使用底下程式碼建立 Post 方法要用到的 Payload

```csharp
Dictionary<string, string> requestValues = new Dictionary<string, string>()
    {
        { "grant_type", "authorization_code" },
        { "code", SmartAppSettingService.Data.AuthCode },
        { "redirect_uri", SmartAppSettingService.Data.RedirectUrl },
        { "launch", SmartAppSettingService.Data.Launch }
    };
```

* 然後，建立 HttpRequestMessage 物件，並且設定好相關屬性值
* 接著，透過 HttpClient 透過 https://thas.mohw.gov.tw/v/r4/auth/token 端點來發送 Post 請求，這個服務端點則是透過授權伺服器所提供的 Token 端點
* 最後，取得回傳的 JSON 內容，並且反序列化成 SmartResponse 物件來使用


### GetPatientAsync 方法說明：
* 這個方法主要是使用取得的存取權杖 Access Token 來呼叫 FHIR API 來讀取病患資料
* 首先，建立 HttpClient 物件，並且設定好 BaseAddress 屬性值
* 接著，設定好 Authorization 標頭內容，這裡使用 Bearer Token 的方式來進行授權
* 然後，建立 FhirClient 物件，並且將剛剛建立的 HttpClient 物件傳遞給 FhirClient 來使用
* 最後，使用 FhirClient 物件來讀取 Patient 資源，並且將讀取到的病患資料存到 patient 欄位中，並且將 isReadPatient 屬性設為 true，代表已經成功讀取到病患資料
* 這裡將會透過 `await fhirClient.ReadAsync<Patient>($"Patient/{smartResponse.PatientId}");` 方法來做到這個需求，也就是說，想要能夠存取 FHIR Server 上的資源，將會使用 FhirClient 物件來進行相關的操作，透過轉換成為 FHIR API，便可以很方便地來進行相關的存取動作
* 這樣一來，頁面便可以顯示出病患的相關資料內容

## 結論
透過這個 ExchangeToken 頁面的程式碼，成功地完成了使用授權碼來交換存取權杖 Access Token，並且使用取得的存取權杖來呼叫 FHIR API 來讀取病患資料的需求，這樣一來，便可以順利地完成 EHR Launch 的整個流程。

若要取得這篇文章的程式碼，可以透過 [https://github.com/vulcanlee/CSharp2025/tree/main/SmartEHRLaunch1](https://github.com/vulcanlee/CSharp2025/tree/main/SmartEHRLaunch1) 取得










## 建立 SmartResponse 資料模型

* 在專案節點下
* 滑鼠右擊 [Models] 資料夾
* 點選彈出功能表的 [加入] > [新增項目] 項目
* 在文字輸入盒內輸入檔案名稱為 [SmartResponse.cs]
* 然後點擊右下方的 [新增] 按鈕
* 使用底下程式碼內容來取代剛剛建立的 [SmartResponse.cs] 程式碼檔案內容

```csharp
using System.Text.Json.Serialization;

namespace SmartEHRLaunch1.Models;

/// <summary>
/// Class to deserialize SMART auth responses.
/// </summary>
public class SmartResponse
{
    [JsonPropertyName("need_patient_banner")]
    public bool NeedPatientBanner { get; set; }

    [JsonPropertyName("smart_style_url")]
    public string SmartStyleUrl { get; set; }

    [JsonPropertyName("patient")]
    public string PatientId { get; set; }

    [JsonPropertyName("token_type")]
    public string TokenType { get; set; }

    [JsonPropertyName("scope")]
    public string Scopes { get; set; }

    [JsonPropertyName("client_id")]
    public string ClientId { get; set; }

    [JsonPropertyName("expires_in")]
    public int ExpiresInSeconds { get; set; }

    [JsonPropertyName("id_token")]
    public string IdToken { get; set; }

    [JsonPropertyName("access_token")]
    public string AccessToken { get; set; }

    [JsonPropertyName("refresh_token")]
    public string RefreshToken { get; set; }
}
```

## 建立解析 JWT 內容的輔助支援類別

* 在專案節點下
* 滑鼠右擊 [Helpers] 資料夾
* 點選彈出功能表的 [加入] > [新增項目] 項目
* 在文字輸入盒內輸入檔案名稱為 [JwtHelper.cs]
* 然後點擊右下方的 [新增] 按鈕
* 使用底下程式碼內容來取代剛剛建立的 [JwtHelper.cs] 程式碼檔案內容

```csharp
using Hl7.Fhir.Model;
using Hl7.Fhir.Rest;
using Microsoft.AspNetCore.Components;
using SmartEHRLaunch1.Helpers;
using SmartEHRLaunch1.Models;
using SmartEHRLaunch1.Servicers;
using System.Net.Http.Headers;
using System.Text.Json;

namespace SmartEHRLaunch1.Components.Pages;

public partial class ExchangeToken
{
    [Inject]
    public SmartAppSettingService SmartAppSettingService { get; init; }
    [Inject]
    public OAuthStateStoreService OAuthStateStoreService { get; init; }

    public string AuthorizationCodeJson { get; private set; } = string.Empty;

    protected override async System.Threading.Tasks.Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await SetAuthCodeAsync();

            StateHasChanged();
            await System.Threading.Tasks.Task.Delay(5000);

            SmartResponse smartResponse = await GetAccessTokenAsync();
            await GetPatientAsync(smartResponse);
            StateHasChanged();
        }
    }

    public async System.Threading.Tasks.Task SetAuthCodeAsync()
    {
        await System.Threading.Tasks.Task.Yield();
        var SmartAppSettingModelItem = await OAuthStateStoreService.LoadAsync<SmartAppSettingModel>(State);

        SmartAppSettingModelItem.AuthCode = Code;
        SmartAppSettingModelItem.State = State;

        SmartAppSettingService.UpdateSetting(SmartAppSettingModelItem);
        Console.WriteLine($"Retrive state: {SmartAppSettingService.Data.State}");

        AuthorizationCodeJson = JwtHelper.DecodePayload(SmartAppSettingModelItem.AuthCode);
    }

    public async System.Threading.Tasks.Task<SmartResponse> GetAccessTokenAsync()
    {
        Dictionary<string, string> requestValues = new Dictionary<string, string>()
            {
                { "grant_type", "authorization_code" },
                { "code", SmartAppSettingService.Data.AuthCode },
                { "redirect_uri", SmartAppSettingService.Data.RedirectUrl },
                { "launch", SmartAppSettingService.Data.Launch }
            };

        HttpRequestMessage request = new HttpRequestMessage()
        {
            Method = HttpMethod.Post,
            RequestUri = new Uri(SmartAppSettingService.Data.TokenUrl),
            Content = new FormUrlEncodedContent(requestValues),
        };

        HttpClient client = new HttpClient();

        HttpResponseMessage response = await client.SendAsync(request);

        if (!response.IsSuccessStatusCode)
        {
            System.Console.WriteLine($"Failed to exchange code for token!");
            throw new Exception($"Unauthorized: {response.StatusCode}");
        }

        string json = await response.Content.ReadAsStringAsync();

        System.Console.WriteLine($"----- Authorization Response -----");
        System.Console.WriteLine(json);
        System.Console.WriteLine($"----- Authorization Response -----");

        SmartResponse smartResponse = JsonSerializer.Deserialize<SmartResponse>(json);
        return smartResponse;
    }

    public async System.Threading.Tasks.Task GetPatientAsync(SmartResponse smartResponse)
    {
        // 1. 先建立 HttpClient，預設好 Authorization header
        HttpClient httpClient = new HttpClient
        {
            BaseAddress = new Uri(SmartAppSettingService.Data.FhirServerUrl)
        };
        httpClient.DefaultRequestHeaders.Authorization =
            new AuthenticationHeaderValue("Bearer", smartResponse.AccessToken);

        // 2. 用這個 HttpClient 建立 FhirClient
        FhirClientSettings settings = new FhirClientSettings
        {
            PreferredFormat = ResourceFormat.Json
        };

        FhirClient fhirClient = new FhirClient(SmartAppSettingService.Data.FhirServerUrl, httpClient, settings);

        // 3. 讀取 Patient
        patient = await fhirClient.ReadAsync<Patient>($"Patient/{smartResponse.PatientId}");

        System.Console.WriteLine($"Read back patient: {patient.Name[0].ToString()}");

        isReadPatient = true;
    }
}
```

## 建立 ExchangeToken 頁面


* 在剛剛建立的 [SmartEHRLaunch1] 專案中，滑鼠右擊 [Components] > [Pages] 資料夾
* 點選彈出功能表的 [加入] > [Razor 元件] 項目
* 在 [新增項目] 對話窗的最下方之 [名稱] 欄位中，輸入頁面名稱為 [ExchangeToken]
* 然後點擊右下方的 [新增] 按鈕
* 在剛剛建立的 [ExchangeToken.razor] 頁面中，輸入底下的程式碼內容

```razor
@page "/ExchangeToken"
@using Hl7.Fhir.Model
@using SmartApp.Models
@inject NavigationManager NavigationManager

@if (isReadPatient == false)
{
    <h3>請稍後，讀取病患資料</h3>
    <div>Code:@Code</div>
    <div>State:@State</div>

    @if (string.IsNullOrEmpty(AuthorizationCodeJson))
    {
        <pre>(no payload)</pre>
    }
    else
    {
        <div>Authorization Code 的實際內容</div>
        <!-- 使用 <pre> 讓空白與換行都顯示出來 -->
        <pre>@AuthorizationCodeJson</pre>
    }
}
else
{
    <div>
        <p>病患名稱</p>
        <p>@patient.Name[0].ToString()</p>
        <p>病患名稱</p>
        <p>@patient.BirthDate.ToString()</p>
    </div>
}

@code {
    [SupplyParameterFromQuery(Name = "code")]
    public string? Code { get; set; }
    [SupplyParameterFromQuery(Name = "state")]
    public string? State { get; set; }

    bool isReadPatient = false;
    Patient patient = new();
}
```

* 接下要來建立這個頁面的 Code Behind 的程式碼檔案
* 在剛剛建立的 [SmartEHRLaunch1] 專案中，滑鼠右擊 [Components] > [Pages] 資料夾
* 點選彈出功能表的 [加入] > [新增項目] 項目
* 底下操作將會是在 [顯示精簡檢視] 模式下操作
* 在 [新增項目] 對話窗的文字輸入盒中，輸入檔案名稱為 [ExchangeToken.razor.cs]
* 然後點擊右下方的 [新增] 按鈕
* 在剛剛建立的 [Launch.razor.cs] 程式碼檔案中，輸入底下的程式碼內容

```csharp
using Hl7.Fhir.Model;
using Hl7.Fhir.Rest;
using Microsoft.AspNetCore.Components;
using SmartEHRLaunch1.Helpers;
using SmartEHRLaunch1.Models;
using SmartEHRLaunch1.Servicers;
using System.Net.Http.Headers;
using System.Text.Json;

namespace SmartEHRLaunch1.Components.Pages;

public partial class ExchangeToken
{
    [Inject]
    public SmartAppSettingService SmartAppSettingService { get; init; }
    [Inject]
    public OAuthStateStoreService OAuthStateStoreService { get; init; }

    public string AuthorizationCodeJson { get; private set; } = string.Empty;

    protected override async System.Threading.Tasks.Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await SetAuthCodeAsync();

            StateHasChanged();
            await System.Threading.Tasks.Task.Delay(5000);

            SmartResponse smartResponse = await GetAccessTokenAsync();
            await GetPatientAsync(smartResponse);
            StateHasChanged();
        }
    }

    public async System.Threading.Tasks.Task SetAuthCodeAsync()
    {
        await System.Threading.Tasks.Task.Yield();
        var SmartAppSettingModelItem = await OAuthStateStoreService.LoadAsync<SmartAppSettingModel>(State);

        SmartAppSettingModelItem.AuthCode = Code;
        SmartAppSettingModelItem.State = State;

        SmartAppSettingService.UpdateSetting(SmartAppSettingModelItem);
        Console.WriteLine($"Retrive state: {SmartAppSettingService.Data.State}");

        AuthorizationCodeJson = JwtHelper.DecodePayload(SmartAppSettingModelItem.AuthCode);
    }

    public async System.Threading.Tasks.Task<SmartResponse> GetAccessTokenAsync()
    {
        Dictionary<string, string> requestValues = new Dictionary<string, string>()
            {
                { "grant_type", "authorization_code" },
                { "code", SmartAppSettingService.Data.AuthCode },
                { "redirect_uri", SmartAppSettingService.Data.RedirectUrl },
                { "launch", SmartAppSettingService.Data.Launch }
            };

        HttpRequestMessage request = new HttpRequestMessage()
        {
            Method = HttpMethod.Post,
            RequestUri = new Uri(SmartAppSettingService.Data.TokenUrl),
            Content = new FormUrlEncodedContent(requestValues),
        };

        HttpClient client = new HttpClient();

        HttpResponseMessage response = await client.SendAsync(request);

        if (!response.IsSuccessStatusCode)
        {
            System.Console.WriteLine($"Failed to exchange code for token!");
            throw new Exception($"Unauthorized: {response.StatusCode}");
        }

        string json = await response.Content.ReadAsStringAsync();

        System.Console.WriteLine($"----- Authorization Response -----");
        System.Console.WriteLine(json);
        System.Console.WriteLine($"----- Authorization Response -----");

        SmartResponse smartResponse = JsonSerializer.Deserialize<SmartResponse>(json);
        return smartResponse;
    }

    public async System.Threading.Tasks.Task GetPatientAsync(SmartResponse smartResponse)
    {
        // 1. 先建立 HttpClient，預設好 Authorization header
        HttpClient httpClient = new HttpClient
        {
            BaseAddress = new Uri(SmartAppSettingService.Data.FhirServerUrl)
        };
        httpClient.DefaultRequestHeaders.Authorization =
            new AuthenticationHeaderValue("Bearer", smartResponse.AccessToken);

        // 2. 用這個 HttpClient 建立 FhirClient
        FhirClientSettings settings = new FhirClientSettings
        {
            PreferredFormat = ResourceFormat.Json
        };

        FhirClient fhirClient = new FhirClient(SmartAppSettingService.Data.FhirServerUrl, httpClient, settings);

        // 3. 讀取 Patient
        patient = await fhirClient.ReadAsync<Patient>($"Patient/{smartResponse.PatientId}");

        System.Console.WriteLine($"Read back patient: {patient.Name[0].ToString()}");

        isReadPatient = true;
    }
}
```

