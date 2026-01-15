# FHIR 09 說明 Standalone 建立 ExchangeToken 新頁面的程式碼做法與實測過程

在上一篇文章中 [FHIR 08 在 Standalone 建立新頁面來接收授權碼進而取得存取權杖並呼叫 FHIR API](https://csharpkh.blogspot.com/2026/01/FHIR-08-Standalone-FHIR-API.html)  ，已經完成了相關 Exchange Token 的程式碼，透過這些新增的程式碼與頁面，透過準備新得到的存取權杖 Access Token 成功地呼叫 FHIR API 來讀取病患資料。

首先，先來執行這個專案，看看執行成果，接著，再來說明這些程式碼的做法與實測過程。

## 執行結果說明

依照底下步驟操作，完成使用 Smart On FHIR 沙盒環境來進行 EHR Launch 的授權碼取得流程：

* 開啟並且執行 [SmartStandalone1]
* 在瀏覽器的位置列中，輸入 `https://localhost:7108/launch`
* 在 [Launch] 端點頁面中，將會顯示出文字 [請稍後，正在初始化中]
* 
* 接著，系統會自動將我們的瀏覽器，重新導向到授權伺服器
* 現在，將會進入到 FHIR 的授權伺服器，要取得授權碼，當取得了授權碼之後，就會使用 redirect_uri 參數，重新導向回到我們開發的頁面端點 `https://localhost:7108/ExchangeToken` ，並且在網址列中帶入授權碼參數 code
* 在網頁上，將會看到下面的畫面截圖，準備進入到身分驗證階段
* 在這裡使用預設的帳號與密碼，點選 [Login] 按鈕
* 現在的畫面將會切換到選擇病患的畫面
* 點選名稱為 [Geoffrey Abbott] 病患項目，這個病人位於第二頁，準備進入到下一個階段
* 這裡出現了 [Authorize App Launch] 對話窗，這裡需要你同意這個應用程式代表你，可以對 FHIR Server 做列出能力清單的操作。
* 此時，現在可以點選 [Approve] 按鈕，來同意這個應用程式的授權要求
* 當點選了 [Approve] 按鈕之後，系統將會重新導向回到我們的應用程式頁面端點 `https://localhost:7108/ExchangeToken` ，並且在網址列中帶入授權碼參數 code 與 state 參數
* 這時候，ExchangeToken 頁面將會被載入，並且開始執行頁面程式碼
* 之後，就會在螢幕上看到下面截圖畫面，這裡出現了從授權伺服器回傳的授權碼內容，這裡列出了 code & state 這兩個參數
* 當使用 code 這個 OAuth2 授權碼來交換存取權杖，將會得到底下的內容

```json
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzY29wZSI6Im9wZW5pZCBmaGlyVXNlciBwcm9maWxlIGxhdW5jaC9wYXRpZW50IHBhdGllbnQvKi5yZWFkIHBhdGllbnQvRW5jb3VudGVyLnJlYWQgcGF0aWVudC9NZWRpY2F0aW9uUmVxdWVzdC5yZWFkIHBhdGllbnQvU2VydmljZVJlcXVlc3QucmVhZCIsImNvbnRleHQiOnsibmVlZF9wYXRpZW50X2Jhbm5lciI6dHJ1ZSwic21hcnRfc3R5bGVfdXJsIjoiaHR0cHM6Ly90aGFzLm1vaHcuZ292LnR3L3NtYXJ0LXN0eWxlLmpzb24iLCJwYXRpZW50IjoiMmNkYTVhYWQtZTQwOS00MDcwLTlhMTUtZTFjMzVjNDZlZDVhIn0sImNsaWVudF9pZCI6InNtYXJ0LWFwcCIsImZoaXJVc2VyIjoiUHJhY3RpdGlvbmVyLzE0NzUzMyIsImlhdCI6MTc2ODQ2NDg5NywiZXhwIjoxNzY4NDY4NDk3fQ.haFOCN0Bx7hLyxmrvf-6_4MEDGnFBf8iATmV9TfX8NU","token_type":"Bearer","expires_in":3600,"scope":"openid fhirUser profile launch/patient patient/*.read patient/Encounter.read patient/MedicationRequest.read patient/ServiceRequest.read","id_token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJwcm9maWxlIjoiUHJhY3RpdGlvbmVyLzE0NzUzMyIsImZoaXJVc2VyIjoiUHJhY3RpdGlvbmVyLzE0NzUzMyIsImF1ZCI6InNtYXJ0LWFwcCIsInN1YiI6IjFhN2M3NjFiN2MzM2NiNGRmNGVhMDcwZTMwZmRiZjUxMzUxNjM1OWZjYzQ5YzNhZDYzYjY0MmU0NTEwOTc2NDQiLCJpc3MiOiJodHRwczovL3RoYXMubW9ody5nb3YudHcvdi9yNC9zaW0vV3pJc0lpSXNJaUlzSWtGVlZFOGlMREFzTUN3d0xDSWlMQ0lpTENJaUxDSWlMQ0lpTENJaUxDSWlMREFzTVN3aUlsMC9maGlyIiwiaWF0IjoxNzY4NDY0ODk3LCJleHAiOjE3Njg0Njg0OTd9.NxDdzAzT8wz1rl_sFGTCKUAE8gR0_WoVIVtgPK8jpi30UI0SWLSblp9a3RNqZTefKUStScrF8bKqV_oPkPE1cNB_jgy_X6SM1YawBE2J0zv_ShObwDfxunOPYmOOl1yfQf-qQuNP96ywAuFe3yljbnbvBd_zTfy5_H2ERSOxE0vN4Q8irY5p02YiJhWLTl62Q3E_aS2GZ4Kn-IGGnvAgmOWJEluVdVQ35s3UP-L_h272H8lS8iFs-5FO7xR4_iltleHBRZyBGMLnaZ-8IU_zPCxmcwBTLklLK1JE_f1HUFMNxa5itfb03tCV5kUC9wHTOdD_52h4jAhKrI1B1hSwdA","need_patient_banner":true,"smart_style_url":"https://thas.mohw.gov.tw/smart-style.json","patient":"2cda5aad-e409-4070-9a15-e1c35c46ed5a"}
```
* 從上面的內容中，可以看到幾個重要的欄位值
  * access_token：這就是我們要用來存取 FHIR API 的存取權杖
  * id_token：這是一個 JWT 格式的身分識別權杖
  * patient：這就是我們這次所選擇的病患資源 ID
* 透過上面提供的 Access Token 與 Patient ID ，接著，頁面程式碼會呼叫 FHIR API 來讀取病患資料
* 讀取到的病患資料內容，包含病患名稱與出生日期
* 最後，會取得該病人的身高與體重值，這裡是透過 FHIR 內的 Observation 資源來取得這些資料
* 下面的截圖，顯示了從 FHIR 伺服器讀取到的病患資料內容
![](../Images/cs2025-898.png)

## 剖析 Access Token 的輔助方法與模型

在這個 ExchangeToken 頁面的程式碼中，為了要能夠順利地交換取得存取權杖 Access Token 與呼叫 FHIR API 來讀取病患資料，必須要先建立兩個輔助的程式碼檔案，分別是 SmartResponse 資料模型與 JwtHelper 輔助支援類別。

### SmartResponse.cs

* 當要透過授權碼來取得存取權杖時候，將會得到剛剛提到的 JSON 物件
* 該 json 物件結構符合：SMART on FHIR + OpenID Connect Hybrid
* 這裡取得的 access_token 經過解碼之後的內容如下，這個存取權杖的有效期先將會是1 小時

```json
{
  "scope": "openid fhirUser profile launch/patient patient/*.read patient/Encounter.read patient/MedicationRequest.read patient/ServiceRequest.read",
  "context": {
    "need_patient_banner": true,
    "smart_style_url": "https://thas.mohw.gov.tw/smart-style.json",
    "patient": "2cda5aad-e409-4070-9a15-e1c35c46ed5a"
  },
  "client_id": "smart-app",
  "fhirUser": "Practitioner/147533",
  "iat": 1768464897,
  "exp": 1768468497
}
```  

* 這裡取得的 id_token 經過解碼之後的內容如下，是 RS256，內容是 OpenID Connect 身分聲明

```json
{
  "profile": "Practitioner/147533",
  "fhirUser": "Practitioner/147533",
  "aud": "smart-app",
  "sub": "1a7c761b7c33cb4df4ea070e30fdbf513516359fcc49c3ad63b642e451097644",
  "iss": "https://thas.mohw.gov.tw/v/r4/sim/WzIsIiIsIiIsIkFVVE8iLDAsMCwwLCIiLCIiLCIiLCIiLCIiLCIiLCIiLDAsMSwiIl0/fhir",
  "iat": 1768464897,
  "exp": 1768468497
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
heightAndWeight = await GetHeightAndWeightAsync(smartResponse);
StateHasChanged();
```

* 在 Blazor 元件第一次渲染後，自動完成「授權碼處理 → 換取 token → 讀取病人資料 → 讀取病人身高與體重」這一連串動作：
* await SetAuthCodeAsync()
   - 從 `OAuthStateStoreService` 載入對應 `State` 的設定物件，寫入目前的 `Code`、`State`，更新 `SmartAppSettingService`，並用 `JwtHelper` 解碼 `AuthCode`（當作 JWT）存到 `AuthorizationCodeJson`，方便頁面顯示。
* GetAccessTokenAsync()
   - 建立 `POST` 請求到 `TokenUrl`，帶上 `grant_type=authorization_code`、`code`、`redirect_uri`、`launch`，呼叫 OAuth token endpoint。
   - 回傳 JSON 反序列化成 `SmartResponse`（裡面包含 `AccessToken`、`PatientId` 等）。
* GetPatientAsync(smartResponse)
   - 透過 `FhirClient` 讀取 `Patient/{smartResponse.PatientId}`，把結果存到 `patient` 欄位，並把 `isReadPatient` 設為 `true`。
* GetHeightAndWeightAsync(smartResponse)
   - 透過 `FhirClient` 搜尋 `Observation`，條件是 `patient=smartResponse.PatientId` 且 `code` 是身高或體重的 LOINC code，取得最新的一筆身高與體重資料。

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

### GetHeightAndWeightAsync 方法說明：

下面是針對 `GetHeightAndWeightAsync` 方法，採用你給的 `GetPatientAsync` 說明風格所寫的說明內容：

### GetHeightAndWeightAsync 方法說明：
* 這個方法主要是使用取得的存取權杖 Access Token，透過 FHIR API 讀取指定病患的生命徵象（vital signs）觀測值，並從中擷取「身高」與「體重」資料
* 首先，建立 `HttpClient` 物件，並設定好 `BaseAddress` 為 FHIR Server 的 URL
* 接著，設定好 `Authorization` 標頭內容，這裡同樣是使用 Bearer Token 的方式來進行授權，Token 來源為 `smartResponse.AccessToken`
* 然後，建立 `FhirClientSettings`，並將 `PreferredFormat` 設定為 `ResourceFormat.Json`，代表後續與 FHIR Server 溝通時，預期使用 JSON 格式
* 之後，建立 `FhirClient` 物件，並將剛剛建立的 `HttpClient` 物件與 FHIR Server 的 URL 傳入，之後便可以透過這個 `FhirClient` 物件來呼叫 FHIR API
* 接下來，建立一個 `SearchParams` 查詢條件：
  * 使用 `patient={smartResponse.PatientId}` 限定只查詢該病患的資料  
  * 使用 `category=vital-signs` 限定只查詢屬於「生命徵象」類別的 Observation  
  * 使用 `LimitTo(50)` 限制最多回傳 50 筆資料
* 使用 `await fhirClient.SearchAsync<Observation>(searchParams);` 來呼叫 FHIR API，取得一個 `Bundle`，其中包含多筆 Observation 資源
* 接著，逐筆走訪 `bundle.Entry`，針對每一筆 `Observation` 做以下處理：
  * 先確認 `entry.Resource` 真的是 `Observation` 型別
  * 取出 `Observation.Code.Coding` 中第一筆 coding 的 `Code` 欄位，把它視為 LOINC 代碼
  * 僅處理 `Value` 為 `Quantity` 型別的 Observation，並從 `Quantity.Value` 取得實際數值，從 `Quantity.Unit` 或 `Quantity.Code` 取得單位
  * 當 LOINC 代碼是 `8302-2` 時，代表身高（Body height），將其數值與單位存到 `heightValue` 與 `heightUnit`
  * 當 LOINC 代碼是 `29463-7` 時，代表體重（Body weight），將其數值與單位存到 `weightValue` 與 `weightUnit`
* 這裡會將搜尋結果中最後一筆符合條件的 Observation 視為最終的身高或體重值（若有多筆同 code 的 Observation，後面的會覆蓋前面的變數值）
* 最後，將取得到的身高與體重數值與單位，轉成字串，包裝成一個 `VitalSignsResult` 物件並回傳
* 在 Blazor 頁面中，便可以利用這個欄位來顯示病患的身高與體重資訊

## 結論
透過這個 ExchangeToken 頁面的程式碼，成功地完成了使用授權碼來交換存取權杖 Access Token，並且使用取得的存取權杖來呼叫 FHIR API 來讀取病患資料的需求，這樣一來，便可以順利地完成 Standalone Launch 的整個流程。

若要取得這篇文章的程式碼，可以透過 [https://github.com/vulcanlee/CSharp2025/tree/main/SmartStandalone1](https://github.com/vulcanlee/CSharp2025/tree/main/SmartStandalone1) 取得










## 建立 SmartResponse 資料模型

* 在專案節點下
* 滑鼠右擊 [Models] 資料夾
* 點選彈出功能表的 [加入] > [新增項目] 項目
* 在文字輸入盒內輸入檔案名稱為 [SmartResponse.cs]
* 然後點擊右下方的 [新增] 按鈕
* 使用底下程式碼內容來取代剛剛建立的 [SmartResponse.cs] 程式碼檔案內容

```csharp
using System.Text.Json.Serialization;

namespace SmartStandalone1.Models;

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
using SmartStandalone1.Helpers;
using SmartStandalone1.Models;
using SmartStandalone1.Servicers;
using System.Net.Http.Headers;
using System.Text.Json;

namespace SmartStandalone1.Components.Pages;

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


* 在剛剛建立的 [SmartStandalone1] 專案中，滑鼠右擊 [Components] > [Pages] 資料夾
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
* 在剛剛建立的 [SmartStandalone1] 專案中，滑鼠右擊 [Components] > [Pages] 資料夾
* 點選彈出功能表的 [加入] > [新增項目] 項目
* 底下操作將會是在 [顯示精簡檢視] 模式下操作
* 在 [新增項目] 對話窗的文字輸入盒中，輸入檔案名稱為 [ExchangeToken.razor.cs]
* 然後點擊右下方的 [新增] 按鈕
* 在剛剛建立的 [Launch.razor.cs] 程式碼檔案中，輸入底下的程式碼內容

```csharp
using Hl7.Fhir.Model;
using Hl7.Fhir.Rest;
using Microsoft.AspNetCore.Components;
using SmartStandalone1.Helpers;
using SmartStandalone1.Models;
using SmartStandalone1.Servicers;
using System.Net.Http.Headers;
using System.Text.Json;

namespace SmartStandalone1.Components.Pages;

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

