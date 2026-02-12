# FHIR 14 對於 FHIR DiagnosticReport 的操作範例

最近正在進行一個 SMART on FHIR 應用程式的開發，這個應用程式會從 FHIR 伺服器讀取病患資料(身高、體重、性別、年紀)，接著會由操作者上傳 L3 DICOM 影像道系統中，一旦使用者觸發了 [AI推論] 功能之後，將會把這些數據與影像檔案傳送到後端 Web API 內，並交由 AI 推論系統進行分析後得到這個病患的身體組成指標與一個五彩影像圖片，此時，這個 SMART on FHIR 應用系統，將會取得這些資訊，接著需要這些AI推論結果要寫入到 FHIR 伺服器中。

這裡遇到的第一個問題就是，這些資訊需要寫到 FHIR Server 的那些資源內，並且要採用甚麼樣的格式與內容轉成 FHIR 需要的格式資源，才能夠讓這些資訊在 FHIR 伺服器內被妥善的儲存與管理，並且能夠被其他系統讀取與使用。

面對這樣的需求，需要將 AI 分析結果與產生後的圖片寫入到 FHIR 內，會有不同的做法，而且各有其優缺點，以下將會說明兩種常見的做法：

* PNG 圖片外部連結：方法一，使用 URL 外部參考

**適合**：你不想把 PNG 當成 FHIR 內的「資源」，只是想在報告裡附一張圖。

```
Patient (test-patient-001)
  ▲               ▲
  │ subject       │ subject
  │               │
  │         Observation (1筆，含6個 component)
  │               │ • SMI, SMD, IMAT
  │               │ • LAMA, NAMA, MYOSTEATOSIS
  │               ▲
  │               │ result[]
  │               │
DiagnosticReport
      │
      │ presentedForm[].Url
      ▼
PNG 圖片 URL (外部連結: http://localhost/result/2311)
```

* PNG 圖片編碼：方法二，使用 Media 資源

這裡將會採用將這個圖片當成一個 Media 資源的方式來處理，並且在 DiagnosticReport 內以 result[] 的方式來參照這個 Media 資源。

**適合**：你想把 PNG 當成 FHIR 內的「資源」，並且希望在報告裡附一張圖。

```
Patient
  ▲
  │ subject
  │
DiagnosticReport  ── result[] ──► Observation(1筆，含6個component)
      │                              │
      │ (選配) imagingStudy[]         │ derivedFrom[]
      ▼                              ▼
  ImagingStudy (選配)                PNG
```

比較這兩種做法的差異與優缺點如下：

| 方式 | 優點 | 缺點 |
|---|---|---|
| URL 外部連結 | 不需要將圖片存入 FHIR 伺服器，節省儲存空間 | 依賴外部服務，可能會有存取問題 |
| Media 資源 | 將圖片當成 FHIR 資源，方便管理與存取 | 需要將圖片存入 FHIR 伺服器，增加儲存需求 |

在此的考量將會是採用第一種方案，也就是將圖片的 URL 參考寫入到 DiagnosticReport 的 presentedForm[] 內，這樣就不需要把圖片當成一個 FHIR 資源來管理，而是直接在報告裡附上圖片的連結，這樣的做法比較簡單，而且似乎對於 FHIR Server 的負擔不會過重。

對於這次的需求，也需要針對 DiagnosticReport 這個資源進行操作，因此，這裡也對於 DiagnosticReport 資源做個了解說明，

DiagnosticReport 是 FHIR Workflow 中的一種事件型資源，其主要用途為：

* 彙整和表達完整的診斷服務成果，涵蓋文字敘述、編碼結果、影像或可附加完整報告（如 PDF）。
* 結合診斷程序上下游關係，例如檢驗請求、觀察結果、醫療就診事件等。
* 與 Observation 形成一對多的關係：報告可能引用多個 Observation 來表示原子值（例如血液檢查各項指標）。
* 能支援多樣診斷類別：實驗室、影像、病理、心臟檢查等。

從臨床流程來看，DiagnosticReport 是 [檢查完成後的產品] ，可連結到請求來源（例如 ServiceRequest）、檢體（Specimen）、受檢者（Patient）、醫療事件（Encounter）、解讀者（Practitioner）等多種資源。

## DiagnosticReport 的欄位解構：名稱、意義與實際用法

以下依照官方定義逐一解析 DiagnosticReport 項目（含基本資源層級與臨床需求）：

