# FHIR 12 建立三個醫院的 Organization 操作範例

在這篇文章中，將會針對 FHIR 中的 Organization 資源，示範如何建立三家醫院（院級 Organization）以及每家醫院底下的一個科室（Department Organization），並且用 `partOf` 把科室接到對應的院級 Organization 底下。

**Organization =「法人/機構/部門」的概念層級（conceptual structure）**，常用來表示：醫院（院級機構）、分院（同一法人不同院區也可）、科室/部門（例如 內科、放射科、檢驗科）、醫療團隊/單位（例如 安寧團隊、糖尿病衛教中心），並且在 FHIR 官方也明確提到：Organization 常用 **partOf** 組成階層；而 **Location** 更偏向「實體地點/空間」的階層（病房、門診區、檢查室等）。也就是說：> **「醫院/科室」通常就放 Organization（概念層級）；「空間/房間/病房/院區地址」通常放 Location（物理層級）。**

關於「醫院/科室」資源中的最主要的欄位將會有以下幾個：

* identifier（跨系統識別碼）
  用來放院內代碼、機構代碼、統編/院所代碼、HIS 內部代碼等（**跨系統對齊最重要**）。
* type（機構/單位類型）
  用來標示「這個 Organization 是醫院、診所、科室、或者是檢驗單位」等分類。
* name / alias（顯示名稱與別名）
  * name：正式名稱（例如「國立成功大學醫學院附設醫院」）
  * alias：常用簡稱（例如「成大醫院」「NCKUH」）
* telecom / address（聯絡方式與地址）
  院級 Organization 很常用 address（醫院地址）；科室通常可省略（地址更常落在 Location）。
* partOf（組織階層：科室隸屬哪家醫院）
  **科室/部門** 幾乎都用 `partOf` 指向院級 Organization，形成階層樹。

對於要建立一個 FHIR Resource，可以使用 POST 方法，對 FHIR Server 發出請求，並且在 Request Body 中放入符合 FHIR 結構的 JSON 內容。以下會示範如何建立三家醫院（院級 Organization）以及每家醫院底下的一個科室（Department Organization），並且用 `partOf` 把科室接到對應的院級 Organization 底下。

這裡將會是建立一個醫院的組織資源（Organization）

```json
{
  "resourceType": "Organization",
  "identifier": [
    { "system": "https://hospital.example.org/org-code", "value": "HOSP-001" }
  ],
  "type": [
    { "text": "Hospital" }
  ],
  "name": "國立成功大學醫學院附設醫院",
  "alias": ["成大醫院", "NCKUH"],
  "telecom": [
    { "system": "phone", "value": "+886-6-2353535" }
  ],
  "address": [
    { "text": "台南市北區勝利路138號" }
  ]
}
```

這裡將會是建立一個醫院科室（Department）資源，並且用 `partOf` 指向院級 Organization 的 ID：

```json
{
  "resourceType": "Organization",
  "identifier": [
    { "system": "https://hospital.example.org/dept-code", "value": "RAD" }
  ],
  "type": [
    { "text": "Department" }
  ],
  "name": "放射科",
  "partOf": { "reference": "Organization/1084" }
}
```

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
* 找到 [專案名稱] 欄位，輸入 `csOrganization` 作為專案名稱
* 在剛剛輸入的 [專案名稱] 欄位下方，確認沒有勾選 [將解決方案與專案至於相同目錄中] 這個檢查盒控制項
* 點選右下角的 [下一步] 按鈕
* 現在將會看到 [其他資訊] 對話窗
* 在 [架構] 欄位中，請選擇最新的開發框架，這裡選擇的 [架構] 是 : `.NET 10.0 (長期支援)`
* 在這個練習中，需要去勾選 [不要使用最上層陳述式(T)] 這個檢查盒控制項
  > 這裡的這個操作，可以由讀者自行決定是否要勾選這個檢查盒控制項
* 請點選右下角的 [建立] 按鈕

稍微等候一下，這個 背景工作服務 專案將會建立完成

## 安裝要用到的 NuGet 開發套件

因為開發此專案時會用到這些 NuGet 套件，請依照底下說明，將需要用到的 NuGet 套件安裝起來。

