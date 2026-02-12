# FHIR 14 對於 FHIR Observation 的操作範例

在這篇文章內指向的 FHIR Server 內的資料，將會採用 [Synthea] 所產生的模擬醫療紀錄，這是透過 [Sample FHIR Bulk Export Datasets](https://github.com/smart-on-fhir/sample-bulk-fhir-datasets) 下載 Medium (100 patients, 17MB zipped, 129MB unzipped) 大小的資料集。

這在 FHIR Server 是在內部主機上，採用的同樣是 R4 的版本，並且在這個 FHIR Server 上，已經事先載入了這個 Synthea 產生的模擬醫療紀錄資料集。

我個人覺得 Observation 資源在 FHIR 中，相對是比較複雜的，因此，同樣的還是先來簡單了解這個資源用途

## Observation 的定位與常見用途

Observation 用來表達「被觀察到的事實」：包含檢驗、生命徵象、影像量測結果、問卷量測值、裝置量測值等，通常以「一個 code + 一個 value（或缺值原因）」為核心，再搭配時間、對象、方法、檢體、參考區間等語意。

## Observation 主體欄位

> 下列每個欄位都列出：意義、用法、注意事項。欄位清單與基礎定義來自 Observation 結構與範本。

* identifier（0..*）：意義：業務識別碼（例如 LIS 檢體報告號、量測單號）。用法：用於跨系統對帳、避免只靠 `id`（伺服器內部 id）做整合。* 注意：FHIR Server 內部的 `id` 是資源版本控制的識別碼，不適合用於業務層面的對帳；`identifier` 才是業務識別碼。

* basedOn（0..*）：意義：此 Observation 用來履行的「計畫/醫囑/請求」。用法：常指向 `ServiceRequest`（檢驗/檢查醫囑）、`MedicationRequest` 等。注意：同一檢驗結果可能同時對應多個請求（例如合併下單）。

* partOf（0..*）：意義：此 Observation 是某事件的一部分（例如某次處置、用藥給予、影像檢查流程）。用法：可指向 `Procedure`、`ImagingStudy`、`MedicationAdministration` 等。注意：用於表達「結果隸屬於某個已發生事件」。
* status（1..1）：意義：Observation 的狀態（registered / preliminary / final / amended…）。用法：臨床通常最重視 `final`（已確認）。注意：必填；也是搜尋/流程判斷的關鍵欄位。
* category（0..*）：意義：高階分類（例如 laboratory / vital-signs / imaging…）。用法：讓 UI 或查詢能快速分群（例如「先抓生命徵象」）。注意：可多值（同一筆 Observation 可同時落在不同分類視角）。
* code（1..1）：意義：觀察項目是什麼（觀察的名稱/代碼），例如「血糖」、「體溫」。用法：通常用標準術語（常見是 LOINC），也可加多個 coding 表示更細緻或不同視角的同義碼。注意：`code` 定義了 Observation 的語意核心；很多時候「測哪裡」其實已經隱含在 code（例如 Blood Glucose）。
* subject（0..1）：意義：Observation 關於誰/什麼（常見：病人）。用法：通常是 `Patient`，也可能是 `Group`、`Device`、`Location`。注意：有些量測不是「病人層級」而是設備或地點層級。
* focus（0..*）：意義：當「觀察焦點」不是 `subject` 本人時，用 `focus` 指出真正被關注的對象。用法：例如 subject=Patient，但 focus=植入裝置、某個檢體、或另一個 Observation。注意：與 `specimen`、`bodySite` 一起用來精準表達觀察焦點差異。
* encounter（0..1）：意義：量測發生所屬的就醫事件（門診/急診/住院的一次 Encounter）。用法：做時序圖、就醫分段、或住院期間趨勢很重要。注意：不是所有 Observation 都一定能對應到 Encounter（例如居家裝置回傳）。
* effective[x]（0..1）：意義：臨床上最相關的時間/期間（physiologically relevant time）。型別：`dateTime` / `Period` / `Timing` / `instant`。用法：抽血：通常是採檢時間（或採檢期間）。24 小時尿液：用 `Period` 表示收集起訖。注意：這個時間通常比 `issued` 更能代表「這個數值對病人的意義」。
* issued（0..1）：意義：此版本 Observation 被發布/可取得 的時間點。用法：例如 LIS 驗證後釋出結果時間。注意：與 `effective[x]` 不同：effective 是「量測對病人的生理/臨床時間」，issued 是「系統釋出時間」。
* performer（0..*）：意義：對此 Observation 負責/執行的人或組織。用法：可指向 Practitioner / Organization / CareTeam 等。注意：檢驗可能是科室或實驗室（Organization），生命徵象可能是護理人員（PractitionerRole）。

## 觀察的上下文：部位、方法、檢體、設備

* bodySite（0..1）：意義：觀察的身體部位。用法：例如左右手、特定肌群、特定器官部位。注意：當部位並未由 `code` 足夠描述時才特別需要。
* method（0..1）：意義：量測方法。用法：例如「pulse oximetry」、「酵素法」等。注意：方法不同可能導致可比性差，若要做趨勢/分析很重要。
* specimen（0..1）：意義：檢體。用法：血液、尿液、組織等，通常指向 Specimen 資源。注意：與 `effective[x]` 常一起解釋（採檢時間 vs 報告釋出時間）。
* device（0..1）：意義：產生量測資料的裝置。用法：床邊監視器、居家量測設備等。注意：跨設備趨勢比較、校正與可信度判斷會用到。  

## 觀察的群組與衍生關係：`hasMember` / `derivedFrom` / `component`

* hasMember（0..*）：意義：把多個 Observation（或 QuestionnaireResponse、MolecularSequence）聚成一組的關聯。用法：適合「一組內的成員可獨立理解/可獨立使用」的情境（相對於 component）。注意：規格明確把 `hasMember` 視為 Observation grouping 的一種手段。
* derivedFrom（0..*）：意義：此 Observation 由哪些資料/測量衍生而來（DocumentReference、ImagingStudy、Media、QuestionnaireResponse、Observation…）。用法：例如 AI 從影像推論出的身體組成結果，可用 derivedFrom 指回影像或原始量測。注意：也是 Observation grouping/關聯的重要元素之一。
* component（0..*）：意義：同一個 Observation 內的「組成結果」（血壓：收縮壓/舒張壓；基因檢測：多個 component）。用法：component 適用於「離開母 Observation 就很難被合理解讀/使用」的子結果。注意：如果 `Observation.code` 與某個 `Observation.component.code` 相同，則 母 Observation 對應該 code 的 `value[x]` 不得出現（避免重複/衝突）。

現在對於 Observation 資源有了初步認識，接下來，我們就來看看如何使用 C# 程式碼，透過 FHIR Client 來查詢 Observation 資源，並且將查詢到的結果顯示在主控台視窗內。

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

### 安裝 Hl7.Fhir.R4 套件

這個套件是用來處理 FHIR R4 版本的資源物件，並且提供了與 FHIR Server 進行互動的功能。這裡是該套件提供功能的主要清單：
* FHIR 資源物件模型：提供了對 FHIR R4 版本的資源物件模型的定義，讓開發者可以使用這些物件來表示和操作 FHIR 資源。
* FHIR 客戶端功能：提供了與 FHIR Server 進行互動的功能，包括發出請求、接收回應、處理錯誤等。

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

namespace csObservation;

internal class Program
{
    static void Main(string[] args)
    {
        var fhirBaseUrl = "http://10.1.1.113:8080/fhir"; 
        var patientId = "01707a0c-9619-ccba-695a-b270744d76c2"; 
        var startDate = new DateTimeOffset(2014, 01, 01, 0, 0, 0, TimeSpan.FromHours(8));
        var endDate = new DateTimeOffset(2024, 12, 31, 23, 59, 59, TimeSpan.FromHours(8));

        var clientSettings = new FhirClientSettings
        {
            PreferredFormat = ResourceFormat.Json,
            Timeout = 60_000,
            VerifyFhirVersion = false 
        };

        FhirClient client;
        try
        {
            // 確保 Base Url 合法且有結尾斜線
            var baseUri = new Uri(fhirBaseUrl.TrimEnd('/') + "/");
            client = new FhirClient(baseUri, clientSettings);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"建立 FhirClient 失敗，請檢查 fhirBaseUrl 設定：{ex.Message}");
            return;
        }

        var searchParams = new SearchParams()
            .Where($"subject=Patient/{patientId}")
            .Where($"date=ge{startDate:yyyy-MM-dd}")
            .Where($"date=le{endDate:yyyy-MM-dd}")
            .OrderBy("-date")      // 「-欄位」代表 desc
            .LimitTo(100);  

        // 你可以視需求 include 參考資源（伺服器要支援）
        // Observation 常見 include:
        // - subject (Patient)  encounter  performer  specimen  device
        // 注意：include 太多可能很慢，真正在產品通常會分開查或做快取
        // 新版 API 的 Include 集合需要指定 IncludeModifier，這裡使用 None
        searchParams.Include.Add(("Observation:subject", IncludeModifier.None));
        searchParams.Include.Add(("Observation:encounter", IncludeModifier.None));
        searchParams.Include.Add(("Observation:performer", IncludeModifier.None));
        searchParams.Include.Add(("Observation:specimen", IncludeModifier.None));
        searchParams.Include.Add(("Observation:device", IncludeModifier.None));

        Bundle bundle = client.Search<Observation>(searchParams);

        Console.WriteLine($"FHIR Base: {fhirBaseUrl}");
        Console.WriteLine($"Patient: Patient/{patientId}");
        Console.WriteLine($"Range: {startDate:yyyy-MM-dd} ~ {endDate:yyyy-MM-dd}");
        Console.WriteLine(new string('-', 80));

        int row = 0;
        foreach (var entry in bundle.Entry.Where(e => e.Resource is Observation))
        {
            var obs = (Observation)entry.Resource;
            row++;

            Console.WriteLine($"[{row}] Observation/{obs.Id}");
            Console.WriteLine($"Status: {obs.Status}  |  Category: {FormatCodeableConcepts(obs.Category)}");
            Console.WriteLine($"Code: {FormatCodeableConcept(obs.Code)}");
            Console.WriteLine($"Effective: {FormatEffective(obs.Effective)}");

            // 值與單位
            Console.WriteLine($"Value: {FormatValue(obs.Value)}");
            if (obs.ReferenceRange?.Any() == true)
            {
                Console.WriteLine($"ReferenceRange: {FormatReferenceRanges(obs.ReferenceRange)}");
            }

            // 這筆 Observation 參考到哪些 Resource（如果有 include 的話，這些參考資源會被一起抓回來）
            Console.WriteLine($"Subject: {FormatRef(obs.Subject)}"); // 幾乎一定是 Patient（或 Group/Device）
            Console.WriteLine($"Encounter: {FormatRef(obs.Encounter)}"); // 常用：連回就醫事件
            Console.WriteLine($"Performer: {FormatRefs(obs.Performer)}"); // 可能是 Practitioner / PractitionerRole / Organization / CareTeam / Patient / Device
            Console.WriteLine($"Specimen: {FormatRef(obs.Specimen)}"); // 檢體 (laboratory 常見)
            Console.WriteLine($"Device: {FormatRef(obs.Device)}"); // 量測儀器或系統
            Console.WriteLine($"BasedOn: {FormatRefs(obs.BasedOn)}"); // ServiceRequest / MedicationRequest / CarePlan ...
            Console.WriteLine($"PartOf: {FormatRefs(obs.PartOf)}"); // Procedure / MedicationAdministration / ImagingStudy ...
            Console.WriteLine($"HasMember: {FormatRefs(obs.HasMember)}"); // Panel/Group
            Console.WriteLine($"DerivedFrom: {FormatRefs(obs.DerivedFrom)}"); // 由其他 Observation/DocumentReference/ImagingStudy 派生

            // Coding system / 資料分類推論
            Console.WriteLine($"CodingSystems(code): {ListCodingSystems(obs.Code)}");
            var categorySystemsStr = obs.Category == null
                ? string.Empty
                : string.Join(", ", obs.Category.Select(ListCodingSystems));
            Console.WriteLine($"CodingSystems(category): {categorySystemsStr}");
            Console.WriteLine($"LikelyDomainHint: {InferDomainHint(obs)}");

            Console.WriteLine(new string('-', 80));
        }

        // 若有下一頁
        while (bundle.NextLink != null)
        {
            bundle = client.Continue(bundle, PageDirection.Next);
            foreach (var entry in bundle.Entry.Where(e => e.Resource is Observation))
            {
                // 在這裡簡化僅查看最前面 100 筆
            }
        }
    }

    static string FormatEffective(DataType? effective)
    {
        return effective switch
        {
            FhirDateTime dt => dt.Value,
            Period p => $"{p.Start} ~ {p.End}",
            Timing t => "Timing(...)",
            Instant i => i.Value?.ToString("O") ?? "",
            _ => effective?.TypeName ?? ""
        };
    }

    static string FormatValue(DataType? value)
    {
        if (value == null) return "(no value)";
        return value switch
        {
            Quantity q => $"{q.Value} {q.Unit} (system={q.System}, code={q.Code})",
            CodeableConcept cc => FormatCodeableConcept(cc),
            FhirString s => s.Value,
            FhirBoolean b => b.Value?.ToString() ?? "",
            Integer i => i.Value?.ToString() ?? "",
            Hl7.Fhir.Model.Range r => $"{FormatValue(r.Low)} ~ {FormatValue(r.High)}",
            Ratio ratio => $"{FormatValue(ratio.Numerator)} / {FormatValue(ratio.Denominator)}",
            SampledData sd => $"SampledData(points={sd.Data?.Split(' ').Length ?? 0}, unit={sd.Origin?.Unit})",
            Attachment a => $"Attachment(contentType={a.ContentType}, url={a.Url})",
            _ => $"{value.TypeName}"
        };
    }

    static string FormatReferenceRanges(List<Observation.ReferenceRangeComponent> ranges)
    {
        return string.Join(" | ", ranges.Select(r =>
        {
            var low = r.Low != null ? $"{r.Low.Value} {r.Low.Unit}" : "";
            var high = r.High != null ? $"{r.High.Value} {r.High.Unit}" : "";
            var text = r.Text ?? "";
            var type = r.Type != null ? FormatCodeableConcept(r.Type) : "";
            return $"[{low}~{high}] {text} {type}".Trim();
        }));
    }

    static string FormatCodeableConcept(CodeableConcept? cc)
    {
        if (cc == null) return "(none)";
        var text = !string.IsNullOrWhiteSpace(cc.Text) ? cc.Text : "";
        var codings = cc.Coding?.Select(c => $"{c.System}|{c.Code}{(string.IsNullOrWhiteSpace(c.Display) ? "" : $" ({c.Display})")}") ?? Enumerable.Empty<string>();
        return $"{text}  =>  {string.Join(", ", codings)}".Trim();
    }

    static string FormatCodeableConcepts(IEnumerable<CodeableConcept>? ccs)
        => ccs == null ? "(none)" : string.Join(" ; ", ccs.Select(FormatCodeableConcept));

    static string FormatRef(ResourceReference? r)
        => r == null ? "(none)" : $"{r.Reference}{(string.IsNullOrWhiteSpace(r.Display) ? "" : $" ({r.Display})")}";

    static string FormatRefs(IEnumerable<ResourceReference>? rs)
        => rs == null ? "(none)" : string.Join(", ", rs.Select(FormatRef));

    static string ListCodingSystems(CodeableConcept? cc)
    {
        if (cc?.Coding == null || cc.Coding.Count == 0) return "(none)";
        return string.Join(", ", cc.Coding.Where(c => !string.IsNullOrWhiteSpace(c.System)).Select(c => c.System).Distinct());
    }

    static string InferDomainHint(Observation obs)
    {
        // 1) category 常用：vital-signs/laboratory/imaging/social-history
        var categorySystems = obs.Category?
            .SelectMany(c => c.Coding ?? new List<Coding>())
            .Where(c => c.System == "http://terminology.hl7.org/CodeSystem/observation-category")
            .Select(c => c.Code)
            .Where(c => !string.IsNullOrWhiteSpace(c))
            .Distinct()
            .ToList() ?? new List<string>();

        if (categorySystems.Contains("vital-signs"))
            return "Likely Vital Signs (often recorded by nursing devices/staff); units usually UCUM.";
        if (categorySystems.Contains("laboratory"))
            return "Likely Laboratory result (often from LIS); specimen commonly present; code usually LOINC; units often UCUM.";
        if (categorySystems.Contains("imaging"))
            return "Likely Imaging-related observation (may be derivedFrom ImagingStudy/DiagnosticReport).";
        if (categorySystems.Contains("social-history"))
            return "Likely Social History (e.g., smoking, alcohol).";

        // 2) specimen 有時可視為檢驗/病理類訊號
        if (obs.Specimen != null)
            return "Specimen present -> often lab/pathology workflow.";

        // 3) performer 是 Organization / PractitionerRole 時，常用來推測科室/檢驗單位
        if (obs.Performer?.Any(p => p.Reference != null && p.Reference.StartsWith("Organization/")) == true)
            return "Performer includes Organization -> may indicate department/lab organization.";

        return "No strong hint; use category + code system + performer/specimen to classify.";
    }
}
```

這個程式一開始將會宣告四個變數，分別是 `fhirBaseUrl`、`patientId`、`startDate` 以及 `endDate`，這些變數分別用來設定 FHIR Server 的 URL、病人 ID，以及查詢的日期範圍。因此，這個範例程式碼，將會針對特定的病患與在特定的日期範圍內，去查詢 Observation 資源。

使用 [FhirClientSettings] 物件來設定 FHIR Client 的一些行為，例如：PreferredFormat、Timeout、VerifyFhirVersion 等等，這些設定會影響到後續與 FHIR Server 互動的方式。其中，PreferredFormat 設定為 ResourceFormat.Json，表示希望以 JSON 格式來與 FHIR Server 進行資料交換；Timeout 設定為 60 秒，表示如果與 FHIR Server 的互動超過這個時間，就會觸發逾時錯誤；VerifyFhirVersion 設定為 false，表示不強制檢查 FHIR Server 的版本相容性。

接下來建立一個 [FhirClient] 實例，這個實例物件將會作為要與 FHIR 發送請求的需求。

[SearchParams] 物件用來設定要查詢 Observation 資源的條件，這裡設定了以下條件：
- subject=Patient/{patientId}：表示要查詢關於特定病人的 Observation 資源
- date=ge{startDate:yyyy-MM-dd}：表示要查詢在 startDate 之後（包含 startDate）的 Observation 資源
- date=le{endDate:yyyy-MM-dd}：表示要查詢在 endDate 之前（包含 endDate）的 Observation 資源
- OrderBy("-date")：表示要按照日期降冪排序（最新的在前面）
- LimitTo(100)：表示最多只查詢 100 筆符合條件的 Observation 資源

這個 [SearchParams] 物件還有其他的用法，這包括了可以使用 [Include] 來指定要包含哪些相關資源，例如：Observation:subject、Observation:encounter、Observation:performer、Observation:specimen、Observation:device 等等，這些參數會讓 FHIR Server 在回應時，一次性將這些相關資源也包含在回應的 Bundle 中，這樣就不需要再額外發出多次請求來取得這些相關資源的資料。還有一些其他的參數，例如：_elements 可以用來指定要回應哪些欄位，_summary 可以用來指定要回應資源的哪個摘要層級（例如：true、text、data、count），這些參數可以幫助我們優化查詢的效率與回應的大小。

對於 [searchParams.Include.Add] 方法，其功能與目的在於指定在查詢 Observation 資源時，要同時包含哪些相關資源。這裡的參數是以 "Observation:subject"、"Observation:encounter"、"Observation:performer"、"Observation:specimen"、"Observation:device" 的格式來指定要包含的相關資源，這些參數會告訴 FHIR Server 在回應時，一次性將這些相關資源也包含在回應的 Bundle 中，這樣就不需要再額外發出多次請求來取得這些相關資源的資料。而第二個參數為 IncludeModifier.None，表示不使用任何特殊的 include 修飾符，這是 FHIR API 中的一種用法，表示要包含的資源是直接相關的，而不是透過某些特殊的關聯方式。

一旦都設定完成，這裡使用了 [client.Search<Observation>(searchParams)] 方法來發出查詢請求，這個方法會根據我們設定的條件來查詢 Observation 資源，並且回傳一個包含符合條件的 Observation 資源的 Bundle。接下來程式碼中會對這個 Bundle 進行處理，將其中的 Observation 資源以及相關資源的資料顯示在主控台視窗內。這個方法呼叫，將會得到一個 `Bundle bundle` 物件，這個物件包含了查詢結果的 Observation 資源以及相關資源的資料

接下來程式碼中會對這個 Bundle 進行處理，在此使用了 `bundle.Entry.Where(e => e.Resource is Observation)`敘述，這個敘述會從 Bundle 的 Entry 中過濾出 Resource 是 Observation 的項目，然後對每一個符合條件的 Entry 進行處理。

對於 `var obs = (Observation)entry.Resource`這個敘述，則是將 Entry 中的 Resource 強制轉型為 Observation 類型，這樣就可以使用 Observation 類別中定義的屬性和方法來存取 Observation 的資料。

透過 obs 物件，可以先取得與顯示出這個 Observation 的 [Category] & [Code] & [Effective]，而 [ReferenceRange] 表示了參考值範圍

接著，會把有參考到這個觀察物件的其他資源顯示出來，這包括了 Subject、Encounter、Performer、Specimen、Device、BasedOn、PartOf、HasMember 以及 DerivedFrom 等等，這些都是 Observation 資源中常見的參考欄位，這些欄位通常會指向其他的資源，例如：Patient、Encounter、Practitioner、Organization、ServiceRequest 等等。這些參考資源的資料，會在前面使用 [Include] 參數指定要包含在查詢結果的 Bundle 中。

在這裡也設計了許多支援方法，用來進一步的將這些觀察物件的詳細資訊，使用格式化的方式來顯示在螢幕上，這些結果可以從底下的執行結果輸出內容，看到各種各種不同屬性的格式化表現方式，底下將會把這些支援方法所做的事情整理出來。
- FormatEffective：用來格式化 Observation 的 Effective 欄位，這個欄位的型別是 DataType，可以是 FhirDateTime、Period、Timing、Instant 等等，這個方法會根據不同的型別來格式化成適合顯示的字串。
- FormatValue：用來格式化 Observation 的 Value 欄位，這個欄位的型別也是 DataType，可以是 Quantity、CodeableConcept、FhirString、FhirBoolean、Integer、Range、Ratio、SampledData、Attachment 等等，這個方法會根據不同的型別來格式化成適合顯示的字串。
- FormatReferenceRanges：用來格式化 Observation 的 ReferenceRange 欄位，這個欄位是一個 List<Observation.ReferenceRangeComponent>，這個方法會將每一個 ReferenceRangeComponent 的 Low、High、Text、Type 等等資訊格式化成適合顯示的字串。
- FormatCodeableConcept：用來格式化 CodeableConcept 類型的欄位，這個方法會將 CodeableConcept 的 Text 以及 Coding 中的 System、Code、Display 等等資訊格式化成適合顯示的字串。
- FormatCodeableConcepts：用來格式化多個 CodeableConcept 的欄位，這個方法會將每一個 CodeableConcept 都使用 FormatCodeableConcept 方法來格式化，然後再將它們串接成一個字串。
- FormatRef：用來格式化 ResourceReference 類型的欄位，這個方法會將 ResourceReference 的 Reference 以及 Display 等等資訊格式化成適合顯示的字串。
- FormatRefs：用來格式化多個 ResourceReference 的欄位，這個方法會將每一個 ResourceReference 都使用 FormatRef 方法來格式化，然後再將它們串接成一個字串。
- ListCodingSystems：用來列出 CodeableConcept 中的 Coding 所使用的系統，這個方法會從 CodeableConcept 的 Coding 中過濾出 System 不為空的 Coding，然後將它們的 System 串接成一個字串。
- InferDomainHint：用來根據 Observation 的屬性來推測這個 Observation 可能屬於哪個臨床領域，這個方法會根據 Category、Specimen、Performer 等等資訊來推測這個 Observation 可能是 Vital Signs、Laboratory、Imaging、Social History 等等。

## 執行程式碼

* 按下 `F5` 鍵，開始執行這個程式
* 程式將會開始執行，並且在主控台視窗內，將會看到類似下圖的輸出結果

```
FHIR Base: http://10.1.1.113:8080/fhir
Patient: Patient/01707a0c-9619-ccba-695a-b270744d76c2
Range: 2014-01-01 ~ 2024-12-31
--------------------------------------------------------------------------------
[1] Observation/40038ea0-606c-4fad-5901-7f93e51163b4
Status: Final  |  Category: =>  http://terminology.hl7.org/CodeSystem/observation-category|survey (Survey)
Code: Patient Health Questionnaire 2 item (PHQ-2) total score [Reported]  =>  http://loinc.org|55758-7 (Patient Health Questionnaire 2 item (PHQ-2) total score [Reported])
Effective: 2022-06-07T11:31:43-04:00
Value: 1 {score} (system=http://unitsofmeasure.org, code={score})
Subject: Patient/01707a0c-9619-ccba-695a-b270744d76c2
Encounter: Encounter/fcb63ed7-d6b3-b3cf-9671-ce99c8a5e033
Performer:
Specimen: (none)
Device: (none)
BasedOn:
PartOf:
HasMember:
DerivedFrom:
CodingSystems(code): http://loinc.org
CodingSystems(category): http://terminology.hl7.org/CodeSystem/observation-category
LikelyDomainHint: No strong hint; use category + code system + performer/specimen to classify.
--------------------------------------------------------------------------------
[2] Observation/bfd6a17b-5e25-8cab-91e9-600016493e2b
Status: Final  |  Category: =>  http://terminology.hl7.org/CodeSystem/observation-category|survey (Survey)
Code: Total score [HARK]  =>  http://loinc.org|76504-0 (Total score [HARK])
Effective: 2022-06-07T10:52:57-04:00
Value: 0 {score} (system=http://unitsofmeasure.org, code={score})
Subject: Patient/01707a0c-9619-ccba-695a-b270744d76c2
Encounter: Encounter/fcb63ed7-d6b3-b3cf-9671-ce99c8a5e033
Performer:
Specimen: (none)
Device: (none)
BasedOn:
PartOf:
HasMember:
DerivedFrom:
CodingSystems(code): http://loinc.org
CodingSystems(category): http://terminology.hl7.org/CodeSystem/observation-category
LikelyDomainHint: No strong hint; use category + code system + performer/specimen to classify.
--------------------------------------------------------------------------------

... (以下省略部分輸出結果) ...

--------------------------------------------------------------------------------
[99] Observation/137d9d6b-b96d-57fa-c668-fd1edf2e1214
Status: Final  |  Category: =>  http://terminology.hl7.org/CodeSystem/observation-category|laboratory (Laboratory)
Code: Lymphocytes [#/volume] in Blood by Automated count  =>  http://loinc.org|731-0 (Lymphocytes [#/volume] in Blood by Automated count)
Effective: 2020-12-02T09:24:10-05:00
Value: 1.1834 10*3/uL (system=http://unitsofmeasure.org, code=10*3/uL)
Subject: Patient/01707a0c-9619-ccba-695a-b270744d76c2
Encounter: Encounter/1ae31462-f120-b60d-6067-d8b43657986c
Performer:
Specimen: (none)
Device: (none)
BasedOn:
PartOf:
HasMember:
DerivedFrom:
CodingSystems(code): http://loinc.org
CodingSystems(category): http://terminology.hl7.org/CodeSystem/observation-category
LikelyDomainHint: Likely Laboratory result (often from LIS); specimen commonly present; code usually LOINC; units often UCUM.
--------------------------------------------------------------------------------
[100] Observation/358abffd-1da0-793c-0809-66cc078aae12
Status: Final  |  Category: =>  http://terminology.hl7.org/CodeSystem/observation-category|laboratory (Laboratory)
Code: Urea nitrogen [Mass/volume] in Serum or Plasma  =>  http://loinc.org|3094-0 (Urea nitrogen [Mass/volume] in Serum or Plasma)
Effective: 2020-12-02T09:24:10-05:00
Value: 17.86 mg/dL (system=http://unitsofmeasure.org, code=mg/dL)
Subject: Patient/01707a0c-9619-ccba-695a-b270744d76c2
Encounter: Encounter/1ae31462-f120-b60d-6067-d8b43657986c
Performer:
Specimen: (none)
Device: (none)
BasedOn:
PartOf:
HasMember:
DerivedFrom:
CodingSystems(code): http://loinc.org
CodingSystems(category): http://terminology.hl7.org/CodeSystem/observation-category
LikelyDomainHint: Likely Laboratory result (often from LIS); specimen commonly present; code usually LOINC; units often UCUM.
-------------------------------------------------------------------------------
```