* identifier ： 報告的商業識別碼，可為多個（例如各系統內部 ID、實驗室編號等）。此欄位利於跨系統辨識同一報告。
* basedOn ： 表示這份 DiagnosticReport 是基於何種請求資源所產生，如 CarePlan、MedicationRequest、NutritionOrder 或 ServiceRequest。這是一種 workflow 追蹤，用於回溯診斷產出的起源。
* status ： 報告的狀態，例如 registered、partial、preliminary、final 等，影響臨床使用時機（是否為最終結果）。
* category ： 代表報告所屬的服務類型，例如實驗室（lab）、影像（imaging）等，用以檢索與分類。代表報告所屬的服務類型，例如實驗室（lab）、影像（imaging）等，用以檢索與分類。
* code ： 為必填欄位，用來標示這份報告的具體診斷性質（如一般實驗室報告、特定掃描診斷等），典型使用 LOINC 等編碼系統。
* subject ： **核心欄位**，這是報告的主體（subject），通常會 reference Patient。FHIR 的設計允許 subject 為 Patient、Group、Device 或 Location，但在大部分臨床上下文中的常見用法是指向 Patient（受檢者）。
這個 reference 是 DiagnosticReport 與 Patient 之間最直接的連結點。
* encounter ： 連結到 Encounter 資源，用於表明這份報告關聯到何種就診事件（例如門診、住院事件等），有助於維持臨床事件的時間與上下文關係。
* effective[x] ： 表示報告結果與檢查執行的時間相關資訊，可以是單一時間（effectiveDateTime）或時間區間（effectivePeriod），用於時間序列與診療路徑分析。
* issued ： 報告何時正式發布或可供查閱。這在流程控制與資料版本管理中非常重要。
* performer ： 這是執行該報告之診斷服務的實體參考，可能是 Practitioner、PractitionerRole、Organization 或 CareTeam。可用於責任區分與品質評估。
* resultsInterpreter ： 引用那些負責最終結果解讀之專業角色（例如主治醫師或解讀者 team）。引用那些負責最終結果解讀之專業角色（例如主治醫師或解讀者 team）。
* specimen ： 如果診斷基於檢體（例如血液、組織切片），會 Reference Specimen 資源。這能讓前後端系統將報告結果與原始檢體檔案串連。
* result ： DiagnosticReport 與 Observation 資源最重要的關聯之一：這表示此報告對“各別觀察值”的引用（例如血中紅血球濃度這種原子值），通常是 0..* 多對多。
* imagingStudy ： 在影像報告中，直接引用 ImagingStudy，使接收端能查詢完整的影像內容（例如 DICOM Study）以供瀏覽與視覺化。
* media ： 若診斷報告涉及附加的影像（例如拍攝圖片、病理切片圖像），透過 media 子元素設定圖像相關的 meta 與連結。
* conclusion / conclusionCode ： 以文字或可編碼方式提供測試的臨床結論或總結，用於提高理解或自動化決策支援（如結論標籤）。
* presentedForm ： 提供可下載的完整報告形式（例如 PDF、Word、影像檔等），適合直接呈現在使用者介面或儲存作為法律/醫療文件。

## 🔗 Resource 之間的 Reference 與 Patient 的關連鏈

在 DiagnosticReport 中，與 Patient 相關或可能有跨資源 Reference 的欄位包括：

* **subject** → Patient（最直接）
* **encounter** → Encounter（此欄位中 Encounter 通常也含有對 Patient 的 reference）
* **specimen** → Specimen（Specimen 通常會 reference Patient/Encounter）
* **result** → Observation（Observation 通常 reference Patient）
* **basedOn** → 可能 reference ServiceRequest 或其他 request，這些 request 也會 reference Patient
* **performer / resultsInterpreter** → 可能 reference Practitioner / Organization，但這些角色通常也有關聯 Patient 通常不直接 reference
* **imagingStudy** → ImagingStudy 內部也 reference Patient 以表明影像結果是針對誰執行的

因此，從 DiagnosticReport → subject → Patient → Observation / Encounter / Specimen / ImagingStudy，可以建立一條臨床事件詳細鏈結的資料關係圖.

DiagnosticReport 是用來描述診斷報告的資源，它可以包含多個 Observation 作為結果，並且可以有一個或多個圖片或其他附件作為報告的呈現形式。在這裡，我們會把 AI 分析後的六個指標數值寫入到 Observation 內，而把五彩圖片的 URL 參考寫入到 DiagnosticReport 的 presentedForm[] 內。

以下是我在開發過程中對於 FHIR DiagnosticReport 資源的操作範例說明。

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

namespace csDiagnosticReport;

internal class Program
{
    private const string FhirBaseUrl = "http://10.1.1.113:8080/fhir";

