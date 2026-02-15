# 將語音檔案進行批次轉錄成為文字

因為工作的關係，經常會與客戶進行開會，討論軟體或者系統該如何開發或者修正的需求，又或者在內部進行開會，因此，無論是會議錄音、訪談內容、醫療紀錄，或是客服通話內容，只要能夠轉換為可搜尋、可分析的文字資料，如此，就可以將這些錄音檔案轉換出來文字稿，使用 ChatGPT 這個 LLM 大語言模型，整理出來一份會議紀錄，可以把該會議的內容分類歸納出來，做為日後執行與追蹤之用。這時候，Azure Speech to Text 便成為一項極具價值的雲端服務。它隸屬於 Microsoft Azure，透過深度學習模型將語音轉換為高品質文字，支援即時串流與離線批次轉錄。

當我們手上有錄音筆或手機錄製的語音檔案，例如 MP3、WAV 或 M4A 檔案，其實整個轉換流程並不複雜。第一步通常是在 Azure 入口網站中建立 Speech 資源。建立完成後，系統會提供金鑰（Key）與服務端點（Endpoint），這兩個資訊就是後續呼叫 API 的核心憑證。

接著，使用者可以選擇透過 SDK 或 REST API 進行語音轉換。如果是單一檔案測試，開發者可以透過簡單的程式碼將音訊檔上傳至服務端點，系統會回傳轉換後的文字內容。這種方式適合小型專案或即時處理需求，例如在 Web 應用程式中即時顯示逐字稿。

在技術實作層面上，開發流程大致包含三個核心動作。首先是音訊格式的確認與優化，例如建議使用單聲道、16kHz 或 44.1kHz 的標準音訊格式，以提升辨識準確率。接著是透過 SDK 初始化 SpeechConfig，設定語言（例如 zh-TW 或 en-US）與金鑰資訊。最後將音訊串流傳入 SpeechRecognizer 物件，等待服務回傳辨識結果。整個過程可以在幾十行程式碼內完成。

從架構角度來看，實務上常見的做法是建立一條完整的語音處理流程。音訊由前端裝置上傳至雲端儲存空間，再由後端服務呼叫 Speech API 進行轉錄，轉換完成後將結果寫入資料庫，最後透過搜尋或 AI 分析系統進行後續應用。這樣的設計能讓語音資料真正成為企業可再利用的數位資產，而不是一次性的錄音檔案。

整體而言，Azure Speech to Text 的導入並不困難，但其價值並不只在於「轉成文字」本身，而是在於它打開了語音資料數位化、結構化與智慧化應用的大門。當語音可以被搜尋、索引、分析與整合時，它就不再只是聲音，而是企業知識的一部分。

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

## 安裝 Newtonsoft.Json 套件

套件 [Newtonsoft.Json] 是一個流行的 JSON 處理庫，提供了方便的序列化和反序列化功能。透過這個套件，我們可以輕鬆地在 C# 程式碼中處理 JSON 資料，例如將物件轉換為 JSON 字串，或將 JSON 字串轉換為物件。這裡將會列出這個套件提供的功能清單：
* 將物件序列化為 JSON 字串
* 將 JSON 字串反序列化為物件
* 支援自訂序列化設定
* 支援 LINQ to JSON 操作

* 在 Visual Studio 的「方案總管」視窗中，右鍵點擊專案名稱
* 從右鍵選單中，選擇「管理 NuGet 套件」
* 在 NuGet 套件管理器視窗中，切換到「瀏覽」標籤頁
* 在搜尋框中，輸入 "Newtonsoft.Json" 並按下 Enter 鍵
* 從搜尋結果中，找到 "Newtonsoft.Json" 套件 並點擊它
* 在這裡的範例中，使用該套件的版本為 13.0.3
* 在右側的詳細資訊面板中，點擊「安裝」按鈕

## 撰寫程式碼
* 打開 Program.cs 檔案，並將其內容替換為以下程式碼：

