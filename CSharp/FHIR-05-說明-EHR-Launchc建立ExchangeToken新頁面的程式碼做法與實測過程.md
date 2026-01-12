# FHIR 05 說明 EHR Launchc 建立 ExchangeToken 新頁面的程式碼做法與實測過程

在上一篇文章中 [FHIR 03 說明透過 EHR Launch 取得授權碼的程式碼做法與實測過程
](https://csharpkh.blogspot.com/2026/01/FHIR-03--EHR-Launch-.html)  ，我們已經成功地透過 EHR Launch 流程取得了授權碼 (Authorization Code) 。

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

