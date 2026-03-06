# 使用 OpenAI Codex 與透過自然語言用 Blazor 來動手實作進行開發出一個 FHIR 為基礎的病人時序圖應用系統

先說結論，經過最終結果，雖然花了約 1 個小時的時間，但是，我也是完成了採用 ASP.NET Core Blazor 開發框架技術來開發出另外一套病人時序圖的前端應用程式，並且在開發過程中，我也是順利地使用 OpenAI Codex 來幫助我完成這個專案的開發，從專案的建立、到功能的實作、到測試與驗證，整個過程都是非常順利的，並且最終也成功地完成了這個專案的開發。這裡可以確認的是，比起之前採用純粹人工的方式來開發這個病人時序圖的前端應用程式，當時花費了兩個星期的時間，但是，比起一個小時的時間，這次使用 OpenAI Codex 來開發這個病人時序圖的前端應用程式，確實是大幅度地縮短了開發的時間，並且在開發過程中，也確實是非常順利的，沒有遇到什麼特別大的問題，整個過程也是非常愉快的。

在上一篇文章 [使用 OpenAI Codex 與透過自然語言來動手實作進行開發出一個 FHIR 為基礎的病人時序圖應用系統](https://csharpkh.blogspot.com/2026/02/openai-codex-fhir.html)，我第一次使用 OpenAI Codex 工具，此用 React + TypeScript 來建立一個系統，這個系統是採用讀取 FHIR Server 上的資料，然後將這些資料以時序圖的形式展示出來，這個系統的功能是可以輸入病人 ID，然後從 Firely Server 上獲取該病人的相關 FHIR 資料，並且將這些資料以時序圖的形式展示出來，從病人時序圖畫面，可以看到指定區間內該病人的所有就醫紀錄與該病人的綜合摘要說明，並且可以點擊每一筆就醫紀錄來查看該就醫紀錄的詳細資訊。

現在，我將會使用 OpenAI Codex 來開發一個採用 ASP.NET Core Blazor (Server Side Interactive Rendering & 不要 per page 方式，而是採用 Global 方式) 的前端應用程式，該系統是病人時序圖，用於展示病人從就診到治療的整個過程。在這裡可以輸入病人 ID，如 129c6ac7-8d06-89de-ad63-0204a93e76c3，然後從 Firely Server 上獲取該病人的相關 FHIR 資料，並將這些資料以時序圖的形式展示出來。例如，這個病人可以從這個 URL 取得 https://server.fire.ly/Patient/129c6ac7-8d06-89de-ad63-0204a93e76c3

我的目的在於需要驗證使用 OpenAI Codex 來開發這個病人時序圖的前端應用程式的可行性，是否可以採用另外一種技術，也就是採用 ASP.NET Core Blazor 的技術來開發這個病人時序圖的前端應用程式，並且看看使用 OpenAI Codex 來開發這個系統的過程中，是否可以順利地完成這個專案的開發，以及在開發過程中，是否可以順利地使用 OpenAI Codex 來幫助我完成這個專案的開發。

## 事前準備

在前一周，我下載的 Synthea™ 產生出來的 10 個模擬 FHIR 病人就醫紀錄，並且將這些紀錄匯入到網路上的免費 FHIR Server 上 ( https://server.fire.ly )，這些模擬病人就醫紀錄資料將會包含這些 FHIR 資源，包括 Organization、Location、Practitioner、PractitionerRole、Patient、Encounter、Condition、AllergyIntolerance、Immunization、Device、Observation、Procedure、DiagnosticReport、DocumentReference 以及 MedicationRequest 等等，這些資料將會成為我們開發病人時序圖應用系統的測試資料，讓我們能夠在開發過程中有實際的資料來進行測試和驗證。

## 使用 OpenAI Codex 來進行病人時序圖開發

* * 使用網頁開啟 OpenAI Codex 的平台後，並且登入到平台上
* 我在 Github 網頁上建立一個新的 Repository [PatientTimelineBlazor](https://github.com/vulcanlee/PatientTimelineBlazor)
* 在 OpenAI Codex 網頁上，設定與綁定此次要做的後續工作，將會在這個 Repository 內來呈現
* 在 OpenAI Codex 網頁上，介面相當的簡單，如同 ChatGPT 一樣，就只有一個文字輸入框
* 然後在這個文字輸入框裡面，我就可以輸入我想要 OpenAI Codex 來幫我完成的工作內容，這些工作內容可以是一些具體的指令，例如「建立一個 React / TypeScript 的前端應用程式」，或者是一些更抽象的描述，例如「開發一個 FHIR 為基礎的病人時序圖應用系統」，無論是什麼樣的內容，我都可以直接在這個文字輸入框裡面輸入，然後 OpenAI Codex 就會根據我的輸入來生成相應的程式碼和文件，讓我能夠快速地完成我的專案開發。
* 我將底下的內容輸入到這個文字輸入框裡面，讓 OpenAI Codex 來幫我完成這個專案的開發，這些內容包括了專案的目標、需求、開發工具與環境設定，以及每一個步驟的詳細說明，讓我能夠順利地使用 OpenAI Codex 來開發這個病人時序圖的前端應用程式。

```
我從 Synthea™ Patient Generator (GitHub) 下載了 Synthea™ 生成的 FHIR 資料，並使用 HAPI FHIR CLI 工具將資料匯入到 Firely Server 上 ( https://server.fire.ly )。以下是匯入的 FHIR Resource :
   * Organization ： 模擬的醫療機構資料。
   * Location ： 模擬的醫療機構地點資料。
   * Practitioner ： 模擬的醫療人員資料。
   * PractitionerRole ： 模擬的醫療人員角色資料。
   * Patient ： 模擬的病人資料。
   * Encounter ： 模擬的就診事件資料。
   * Condition ： 模擬的疾病或健康狀態資料。
   * AllergyIntolerance ： 模擬的過敏或不耐受資料。
   * Immunization ： 模擬的疫苗接種資料。
   * Device ： 模擬的醫療設備資料。
   * Observation ： 模擬的觀察資料。
   * Procedure ： 模擬的醫療程序資料。
   * DiagnosticReport ： 模擬的診斷報告資料。
   * DocumentReference ： 模擬的文件參考資料。
   * MedicationRequest ： 模擬的用藥請求資料。

我需要一份使用 OpenAI Codex 來進行詳細操作指引規劃文件，可以參考 https://developers.openai.com/codex 官方文件的操作與建議說明，在這裡需要開發一個採用 ASP.NET Core Blazor (Server Side Interactive Rendering & 不要 per page 方式，而是採用 Global 方式) 的前端應用程式，該系統是病人時序圖，用於展示病人從就診到治療的整個過程。在這裡可以輸入病人 ID，如 129c6ac7-8d06-89de-ad63-0204a93e76c3，然後從 Firely Server 上獲取該病人的相關 FHIR 資料，並將這些資料以時序圖的形式展示出來。例如，這個病人可以從這個 URL 取得 https://server.fire.ly/Patient/129c6ac7-8d06-89de-ad63-0204a93e76c3

從病人時序圖畫面，可以看到指定區間內該病人的所有就醫紀錄與該病人的綜合摘要說明，並且可以點擊每一筆就醫紀錄來查看該就醫紀錄的詳細資訊。這些就醫紀錄可以包括病人就診的日期、就診的醫療機構、就診的醫療人員、就診的疾病或健康狀態、就診的過敏或不耐受、就診的疫苗接種、就診的醫療設備、就診的觀察、就診的醫療程序、就診的診斷報告、就診的文件參考以及就診的用藥請求等資訊。

由於我沒有使用過 OpenAI Codex 工具的經驗，需要列出如何安裝那些工具與軟體或檔案，輸入那些指令，在哪裡做甚麼操作， Prompt 該輸入甚麼內容，每一個步驟的詳細說明，讓我可以順利地使用 OpenAI Codex 來開發這個病人時序圖的前端應用程式。開發工具將會採用 VS Code，並且已經安裝 https://marketplace.visualstudio.com/items?itemName=openai.chatgpt 這個 VS Code 擴充套件來使用 OpenAI Codex 進行 開發與完成此專案。

最後，產生一個 markdown 檔案，告訴 OpenAI Codex，這個專案的目標、需求、開發工具與環境設定，以及每一個步驟的詳細說明，讓我可以順利地使用 OpenAI Codex 來開發這個病人時序圖的前端應用程式。
```

* 在這裡，將會建立一個檔案 [Codex_Blazor_Patient_Timeline_Implementation_Guide_zh-TW.md]
* 底下將會是這個檔案內容

```markdown

# 使用 OpenAI Codex 開發「FHIR 病人時序圖」前端系統完整操作指引（ASP.NET Core Blazor）

> 適用對象：第一次使用 OpenAI Codex / VS Code ChatGPT 擴充套件的開發者  
> 專案型態：ASP.NET Core Blazor（Server Side Interactive Rendering，採 **Global** 互動模式，非 per-page）  
> 後端資料來源：Firely Server（FHIR R4 API，`https://server.fire.ly`）

---

## 1. 專案目標（Project Goal）

建立一個「病人時序圖」前端應用程式，讓使用者可輸入病人 ID（例如：`129c6ac7-8d06-89de-ad63-0204a93e76c3`），並從 Firely Server 取得該病人的 FHIR 資料，彙整成可視化時間軸（Timeline），支援：

1. 指定時間區間（起迄日期）查詢病人就醫紀錄。
2. 顯示病人的綜合摘要（Summary）。
3. 以 Encounter 為主軸，串連同次或相近時段的臨床事件（Condition、Observation、Procedure、MedicationRequest…）。
4. 點擊任一就醫紀錄後，顯示詳細資訊（醫療機構、醫療人員、診斷報告、文件參考等）。

---

## 2. 功能需求（Functional Requirements）

### 2.1 主要查詢流程

- 使用者輸入 Patient ID。
- 系統向 Firely Server 查詢 Patient 基本資料。
- 以 Patient 為條件查詢：
  - Encounter
  - Condition
  - AllergyIntolerance
  - Immunization
  - Device
  - Observation
  - Procedure
  - DiagnosticReport
  - DocumentReference
  - MedicationRequest
- 前端將資料轉換為「時間軸事件模型」，依時間排序顯示。

### 2.2 明細檢視

- 點擊任一 Encounter（或事件節點）開啟詳細面板：
  - 就診日期、醫療機構（Organization / Location）
  - 醫療人員與角色（Practitioner / PractitionerRole）
  - 對應疾病、觀察、處置、檢查報告、用藥等

### 2.3 非功能需求

- UI 需可處理資料量偏大的病人（例如 Synthea 產生的長期病史）。
- 支援錯誤處理（找不到病人、FHIR API timeout、部分資源缺漏）。
- 使用可重複維護的分層設計（UI / Service / Mapping）。

---

## 3. 技術選型與架構（Tech Stack & Architecture）

## 3.1 技術選型

- .NET 8 SDK（建議）
- ASP.NET Core Blazor Web App
- Render mode：**Interactive Server（Global）**
- FHIR SDK：`Hl7.Fhir.R4`（或依目標版本選擇）
- HTTP：`HttpClientFactory`
- 開發工具：VS Code + C# + ChatGPT 擴充套件（OpenAI Codex 協作）

### 3.2 建議分層

- `Pages/`：查詢頁、時間軸頁
- `Components/`：Timeline、EventCard、SummaryPanel、DetailDrawer
- `Services/`：FHIR 查詢與聚合邏輯
- `Models/`：ViewModel / DTO（TimelineEvent、PatientSummary）
- `Mapping/`：FHIR Resource → UI Model

---

## 4. 開發環境安裝與設定（一步一步）

## 4.1 必要安裝清單

1. 安裝 .NET SDK 8
2. 安裝 VS Code
3. 安裝 VS Code Extensions：
   - C#（ms-dotnettools.csharp）
   - C# Dev Kit（建議）
   - ChatGPT（你已安裝）
4. 安裝 Git

## 4.2 驗證安裝

在終端機執行：

```bash
dotnet --version
code --version
git --version
```

若均正常輸出版本號，即可進入下一步。

---

## 5. 建立專案（Blazor + Global Interactive Server）

在專案資料夾執行：

```bash
dotnet new blazor -n PatientTimelineBlazor
cd PatientTimelineBlazor
```

> 若範本支援參數，可使用互動模式相關選項；若未提供，後續手動設定。

### 5.1 設定 Global Interactive Server

在 `Program.cs` 概念上需確保：

```csharp
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents();

app.MapRazorComponents<App>()
    .AddInteractiveServerRenderMode();
```

這代表全域啟用 Interactive Server，而非每頁 individually 指定。

---

## 6. Firely Server/FHIR 連線策略

### 6.1 基本 URL

- Base URL：`https://server.fire.ly`
- Patient 例：`GET /Patient/129c6ac7-8d06-89de-ad63-0204a93e76c3`

### 6.2 建議查詢方式

先查病人，再依病人關聯查詢資源：

- `Encounter?patient={id}`
- `Condition?patient={id}`
- `Observation?patient={id}`
- `Procedure?patient={id}`
- `MedicationRequest?patient={id}`
- 其他同理

### 6.3 CORS 注意事項

若直接由瀏覽器呼叫 Firely Server 遭遇 CORS，建議：

- 由 Blazor Server 端（伺服器端）發送 FHIR API 請求，再回傳 UI。
- 避免將第三方 API 呼叫留在瀏覽器端。

---

## 7. 實作步驟（可直接給 Codex 的工作拆解）

以下每一階段都可以用「小步驟 + 可驗證輸出」方式讓 Codex 逐步完成。

### Step 1：建立骨架與路由

- 建立頁面：
  - `Pages/PatientTimeline.razor`
- 建立元件：
  - `Components/Timeline/TimelineView.razor`
  - `Components/Timeline/TimelineEventCard.razor`
  - `Components/Timeline/EncounterDetailPanel.razor`

**驗收標準**：可進入 `/patient-timeline` 頁面，看到 Patient ID 輸入框與查詢按鈕。

### Step 2：FHIR 查詢服務

- 建立 `IFhirDataService` / `FhirDataService`。
- 封裝方法：
  - `GetPatientAsync(patientId)`
  - `GetEncountersAsync(patientId, from, to)`
  - `GetConditionsAsync(patientId, from, to)`
  - …（其餘資源）

**驗收標準**：可在查詢後列印資源筆數（先文字顯示即可）。

### Step 3：資料聚合（Timeline Mapping）

- 建立 `TimelineEvent` 模型（DateTime、Type、Title、ReferenceIds、EncounterId、Details）。
- 將各資源投影成統一事件模型。
- 依日期排序後輸出。

**驗收標準**：畫面可顯示時間排序的事件清單。

### Step 4：摘要與細節面板

- `PatientSummary`：年齡、性別、慢性病摘要、最近就醫日期。
- 點擊事件顯示 detail drawer/modal。

**驗收標準**：點擊某筆 Encounter 可看到該次就醫關聯資料。

### Step 5：區間篩選與錯誤處理

- 加入 From/To 日期篩選。
- 處理 404、timeout、空資料。
- 顯示 loading/skeleton。

**驗收標準**：指定區間後資料會變更，錯誤情境有友善提示。

### Step 6：視覺優化與可維護性

- 類型顏色標示（Condition/Observation/Medication…）。
- 統一元件樣式與命名。
- 補上 README 與操作說明。

**驗收標準**：畫面可讀性佳、結構清楚、便於後續擴充。

---

## 8. 在 VS Code 使用 OpenAI Codex 的實際操作流程

> 你已安裝 ChatGPT 擴充套件，可直接在 VS Code 內用對話方式驅動開發。

### 8.1 建議工作模式

1. 先手動建立最小專案骨架。
2. 每次給 Codex 一個明確、可驗證的小任務。
3. 要求 Codex 回覆「修改哪些檔案 + 驗證指令」。
4. 你執行測試後，再給下一步。

### 8.2 Prompt 模板（可直接貼）

#### 模板 A：建立服務層

```text
你是資深 ASP.NET Core Blazor 工程師。
請在目前專案中新增 FHIR 查詢服務層，需求：
1) 建立 IFhirDataService 與 FhirDataService
2) 透過 HttpClient 呼叫 https://server.fire.ly
3) 實作 GetPatientAsync、GetEncountersAsync、GetConditionsAsync、GetObservationsAsync
4) 加入基本錯誤處理與 cancellation token
5) 更新 DI 註冊
請直接產生可編譯程式碼，並列出你修改的檔案與原因。
```

#### 模板 B：建立時間軸 UI

```text
請在 Blazor Server (global interactive) 專案中建立病人時序圖頁面：
1) Patient ID 輸入欄位 + 查詢按鈕
2) 日期區間篩選
3) Timeline 元件（按日期排序）
4) 點擊事件顯示 Detail Panel
5) 使用可重用元件拆分
請以最小可用版本完成，並附上每個元件用途。
```

#### 模板 C：重構與品質提升

```text
請檢查目前程式碼並進行重構：
1) 移除重複邏輯
2) 補上 XML 註解
3) 改善命名一致性
4) 增加單元測試（至少涵蓋 mapping）
5) 確保 dotnet build 與 dotnet test 可通過
請提供變更摘要與測試結果。
```

### 8.3 對 Codex 下指令的技巧

- 一次只做一件事（例如「先做 service，不要動 UI」）。
- 在 prompt 中寫出「完成定義（Definition of Done）」。
- 要求「不要改動無關檔案」。
- 每次完成都執行：
  - `dotnet build`
  - `dotnet test`（若已有測試）

---

## 9. 建議開發里程碑（Milestones）

- M1：可以輸入 Patient ID 並顯示基本資料。
- M2：可載入 Encounter 並顯示時間序。
- M3：可串聯 Condition/Observation/Procedure/MedicationRequest。
- M4：摘要 + 詳細面板完成。
- M5：日期篩選、錯誤處理、效能優化。
- M6：文件化與部署準備。

---

## 10. 驗證與測試建議

### 10.1 最小驗證清單

1. Patient ID 合法時可取得資料。
2. Patient ID 不存在時顯示明確錯誤。
3. 日期區間可正確過濾。
4. 點擊事件可看到正確明細。
5. 大量資料仍可順暢瀏覽（必要時分頁/虛擬化）。

### 10.2 命令列檢查

```bash
dotnet restore
dotnet build
dotnet test
```

---

## 11. 風險與排除建議

- **CORS/跨網域問題**：改由 Blazor Server 後端代理請求。
- **FHIR 資源關聯不完整**：Mapping 層需容忍 null 與缺漏 reference。
- **資料過大造成 UI 卡頓**：使用分段載入、虛擬清單、延遲展開明細。
- **Prompt 太大導致 Codex 回覆失焦**：拆小任務、逐步整合。

---

## 12. 建議專案初始化命令（範例）

```bash
# 1) 建立專案
dotnet new blazor -n PatientTimelineBlazor
cd PatientTimelineBlazor

# 2) 新增常用套件（依需求調整）
dotnet add package Hl7.Fhir.R4

# 3) 啟動
DOTNET_ENVIRONMENT=Development dotnet run

# 4) 驗證建置
dotnet build
```

---

## 13. 交付物定義（你最終應該擁有）

1. 可執行的 Blazor Server（Global Interactive）專案。
2. 可輸入 Patient ID 並查詢 Firely FHIR 的時序圖頁面。
3. 時序圖事件點選後可看完整細節。
4. 專案文件（README + 本指引檔）。
5. 基本測試與建置驗證流程。

---

## 14. 給 Codex 的「專案總指令」範本（可貼在新對話第一句）

```text
請協助我開發一個 ASP.NET Core Blazor Web App（Interactive Server Global）專案：
- 專案名稱：PatientTimelineBlazor
- 目標：輸入 Patient ID 後，從 https://server.fire.ly 取得病人與相關 FHIR 資源，顯示病人時序圖與摘要
- 資源包含：Encounter、Condition、AllergyIntolerance、Immunization、Device、Observation、Procedure、DiagnosticReport、DocumentReference、MedicationRequest，以及 Organization/Location/Practitioner/PractitionerRole 的關聯資訊
- 要求：
  1) 以服務層封裝 FHIR 查詢
  2) 建立 Timeline 映射模型
  3) 使用元件化 UI 呈現時間軸與明細面板
  4) 支援日期區間篩選、錯誤處理、loading 狀態
  5) 每完成一步都提供：修改檔案清單、程式碼說明、驗證命令
請先從最小可用版本開始，並確保 dotnet build 可通過。
```

---

* 接著，回到 OpenAI Codex 
* 輸入這樣的文字 `建立這個專案的 README.md，內容就是上面這個 markdown 的內容`
* 經過一些時間，就發現到這個 ASP.NET Core Blazor 專案已經建立完成了
* 由於對於這些工具不太孰悉，花了一些時間來熟悉這些工具的使用方式
* 第一次建立的專案發現到建置上的錯誤與問題
* 我將這些錯誤訊息貼到 OpenAI Codex 的對話框裡面，讓 OpenAI Codex 來幫我分析這些錯誤訊息，並且給我一些建議的解決方案
* 經過這次之後，我得到一個正確的 Blazor 專案骨架，並且可以成功地建置與執行這個專案了
* 現在執行這個專案，看看是否能夠成功地從 Firely Server 上獲取到指定病人的相關 FHIR 資料，並且將這些資料以時序圖的形式展示出來，看看整個系統的功能是否正常運作，以及使用體驗是否良好。
* 打開瀏覽器，將會看到底下畫面
![](../Images/cs2025-851.png)
* 點擊 [Patient Timeline] 連結，將會看到這個畫面
* 由於病歷號已經預設在畫面上了，這裡將僅輸入開始與結束時間
* 點擊 [Search] 按鈕之後，將會出現這個畫面
![](../Images/cs2025-850.png)
