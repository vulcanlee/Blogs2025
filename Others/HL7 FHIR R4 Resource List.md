# HL7 FHIR v4.0.1 (R4) 官方 Resource List

[耀瑄科技 新創小組]

為了要充分掌握 FHIR 的各 Resource 的使用與操作，有必要對於 FHIR R4 的 145 個 Resource 有一個完整的認識與理解。以下是 FHIR R4 官方文件中所列出的 Resource 清單，包含每個 Resource 的名稱與簡要說明。

以下內容**以 HL7 FHIR v4.0.1 (R4) 官方 Resource List（Categorized）為準**整理（分類：Foundation / Base / Clinical / Financial / Specialized）。([HL7][1])

> 中文名稱為**常見譯名**（台灣業界/專案常用），不同專案可能有些許差異；真正「強制」與否，仍以該 Resource 的元素基數（min..max）與你採用的 Profile/IG 為準。

---

## Foundation（基礎層）

| Resource                | 中文名稱           | 意義                        | 用途                                   | 注意事項                          |
| ----------------------- | -------------- | ------------------------- | ------------------------------------ | ----------------------------- |
| CapabilityStatement     | 能力宣告           | 伺服器/系統能力描述                | 互通前能力盤點、合規                           | 常用於 conformance；內容要與實作一致      |
| StructureDefinition     | 結構定義(Profiles) | 定義/約束資源結構                 | Profile、Extension、IG 基礎              | Profile 決定「你家 FHIR」長什麼樣       |
| ImplementationGuide     | 實作指引           | IG 封裝（profiles/valuesets） | 發佈規範、導入指南                            | 版本控管很重要（IG 版本=契約）             |
| SearchParameter         | 查詢參數           | 定義可查欄位/鏈結                 | 自訂搜尋、索引策略                            | 伺服器不一定支援自訂 SearchParam        |
| MessageDefinition       | 訊息定義           | 訊息事件/結構                   | FHIR Messaging                       | 與 MessageHeader 搭配            |
| OperationDefinition     | 操作定義           | $operation 定義             | 自訂操作（如 $evaluate）                    | 需明確輸入輸出 Parameters            |
| CompartmentDefinition   | 區隔定義           | Compartment 規則            | 權限/分艙、資料範圍                           | 常影響安全與查詢範圍                    |
| StructureMap            | 結構對映           | 模型轉換規則                    | HL7v2/CDA/內碼→FHIR                    | 實作通常靠引擎，不是純 REST              |
| GraphDefinition         | 圖譜定義           | 連帶查詢結構                    | $graph 一次取關聯資源                       | 伺服器支援度差異大                     |
| ExampleScenario         | 範例情境           | 情境與交換範例                   | 文件化互通流程                              | 用於說明，不是臨床資料主體                 |
| CodeSystem              | 代碼系統           | 定義 codes                  | SNOMED/自訂碼系統                         | 發佈/版控/授權要注意                   |
| ValueSet                | 值集             | 可用碼集合                     | 欄位允許值                                | 需分清「綁定強度」(required/preferred) |
| ConceptMap              | 概念對照           | code 映射                   | 內碼↔標準碼對照                             | 需記錄 mapping 等級與維護責任           |
| NamingSystem            | 命名系統           | identifier system 註冊      | OID/URI 管理                           | identifier.system 的治理基礎       |
| TerminologyCapabilities | 術語能力           | Terminology 服務能力          | 驗證/擴展/翻譯能力                           | 常配合 terminology server        |
| Provenance              | 溯源             | 資料來源/處理歷程                 | 審計、可信度                               | target 可指向任何資源；量大需策略化儲存       |
| AuditEvent              | 稽核事件           | 安全稽核紀錄                    | 存取紀錄、合規                              | 需注意效能/容量；與法規一致                |
| Consent                 | 同意書/同意狀態       | 授權/拒絕規則                   | 資料使用控管                               | 實作常需和 IAM/Policy engine 整合    |
| Composition             | 文件組成           | 文件章節結構                    | 病歷文件（出院摘要等）                          | 常作為 Document Bundle 的主檔       |
| DocumentManifest        | 文件清單           | 文件集合索引                    | 一次交付多文件                              | 與 DocumentReference 配合        |
| DocumentReference       | 文件參照           | 文件中繼資料                    | PDF/影像報告索引                           | attachment 可放外部 URL；隱私/存取控制要做 |
| CatalogEntry            | 目錄條目           | 目錄化項目                     | 服務/產品型目錄                             | 實務採用度較低，常被客製替代                |
| Basic                   | 基本資源           | 可擴充的「容器」                  | 臨時/過渡資料                              | 能用正式資源就別用 Basic（互通性差）         |
| Binary                  | 二進位            | 原始二進位內容                   | 存檔（PDF、影像）                           | 通常不建議直接存大量 Binary（成本/效能）      |
| Bundle                  | 資源包            | 多資源容器                     | transaction/batch/searchset/document | type 不同語意不同；transaction 需原子性  |
| Linkage                 | 連結             | 多筆記錄合併/關聯                 | 主索引(MPI)相關                           | 用於「同一人多ID」等情境                 |
| MessageHeader           | 訊息標頭           | FHIR Messaging 入口         | 訊息路由/事件                              | 需搭配 MessageDefinition         |
| OperationOutcome        | 操作結果           | 錯誤/警告/資訊                  | API 回錯、驗證回應                          | issue.code/diagnostics 要可讀可追  |
| Parameters              | 參數             | $operation 入出參            | 自訂/標準 operation                      | 不是臨床主資料；只作封裝                  |
| Subscription            | 訂閱             | 事件通知                      | 資料變更推播                               | R4/後續版本差異大；交付通道需治理            |

