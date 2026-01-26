# FHIR 工作坊 第一段 FHIR 的介紹 筆記

[耀瑄科技 新創小組]

---

# 把「工作坊口語」翻成「可落地的 FHIR 方案」：從 Resource、Profile、IG 到 CDI、CQL、CDS（含 TW Core）

> 這是「工研院受衛福部委託之 FHIR 工作坊」活動的筆記，，因此我會主動補齊背景、定義、流程，並把零散出現的關鍵字（Resource / Profile / IG / CDI / CQL / CDS / TW Core IG / TW Core CDI / Bundle / Extension / Binding / 標準代碼）整合成完整章節。

---

## 1) 背景脈絡：為什麼「現在」要把醫療資料講成 FHIR？

工作坊的核心目的，不是「教你背規格」，而是把醫療資料交換從「各院各寫一套」拉到「可重用、可驗證、可上架/可擴散」的軌道上。

在台灣情境，常見困境是：

* HIS/EMR 資料結構與欄位命名高度院內化，跨院專案幾乎都在做「一次性 mapping + 一次性介接」。
* 想做 AI、決策支援（CDS）、跨系統流程整合時，資料定義不一致會讓後段成本指數上升。
* 所以需要一套「可被共同遵循」的資料表達方式，FHIR 的價值在於：**把「資料結構」與「交換方式」標準化**，讓上層應用（AI、研究、提醒、流程）可以在一致基礎上擴張。

---

## 2) 先把名詞放到同一張地圖：Resource、Profile、IG、CDI、CQL、CDS 各在解哪個問題？

這些詞在口語常被混用。最容易落地的理解方式是：**每個名詞都在解「一致性」的不同層級問題**。

### 2.1 Resource：FHIR 的「資料結構語言」

* **Resource** 是 FHIR 用來表達醫療資料的最小可交換單位（例如 Patient、Observation、Condition、Encounter、MedicationRequest…）。
* 你可以把 Resource 想成「醫療資料的 JSON 物件模板」：欄位名稱、資料型別、可否重複、可否省略等，都有基本規格。
* 工作坊裡講者用「Resource 就是一個資料結構，用來把病人的狀態/問題/過程表達出來」來帶入這個概念。

### 2.2 Profile：在 Resource 之上加「約束」，讓它可被實務採用

FHIR R4 原生 Resource 設計得很彈性（為了涵蓋全球場景），但也因此在落地時會遇到兩個問題：

1. 欄位太多，大家各填各的
2. 同一件事可以用好幾種表示法（例如身分證號、病歷號、檢驗代碼、機構代碼）

**Profile** 就是在「某個 Resource」上加上更精確的規範（Constraint），例如：

* 指定哪些欄位必填（Must Support / Required）
* 限定欄位的 cardinality（0..1、1..1、0..*…）
* 限定可用的代碼系統（Binding 到 LOINC/SNOMED/ICD-10…）
* 限定參照對象（Reference 只能指到哪些 Resource/Profile）
* 規定 Extension 的使用方式

工作坊口語中也提到「原本 Resource 的範圍可能很大，但在某個規範（你可以理解為 profile/guide）裡會更進一步限制，讓它符合特定情境需要」。

### 2.3 IG（Implementation Guide）：把一整套「Profile + 規則 + 範例 + 工具鏈」包成可落地的規格書

如果 Profile 是「單一 Resource 的約束」，那 **IG** 就是「一整個領域/專案的可落地規格」：

IG 通常包含：

* 使用哪些 Resource（與其 Profile）
* 這些 Resource 之間怎麼關聯（Reference / Workflow）
* 代碼系統與 ValueSet（Binding 規則）
* 必要的 Extensions
* 範例（Examples）
* 甚至提供模板（Template）、對應表、驗證方式、轉換器等

課程中提到「IG 裡面會有模板、example、也能看每個欄位怎麼填、怎麼對應」以及「可透過轉換器、CSV 做資料對應與上傳」，這正是 IG 在落地時的典型形態：不是只有文件，還會帶工具與樣板。

### 2.4 CDI（Clinical Data Interchange）：最小可交換資料集，解「跨系統先跑起來」的問題

CDI 常被理解成「臨床資料交換的最小集合」。在推動初期，最務實的策略是：

