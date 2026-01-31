# FHIR Questionnaire Resource 問卷資源結構說明

[https://hl7.org/fhir/R4/questionnaire.html](https://hl7.org/fhir/R4/questionnaire.html)

## Questionnaire（Resource 本體）欄位

> 這一層是在描述「整份問卷」：識別、版本、發布狀態、描述、適用情境、以及題目清單（item）。([hl7.org][1])

### A) 從祖先繼承（Resource / DomainResource）常見欄位

* `id`：此資源在伺服器上的邏輯 id（例如 `Questionnaire/123`）。常用於 read/update。([hl7.org][1])
* `meta`：版本、lastUpdated、profile 等中繼資訊。做版本控管、符合 TW Core/IG 時很常用。([hl7.org][1])
* `implicitRules`：若內容依賴某些「未明示」規則，放在這。實務上較少用。([hl7.org][1])
* `language`：資源內容的語言（例如 `zh-TW`），影響顯示/解讀。([hl7.org][1])
* `text`：Narrative（可讀的摘要 HTML）。用於人類閱讀，不是結構化資料來源。([hl7.org][1])
* `contained`：把小資源內嵌在此 Questionnaire 裡（較少用於問卷）。([hl7.org][1])
* `extension` / `modifierExtension`：擴充欄位（例如 SDC IG、各種問卷行為擴充常靠它）。([hl7.org][1])

### B) Questionnaire 自己定義的欄位（問卷層）

* `url`：此問卷的 canonical URL（全球唯一、可被引用）。做「版本/衍生/引用」時很重要。([hl7.org][1])
* `identifier[]`：額外識別碼（例如院內表單代碼、表單流水號）。([hl7.org][1])
* `version`：商務/內容版本字串（例如 `1.0.3`）。([hl7.org][1])
* `name`：電腦友善名稱（偏 code-friendly）。例如 `ChemoAE_v1`。([hl7.org][1])
* `title`：人類友善標題（畫面顯示常用）。([hl7.org][1])
* `derivedFrom[]`：衍生自哪個 Questionnaire（canonical reference）。用於「以既有表單為模板改版」。([hl7.org][1])
* `status`：`draft | active | retired | unknown`。系統列出可填問卷時，通常只挑 `active`。([hl7.org][1])
* `experimental`：是否僅測試用。([hl7.org][1])
* `subjectType[]`：此問卷回覆（QuestionnaireResponse）的 subject 可以是哪種 Resource（常見 `Patient`，也可能是 `Practitioner` 等）。([hl7.org][1])
* `date`：問卷最後變更時間。([hl7.org][1])
* `publisher`：發布者名稱（機構/個人）。([hl7.org][1])
* `contact[]`：發布者聯絡資訊。([hl7.org][1])
* `description`：問卷自然語言描述（markdown）。([hl7.org][1])
* `useContext[]`：適用情境（例如場域、族群、情境標籤）。常用於「系統過濾出某情境可用的問卷」。([hl7.org][1])
* `jurisdiction[]`：適用司法管轄/地區（例如國家/區域）。([hl7.org][1])
* `purpose`：為什麼定義此問卷（markdown）。([hl7.org][1])
* `copyright`：使用/出版限制（markdown）。([hl7.org][1])
* `approvalDate`：發布者核准日期。([hl7.org][1])
* `lastReviewDate`：最後審查日期。([hl7.org][1])
* `effectivePeriod`：預期使用期間（Period）。例如只在某計畫期間內有效。([hl7.org][1])
* `code[]`：代表「整份問卷」的概念代碼（Coding）。用於把問卷分類、與術語系統對應。([hl7.org][1])
* `item[]`：題目與群組的樹狀結構（最核心）。([hl7.org][1])

---

## Questionnaire.item（題目/群組）欄位

> 一個 `item` 可以是：**群組(group)**、**純顯示(display)**、或**可回答的題目（boolean/integer/choice/...）**。問卷就是靠 item 形成樹狀結構。([hl7.org][1])

### item 基本欄位

* `linkId`（必填）：**問卷內唯一**的題目 id（機器用）。後續 QuestionnaireResponse 會用它對答案。([hl7.org][1])
* `definition`：把題目連到某個資料定義（ElementDefinition / StructureDefinition）—常用於「之後把答案抽取成 Observation、ServiceRequest…」。([hl7.org][1])
* `code[]`：題目對應的術語概念（Coding）。注意：`display` 類型不允許宣告 `code`（規則）。([hl7.org][1])
* `prefix`：顯示用題號（如 `1(a)`、`2.5.3`）。([hl7.org][1])
* `text`：顯示給使用者看的題目文字。([hl7.org][1])
* `type`（必填）：`group | display | boolean | decimal | integer | date | dateTime | time | string | text | url | choice | open-choice | attachment | reference | quantity` 等（頁面列出）。用來決定答案型別與可用欄位。([hl7.org][1])

### 題目顯示/啟用邏輯

* `enableWhen[]`：**條件成立才顯示/允許作答**（可多筆）。每筆包含：([hl7.org][1])

  * `question`：拿哪一題（用 linkId）來判斷
  * `operator`：`exists | = | != | > | < | >= | <=`
  * `answer[x]`：用來比對的值（多型別：boolean/decimal/integer/date/dateTime/time/string/coding/quantity/reference…）
  * 規則：若 `operator` 是 `exists`，則 `answer[x]` 必須是 boolean。([hl7.org][1])
* `enableBehavior`：當你有多個 `enableWhen` 時，指定 `all | any`（全部成立 or 任一成立）。**若 `enableWhen` 超過 1 個，必須填 enableBehavior**（規則）。([hl7.org][1])

### 作答限制與 UI 行為

* `required`：此題是否必填（注意：`display` 不允許 required）。([hl7.org][1])
* `repeats`：此題是否可重複（例如可多選、多筆輸入）。([hl7.org][1])
* `readOnly`：是否唯讀（通常用在「系統帶入、使用者不可改」；`display` 不允許）。([hl7.org][1])
* `maxLength`：字串最大長度（規則：只適用簡單題型）。([hl7.org][1])

### 允許的答案（choice/open-choice）

* `answerValueSet`：允許答案來自某個 ValueSet（canonical）。規則：**只有 `choice` / `open-choice` 可用**。([hl7.org][1])
* `answerOption[]`：直接列舉允許答案。每個 option 有：([hl7.org][1])

  * `value[x]`：答案值（integer/date/time/string/coding/reference…）
  * `initialSelected`：是否預設選取
* 規則：**`answerOption` 與 `answerValueSet` 不能同時存在**。([hl7.org][1])

### 初始值與巢狀題目

* `initial[]`：題目第一次呈現時的初始值（多型別 value[x]）。規則：

  * 群組(group)與顯示(display)不能設 initial
  * 若有 `answerOption`，則 `initial[x]` 必須不存在
  * 只有 repeating 題目才能有多個 initial（多筆）([hl7.org][1])
* `item[]`：巢狀子題目（建立群組/子題、或用 enableWhen 做條件題）。同一套結構遞迴套用。([hl7.org][1])

---

## 規格裡明講的幾個「容易踩雷」規則（你設計問卷邏輯時超常遇到）

* `group` 類型必須有巢狀 `item`；`display` 類型不能有巢狀 `item`。([hl7.org][1])
* `display` 類型：不能有 `code`、不能 `required/repeats`、不能 `readOnly`、不能 `initial`。([hl7.org][1])
* `answerOption` vs `answerValueSet` 二選一；且只有 `choice/open-choice` 能用 `answerValueSet`。([hl7.org][1])
* 多個 `enableWhen` 時一定要填 `enableBehavior`。([hl7.org][1])

---

## 一、為什麼 Questionnaire 需要 Profiles 與 Extensions？

FHIR 的設計哲學是：

> **核心規格只定「最小公約數」，實務需求用 Profile / Extension 補齊**

對 Questionnaire 而言，核心資源**刻意不做太多 UI / 行為限制**，例如：

* 題目要不要「即時計算」
* 題目要不要「從既有資料自動帶入」
* 題目要不要「隱藏但仍送出答案」
* 問卷要不要「符合某個國家 / 計畫 / 平台規範」

這些就交給：

* **Profile**：限制、收斂、標準化「怎麼用 Questionnaire」
* **Extension**：補上核心模型沒有的能力

---

## 二、Profiles（設定「這種問卷該長怎樣」）

### 1️⃣ Profile 是什麼？

Profile 是一個 **StructureDefinition**，用來：

* 限制欄位「必填 / 不可用」
* 限制欄位的 **Cardinality**
* 限制 `type`、`value[x]` 可用的型別
* 指定 **必須使用哪些 Extension**

在 Questionnaire 領域，Profile 常用來定義：
-「這是 **SDC 問卷**」
-「這是 **國家級標準問卷**」
-「這是 **只能給 Patient 填的問卷**」

---

### 2️⃣ Questionnaire 常見 Profile 範例（規格頁面重點）

在你提供的頁面中，官方特別點出 **SDC（Structured Data Capture）Profiles**，這是 Questionnaire 最重要的一組 Profile。

#### 🔹 SDC 問卷 Profile（重點）

用途：

> 讓問卷可以「結構化、可驗證、可抽取成臨床資料」

常見限制包括：

* 必須有 `item.linkId`
* 必須定義清楚 `type`
* 問卷可被系統解析、轉換為 Observation / Condition

👉 實務上你在做：

* 化療副作用量表
* 病人自填 PROMs
* 結構化評估表

**幾乎一定要用 SDC Profile**

---

### 3️⃣ Profile 實際怎麼套在 Questionnaire？

靠的是 **meta.profile**

```json
{
  "resourceType": "Questionnaire",
  "meta": {
    "profile": [
      "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire"
    ]
  }
}
```

📌 意義是：

> 「我這份 Questionnaire **承諾** 符合這個 Profile 的所有規則」

FHIR Server / Validator 會：

* 驗證你有沒有違反規範
* 下游系統可根據 profile 判斷「能不能吃這份問卷」

---

## 三、Extensions（補上 Questionnaire 原本沒有的能力）

### 1️⃣ Extension 是什麼？

Extension 是 **標準化的「加欄位機制」**，用來：

* 不破壞核心模型
* 但又能精準補上行為、語意、UI 控制

Extension 有：

* **官方定義**
* **Implementation Guide（IG）定義**
* **你自己定義（院內 / 專案）**

---

### 2️⃣ Questionnaire 常見 Extension 類型（規格頁面重點）

以下是 **在 questionnaire-profiles.html 中被明確提到、而且實務超常用的類型**：

---

#### 🔹 問卷層級 Extension

用在 `Questionnaire.extension`

用途包含：

* 問卷分類（例如：副作用 / 生活品質）
* 問卷填寫說明
* 問卷適用流程階段（入院 / 化療中 / 出院）

📌 實務意義
👉 你之前問的「**哪些病人要填哪些問卷**」，**90% 就靠這一層 Extension**

---

#### 🔹 題目層級 Extension（最重要）

用在 `Questionnaire.item.extension`

常見能力：

##### ✅ UI 行為控制

* 隱藏題目（但仍送出）
* 顯示提示文字（help / tooltip）
* 控制呈現方式（slider / checkbox / dropdown）

##### ✅ 資料綁定 / 抽取

* 對應 Observation.code
* 指定答案要存成哪一種 Resource
* 設定「自動帶入既有病歷值」

📌 **核心概念**

> enableWhen 決定「顯不顯示」
> extension 決定「怎麼顯示 / 怎麼用」

---

### 3️⃣ Extension 實際長怎樣？

基本結構永遠是：

```json
"extension": [
  {
    "url": "extension 的 canonical URL",
    "value[x]": "實際值"
  }
]
```

📌 重點：

* `url` = Extension 的「身分證」
* `value[x]` = 真正的資料（型別固定）

---

### 4️⃣ 官方 vs 自訂 Extension 怎麼選？

**原則很簡單：**

1️⃣ 先找

* HL7 官方
* SDC IG
* 國家 IG（例如 TW Core）

2️⃣ 找不到才自己定義

* 但要給 **穩定 canonical URL**
* 文件一定要寫清楚語意

👉 你現在做的「癌症 / 化療 / 副作用」問卷
**非常適合定義「院級 Extension + 未來升級國家 IG」**

---

## 四、Profile × Extension 的分工總結（這張很重要）

| 功能         | 用 Profile | 用 Extension |
| ---------- | --------- | ----------- |
| 限制哪些欄位能用   | ✅         | ❌           |
| 指定必填欄位     | ✅         | ❌           |
| 定義問卷「類型」   | ✅         | ❌           |
| 加入 UI 行為   | ❌         | ✅           |
| 加入條件顯示邏輯   | ❌         | ✅（補強）       |
| 問卷分類 / 標籤  | ❌         | ✅           |
| 把答案對應到臨床資料 | ❌         | ✅           |

一句話記憶法：

> **Profile 管規則，Extension 管行為**

