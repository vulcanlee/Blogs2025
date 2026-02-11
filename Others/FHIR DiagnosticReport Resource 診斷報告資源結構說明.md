# FHIR R4 DiagnosticReport：深入解析資源、欄位與 Patient 關聯

在 HL7 FHIR 中，**DiagnosticReport（診斷報告資源）** 是用來表示由診斷服務提供者出具之報告，包括實驗室結果、影像解讀、病理報告等資訊。與單一的 Observation（觀察結果）相比，它能彙整更多臨床語意與文檔層級的結果，是診斷流程中重要的事件資源。

## 📌 DiagnosticReport 的核心定位與價值

DiagnosticReport 是 FHIR Workflow 中的一種事件型資源，其主要用途為：

* 彙整和表達完整的診斷服務成果，涵蓋文字敘述、編碼結果、影像或可附加完整報告（如 PDF）。
* 結合診斷程序上下游關係，例如檢驗請求、觀察結果、醫療就診事件等。
* 與 Observation 形成一對多的關係：報告可能引用多個 Observation 來表示原子值（例如血液檢查各項指標）。
* 能支援多樣診斷類別：實驗室、影像、病理、心臟檢查等。

從臨床流程來看，DiagnosticReport 是“檢查完成後的產品”，可連結到請求來源（例如 ServiceRequest）、檢體（Specimen）、受檢者（Patient）、醫療事件（Encounter）、解讀者（Practitioner）等多種資源。

---

## 🧱 DiagnosticReport 的欄位解構：名稱、意義與實際用法

以下依照官方定義逐一解析 DiagnosticReport 項目（含基本資源層級與臨床需求）：

### **identifier**

報告的商業識別碼，可為多個（例如各系統內部 ID、實驗室編號等）。此欄位利於跨系統辨識同一報告。

### **basedOn**

表示這份 DiagnosticReport 是基於何種請求資源所產生，如 CarePlan、MedicationRequest、NutritionOrder 或 ServiceRequest。這是一種 workflow 追蹤，用於回溯診斷產出的起源。

### **status**

報告的狀態，例如 registered、partial、preliminary、final 等，影響臨床使用時機（是否為最終結果）。

### **category**

代表報告所屬的服務類型，例如實驗室（lab）、影像（imaging）等，用以檢索與分類。

### **code**

為必填欄位，用來標示這份報告的具體診斷性質（如一般實驗室報告、特定掃描診斷等），典型使用 LOINC 等編碼系統。

### **subject**

****核心欄位**，這是報告的主體（subject），通常會 reference Patient。
FHIR 的設計允許 subject 為 Patient、Group、Device 或 Location，但在大部分臨床上下文中的常見用法是指向 Patient（受檢者）。
這個 reference 是 DiagnosticReport 與 Patient 之間最直接的連結點。

### **encounter**

連結到 Encounter 資源，用於表明這份報告關聯到何種就診事件（例如門診、住院事件等），有助於維持臨床事件的時間與上下文關係。

### **effective[x]**

表示報告結果與檢查執行的時間相關資訊，可以是單一時間（effectiveDateTime）或時間區間（effectivePeriod），用於時間序列與診療路徑分析。

### **issued**

報告何時正式發布或可供查閱。這在流程控制與資料版本管理中非常重要。

### **performer**

這是執行該報告之診斷服務的實體參考，可能是 Practitioner、PractitionerRole、Organization 或 CareTeam。可用於責任區分與品質評估。

### **resultsInterpreter**

引用那些負責最終結果解讀之專業角色（例如主治醫師或解讀者 team）。

### **specimen**

如果診斷基於檢體（例如血液、組織切片），會 Reference Specimen 資源。這能讓前後端系統將報告結果與原始檢體檔案串連。

### **result**

DiagnosticReport 與 Observation 資源最重要的關聯之一：這表示此報告對“各別觀察值”的引用（例如血中紅血球濃度這種原子值），通常是 0..* 多對多。

### **imagingStudy**

在影像報告中，直接引用 ImagingStudy，使接收端能查詢完整的影像內容（例如 DICOM Study）以供瀏覽與視覺化。

### **media**

若診斷報告涉及附加的影像（例如拍攝圖片、病理切片圖像），透過 media 子元素設定圖像相關的 meta 與連結。

### **conclusion / conclusionCode**

以文字或可編碼方式提供測試的臨床結論或總結，用於提高理解或自動化決策支援（如結論標籤）。

### **presentedForm**

提供可下載的完整報告形式（例如 PDF、Word、影像檔等），適合直接呈現在使用者介面或儲存作為法律/醫療文件。

---

## 🔗 Resource 之間的 Reference 與 Patient 的關連鏈

在 DiagnosticReport 中，與 Patient 相關或可能有跨資源 Reference 的欄位包括：

* **subject** → Patient（最直接）
* **encounter** → Encounter（此欄位中 Encounter 通常也含有對 Patient 的 reference）
* **specimen** → Specimen（Specimen 通常會 reference Patient/Encounter）
* **result** → Observation（Observation 通常 reference Patient）
* **basedOn** → 可能 reference ServiceRequest 或其他 request，這些 request 也會 reference Patient
* **performer / resultsInterpreter** → 可能 reference Practitioner / Organization，但這些角色通常也有關聯 Patient 通常不直接 reference
* **imagingStudy** → ImagingStudy 內部也 reference Patient 以表明影像結果是針對誰執行的

因此，從 DiagnosticReport → subject → Patient → Observation / Encounter / Specimen / ImagingStudy，可以建立一條臨床事件詳細鏈結的資料關係圖。

---

## 🧠 Profiles 與 Extensions 的角色

除了基本資源外，FHIR 的 **Profiles** 與 **Extensions** 扮演著重要角色：

* **Profiles** 是針對某一特定需求或領域的約束/延伸版本，可限制欄位 cardinality、value set、必填性等，讓資料交換更嚴謹且符合領域規範。例如許多 Implementation Guide 提供了 “Laboratory Report Profile” 或 “DiagnosticReport Patient Profile”。([HL7 Europe][4])
* **Extensions** 是在標準欄位不夠表達時的擴充機制，用於補充特定場景資訊。

Profiles 是否強制使用、或有哪些特定 extension 需要加入，通常依照業務需求或 IG（Implementation Guide）規範決定。

---

## 📍 與 Patient 的 Reference Network（經由 DiagnosticReport）

在實際 FHIR 生態中，DiagnosticReport 可能與 Patient 產生以下重要關聯：

* **直接的 subject Reference**：表明報告主體
* **經由 Encounter 間接 Reference**：診斷報告可能屬於某次就診
* **Result Observation 間接 Reference**：Observation 資源的 subject 通常 reference Patient
* **請求來源資源 Reference**（ServiceRequest 等）之間的 Patient 參考鏈

這些參考可以支援完整的臨床資料串接與審計流程，讓查詢和分析 Patient 的診斷流程更加完整。

---

## ✅ 結語

FHIR 的 DiagnosticReport 是一個高度連結性與臨床語意豐富的資源，能夠承接一份診斷在一個病歷生命週期中的全貌——從請求、執行、結果至最終發佈與結論。

透過理解這些欄位與可能的 reference paths，我們可以設計更完善的 FHIR 醫療互操作系統，讓資料在系統間流動時能夠完整、可識別並可重用，尤其是與 Patient 關聯的上下游資訊。