    private const string CsBodyComp = "https://example.org/fhir/CodeSystem/body-composition";
    private const string CsObsCategory = "http://terminology.hl7.org/CodeSystem/observation-category";
    private const string CsV2_0074 = "http://terminology.hl7.org/CodeSystem/v2-0074";
    private const string Ucum = "http://unitsofmeasure.org";
    private const string patientId = "test-patient-001"; // 固定的 Patient ID
    private const string AIResultImageUrl = "http://localhost/result/2311";
    static async System.Threading.Tasks.Task Main(string[] args)
    {
        using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
        var ct = cts.Token;

        var client = CreateFhirClient(FhirBaseUrl);

        var patient = await CreateTestPatientAsync(client, ct);
        Console.WriteLine($"Selected Patient: {patient.Id}");

        // 準備六個指標數值（示範：你可改成實際推論結果）
        // SMI: cm2/m2, SMD: HU, 其餘: cm2
        var effectiveTime = DateTimeOffset.Now;

        var smi = 29.42m;
        var smd = 39.50m;
        var imat = 11.69m;
        var lama = 20.93m;
        var nama = 69.16m;
        var myosteatosis = 32.62m;

        // 建立 Observation + DiagnosticReport，並 Transaction 寫入
        var transaction = BuildTransactionBundle(
            patientId: patient.Id,
            effectiveTime: effectiveTime,
            imageUrl: AIResultImageUrl,
            smi: smi,
            smd: smd,
            imat: imat,
            lama: lama,
            nama: nama,
            myosteatosis: myosteatosis
        );

        var response = await client.TransactionAsync(transaction, ct);

        // 從回應 Bundle 取出 server 指派的資源 id（如果 server 有回傳）
        // 有些 server 會把 location 放在 entry.response.location
        Console.WriteLine("Transaction completed.");
        foreach (var entry in response.Entry)
        {
            var loc = entry.Response?.Location ?? "(no location)";
            Console.WriteLine($"- {entry.Resource?.TypeName ?? "(no resource)"} => {loc}");
        }

        await DeletePatientAndRelatedResourcesAsync(client, patient.Id, ct);
    }

    private static async System.Threading.Tasks.Task DeletePatientAndRelatedResourcesAsync(
        FhirClient client,
        string patientId,
        CancellationToken ct)
    {
        Console.WriteLine($"開始刪除 Patient ID: {patientId} 及其相關資源...");

        var deleteEntries = new List<Bundle.EntryComponent>();

        try
        {
            var obsBundle = await client.SearchAsync<Observation>(
                new[] { $"subject=Patient/{patientId}" },
                ct: ct);

            if (obsBundle?.Entry != null)
            {
                foreach (var entry in obsBundle.Entry)
                {
                    if (entry.Resource is Observation obs && !string.IsNullOrEmpty(obs.Id))
                    {
                        deleteEntries.Add(new Bundle.EntryComponent
                        {
                            Request = new Bundle.RequestComponent
                            {
                                Method = Bundle.HTTPVerb.DELETE,
                                Url = $"Observation/{obs.Id}"
                            }
                        });
                        Console.WriteLine($"- 標記刪除 Observation: {obs.Id}");
                    }
                }
            }

            var drBundle = await client.SearchAsync<DiagnosticReport>(
                new[] { $"subject=Patient/{patientId}" },
                ct: ct);

            if (drBundle?.Entry != null)
            {
                foreach (var entry in drBundle.Entry)
                {
                    if (entry.Resource is DiagnosticReport dr && !string.IsNullOrEmpty(dr.Id))
                    {
                        deleteEntries.Add(new Bundle.EntryComponent
                        {
                            Request = new Bundle.RequestComponent
                            {
                                Method = Bundle.HTTPVerb.DELETE,
                                Url = $"DiagnosticReport/{dr.Id}"
                            }
                        });
                        Console.WriteLine($"- 標記刪除 DiagnosticReport: {dr.Id}");
                    }
                }
            }

            deleteEntries.Add(new Bundle.EntryComponent
            {
                Request = new Bundle.RequestComponent
                {
                    Method = Bundle.HTTPVerb.DELETE,
                    Url = $"Patient/{patientId}"
                }
            });
            Console.WriteLine($"- 標記刪除 Patient: {patientId}");

            if (deleteEntries.Count > 0)
            {
                var deleteBundle = new Bundle
                {
                    Type = Bundle.BundleType.Transaction,
                    Entry = deleteEntries
                };

                var response = await client.TransactionAsync(deleteBundle, ct);
                Console.WriteLine($"成功刪除 {deleteEntries.Count} 筆資源");

                foreach (var entry in response.Entry)
                {
                    var status = entry.Response?.Status ?? "unknown";
                    var location = entry.Response?.Location ?? "unknown";
                    Console.WriteLine($"  - {location}: {status}");
                }
            }
            else
            {
                Console.WriteLine("未找到需要刪除的資源");
            }
        }
        catch (FhirOperationException ex)
        {
            Console.Error.WriteLine($"刪除失敗: {ex.Message}");
            Console.Error.WriteLine($"Status: {ex.Status}");
            throw;
        }
    }

