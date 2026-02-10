# FHIR Organization Resource 資源結構說明

## 從醫院、科室到組織階層，該如何正確建模？

在導入 **FHIR（Fast Healthcare Interoperability Resources）** 的過程中，許多醫院資訊人員與系統開發者，第一個會卡住的問題往往不是 Observation 或 Encounter，而是一個看似簡單卻非常關鍵的資源：

> **醫院、科室、單位，在 FHIR 裡到底要怎麼表示？**

答案就是 **Organization**。

本文將以 **FHIR R4** 為基礎，從實務角度完整說明：

* Organization 是什麼
* 在醫院裡通常怎麼用
* Organization 與 Location 的差異
* Organization.type 與 CodeableConcept 的正確用法
* 如何用 Organization 建立「醫院 → 科室 → 團隊」的組織階層架構

---

## 一、什麼是 Organization？

在 FHIR 中，**Organization 用來描述一個被認可的組織單位（Formal Organization）**。

它可以代表：

* 一家醫院或醫療法人
* 醫院底下的科室（內科、外科、放射科）
* 功能型單位或醫療團隊（安寧團隊、癌症照護團隊）
* 行政單位、研究中心、資訊室，甚至政府機構

👉 **重點不是「是不是醫療人員」**，而是「這是不是一個正式存在、可被引用的組織單位」。

---

## 二、Organization 在醫院場景中通常用來表示什麼？

在實際醫療系統中，Organization 最常出現在以下層級：

### 1️⃣ 院級（Hospital / Medical Center）

* 醫學中心
* 區域醫院
* 分院

### 2️⃣ 科室 / 部門（Department）

* 內科、外科、放射科
* 檢驗科（LIS）
* 護理部、資訊室

### 3️⃣ 團隊 / 功能單位

* 安寧團隊
* 癌症照護團隊
* 臨床創新中心（CIC）

這些都非常適合用 **Organization** 來建模。

---

## 三、Organization ≠ Location（非常重要的觀念）

這是導入 FHIR 時**最常見的誤解之一**。

| 類型        | 正確資源              | 說明       |
| --------- | ----------------- | -------- |
| 醫院、科室、單位  | Organization      | 組織與隸屬關係  |
| 病房、診間、檢查室 | Location          | 實體空間     |
| 可提供的服務項目  | HealthcareService | 門診、檢查、治療 |

可以這樣記：

> **Organization 管「誰」
> Location 管「在哪裡」
> HealthcareService 管「提供什麼服務」**

把這三者拆清楚，後續在查詢、權限、統計與視覺化上會非常乾淨。

---

## 四、Organization 的核心欄位（實務必懂）

### 1️⃣ identifier：跨系統識別碼（強烈建議使用）

```json
"identifier": [{
  "system": "https://hospital.example.org/org-code",
  "value": "HOSP-001"
}]
```

* 放院內代碼、科室代碼
* 是跨系統對齊的關鍵
* 比 `id` 更穩定（id 通常由伺服器產生）

---

### 2️⃣ name / alias：名稱與別名

```json
"name": "國立成功大學醫學院附設醫院",
"alias": ["成大醫院", "NCKUH"]
```

* `name`：正式名稱
* `alias`：簡稱、英文名

---

### 3️⃣ type：這個組織是「什麼類型」？

`Organization.type` 是 **CodeableConcept**，不是單純字串。

```json
"type": [{
  "coding": [{
    "system": "http://terminology.hl7.org/CodeSystem/organization-type",
    "code": "dept",
    "display": "Hospital Department"
  }],
  "text": "放射科"
}]
```

這代表：

* 在國際標準中，它是一個「部門（department）」
* 在人類閱讀時，顯示為「放射科」

⚠️ 注意一個關鍵觀念：

> **type 是「組織類型」
> name 才是「科別名稱」**

---

## 五、什麼是 CodeableConcept？為什麼這麼重要？

**CodeableConcept 是 FHIR 用來表示「一個概念，可以同時對應多個代碼體系」的設計。**

它通常包含：

* 一或多個 `coding`（給系統判斷與搜尋）
* 一個 `text`（給人類閱讀）

實務上常見用法是：

* 一個 **國際標準 code**（HL7 / SNOMED / LOINC）
* 一個 **院內或在地 code**
* 一個 **中文顯示文字**

這讓 FHIR 能同時滿足：

* 跨院 / 跨國交換
* 系統自動判斷
* 人類友善顯示

---

## 六、如何用 Organization 定義「組織階層架構」？

在真實醫院中，組織一定是有層級的：

* 醫院
  └── 科室
  　　　└── 團隊

FHIR 並沒有額外的階層資源，而是用 **Organization.partOf** 來描述隸屬關係。

### 🔑 `partOf` 的設計重點

* 子組織在自己的 `partOf` 指向父組織
* 階層是「自下而上」定義
* 不限制層數

---

### 範例：建立完整的醫院組織階層

#### 🏥 院級 Organization

```json
{
  "resourceType": "Organization",
  "id": "hospital-001",
  "name": "成大醫院",
  "type": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/organization-type",
      "code": "prov"
    }]
  }]
}
```

#### 🩺 科室（隸屬醫院）

```json
{
  "resourceType": "Organization",
  "id": "dept-im",
  "name": "內科",
  "type": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/organization-type",
      "code": "dept"
    }]
  }],
  "partOf": {
    "reference": "Organization/hospital-001"
  }
}
```

#### 👥 團隊（隸屬科室）

```json
{
  "resourceType": "Organization",
  "id": "team-nephrology",
  "name": "腎臟科照護團隊",
  "type": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/organization-type",
      "code": "team"
    }]
  }],
  "partOf": {
    "reference": "Organization/dept-im"
  }
}
```

## 七、Organization 會被哪些資源引用？

Organization 是「全系統共用的基礎資源」，常被以下資源引用：

* `Encounter.serviceProvider`：這次就診在哪個醫院
* `PractitionerRole.organization`：醫師隸屬哪個單位
* `Location.managingOrganization`：空間由哪個組織管理
* `HealthcareService.providedBy`：服務由哪個單位提供

👉 **只要 Organization 建得好，整個醫療資料就能自然串起來。**

---

## 八、實務常見錯誤（請避免）

❌ 把科室當成 Location
❌ 不使用 `partOf`，所有 Organization 平鋪
❌ 用文字描述階層而非結構關係
❌ 在 `type` 放科別名稱

這些錯誤在小系統也許勉強可行，但在 **FHIR 中台 / 跨院 / SMART on FHIR** 場景一定會出問題。

---

## 結語：Organization 是醫療資料世界的「骨架」

**Organization 看似簡單，卻是整個 FHIR 架構的骨架。**

當組織結構定義清楚：

* Encounter 才知道屬於哪個單位
* Observation 才能依科別分析
* 權限控管與統計才有依據
* SMART on FHIR 與 AI 應用才有正確上下文

在導入 FHIR 的第一步，
**把 Organization 與組織階層設計好，永遠是最值得投資的工作之一。**



