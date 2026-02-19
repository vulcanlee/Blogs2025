# 匯入 FHIR 範例資料集

**[Synthea™ Patient Generator (GitHub)](https://github.com/synthetichealth/synthea?utm_source=chatgpt.com)** 是一個 **開源的合成病人（synthetic patient）資料生成器（Synthetic Patient Population Simulator）**，用來在不涉及真實個人隱私資料的情況下，**模擬完整的病人健康歷程與電子健康紀錄（EHR）**。

合成患者（synthetic patient）是由電腦模型生成的虛構個體，**看起來像真實患者但不對應任何真實個人**。這種資料不含任何實際個人資訊，因此沒有隱私風險。Synthea 的使命是生成：**逼真但不是真實的健康資料（realistic synthetic data）**、* **涵蓋病人從出生到死亡的完整醫療歷程**、* **各種與健康照護相關的資料，包括症狀、診斷、處方、檢驗、就診事件等**，  生成的資料可用於研究、測試、教育和開發，而不受 HIPAA 或其他隱私法律限制。

🛠 主要特色與功能
📌 **全生命周期模擬**
從出生到死亡，逐步模擬個體醫療事件及健康狀態。
📌 **可配置的人口統計與統計模型**
預設使用馬薩諸塞州人口統計資料，也可以配置其他區域或人口設定。
📌 **模組化規則系統**
採用「疾病模組（disease modules）」和通用規則框架（Generic Module Framework）來模擬疾病發生與演進。這些規則參考公開資料與統計資料（如 CDC、NIH）。
📌 **輸出格式多樣性**
支援多種業界標準與格式輸出，包括：
* **HL7 FHIR**（R4、STU3、DSTU2）
* **C-CDA、CSV、Bulk FHIR（ndjson）**

## 下載與使用 Synthea 生成 FHIR 資料集

* 打開 [Sample FHIR Bulk Export Datasets](https://github.com/smart-on-fhir/sample-bulk-fhir-datasets) 的 GitHub 頁面
* 找到 [Small] 連結，點選這個連結
* 如此，將會下載 10 個模擬病人資料集，這些資料集以 ndjson 格式存儲，並且包含了 FHIR 資源的完整結構，可以用來測試和開發 FHIR API 的功能。
* 打開並且解壓縮這個 [sample-bulk-fhir-datasets-10-patients.zip] 檔案
* 這個檔案內將會有底下的FHIR資源檔案：
   *Organization ： 模擬的醫療機構資料。
   *Location ： 模擬的醫療機構地點資料。
   *Practitioner ： 模擬的醫療人員資料。
   *PractitionerRole ： 模擬的醫療人員角色資料。
   *Patient ： 模擬的病人資料。
   *Encounter ： 模擬的就診事件資料。
   *Condition ： 模擬的疾病或健康狀態資料。
   *AllergyIntolerance ： 模擬的過敏或不耐受資料。
   *Immunization ： 模擬的疫苗接種資料。
   *Device ： 模擬的醫療設備資料。
   *Observation ： 模擬的觀察資料。
   *Procedure ： 模擬的醫療程序資料。
   *DiagnosticReport ： 模擬的診斷報告資料。
   *DocumentReference ： 模擬的文件參考資料。
   *MedicationRequest ： 模擬的用藥請求資料。

* 在這個目錄下，建立一個 PowerShell 檔案 [import.ps1]
* 在這個檔案內輸入底下的程式碼：

```powershell
param(
  [string]$Base   = "https://hapi.fhir.org/baseR4",
  [string]$Folder = "."
)

$ErrorActionPreference = "Stop"

# 判斷物件是否「有內容」（"不是空物件/空陣列/空字串/null）
function Test-HasContent {
  param($Value)

  if ($null -eq $Value) { return $false }

  # string
  if ($Value -is [string]) {
    return -not [string]::IsNullOrWhiteSpace($Value)
  }

  # array
  if ($Value -is [System.Collections.IEnumerable] -and -not ($Value -is [PSCustomObject])) {
    # 注意：string 也是 IEnumerable，上面已處理
    try {
      $arr = @($Value)
      return ($arr.Count -gt 0)
    } catch {
      return $true
    }
  }

  # object (PSCustomObject)
  if ($Value -is [PSCustomObject] -or $Value -is [hashtable]) {
    $props = $Value.PSObject.Properties
    if ($props.Count -eq 0) { return $false }

    foreach ($p in $props) {
      if (Test-HasContent $p.Value) { return $true }
    }
    return $false
  }

  # other primitive types
  return $true
}

# 清洗 FHIR JSON：
# - 移除 extension（synthea 常見 unknown extension）
# - 移除 meta.profile；若 meta 變空 => 移除整個 meta（避免 dom-6 / meta empty）
function Clean-FhirJsonLine {
  param([string]$JsonLine)

  $obj = $JsonLine | ConvertFrom-Json

  # 1) 移除 extension
  if ($obj.PSObject.Properties.Name -contains "extension") {
    $obj.PSObject.Properties.Remove("extension")
  }

  # 2) meta: 移除 profile；meta 若空就整個移除
  if ($obj.PSObject.Properties.Name -contains "meta") {
    $meta = $obj.meta

    if ($meta -and ($meta.PSObject.Properties.Name -contains "profile")) {
      $meta.PSObject.Properties.Remove("profile")
    }

    # 有些資料可能還留 profile: []，確保刪乾淨
    if ($meta -and ($meta.PSObject.Properties.Name -contains "profile")) {
      $meta.PSObject.Properties.Remove("profile")
    }

    # 若 meta 沒有任何有效內容，移除 meta
    if (-not (Test-HasContent $meta)) {
      $obj.PSObject.Properties.Remove("meta")
    }
  }

  # 3) 可選：移除空的 text（某些 server 對空 narrative 也會抱怨）
  if ($obj.PSObject.Properties.Name -contains "text") {
    if (-not (Test-HasContent $obj.text)) {
      $obj.PSObject.Properties.Remove("text")
    }
  }

  return ($obj | ConvertTo-Json -Depth 100 -Compress)
}

# PUT 單筆 Resource（保留 client-assigned id）
function Invoke-FhirPut {
  param(
    [string]$Url,
    [string]$JsonLine
  )

  $tmp = Join-Path $env:TEMP ("fhir_import_" + [Guid]::NewGuid().ToString("N") + ".json")

  # ✅ 用 UTF-8 無 BOM 寫入（避免 HAPI-0450 / HAPI-1861）
  $utf8NoBom = New-Object System.Text.UTF8Encoding($false)
  [System.IO.File]::WriteAllText($tmp, $JsonLine, $utf8NoBom)

  try {
    $output = & curl.exe -sS `
      -X PUT $Url `
      -H "Content-Type: application/fhir+json; charset=utf-8" `
      --data-binary "@$tmp" `
      -w "`n%{http_code}"

    $lines = $output -split "`n"
    $http  = ($lines[-1]).Trim()
    $body  = ($lines[0..($lines.Length-2)] -join "`n").Trim()

    return @{
      Http = $http
      Body = $body
    }
  }
  finally {
    Remove-Item -Force -ErrorAction SilentlyContinue $tmp
  }
}