    private static FhirClient CreateFhirClient(string baseUrl)
    {
        var settings = new FhirClientSettings
        {
            PreferredFormat = ResourceFormat.Json,
            VerifyFhirVersion = true,
            ReturnPreference = ReturnPreference.Representation
        };

        return new FhirClient(baseUrl, settings);
    }


    private static async Task<Patient> CreateTestPatientAsync(FhirClient client, CancellationToken ct)
    {
        try
        {
            var patient = new Patient
            {
                Id = patientId,
                Active = true,
                Name =
                {
                    new HumanName
                    {
                        Use = HumanName.NameUse.Official,
                        Family = "測試",
                        Given = new[] { "病患", "範例" }
                    }
                },
                Gender = AdministrativeGender.Male,
                BirthDate = "1980-01-01",
                Telecom =
                {
                    new ContactPoint
                    {
                        System = ContactPoint.ContactPointSystem.Phone,
                        Value = "0912-345-678",
                        Use = ContactPoint.ContactPointUse.Mobile
                    }
                }
            };

            var createdPatient = await client.UpdateAsync(patient);
            Console.WriteLine($"已建立病患: {createdPatient.Id}");
            return createdPatient;
        }
        catch (FhirOperationException ex)
        {
            Console.WriteLine($"無法建立/更新病患: {ex.Message}");
            throw;
        }

        throw new InvalidOperationException("無法建立或取得 Patient");
    }
    
    private static Bundle BuildTransactionBundle(
        string patientId,
        DateTimeOffset effectiveTime,
        string imageUrl,
        decimal smi,
        decimal smd,
        decimal imat,
        decimal lama,
        decimal nama,
        decimal myosteatosis)
    {
        var patientRef = new ResourceReference($"Patient/{patientId}");

        var obsFullUrl = $"urn:uuid:{Guid.NewGuid()}";
        var drFullUrl = $"urn:uuid:{Guid.NewGuid()}";

        var obs = BuildObservation(patientRef, effectiveTime, smi, smd, imat, lama, nama, myosteatosis);
        var dr = BuildDiagnosticReport(patientRef, effectiveTime, obsFullUrl, imageUrl);

        return new Bundle
        {
            Type = Bundle.BundleType.Transaction,
            Entry =
            {
                NewPostEntry(obsFullUrl, "Observation", obs),
                NewPostEntry(drFullUrl, "DiagnosticReport", dr)
            }
        };
    }

    private static Bundle.EntryComponent NewPostEntry(string fullUrl, string url, Resource resource) =>
        new Bundle.EntryComponent
        {
            FullUrl = fullUrl,
            Resource = resource,
            Request = new Bundle.RequestComponent
            {
                Method = Bundle.HTTPVerb.POST,
                Url = url
            }
        };

    private static Observation BuildObservation(
        ResourceReference patientRef,
        DateTimeOffset effectiveTime,
        decimal smi,
        decimal smd,
        decimal imat,
        decimal lama,
        decimal nama,
        decimal myosteatosis)
    {
        var obs = new Observation
        {
            Status = ObservationStatus.Final,
            Subject = patientRef,
            Effective = new FhirDateTime(effectiveTime),
            Category =
            {
                new CodeableConcept(CsObsCategory, "imaging", "Imaging", null)
            },
            Code = new CodeableConcept(CsBodyComp, "body-composition-summary", "Body composition AI summary", null),
        };

        // 6 個 component：你指定的做法
        obs.Component.Add(Component("SMI", "骨骼肌指標 (SMI)", Quantity(smi, "cm2/m2", Ucum, "cm2/m2")));
        obs.Component.Add(Component("SMD", "骨骼肌密度 (SMD)", new Quantity { Value = smd, Unit = "HU" }));
        obs.Component.Add(Component("IMAT", "肌間/肌內脂肪組織 (IMAT)", Quantity(imat, "cm2", Ucum, "cm2")));
        obs.Component.Add(Component("LAMA", "低密度肌肉區域 (LAMA)", Quantity(lama, "cm2", Ucum, "cm2")));
        obs.Component.Add(Component("NAMA", "正常密度肌肉區域 (NAMA)", Quantity(nama, "cm2", Ucum, "cm2")));
        obs.Component.Add(Component("MYOSTEATOSIS", "肌肉脂肪變性 (Myosteatosis)", Quantity(myosteatosis, "cm2", Ucum, "cm2")));

        return obs;
    }