---

## Base（基本層）

| Resource                   | 中文名稱      | 意義        | 用途              | 注意事項                           |
| -------------------------- | --------- | --------- | --------------- | ------------------------------ |
| Patient                    | 病人        | 照護對象      | 人口學、主索引         | identifier 治理（病歷號/身分證/跨院）      |
| Practitioner               | 醫事人員      | 提供照護的人    | 醫師/護理師資料        | 常與 PractitionerRole 搭配呈現職責/科別  |
| PractitionerRole           | 人員角色      | 人員在某機構的角色 | 科別/職稱/服務        | 比 Practitioner 更貼近「在這家院所的身份」   |
| RelatedPerson              | 相關人       | 病人的家屬/照護者 | 緊急聯絡、監護人        | 與 Patient 關係必須明確               |
| Person                     | 人(泛用)     | 可連多角色/多身份 | MPI/跨系統人員       | 常用於合併多 Patient/Practitioner 身份 |
| Group                      | 群組        | 人/物件集合    | cohort、群組處置     | 可動態/靜態；注意 membership 管理        |
| Organization               | 組織        | 醫療機構/單位   | 院所/科部/機構        | partOf 用於樹狀組織                  |
| OrganizationAffiliation    | 組織關係      | 組織間合作     | 轉診/合作網          | 使用度較低，但跨院合作可用                  |
| HealthcareService          | 醫療服務      | 提供的服務項    | 門診服務、專科服務       | 常與 Location/Endpoint 關聯        |
| Endpoint                   | 端點        | 系統存取端點    | FHIR/HL7v2/影像端點 | 安全屬性、連線資訊要受控                   |
| Location                   | 地點        | 實體位置      | 病房/診間/病床        | 與 Encounter/Appointment 常關聯    |
| Substance                  | 物質        | 物質定義      | 過敏、檢驗、藥材        | 通常搭配術語系統（SNOMED 等）             |
| BiologicallyDerivedProduct | 生物衍生製品    | 血品/組織等    | 輸血/移植鏈          | 採用度視專案而定                       |
| Device                     | 裝置        | 醫療/IT 裝置  | 儀器、植入物          | UDI、狀態、病人綁定要清楚                 |
| DeviceMetric               | 裝置量測      | 裝置輸出的量測項  | 監測設備指標          | 常與 Observation/Device 搭配       |
| Task                       | 任務        | 工作項       | 工作流程追蹤          | 常被用作 workflow glue；狀態機要定義好     |
| Appointment                | 預約        | 排程事件      | 掛號/檢查排程         | 需處理取消/改期；participant 管理        |
| AppointmentResponse        | 預約回覆      | 參與者回覆     | 確認/拒絕           | 與 Appointment 的狀態一致性           |
| Schedule                   | 班表/時段表    | 可預約時段集合   | 醫師/設備可用性        | 與 Slot 配合                      |
| Slot                       | 時段        | 單一可預約時段   | 排程細粒度           | 時區、重複規則要一致                     |
| VerificationResult         | 驗證結果      | 資料驗證狀態    | 資料可信度/驗證        | 常用於「資料來源已驗證」                   |
| Encounter                  | 就醫事件      | 一次照護接觸    | 門診/住院/急診        | 多數臨床資源「可選」綁 encounter，強烈建議綁    |
| EpisodeOfCare              | 照護階段      | 長期照護段落    | 慢病、癌症療程         | 用於串多次 Encounter 的主軸            |
| Flag                       | 旗標        | 警示/提示     | 過敏提醒、隔離等        | 與安全性/提醒流程密切                    |
| List                       | 清單        | 條目集合      | 問題清單、用藥清單       | 不等同於「真實資料」，多為整理/視圖             |
| Library                    | 知識庫(決策邏輯) | 可重用邏輯/規則  | CDS、計算、評估       | 常與 Measure/PlanDefinition 搭配   |

