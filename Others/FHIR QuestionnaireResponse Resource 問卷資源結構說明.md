# FHIR QuestionnaireResponse Resource 問卷資源結構說明

[https://hl7.org/fhir/R4/questionnaireresponse.html](https://hl7.org/fhir/R4/questionnaireresponse.html)

# 一、QuestionnaireResponse 是什麼？（先建立正確定位）

一句話定義：

> **QuestionnaireResponse =「某一次、某個人，對某一份 Questionnaire 的實際作答結果」**

三個重點你要牢記：

1. **一定要對應一份 Questionnaire**
2. **是「一次填寫行為」的快照**
3. **答案結構必須對齊 Questionnaire.item.linkId**

---

# 二、QuestionnaireResponse（Resource 本體）欄位

## A️⃣ 繼承自 Resource / DomainResource 的通用欄位

這些你應該已經很熟，我只點「用途重點」：

| 欄位                              | 意義            | 實務用法                            |
| ------------------------------- | ------------- | ------------------------------- |
| `id`                            | 此次作答的資源 ID    | 例如 `QuestionnaireResponse/9876` |
| `meta`                          | 版本、時間、profile | 常放 SDC profile                  |
| `language`                      | 作答內容語言        | 多語問卷時有用                         |
| `text`                          | 人類可讀摘要        | 不影響資料結構                         |
| `extension / modifierExtension` | 擴充欄位          | 作答來源、裝置、App 版本                  |

---

## B️⃣ QuestionnaireResponse 專屬欄位（核心）

### 1️⃣ `identifier`

> 此次作答的業務識別碼

* 可用於：

  * 問卷系統內的流水號
  * 與 HIS / EMR 的表單編號對齊
* **不是 FHIR id 的替代品**

---

### 2️⃣ `basedOn`

> 此次作答是為了回應哪個「請求」

常見來源：

* ServiceRequest
* CarePlan
* Task

📌 **臨床流程導向系統很重要**
👉 例如：「化療 Day 1 → 觸發副作用問卷」

---

### 3️⃣ `partOf`

> 此次作答是某個較大事件的一部分

例子：

* 住院 Encounter
* Clinical workflow

---

### 4️⃣ `questionnaire`（⚠️ 非常重要）

> **指向被填寫的 Questionnaire（canonical URL）**

```json
"questionnaire": "http://example.org/fhir/Questionnaire/chemo-ae"
```

📌 關鍵規則：

* **一定要填**
* 用 canonical（不是 resource reference）
* 系統靠它來：

  * 驗證答案結構
  * 解讀 linkId
  * 重建 UI

---

### 5️⃣ `status`（⚠️ 必填）

| 狀態                 | 意義       |
| ------------------ | -------- |
| `in-progress`      | 尚未填完     |
| `completed`        | 已完成（最常見） |
| `amended`          | 填完後又修改   |
| `entered-in-error` | 作廢       |

📌 **資料分析時通常只看 `completed`**

---

### 6️⃣ `subject`

> 問卷是「為誰填的」

* 通常是 `Patient`
* 但也可以是：

  * Practitioner
  * RelatedPerson
  * Group

👉 對應 Questionnaire.subjectType

---

### 7️⃣ `encounter`

> 作答發生在哪一次就醫事件

📌 臨床問卷（化療、住院評估）**強烈建議填**

---

### 8️⃣ `authored`

> 問卷實際填寫完成時間

⚠️ 注意：

* **不是** 上傳時間
* **不是** 建立 resource 的時間

---

### 9️⃣ `author`

> 是誰填的

常見組合：

* 病人自填 → Patient
* 護理師代填 → Practitioner
* 家屬代填 → RelatedPerson

---

### 🔟 `source`

> 資料來源「代表誰的說法」

📌 差異說明（很多人會搞混）：

| 欄位     | 問的是    |
| ------ | ------ |
| author | 誰實際輸入  |
| source | 代表誰的觀點 |

---

# 三、QuestionnaireResponse.item（答案結構）

> 結構 **必須 mirror Questionnaire.item**

---

## item 欄位一覽

### 1️⃣ `linkId`（⚠️ 必填）

> 對應 Questionnaire.item.linkId

📌 **這是對齊問卷與答案的唯一關鍵**

---

### 2️⃣ `definition`

> 題目所對應的定義（可選）

常用於：

* 後續資料抽取（Observation）
* 表示「這一題在語意上是什麼」

---

### 3️⃣ `text`

> 填寫當下看到的題目文字（可選）

📌 好處：

* 問卷改版後仍保留當時語意
* 稽核、法規、研究很愛

---

### 4️⃣ `answer[]`

> 實際答案（可重複）

#### answer 欄位：

| 欄位         | 意義             |
| ---------- | -------------- |
| `value[x]` | 真正的答案          |
| `item[]`   | 子題答案（用於群組/條件題） |

---

### 5️⃣ `value[x]`（⚠️ 超重要）

型別必須對應 Questionnaire.item.type：

| 題目 type   | 對應 value       |
| --------- | -------------- |
| boolean   | valueBoolean   |
| integer   | valueInteger   |
| decimal   | valueDecimal   |
| string    | valueString    |
| text      | valueString    |
| choice    | valueCoding    |
| date      | valueDate      |
| dateTime  | valueDateTime  |
| quantity  | valueQuantity  |
| reference | valueReference |

📌 **型別錯 = 不合法資源**

---

# 四、巢狀 item 的意義（enableWhen 的結果）

若 Questionnaire 有：

* group
* enableWhen 條件題

👉 QuestionnaireResponse 會：

* **只出現實際被顯示、被回答的 item**
* 未顯示的題目 **不會出現**

📌 這是 FHIR 設計的「事實紀錄」概念

---

# 五、常見實務錯誤（你一定會遇到）

### ❌ 錯誤 1：用題目順序配答案

👉 **一定要用 linkId**

---

### ❌ 錯誤 2：把 UI 顯示邏輯存進 QuestionnaireResponse

👉 UI 是 Questionnaire 的責任

---

### ❌ 錯誤 3：completed 但缺題

👉 若 Questionnaire 設 required，Validator 會擋

---

# 六、你現在這個階段，應該怎麼用？

結合你前面問的內容，實務建議是：

1. **Questionnaire**

   * 定義結構、條件、型別、Extension

2. **QuestionnaireResponse**

   * 只存「事實答案」
   * 嚴格對齊 linkId + type

3. **後處理**

   * 把 QuestionnaireResponse 轉成：

     * Observation
     * Condition
     * Clinical Impression

---

# 一、先定位：為什麼 QuestionnaireResponse 也需要 Profiles / Extensions？

核心規格的 QuestionnaireResponse 只保證三件事：

1. 能對應到一份 Questionnaire
2. 能忠實保存「當次作答的事實」
3. 能被任何 FHIR Server 接收

但它**刻意不規定**：

* 哪些欄位一定要有（例如 encounter、author）
* 哪些答案要能被抽取成 Observation / Condition
* 問卷是不是「臨床可用」還是「純研究 / 試驗」
* 問卷是不是「符合某個 IG / 國家標準」

👉 **這些「如何用」的規則，就交給 Profiles 與 Extensions。**

---

# 二、Profiles：定義「這種 QuestionnaireResponse 應該長怎樣」

## 1️⃣ QuestionnaireResponse Profile 是什麼？

在 FHIR 中，Profile 本質上是 **StructureDefinition**，用來：

* 限制欄位 **必填 / 不可用**
* 限制 Reference 的型別（只能是 Patient？Encounter？）
* 規定某些 Extension **必須存在**
* 宣告「這份 QuestionnaireResponse 的語意等級」

在 questionnaire-response-profiles.html 中，**官方重點放在 SDC（Structured Data Capture）Profiles**。

---

## 2️⃣ 最重要的：SDC QuestionnaireResponse Profile

### 🔹 這個 Profile 解決什麼問題？

一句話：

> **讓 QuestionnaireResponse 不只是「填完的表單」，而是「可被系統理解、驗證、抽取的臨床資料來源」**

它主要做三件事：

1. **收斂用法**

   * 明確要求 linkId、answer.value[x] 的對齊
   * 禁止模稜兩可的結構

2. **為後處理鋪路**

   * 讓系統可以把答案轉成 Observation / Condition
   * 與 Data Extraction、Clinical Reasoning 相容

3. **提高互通性**

   * 不同系統看到 profile，就知道「這份作答能不能用」

---

## 3️⃣ Profile 是怎麼套用到 QuestionnaireResponse？

**只靠一個地方：`meta.profile`**

```json
{
  "resourceType": "QuestionnaireResponse",
  "meta": {
    "profile": [
      "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaireresponse"
    ]
  }
}
```

📌 這代表：

* 這份 QuestionnaireResponse **宣告自己符合 SDC 規範**
* FHIR Validator 可以直接檢查
* 下游系統（ETL、AI、研究平台）可以信任它的結構

---

## 4️⃣ 實務上 Profile 常用來「規定」哪些事？

在 QuestionnaireResponse 上，Profile 常見限制包含：

* `status` **必須是 completed**
* `subject` **必須是 Patient**
* `questionnaire` **必須存在**
* `item.linkId` 與 Questionnaire 必須可對齊
* 禁止出現不在 Questionnaire 中的 item

👉 這些都不是核心規格強制
👉 但在「臨床系統」裡幾乎是**必要規則**

---

# 三、Extensions：補上 QuestionnaireResponse 原本沒有的語意

## 1️⃣ 為什麼 QuestionnaireResponse 需要 Extension？

因為「**填完的答案本身**」還不夠，實務還會問：

* 這份作答是：

  * 病人自填？
  * 護理師代填？
  * App 自動帶入？
* 這份作答：

  * 是否已經被醫師確認？
  * 是否可進入正式病歷？
* 這些答案：

  * 哪些要抽成 Observation？
  * 哪些只是輔助資訊？

👉 這些都**不是核心欄位能表達的**

---

## 2️⃣ 官方頁面中強調的 Extension 使用層級

### 🔹 QuestionnaireResponse.extension（整份作答）

常用來放「**作答層級的語意**」，例如：

* 作答來源系統（App / Web / Kiosk）
* 填寫裝置（手機 / 平板）
* 臨床審核狀態
* 問卷用途（臨床 / 研究 / 試驗）

📌 你在做 SMART on FHIR App 時，**這一層非常重要**

---

### 🔹 QuestionnaireResponse.item.extension（單題答案）

這一層是**進階用法的核心**，常見用途：

* 指定「這一題」要抽成：

  * Observation
  * Condition
  * ClinicalImpression
* 指定答案的：

  * 單位轉換
  * 正規化規則
* 標記：

  * 這題是否臨床關鍵
  * 是否需要醫師確認

👉 **注意**：
UI 行為（顯示/隱藏）**不屬於 QuestionnaireResponse**
那是 Questionnaire + Extension 的責任

---

## 3️⃣ Extension 的基本結構（永遠長這樣）

```json
"extension": [
  {
    "url": "extension 的 canonical URL",
    "valueBoolean": true
  }
]
```

關鍵重點：

* `url` = Extension 的身分與語意（一定要穩定）
* `value[x]` = 真正資料（型別固定）

---

## 4️⃣ 官方 vs 自訂 Extension 的取捨原則

**實務準則（非常重要）**：

1. **先找 IG**

   * HL7 SDC
   * 國家 IG（例如 TW Core）

2. **找不到再自訂**

   * 用你們組織的 domain 當 canonical
   * 一定要寫清楚語意文件

👉 你目前做的「癌症 / 化療 / 副作用問卷」
**非常適合先院內 Extension → 之後升級國家 IG**

---

# 四、Profile × Extension 的分工（一定要記住）

| 問題                     | 用 Profile | 用 Extension |
| ---------------------- | --------- | ----------- |
| 哪些欄位一定要有               | ✅         | ❌           |
| subject / encounter 限制 | ✅         | ❌           |
| 作答結構是否合法               | ✅         | ❌           |
| 作答來源、狀態、語意             | ❌         | ✅           |
| 哪些答案要抽資料               | ❌         | ✅           |
| 與臨床流程整合                | ❌         | ✅           |

一句話總結：

> **Profile 管「能不能用」，Extension 管「怎麼用」**

---

# 五、直接對應你目前的專案（給你一個實務判斷）

以你正在做的 **SMART on FHIR 問卷系統**，合理的最小組合是：

1. **Questionnaire**

   * 使用 SDC Questionnaire Profile
   * 用 enableWhen + item.extension 描述行為

2. **QuestionnaireResponse**

   * 使用 SDC QuestionnaireResponse Profile
   * 用 extension 標記來源、用途、抽取規則

這樣你未來要：

* 接 AI
* 做病人時序圖
* 把問卷變成結構化臨床資料

**都不用回頭重做**



