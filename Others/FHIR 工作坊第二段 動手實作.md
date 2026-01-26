# FHIR 工作坊 第二段 動手實作 筆記

[耀瑄科技 新創小組]

---

# 從零開始把 FHIR 跑起來：

## 一場「HAPI FHIR Server 動手實作」背後，其實在教你什麼？

> 這篇文章，整理自一場由 **工研院** 受 **衛生福利部** 委託舉辦的 FHIR 工作坊 Hands-on Lab。
> 表面上它在「教你安裝 HAPI FHIR Server」，但實際上，它在傳遞的是：
> **一個 FHIR Server 要能「真的被拿來用」，背後需要哪些關鍵元件與正確順序。**

本文特別適合：

* 第一次接觸 **FHIR / SMART on FHIR** 的醫院資訊人員
* 正在評估或嘗試 **FHIR Server PoC** 的醫療新創團隊
* 曾經「架過 Server，但一直卡在 validation、代碼、IG」的人

---

## 一、為什麼工作坊一開始就選 HAPI FHIR Server？

### 背景脈絡（錄音裡沒講清楚，但這是關鍵）

在 FHIR 生態系中，HAPI FHIR Server 幾乎是**學習與 PoC 階段的事實標準**，原因不是因為它「最好」，而是因為它：

* 完整支援 **FHIR R4 / R5**
* 有清楚的 RESTful API 實作
* 內建 **Validation、Terminology、IG 支援機制**
* 能快速切換不同資料儲存後端（in-memory / Lucene / Elasticsearch）

👉 **換句話說**：
這場 Hands-on Lab 的目的，不是教你「怎麼安裝一個 Java 專案」，而是讓你**第一次把 FHIR Server 的完整能力「摸過一輪」**。

---

## 二、FHIR Server 的基本架構與運作原理（先把全貌看清楚）

在開始任何安裝前，先用一句話理解 **FHIR Server 在做什麼**：

> **FHIR Server = 一個懂 FHIR Schema、會驗證資料、能處理醫療術語、並提供 REST API 的資料平台**

它通常包含以下幾個核心模組：

```
FHIR REST API
 ├─ Resource CRUD (Patient / Observation / Condition ...)
 ├─ Validation Engine (Schema + Profile)
 ├─ Terminology Service (CodeSystem / ValueSet)
 ├─ Persistence Layer (資料庫)
 └─ Optional: Search / Index / Analytics
```

這正是為什麼工作坊後面會一直提到：

* Validation
* Terminology
* IG
* Backend 切換（Lucene / Elasticsearch）

👉 **它們不是附加功能，而是 FHIR Server 的「標準配備」**

---

## 三、application.yaml：真正控制 FHIR Server 行為的地方

在 Hands-on Lab 中，講者反覆提到 `application.yaml`，但沒有一次把它講清楚。

### application.yaml 是什麼？

它是 **HAPI FHIR Server 的行為設定中樞**，決定了：

* 用哪個 FHIR 版本（R4 / R5）
* 使用哪種資料儲存後端
* 是否啟用 validation
* 是否啟用 terminology
* 是否預先展開 ValueSet（Pre-expand）
* 是否允許不合法的 Resource 被寫入

### 為什麼這麼重要？

因為：

> **你遇到的 80% 問題，其實不是 Resource 寫錯，而是 application.yaml 沒設好**

---

## 四、資料儲存後端：Lucene vs Elasticsearch（不是效能比較而已）

### 1️⃣ Lucene backend 是什麼？

* 內建、免安裝
* 適合教學、PoC、Lab
* 啟動快、設定少
* 搜尋功能完整但不適合大量資料

👉 Hands-on Lab 預設使用 Lucene，是為了**降低門檻**

### 2️⃣ Elasticsearch backend 在幹嘛？

* 適合大量病人資料
* 搜尋與 aggregation 能力強
* 常用於「研究平台 / 病患篩選 / cohort discovery」

👉 錄音裡提到 Elasticsearch，其實是在暗示：

> **當你真的要「上線用」時，後端一定會換**