---

## Clinical（臨床層）

| Resource                   | 中文名稱   | 意義             | 用途             | 注意事項                                   |
| -------------------------- | ------ | -------------- | -------------- | -------------------------------------- |
| AllergyIntolerance         | 過敏/不耐受 | 過敏與反應          | 過敏史            | code/反應嚴重度要可用於決策                       |
| AdverseEvent               | 不良事件   | 醫療相關不良事件       | 藥物/處置不良反應      | 與 SuspectEntity、處置關聯                   |
| Condition                  | 疾病/問題  | 診斷/問題狀態        | 問題清單、診斷        | clinicalStatus/verificationStatus 很關鍵  |
| Procedure                  | 處置/手術  | 已執行行為          | 手術、治療          | performed[x] 時間型態要一致                   |
| FamilyMemberHistory        | 家族史    | 家族疾病史          | 遺傳/風險評估        | 關係、發病年齡等結構化                            |
| ClinicalImpression         | 臨床印象   | 臨床判斷摘要         | 評估/推論結果        | 不等同診斷；常引述 Observation/Condition        |
| DetectedIssue              | 發現問題   | 系統/人員發現的問題     | 交互作用、禁忌        | 常由 CDS 產生；status/mitigation            |
| Observation                | 觀察/檢驗  | 量測/檢驗/評分       | Vital/Lab/評估量表 | value[x] 多型別；units/參考區間一致              |
| Media                      | 媒體     | 影像/聲音等         | 內視鏡圖、照片        | 大檔案多用 DocumentReference/Binary 外掛      |
| DiagnosticReport           | 診斷報告   | 檢查報告彙總         | 放射/檢驗報告        | result 指向 Observation；結構與 narrative 平衡 |
| Specimen                   | 檢體     | 樣本資訊           | 血液/組織/尿液       | 與 Observation/DiagnosticReport 關聯      |
| BodyStructure              | 身體構造   | 解剖部位           | 病灶位置           | 可用於 procedure、imaging 標記               |
| ImagingStudy               | 影像檢查   | DICOM Study 中繼 | CT/MR 檢查索引     | SOP/Series UID 等欄位治理；存取權限              |
| QuestionnaireResponse      | 問卷回覆   | 表單結果           | PRO、量表         | 對應 Questionnaire；版本一致                  |
| MolecularSequence          | 分子序列   | 基因序列資料         | 基因體資訊          | R4 已存在，但採用度視情境                         |
| MedicationRequest          | 用藥處方   | 開立用藥意圖         | 處方             | intent/order；與 dispense/admin 串起來      |
| MedicationAdministration   | 給藥     | 實際給藥           | 住院給藥           | 與 MedicationRequest 串聯一致性              |
| MedicationDispense         | 調劑     | 藥局調劑           | 出院帶藥、領藥        | 與處方/給藥的狀態對齊                            |
| MedicationStatement        | 用藥陳述   | 病人用藥敘述         | 現用藥（自述/整合）     | 可信度來源要標（Provenance）                    |
| Medication                 | 藥品     | 藥品定義           | 藥碼/成分          | 常用 RxNorm/院內碼；治理重點                     |
| MedicationKnowledge        | 藥品知識   | 藥物知識庫          | 禁忌/交互作用        | 不一定每家伺服器都落地完整                          |
| Immunization               | 疫苗接種   | 接種事件           | 疫苗史            | lot/performer/occurrence               |
| ImmunizationEvaluation     | 接種評估   | 接種有效性評估        | 免疫成效           | 多用於公衛流程                                |
| ImmunizationRecommendation | 接種建議   | 建議接種計畫         | 接種排程           | 與免疫規則/族群建議                             |
| CarePlan                   | 照護計畫   | 照護活動集合         | 個案管理           | 與 Goal/Activity 串聯；狀態管理                |
| CareTeam                   | 照護團隊   | 團隊成員           | 多專業協作          | role/period 很重要                        |
| Goal                       | 目標     | 照護目標           | 控糖/減重等         | 可量化指標與時間窗                              |
| ServiceRequest             | 服務申請   | 檢查/會診申請        | 檢驗單、影像申請       | 與 DiagnosticReport/Procedure 常形成鏈      |
| NutritionOrder             | 營養醫囑   | 飲食處方           | 住院餐            | 結構較複雜；採用度依院所                           |
| VisionPrescription         | 視力處方   | 眼鏡/隱形眼鏡處方      | 眼科             | 多為專科情境                                 |
| RiskAssessment             | 風險評估   | 風險結果           | 跌倒/再入院風險       | prediction/概率/分層要明確                    |
| RequestGroup               | 申請群組   | 多個 request 的組合 | order sets     | 常與 PlanDefinition/ActivityDefinition   |
| Communication              | 溝通     | 已發生溝通          | 通知/交班          | payload 內容與隱私控管                        |
| CommunicationRequest       | 溝通請求   | 要求溝通           | 通知需求           | 與 Task/Communication 可互補               |
| DeviceRequest              | 裝置申請   | 申請裝置使用/供應      | 植入物/耗材         | 與 SupplyRequest 區分清楚                   |
| DeviceUseStatement         | 裝置使用陳述 | 病人使用裝置         | 居家/院內設備        | 多用於追蹤與陳述                               |
| GuidanceResponse           | 指引回覆   | CDS 回傳結果       | 決策支援輸出         | 通常由 CDS engine 產生                      |
| SupplyRequest              | 供應申請   | 耗材/供應需求        | 藥材/耗材申請        | 與 SupplyDelivery 鏈起來                   |
| SupplyDelivery             | 供應交付   | 實際交付           | 出庫/交付          | 狀態/數量一致性                               |

