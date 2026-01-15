# FHIR 08 在 Standalone 建立新頁面來接收授權碼進而取得存取權杖並呼叫 FHIR API

在上一篇文章中 [FHIR 07 說明透過 Standalone 取得授權碼的程式碼做法與實測過程](https://csharpkh.blogspot.com/2026/01/FHIR-07-Standalone.html)  ，我們已經成功地透過 Standalone Launch 流程取得了授權碼 (Authorization Code) 。

根據 Smart On FHIR 規格文件說明內容，我們接下來要做的工作是：
1. 建立一個新的頁面 (Razor Page) 來接收授權碼
2. 在這個頁面中，使用授權碼來交換取得存取權杖 (Access Token)
3. 使用取得的存取權杖來呼叫 FHIR API 以讀取病患資料

為了要完成這些需求，需要額外再增加一個頁面、讀取存取權杖用的資料模型、解析JWT內容，用來取出指定的病患ID與呼叫 FHIR Server API的程式碼。

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

## 建立 VitalSignsResult 資料模型

* 在專案節點下
* 滑鼠右擊 [Models] 資料夾
* 點選彈出功能表的 [加入] > [新增項目] 項目
* 在文字輸入盒內輸入檔案名稱為 [VitalSignsResult.cs]
* 然後點擊右下方的 [新增] 按鈕
* 使用底下程式碼內容來取代剛剛建立的 [VitalSignsResult.cs] 程式碼檔案內容

```csharp
namespace SmartStandalone1.Models;

public class VitalSignsResult
{
    public string HeightValue { get; set; }
    public string HeightUnit { get; set; }
    public string WeightValue { get; set; }
    public string WeightUnit { get; set; }
}
```

## 建立 VitalSignsResult 資料模型

* 在專案節點下
* 滑鼠右擊 [Models] 資料夾
* 點選彈出功能表的 [加入] > [新增項目] 項目
* 在文字輸入盒內輸入檔案名稱為 [VitalSignsResult.cs]
* 然後點擊右下方的 [新增] 按鈕
* 使用底下程式碼內容來取代剛剛建立的 [VitalSignsResult.cs] 程式碼檔案內容

```csharp
namespace SmartStandalone1.Models;

public class VitalSignsResult
{
    public string HeightValue { get; set; }
    public string HeightUnit { get; set; }
    public string WeightValue { get; set; }
    public string WeightUnit { get; set; }
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
@inject NavigationManager NavigationManager

@if (isReadPatient == false)
{
    <h3>請稍後，讀取病患資料</h3>
}
else
{
    <div>
        <p>Id</p>
        <p>@patient.Id</p>
        <p>病患名稱</p>
        <p>@patient.Name[0].ToString()</p>
        <p>出生日</p>
        <p>@patient.BirthDate.ToString()</p>
        <p>身高</p>
        <p>@heightAndWeight.HeightValue <span> @heightAndWeight.HeightUnit</span></p>
        <p>體重</p>
        <p>@heightAndWeight.WeightValue <span> @heightAndWeight.WeightUnit</span></p>
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
* 在剛剛建立的 [ExchangeToken.razor.cs] 程式碼檔案中，輸入底下的程式碼內容

```csharp
using Hl7.Fhir.Model;
using Hl7.Fhir.Rest;
using Microsoft.AspNetCore.Components;
using SmartStandalone1.Models;
using SmartStandalone1.Servicers;
using System.Net.Http.Headers;
using System.Text.Json;

namespace SmartStandalone1.Components.Pages;

public partial class ExchangeToken
{
    VitalSignsResult heightAndWeight = new VitalSignsResult();
    [Inject]
    public SmartAppSettingService SmartAppSettingService { get; init; }
    [Inject]
    public OAuthStateStoreService OAuthStateStoreService { get; init; }

    protected override async System.Threading.Tasks.Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await SetAuthCodeAsync();
            SmartResponse smartResponse = await GetAccessTokenAsync();
            await GetPatientAsync(smartResponse);
            heightAndWeight = await GetHeightAndWeightAsync(smartResponse);
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

        FhirClientSettings settings = new FhirClientSettings
        {
            PreferredFormat = ResourceFormat.Json
        };

        FhirClient fhirClient = new FhirClient(SmartAppSettingService.Data.FhirServerUrl, httpClient, settings);

        patient = await fhirClient.ReadAsync<Patient>($"Patient/{smartResponse.PatientId}");

        System.Console.WriteLine($"Read back patient: {patient.Name[0].ToString()}");

        isReadPatient = true;
    }

    private async System.Threading.Tasks.Task<VitalSignsResult> GetHeightAndWeightAsync(SmartResponse smartResponse)
    {
        HttpClient httpClient = new HttpClient
        {
            BaseAddress = new Uri(SmartAppSettingService.Data.FhirServerUrl)
        };
        httpClient.DefaultRequestHeaders.Authorization =
            new AuthenticationHeaderValue("Bearer", smartResponse.AccessToken);

        FhirClientSettings settings = new FhirClientSettings
        {
            PreferredFormat = ResourceFormat.Json
        };

        FhirClient fhirClient = new FhirClient(SmartAppSettingService.Data.FhirServerUrl, httpClient, settings);

        // 查詢該病人的 Observation（限制常見 vital-sign codes）
        SearchParams searchParams = new SearchParams()
            .Where($"patient={smartResponse.PatientId}")
            .Where("category=vital-signs")
            .Include("Observation:patient")
            .LimitTo(50);

        Bundle bundle = await fhirClient.SearchAsync<Observation>(searchParams);

        decimal? heightValue = null;
        string? heightUnit = null;
        decimal? weightValue = null;
        string? weightUnit = null;

        foreach (Bundle.EntryComponent entry in bundle.Entry)
        {
            if (entry.Resource is not Observation obs)
            {
                continue;
            }

            // Observation.code.coding[].code 比對 LOINC
            string? loincCode = obs.Code?.Coding?.FirstOrDefault()?.Code;

            if (loincCode is null)
            {
                continue;
            }

            Quantity? quantity = obs.Value as Quantity;
            if (quantity is null)
            {
                continue;
            }

            if (loincCode == "8302-2")
            {
                // 身高
                if (quantity.Value.HasValue)
                {
                    heightValue = (decimal)quantity.Value.Value;
                    heightUnit = quantity.Unit ?? quantity.Code;
                }
            }
            else if (loincCode == "29463-7")
            {
                // 體重
                if (quantity.Value.HasValue)
                {
                    weightValue = (decimal)quantity.Value.Value;
                    weightUnit = quantity.Unit ?? quantity.Code;
                }
            }
        }

        System.Console.WriteLine($"Height: {heightValue} {heightUnit}");
        System.Console.WriteLine($"Weight: {weightValue} {weightUnit}");

        return new VitalSignsResult
        {
            HeightValue = heightValue?.ToString(),
            HeightUnit = heightUnit?.ToString(),
            WeightValue = weightValue?.ToString(),
            WeightUnit = weightUnit?.ToString()
        };
    }

    /// <summary>
    /// 判斷 Encounter 是否為門診 / 急診 / 住院。
    /// 真正的判斷規則要依實際 FHIR 伺服器的 coding 慣例調整。
    /// 這裡只示意用 Encounter.Class.Code 或 Type.Coding.Code 來區分。
    /// </summary>
    public bool IsOpdErIpdEncounter(Encounter encounter)
    {
        string? cls = encounter.Class?.Code;

        // 以下的 code 只是舉例，你需要依照實際 coding 規格（如 HL7 v2, local code set）調整
        if (string.Equals(cls, "AMB", StringComparison.OrdinalIgnoreCase))
        {
            // Ambulatory / 門診
            return true;
        }

        if (string.Equals(cls, "EMER", StringComparison.OrdinalIgnoreCase))
        {
            // Emergency / 急診
            return true;
        }

        if (string.Equals(cls, "IMP", StringComparison.OrdinalIgnoreCase))
        {
            // Inpatient / 住院
            return true;
        }

        // 也可以再看 Type.Coding 裡的 code 來判斷
        if (encounter.Type != null)
        {
            foreach (CodeableConcept type in encounter.Type)
            {
                Coding? coding = type.Coding?.FirstOrDefault();
                if (coding == null)
                {
                    continue;
                }

                string? code = coding.Code;

                // 依實際系統定義調整
                if (string.Equals(code, "OPD", StringComparison.OrdinalIgnoreCase) ||
                    string.Equals(code, "ER", StringComparison.OrdinalIgnoreCase) ||
                    string.Equals(code, "IPD", StringComparison.OrdinalIgnoreCase))
                {
                    return true;
                }
            }
        }

        return false;
    }

    /// <summary>
    /// 回傳簡化後的 Encounter 類型字串（OPD/ER/IPD/...）
    /// </summary>
    public string GetEncounterType(Encounter encounter)
    {
        string? cls = encounter.Class?.Code;

        if (string.Equals(cls, "AMB", StringComparison.OrdinalIgnoreCase))
        {
            return "OPD";
        }

        if (string.Equals(cls, "EMER", StringComparison.OrdinalIgnoreCase))
        {
            return "ER";
        }

        if (string.Equals(cls, "IMP", StringComparison.OrdinalIgnoreCase))
        {
            return "IPD";
        }

        // 也可以從 Type.Coding 推斷
        if (encounter.Type != null)
        {
            foreach (CodeableConcept type in encounter.Type)
            {
                Coding? coding = type.Coding?.FirstOrDefault();
                if (coding?.Code != null)
                {
                    return coding.Code;
                }
            }
        }

        return string.Empty;
    }
}
```

