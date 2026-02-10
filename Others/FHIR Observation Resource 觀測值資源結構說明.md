# FHIR Observation Resource 觀測值資源結構說明

[https://hl7.org/fhir/R4/observation.html](https://hl7.org/fhir/R4/observation.html)

# FHIR R4 Observation 資源欄位完整說明（含 Profiles / Extensions）

以下內容依據 HL7 FHIR R4（v4.0.1）的 **Observation** 規格頁與其 **Profiles & Extensions** 彙整頁整理。([hl7.org][1])

---

## 1) Observation 的定位與常見用途

**Observation** 用來表達「被觀察到的事實」：包含檢驗、生命徵象、影像量測結果、問卷量測值、裝置量測值等，通常以「一個 code + 一個 value（或缺值原因）」為核心，再搭配時間、對象、方法、檢體、參考區間等語意。([hl7.org][1])

---

## 2) 共同基底欄位（Resource / DomainResource 來源）

Observation 繼承自 Resource 與 DomainResource，因此除了 Observation 自己的欄位外，也「天然擁有」這些基底欄位：([hl7.org][1])

### 2.1 Resource 層級

* `id`

  * 資源在伺服器上的邏輯 id（常見於 URL 的 `/Observation/{id}`）。
* `meta`

  * 版本、最後更新時間、標記符合哪些 profile（`meta.profile`）等。
* `implicitRules`

  * 若此 instance 依賴某些未明示的規則才可正確理解，可在此指出。
* `language`

  * 敘述性內容（如 `text`）的語言。

### 2.2 DomainResource 層級

* `text`

  * 人類可讀摘要（Narrative）。
* `contained`

  * 內嵌資源（不建議濫用；通常用 Reference 連結較佳）。
* `extension`

  * **非破壞性擴充**（不改變原本語意，只補充額外資訊）。
* `modifierExtension`

  * **會改變語意 / 不能忽略** 的擴充（例如「否定」、「修飾」等）；處理系統必須特別注意（詳見第 6 節）。([hl7.org][2])

---

## 3) Observation 主體欄位（逐一解釋）

> 下列每個欄位都列出：**意義**、**用法**、**注意事項**。欄位清單與基礎定義來自 Observation 結構與範本。([hl7.org][1])

### `identifier`（0..*）

* **意義**：業務識別碼（例如 LIS 檢體報告號、量測單號）。([hl7.org][1])
* **用法**：用於跨系統對帳、避免只靠 `id`（伺服器內部 id）做整合。
* **注意**：可多筆（不同系統各自的識別碼）。

### `basedOn`（0..*）

* **意義**：此 Observation 用來履行的「計畫/醫囑/請求」。([hl7.org][1])
* **用法**：常指向 `ServiceRequest`（檢驗/檢查醫囑）、`MedicationRequest` 等。
* **注意**：同一檢驗結果可能同時對應多個請求（例如合併下單）。

### `partOf`（0..*）

* **意義**：此 Observation 是某事件的一部分（例如某次處置、用藥給予、影像檢查流程）。([hl7.org][1])
* **用法**：可指向 `Procedure`、`ImagingStudy`、`MedicationAdministration` 等。
* **注意**：用於表達「結果隸屬於某個已發生事件」。

### `status`（1..1）

* **意義**：Observation 的狀態（registered / preliminary / final / amended…）。([hl7.org][1])
* **用法**：臨床通常最重視 `final`（已確認）。
* **注意**：必填；也是搜尋/流程判斷的關鍵欄位。

### `category`（0..*）

* **意義**：高階分類（例如 laboratory / vital-signs / imaging…）。([hl7.org][1])
* **用法**：讓 UI 或查詢能快速分群（例如「先抓生命徵象」）。
* **注意**：可多值（同一筆 Observation 可同時落在不同分類視角）。

### `code`（1..1）

* **意義**：**觀察項目是什麼**（觀察的名稱/代碼），例如「血糖」、「體溫」。([hl7.org][1])
* **用法**：通常用標準術語（常見是 LOINC），也可加多個 coding 表示更細緻或不同視角的同義碼。([hl7.org][1])
* **注意**：

  * `code` 定義了 Observation 的語意核心；很多時候「測哪裡」其實已經隱含在 code（例如 Blood Glucose）。([hl7.org][1])

### `subject`（0..1）

* **意義**：Observation 關於誰/什麼（常見：病人）。([hl7.org][1])
* **用法**：通常是 `Patient`，也可能是 `Group`、`Device`、`Location`。
* **注意**：有些量測不是「病人層級」而是設備或地點層級。

### `focus`（0..*）

* **意義**：當「觀察焦點」不是 `subject` 本人時，用 `focus` 指出真正被關注的對象。([hl7.org][1])
* **用法**：例如 subject=Patient，但 focus=植入裝置、某個檢體、或另一個 Observation。
* **注意**：與 `specimen`、`bodySite` 一起用來精準表達觀察焦點差異。([hl7.org][1])

### `encounter`（0..1）

* **意義**：量測發生所屬的就醫事件（門診/急診/住院的一次 Encounter）。([hl7.org][1])
* **用法**：做時序圖、就醫分段、或住院期間趨勢很重要。
* **注意**：不是所有 Observation 都一定能對應到 Encounter（例如居家裝置回傳）。

### `effective[x]`（0..1）

* **意義**：**臨床上最相關的時間/期間**（physiologically relevant time）。([hl7.org][1])
* **型別**：`dateTime` / `Period` / `Timing` / `instant`。([hl7.org][1])
* **用法**：

  * 抽血：通常是採檢時間（或採檢期間）。
  * 24 小時尿液：用 `Period` 表示收集起訖。([hl7.org][1])
* **注意**：這個時間通常比 `issued` 更能代表「這個數值對病人的意義」。

### `issued`（0..1）

* **意義**：此版本 Observation **被發布/可取得** 的時間點。([hl7.org][1])
* **用法**：例如 LIS 驗證後釋出結果時間。
* **注意**：與 `effective[x]` 不同：effective 是「量測對病人的生理/臨床時間」，issued 是「系統釋出時間」。([hl7.org][1])

### `performer`（0..*）

* **意義**：對此 Observation 負責/執行的人或組織。([hl7.org][1])
* **用法**：可指向 Practitioner / Organization / CareTeam 等。
* **注意**：檢驗可能是科室或實驗室（Organization），生命徵象可能是護理人員（PractitionerRole）。

---

## 4) 結果值的表達：`value[x]` 與缺值原因

### `value[x]`（0..1）

* **意義**：Observation 的「結果」。([hl7.org][1])
* **允許型別**（會影響 JSON 欄位名）：

  * `valueQuantity`, `valueCodeableConcept`, `valueString`, `valueBoolean`, `valueInteger`, `valueRange`, `valueRatio`, `valueSampledData`, `valueTime`, `valueDateTime`, `valuePeriod`。([hl7.org][1])
* **用法要點**：

  * 量化數值（有單位）用 `Quantity`（例：mmHg、mg/dL）。
  * 分類/判讀（例：Positive/Negative、+/-、分級）常用 `CodeableConcept`。
* **注意（很關鍵）**：`valueBoolean` 在實務上很少用，因為多數「是/否」都有例外值如 unknown；規格建議改用 `CodeableConcept` 來表達 yes/no/unknown 等更完整語意。([hl7.org][1])

### `dataAbsentReason`（0..1）

* **意義**：當「預期應有 value 但沒有」時，說明為何缺值（unknown / asked-unknown / not-performed / error / NaN…）。([hl7.org][1])
* **強制規則（Constraint: obs-6）**：**只有在 `value[x]` 不存在時才允許出現**。([hl7.org][1])

### `interpretation`（0..*）

* **意義**：對結果的分類判讀（高/低/正常/危急…）。([hl7.org][1])
* **用法**：常見配合 `referenceRange` 或院內規則產出。
* **注意**：可多筆（不同判讀系統或不同層次判讀）。

### `note`（0..*）

* **意義**：備註（Annotation）。([hl7.org][1])
* **用法**：如「溶血」「檢體不足」「重測」或臨床解讀文字。
* **注意**：這是自由文字；若要可計算/可查詢，通常要另用結構化欄位或 extension/profile 約束。

---

## 5) 觀察的上下文：部位、方法、檢體、設備

### `bodySite`（0..1）

* **意義**：觀察的身體部位。([hl7.org][1])
* **用法**：例如左右手、特定肌群、特定器官部位。
* **注意**：當部位並未由 `code` 足夠描述時才特別需要。

### `method`（0..1）

* **意義**：量測方法。([hl7.org][1])
* **用法**：例如「pulse oximetry」、「酵素法」等。
* **注意**：方法不同可能導致可比性差，若要做趨勢/分析很重要。

### `specimen`（0..1）

* **意義**：檢體。([hl7.org][1])
* **用法**：血液、尿液、組織等，通常指向 Specimen 資源。
* **注意**：與 `effective[x]` 常一起解釋（採檢時間 vs 報告釋出時間）。([hl7.org][1])

### `device`（0..1）

* **意義**：產生量測資料的裝置。([hl7.org][1])
* **用法**：床邊監視器、居家量測設備等。
* **注意**：跨設備趨勢比較、校正與可信度判斷會用到。

---

## 6) 參考區間：`referenceRange`（0..*）與其子欄位

### `referenceRange`（0..*）

* **意義**：提供解讀 value 的參考範圍（可依族群、年齡、條件不同而多筆）。([hl7.org][1])
* **注意（Constraint: obs-3）**：每個 referenceRange **至少要有** `low` 或 `high` 或 `text` 其中之一。([hl7.org][1])

#### `referenceRange.low`（0..1）

* **意義**：低界（Quantity）。([hl7.org][1])

#### `referenceRange.high`（0..1）

* **意義**：高界（Quantity）。([hl7.org][1])

#### `referenceRange.type`（0..1）

* **意義**：此參考區間的「意義/種類」（例如 normal range、therapeutic range…）。([hl7.org][1])

#### `referenceRange.appliesTo`（0..*）

* **意義**：適用族群（例如性別、族裔、孕期狀態等）。([hl7.org][1])

#### `referenceRange.age`（0..1）

* **意義**：適用年齡範圍（Range）。([hl7.org][1])

#### `referenceRange.text`（0..1）

* **意義**：無法用量化 low/high 表達時，用文字表達參考值（例如「Negative」、「見表」）。([hl7.org][1])

---

## 7) 觀察的群組與衍生關係：`hasMember` / `derivedFrom` / `component`

### `hasMember`（0..*）

* **意義**：把多個 Observation（或 QuestionnaireResponse、MolecularSequence）聚成一組的關聯。([hl7.org][1])
* **用法**：適合「一組內的成員可獨立理解/可獨立使用」的情境（相對於 component）。
* **注意**：規格明確把 `hasMember` 視為 Observation grouping 的一種手段。([hl7.org][1])

### `derivedFrom`（0..*）

* **意義**：此 Observation 由哪些資料/測量衍生而來（DocumentReference、ImagingStudy、Media、QuestionnaireResponse、Observation…）。([hl7.org][1])
* **用法**：例如 AI 從影像推論出的身體組成結果，可用 derivedFrom 指回影像或原始量測。
* **注意**：也是 Observation grouping/關聯的重要元素之一。([hl7.org][1])

### `component`（0..*）

* **意義**：同一個 Observation 內的「組成結果」（血壓：收縮壓/舒張壓；基因檢測：多個 component）。([hl7.org][1])
* **使用指引（重點）**：component 適用於「離開母 Observation 就很難被合理解讀/使用」的子結果。([hl7.org][1])
* **強制規則（Constraint: obs-7）**：如果 `Observation.code` 與某個 `Observation.component.code` 相同，則 **母 Observation 對應該 code 的 `value[x]` 不得出現**（避免重複/衝突）。([hl7.org][1])

#### `component.code`（1..1）

* **意義**：該 component 的代碼/名稱。([hl7.org][1])

#### `component.value[x]`（0..1）

* **意義**：該 component 的結果值（允許型別與 `value[x]` 類似）。([hl7.org][1])

#### `component.dataAbsentReason`（0..1）

* **意義**：component 缺值原因。([hl7.org][1])

#### `component.interpretation`（0..*）

* **意義**：component 的高低/正常等判讀。([hl7.org][1])

#### `component.referenceRange`（0..*）

* **意義**：component 的參考區間（內容結構與 `Observation.referenceRange` 相同）。([hl7.org][1])

---

## 8) Profiles：為何需要、怎麼用、Observation 內建有哪些

### 8.1 Profiles 是什麼（為何存在）

FHIR 是「平台規格」，不同國家/機構/領域需要再做在地化與情境化約束；這些約束通常以 **Implementation Guide / Profile** 等一致性工件呈現。([hl7.org][3])

### 8.2 Profile 用什麼資源表示

* Profile 本質上是 **StructureDefinition（kind = constraint）**，用來：

  * 限縮/要求欄位是否可用、卡片數（cardinality）、固定值、術語綁定、參照目標 profile 等。([hl7.org][3])

### 8.3 宣告「此資源符合某 profile」

* 生產者通常會用 `meta.profile` 放入該 profile 的 canonical URL（讓消費者可依 profile 辨識/索引）。規格也說明：若宣告支援某 supported profile，生產者**應標記資源具備 profile 斷言**，伺服器端也**應支援用 `_profile` 搜尋**。([hl7.org][3])

> 範例（示意；canonical URL 請以實際 profile 定義為準）：

```json
{
  "resourceType": "Observation",
  "meta": {
    "profile": [
      "http://example.org/fhir/StructureDefinition/my-observation-profile"
    ]
  }
}
```

### 8.4 Observation 在 R4 內建的 Profiles（節錄清單）

Observation 的 Profiles & Extensions 彙整頁列出多個常用 profile，包含 Vital Signs 與特定生命徵象（BMI、血壓、體溫、身高、體重、頭圍、心率等），以及 Vital Signs Panel 等。([hl7.org][4])

---

## 9) Extensions：規則、表示法、Observation 相關延伸

### 9.1 Extension 的基本規則（所有資源通用）

* 每個元素都可以帶 `extension`（0..*）。([hl7.org][2])
* 每個 extension **必須有** `url`（1..1）用來識別其語意，且 `url` 應是定義該 extension 的 StructureDefinition canonical URL。([hl7.org][2])
* extension **只能**「有 value」或「有子 extension」二選一，不能同時存在。([hl7.org][2])
* 若「不能安全忽略」某 extension（會改變既有語意），必須用 `modifierExtension` 表示，而不是一般 `extension`。([hl7.org][2])

> 範例（取自規格頁示意）：

```json
{
  "extension": [
    {
      "url": "http://hl7.org/fhir/StructureDefinition/iso-21090-EN-use",
      "valueCode": "I"
    }
  ]
}
```

([hl7.org][2])

### 9.2 Observation 專屬（或常用於 Observation）的 Extensions 清單（R4 規格彙整頁）

Observation 的 Profiles & Extensions 頁列出了多種延伸（含事件樣式、工作流程樣式、基因檢測相關等）。以下是該頁面列出的主要 Observation extensions（逐條照頁面列出，並用一句話說明它通常在幹嘛）：([hl7.org][4])

* `diagnosticReport-risk`：把風險資訊與 DiagnosticReport 關聯的延伸用法。([hl7.org][4])
* `event-eventHistory`：事件歷程（event pattern）。([hl7.org][4])
* `event-location`：事件地點（event pattern）。([hl7.org][4])
* `event-performerFunction`：在 `Observation.performer` 上補「角色/功能」。([hl7.org][4])
* `event-statusReason`：狀態原因（event pattern）。([hl7.org][4])
* `observation-bodyPosition`：量測時的身體姿勢（如臥、坐、站）。([hl7.org][4])
* `observation-delta`：呈現變化量/差值（delta）。([hl7.org][4])
* `observation-deviceCode`：補充設備代碼（不只 device reference）。([hl7.org][4])
* `observation-focusCode`：補充 focus 的代碼資訊。([hl7.org][4])
* `observation-gatewayDevice`：資料經由哪個 gateway 裝置上傳/轉送。([hl7.org][4])
* （基因檢測相關）`observation-geneticsAllele` / `…AminoAcidChange` / `…Ancestry` / `…CopyNumberEvent` / `…DNARegionName` / `…Gene` / `…GenomicSourceClass` / `…Interpretation` / `…PhaseSet` / `…Variant`：用於結構化表達基因檢測觀察結果。([hl7.org][4])
* `observation-precondition`：前提條件（例如量測前狀態）。([hl7.org][4])
* `observation-reagent`：試劑資訊。([hl7.org][4])
* `observation-replaces`：此 Observation 取代了哪一筆（替換關係）。([hl7.org][4])
* `observation-secondaryFinding`：次要發現（secondary finding）。([hl7.org][4])
* `observation-sequelTo`：前後續（sequelTo）關係。([hl7.org][4])
* `observation-specimenCode`：補充 specimen 的代碼資訊。([hl7.org][4])
* `observation-timeOffset`：在 `Observation.component` 上補時間偏移（例如波形/序列點偏移）。([hl7.org][4])
* （workflow pattern）`workflow-episodeOfCare` / `workflow-instantiatesCanonical` / `workflow-instantiatesUri` / `workflow-reasonCode` / `workflow-reasonReference` / `workflow-relatedArtifact` / `workflow-researchStudy` / `workflow-supportingInfo`：把 Observation 放進照護流程/研究流程脈絡的延伸。([hl7.org][4])

---

## 10) 實作時最常踩雷的 3 個規則（建議你在驗證器/單元測試先擋）

1. **`dataAbsentReason` 不可跟 `value[x]` 同時出現**（obs-6）。([hl7.org][1])
2. **`referenceRange` 至少要有 low/high/text 其一**（obs-3）。([hl7.org][1])
3. **`Observation.code` 若與某個 `component.code` 相同，母 Observation 不得再給 `value[x]`**（obs-7）。([hl7.org][1])

[1]: https://hl7.org/fhir/R4/observation.html "Observation - FHIR v4.0.1"
[2]: https://hl7.org/fhir/R4/extensibility.html "Extensibility - FHIR v4.0.1"
[3]: https://hl7.org/fhir/R4/profiling.html "Profiling - FHIR v4.0.1"
[4]: https://hl7.org/fhir/R4/observation-profiles.html "Observation - FHIR v4.0.1"