* **不要一開始就想把全院資料都 FHIR 化**
* 先選「高價值場景」需要的最小資料集（CDI）
* 把最小集合做準、做穩、可驗證、可重用
* 再逐步擴大

> 你可以把 CDI 視為「分期導入的切片」：每片都可交換、可驗證、可迭代。

### 2.5 CQL 與 CDS：資料一致後，上層才能做「可攜帶的規則」

* **CQL（Clinical Quality Language）**：用來寫臨床邏輯/品質量測/條件判斷的語言（例如某族群篩選、指標計算、品質規則）。
* **CDS（Clinical Decision Support）**：把規則實際用在臨床流程裡（提醒、建議、警示、建議檢查/用藥等）。

工作坊口語把它描述成：醫院在診斷時不只用資料，也會用「規則」協助判斷，並提到匯入規則後可協助臨床判斷。
把它翻成落地語言就是：**FHIR 是讓資料一致；CQL/CDS 是讓「邏輯」也能跨系統重用。**

---

## 3) TW Core IG 與 TW Core CDI：台灣落地時你「應該先看」的那一套

對台灣醫院資訊人員/醫療新創來說，最重要的一句話是：

> 不要從「FHIR 原廠規格」開始做本地互通；請優先對齊 **TW Core IG**（以及其衍生的 CDI/專案 IG）。

原因很現實：

* 台灣會有特定的識別碼、機構結構、健保相關代碼、文件與流程習慣
* TW Core 會對常用資源給出更一致的 Profile 與術語用法（例如 Patient 身分識別、Organization/Location 的表達、Observation 常見類型、Encounter 形式等）
* 對外互通（跨院/沙盒/上架）時，對齊 TW Core 的成本遠低於你自創一套再去解釋

> 你可以把 TW Core IG 當成「台灣版的共同語言底座」。
> TW Core CDI 則像是「先跑起來」的最小資料集切片。

---

## 4) Resource vs Profile：最常見的新手誤解（但也是專案成敗關鍵）

**一句話辨識：**

* **Resource**：FHIR 原生「能表達什麼」
* **Profile**：在某個專案/國家/場景下「你必須怎麼表達」

具體差異表（用工程語言表達）：

| 面向   | Resource    | Profile                                             |
| ---- | ----------- | --------------------------------------------------- |
| 定位   | 基礎資料模型（通用）  | 專案/國家/領域的約束規格                                       |
| 目的   | 覆蓋最大可能性     | 降低自由度、提高一致性                                         |
| 控制手段 | 標準欄位/資料型別   | 必填/可選、Cardinality、Binding、Reference 限制、Extension 規範 |
| 導入結果 | 「能交換」但可能不一致 | 「可互通」且可驗證                                           |

工作坊中提到「Resource 本身彈性很大，但在更精緻化的規範會做進一步限制」就是這件事。

---

## 5) Bundle：為什麼要把 Resource「打包」？

當你做資料交換時，常見需求不是「只傳一個 Observation」，而是要傳一組互相關聯的資料，例如：

* 一位病人（Patient）
* 一次就診（Encounter）
* 當次的診斷（Condition）
* 當次的檢驗（Observation）
* 當次的處置（Procedure）或用藥（MedicationRequest）

這時就會用 **Bundle**：

* Bundle 是 FHIR 的「封裝容器」：把多個 Resource 以固定格式打包
* 交換、上傳、測試、示範時都非常常用
* 也適合做「最小可重現案例」（對沙盒測試/跨院對接非常重要）

實務上常見的 Bundle 類型：

* `transaction`：一次打包送到 FHIR Server，伺服器依序建立/更新
* `collection`：純收集，不含伺服器端交易語意
* `document`：偏文件式封裝（例如 CDA 轉換後的文件包）

---

## 6) Extension 與 Binding：你一定會用到的「兩個現實妥協」

### 6.1 Extension：當 Resource 沒有你要的欄位，怎麼辦？

FHIR 的核心原則是：**不要隨便加自訂欄位破壞互通**。
所以擴充要走 **Extension** 機制：

* 用標準方式在 Resource 上掛「額外欄位」
* Extension 本身也可以被 Profile 約束（例如必填、型別、用途）
* 延伸欄位若可被社群共用，最好在 IG 裡正式定義，避免各案各寫