    private static DiagnosticReport BuildDiagnosticReport(
        ResourceReference patientRef,
        DateTimeOffset effectiveTime,
        string observationFullUrl,
        string imageUrl)
    {
        var diagnosticReport = new DiagnosticReport
        {
            Status = DiagnosticReport.DiagnosticReportStatus.Final,
            Subject = patientRef,
            Effective = new FhirDateTime(effectiveTime),
            Category =
            {
                new CodeableConcept(CsV2_0074, "RAD", "Radiology", null)
            },
            Code = new CodeableConcept(CsBodyComp, "bodycomp-ai-report", "Body composition AI report", null),
            Result =
            {
                new ResourceReference(observationFullUrl)
            }
        };

        diagnosticReport.PresentedForm.Add(new Attachment
        {
            ContentType = "image/png",
            Title = "Segmentation overlay",
            Url = imageUrl
        });

        return diagnosticReport;
    }

    private static Observation.ComponentComponent Component(string code, string text, Quantity qty) =>
        new Observation.ComponentComponent
        {
            Code = new CodeableConcept(CsBodyComp, code, text, null),
            Value = qty
        };

    private static Quantity Quantity(decimal value, string unit, string system, string code) =>
        new Quantity
        {
            Value = value,
            Unit = unit,
            System = system,
            Code = code
        };
}
```

程式一開始定義了一些常數，例如 FHIR 伺服器的基底 URL，以及一些 CodeSystem URL 和單位系統 URL。[FhirBaseUrl] 是 FHIR 伺服器的基底 URL， http://10.1.1.113:8080/fhir 。另外，這裡也定義了一些常數字串，這裡有 CsBodyComp 是一個自訂的 CodeSystem URL，用來定義身體組成相關的 code；CsObsCategory 是觀察類別的 CodeSystem URL；CsV2_0074 是診斷報告類別的 CodeSystem URL；Ucum 是單位系統的 URL。patientId 是一個固定的 Patient ID，這裡設定為 "test-patient-001"。AIResultImageUrl 是 AI 推論結果圖片的 URL，這裡設定為 "http://localhost/result/2311"。

在系統進入點的 [Main] 方法中，首先建立了一個 CancellationTokenSource，設定了 30 秒的逾時時間，也就是接下來的所有程式碼若在30秒內未完成，將會取消操作，接著取得 CancellationToken。接著，使用 CreateFhirClient 方法建立了一個 FhirClient 實例，並傳入 FHIR 伺服器的基底 URL。這樣子就已經完成的相關準備工作。

現在要來做第一個動作，就是建立一個病患資源，這裡使用了 CreateTestPatientAsync 方法來建立一個病患，並且傳入 FhirClient 實例和 CancellationToken。

在這個 [CreateTestPatientAsync] 方法內，首先建立了一個 Patient 物件，並且設定了它的 Id、Active 狀態、Name、Gender、BirthDate 和 Telecom 等欄位(因為這是一個測試用的 Patient 紀錄，這裡的相關欄位值，都是隨機設定的)。不過，這裡的 Id 是固定的 "test-patient-001"，這樣子在每次執行程式時，都會使用同一個 Patient ID，當然，在程式快要結束執行之後，便可以把關於這個病患的相關紀錄都刪除掉。

接著，使用 FhirClient 的 UpdateAsync 方法來建立或更新這個病患資源。UpdateAsync 方法會根據傳入的 Patient 物件的 Id 來決定是要建立一個新的資源還是更新已存在的資源。如果該 Id 的資源不存在，則會建立一個新的資源；如果該 Id 的資源已經存在，則會更新該資源。這裡的做法是利用 UpdateAsync 的特性來確保每次執行程式時，都能夠得到同一個 Patient 資源。

回到 [Main] 方法內，當成功建立或取得病患資源之後，會印出該病患的 Id。接著，準備六個指標數值，這些數值是示範用的，當要把這些程式碼修改成為實際的 SMART App 需要用的時候，這裡將會採用實際推論結果。這裡的指標包括 SMI、SMD、IMAT、LAMA、NAMA 和 MYOSTEATOSIS，每個指標都有一個對應的數值。這六個指標的意義分別為：
* SMI: 骨骼肌指標 (Skeletal Muscle Index)，單位為 cm2/m2
* SMD: 骨骼肌密度 (Skeletal Muscle Density)，單位為 HU (Hounsfield Units)
* IMAT: 肌間/肌內脂肪組織 (Intermuscular Adipose Tissue)，單位為 cm2
* LAMA: 低密度肌肉區域 (Low Attenuation Muscle Area)，單位為 cm2
* NAMA: 正常密度肌肉區域 (Normal Attenuation Muscle Area)，單位為 cm2
* MYOSTEATOSIS: 肌肉脂肪變性 (Myosteatosis)，單位為 cm2

接著，使用 BuildTransactionBundle 方法來建立一個 Transaction Bundle，這個 Bundle 內會包含一個 Observation 資源和一個 DiagnosticReport 資源，並且這兩個資源都會 reference 到剛剛建立的 Patient 資源。Observation 內會包含這六個指標數值，而 DiagnosticReport 內會包含對 Observation 的 reference，以及圖片的 URL 參考。

在這個 [BuildTransactionBundle] 方法內，首先建立了一個 ResourceReference，指向 Patient/{patientId}，這樣子就可以在 Observation 和 DiagnosticReport 內 reference 到這個病患。

接著，對於 [obsFullUrl] & [drFullUrl] 分別將會用來在 Transaction Bundle 內作為 Observation 和 DiagnosticReport 的 fullUrl，這裡使用了 urn:uuid 的方式來產生一個唯一的 fullUrl，這樣子在 Transaction Bundle 內就可以互相 reference，而不需要事先知道 server 會分配什麼樣的 id。當這個交易成功結束之後，這兩個 Url 將會被 server 替換成實際的資源位置。

然後，使用 BuildObservation 方法來建立一個 Observation 資源，並且傳入 Patient 的 reference、有效時間、六個指標數值等參數。這個 Observation 內會設定 status、category、code、subject、effective 等欄位，並且在 component[] 內加入六個 component，每個 component 都會有一個 code 和對應的數值。

在 FHIR 的 Observation 資源中， [Category] 是用來表示這個觀察值的類別，例如實驗室、影像、生命徵象等，這裡設定為 "imaging"。 [Code] 是用來表示這個觀察值的具體內容，這裡設定為 "body-composition-summary"，表示這是一個身體組成的摘要觀察值。 [Subject] 是用來 reference 到這個觀察值所屬的病患，這裡 reference 到剛剛建立的 Patient。 [Effective] 是用來表示這個觀察值的有效時間，這裡設定為目前的時間。

另外，[Component] 是用來表示這個觀察值的組成部分，這裡加入了六個 component，每個 component 都有一個 code 和對應的數值。這些 component 的 code 都是使用自訂的 CodeSystem URL "https://example.org/fhir/CodeSystem/body-composition" 來定義，這樣子在 FHIR 伺服器內就可以辨識這些 component 的意義。

在 [BuildTransactionBundle] 方法內，接著使用 BuildDiagnosticReport 方法來建立一個 DiagnosticReport 資源，並且傳入 Patient 的 reference、有效時間、Observation 的 fullUrl、圖片的 URL 參考等參數。

這個 DiagnosticReport 內會設定 status、category、code、subject、effective 等欄位，並且在 result[] 內加入對 Observation 的 reference，透過這樣的設計，當取得了這個診斷報告後，便可以透過 result[] 內的 reference 來取得這個報告所包含的觀察值。

在 presentedForm[] 內加入圖片的 URL 參考。在 FHIR 的 DiagnosticReport 資源中， [Category] 是用來表示這個報告的類別，例如實驗室、影像、病理等，這裡設定為 "RAD" (Radiology)。 [Code] 是用來表示這個報告的具體內容，這裡設定為 "bodycomp-ai-report"，表示這是一個身體組成 AI 報告。 [Subject] 是用來 reference 到這個報告所屬的病患，這裡 reference 到剛剛建立的 Patient。 [Effective] 是用來表示這個報告的有效時間，這裡設定為目前的時間。 [Result] 是用來 reference 到這個報告的觀察值，這裡 reference 到剛剛建立的 Observation。 [PresentedForm] 是用來表示這個報告的呈現形式，這裡加入了一個 Attachment，這個 Attachment 的 contentType 是 "image/png"，title 是 "Segmentation overlay"，url 是圖片的 URL 參考。

最後，透過了 obs & dr 這兩個資源，建立了一個 Transaction Bundle，這個 Bundle 的 type 是 Transaction，並且在 entry[] 內加入了兩個 entry，分別是 Observation 和 DiagnosticReport 的 entry，每個 entry 都有一個 fullUrl 和對應的 resource，以及一個 request component，request component 的 method 是 POST，url 分別是 "Observation" 和 "DiagnosticReport"，表示這兩個資源都是要被建立的。

Bundle內的 type 設定為 Transaction，表示這是一個交易性的 Bundle，當這個 Bundle 被送到 FHIR 伺服器時，伺服器會將這兩個資源當成一個整體來處理。type的值可以是以下幾種：
* document：表示這是一個文件型的 Bundle，通常用於表示一個完整的醫療文件，例如一個病歷摘要、一個診斷報告等。這種 Bundle 通常會有一個 Composition 資源作為根資源，其他資源則是 Composition 的 component[] 內的 reference。
* message：表示這是一個訊息型的 Bundle，通常用於表示一個訊息事件，例如一個訂單、一個通知等。這種 Bundle 通常會有一個 MessageHeader 資源作為根資源，其他資源則是 MessageHeader 的 reference。
* transaction：表示這是一個交易型的 Bundle，當這個 Bundle 被送到 FHIR 伺服器時，伺服器會將這些資源當成一個整體來處理，要麼全部成功，要麼全部失敗。這種 Bundle 通常用於表示一個原子性的操作，例如建立一個病患和相關的觀察值、報告等。

當這個 [BuildTransactionBundle] 方法執行完成後，就會得到一個包含 Observation 和 DiagnosticReport 的 Transaction Bundle。

在 [Main] 方法內，使用 FhirClient 的 TransactionAsync 方法來送出這個 Bundle 到 FHIR 伺服器，這樣子就可以同時建立 Observation 和 DiagnosticReport 這兩個資源，並且它們之間的 reference 也會被正確處理。

最後，當這個交易完成之後，會從回應的 Bundle 內取出 server 指派的資源 id，並且印出來。這裡的做法是從 response.Entry 內的每個 entry 中，取出 entry.Response.Location，這個 Location 通常會包含 server 分配的資源位置，例如 "Observation/123/_history/1" 或 "DiagnosticReport/456/_history/1" 等等。如果 server 沒有回傳 Location，則會印出 "(no location)"。這樣子就可以知道這個交易所建立的資源在 server 上的實際位置，這對於後續的操作，例如更新或刪除這些資源，都會非常有幫助。

由於這是一個模擬測試的練習程式碼，為了下次能夠再度重複執行，在這裡將會呼叫 [DeletePatientAndRelatedResourcesAsync] 方法來刪除剛剛建立的 Patient 以及相關的 Observation 和 DiagnosticReport 資源，這樣子就可以確保每次執行程式時，都能夠從一個乾淨的狀態開始。

# 執行程式

首先先來看這個專案的執行結果：

* 按下 F5 鍵或點擊「開始」按鈕來執行程式
* 底下為實際操作過程的輸出文字

```
已建立病患: test-patient-001
Selected Patient: test-patient-001
Transaction completed.
- Observation => Observation/132801/_history/1
- DiagnosticReport => DiagnosticReport/132802/_history/1
開始刪除 Patient ID: test-patient-001 及其相關資源...
- 標記刪除 Observation: 132801
- 標記刪除 DiagnosticReport: 132802
- 標記刪除 Patient: test-patient-001
成功刪除 3 筆資源
  - Observation/132801/_history/2: 204 No Content
  - DiagnosticReport/132802/_history/2: 204 No Content
  - Patient/test-patient-001/_history/8: 204 No Content
