# FHIR 11 如何對FhirClient的API呼叫，看到實際的 HTTP Payload

對於.NET的開發者，想要存取 FHIR Server上的資料，透過 FhirClient 物件來操作，是最為方便與簡潔的作法，想要使用這個物件，在 .NET 下主要需要安裝 Firely SDK（前身是 FHIR .NET API），在這裡的例子將會安裝了 Hl7.Fhir.R4 套件。

一旦安裝了安裝了 Hl7.Fhir.R4 套件之後，就可以在程式碼中使用 FhirClient 物件來對 FHIR Server 進行各種操作，例如讀取、查詢、更新、刪除等。可以想像你要去醫院查病歷資料，FhirClient 就像是一個「翻譯員 + 信差」：

* 翻譯員角色：把你的請求（「我要查某個病人的資料」）翻譯成 FHIR 伺服器能懂的語言
* 信差角色：幫你把請求送到伺服器，再把結果帶回來給你

因此，面對 FHIR Server  中超過上百個 Resource，操作起來更加輕鬆雨容易，這裡可以做到這些實際用途，例如：讀取病人資料（Patient）、查詢檢驗結果（Observation）、取得用藥記錄（Medication）、新增或更新醫療資料等。

就技術上來說，FhirClient 是一個程式庫（library），封裝了 HTTP 請求、資料格式轉換、錯誤處理等複雜細節，讓開發者可以用簡單的程式碼就能存取 FHIR 醫療資料，不用自己處理那些繁瑣的通訊協定和資料格式。簡單來說，它就是讓程式開發者能輕鬆存取 FHIR 醫療資料標準系統的工具。

有些時候，開發者可能會想要知道 FhirClient 在背後實際傳送了什麼 HTTP 請求，或是伺服器回傳了什麼樣的 HTTP 回應，這對於除錯、效能優化、了解系統行為等都很有幫助。甚至可以透過這些 HTTP 請求與回應，更深入的取學習與理解 FHIR API 的用法。

## 建立 Http 管道處理器

* 延續 []() 文章中做做出的程式碼
* 在 [Program.cs] 檔案中找到 `namespace csPatientCRUD;`，在其下方加入這個類別定義  

```csharp
public class HttpLoggingHandler : DelegatingHandler
{
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
    {
        var startTime = DateTime.UtcNow;

        // 記錄請求
        Console.WriteLine("===== HTTP 請求 =====");
        Console.WriteLine($"{request.Method} {request.RequestUri}");
        Console.WriteLine("標頭:");
        foreach (var header in request.Headers)
        {
            Console.WriteLine($"  {header.Key}: {string.Join(", ", header.Value)}");
        }

        if (request.Content != null)
        {
            foreach (var header in request.Content.Headers)
            {
                Console.WriteLine($"  {header.Key}: {string.Join(", ", header.Value)}");
            }

            var requestBody = await request.Content.ReadAsStringAsync(cancellationToken);
            if (!string.IsNullOrEmpty(requestBody))
            {
                Console.WriteLine("請求內容:");
                Console.WriteLine(requestBody);
            }
        }

        // 發送請求
        var response = await base.SendAsync(request, cancellationToken);

        var duration = DateTime.UtcNow - startTime;

        // 記錄回應
        Console.WriteLine("\n===== HTTP 回應 =====");
        Console.WriteLine($"狀態: {(int)response.StatusCode} {response.StatusCode}");
        Console.WriteLine("標頭:");
        foreach (var header in response.Headers)
        {
            Console.WriteLine($"  {header.Key}: {string.Join(", ", header.Value)}");
        }

        if (response.Content != null)
        {
            foreach (var header in response.Content.Headers)
            {
                Console.WriteLine($"  {header.Key}: {string.Join(", ", header.Value)}");
            }

            var responseBody = await response.Content.ReadAsStringAsync(cancellationToken);
            if (!string.IsNullOrEmpty(responseBody))
            {
                Console.WriteLine("回應內容:");
                Console.WriteLine(responseBody);
            }
        }

        Console.WriteLine($"耗時: {duration.TotalMilliseconds:F2} ms");
        Console.WriteLine("".PadRight(50, '='));

        return response;
    }
}
```