---

## 五、Terminology 是什麼？為什麼不裝，後面全都會卡？

### Terminology ≠ 單純代碼表

在 FHIR 世界中，Terminology Service 負責：

* 管理 **CodeSystem**（ICD-10、LOINC、SNOMED CT）
* 管理 **ValueSet**
* 檢查 coding 是否合法
* 支援 validation 與 decision logic

如果沒有 Terminology：

* Observation.code 你怎麼知道填的 LOINC 合不合法？
* Profile 裡的 binding 根本無法驗證

👉 這也是為什麼講者強調順序：

> **一定要先裝好 ICD-10 / LOINC / SNOMED CT，才能裝 IG**

---

## 六、Pre-expand ValueSet：為什麼效能突然差很多？

這是 Hands-on Lab 中「只被輕描淡寫帶過，但實務超重要」的一段。

### 什麼是 Pre-expand？

* 把 ValueSet 中可能的代碼 **事先展開並快取**
* 換取 validation 與查詢時的速度

### 為什麼重要？

* 沒 pre-expand → validation 很慢
* 大型 IG（例如 TW Core）幾乎**一定要開**

👉 這也是為什麼很多人第一次驗證 Resource 時會覺得：

> 「怎麼這麼慢？是不是 server 壞了？」

---

## 七、FHIR Schema 與 Validation：你到底在驗證什麼？

### 兩層驗證，常被混在一起

#### 1️⃣ FHIR Schema Validation

* Resource 結構是否正確
* 欄位型別是否符合（string / date / CodeableConcept）

#### 2️⃣ Profile Validation

* 是否符合 TW Core IG 的約束
* 必填欄位有沒有填
* coding 是否符合 binding 的 ValueSet
* Reference 是否指向正確 Resource/Profile

👉 Hands-on Lab 的重點不是「送資料成功」，而是：

> **你是否看得懂 validation error 在說什麼**

---

## 八、為什麼「一定要先裝 TW Core IG」？

### IG（Implementation Guide）在這裡扮演什麼角色？

TW Core IG 定義了：

* 台灣情境下「應該怎麼用 FHIR」
* Patient / Observation / Encounter 的必要欄位
* 身分證、病歷號、機構結構的建議做法
* 常用代碼的 binding 規則

### 正確順序（這是 Hands-on Lab 真正想教的）

1. 安裝 HAPI FHIR Server
2. 啟用 Terminology
3. 安裝 ICD-10 / LOINC / SNOMED CT
4. 設定 Pre-expand
5. 安裝 TW Core IG
6. 開始送 Resource
7. 讀懂 Validation 結果

👉 **順序錯，後面一定全錯**

---

## 九、FHIR RESTful API：你其實已經會用了，只是不知道名字

最後，工作坊會帶大家實際呼叫 API，例如：

* `POST /Patient`
* `POST /Observation`
* `GET /Patient?_id=123`
* `POST /Bundle`

這些不是什麼特別 API，而是：

> **FHIR RESTful API = 標準化的醫療資料 CRUD**

你會的 HTTP、JSON、REST，全都可以直接用上。

---

以下為你提供的這份 **FHIR 工作坊 Hands-on Lab 文件（來源：hackmd.io/@medstandard/Byv5yOhB-g）** 的 **簡明摘要與做法整理**，保留必要步驟與重點概念，但不涵蓋細節指令或完整設定内容，適合入門者快速理解與實作參考：

---

# 簡潔版：FHIR Server 安裝與實作總覽

本工作坊旨在讓開發者能 **動手安裝、設定與測試符合臺灣 FHIR 標準（TW Core IG）的 HAPI FHIR Server 實例**，並透過相關工具與流程建立開發能力。

---

## 1) 必備軟體與環境準備

在開始實作之前，請先在您的開發機或 VM 上安裝與準備：

* Visual Studio Code 或其他編輯器
* Docker / Docker Desktop / Podman
* Docker Compose
* Git
* Postman（用於 API 測試） 

**建議環境配備**：
CPU 8 核、記憶體 16GB（若安裝完整術語庫與 IG 建議 32GB），硬碟空間 >100GB。 

