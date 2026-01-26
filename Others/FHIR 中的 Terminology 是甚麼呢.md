# FHIR 中的 Terminology 是甚麼呢

[耀瑄科技 新創小組]

在 **FHIR** 裡，「**Terminology（醫療術語）**」不是一個附加功能，而是**讓資料「有共同語意、能被電腦理解與計算」的核心基礎**。

---

## 一句話先講清楚

> **FHIR Terminology = 用標準代碼，確保「你寫的醫療資料，別人看得懂、電腦算得動」**

如果沒有 Terminology，FHIR 就只是一堆結構漂亮的 JSON；
**有了 Terminology，FHIR 才能真的跨系統、跨醫院、跨應用。**

---

## 為什麼 FHIR 一定需要 Terminology？

想像下面這個情境：

* 醫師輸入：
  -「高血壓」
  -「HTN」
  -「Hypertension」
  -「I10」

**人看得懂，但電腦怎麼知道它們是同一件事？**

FHIR 的解法是：
👉 **不要只存文字，而是存「代碼 + 代碼系統」**

這整套「代碼的世界」，就是 **Terminology**。

---

## Terminology 在 FHIR 中包含哪幾個核心概念？

FHIR 把醫療術語拆成 **四個關鍵元件**，你只要掌握這四個，就算入門了。

---

### 1️⃣ CodeSystem（代碼系統）

👉 **定義「有哪些代碼」**

CodeSystem 就是一套「官方字典」，例如：

* ICD-10：疾病診斷
* LOINC：檢驗與量測
* SNOMED CT：臨床概念（症狀、處置、狀態）
* ATC：藥物分類
* 院內自訂代碼系統

在 FHIR 中，一個代碼一定要帶：

```json
{
  "system": "代碼系統的 URI",
  "code": "實際代碼",
  "display": "人看的名稱"
}
```

---

### 2️⃣ ValueSet（可用代碼集合）

👉 **定義「這個欄位允許用哪些代碼」**

ValueSet 不是新代碼，而是：

* 從一個或多個 CodeSystem
* 挑出「這個情境允許的代碼集合」

例如：

* 「性別」只能用 Male / Female / Other
* 「Observation.category」只能用 laboratory / vital-signs / imaging

👉 **ValueSet 是「規則」，不是資料本身**

---

### 3️⃣ Binding（綁定規則）

👉 **把 Resource 的欄位，綁定到某個 ValueSet**

例如：

* Observation.code
  → 必須來自「LOINC 檢驗代碼 ValueSet」

Binding 會定義嚴格程度：

* `required`：一定要在 ValueSet 裡，否則錯
* `preferred`：建議用，但可接受其他
* `example`：只是示範
* `extensible`：先用這些，沒有再擴充

這一步，決定了 **FHIR 能不能自動驗證資料正不正確**。

---

### 4️⃣ Terminology Service（術語服務）

👉 **讓 FHIR Server「真的會用代碼」的引擎**

它負責：

* 驗證代碼是不是存在
* 驗證代碼是否符合 ValueSet
* 執行 `$expand`（展開 ValueSet）
* 支援 Profile / IG 的 validation
* 支援 CDS、CQL、品質指標計算

在 HAPI FHIR Server 裡，你必須：

* 安裝 CodeSystem（ICD-10 / LOINC / SNOMED CT）
* 啟用 Terminology 功能
* （常見）啟用 ValueSet 預先展開（Pre-expand）

否則很多功能**不是慢，就是直接失效**。

---

## Terminology 跟 Resource 的關係是什麼？

一句話：

> **Resource 是「資料容器」，Terminology 是「語意標準」**

例如：

```json
"code": {
  "coding": [
    {
      "system": "http://loinc.org",
      "code": "718-7",
      "display": "Hemoglobin [Mass/volume] in Blood"
    }
  ]
}
```

* Resource 決定結構（Observation.code 是什麼型別）
* Terminology 決定內容是否有意義、是否正確

---

## 為什麼在實作時「一定要先裝 Terminology，再裝 IG」？

這是很多新手最容易踩雷的地方。

原因很簡單：

* IG（例如 TW Core IG）
* 會在 Profile 裡 **Binding 到 ValueSet**
* ValueSet 又引用 CodeSystem（ICD-10 / LOINC / SNOMED CT）

👉 **如果術語庫不存在，IG 驗證一定失敗**

所以正確順序一定是：

1. 啟用 Terminology Service
2. 安裝 ICD-10、LOINC、SNOMED CT
3. 確認 ValueSet 可 `$expand`
4. 再安裝 IG（如 TW Core IG）
5. 最後才開始送 Resource

---

## Terminology 不只是「對資料嚴格」，而是「讓後面都能自動化」

一旦 Terminology 建好，你就能做到：

* 自動驗證資料正確性
* 跨院資料語意一致
* 病患篩選（cohort）不靠人工對字
* 臨床決策支援（CDS）
* CQL 規則可重用
* AI / 分析模型資料前處理大幅簡化

---

# FHIR 常用 Code System 一覽（實務導向）

---

## 一、診斷與疾病相關（Disease / Problem）

### 1️⃣ ICD-10-CM

**國際疾病分類（臨床修訂版）**

