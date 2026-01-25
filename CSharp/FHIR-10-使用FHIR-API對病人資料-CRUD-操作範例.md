# FHIR 10 使用 FHIR API 對病人資料 CRUD 操作範例

想要對 FHIR Server 進行資料的存取與管理，通常會使用 FHIR API 來進行各種 CRUD（Create、Read、Update、Delete）操作。當然，想要在 .NET C# 環境內進行這樣的操作，是可以自行建立一個 [HttpClient] 物件，然後手動撰寫各種 HTTP 請求來呼叫 FHIR API。不過，這樣的做法會比較繁瑣，且需要自行處理許多細節。

為了簡化相關的操作，Hl7.Fhir.R4 套件提供了一個方便的 [FhirClient] 類別，讓開發者可以更輕鬆地與 FHIR Server 進行互動。這個類別封裝了許多常用的 FHIR API 操作，讓我們可以透過簡單的方法呼叫來完成各種 CRUD 任務。

首先，FHIR R4 有多少種 Resource 呢？FHIR R4 正式定義了 **157 種 Resource**（不含 Extension / DataType / ComplexType），涵蓋臨床、管理、財務、公共衛生等領域。

最常見、最重要的基本 Resource，大約有 20 多種，這些 Resource 是我們在日常醫療資訊系統中最常會用到的。以下將常用 Resource 分為幾類，並描述用途與彼此關聯：

## 病人與照護相關

| Resource             | 用途                     | 跟其他的關聯                                      |
| -------------------- | ---------------------- | ------------------------------------------- |
| **Patient**          | 病人核心資料（姓名、性別、出生年月、識別碼） | 其他所有與病人相關的記錄都會 reference Patient            |
| **Practitioner**     | 醫療人員資料（醫師、護理、物理治療師等）   | Encounter, Procedure, CarePlan 等會 reference |
| **PractitionerRole** | 醫療人員角色（哪科、哪機構）         | 連結 Practitioner 與 Organization              |
| **RelatedPerson**    | 與病人有關的親屬等              | Patient 的關聯者                                |

## 照護事件與行為

| Resource        | 用途             | 關聯                              |
| --------------- | -------------- | ------------------------------- |
| **Encounter**   | 就診事件（門診、住院、急診） | Patient, Practitioner, Location |
| **Appointment** | 約診             | Patient, Practitioner, Schedule |
| **Procedure**   | 醫療處置（手術、檢查等）   | Encounter, Patient              |
| **CarePlan**    | 照護計畫           | Patient, Practitioner           |

**簡單例子：**

* Encounter → 描述一次病人在院內的流動
* Appointment → 有時病人需先掛號
* Procedure → 真正做的檢查或手術

## 3. 醫療資料（主要）

| Resource                     | 用途        | 常跟患者關聯                |
| ---------------------------- | --------- | --------------------- |
| **Observation**              | 測量 / 檢驗值  | vitals, lab result 等  |
| **Condition**                | 疾病 / 健康問題 | 記錄病人目前的診斷             |
| **Medication**               | 藥品資訊      | 供用於 Rx                |
| **MedicationRequest**        | 開立處方      | Patient, Practitioner |
| **MedicationAdministration** | 實際給藥狀態    | Patient, Practitioner |
| **ServiceRequest**           | 檢查/治療請求   | Patient, Practitioner |
| **DiagnosticReport**         | 檢驗報告      | Observation 集合        |
| **AllergyIntolerance**       | 過敏資訊      | Patient               |

**簡單例子：**

* Observation → 量血壓、血糖
* Condition → 糖尿病診斷
* MedicationRequest → 醫師開立藥物
* DiagnosticReport → 實驗室報告

## 4. 組織與地點

| Resource              | 用途          |
| --------------------- | ----------- |
| **Organization**      | 醫院 / 診所     |
| **Location**          | 醫療地點（病房、診間） |
| **HealthcareService** | 該地方提供什麼服務   |

## 5. 身分與安全

| Resource       | 用途          |
| -------------- | ----------- |
| **Consent**    | 病人同意/拒絕資料使用 |
| **AuditEvent** | 記錄誰做過什麼（稽核） |

