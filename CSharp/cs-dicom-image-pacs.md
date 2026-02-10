# 將 DICOM 轉換成為一個 Image

2025 年，我大多在進行醫療相關應用軟體開發，在進行 AI 臨床試驗平台開發的過程中，經常需要處理 DICOM 影像檔案。這個系統需要讓使用者上傳一個 L3 DICOM 影像檔案，然後將其轉換為 PNG 格式，以便在網頁上顯示。這個過程涉及到 DICOM 影像的解析、處理和轉換，並且需要確保轉換後的圖像質量和準確性。

對於 DICOM（Digital Imaging and Communications in Medicine）是一種用於儲存和傳輸醫療影像的標準格式。將 DICOM 影像轉換為一般的圖像格式（如 PNG 或 JPEG）是非常常見的需求，這樣可以方便地在各種應用程式中顯示和處理這些影像。

為了要完成這樣的需求，我們可以使用 C# 語言來開發一個簡單的應用程式，利用一些現有的庫來處理 DICOM 影像的解析和轉換。這裡選擇的是 fo-dicom 這個開源的 DICOM 庫，它提供了豐富的功能來處理 DICOM 影像，並且支援多種圖像格式的轉換。

底下是我的測試過程，這裡會將一個 DICOM 檔案轉換成為 PNG 格式的圖像，並且在過程中會顯示一些相關的資訊。

## 建立 主控台應用程式 專案
* 開啟 Visual Studio 2026
* 選擇「建立新專案」
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇 [主控台應用程式] 項目
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 "csDicomToImage"
* 然後點擊右下方「下一步」按鈕
* 接著會看到 [其他資訊] 對話窗
* 在這個對話窗內，確認使用底下的選項
    * 架構：.NET 10.0 (或更新版本)
    * 勾選 不要使用最上層陳述式 (這是我的個人習慣)
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 主控台應用程式 專案的建立

## 安裝 fo-dicom.Imaging.ImageSharp 套件

套件 fo-dicom.Imaging.ImageSharp 是一個用於處理 DICOM 影像的套件，它提供了將 DICOM 影像轉換為 ImageSharp 影像的功能。這裡是該套件提供功能的主要清單：
* DICOM 影像轉換：將 DICOM 影像轉換為 ImageSharp 影像格式，方便進行後續的圖像處理和顯示。
* 支援多種 DICOM 影像格式：該套件支援多種 DICOM 影像格式，包括單幀和多幀的 DICOM 影像。
* 圖像處理功能：提供了一些基本的圖像處理功能，例如調整亮度、對比度、旋轉等，方便用戶對 DICOM 影像進行處理。

使用底下方式進行安裝此套件

* 在 Visual Studio 的「方案總管」視窗中，右鍵點擊專案名稱
* 從右鍵選單中，選擇「管理 NuGet 套件」
* 在 NuGet 套件管理器視窗中，切換到「瀏覽」標籤頁
* 在搜尋框中，輸入 "fo-dicom.Imaging.ImageSharp" 並按下 Enter 鍵
* 從搜尋結果中，找到 "fo-dicom.Imaging.ImageSharp" 套件 並點擊它
* 在這裡的範例中，使用該套件的版本為 5.2.2
* 在右側的詳細資訊面板中，點擊「安裝」按鈕

## 安裝 fo-dicom.Codecs 套件

套件 fo-dicom.Codecs 是一個用於處理 DICOM 影像編解碼的套件，它提供了對 DICOM 影像進行編碼和解碼的功能。這裡是該套件提供功能的主要清單：
* DICOM 影像編碼：將圖像數據編碼為 DICOM 影像格式，方便儲存和傳輸。
* DICOM 影像解碼：將 DICOM 影像解碼為圖像數據，方便進行後續的圖像處理和顯示。

使用底下方式進行安裝此套件

* 在 Visual Studio 的「方案總管」視窗中，右鍵點擊專案名稱
* 從右鍵選單中，選擇「管理 NuGet 套件」
* 在 NuGet 套件管理器視窗中，切換到「瀏覽」標籤頁
* 在搜尋框中，輸入 "fo-dicom.Codecs" 並按下 Enter 鍵
* 從搜尋結果中，找到 "fo-dicom.Codecs" 套件 並點擊它
* 在這裡的範例中，使用該套件的版本為 5.2.2
* 在右側的詳細資訊面板中，點擊「安裝」按鈕

## 撰寫程式碼
* 打開 Program.cs 檔案，並將其內容替換為以下程式碼：