### 安裝 Newtonsoft.Json 套件

請依照底下說明操作步驟，將這個套件安裝到專案內

* 滑鼠右擊 [方案總管] 視窗內的 [專案節點] 下方的 [相依性] 節點
* 從彈出功能表清單中，點選 [管理 NuGet 套件] 這個功能選項清單
* 此時，將會看到 [NuGet: csJsonMerge] 視窗
* 切換此視窗的標籤頁次到名稱為 [瀏覽] 這個標籤頁次
* 在左上方找到一個搜尋文字輸入盒，在此輸入 `Hl7.Fhir.R4`
* 在視窗右方，將會看到該套件詳細說明的內容，其中，右上方有的 [安裝] 按鈕
* 點選這個 [安裝] 按鈕，將這個套件安裝到專案內

## 修改 Program.cs 類別內容

* 在專案中找到並且打開 [Program.cs] 檔案
* 將底下的程式碼取代掉 `Program.cs` 檔案中內容

```csharp
using Hl7.Fhir.Model;
using Hl7.Fhir.Rest;
using Hl7.Fhir.Serialization;

namespace csOrganization;

internal class Program
{
    private const string FHIR_BASE = "https://server.fire.ly";

    private const string ORG_ID_SYSTEM = "https://vulcan.org/fhir/identifier/org-code";
    private const string DEPT_ID_SYSTEM = "https://vulcan.org/fhir/identifier/dept-code";
    private record HospitalSeed(string Code, string LegalName, string DisplayName);

    static async System.Threading.Tasks.Task Main(string[] args)
    {
        var client = CreateFhirClient(FHIR_BASE);

        // 建立 3 家醫院（院級 Organization）
        var hospitals = new[]
        {
            new HospitalSeed("NCKUH", "國立成功大學醫學院附設醫院", "成大醫院"),
            new HospitalSeed("CMH",   "奇美醫院", "奇美"),
            new HospitalSeed("KGH",   "郭綜合醫院", "郭綜合")
        };

        Organization createdHospital = new Organization();
        Organization createdDept = new Organization();
        foreach (var h in hospitals)
        {
            // 建院級 Organization
            var hospitalOrg = BuildHospitalOrganization(h);

            try
            {
                createdHospital = await client.CreateAsync(hospitalOrg);
                Console.WriteLine($"已經建立醫院組織 : {h.DisplayName} => {createdHospital.Id}");
                Console.WriteLine($"{createdHospital.ToJson()}");
            }
            catch (FhirOperationException ex)
            {
                Console.WriteLine($"建立醫院組織發生錯誤 : {h.DisplayName}");
                Console.WriteLine(ex.Message);
                if (ex.Outcome != null) Console.WriteLine(ex.Outcome.ToJson());
                continue;
            }

            // 建立婦產科（Department）並 partOf 指向該醫院
            var obgynDept = BuildDepartmentOrganization(
                deptCode: $"{h.Code}-OBGYN",
                deptName: "婦產科",
                parentHospitalId: createdHospital.Id
            );

            try
            {
                createdDept = await client.CreateAsync(obgynDept);
                Console.WriteLine($"已經建立醫院科室 : {h.DisplayName} / 婦產科 => {createdDept.Id}");
                Console.WriteLine($"{createdDept.ToJson()}");
            }
            catch (FhirOperationException ex)
            {
                Console.WriteLine($"建立醫院科室發生錯誤 : {h.DisplayName} / 婦產科");
                Console.WriteLine(ex.Message);
                if (ex.Outcome != null) Console.WriteLine(ex.Outcome.ToJson());
            }

            Console.WriteLine("--------------------------------------------------");

            #region 移除剛剛建立的 醫院與科室 紀錄
            Console.WriteLine($"開始刪除測試資料 : {h.DisplayName} 以及其下的科室...");
            try
            {
                await client.DeleteAsync($"Organization/{createdDept.Id}");
            }
            catch (Exception ex)
            {
                Console.WriteLine("刪除測試資料發生錯誤 : ");
                Console.WriteLine(ex.Message);
            }
            try
            {
                await client.DeleteAsync($"Organization/{createdHospital.Id}");
            }
            catch (Exception ex)
            {
                Console.WriteLine("刪除測試資料發生錯誤 : ");
                Console.WriteLine(ex.Message);
            }
            #endregion
        }
    }

    private static FhirClient CreateFhirClient(string baseUrl)
    {
        var settings = new FhirClientSettings
        {
            PreferredFormat = ResourceFormat.Json,
            VerifyFhirVersion = true,
            Timeout = 60_000
        };

        var client = new FhirClient(baseUrl, settings);
        return client;
    }

    private static Organization BuildHospitalOrganization(HospitalSeed h)
    {
        Organization organization = new Organization
        {
            Active = true,
            Identifier = new List<Identifier>
            {
                new Identifier(ORG_ID_SYSTEM, h.Code) { Use = Identifier.IdentifierUse.Official }
            },
            Name = h.LegalName,
            Alias = new List<string> { h.DisplayName },
            Type = new List<CodeableConcept>
            {
                // HL7 organization-type: prov = Healthcare Provider
                new CodeableConcept(
                    "http://terminology.hl7.org/CodeSystem/organization-type",
                    "prov",
                    "Healthcare Provider",
                    text: "Hospital"
                )
            }
        };
        return organization;
    }

    private static Organization BuildDepartmentOrganization(string deptCode, string deptName, string parentHospitalId)
    {
        return new Organization
        {
            Active = true,
            Identifier = new List<Identifier>
            {
                new Identifier(DEPT_ID_SYSTEM, deptCode) { Use = Identifier.IdentifierUse.Official }
            },
            Name = deptName,
            Type = new List<CodeableConcept>
            {
                // HL7 organization-type: dept = Hospital Department
                new CodeableConcept(
                    "http://terminology.hl7.org/CodeSystem/organization-type",
                    "dept",
                    "Hospital Department",
                    text: deptName
                )
            },
            PartOf = new ResourceReference($"Organization/{parentHospitalId}")
        };
    }
}
```