---

## Financial（財務層）

| Resource                    | 中文名稱      | 意義         | 用途        | 注意事項                          |
| --------------------------- | --------- | ---------- | --------- | ----------------------------- |
| Coverage                    | 保險保障      | 保險/給付      | 投保資訊      | 與 payor、policyHolder 連動       |
| CoverageEligibilityRequest  | 給付資格請求    | 查資格        | 事前審核      | 常需與 payer 系統整合                |
| CoverageEligibilityResponse | 給付資格回覆    | 資格結果       | 回覆/限制     | benefit 結構要理解                 |
| EnrollmentRequest           | 加入請求      | 參與保險計畫     | 註冊流程      | 採用度依地區制度                      |
| EnrollmentResponse          | 加入回覆      | 加入結果       | 回覆狀態      | 同上                            |
| Claim                       | 請款        | 醫療費用申報     | 申報/請款     | 欄位非常多；與在地制度高度相關               |
| ClaimResponse               | 請款回覆      | 審核結果       | 核付/拒付     | adjudication 需對應 payer 規則     |
| Invoice                     | 發票/帳單     | 帳單彙總       | 出帳        | 與 ChargeItem/Account 鏈結       |
| PaymentNotice               | 付款通知      | 已付款告知      | 付款事件      | 與 reconciliation 一起看          |
| PaymentReconciliation       | 對帳        | 付款對帳       | 金流對帳      | 需對應外部金流系統                     |
| Account                     | 帳戶        | 病人/事件帳戶    | 帳務彙總      | 常與 Encounter/EpisodeOfCare 關聯 |
| ChargeItem                  | 計費項目      | 可計費項       | 計價明細      | code/quantity/來源要可追溯          |
| ChargeItemDefinition        | 計費定義      | 計費規則       | 價格表/規則    | 治理與版控是重點                      |
| Contract                    | 合約        | 合約條款       | 院所合約/保險合約 | 多為行政/法務資料                     |
| ExplanationOfBenefit        | 給付說明(EOB) | payer 最終說明 | 理賠結果      | 與 Claim/ClaimResponse 鏈完整     |
| InsurancePlan               | 保險計畫      | 計畫內容       | 方案/網絡     | 通常由 payer 提供                  |

