# FHIR Procedure Resource 臨床程序資源結構說明

Procedure 資源是 FHIR 中用來描述「臨床程序」的核心資源，其用途、欄位設計與和其他資源（特別是 Patient）之間的連結機制，是理解 EHR 臨床事件數據交換的關鍵。

---

## 什麼是 FHIR Procedure？

在 FHIR R4 中，Procedure 定義為一項已執行或正在執行的臨床行為，是「對 Patient（或 Group）」所進行的活動。它不只限於手術，也包括所有對病人身體或心理產生變化的活動。這些活動包括了手術、診斷檢查、活體組織檢查（biopsy）、輔導、物理治療等。這個資源提供的是該程序的摘要資訊，而不是過程的即時進度詳細資料。

這個定義讓 Procedure 資源在醫療紀錄中扮演「事件描述」的角色，可以串聯不同系統中的程序紀錄，並可被其他內容（Observation、DiagnosticReport 等）引用。

---

## Procedure 資源的價值

FHIR Procedure 資源的價值在於它做為橋接臨床流程與患者記錄之間的標準化描述：

* 統一紀錄程序發生的時間、狀態與分類，利於跨系統查詢與交換。
* 明確的欄位可與多種其他資源建立關聯，促進資訊整合（例如：Observation、DiagnosticReport 等）。
* 標準化的參考連結（Reference） 使得程序與病人、照護事件、醫療提供者之間的關係可追蹤與串接。

---

## Procedure 資源的欄位與用途

FHIR Specification 官方文件以清單方式定義 Procedure 資源的所有欄位。下列內容按照規格逐項解釋每個欄位的意義與用法：

**identifier**
用來存放 Procedure 的標識符（Identifier），可以對應外部系統中的程序編號或紀錄碼。

**instantiatesCanonical & instantiatesUri**
可連結程序所依據的流程定義或規範，例如：PlanDefinition、ActivityDefinition 或外部規範 URI，表示該程序實際符合某個醫療工作流程。

**basedOn**
Reference 指向與程序相關的請求或計劃，例如 CarePlan、ServiceRequest。它反映出這個程序是由哪個請求觸發或依據。

**partOf**
Reference 可以指向其他更大的事件（例如另一個 Procedure、Observation 或 MedicationAdministration），表示此程序是某個更大事件的一部分。

**status**
指定 Procedure 當前的狀態，例如「completed」、「in-progress」、「entered-in-error」等，描述程序是否執行完成或有效。

**statusReason**
若狀態非正常完成，可以指出原因，如「cancelled due to patient refusal」等。

**category**
程序類別， 可用 SNOMED CT 之類的系統做分類，例如外科手術、理療等。

**code**
程序的指定內容，用特定分類系統編碼描述程序本身的類型，例如手術名稱、檢查名稱等。

**subject**
最具關聯性的欄位之一，用來參考該程序是在哪個 Patient 或 Group 上被執行。這是 Procedure 與病患之間最基本的聯結（見下一節詳述）。

**encounter**
Reference 指向 Encounter（就醫事件），表示程序是在那次門急診/住院事件中執行。

**performed[x]**
程序執行的時間，可以是 dateTime、Period、Age、Range、或文字等多種型別，視實際需求而定。

**recorder**
Reference 誰記錄了此程序資料，可能是 Patient、自身 RelatedPerson、Practitioner 或 PractitionerRole。

**asserter**
Reference 誰宣稱程序已經發生，表示程序紀錄的宣告者。

**performer**（Backbone 元素）
這是一個包含 function（參與角色，比如 surgeon、anesthetist）和 actor（Reference） 的結構，用於指引具體的執行者。actor 可能 reference Practitioner、PractitionerRole、Organization、Patient（例如自主治療）、RelatedPerson 或 Device。

**location**
Reference 指向 Location（地點），標示該程序在哪裡執行。

**reasonCode** 與 **reasonReference**
reasonCode 給出程序執行的原因（以 CodeableConcept 表示）。reasonReference 是可 Reference 至 Condition、Observation、Procedure、DiagnosticReport 或 DocumentReference 的欄位，以便指向具體的理由來源或診斷支持。

**bodySite**
表示程序涉及的身體部位，可用標準解剖碼表描述。