## 6. 支援性的管理與財務

| Resource                    | 用途          |
| --------------------------- | ----------- |
| **OrganizationAffiliation** | 醫療機構的合作     |
| **Claim / ClaimResponse**   | 保險理賠請求與回覆   |
| **Coverage**                | 保險方案（是否有給付） |

## 簡單的使用流程與實際意義（用例）

### 情境：門診看診

1. **Patient** 進入門診
2. 建立 **Encounter**（這次就診）
3. 醫師用 **Observation** 記錄血壓
4. 開立 **MedicationRequest** 處方
5. 可能建立 **Condition**（病名）
6. 若做檢查，會有 **ServiceRequest** 與後續 **DiagnosticReport**

給初學者的建議學習順序

你可以按照以下順序理解與練習：

| 步驟 | 重點                                       |
| -- | ---------------------------------------- |
| 1  | 先掌握 Patient / Practitioner               |
| 2  | 再理解 Encounter / Appointment              |
| 3  | 再熟悉 Observation / Condition              |
| 4  | 最後看 MedicationRequest / DiagnosticReport |


在初步了解了 FHIR Server 中的資源類型與應用之後，接下來我們將透過一個簡單的範例程式碼，示範如何使用 Hl7.Fhir.R4 套件中的 FhirClient 類別，先來針對單一資源來了解如何做到 CRUD ： 新增、修改、刪除、查詢的操作，要如何能夠在 .NET C# 中來完成。為了要簡化體驗開發過程，這裡將會採用 主控台應用程式 (Console App) 的方式來進行示範。

## 建立 主控台應用程式 專案
* 開啟 Visual Studio 2026
* 選擇「建立新專案」
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇 [主控台應用程式] 項目
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 "csPatientCRUD"
* 然後點擊右下方「下一步」按鈕
* 接著會看到 [其他資訊] 對話窗
* 在這個對話窗內，確認使用底下的選項
    * 架構：.NET 10.0 (或更新版本)
    * 勾選 不要使用最上層陳述式 (這是我的個人習慣)
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 主控台應用程式 專案的建立

## 安裝 Hl7.Fhir.R4 套件

Hl7.Fhir.R4 套件是由 HL7 官方 FHIR 團隊發佈的 .NET SDK（程式開發套件），用來讓開發者在 C# / .NET 中以「物件模型」的方式存取與操作 FHIR R4（Release 4） 的所有資源。

根據在 .NET NuGet 上看到的資訊，其作者是 Firely（原名 Furore），其為 HL7 FHIR 官方 .NET 參考實作團隊，也是 寫 FHIR 規格的人，同時寫 .NET SDK，這正是目前 .NET 世界中標準且官方等級的 FHIR R4 SDK。

* 在 Visual Studio 的「方案總管」視窗中，右鍵點擊專案名稱
* 從右鍵選單中，選擇「管理 NuGet 套件」
* 在 NuGet 套件管理器視窗中，切換到「瀏覽」標籤頁
* 在搜尋框中，輸入 "Hl7.Fhir.R4" 並按下 Enter 鍵
* 從搜尋結果中，找到 "Hl7.Fhir.R4" 套件 並點擊它
* 在這裡的範例中，使用該套件的版本為 5.12.1
* 在右側的詳細資訊面板中，點擊「安裝」按鈕

## 撰寫程式碼
* 打開 Program.cs 檔案，並將其內容替換為以下程式碼：