---

## Specialized（特殊領域層）

| Resource                          | 中文名稱     | 意義             | 用途                       | 注意事項                            |
| --------------------------------- | -------- | -------------- | ------------------------ | ------------------------------- |
| ResearchStudy                     | 研究計畫     | 研究基本資料         | 臨床研究                     | 與 ResearchSubject 串受試者          |
| ResearchSubject                   | 研究受試者    | 受試者參與          | 收案/追蹤                    | 與 Patient、研究狀態關聯                |
| ActivityDefinition                | 活動定義     | 可重用活動模板        | order sets/流程            | 用於生成 request（搭配 PlanDefinition） |
| DeviceDefinition                  | 裝置定義     | 裝置型錄/規格        | 產品主檔                     | Device 實例的「型號主檔」                |
| EventDefinition                   | 事件定義     | 事件模板           | 工作流/事件                   | 採用度依實作                          |
| ObservationDefinition             | 觀察定義     | Observation 模板 | 檢驗項定義                    | 與 LOINC/值域綁定常見                  |
| PlanDefinition                    | 計畫定義     | 流程/照護路徑模板      | 產生 CarePlan/RequestGroup | 與 ActivityDefinition 配合         |
| Questionnaire                     | 問卷       | 問卷模板           | 表單定義                     | 版本管理很重要                         |
| SpecimenDefinition                | 檢體定義     | 檢體規格           | 檢體處理規範                   | 專業實驗室情境                         |
| ResearchDefinition                | 研究定義     | EBM/研究規格       | 證據鏈                      | 採用度依 EBM 平台                     |
| ResearchElementDefinition         | 研究元素定義   | 族群/曝露/結果定義     | EBM 計算                   | 與 EvidenceVariable 概念相近         |
| Evidence                          | 證據       | 證據資源           | 指引/證據管理                  | 結構較複雜                           |
| EvidenceVariable                  | 證據變數     | 族群/介入/結果變數     | EBM                      | 定義嚴謹度決定可重用性                     |
| EffectEvidenceSynthesis           | 效果整合     | 統合分析結果         | EBM                      | 多用於研究平台                         |
| RiskEvidenceSynthesis             | 風險整合     | 風險統合           | EBM                      | 同上                              |
| Measure                           | 品質量測     | 指標定義           | 品質指標                     | 通常搭配 CQL/Library                |
| MeasureReport                     | 指標報告     | 指標結果           | 回報/稽核                    | 期間、族群要一致                        |
| TestScript                        | 測試腳本     | 互通測試腳本         | conformance 測試           | 伺服器支援度不同                        |
| TestReport                        | 測試報告     | 測試結果           | 測試產出                     | 與 TestScript 配合                 |
| MedicinalProduct                  | 藥品主檔(專門) | 藥政層級資料         | 藥證/法規                    | R4 後續版本有變動；採用前先確認版本策略           |
| MedicinalProductAuthorization     | 藥證核准     | 藥證資訊           | 法規                       | 同上                              |
| MedicinalProductContraindication  | 禁忌症      | 禁忌             | 藥政                       | 同上                              |
| MedicinalProductIndication        | 適應症      | 適應症            | 藥政                       | 同上                              |
| MedicinalProductIngredient        | 成分       | 成分構成           | 藥政                       | 同上                              |
| MedicinalProductInteraction       | 交互作用     | 交互作用           | 藥政                       | 同上                              |
| MedicinalProductManufactured      | 製造品      | 製造資訊           | 藥政                       | 同上                              |
| MedicinalProductPackaged          | 包裝品      | 包裝資訊           | 藥政                       | 同上                              |
| MedicinalProductPharmaceutical    | 藥劑       | 劑型等            | 藥政                       | 同上                              |
| MedicinalProductUndesirableEffect | 不良反應(藥政) | 不良反應           | 藥政                       | 同上                              |
| SubstanceNucleicAcid              | 物質-核酸    | 物質定義           | 生物製劑                     | 專業領域採用                          |
| SubstancePolymer                  | 物質-聚合物   | 物質定義           | 生物製劑                     | 同上                              |
| SubstanceProtein                  | 物質-蛋白    | 物質定義           | 生物製劑                     | 同上                              |
| SubstanceReferenceInformation     | 物質-參考資訊  | 物質參考           | 生物製劑                     | 同上                              |
| SubstanceSpecification            | 物質-規格    | 物質規格           | 生物製劑                     | 同上                              |
| SubstanceSourceMaterial           | 物質-來源材料  | 來源材料           | 生物製劑                     | 同上                              |