在這段程式碼中，主要的邏輯是：
* 建立一個 FhirClient 物件，連接到指定的 FHIR Server。
  這裡是建立一個方法 [CreateFhirClient]，用來建立並且回傳一個 FhirClient 物件。這裡使用 [FhirClientSettings] 來設定一些連線的參數，例如：PreferredFormat、VerifyFhirVersion、Timeout 等。在這裡使用 FHIR_BASE = "https://server.fire.ly" 作為 FHIR Server 的基底 URL。
* 定義三家醫院的基本資訊（代碼、正式名稱、顯示名稱）。
  為了要完成這樣的需求，定義了 [hospitals] 這個陣列，裡面包含了三家醫院的基本資訊，這裡使用了一個 record 類別 [HospitalSeed] 來定義這些資訊的結構。
* 迴圈處理每一家醫院：
   - 建立院級 Organization 資源，並且發出 Create 請求到 FHIR Server。
     由於在這裡建立了一個 Organization 物件，所以透過 `await client.CreateAsync(hospitalOrg)` 這個方法，將這個 Organization 資源發出 Create 請求到 FHIR Server，並且等待回應。成功的話，會回傳剛剛建立的 Organization 資源，其中包含了 FHIR Server 分配的 ID。在這裡沒有使用特定的 Organization 路由，便可以透過 [Hl7.Fhir.R4] 這個 NuGet 套件，直接使用 `client.CreateAsync` 方法來發出請求。一旦成功在 FHIR Server 中建立了 Organization 之後，就會印出剛剛建立的醫院組織的 ID 與 JSON 內容。從這個回傳的 JSON 物件，可以得到該物件的 [Id] 值，這個 ID 就是剛剛在 FHIR Server 中建立的 Organization 的 ID。這裡將會是底下執行的其中一個結果 `{"resourceType":"Organization","id":"a9fed323-499f-4112-93c7-047bf0ac8e1c","meta":{"versionId":"8620de1d-0fa5-42eb-8afc-bfa547adc096","lastUpdated":"2026-02-08T05:19:44.574+00:00"},"identifier":[{"use":"official","system":"https://vulcan.org/fhir/identifier/org-code","value":"NCKUH"}],"active":true,"type":[{"coding":[{"system":"http://terminology.hl7.org/CodeSystem/organization-type","code":"prov","display":"Healthcare Provider"}],"text":"Hospital"}],"name":"國立成功大學醫學 院附設醫院","alias":["成大醫院"]}`，從這裡可看到 [Id] 值為 `a9fed323-499f-4112-93c7-047bf0ac8e1c`，這個 ID 就是剛剛在 FHIR Server 中建立的 Organization 的 ID。下次便可以透過這個服務端點來取得這個 Organization 物件值 `https://server.fire.ly/Organization/a9fed323-499f-4112-93c7-047bf0ac8e1c`。
   - 建立科室 Organization 資源，並且用 `partOf` 指向剛剛建立的院級 Organization，然後發出 Create 請求。
        在這裡建立了一個科室的 Organization 物件，並且在 `PartOf` 欄位中，使用 `ResourceReference` 來指向剛剛建立的院級 Organization 的 ID。這裡同樣使用 `await client.CreateAsync(obgynDept)` 這個方法，將這個科室的 Organization 資源發出 Create 請求到 FHIR Server，並且等待回應。成功的話，會回傳剛剛建立的科室 Organization 資源，其中包含了 FHIR Server 分配的 ID。在這裡沒有使用特定的 Organization 路由，便可以透過 [Hl7.Fhir.R4] 這個 NuGet 套件，直接使用 `client.CreateAsync` 方法來發出請求。一旦成功在 FHIR Server 中建立了科室 Organization 之後，就會印出剛剛建立的醫院科室的 ID 與 JSON 內容。從這個回傳的 JSON 物件，可以得到該物件的 [Id] 值，這個 ID 就是剛剛在 FHIR Server 中建立的科室 Organization 的 ID。這裡將會是底下執行的其中一個結果 `{"resourceType":"Organization","id":"f4e7b836-46c2-4668-bb1b-00e1bcadd241","meta":{"versionId":"659bd514-9318-4a05-88a5-0115e7390acc","lastUpdated":"2026-02-08T05:19:45.016+00:00"},"identifier":[{"use":"official","system":"https://vulcan.org/fhir/identifier/dept-code","value":"NCKUH-OBGYN"}],"active":true,"type":[{"coding":[{"system":"http://terminology.hl7.org/CodeSystem/organization-type","code":"dept","display":"Hospital Department"}],"text":"婦產科"}],"name":"婦產科","partOf":{"reference":"https://server.fire.ly/Organization/a9fed323-499f-4112-93c7-047bf0ac8e1c"}}`，從這裡可看到 [Id] 值為 `f4e7b836-46c2-4668-bb1b-00e1bcadd241`，這個 ID 就是剛剛在 FHIR Server 中建立的科室 Organization 的 ID。下次便可以透過這個服務端點來取得這個科室 Organization 物件值 `https://server.fire.ly/Organization/f4e7b836-46c2-4668-bb1b-00e1bcadd241`。
   - 最後，刪除剛剛建立的科室與院級 Organization 測試資料。