```csharp
using Hl7.Fhir.Model;
using Hl7.Fhir.Rest;
using Hl7.Fhir.Serialization;

namespace csPatientCRUD;

internal class Program
{
    private const string FhirBaseUrl = "https://hapi.fhir.org/baseR4";

    static async System.Threading.Tasks.Task Main()
    {
        string GivenName = "Vulcan20250814111";
        string FamilynName = "Lee";

        var settings = new FhirClientSettings
        {
            PreferredFormat = ResourceFormat.Json,
            Timeout = 60_000
        };

        var httpClient = new HttpClient();

        var client = new FhirClient(FhirBaseUrl, httpClient, settings);

        try
        {
            #region ============== Create ==============
            string identityValue = $"MRN-{Guid.NewGuid():N}".Substring(0, 12);
            identityValue = "MRN-20240814A1";
            var newPatient = new Patient
            {
                Identifier =
                {
                    new Identifier("http://example.org/mrn", identityValue)
                },
                Name = { new HumanName().WithGiven(GivenName).AndFamily(FamilynName) },
                Gender = AdministrativeGender.Female,
                BirthDate = "1990-01-01",
                Telecom = { new ContactPoint(ContactPoint.ContactPointSystem.Phone, ContactPoint.ContactPointUse.Mobile, "0912-345-678") },
                Active = true
            };

            Console.WriteLine("Creating Patient ...");
            var json = newPatient.ToJson();
            Console.WriteLine($"JSON: {json}");
            var created = await client.CreateAsync(newPatient); // POST /Patient
            Console.WriteLine($"Created: id={created.Id}, version={created.Meta?.VersionId}");

            PressAnyKeyToContinue();
            #endregion

            #region ============== Read ==============
            Console.WriteLine("Reading Patient by id ...");
            var readBack = await client.ReadAsync<Patient>($"Patient/{created.Id}"); // GET /Patient/{id}
            Console.WriteLine($"Read: {readBack.Name?.FirstOrDefault()} | active={readBack.Active}");

            PressAnyKeyToContinue();
            #endregion

            #region ============== Update ==============
            Console.WriteLine("Updating Patient (add email, set active=false) ...");
            readBack.Active = false;
            readBack.Telecom.Add(new ContactPoint(ContactPoint.ContactPointSystem.Email, null, $"{GivenName}.{FamilynName}@example.org"));
            var updated = await client.UpdateAsync(readBack); // PUT /Patient/{id}
            Console.WriteLine($"Updated: version={updated.Meta?.VersionId}, telecom={string.Join(", ", updated.Telecom.Select(t => $"{t.System}:{t.Value}"))}");

            PressAnyKeyToContinue();
            #endregion

            #region ============== Search ==============
            // 使用 identifier 精準搜尋剛建立的 Patient
            Console.WriteLine($"Searching Patient by identifier '{identityValue}' ...");
            var bundle = await client.SearchAsync<Patient>(new string[]
            {
                $"identifier={identityValue}",
                "_count=5"
            }); // GET /Patient?identifier={identityValue}&_count=5
            Console.WriteLine($"Search total (if provided): {bundle.Total}");
            foreach (var entry in bundle.Entry ?? Enumerable.Empty<Bundle.EntryComponent>())
            {
                if (entry.Resource is Patient p)
                    Console.WriteLine($" - {p.Id} | {p.Name?.FirstOrDefault()} | active={p.Active}");
            }

            PressAnyKeyToContinue();
            #endregion

            #region ============== Delete ==============
            Console.WriteLine("Deleting Patient ...");
            await client.DeleteAsync($"Patient/{created.Id}"); // DELETE /Patient/{id}
            Console.WriteLine("Deleted.");

            // 驗證刪除（預期 404）
            try
            {
                await client.ReadAsync<Patient>($"Patient/{created.Id}");
                Console.WriteLine("⚠️ Still readable (server may be eventual consistent).");
            }
            catch (FhirOperationException foe) when ((int)foe.Status == 404 || (int)foe.Status == 410)
            {
                Console.WriteLine($"Confirmed {(int)foe.Status} {foe.Status} after delete.");
            }
            PressAnyKeyToContinue();
            #endregion
        }
        catch (FhirOperationException foe)
        {
            Console.WriteLine($"FHIR error: HTTP {(int)foe.Status} {foe.Status}");
            if (foe.Outcome is OperationOutcome oo)
            {
                foreach (var i in oo.Issue)
                    Console.WriteLine($" - {i.Severity} {i.Code}: {i.Details?.Text}");
            }
            else
            {
                Console.WriteLine(foe.Message);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("ERR: " + ex.Message);
        }
    }

    // press any key to continue
    private static void PressAnyKeyToContinue()
    {
        Console.WriteLine("Press any key to continue...");
        Console.ReadKey();
        Console.WriteLine("");
        Console.WriteLine("");
        Console.WriteLine("");

    }
}
```