```


# 查詢出來的 FHIR Bundle JSON

底下為當完成了 DiagnosticReport + Observation 的 Transaction 之後，從 FHIR 伺服器查詢出來的 Bundle JSON，這個 Bundle 是在執行 [DeletePatientAndRelatedResourcesAsync] 方法之前，透過 FhirClient 的 SearchAsync 方法，以 subject=Patient/test-patient-001 的條件來查詢 Observation 資源時所得到的結果。這個 Bundle 內包含了剛剛建立的 Patient 資源以及相關的 Observation 資源。

```json
{
  "resourceType": "Bundle",
  "id": "64c57cc5-d0cd-4795-9838-827b8c048e93",
  "meta": {
    "lastUpdated": "2026-02-11T10:04:46.518+08:00"
  },
  "type": "searchset",
  "total": 3,
  "link": [ {
    "relation": "self",
    "url": "http://10.1.1.113:8080/fhir/Patient/test-patient-001/$everything?_format=json"
  } ],
  "entry": [ {
    "fullUrl": "http://10.1.1.113:8080/fhir/Patient/test-patient-001",
    "resource": {
      "resourceType": "Patient",
      "id": "test-patient-001",
      "meta": {
        "versionId": "5",
        "lastUpdated": "2026-02-11T09:59:15.485+08:00"
      },
      "active": true,
      "name": [ {
        "use": "official",
        "family": "測試",
        "given": [ "病患", "範例" ]
      } ],
      "telecom": [ {
        "system": "phone",
        "value": "0912-345-678",
        "use": "mobile"
      } ],
      "gender": "male",
      "birthDate": "1980-01-01"
    },
    "search": {
      "mode": "match"
    }
  }, {
    "fullUrl": "http://10.1.1.113:8080/fhir/Observation/132756",
    "resource": {
      "resourceType": "Observation",
      "id": "132756",
      "meta": {
        "versionId": "1",
        "lastUpdated": "2026-02-11T09:59:15.663+08:00"
      },
      "status": "final",
      "category": [ {
        "coding": [ {
          "system": "http://terminology.hl7.org/CodeSystem/observation-category",
          "code": "imaging",
          "display": "Imaging"
        } ]
      } ],
      "code": {
        "coding": [ {
          "system": "https://example.org/fhir/CodeSystem/body-composition",
          "code": "body-composition-summary",
          "display": "Body composition AI summary"
        } ]
      },
      "subject": {
        "reference": "Patient/test-patient-001"
      },
      "effectiveDateTime": "2026-02-11T09:59:15.5716076+08:00",
      "component": [ {
        "code": {
          "coding": [ {
            "system": "https://example.org/fhir/CodeSystem/body-composition",
            "code": "SMI",
            "display": "骨骼肌指標 (SMI)"
          } ]
        },
        "valueQuantity": {
          "value": 29.42,
          "unit": "cm2/m2",
          "system": "http://unitsofmeasure.org",
          "code": "cm2/m2"
        }
      }, {
        "code": {
          "coding": [ {
            "system": "https://example.org/fhir/CodeSystem/body-composition",
            "code": "SMD",
            "display": "骨骼肌密度 (SMD)"
          } ]
        },
        "valueQuantity": {
          "value": 39.50,
          "unit": "HU"
        }
      }, {
        "code": {
          "coding": [ {
            "system": "https://example.org/fhir/CodeSystem/body-composition",
            "code": "IMAT",
            "display": "肌間/肌內脂肪組織 (IMAT)"
          } ]
        },
        "valueQuantity": {
          "value": 11.69,
          "unit": "cm2",
          "system": "http://unitsofmeasure.org",
          "code": "cm2"
        }
      }, {
        "code": {
          "coding": [ {
            "system": "https://example.org/fhir/CodeSystem/body-composition",
            "code": "LAMA",
            "display": "低密度肌肉區域 (LAMA)"
          } ]
        },
        "valueQuantity": {
          "value": 20.93,
          "unit": "cm2",
          "system": "http://unitsofmeasure.org",
          "code": "cm2"
        }
      }, {
        "code": {
          "coding": [ {
            "system": "https://example.org/fhir/CodeSystem/body-composition",
            "code": "NAMA",
            "display": "正常密度肌肉區域 (NAMA)"
          } ]
        },
        "valueQuantity": {
          "value": 69.16,
          "unit": "cm2",
          "system": "http://unitsofmeasure.org",
          "code": "cm2"
        }
      }, {
        "code": {
          "coding": [ {
            "system": "https://example.org/fhir/CodeSystem/body-composition",
            "code": "MYOSTEATOSIS",
            "display": "肌肉脂肪變性 (Myosteatosis)"
          } ]
        },
        "valueQuantity": {
          "value": 32.62,
          "unit": "cm2",
          "system": "http://unitsofmeasure.org",
          "code": "cm2"
        }
      } ]
    },
    "search": {
      "mode": "match"
    }
  }, {
    "fullUrl": "http://10.1.1.113:8080/fhir/DiagnosticReport/132757",
    "resource": {
      "resourceType": "DiagnosticReport",
      "id": "132757",
      "meta": {
        "versionId": "1",
        "lastUpdated": "2026-02-11T09:59:15.663+08:00"
      },
      "status": "final",
      "category": [ {
        "coding": [ {
          "system": "http://terminology.hl7.org/CodeSystem/v2-0074",
          "code": "RAD",
          "display": "Radiology"
        } ]
      } ],
      "code": {
        "coding": [ {
          "system": "https://example.org/fhir/CodeSystem/body-composition",
          "code": "bodycomp-ai-report",
          "display": "Body composition AI report"
        } ]
      },
      "subject": {
        "reference": "Patient/test-patient-001"
      },
      "effectiveDateTime": "2026-02-11T09:59:15.5716076+08:00",
      "result": [ {
        "reference": "Observation/132756"
      } ],
      "presentedForm": [ {
        "contentType": "image/png",
        "url": "http://localhost/result/2311",
        "title": "Segmentation overlay"
      } ]
    },
    "search": {
      "mode": "match"
    }
  } ]
}
```