## 執行程式碼

* 按下 `F5` 鍵，開始執行這個程式
* 程式將會開始執行，並且在主控台視窗內，將會看到類似下圖的輸出結果

```
已經建立醫院組織 : 成大醫院 => a9fed323-499f-4112-93c7-047bf0ac8e1c
{"resourceType":"Organization","id":"a9fed323-499f-4112-93c7-047bf0ac8e1c","meta":{"versionId":"8620de1d-0fa5-42eb-8afc-bfa547adc096","lastUpdated":"2026-02-08T05:19:44.574+00:00"},"identifier":[{"use":"official","system":"https://vulcan.org/fhir/identifier/org-code","value":"NCKUH"}],"active":true,"type":[{"coding":[{"system":"http://terminology.hl7.org/CodeSystem/organization-type","code":"prov","display":"Healthcare Provider"}],"text":"Hospital"}],"name":"國立成功大學醫學 院附設醫院","alias":["成大醫院"]}
已經建立醫院科室 : 成大醫院 / 婦產科 => f4e7b836-46c2-4668-bb1b-00e1bcadd241
{"resourceType":"Organization","id":"f4e7b836-46c2-4668-bb1b-00e1bcadd241","meta":{"versionId":"659bd514-9318-4a05-88a5-0115e7390acc","lastUpdated":"2026-02-08T05:19:45.016+00:00"},"identifier":[{"use":"official","system":"https://vulcan.org/fhir/identifier/dept-code","value":"NCKUH-OBGYN"}],"active":true,"type":[{"coding":[{"system":"http://terminology.hl7.org/CodeSystem/organization-type","code":"dept","display":"Hospital Department"}],"text":"婦產科"}],"name":"婦產科","partOf":{"reference":"https://server.fire.ly/Organization/a9fed323-499f-4112-93c7-047bf0ac8e1c"}}
--------------------------------------------------
開始刪除測試資料 : 成大醫院 以及其下的科室...
已經建立醫院組織 : 奇美 => 3d4b4ed4-d326-4690-88a3-476070529528
{"resourceType":"Organization","id":"3d4b4ed4-d326-4690-88a3-476070529528","meta":{"versionId":"d8cfc4df-0703-4020-880c-2804db8df8e5","lastUpdated":"2026-02-08T05:19:46.269+00:00"},"identifier":[{"use":"official","system":"https://vulcan.org/fhir/identifier/org-code","value":"CMH"}],"active":true,"type":[{"coding":[{"system":"http://terminology.hl7.org/CodeSystem/organization-type","code":"prov","display":"Healthcare Provider"}],"text":"Hospital"}],"name":"奇美醫院","alias":["奇美"]}
已經建立醫院科室 : 奇美 / 婦產科 => d4e2ebf6-27aa-4fda-925b-e95dc296ff42
{"resourceType":"Organization","id":"d4e2ebf6-27aa-4fda-925b-e95dc296ff42","meta":{"versionId":"a42c52c8-f040-4df7-9fce-0d1d81dca20b","lastUpdated":"2026-02-08T05:19:46.61+00:00"},"identifier":[{"use":"official","system":"https://vulcan.org/fhir/identifier/dept-code","value":"CMH-OBGYN"}],"active":true,"type":[{"coding":[{"system":"http://terminology.hl7.org/CodeSystem/organization-type","code":"dept","display":"Hospital Department"}],"text":"婦產科"}],"name":"婦產科","partOf":{"reference":"https://server.fire.ly/Organization/3d4b4ed4-d326-4690-88a3-476070529528"}}
--------------------------------------------------
開始刪除測試資料 : 奇美 以及其下的科室...
已經建立醫院組織 : 郭綜合 => a7b92d4e-8bad-4e28-a838-0d1f3605bfdd
{"resourceType":"Organization","id":"a7b92d4e-8bad-4e28-a838-0d1f3605bfdd","meta":{"versionId":"bbc9d5fe-c3ca-479d-8006-3edd1113f0ca","lastUpdated":"2026-02-08T05:19:47.735+00:00"},"identifier":[{"use":"official","system":"https://vulcan.org/fhir/identifier/org-code","value":"KGH"}],"active":true,"type":[{"coding":[{"system":"http://terminology.hl7.org/CodeSystem/organization-type","code":"prov","display":"Healthcare Provider"}],"text":"Hospital"}],"name":"郭綜合醫院","alias":["郭綜合"]}
已經建立醫院科室 : 郭綜合 / 婦產科 => 182c533e-d552-4b88-94c4-33a2e9cdecd2
{"resourceType":"Organization","id":"182c533e-d552-4b88-94c4-33a2e9cdecd2","meta":{"versionId":"156fe61e-b558-4f56-86ea-222b7946f5fa","lastUpdated":"2026-02-08T05:19:48.069+00:00"},"identifier":[{"use":"official","system":"https://vulcan.org/fhir/identifier/dept-code","value":"KGH-OBGYN"}],"active":true,"type":[{"coding":[{"system":"http://terminology.hl7.org/CodeSystem/organization-type","code":"dept","display":"Hospital Department"}],"text":"婦產科"}],"name":"婦產科","partOf":{"reference":"https://server.fire.ly/Organization/a7b92d4e-8bad-4e28-a838-0d1f3605bfdd"}}
--------------------------------------------------
開始刪除測試資料 : 郭綜合 以及其下的科室..
```