# 執行程式

首先先來看這個專案的執行結果：

* 按下 F5 鍵或點擊「開始」按鈕來執行程式
* 這個專案將會依序針對 Patient 這個資源進行 Create 新增、Retrive 查詢、Update 修改、Search 搜尋、Delete 刪除 等操作
* 每個步驟都會在主控台視窗中顯示相關的訊息，並等待使用者按下任意鍵後繼續下一步
* 底下為實際操作過程的輸出文字

```
Creating Patient ...
JSON: {"resourceType":"Patient","identifier":[{"system":"http://example.org/mrn","value":"MRN-20240814A1"}],"active":true,"name":[{"family":"Lee","given":["Vulcan20250814111"]}],"telecom":[{"system":"phone","value":"0912-345-678","use":"mobile"}],"gender":"female","birthDate":"1990-01-01"}
Created: id=53797442, version=1
Press any key to continue...



Reading Patient by id ...
Read: Vulcan20250814111 Lee | active=True
Press any key to continue...



Updating Patient (add email, set active=false) ...
Updated: version=2, telecom=Phone:0912-345-678, Email:Vulcan20250814111.Lee@example.org
Press any key to continue...



Searching Patient by identifier 'MRN-20240814A1' ...
Search total (if provided): 1
 - 53797442 | Vulcan20250814111 Lee | active=False
Press any key to continue...



Deleting Patient ...
Deleted.
Confirmed 410 Gone after delete.
Press any key to continue...
```

# 如何進行程式碼設計說明

底下的操作，根據這個變數宣告：

```csharp
private const string FhirBaseUrl = "https://hapi.fhir.org/baseR4";
```

將會採用 HAPI FHIR Server 的 R4 版本作為呼叫對象。

* 在系統一開始執行前
* 將會建立一個 [FhirClientSettings] & [HttpClient] 物件
* 前者 [FhirClientSettings] 將用於設定 FHIR 用戶端與伺服器溝通行為 的組態物件，例如資料格式（JSON/XML）、逾時時間、是否允許重新導向與錯誤處理方式。
* 後者 [HttpClient] 則是用於實際發送 HTTP 請求與接收回應的物件
* 上面提到的三個物件，都會傳送給 [FhirClient] 建構式
* [FhirClient] 這個物件將用於與 FHIR 伺服器進行互動，並提供各種方法來執行 CRUD 操作

## 新增

* 為了方便日後反覆操作與展示，這裡的 Patient Identifier 將會採用固定職 `identityValue = "MRN-20240814A1";` 來進行
* 建立一個新的 [Patient] 物件，並設定其屬性，例如 Identifier、Name、Gender、BirthDate、Telecom、Active 等
* 這些屬性的意義為：
  * Identifier：病人的識別碼，這裡使用一個自訂的系統 URI 與剛剛設定的值
  * Name：病人的姓名
  * Gender：病人的性別
  * BirthDate：病人的出生日期
  * Telecom：病人的聯絡資訊，這裡設定一個手機號碼
  * Active：表示病人是否為活躍狀態
* 當然，對於 Patient 這個 FHIR Resource 還有其他屬性可以使用，這裡僅點綴說明而已
* 有了 Patient 這個物件之後，就可以透過 `wait client.CreateAsync(newPatient);` 來進行寫入到遠端 FHIR Server 上了
* 這個方法會發送一個 HTTP POST 請求到 FHIR 伺服器的 /Patient 端點，並將 Patient 物件序列化為 JSON 格式的請求主體
* 當伺服器成功處理請求後，會回傳一個包含新建立的 Patient 資源的回應
* 回傳的 Patient 物件會包含伺服器分配的唯一識別碼（ID）與版本資訊（VersionId）
* 顯示該 Patient 的 ID & 版本資訊

## 查詢