> 分類與清單來源：FHIR R4 (v4.0.1) Resource List（Categorized）。([HL7][1])

---

# Resource 彼此互相參考（Reference）關聯方式整理

FHIR 的「關聯」幾乎都透過 **Reference**（`Reference(ResourceType/id)`）來表達；是否「強制」主要看兩件事：

1. 該欄位在該 Resource 的 **min cardinality 是否為 1**；
2. 你採用的 **Profile/IG 是否加嚴（把 0..1 改成 1..1）**。([HL7][1])

下面用「架構上最常見、最有用」的方式幫你整理（也最符合你做**病人時序圖/資料中台/臨床平台**的實作需求）。

---

## 1) 幾乎所有臨床資料的「主軸」：Patient / Encounter / EpisodeOfCare

### A. Patient（最常見：強烈建議、很多情境也接近強制）

* **目的**：把所有臨床事件聚焦到「人」。
* **常見欄位名**：`subject` / `patient`（不同資源命名不同）
* **實務結論**：

  * 對「病人導向」系統（你的時序圖、個管、研究 cohort），**沒有 Patient 幾乎就失去意義**。
  * 就算某些資源規範上允許不是病人（例如群體/設備/地點），你仍可用 Profile 把常用情境固定成 Patient。

### B. Encounter（多數是可選 0..1，但臨床系統強烈建議綁）

