# 如何使用與存取 Azure Blob Storage Service

因為需要設計一個將錄音聲音檔案，轉換成為文字文稿的系統，所以需要一個地方來儲存這些聲音檔案。當然，最簡單的做法就是把它們放在本地磁碟裡，但這樣就無法享受到雲端儲存的彈性與便利性了。於是，我們選擇了 Azure Blob Storage 來作為我們的儲存解決方案。另外一個採用 Azure Blob Storage 的原因是，有些時候語音檔案太大了，為了避免陸續上船語音內容到雲端上，讓 STT (Speech To Text) 可以完成與順暢，因此，會先把語音檔案上傳到 Azure Blob Storage，然後再讓 STT 直接從 Azure Blob Storage 上讀取檔案來進行轉錄，這樣就能大幅提升整體的效率。

一旦將與音檔案上傳到 Azure Blob Storage 之後，就會取得該 Blog 物件與 SAS URI，接著就可以把這些資訊傳遞給 STT 來進行轉錄了。當然，除了上傳檔案之外，我們也可以使用 Azure Blob Storage 的 SDK 來進行其他的操作，例如：下載檔案、刪除檔案、列出容器中的檔案等等。這些功能都可以透過 Azure Blob Storage 的 SDK 來輕鬆實現。

首先要先來了解關於 [Azure Blob Storage](https://learn.microsoft.com/zh-tw/azure/storage/blobs/) 的一些基本概念與架構，才能更順利地在程式碼中使用它的 SDK 來進行上傳、下載、管理等操作。

在現代雲端架構中，「儲存」早已不只是把檔案放進硬碟那麼單純。當系統需要面對大量影像檔、備份資料、日誌紀錄、AI 訓練素材，甚至醫療系統中的 DICOM 或 FHIR 匯出資料時，我們真正需要的是一種可水平擴充、高可用、低維運負擔的儲存解決方案。這時候就會遇到一個關鍵服務 —— Azure Blob Storage。

Azure Blob Storage 是 Microsoft Azure 雲端平台提供的物件儲存（Object Storage）服務，專門用來儲存例如非結構化資料（Unstructured Data）、大型檔案、二進位資料（Binary Large Object, Blob）等各種類型的檔案內容。所謂 Blob，其實是「Binary Large Object」的縮寫，意思是「大型二進位物件」。這類資料通常無法用傳統資料庫表格方式儲存與查詢，因此更適合放在物件儲存系統中。

在傳統架構裡，我們可能會使用：本地磁碟、NAS 檔案伺服器、SAN 儲存設備。但在雲端環境中，更推薦使用物件儲存架構，而 Azure Blob Storage 就是這類架構的代表性服務。

Azure Blob Storage 採用三層邏輯架構：

```
Storage Account
    └── Container
            └── Blob
```

Storage Account 是最上層單位，可以視為一個儲存帳戶邊界。在這個層級中，你可以管理：存取金鑰、存取控制（RBAC）、網路限制、備援策略。它就像是一個獨立的儲存資源空間。

Container 類似資料夾概念，但本質更接近物件儲存中的 Bucket。你可以依用途建立不同的 Container，例如：fhir-export、dicom-images、ai-training-data、system-logs、backup-archives 等等，來邏輯分類與管理不同類型的資料。每個 Container 都有自己的存取權限設定，可以是公開讀取、私有存取，或是透過 SAS Token 進行細粒度授權。

Blob 才是真正儲存的檔案內容，例如 report.pdf、image.png、dataset.ndjson、backup.zip 等。每一個 Blob 都是獨立的物件，透過 HTTP REST API 進行存取。Azure Blob Storage 提供三種 Blob 類型，依用途不同而選擇：**Block Blob** 是最常見的類型，適合儲存文件、影像、影片、備份檔與 FHIR 匯出資料等大型物件。大多數應用都會使用這種類型。、**Append Blob** 適合日誌寫入情境，例如系統 Log 需要持續追加資料。、**Page Blob** 則多用於虛擬機器磁碟（VHD），支援隨機讀寫操作。

Azure Blob Storage 是 Microsoft Azure 雲端平台提供的物件儲存服務，專門用來儲存例如非結構化資料（Unstructured Data）、大型檔案、二進位資料（Binary Large Object, Blob）等各類型內容，具備高擴充性、高可用性與彈性成本控制能力。如果用一句話總結：Azure Blob Storage 是雲端資料架構中的基礎儲存層，是現代 AI 與大型資料系統不可或缺的核心服務。

## 建立 主控台應用程式 專案
* 開啟 Visual Studio 2026
* 選擇「建立新專案」
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇 [主控台應用程式] 項目
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 "csAzureBlobStorage"
* 然後點擊右下方「下一步」按鈕
* 接著會看到 [其他資訊] 對話窗
* 在這個對話窗內，確認使用底下的選項
    * 架構：.NET 10.0 (或更新版本)
    * 勾選 不要使用最上層陳述式 (這是我的個人習慣)
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 主控台應用程式 專案的建立

## 安裝 Azure.Storage.Blobs 套件

套件 [Azure.Storage.Blobs] 是 Microsoft 官方提供的用於與 Azure Blob Storage 進行互動的 SDK。透過這個套件，我們可以輕鬆地在 C# 程式碼中實現對 Azure Blob Storage 的各種操作，例如上傳、下載、刪除檔案，以及管理容器等。這裡將會列出這個套件提供的功能清單：
* 上傳檔案到 Azure Blob Storage
* 從 Azure Blob Storage 下載檔案
* 刪除 Azure Blob Storage 中的檔案
* 列出 Azure Blob Storage 中的檔案和容器
* 管理 Azure Blob Storage 中的容器
* 生成 Azure Blob Storage 的共享存取簽章 (SAS) URL

* 在 Visual Studio 的「方案總管」視窗中，右鍵點擊專案名稱
* 從右鍵選單中，選擇「管理 NuGet 套件」
* 在 NuGet 套件管理器視窗中，切換到「瀏覽」標籤頁
* 在搜尋框中，輸入 "Azure.Storage.Blobs" 並按下 Enter 鍵
* 從搜尋結果中，找到 "Azure.Storage.Blobs" 套件 並點擊它
* 在這裡的範例中，使用該套件的版本為 12.24.0
* 在右側的詳細資訊面板中，點擊「安裝」按鈕

## 撰寫程式碼
* 打開 Program.cs 檔案，並將其內容替換為以下程式碼：

```csharp
using Azure.Storage.Blobs.Models;
using Azure.Storage.Blobs;
using Azure.Storage.Sas;

namespace csAzureBlobStorage;

internal class Program
{
    static async Task Main(string[] args)
    {
        string connectionString = Environment.GetEnvironmentVariable("AZURE_STORAGE_CONNECTION_STRING");
        string containerName = "audio-files";     // 要上傳到的 container
        string currentDirectory = Directory.GetCurrentDirectory();
        string localFilePath =Path.Combine( Directory.GetCurrentDirectory(), "250501_0814.mp3"); 
        string blobName = Path.GetFileName(localFilePath); // blob 名稱
        var blobServiceClient = new BlobServiceClient(connectionString);
        var containerClient = blobServiceClient.GetBlobContainerClient(containerName);
        await containerClient.CreateIfNotExistsAsync(PublicAccessType.None);
        var blobClient = containerClient.GetBlobClient(blobName);
        Console.WriteLine($"開始上傳 {localFilePath} → {containerClient.Uri}/{blobName} ...");
        using FileStream uploadFileStream = File.OpenRead(localFilePath);
        var blobHttpHeaders = new BlobHttpHeaders { ContentType = "audio/mpeg" };
        await blobClient.UploadAsync( uploadFileStream, new BlobUploadOptions { HttpHeaders = blobHttpHeaders } );
        uploadFileStream.Close();
        Console.WriteLine("上傳完成！");
        var blobItemClient = containerClient.GetBlobClient(blobName);
        Uri blobUri = blobItemClient.Uri;
        Console.WriteLine(blobUri.ToString());
        var sasToken = blobItemClient.GenerateSasUri(BlobSasPermissions.Read, DateTimeOffset.UtcNow.AddHours(5));
        Console.WriteLine($"SAS URI: {sasToken}");
    }
}
```


在這個 Console 專案進入點方法中，首先宣告了五個字串，分別為 [connectionString] 其目的在為了儲存 Azure Blob Storage 的連線字串，這個連線字串會從環境變數中讀取，這樣就不需要把敏感資訊直接寫在程式碼裡面了。接著宣告了 [containerName] 這個字串變數，代表要上傳到 Azure Blob Storage 的哪一個 container。然後宣告了 [currentDirectory] 這個字串變數，透過 `Directory.GetCurrentDirectory()` 方法來取得當前的工作目錄路徑。接著宣告了 [localFilePath] 這個字串變數，將當前目錄與要上傳的檔案名稱結合成為完整的檔案路徑。最後宣告了 [blobName] 這個字串變數，從 localFilePath 中提取出檔案名稱作為 blob 的名稱。

接著，程式碼中建立了一個 [BlobServiceClient] 的實例，這個實例是用來與 Azure Blob Storage 進行互動的主要物件，這裡是透過這個 [BlobServiceClient]建構式與傳入 Azure Blog Storage 連線字串來建立此物件，接著使用了 [GetBlobContainerClient] 方法來取得一個 BlobContainerClient 的實例，這個實例代表了要操作的 container [containerClient]。然後呼叫了 [CreateIfNotExistsAsync] 方法，如果這個 container 不存在，則會自動建立一個新的 container。接著再透過 BlobContainerClient 來取得一個 BlobClient 的實例，這個實例代表了要操作的 blob。

之後，使用 [containerClient.GetBlobClient(blobName)] 來取得一個 BlobClient 的實例，這個實例代表了要操作的 blob。接著使用 [File.OpenRead(localFilePath)] 方法來開啟要上傳的檔案，並將它包裝成為一個 FileStream 的物件。然後建立了一個 [BlobHttpHeaders] 的物件，並設定 ContentType 為 "audio/mpeg"，這樣在瀏覽器或其他客戶端讀取這個 blob 時，就能正確地識別它的內容類型。最後，呼叫了 [UploadAsync] 方法來將檔案上傳到 Azure Blob Storage 中，並傳入 BlobUploadOptions 來設定 HTTP 標頭。一旦上傳完成，就會關閉檔案流並印出上傳完成的訊息。

對於 [blobItemClient] 這個 BlobClient 實例，其主要的用途在於操作特定的 blob，例如取得其 URL 或生成 SAS URI。我們可以直接從它的 Uri 屬性來取得這個 blob 的 URL，然後印出來。接著，使用 [GenerateSasUri] 方法來生成一個帶有讀取權限的 SAS URI。

對於 SAS 這樣的物件，其目的在於提供一個 URI 可以讓其他人或服務在指定的時間內存取這個 blob，而不需要提供 Azure Blob Storage 的帳戶金鑰。最後，將生成的 SAS URI 印出來。