# 匯入單一 ndjson 檔
function Import-NdjsonFile {
  param([string]$Path)

  $fileName     = [System.IO.Path]::GetFileName($Path)
  $resourceType = $fileName.Split(".")[0]

  Write-Host "==> Importing $fileName -> $resourceType"

  $i = 0
  Get-Content -Path $Path -ReadCount 1 | ForEach-Object {
    $line = $_
    if ([string]::IsNullOrWhiteSpace($line)) { return }
    $i++

    try {
      $clean = Clean-FhirJsonLine -JsonLine $line
      $obj   = $clean | ConvertFrom-Json
    } catch {
      throw "Invalid JSON at $fileName line $i"
    }

    $id = $obj.id
    if ([string]::IsNullOrWhiteSpace($id)) {
      throw "Missing id at $fileName line $i"
    }

    $url  = "$Base/$resourceType/$id"
    $resp = Invoke-FhirPut -Url $url -JsonLine $clean

    if ($resp.Http -ne "200" -and $resp.Http -ne "201") {
      Write-Host ""
      Write-Host "FAILED: $fileName line $i => HTTP $($resp.Http)"
      if (-not [string]::IsNullOrWhiteSpace($resp.Body)) {
        Write-Host "Response body (first 1600 chars):"
        Write-Host ($resp.Body.Substring(0, [Math]::Min(1600, $resp.Body.Length)))
      } else {
        Write-Host "Response body: <empty>"
      }
      throw "Import failed."
    }

    if (($i % 200) -eq 0) {
      Write-Host "  ...$i resources imported"
    }
  }

  Write-Host "    OK ($i resources)"
}

# 匯入順序（先被參考者）
$order = @(
  "Organization
  "Location
  "Practitioner
  "PractitionerRole
  "Patient
  "Encounter
  "Condition
  "AllergyIntolerance
  "Immunization
  "Device
  "Observation
  "Procedure
  "DiagnosticReport
  "DocumentReference
  "MedicationRequest*.ndjson"
)

foreach ($pattern in $order) {
  $files = Get-ChildItem -Path $Folder -Filter $pattern -File | Sort-Object Name
  if ($files.Count -eq 0) {
    Write-Host "Skip (not found): $pattern"
    continue
  }

  foreach ($f in $files) {
    Import-NdjsonFile -Path $f.FullName
  }
}

Write-Host ""
Write-Host "=============================="
Write-Host "FHIR NDJSON IMPORT COMPLETED"
Write-Host "=============================="
```

* 使用底下命令來執行這個 PowerShell 腳本

```shell
powershell -ExecutionPolicy Bypass -File .\import.ps1 -Base "https://server.fire.ly" -Folder "."
```

*將會把這些 FHIR 資源逐一匯入到指定的 FHIR Server（預設是 https://hapi.fhir.org/baseR4）。腳本會自動處理 JSON 清洗（移除不必要的 extension 和 meta.profile）並且使用 HTTP PUT 方法來上傳資源，保留原有的 client-assigned id。匯入過程中會顯示進度和任何可能的錯誤訊息。