```csharp
using Newtonsoft.Json;
using System.Net.Http.Headers;

namespace csSpeechToTextBatch;

// 定義 JSON 結構對應的 POCO
class TranscriptionJson
{
    [JsonProperty("combinedRecognizedPhrases")]
    public CombinedPhrase[] CombinedRecognizedPhrases { get; set; }
}
class CombinedPhrase
{
    [JsonProperty("display")]
    public string Display { get; set; }
}
internal class Program
{
    static async Task Main(string[] args)
    {
        // Speech 服務金鑰與區域
        string SubscriptionKey = Environment.GetEnvironmentVariable("AzureSpeechServiceSubscriptionKey");
        string ServiceRegion = Environment.GetEnvironmentVariable("AzureSpeechServiceRegion");

        // 上傳到 Blob Storage 的音檔 SAS URI
        string AudioFileSasUri = "https://blogstoragekh.blob.core.windows.net/audio-files/250501_0814.mp3?sv=2025-05-05&se=2025-05-01T10%3A53%3A53Z&sr=b&sp=r&sig=HVG%2Bs3hxD5cmv%2FrVOs5HZbekqmIBJujOGJWnsRLTjUQ%3D";

        var client = new HttpClient();
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", SubscriptionKey);

        // 1. 建立轉錄工作
        var createUrl = $"https://{ServiceRegion}.api.cognitive.microsoft.com/speechtotext/v3.2/transcriptions";
        var createBody = new
        {
            contentUrls = new[] { AudioFileSasUri },
            locale = "zh-TW",
            displayName = "My Batch Transcription",
            properties = new
            {
                diarizationEnabled = false,
                wordLevelTimestampsEnabled = true,
                punctuationMode = "DictatedAndAutomatic"
            }
        };

        var jsonContent = new StringContent(JsonConvert.SerializeObject(createBody));
        jsonContent.Headers.ContentType = new MediaTypeHeaderValue("application/json");
        var createResponse = await client.PostAsync(createUrl, jsonContent);
        createResponse.EnsureSuccessStatusCode();

        var createResult = await createResponse.Content.ReadAsStringAsync();
        Console.WriteLine("已建立批次轉錄工作：");
        Console.WriteLine(createResult);

        // 解析 self URL
        dynamic createJson = JsonConvert.DeserializeObject(createResult);
        string transcriptionUrl = createJson.self;

        // 2. 輪詢狀態
        Console.WriteLine("開始輪詢轉錄狀態…");
        TimeSpan elapsedTime;
        DateTime startTime = DateTime.Now;
        while (true)
        {
            elapsedTime = DateTime.Now - startTime;
            // 顯示已經花費時間 小時:分鐘:秒
            Console.WriteLine($"已經花費時間：{elapsedTime.Hours:D2}:{elapsedTime.Minutes:D2}:{elapsedTime.Seconds:D2}");

            var statusResponse = await client.GetAsync(transcriptionUrl);
            statusResponse.EnsureSuccessStatusCode();

            var statusJson = await statusResponse.Content.ReadAsStringAsync();
            dynamic statusObj = JsonConvert.DeserializeObject(statusJson);
            string status = statusObj.status;
            Console.WriteLine($"目前狀態：{status}");

            if (status == "Succeeded")
            {
                // 3. 取得並下載轉錄結果
                string filesUrl = statusObj.links.files;
                var filesResponse = await client.GetAsync(filesUrl);
                filesResponse.EnsureSuccessStatusCode();

                var filesJson = await filesResponse.Content.ReadAsStringAsync();
                dynamic filesObj = JsonConvert.DeserializeObject(filesJson);

                foreach (var file in filesObj.values)
                {
                    if ((string)file.kind == "Transcription")
                    {
                        var fileUrl = (string)file.links.contentUrl;
                        var transcriptionResult = await client.GetStringAsync(fileUrl);
                        //Console.WriteLine("---- 轉錄結果 ----");
                        //Console.WriteLine(transcriptionResult);

                        // 取得最終錄音文字
                        // 反序列化
                        var resultObj = JsonConvert.DeserializeObject<TranscriptionJson>(transcriptionResult);

                        // 串接所有 display 文字，並印出完整內容
                        string fullText = string.Join(" ",
                            resultObj.CombinedRecognizedPhrases
                                     .Select(p => p.Display?.Trim())
                                     .Where(s => !string.IsNullOrEmpty(s))
                        );
                        Console.WriteLine("---- 完整轉錄文字 ----");
                        Console.WriteLine(fullText);
                    }
                }
                break;
            }
            else if (status == "Failed")
            {
                Console.WriteLine("轉錄失敗");
                break;
            }

            await Task.Delay(TimeSpan.FromSeconds(5));
        }
    }
}
```