### 6.2 Binding：同一個欄位可以填任何字？不行，醫療需要「可計算、可比對」

很多欄位是 `CodeableConcept` / `Coding`，如果大家用自由文字，後續 AI、研究、CDS 都會失效。
因此會用 **Binding** 把欄位綁到 ValueSet/CodeSystem，例如：

* 檢驗：LOINC
* 診斷：ICD-10-CM、SNOMED CT
* 藥品：ATC、院內碼、健保碼（需清楚標示 system）
* 手術/處置：ICD-10-PCS、SNOMED CT、院內處置碼
* 機構：院內機構碼、或 TW Core 的建議結構

---

## 7) 從「院內欄位」到「可交換的 FHIR」：一條新手最需要的落地流程

逐字稿裡的流程散落在多段，例如「IG 有模板與 example」、「可以用轉換器」、「CSV 對應後上傳」、「欄位有資料型別、Coding、限制」等。我把它整理成一條可執行的工程流程：

### Step 0：選場景（用 CDI 思維切片）

* 先定義「最小可用資料集」：你要解的問題是什麼？
  例如：病人基本資料 + 就診 + 診斷 + 檢驗（足以做時序圖、篩選、基礎 CDS）
* 避免一開始就做全院資料，否則你會卡在 mapping 地獄

### Step 1：選 Resource 子集（先用主幹）

對大多數「第一次接觸」的專案，最常用的主幹是：

* Patient（病人）
* Encounter（就診）
* Condition（問題/診斷）
* Observation（檢驗/生命徵象/量測）
* MedicationRequest（處方/用藥）
* Organization / Practitioner / Location（誰提供服務、在哪裡）
* Procedure / ServiceRequest（處置/開立檢查）

### Step 2：對齊 Profile（優先 TW Core IG）

* 不要直接「照你想的填 JSON」
* 先看 TW Core IG 對這些資源的要求：必填欄位、身分識別寫法、代碼綁定、Reference 方向
* 你要做跨院或沙盒測試時，這一步會決定你後面省多少工

### Step 3：做欄位對應（Mapping）

把院內資料欄位逐一對應到：

* FHIR 元素（element）
* 需要的 Coding system（system + code + display）
* 是否需要 Extension
* 是否需要轉換資料型別（字串 → date、數字 → Quantity、院內碼 → Coding）

> 這一步通常會產出「Mapping 表」：院內欄位 → FHIR element → 轉換規則 → 代碼系統

### Step 4：用模板/Example 快速對照（避免憑空寫）

IG 內通常會提供模板與 example，讓你直接看「正確長相」。逐字稿也提到「每個模板有對應 example，可以連過去看」。

### Step 5：驗證（Validation）

驗證要分兩層：

* **結構驗證**：JSON 是否符合 Resource schema
* **規範驗證**：是否符合 Profile（必填、cardinality、binding、reference 限制）

### Step 6：Bundle 打包、交易上傳（Transaction）

把「一個病人 + 一次就診 + 當次資料」打成 transaction Bundle：

* 測試最容易
* Demo 最清楚
* 錯誤最可追

---

## 8) 常用標準代碼：你不用一次背完，但要知道「system」要寫清楚

工作坊口語提到「藥物在 FHIR 裡會有不同方式，健保署的藥物碼也會用到」這一類情境，背後的工程原則是：**代碼一定要可追溯**（system 明確）。

建議你在專案初期就訂好「代碼策略」：

* 能用國際標準就用國際標準（LOINC/SNOMED/ICD/ATC）
* 非用院內碼不可時，也要：

  * 定義自家 CodeSystem URI（例如 `https://hospital.example.tw/CodeSystem/lab-item`）
  * 在 IG 或專案文件明確描述其語意、版本、治理方式
* 跨院互通時，至少要提供「院內碼 ↔ 標準碼」對照策略（哪怕先做高頻項目）

---

## 9) 小結：你應該帶走的三個「落地判斷準則」

1. **FHIR 導入不是先寫 API**，而是先把「資料語意」固定下來：Resource → Profile → IG → CDI 切片。
2. **TW Core 是台灣互通的共同底座**：先對齊它，你的擴充與說服成本會顯著下降。
3. **資料一致後才談規則（CQL/CDS）**：否則規則不可攜、不可驗證、不可重用，最終變成每院一套客製。





