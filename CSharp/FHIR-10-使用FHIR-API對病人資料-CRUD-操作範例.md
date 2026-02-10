# FHIR 10 使用 FHIR API 對病人資料 CRUD 操作範例

想要對 FHIR Server 進行資料的存取與管理，通常會使用 FHIR API 來進行各種 CRUD（Create、Read、Update、Delete）操作。當然，想要在 .NET C# 環境內進行這樣的操作，是可以自行建立一個 [HttpClient] 物件，然後手動撰寫各種 HTTP 請求來呼叫 FHIR API。不過，這樣的做法會比較繁瑣，且需要自行處理許多細節。

為了簡化相關的操作，Hl7.Fhir.R4 套件提供了一個方便的 [FhirClient] 類別，讓開發者可以更輕鬆地與 FHIR Server 進行互動。這個類別封裝了許多常用的 FHIR API 操作，讓我們可以透過簡單的方法呼叫來完成各種 CRUD 任務。

在這篇文章中，將會針對 FHIR 中的 Patient 資源，示範如何建立病患資料，以及進行 CRUD（Create、Read、Update、Delete）操作。在此之前，先來了解何謂 FHIR Patient 病患資源以及有哪些欄位可以使用。

FHIR 是目前醫療領域最重要的跨系統交換標準之一，其核心定義了許多常用的臨床與行政資訊資源，FHIR Patient 資源代表接受照護「個人（或動物）」的基本資料，用來建立、辨識、管理病人的人口統計學與行政資訊。Patient 不包含病人病歷細節（例如臨床紀錄），但常被其他資料資源所引用。

Patient 資源常見欄位包括：

* 基礎識別與狀態欄位
  * identifier : 多個病人識別碼，如醫療記錄號碼（MRN）、健保號碼等。FHIR 中用 identifier 存放這些外部系統識別碼。
  * active : Boolean 值表示該病人紀錄是否目前有效，例如是否為現行病人或已退件狀態。
* 名稱與通訊資訊
  * name : 病人的姓名（HumanName）。可以有多個，如正式姓名、別名等。
  * telecom : 聯絡方式清單，例如電話、Email、SMS 等。
* 基本人口數據
  * gender : 行政性別。Allowed values：male | female | other | unknown。
  * birthDate : 出生日期。
  * deceased[x] : 病人是否已逝。依型別可選 boolean 或 datetime。
* 位址與社會狀態
  * address : 病人地址資訊（可以有多筆）。 
  * maritalStatus : 婚姻狀態（CodeableConcept）。
* 其他識別與描述
  * multipleBirth[x] : 是否為多胞胎（Boolean 或整數表出生序）。
  * photo : 病人照片圖檔（Attachment）。
* Contact
  * contact : 病人的緊急聯絡人或代表者，其下可含：
    * relationship：與病人的關係
    * name：名稱
    * telecom：聯絡方式
    * address：地址
  這些欄位通常在臨床場景中被用來標註病人的緊急聯絡、監護人或代理人資訊。
* Communication
  * communication : 病人可用的語言資訊
    * language：ISO language code
    * preferred：是否為優先語言
  此元素有助於跨語言環境中的照護與問診溝通。
* Patient 的 Reference（鏈結）
    * FHIR 設計中，Patient 資源本身並不直接含有大量臨床病歷，但其他 FHIR 資源會使用 **Reference(Patient)** 去指向病人。FHIR 官方文件列出了很多可能會參考 Patient 的資源。
    * 支援 Reference 到 Patient 的資源`,下列為官方 Patient 規範中列出的部分資源，這些資源可能透過 reference 欄位指向 Patient
      * Encounter：臨床訪視事件其 subject 可能是 Patient
      * Observation：病人的測量或評估結果
      * Condition：病人的臨床診斷
      * Immunization：免疫接種紀錄
      * MedicationRequest / MedicationAdministration / MedicationStatement：用藥相關資源
      * AllergyIntolerance：過敏或不耐症
      * Procedure：執行的醫療程序
      * Claim / ExplanationOfBenefit：保險理賠申請
      * CarePlan / CareTeam：照護計畫與團隊
      …以及許多其他以 Patient 為核心的臨床紀錄相關資源。
* Patient 會 Reference 其他資源
  * 在 Patient 的欄位裡也包含對其他資源的 reference，用來建立直接關聯：
  * generalPractitioner : 患者的主要照護提供者，可指向：
    * Organization
    * Practitioner
    * PractitionerRole
    這讓病人可以追蹤其主要照護醫師或醫療機構。
  * managingOrganization : 管理此病人紀錄的組織。
  * link.other : 用來指向另一個 Patient 或 RelatedPerson，用於描述此紀錄與另一個實體之間的關係（例如合併紀錄）。
* Patient Resource 的 Extensions
  * FHIR 的一大特色是擴展性，當基礎欄位不足以描述某些地區或業務需求時，可以定義 **Extension**：
  * Extension 可以加在任一 Patient 層級，用以補充**無於基本資源中的資訊**
  * 常見使用場景包括：
    * 種族與族裔資訊
    * 國籍、語言偏好的詳細資訊
    * 臟器捐贈者狀態
    * 其他地區政府需求的欄位
  * 定義 Extension 時，需提供唯一的 URL 作為識別，並且說明其結構與使用方式。

接下來我們將透過一個簡單的範例程式碼，示範如何使用 Hl7.Fhir.R4 套件中的 FhirClient 類別，先來針對單一資源來了解如何做到 CRUD ： 新增、修改、刪除、查詢的操作，要如何能夠在 .NET C# 中來完成。為了要簡化體驗開發過程，這裡將會採用 主控台應用程式 (Console App) 的方式來進行示範。

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