在這個檔案內，宣告了 [TranscriptionJson] 與 [CombinedPhrase] 兩個類別，這些類別用來對應從 Azure Speech to Text API 回傳的 JSON 結構。

在程式進入點，宣告了三個字串 [SubscriptionKey]、[ServiceRegion] 與 [AudioFileSasUri]，分別用來存放 Azure Speech 服務的金鑰、區域與要轉錄的音訊檔案的 SAS URI。其中，關於 [SAS URI] 部分，想要取得這個服務端點，可以參考 [如何使用與存取 Azure Blob Storage Service](https://csharpkh.blogspot.com/2026/02/csharp-Azure-Blob-Storage.html) 這篇文章內的說明。

接著，程式碼中使用了 [HttpClient] 類別來與 Azure Speech API 進行 HTTP 通訊。首先，建立了一個新的 HttpClient 實例，並在其標頭中加入了 [Ocp-Apim-Subscription-Key] 的訂閱金鑰。宣告了 [createUrl] 這個服務端點字串，作為呼叫 Speech To Text 的 API 端點，其中，[ServiceRegion] 是前面宣告的區域字串，這個值會根據你在 Azure 入口網站中建立 Speech 資源時所選擇的區域而有所不同。接著要產生一個 [createBody] 的物件，這個物件包含了轉錄工作所需的參數，例如使用 [contentUrls] 做為要轉錄的音訊檔案 URL、[locale] 設定語言，這裡指定了 [zh-tw] 也就是要產生出繁體中文的文稿、[displayName] 是這個轉錄工作的顯示名稱，還有一些屬性設定，例如 [diarizationEnabled] 是否啟用說話人分離、[wordLevelTimestampsEnabled] 是否啟用逐字時間戳記、[punctuationMode] 標點符號的處理方式等等。然後，將這個物件序列化為 JSON 字串，並包裝成一個 [StringContent] 物件，最後呼叫 [PostAsync] 方法來發送 HTTP POST 請求以建立轉錄工作。

一旦這次的 API 呼叫成功，便可以透過 [Content.ReadAsStringAsync()] 方法來取得 API 回傳的結果，這個結果會包含一些關於轉錄工作的資訊，例如轉錄工作的 ID、狀態、以及一個 [self] URL，這個 URL 就是用來輪詢轉錄狀態的端點。透過 [Json.NET] 將取得的回應 JSON 內容，反序列化成為 [createJson] 這個型別為 [dynamic] 的物件，然後從中取出 [self] URL，並將它存放在 [transcriptionUrl] 這個字串變數中。這個 [transcriptionUrl] 就是後續用來輪詢轉錄狀態的 API 端點，透過不斷的呼叫這個 API，確認此次將語音文字轉換成為文字稿的過程是否已經完成。

這裡同樣的是使用 [HttpClient] 來發送 HTTP GET 請求，並在每次呼叫後，解析回傳的 JSON 內容，經過反序列化之後，取得這個 JSON 物件 [statusObj]，透過該物件從中取出 [status] 欄位的值，來確認目前轉錄工作的狀態。如果狀態是 "Succeeded"，就代表轉錄已經完成，可以進一步取得轉錄結果；如果狀態是 "Failed"，則代表轉錄過程中發生了錯誤，需要進行錯誤處理。為了避免過於頻繁地呼叫 API，這裡使用了 [Task.Delay] 方法來讓程式在每次輪詢之間暫停幾秒鐘。

一旦 [status] 的值變成 "Succeeded"，就表示轉換過程已經完成了，接著就可以透過 [filesUrl] 這個 API 端點來取得轉錄結果的相關資訊。這裡同樣是使用 [HttpClient] 發送 HTTP GET 請求，並解析回傳的 JSON 內容，從中找到轉錄結果的 URL，然後再發送一次 HTTP GET 請求來下載轉錄結果的 JSON 內容。最後，將這個轉錄結果的 JSON 字串反序列化成為 [TranscriptionJson] 這個物件，從中取出所有的 [display] 欄位內容，並將它們串接起來形成完整的轉錄文字，最後印出來。