```csharp
using FellowOakDicom;
using FellowOakDicom.Imaging;
using SixLabors.ImageSharp.Formats.Png;

namespace csDicomToImage;

internal class Program
{
    static async Task Main(string[] args)
    {
        // 初始化 DICOM 設定
        InitializeDicom();

        string dicomFile = "image-00000.dcm";
        string pngFile = "out.png";
        string rootPath = Directory.GetCurrentDirectory();
        string dicomFilePath = Path.Combine(rootPath, dicomFile);
        string pngFilePath = Path.Combine(rootPath, pngFile);

        // 轉換單個檔案
        ConvertSingleFile(dicomFilePath, pngFilePath);

        // 批次轉換資料夾中的所有 DICOM 檔案
        // ConvertDirectory("input_folder", "output_folder");
    }

    /// <summary>
    /// 初始化 DICOM 設定，註冊 ImageSharp 影像管理器
    /// </summary>
    private static void InitializeDicom()
    {
        try
        {
            new DicomSetupBuilder()
                .RegisterServices(s =>
                s.AddFellowOakDicom()
                 .AddTranscoderManager<FellowOakDicom.Imaging.NativeCodec.NativeTranscoderManager>()
                 .AddImageManager<ImageSharpImageManager>())
          .SkipValidation()
          .Build();

            Console.WriteLine("DICOM 初始化成功");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"DICOM 初始化失敗: {ex.Message}");
            throw;
        }
    }

    public static void ConvertSingleFile(string dicomPath, string pngPath)
    {
        try
        {
            Console.WriteLine($"開始轉換: {dicomPath}");

            // 開啟 DICOM 檔案
            var dicomFile = DicomFile.Open(dicomPath);
            var dicomImage = new DicomImage(dicomFile.Dataset);

            // 渲染影像
            //var foo = dicomImage.RenderImage().As<Bitmap>();
            var renderedImage = dicomImage.RenderImage();

            // 將 renderedImage 轉換為 ImageSharp Image 並儲存為 PNG
            var sharpImage = renderedImage.AsSharpImage();

            // 儲存為 PNG
            using (var fileStream = new FileStream(pngPath, FileMode.Create))
            {
                sharpImage.Save(fileStream, new PngEncoder());
            }

            Console.WriteLine($"轉換完成: {pngPath}");

            // 顯示影像資訊
            //DisplayImageInfo(dicomFile.Dataset, sharpImage);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"轉換失敗: {ex.Message}");
            throw;
        }
    }
}
```

這個 Console 主控台應用程式，一開始將會呼叫 InitializeDicom() 方法來初始化 DICOM 的設定，這個方法內使用了 DicomSetupBuilder() 來註冊 DICOM 的服務，並且指定使用 ImageSharpImageManager 來處理影像的管理。這個 AddFellowOakDicom() 方法是用來註冊 DICOM 的核心功能，而 AddTranscoderManager() 方法則是用來註冊 DICOM 影像的編解碼器，這裡使用的是 NativeTranscoderManager，這個編解碼器可以支援多種 DICOM 影像格式的轉換。

在第一次接觸這個套件，我對於這個註冊過程感到有些困惑，因為它涉及到一些較為底層的設定，但在閱讀了相關的文件和範例程式碼後，我逐漸理解了這個過程的意義。這個註冊過程是必要的，因為它告訴 DICOM 庫如何處理影像的管理和編解碼，這樣在後續的轉換過程中，DICOM 庫就能夠正確地處理影像數據。

接下來，在 Main() 方法中，我們定義了要轉換的 DICOM 檔案路徑和輸出的 PNG 檔案路徑，然後呼叫 ConvertSingleFile() 方法來進行轉換。

這個方法內部首先會使用 DicomFile.Open 方法來開啟指定的 DICOM 檔案，然後用剛剛開啟的 Dicom 檔案來建立一個 [DicomImage] 物件，使用 DicomImage 類別有一個方法 [RenderImage()] 可以用來渲染影像，該方法會回傳一個 IImage 物件，使用該物件的 [AsSharpImage()] 方法來取得一個 ImageSharp 的圖像格式，最後將其儲存為 PNG 格式的檔案。

如此，便可以透過上述過程將 DICOM 影像成功轉換為 PNG 格式，並且在過程中顯示相關的資訊。

# 執行程式

首先先來看這個專案的執行結果：

* 按下 F5 鍵或點擊「開始」按鈕來執行程式
* 底下為實際操作過程的輸出文字

```
DICOM 初始化成功
開始轉換: C:\Vulcan\Github\CSharp2025\csDicomToImage\csDicomToImage\bin\Debug\net9.0\image-00000.dcm
轉換完成: C:\Vulcan\Github\CSharp2025\csDicomToImage\csDicomToImage\bin\Debug\net9.0\out.png
```