* 在新增 Patient 之後，可以使用其 ID 來查詢該病人的資料
* 使用 `await client.ReadAsync<Patient>($"Patient/{created.Id}");` 方法來根據剛剛建立的 Patient ID 來查詢該病人資料
* 這個方法會發送一個 HTTP GET 請求到 FHIR 伺服器的 /Patient/{id} 端點
* 當伺服器成功處理請求後，會回傳一個包含該 Patient 資源的回應
* 一旦取得了 Patient 之後，就會在螢幕上顯示該 Patient 的姓名與活躍狀態

## 修改

* 查詢到 Patient 之後，可以對其進行修改
* 這裡的範例是將 Patient 的 Active 屬性設為 false，並新增一個 Email 聯絡方式
* 在此，將會使用底下程式碼來修改 Patient 物件的屬性

```csharp
readBack.Active = false;
readBack.Telecom.Add(new ContactPoint(ContactPoint.ContactPointSystem.Email, null, $"{GivenName}.{FamilynName}@example.org"));
```

* 使用 `await client.UpdateAsync(readBack);` 方法來將修改後的 Patient 物件更新到 FHIR 伺服器
* 這個方法會發送一個 HTTP PUT 請求到 FHIR 伺服器的 /Patient/{id} 端點，並將修改後的 Patient 物件序列化為 JSON 格式的請求主體
* 當伺服器成功處理請求後，會回傳一個包含更新後的 Patient 資源的回應
* 顯示更新後的 Patient 版本資訊與聯絡方式

## 搜尋

* 所謂的搜尋，是指根據特定條件來查找符合條件的資源。在這個範例中，我們將使用 Patient 的 Identifier 來進行搜尋。
* 使用底下的程式碼來進行搜尋條件的設定

```csharp
var bundle = await client.SearchAsync<Patient>(new string[]
{
    $"identifier={identityValue}",
    "_count=5"
}); // GET /Patient?identifier={identityValue}&_count=5
```

* 這裡使用 [SearchAsync<T>] 方法來搜尋 Patient 資源，並傳入一個字串陣列作為搜尋參數
* 在這個例子中，我們使用 `identifier={identityValue}` 作為搜尋條件，這表示我們要搜尋具有特定 Identifier 的 Patient 資源
* 同時，我們也使用 `_count=5` 來限制回傳的結果數量為最多 5 筆
* 當伺服器成功處理搜尋請求後，會回傳一個包含符合條件的 Patient 資源的 Bundle 回應
* 在螢幕上顯示搜尋結果的總數量
* 所謂的 Bundle，是 FHIR 中用來封裝多個資源的容器，例如，在這裡將會檢查 bundle.Entry 項目，篩選出其中的 Patient 資源
* 然後逐一列出每個 Patient 的 ID、姓名與活躍狀態
* 這樣就可以看到所有符合搜尋條件的 Patient 資源

## 刪除
* 最後，我們可以刪除剛剛建立的 Patient 資源
* 使用 `await client.DeleteAsync($"Patient/{created.Id}");` 方法來刪除該 Patient 資源
* 這個方法會發送一個 HTTP DELETE 請求到 FHIR 伺服器的 /Patient/{id} 端點
* 當伺服器成功處理刪除請求後，該 Patient 資源將會被移除
* 為了確認刪除是否成功，我們可以嘗試再次查詢該 Patient 資源
* 預期會收到 404 Not Found 或 410 Gone 的回應，表示 該資源已經不存在
* 如果收到預期的回應，則表示刪除操作成功
* 顯示刪除確認的訊息

# 結論

對於一個 FHIR Resource 要對其進行新增、查詢、更新、篩選、刪除的操作，使用 Hl7.Fhir.R4 套件中的 FhirClient 類別，可以大幅簡化程式碼的撰寫與維護工作。 FhirClient 物件提供了相對應的方法，CreateAsync、ReadAsync、UpdateAsync、SearchAsync、DeleteAsync 等，讓開發者可以輕鬆地與 FHIR 伺服器進行互動，而不需要手動處理 HTTP 請求與回應的細節。

這些方法將會轉換成為 FHIR API 的呼叫方式，並自動處理資源的序列化與反序列化，讓開發者可以專注於業務邏輯的實現，而不需要擔心底層的通訊細節。


