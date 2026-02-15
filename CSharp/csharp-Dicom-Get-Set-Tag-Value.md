# 將 取得或設定 DICOM 檔案中的標籤值

在我前面的兩篇文章 [列出 DICOM 檔案中的所有標籤](https://csharpkh.blogspot.com/2026/02/cs-dicom-list-all-tags.html) 與 [將 DICOM 轉換成為一個 Image](https://csharpkh.blogspot.com/2026/02/cs-dicom-image-pacs.html) 中，已經介紹了如何使用 fo-dicom 套件來處理 DICOM 影像，並且展示了如何列出 DICOM 檔案中的所有標籤以及將 DICOM 影像轉換成為一個 Image。接下來，在這篇文章中，我們將進一步探討如何取得或設定 DICOM 檔案中的標籤值。

## 建立 主控台應用程式 專案
* 開啟 Visual Studio 2026
* 選擇「建立新專案」
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇 [主控台應用程式] 項目
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 "csDicomGetSetTagValue"
* 然後點擊右下方「下一步」按鈕
* 接著會看到 [其他資訊] 對話窗
* 在這個對話窗內，確認使用底下的選項
    * 架構：.NET 10.0 (或更新版本)
    * 勾選 不要使用最上層陳述式 (這是我的個人習慣)
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 主控台應用程式 專案的建立

## 安裝 fo-dicom 套件

套件 fo-dicom 是一個用於處理 DICOM 影像的套件，它提供了豐富的功能來處理 DICOM 影像，並且支援多種圖像格式的轉換。這裡是該套件提供功能的主要清單：
* DICOM 影像處理：提供了豐富的功能來處理 DICOM 影像，包括讀取、寫入、修改和轉換等功能。
* 支援多種圖像格式：該套件支援多種圖像格式的轉換，包括 JPEG、PNG、BMP 等，方便用戶將 DICOM 影像轉換為其他格式進行使用。

* 在 Visual Studio 的「方案總管」視窗中，右鍵點擊專案名稱
* 從右鍵選單中，選擇「管理 NuGet 套件」
* 在 NuGet 套件管理器視窗中，切換到「瀏覽」標籤頁
* 在搜尋框中，輸入 "fo-dicom" 並按下 Enter 鍵
* 從搜尋結果中，找到 "fo-dicom" 套件 並點擊它
* 在這裡的範例中，使用該套件的版本為 5.12.1
* 在右側的詳細資訊面板中，點擊「安裝」按鈕

## 撰寫程式碼
* 打開 Program.cs 檔案，並將其內容替換為以下程式碼：

```csharp
using FellowOakDicom;

namespace csDicomGetSetTagValue;

internal class Program
{
    static void Main(string[] args)
    {
        string dicomFile = "image-00000.dcm";
        string dicomUpdateFile = "image-00000Update.dcm";
        string rootPath = Directory.GetCurrentDirectory();
        string dicomFilePath = Path.Combine(rootPath, dicomFile);
        string dicomUpdateFilePath = Path.Combine(rootPath,  dicomUpdateFile);
        GetOrSetDicomTag(dicomFilePath, dicomUpdateFilePath);
    }

    private static void GetOrSetDicomTag(string dicomPath, string outputDicomPath)
    {
        try
        {
            var dicomFile = DicomFile.Open(dicomPath);
            DicomTag patientIdTag = QueryTag(dicomFile);
            SetTag(dicomFile, DicomTag.PatientID);
            DicomTag patientIdTagAgain = QueryTag(dicomFile);
            dicomFile.Save(dicomPath);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"轉換失敗: {ex.Message}");
            throw;
        }
    }

    private static void SetTag(DicomFile dicomFile, DicomTag patientIdTag)
    {
        dicomFile.Dataset.AddOrUpdate(patientIdTag, Guid.NewGuid().ToString());
    }

    private static DicomTag QueryTag(DicomFile dicomFile)
    {
        var patientIdTag = DicomTag.PatientID; // (0010,0020)
        if (dicomFile.Dataset.Contains(patientIdTag))
        {
            var patientId = dicomFile.Dataset.GetString(patientIdTag);
            Console.WriteLine($"原始 Patient ID: {patientId}");
        }
        else
        {
            Console.WriteLine("Patient ID 標籤不存在。");
        }
        return patientIdTag;
    }
}
```

在程式一開始，宣告了五個字串 [dicomFile] 代表了要處理的 DICOM 檔案名稱， [dicomUpdateFile] 代表了修改後的 DICOM 檔案名稱， [rootPath] 代表了目前專案的根目錄路徑， [dicomFilePath] 代表了要處理的 DICOM 檔案的完整路徑， [dicomUpdateFilePath] 代表了修改後的 DICOM 檔案的完整路徑。

然後就會呼叫 [GetOrSetDicomTag] 方法，在這個方法內，首先會使用 [DicomFile.Open] 方法來開啟 DICOM 檔案，在這裡將會得到一個 [DicomFile] 物件，接著會呼叫 [QueryTag] 方法來取得 DICOM 檔案中的 Patient ID 標籤值，在這個  [QueryTag] 方法內，透過 [DicomTag.PatientID] 來指定要查詢的標籤(病人病歷號)，儲存在 [patientIdTag] 變數中，然後使用 [Contains] 方法來檢查該標籤是否存在於 DICOM 檔案中，如果存在，就使用 [GetString] 方法來取得該標籤的值，並且將其輸出到主控台上。

繼續回到 [GetOrSetDicomTag] 方法內，接著會呼叫 [SetTag] 方法來設定 DICOM 檔案中的 Patient ID 標籤值，在這個 [SetTag] 方法內，使用 [AddOrUpdate] 方法來新增或更新 Patient ID 標籤的值，這裡將其設定為一個新的 GUID 字串。最後，剛剛更新的 DICOM 檔案會被儲存回原本的路徑。