* **代表意義**：疾病與診斷的標準分類
* **常見用途**

  * Condition（診斷）
  * 病患篩選（cohort）
  * 健保申報 / 統計
* **FHIR 常見使用位置**

  * `Condition.code.coding`
* **system URI**

  ```
  http://hl7.org/fhir/sid/icd-10-cm
  ```
* **實務提醒**

  * 適合「診斷名稱」
  * 不適合描述症狀、臨床狀態細節（那是 SNOMED CT）

---

### 2️⃣ SNOMED CT

**臨床醫學概念本體（最完整）**

* **代表意義**：臨床世界的「語意模型」
* **常見用途**

  * 診斷（Problem）
  * 症狀、體徵
  * 手術、處置
  * 臨床狀態描述
* **FHIR 常見使用位置**

  * Condition
  * Observation.code / valueCodeableConcept
  * Procedure
* **system URI**

  ```
  http://snomed.info/sct
  ```
* **實務提醒**

  * 非常強大，但**授權與安裝成本高**
  * 台灣多用於 **FHIR / CDS / AI 專案**

---

## 二、檢驗與量測（Lab / Vital）

### 3️⃣ LOINC

**檢驗與臨床量測代碼標準**

* **代表意義**：醫療「測了什麼」
* **常見用途**

  * 檢驗（血液、尿液）
  * 生命徵象（身高、體重、血壓）
* **FHIR 常見使用位置**

  * `Observation.code`
* **system URI**

  ```
  http://loinc.org
  ```
* **實務提醒**

  * Observation 幾乎「一定要用」
  * 與數值（Quantity）搭配最常見

---

## 三、藥物相關（Medication）

### 4️⃣ ATC

**藥物治療分類系統**

* **代表意義**：藥物的藥理與治療分類
* **常見用途**

  * 藥物研究
  * 藥物群組分析
* **FHIR 常見使用位置**

  * Medication.code
* **system URI**

  ```
  http://www.whocc.no/atc
  ```
* **實務提醒**

  * 不等於「實際藥品」
  * 常與院內藥品碼並用

---

### 5️⃣ RxNorm（台灣較少）

* **代表意義**：美國藥品標準名稱
* **用途**

  * 藥品互通
  * 藥物決策支援
* **system URI**

  ```
  http://www.nlm.nih.gov/research/umls/rxnorm
  ```

---

## 四、FHIR 內建系統代碼（非常重要）

### 6️⃣ HL7 FHIR CodeSystem

**FHIR 自己定義的狀態與分類代碼**

* **代表意義**：FHIR 系統內部用語
* **常見用途**

  * 狀態（status）
  * 類型（category）
  * 關係（relationship）
* **system URI（常見）**

  ```
  http://terminology.hl7.org/CodeSystem/...
  ```
* **實務提醒**

  * 幾乎每個 Resource 都會用
  * **不可亂填文字**

---

## 五、行政與流程相關

### 7️⃣ HL7 Act / v3 CodeSystem

* **代表意義**：醫療流程、角色、動作分類
* **用途**

  * Encounter 類型
  * Practitioner 角色
* **system URI**

  ```
  http://terminology.hl7.org/CodeSystem/v3-ActCode
  ```

---

## 六、院內 / 客製 Code System（一定會遇到）

### 8️⃣ 院內自訂 Code System

* **代表意義**：暫時無國際對應的院內代碼
* **常見用途**

  * 檢驗項目代碼
  * 處置代碼
  * 藥品代碼
* **system URI（建議）**

  ```
  https://hospital.example.tw/CodeSystem/lab-item
  ```
* **實務原則**

  * 一定要有 URI
  * 一定要有說明文件
  * 能對應國際碼時，建議「雙 coding」

---

## 七、Code System 要怎麼「正確使用」？

### 標準寫法（FHIR 通用）

```json
"coding": [
  {
    "system": "http://loinc.org",
    "code": "8480-6",
    "display": "Systolic blood pressure"
  }
]
```

### 實務最佳實踐（非常重要）

✅ 可以多個 coding（院內 + 國際）
❌ 不要只放 display
❌ 不要 system 空白
❌ 不要把 code 當文字亂寫

---

## 八、新手最常犯的錯（一次提醒）

* ❌ 把 ICD-10 用在 Observation.code
* ❌ 用自由文字代替 coding
* ❌ system URI 寫錯或亂寫
* ❌ 沒裝 Terminology 就裝 IG
* ❌ 沒定義院內 CodeSystem URI

---

## 九、快速選擇指南（一眼就懂）

| 你在描述    | 用什麼 Code System     |
| ------- | ------------------- |
| 疾病診斷    | ICD-10 / SNOMED CT  |
| 症狀、狀態   | SNOMED CT           |
| 檢驗項目    | LOINC               |
| 生命徵象    | LOINC               |
| 藥物分類    | ATC                 |
| FHIR 狀態 | HL7 FHIR CodeSystem |
| 院內代碼    | 自訂 CodeSystem       |

---

## 一句話總結

> **FHIR 的 Code System，是讓資料「有語意、可驗證、能互通、能計算」的基礎。
> 選對 Code System，比 JSON 寫得漂亮更重要。**