**outcome**
表示程序的結果或結局，例如是否達到預期效果。

**report**
Reference 指向與程序相關的結果報告，例如 DiagnosticReport、DocumentReference 或 Composition。

**complication** 與 **complicationDetail**
Complication 可用 CodeableConcept 表示是否有併發症；complicationDetail 則 reference 到 Condition 表示具體的併發症診斷。

**followUp**
CodeableConcept 顯示後續追蹤與指示，例如安排返診、術後復健等。

**note**
Annotation 形式的自由文本附註，可加入額外說明。

**focalDevice / usedReference / usedCode**
用於紀錄程序中涉及的器材或物質（如 Device、Medication、Substance），包括是否改變了該設備（focalDevice）或使用了哪些資源（usedReference、usedCode），如手術植入物、消毒用品等。

---

## Procedure 與 Patient 之間的 Reference 連結

在 FHIR 設計下，Reference 是在一個資源內部使用的一種結構，用來指向其他資源的 identity（ID 或者 URL）。Procedure 資源內最重要的 Reference 之一就是 **subject**，它表示該程序是在誰身上執行。

在 Procedure 中，subject 的參考目標是 Patient 或 Group。典型情況下 subject 會 reference 一個 Patient 資源的實例。

這種設計使得可以透過 Procedure.subject 指向病人來源，從而在查詢病人相關程序時可以展開反向查詢。例如對一位病人，可以搜尋所有其 subject 為該 Patient 的 Procedure 資源。

除了 subject，Procedure 還有其他可能 reference Patient 的欄位，例如 recorder、asserter 與 performer.actor 甚至某些特定 implementation 也可能讓 performer.actor reference Patient（表示病人在自身身上進行程序）。

---

## Patient 也會 reference 其他資源（與 Procedure 有關）

Patient 資源以自身為中心定義患者的基本資訊，但與 Procedure 相關的情況通常不是 Patient 主動 reference Procedure。FHIR 中的概念是「事件大多由事件資源指向患者」，非由患者主動指向事件。

因此：

* Procedure 引用 Patient 為 subject。
* Patient 資源中並無欄位直接儲存所有與其相關的 Procedure 列表。
* 真正的關聯是在資料交換時透過 query 搜尋例如 `Procedure?subject=Patient/{id}` 這種查詢實現。
* Patient 可 reference 其自己的 GeneralPractitioner、RelatedPerson 等，但這些不直接鏈到 Procedure。

---

## Profiles 與 Extensions：如何客製 Procedure

FHIR 的強項之一是可透過 **Profiles** 和 **Extensions** 自訂標準行為。

### Profiles（規範限制）

Procedure-profiles.html 頁面列出不同 Implementation Guide 所對 Procedure 的具體限制與約束，例如 US Core Procedure Profile、IPS Procedure Profile 等。這些 Profile 會指定某些欄位必須存在、允許值必須使用特定 ValueSet，或對某些 Reference 連結作出額外限制。例如 US Core Procedure Profile 要求一定要有 subject、status 以及 code 且符合 USCDI 資料需求。

Profiles 的目的：

* 提供特定地域或業務需求下更嚴格或更一致的 Procedure 定義。
* 對欄位使用特定 ValueSet 或強制特定格式，使資料更可交換與可解析。

### Extensions（擴充欄位）

FHIR 允許資源在不破壞標準基礎結構下透過 Extensions 擴充欄位。Extensions 允許你新增不在官方規範中的資料點，例如額外的執行詳細資訊、支付相關資訊等。Extensions 可分為一般 Extensions 與 modifierExtensions（後者會影響欄位語義）。Implementation Guide 以及 Profiles 也常會定義一些常用的 Extensions。

---

## 小結：Procedure 的臨床與系統意義

FHIR R4 Procedure 資源是臨床程序記錄的標準化描述結構。它不僅包含時間與分類資訊，更透過豐富的 Reference 將程序與 Patient、Encounter、Practitioner、Observation、DiagnosticReport 等資源串聯，使得臨床事件串接與資料交換變得可靠且可查詢。透過 Profiles 與 Extensions 的可擴展性，可以滿足不同地區或業務的特殊需求。



