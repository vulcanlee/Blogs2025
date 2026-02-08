# FHIR Patient Resource 病患資源結構說明

FHIR 是目前醫療領域最重要的跨系統交換標準之一，其核心定義了許多常用的臨床與行政資訊資源，其中 Patient Resource 負責描述「病人基本人口／身份」資訊，是許多臨床交換與查詢鏈的起點。FHIR Patient 資源代表接受照護「個人（或動物）」的基本資料，用來建立、辨識、管理病人的人口統計學與行政資訊。Patient 不包含病人病歷細節（例如臨床紀錄），但常被其他資料資源所引用。

下面是根據 **FHIR R4 Patient Resource** 官方規範（來源：FHIR v4.0.1 Patient Resource／Profiles & Extensions 章節資訊）整理而成的完整 Markdown 格式內容，內容涵蓋：

* Patient 資源中所有欄位的名稱、意義與基本用法
* Patient 所可能連結（Reference）到的其他資源，以及其他資源可能連結到 Patient
* Patient 相關 **Extension 與 Profile** 的基本知識與擴展使用場景

> 內容主要參考官方 *FHIR R4 Patient* 規格定義（資源欄位），並輔以一般實作與臨床情境中的 Profile/Extension 概念整理。

## Patient 資源常見欄位（逐項說明）

### 基礎識別與狀態欄位

* **identifier**
  多個病人識別碼，如醫療記錄號碼（MRN）、健保號碼等。FHIR 中用 identifier 存放這些外部系統識別碼。

* **active**
  Boolean 值表示該病人紀錄是否目前有效，例如是否為現行病人或已退件狀態。

---

### 名稱與通訊資訊

* **name**
  病人的姓名（HumanName）。可以有多個，如正式姓名、別名等。

* **telecom**
  聯絡方式清單，例如電話、Email、SMS 等。

---

### 基本人口數據

* **gender**
  行政性別。Allowed values：male | female | other | unknown。

* **birthDate**
  出生日期。

* **deceased[x]**
  病人是否已逝。依型別可選 boolean 或 datetime。

---

### 位址與社會狀態

* **address**
  病人地址資訊（可以有多筆）。

* **maritalStatus**
  婚姻狀態（CodeableConcept）。

---

### 其他識別與描述

* **multipleBirth[x]**
  是否為多胞胎（Boolean 或整數表出生序）。

* **photo**
  病人照片圖檔（Attachment）。

---

## 內嵌元素

FHIR 的 Patient Resource 本身也有一些 **backbone elements（複合結構）**：

### Contact

* **contact**
  病人的緊急聯絡人或代表者，其下可含：

  * relationship：與病人的關係
  * name：名稱
  * telecom：聯絡方式
  * address：地址
  * gender：性別
  * organization：此人所屬的組織
  * period：此聯絡人有效期間

這些欄位通常在臨床場景中被用來標註病人的緊急聯絡、監護人或代理人資訊。

---

### Communication

* **communication**
  病人可用的語言資訊

  * language：ISO language code
  * preferred：是否為優先語言

此元素有助於跨語言環境中的照護與問診溝通。

---

## Patient 的 Reference（鏈結）

FHIR 設計中，Patient 資源本身並不直接含有大量臨床病歷，但其他 FHIR 資源會使用 **Reference(Patient)** 去指向病人。FHIR 官方文件列出了很多可能會參考 Patient 的資源。

### 支援 Reference 到 Patient 的資源

下列為官方 Patient 規範中列出的部分資源，這些資源可能透過 reference 欄位指向 Patient：

* **Encounter**：臨床訪視事件其 subject 可能是 Patient
* **Observation**：病人的測量或評估結果
* **Condition**：病人的臨床診斷
* **Immunization**：免疫接種紀錄
* **MedicationRequest / MedicationAdministration / MedicationStatement**：用藥相關資源
* **AllergyIntolerance**：過敏或不耐症
* **Procedure**：執行的醫療程序
* **Claim / ExplanationOfBenefit**：保險理賠申請
* **CarePlan / CareTeam**：照護計畫與團隊
* …以及許多其他以 Patient 為核心的臨床紀錄相關資源。

※ 以上列舉並非全部，但這是 Patient 在臨床流程與醫療資訊系統中最常被引用的情境。

---

## Patient 會 Reference 其他資源

在 Patient 的欄位裡也包含對其他資源的 reference，用來建立直接關聯：

* **generalPractitioner**
  患者的主要照護提供者，可指向：

  * **Organization**
  * **Practitioner**
  * **PractitionerRole**
    這讓病人可以追蹤其主要照護醫師或醫療機構。

* **managingOrganization**
  管理此病人紀錄的組織。

* **link.other**
  用來指向另一個 Patient 或 RelatedPerson，用於描述此紀錄與另一個實體之間的關係（例如合併紀錄）。

---

## Patient Resource 的 Extensions

FHIR 的一大特色是擴展性，當基礎欄位不足以描述某些地區或業務需求時，可以定義 **Extension**：

* Extension 可以加在任一 Patient 層級，用以補充**無於基本資源中的資訊**
* 常見使用場景包括：

  * 種族與族裔資訊
  * 國籍、語言偏好的詳細資訊
  * 臟器捐贈者狀態
  * 其他地區政府需求的欄位

官方規範本身並不定義所有 jurisdiction 的 extension，但規則允許各實作指引（Implementation Guide）及地區標準擴充 Patient 以滿足當地需求。

---

## Patient 的 Profiles（實作指引）

**Profile** 是對 Patient Resource 的一種約束與擴充，用以定義：

* 哪些欄位 *SHALL* 存在
* 如何使用某些欄位
* 在此地區或場景下需要新增哪些 extensions

這在準備跨機構交換或符合特定政策時很重要。常見 Profile 例如：

* **U.S. Core Patient**（用於美國政府互操作標準）
* **TW Core Patient**（臺灣核心實作指引定義的 Patient Profile）
  定義病人基本資料應在 R4 Patient 內如何呈現與使用。([twcore.mohw.gov.tw][2])

這些 Profile 提供更嚴格的約束和必要欄位，讓交換資訊更一致可用。

---

## 小結與實作建議

FHIR Patient 是整個臨床資訊交換鏈路的核心入口：

* 基本人口資訊、識別碼與聯絡方式是 Patient 資源的基礎
* 多數臨床與財務記錄資源都會用 Reference 指向 Patient
* Patient 中也會指向代表其主要提供者、管理組織、聯絡人
* Extensions 與 Profiles 可依地區法規或需求擴充 Patient

在規劃系統或介接標準 API 時，理解 Patient 與其他資源之間的 Reference 關係，有助於正確串接、建立完整病人照護檔案。