* **目的**：把 Observation/Procedure/Medication… 掛到「這一次門診/住院/急診」的情境，便於時序化、結帳、稽核。
* **好處**：同一天多次就診可分開；住院期間每日 labs 可歸到同一次住院 encounter 或其子 encounter。

### C. EpisodeOfCare（通常可選，但對長療程很有價值）

* **目的**：跨多次 Encounter 的更高層「療程/照護段落」（癌症療程、慢病個管）。
* **對你的系統**：做癌症治療平台、安寧、個管時，EpisodeOfCare 是很好的「長期主軸」。

---

## 2) 「誰做的」：Practitioner / PractitionerRole / Organization / Location

* **目的**：把資料的責任歸屬與執行者補齊（誰開立、誰執行、在哪裡做）。
* **常見角色欄位名**：`performer`、`author`、`asserter`、`requester`、`recorder`
* **建議做法**：

  * **人**用 Practitioner；**在某院某科的職責**用 PractitionerRole；**組織/科部**用 Organization；**病房診間**用 Location。
  * 這些通常不是「規範強制」，但對稽核、權限、流程追蹤幾乎是必要。

---

## 3) 「訂單→執行→結果」的鏈：ServiceRequest / Procedure / Observation / DiagnosticReport

最常見的臨床工作流是：

1. **ServiceRequest**（開單：抽血/影像/會診/檢查）
2. **Specimen**（若有檢體）
3. **Observation**（單項結果：每個檢驗值）
4. **DiagnosticReport**（報告彙總：整份檢驗/放射報告）
5. **Procedure**（若是處置/檢查動作本身要記錄）

**關聯目的**：可追溯、可重現流程、可稽核；也最適合做「病人時序圖」。
（你做「抽血項目」需求異動/追蹤時，這條鏈就是核心。）

---

## 4) 用藥鏈：MedicationRequest → MedicationDispense → MedicationAdministration / MedicationStatement

* **MedicationRequest**：醫囑/處方（意圖）
* **MedicationDispense**：藥局調劑（供應）
* **MedicationAdministration**：實際給藥（執行）
* **MedicationStatement**：病人用藥陳述（整合/自述）

**目的**：把「醫師想給」與「實際有沒有拿到/有沒有打」拆清楚，避免資料混用。

---

## 5) 文件/影像：DocumentReference / Composition / Bundle / ImagingStudy / Media

* **DocumentReference**：文件中繼資料（最實用的索引）
* **Composition**：文件章節結構（想要「可讀的臨床文件」時用）
* **Bundle(document)**：交付整份文件包
* **ImagingStudy**：DICOM Study 索引（影像檢查）
* **Media/Binary**：小型媒體/二進位內容（大量影像通常不建議直接塞）

**目的**：把「大檔案」與「可查可索引」分離，確保效能與權限。

---

## 6) 治理與可信度：Provenance / AuditEvent / Consent / VerificationResult

* **Provenance**：資料是誰、何時、從哪個系統、經過什麼處理而來（對研究/法遵極重要）
* **AuditEvent**：誰查了什麼（資安稽核）
* **Consent**：可不可以用（資料使用授權）
* **VerificationResult**：某份資料是否已驗證可信

**目的**：在「醫院上線」與「跨院/研究」情境下，這些往往比資料本體更決定能不能用。

---

## 7) 快速判斷「是否需要強制關聯」的實務規則（你可直接套到 Profile）

1. **只要是病人資料 → 強制綁 Patient**（subject/patient：1..1）
2. **只要能對應到一次就醫 → 強制綁 Encounter**（encounter：1..1）
3. **只要是流程型資料（開單/執行/報告）→ 強制綁 request/基準資源**（例如 result basedOn）
4. **只要會做稽核/回溯/研究 → 至少保留 Provenance（或等價欄位）**

[1]: https://hl7.org/fhir/R4/resourcelist.html "Resourcelist - FHIR v4.0.1"
