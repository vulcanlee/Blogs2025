# 核心臨床時序圖（FHIR R4）關聯矩陣

 **《FHIR 核心臨床時序圖－關聯矩陣（R4）》**，定位就是：

> 👉 **以「病人為主軸（Patient-centric）」
> 👉 可支援：病人時序圖 / Cohort Discovery / 檢驗追蹤 / 癌症與長期療程 / 研究資料可回溯
> 👉 明確標示：哪些關聯「必須有」、哪些「強烈建議」、哪些「可選」**

---

# 🧭 核心臨床時序圖（FHIR R4）關聯矩陣

### 標示說明

* **M（Must）**：強制關聯（實務上 *一定要有*，建議 Profile 加嚴為 `1..1`）
* **S（Should）**：強烈建議（沒有會嚴重影響時序/追蹤/稽核）
* **O（Optional）**：可選（依專案需求）
* **—**：不適用

---

## 一、時序圖「主軸層」資源（骨架）

| Resource          | Patient | Encounter | EpisodeOfCare | Practitioner / Role | Organization | 說明            |
| ----------------- | ------- | --------- | ------------- | ------------------- | ------------ | ------------- |
| **Patient**       | —       | —         | —             | —                   | —            | 時序圖的「絕對主體」    |
| **Encounter**     | **M**   | —         | S             | S                   | S            | 每一次就醫事件       |
| **EpisodeOfCare** | **M**   | S         | —             | S                   | S            | 長期療程主線（癌症/慢病） |
| **CareTeam**      | **M**   | S         | S             | **M**               | S            | 照護團隊軸線        |
| **CarePlan**      | **M**   | S         | **S**         | S                   | S            | 照護計畫（跨時間）     |

📌 **設計重點**

* 👉 **Patient 是絕對錨點**
* 👉 **Encounter = 時序切片**
* 👉 **EpisodeOfCare = 長期縱軸（非常適合你現在做的癌症/安寧）**

---

## 二、檢驗 / 抽血 / 影像（你目前最常用）

| Resource             | Patient | Encounter | ServiceRequest | Specimen | DiagnosticReport | 說明             |
| -------------------- | ------- | --------- | -------------- | -------- | ---------------- | -------------- |
| **ServiceRequest**   | **M**   | **M**     | —              | O        | O                | 開單（抽血/影像/檢查）   |
| **Specimen**         | **M**   | **S**     | **S**          | —        | O                | 檢體（血液/組織）      |
| **Observation**      | **M**   | **M**     | S              | S        | O                | 單一檢驗結果         |
| **DiagnosticReport** | **M**   | **M**     | S              | O        | —                | 報告彙總           |
| **ImagingStudy**     | **M**   | **M**     | S              | —        | O                | DICOM Study 索引 |

📌 **強烈建議的「標準鏈」**

```
ServiceRequest
   ↓
Specimen
   ↓
Observation (多筆)
   ↓
DiagnosticReport
```

👉 這一條 **就是你「抽血項目異動 / 缺漏追蹤」的根本解法**

---

## 三、用藥時序鏈（醫囑 → 調劑 → 給藥）

| Resource                     | Patient | Encounter | MedicationRequest | Medication | Practitioner | 說明     |
| ---------------------------- | ------- | --------- | ----------------- | ---------- | ------------ | ------ |
| **MedicationRequest**        | **M**   | **M**     | —                 | **M**      | **M**        | 醫師開立處方 |
| **MedicationDispense**       | **M**   | **S**     | **M**             | **M**      | S            | 藥局調劑   |
| **MedicationAdministration** | **M**   | **M**     | **M**             | **M**      | **M**        | 實際給藥   |
| **MedicationStatement**      | **M**   | O         | O                 | **M**      | O            | 病人用藥整合 |

📌 **關鍵原則**

* 不要只留 `MedicationStatement`
* 👉 **醫囑（Request）與執行（Administration）一定要分開**

---

## 四、診斷 / 問題 / 評估（臨床判斷）

| Resource               | Patient | Encounter | Practitioner | Evidence (Obs/Proc) | 說明        |
| ---------------------- | ------- | --------- | ------------ | ------------------- | --------- |
| **Condition**          | **M**   | S         | **M**        | S                   | 診斷 / 問題清單 |
| **ClinicalImpression** | **M**   | **M**     | **M**        | **M**               | 臨床評估摘要    |
| **RiskAssessment**     | **M**   | **M**     | S            | **M**               | 風險評估      |
| **DetectedIssue**      | **M**   | **M**     | S            | **M**               | 系統/人工發現問題 |

📌 **實務建議**

* Condition ≠ 推論
* 👉 **ClinicalImpression / RiskAssessment 才是 AI/推論結果的好位置**

---

## 五、處置 / 手術 / 行為

| Resource               | Patient | Encounter | Performer | basedOn | 說明    |
| ---------------------- | ------- | --------- | --------- | ------- | ----- |
| **Procedure**          | **M**   | **M**     | **M**     | S       | 手術/治療 |
| **DeviceUseStatement** | **M**   | S         | S         | O       | 裝置使用  |
| **DeviceRequest**      | **M**   | **M**     | **M**     | —       | 裝置申請  |

---

## 六、文件 / 影像 / 報告（大型內容）

| Resource              | Patient | Encounter | Author | Subject Resource | 說明           |
| --------------------- | ------- | --------- | ------ | ---------------- | ------------ |
| **DocumentReference** | **M**   | **S**     | **M**  | **M**            | 文件索引（PDF/影像） |
| **Composition**       | **M**   | **M**     | **M**  | **M**            | 結構化臨床文件      |
| **Media**             | **M**   | S         | O      | **M**            | 影像/照片        |
| **Binary**            | —       | —         | —      | —                | 不建議大量使用      |

📌 **重要實務原則**

> ❌ 不要把 AI 圖、PDF 全塞 Binary
> ✅ 用 `DocumentReference + 外部儲存`

---

## 七、研究 / AI / 次級利用（你未來一定會用）

| Resource                    | Patient | EpisodeOfCare | Encounter | Provenance | 說明      |
| --------------------------- | ------- | ------------- | --------- | ---------- | ------- |
| **ResearchSubject**         | **M**   | S             | O         | **M**      | 病人參與研究  |
| **Observation (AI Result)** | **M**   | S             | **M**     | **M**      | AI 推論結果 |
| **QuestionnaireResponse**   | **M**   | **M**         | —         | **M**      | PRO/問卷  |
| **MeasureReport**           | **M**   | —             | —         | **M**      | 指標計算    |

👉 **AI 推論結果一定要留 Provenance**
（模型版本、時間、來源影像）

---

## 八、治理 / 稽核 / 法遵（上線必備）

| Resource               | Target Resource | Patient | Practitioner | Organization | 說明        |
| ---------------------- | --------------- | ------- | ------------ | ------------ | --------- |
| **Provenance**         | **M**           | S       | S            | S            | 資料來源/處理歷程 |
| **AuditEvent**         | **M**           | O       | **M**        | **M**        | 存取稽核      |
| **Consent**            | **M**           | **M**   | —            | **M**        | 使用授權      |
| **VerificationResult** | **M**           | O       | O            | O            | 資料可信度     |

---

# 🧠 一句話總結（設計哲學）

> **核心臨床時序圖 = Patient
> ＋ Encounter（時間切片）
> ＋ EpisodeOfCare（長期主線）
> ＋ Request → Event → Result（可回溯鏈）
> ＋ Provenance（可信任）**