---

## 2) 安裝 HAPI FHIR Server（主要應用）

### 主要流程

1. **Clone HAPI FHIR Server Starter 專案**
   把官方 starter repo 下載到本機。 

2. **修改組態檔（application.yaml）**
   啟動下列功能以支援驗證與查詢：

   * 允許上傳 Implementation Guide（IG Runtime Upload）
   * 啟用 Request Validation
   * 啟用搜尋及 Lucene backend
   * 啟用 Pre-expand ValueSet 等功能

3. **Docker Compose 建置與啟動**
   執行建置與啟動命令，並觀察 Server 日誌以確認啟動成功。

4. **觀察啟動狀況**
   使用 `docker compose logs -f …` 觀察 FHIR Server 是否已完全啟動。 

---

## 3) 安裝 Terminology（術語庫）

FHIR 的術語是醫療資料一致性的核心，因此需要安裝下列 **CodeSystem / ValueSet**：

* **ICD-10**（診斷碼）
* **LOINC**（檢驗碼）
* **SNOMED CT**（臨床概念） 

### 安裝重點

完成一個 CodeSystem 之後，請等日誌顯示處理完成才開始下一個；尤其 **LOINC 與 SNOMED CT** 安裝時間較長。 

---

## 4) 使用 VS Code 加入 FHIR Schema 支援

為了在撰寫 FHIR JSON 時獲得 IntelliSense 與 schema 驗證，可以：

1. 下載 **FHIR Schema JSON（例如 FHIR R4 的 `fhir.schema.json`）**
2. 透過 VS Code 設定讓 `.fhir.json` 檔案套用 schema
3. 在資源編輯時自動顯示欄位提示與結構規則 

---

## 5) 使用 FHIR REST API 進行 CRUD

可利用 Postman 或其他 HTTP client：

* `POST` / `PUT` 新增 Resource
* `GET` 查詢 Resource
* `PUT` 更新 Resource
* `DELETE` 移除 Resource
* 使用 `$expand` 對 ValueSet 進行擴展測試 

---

## 6) 安裝 TW Core IG（臺灣核心 Implementation Guide）

**TW Core IG 提供臺灣場景下的 FHIR Profile 與約束規範**。建議安裝流程為：

1. 確定 Terminology（ICD-10/LOINC/SNOMED CT）已全部安裝完成
2. 解壓 TW Core IG 套件（ZIP）
3. 執行 IG 建立與上傳腳本（如 `ig.sh` / `upload_ig.sh`）
4. 在 Server 上完成 IG 套件的部署 

> 注意：TW Core IG 依賴已存在的 Terminology，若先裝 IG、後裝術語，可能導致驗證錯誤。

---

## 7) 驗證 Resource 是否符合 TW Core 規範

安裝完 IG 之後，可以利用 FHIR Server 提供的 `$validate` Operation：

* 呼叫 `POST …/$validate` 並帶入要驗證的 Resource JSON
* 查看驗證結果是否符合 Profile 規範 

這一步是確認資料是否真正 “符合標準”，而非只是能寫入。

---

## 8) 其他補充注意事項（概念性）

### Terminology 與 ValueSet 擴展

要讓 FHIR Server 正確進行代碼查驗與 ValueSet 擴展（$expand），必須：

* 啟用 **Lucene backend 或 Elasticsearch**
* 在 `application.yaml` 設定 **pre_expand_value_sets** 等相關選項
* 避免安裝過程中未完成就進行查詢 

### 在 VS Code 使用 JSON Schema

透過 schema 配置，可以提高 JSON 編輯正確性，降低因結構錯誤而造成的 API 驗證失敗。 

---

## 十、小結：這場 Hands-on Lab 真正想教你的三件事

1️⃣ **FHIR Server 不是資料庫，是一套「有規則的醫療資料平台」**
2️⃣ **Validation 與 Terminology 是核心，不是選配**
3️⃣ **IG（尤其是 TW Core）是讓你「真的能互通」的關鍵**