* [DelegatingHandler 類別] (https://learn.microsoft.com/zh-tw/dotnet/api/system.net.http.delegatinghandler?view=net-8.0) 是 .NET 提供的一個抽象類別，可以用來建立自訂的 HTTP 處理器（Handler）。這些處理器可以用來攔截、修改、記錄或處理 HTTP 請求和回應。透過繼承 DelegatingHandler，開發者可以實現自己的邏輯，並將其插入到 HTTP 請求管道中。
* 這裡新建立的 [HttpLoggingHandler] 類別將會繼承了這個 [DelegatingHandler] 類別，因此，可以用來攔截 HTTP 請求與回應，並記錄相關的資訊，例如請求方法、URL、標頭、內容，以及回應的狀態碼、標頭、內容等。
* 此類別覆寫了 [SendAsync] 方法，這個方法會在每次發送 HTTP 請求時被呼叫。
* 在這個方法中，我們先記錄請求的詳細資訊，然後呼叫 `base.SendAsync` 方法來發送請求，接著再記錄回應的詳細資訊。最後，將回應物件返回。
* 這裡也會將 HTTP Header 的資訊與內容（Content）都記錄下來，方便後續查看。
* 當 [SendAsync] 方法被呼叫後，會得到一個 [HttpResponseMessage] 物件，代表伺服器的回應。
* 有了這個物件，便可以取得此次 HTTP 回應的原始內容，例如狀態碼、標頭、內容等，並將這些資訊記錄下來。
* 最後，這個類別也會計算出每次呼叫 FHIR API 需要花費的時間成本。
* 這樣一來，每次透過 FhirClient 發送的 HTTP 請求與回應，都會被這個處理器攔截並記錄，方便開發者查看實際的 HTTP Payload。

## 在 FhirClient 中使用 HttpLoggingHandler

* 在 [Program.cs] 檔案中，找到 `var httpClient = new HttpClient();` 這一行程式碼，將其修改為以下內容：

```csharp
var httpHandler = new HttpClientHandler();
var loggingHandler = new HttpLoggingHandler { InnerHandler = httpHandler };
var httpClient = new HttpClient(loggingHandler);
```

* 這段程式碼中，我們先建立了一個 [HttpClientHandler] 物件，這是 .NET 提供的預設 HTTP 處理器，負責處理實際的 HTTP 通訊。
* 接著，我們建立了一個 [HttpLoggingHandler] 物件，並將剛剛建立的 [HttpClientHandler] 設定為它的 [InnerHandler]，這樣當 [HttpLoggingHandler] 收到請求時，就會將請求傳遞給內部的 [HttpClientHandler] 來處理。
* 最後，我們使用這個 [HttpLoggingHandler] 來建立 [HttpClient] 物件，這樣所有透過這個 [HttpClient] 發送的請求，都會先經過我們的日誌處理器，從而記錄下詳細的 HTTP 請求與回應資訊。
* 這樣一來，當我們使用 FhirClient 進行各種操作時，例如讀取病人資料、查詢檢驗結果等，都會觸發我們的 [HttpLoggingHandler]，從而在控制台中看到詳細的 HTTP 請求與回應內容，方便我們進行除錯與分析。

## 測試 FhirClient 的 API 呼叫

* 執行 [csPatientCRUD] 專案，觀察控制台輸出

```
Creating Patient ...
JSON: {"resourceType":"Patient","identifier":[{"system":"http://example.org/mrn","value":"MRN-20240814A1"}],"active":true,"name":[{"family":"Lee","given":["Vulcan20250814111"]}],"telecom":[{"system":"phone","value":"0912-345-678","use":"mobile"}],"gender":"female","birthDate":"1990-01-01"}
===== HTTP 請求 =====
POST https://hapi.fhir.org/baseR4/Patient
標頭:
  Accept: application/fhir+json
  Accept-Charset: utf-8
  User-Agent: firely-sdk-client/5.12.1
  Content-Type: application/fhir+json; charset=utf-8
請求內容:
{"resourceType":"Patient","identifier":[{"system":"http://example.org/mrn","value":"MRN-20240814A1"}],"active":true,"name":[{"family":"Lee","given":["Vulcan20250814111"]}],"telecom":[{"system":"phone","value":"0912-345-678","use":"mobile"}],"gender":"female","birthDate":"1990-01-01"}

===== HTTP 回應 =====
狀態: 201 Created
標頭:
  Server: nginx/1.24.0, (Ubuntu)
  Date: Tue, 13 Jan 2026 08:29:11 GMT
  Transfer-Encoding: chunked
  Connection: keep-alive
  X-Powered-By: HAPI FHIR 8.5.3-SNAPSHOT/e3a3c5f741/2025-08-28 REST Server (FHIR Server; FHIR 4.0.1/R4)
  ETag: W/"1"
  X-Request-ID: I9kYzrNbqdbmsueL
  Location: https://hapi.fhir.org/baseR4/Patient/53805146/_history/1
  Content-Type: application/fhir+json; charset=utf-8
  Content-Location: https://hapi.fhir.org/baseR4/Patient/53805146/_history/1
  Last-Modified: Tue, 13 Jan 2026 08:29:11 GMT
回應內容:
{
  "resourceType": "Patient",
  "id": "53805146",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2026-01-13T08:29:11.623+00:00",
    "source": "#I9kYzrNbqdbmsueL"
  },
  "text": {
    "status": "generated",
    "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\"><div class=\"hapiHeaderText\">Vulcan20250814111 <b>LEE </b></div><table class=\"hapiPropertyTable\"><tbody><tr><td>Identifier</td><td>MRN-20240814A1</td></tr><tr><td>Date of birth</td><td><span>01 January 1990</span></td></tr></tbody></table></div>"
  },
  "identifier": [ {
    "system": "http://example.org/mrn",
    "value": "MRN-20240814A1"
  } ],
  "active": true,
  "name": [ {
    "family": "Lee",
    "given": [ "Vulcan20250814111" ]
  } ],
  "telecom": [ {
    "system": "phone",
    "value": "0912-345-678",
    "use": "mobile"
  } ],
  "gender": "female",
  "birthDate": "1990-01-01"
}
耗時: 1122.91 ms
==================================================
Created: id=53805146, version=1
Press any key to continue...
 Reading Patient by id ...
===== HTTP 請求 =====
GET https://hapi.fhir.org/baseR4/Patient/53805146
標頭:
  Accept: application/fhir+json
  Accept-Charset: utf-8
  User-Agent: firely-sdk-client/5.12.1

===== HTTP 回應 =====
狀態: 200 OK
標頭:
  Server: nginx/1.24.0, (Ubuntu)
  Date: Tue, 13 Jan 2026 08:29:14 GMT
  Transfer-Encoding: chunked
  Connection: keep-alive
  X-Powered-By: HAPI FHIR 8.5.3-SNAPSHOT/e3a3c5f741/2025-08-28 REST Server (FHIR Server; FHIR 4.0.1/R4)
  ETag: W/"1"
  X-Request-ID: 6eD6YAfNWkKzdFl5
  Content-Type: application/fhir+json; charset=utf-8
  Content-Location: https://hapi.fhir.org/baseR4/Patient/53805146/_history/1
  Last-Modified: Tue, 13 Jan 2026 08:29:11 GMT
回應內容:
{
  "resourceType": "Patient",
  "id": "53805146",
  "meta": {
    "versionId": "1",
    "lastUpdated": "2026-01-13T08:29:11.623+00:00",
    "source": "#I9kYzrNbqdbmsueL"
  },
  "text": {
    "status": "generated",
    "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\"><div class=\"hapiHeaderText\">Vulcan20250814111 <b>LEE </b></div><table class=\"hapiPropertyTable\"><tbody><tr><td>Identifier</td><td>MRN-20240814A1</td></tr><tr><td>Date of birth</td><td><span>01 January 1990</span></td></tr></tbody></table></div>"
  },
  "identifier": [ {
    "system": "http://example.org/mrn",
    "value": "MRN-20240814A1"
  } ],
  "active": true,
  "name": [ {
    "family": "Lee",
    "given": [ "Vulcan20250814111" ]
  } ],
  "telecom": [ {
    "system": "phone",
    "value": "0912-345-678",
    "use": "mobile"
  } ],
  "gender": "female",
  "birthDate": "1990-01-01"
}
耗時: 200.41 ms
==================================================
Read: Vulcan20250814111 Lee | active=True
Press any key to continue...
 Updating Patient (add email, set active=false) ...
===== HTTP 請求 =====
PUT https://hapi.fhir.org/baseR4/Patient/53805146
標頭:
  Accept: application/fhir+json
  Accept-Charset: utf-8
  User-Agent: firely-sdk-client/5.12.1
  Content-Type: application/fhir+json; charset=utf-8
  Last-Modified: Tue, 13 Jan 2026 08:29:11 GMT
請求內容:
{"resourceType":"Patient","id":"53805146","meta":{"versionId":"1","lastUpdated":"2026-01-13T08:29:11.623+00:00","source":"#I9kYzrNbqdbmsueL"},"text":{"status":"generated","div":"<div xmlns=\"http://www.w3.org/1999/xhtml\"><div class=\"hapiHeaderText\">Vulcan20250814111 <b>LEE </b></div><table class=\"hapiPropertyTable\"><tbody><tr><td>Identifier</td><td>MRN-20240814A1</td></tr><tr><td>Date of birth</td><td><span>01 January 1990</span></td></tr></tbody></table></div>"},"identifier":[{"system":"http://example.org/mrn","value":"MRN-20240814A1"}],"active":false,"name":[{"family":"Lee","given":["Vulcan20250814111"]}],"telecom":[{"system":"phone","value":"0912-345-678","use":"mobile"},{"system":"email","value":"Vulcan20250814111.Lee@example.org"}],"gender":"female","birthDate":"1990-01-01"}

===== HTTP 回應 =====
狀態: 200 OK
標頭:
  Server: nginx/1.24.0, (Ubuntu)
  Date: Tue, 13 Jan 2026 08:29:17 GMT
  Transfer-Encoding: chunked
  Connection: keep-alive
  X-Powered-By: HAPI FHIR 8.5.3-SNAPSHOT/e3a3c5f741/2025-08-28 REST Server (FHIR Server; FHIR 4.0.1/R4)
  ETag: W/"2"
  X-Request-ID: etoip1zPped1yLq4
  Content-Type: application/fhir+json; charset=utf-8
  Content-Location: https://hapi.fhir.org/baseR4/Patient/53805146/_history/2
  Last-Modified: Tue, 13 Jan 2026 08:29:17 GMT
回應內容:
{
  "resourceType": "Patient",
  "id": "53805146",
  "meta": {
    "versionId": "2",
    "lastUpdated": "2026-01-13T08:29:17.196+00:00",
    "source": "#etoip1zPped1yLq4"
  },
  "text": {
    "status": "generated",
    "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\"><div class=\"hapiHeaderText\">Vulcan20250814111 <b>LEE </b></div><table class=\"hapiPropertyTable\"><tbody><tr><td>Identifier</td><td>MRN-20240814A1</td></tr><tr><td>Date of birth</td><td><span>01 January 1990</span></td></tr></tbody></table></div>"
  },
  "identifier": [ {
    "system": "http://example.org/mrn",
    "value": "MRN-20240814A1"
  } ],
  "active": false,
  "name": [ {
    "family": "Lee",
    "given": [ "Vulcan20250814111" ]
  } ],
  "telecom": [ {
    "system": "phone",
    "value": "0912-345-678",
    "use": "mobile"
  }, {
    "system": "email",
    "value": "Vulcan20250814111.Lee@example.org"
  } ],
  "gender": "female",
  "birthDate": "1990-01-01"
}
耗時: 232.04 ms
==================================================
Updated: version=2, telecom=Phone:0912-345-678, Email:Vulcan20250814111.Lee@example.org
Press any key to continue...
 Searching Patient by identifier 'MRN-20240814A1' ...
===== HTTP 請求 =====
GET https://hapi.fhir.org/baseR4/Patient?_count=5&identifier=MRN-20240814A1
標頭:
  Accept: application/fhir+json
  Accept-Charset: utf-8
  User-Agent: firely-sdk-client/5.12.1

===== HTTP 回應 =====
狀態: 200 OK
標頭:
  Server: nginx/1.24.0, (Ubuntu)
  Date: Tue, 13 Jan 2026 08:29:18 GMT
  Transfer-Encoding: chunked
  Connection: keep-alive
  X-Powered-By: HAPI FHIR 8.5.3-SNAPSHOT/e3a3c5f741/2025-08-28 REST Server (FHIR Server; FHIR 4.0.1/R4)
  X-Request-ID: UbbmBpPRELgqIKJ1
  Content-Type: application/fhir+json; charset=utf-8
  Last-Modified: Tue, 13 Jan 2026 08:29:18 GMT
回應內容:
{
  "resourceType": "Bundle",
  "id": "3b672bdf-058e-4580-bab8-dbf3e6335188",
  "meta": {
    "lastUpdated": "2026-01-13T08:29:18.646+00:00"
  },
  "type": "searchset",
  "total": 1,
  "link": [ {
    "relation": "self",
    "url": "https://hapi.fhir.org/baseR4/Patient?_count=5&identifier=MRN-20240814A1"
  } ],
  "entry": [ {
    "fullUrl": "https://hapi.fhir.org/baseR4/Patient/53805146",
    "resource": {
      "resourceType": "Patient",
      "id": "53805146",
      "meta": {
        "versionId": "2",
        "lastUpdated": "2026-01-13T08:29:17.196+00:00",
        "source": "#etoip1zPped1yLq4"
      },
      "text": {
        "status": "generated",
        "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\"><div class=\"hapiHeaderText\">Vulcan20250814111 <b>LEE </b></div><table class=\"hapiPropertyTable\"><tbody><tr><td>Identifier</td><td>MRN-20240814A1</td></tr><tr><td>Date of birth</td><td><span>01 January 1990</span></td></tr></tbody></table></div>"
      },
      "identifier": [ {
        "system": "http://example.org/mrn",
        "value": "MRN-20240814A1"
      } ],
      "active": false,
      "name": [ {
        "family": "Lee",
        "given": [ "Vulcan20250814111" ]
      } ],
      "telecom": [ {
        "system": "phone",
        "value": "0912-345-678",
        "use": "mobile"
      }, {
        "system": "email",
        "value": "Vulcan20250814111.Lee@example.org"
      } ],
      "gender": "female",
      "birthDate": "1990-01-01"
    },
    "search": {
      "mode": "match"
    }
  } ]
}
耗時: 207.54 ms
==================================================
Search total (if provided): 1
 - 53805146 | Vulcan20250814111 Lee | active=False
Press any key to continue...
 Deleting Patient ...
===== HTTP 請求 =====
DELETE https://hapi.fhir.org/baseR4/Patient/53805146
標頭:
  Accept: application/fhir+json
  Accept-Charset: utf-8
  User-Agent: firely-sdk-client/5.12.1

===== HTTP 回應 =====
狀態: 200 OK
標頭:
  Server: nginx/1.24.0, (Ubuntu)
  Date: Tue, 13 Jan 2026 08:29:21 GMT
  Transfer-Encoding: chunked
  Connection: keep-alive
  X-Powered-By: HAPI FHIR 8.5.3-SNAPSHOT/e3a3c5f741/2025-08-28 REST Server (FHIR Server; FHIR 4.0.1/R4)
  X-Request-ID: 483EQAcOrq348Tca
  Content-Type: application/fhir+json; charset=utf-8
  Content-Location: https://hapi.fhir.org/baseR4/Patient/53805146/_history/3
回應內容:
{
  "resourceType": "OperationOutcome",
  "text": {
    "status": "generated",
    "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\"><h1>Operation Outcome</h1><table border=\"0\"><tr><td style=\"font-weight: bold;\">INFORMATION</td><td>[]</td><td>Successfully deleted 1 resource(s). Took 18ms.</td></tr></table></div>"
  },
  "issue": [ {
    "severity": "information",
    "code": "informational",
    "details": {
      "coding": [ {
        "system": "https://hapifhir.io/fhir/CodeSystem/hapi-fhir-storage-response-code",
        "code": "SUCCESSFUL_DELETE",
        "display": "Delete succeeded."
      } ]
    },
    "diagnostics": "Successfully deleted 1 resource(s). Took 18ms."
  } ]
}
耗時: 226.93 ms
==================================================
Deleted.
===== HTTP 請求 =====
GET https://hapi.fhir.org/baseR4/Patient/53805146
標頭:
  Accept: application/fhir+json
  Accept-Charset: utf-8
  User-Agent: firely-sdk-client/5.12.1

===== HTTP 回應 =====
狀態: 410 Gone
標頭:
  Server: nginx/1.24.0, (Ubuntu)
  Date: Tue, 13 Jan 2026 08:29:22 GMT
  Transfer-Encoding: chunked
  Connection: keep-alive
  X-Powered-By: HAPI FHIR 8.5.3-SNAPSHOT/e3a3c5f741/2025-08-28 REST Server (FHIR Server; FHIR 4.0.1/R4)
  X-Request-ID: bxMyna52dXXlZoKB
  Location: https://hapi.fhir.org/baseR4/Patient/53805146/_history/3
  Content-Type: application/fhir+json; charset=utf-8
回應內容:
{
  "resourceType": "OperationOutcome",
  "text": {
    "status": "generated",
    "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\"><h1>Operation Outcome</h1><table border=\"0\"><tr><td style=\"font-weight: bold;\">ERROR</td><td>[]</td><td>Resource was deleted at 2026-01-13T08:29:21.412+00:00</td></tr></table></div>"
  },
  "issue": [ {
    "severity": "error",
    "code": "processing",
    "diagnostics": "Resource was deleted at 2026-01-13T08:29:21.412+00:00"
  } ]
}
耗時: 977.67 ms
==================================================
Confirmed 410 Gone after delete.
Press any key to continue...
```

