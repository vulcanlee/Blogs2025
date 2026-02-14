# 列出 DICOM 檔案中的所有標籤

在醫療資訊系統的世界裡，只要談到影像，幾乎一定會遇到一個關鍵名詞 —— DICOM（Digital Imaging and Communications in Medicine）。很多人會直覺認為：「DICOM 就是醫療用的圖片格式吧？」，這句話只對了一半。事實上，DICOM 不只是影像格式，而是一套完整的 醫療影像資料標準與通訊協定體系。它同時規範了：影像的儲存方式、、醫療資訊的結構、設備之間的傳輸協定、影像的查詢與調閱機制、甚至 AI 推論與放射治療資料的表示方式

DICOM 到底是什麼？DICOM 是由國際標準組織制定的醫療影像標準，主要應用於：CT、MRI、超音波、X 光、乳房攝影、放射治療系統。它的設計目標很明確， 讓不同廠牌的醫療設備可以互相理解影像資料，並安全、完整地傳輸。因此，DICOM 不只是「檔案格式」，它同時是一套：資料模型、物件定義規範、網路通訊協定、影像編碼標準、以及結構化報告格式。這也是為什麼 DICOM 在醫療影像領域能夠成為事實上的全球標準，並且在 PACS（Picture Archiving and Communication System）中扮演核心角色的原因。

很多人會問：DICOM 跟 JPG 有什麼不同？不都是圖片嗎？其實兩者的差異非常巨大。JPG 是單純的像素檔案。JPG（JPEG）是一種常見的影像壓縮格式，它的重點在於高壓縮比、檔案體積小、適合網路傳輸、多為 8-bit 色階，而且屬於有損壓縮。它適合用於生活照片、網頁圖片、社群媒體展示等一般用途。但在醫療場景中，這些特性反而可能成為限制。

醫療影像需要的是診斷等級的精準度。DICOM 通常支援 16-bit 甚至更高的灰階深度，能保留更多細節資訊，並可使用無損壓縮方式，確保影像中的微小差異不被破壞。例如腫瘤邊界、微小鈣化點或輕微出血區域，這些細節在有損壓縮下可能消失。更重要的是，JPG 只包含像素資料，而 DICOM 檔案同時包含影像與完整的醫療資訊，例如病患姓名、病歷號碼、檢查日期、檢查設備型號與掃描參數等。從本質上來看，JPG 是圖片；DICOM 是醫療資料物件。

那麼，DICOM 中常聽到的 Tag 又是什麼？Tag 是 DICOM 資料結構中的基本單位。每一個 Tag 由兩組十六進位數字組成，例如 (0010,0010) 代表 Patient Name，(0010,0020) 代表 Patient ID，(0008,0060) 代表檢查設備類型（Modality），(7FE0,0010) 則是影像像素資料（Pixel Data）。每個 Tag 都包含編號、資料型別（Value Representation）、資料長度與實際內容。可以把 Tag 想像成 DICOM 的「資料欄位定義」，正因為有這些結構化欄位，系統才能理解影像背後的醫療語意，而不是只看到一張灰階圖。

除了 Tag 之外，DICOM 還包含更多重要概念。首先是階層式資料結構：Patient、Study、Series、Instance。也就是說，一位病人可以有多次檢查（Study），每次檢查包含多個掃描序列（Series），而每個序列又包含多張影像（Instance）。這種階層模型完全貼合臨床流程。其次是 UID（Unique Identifier），每個 Study、Series 與影像物件都有全球唯一識別碼，確保跨系統交換時不會產生衝突。

此外，DICOM 還定義了 SOP Class，用來描述不同類型的醫療物件，例如 CT Image Storage、MR Image Storage、Structured Report、Segmentation 等。它也內建網路通訊指令，例如 C-STORE（儲存）、C-FIND（查詢）、C-MOVE（傳送）、C-ECHO（連線測試），讓影像設備能直接與 PACS 溝通。更進一步，DICOM 還支援結構化報告（DICOM SR）、影像分割（SEG）、放射治療結構（RT Structure）與劑量資料（RT Dose），甚至可封裝 AI 分析結果。這些能力，都是一般 JPG 完全無法提供的。

總結來說，JPG 是單純的影像壓縮格式；DICOM 則是一整套醫療影像資訊標準與通訊協定。它包含影像資料、結構化 Tag、階層式模型、全球唯一識別碼、網路傳輸機制，以及 AI 與臨床應用支援能力。當我們在設計醫療影像系統、PACS 架構，或整合 AI 分析與 FHIR 平台時，理解 DICOM，不只是理解一個檔案格式，而是理解整個醫療影像生態系統的基礎架構。

## 建立 主控台應用程式 專案
* 開啟 Visual Studio 2026
* 選擇「建立新專案」
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇 [主控台應用程式] 項目
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 "csDicomListAllTag"
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
using FellowOakDicom.IO.Buffer;

namespace csDicomListAllTag
{
    internal class Program
    {
        static async Task Main(string[] args)
        {
            string dicomFile = "image-00000.dcm";
            string rootPath = Directory.GetCurrentDirectory();
            string dicomFilePath = Path.Combine(rootPath, dicomFile);

            // 轉換單個檔案
            GetAllTags(dicomFilePath);
        }

        public static void GetAllTags(string dicomPath)
        {
            try
            {
                // 開啟 DICOM 檔案
                var dicomFile = DicomFile.Open(dicomPath);
                
                // 創建自定義訪問者以列印所有標籤
                var visitor = new DicomDatasetVisitor();
                var walker = new DicomDatasetWalker(dicomFile.Dataset);
                walker.Walk(visitor);
                
                Console.WriteLine($"DICOM 檔案 {Path.GetFileName(dicomPath)} 總共含有 {visitor.TagCount} 個標籤");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"轉換失敗: {ex.Message}");
                throw;
            }
        }
    }

    // 自定義訪問者類別，用於輸出所有 DICOM 標籤
    public class DicomDatasetVisitor : IDicomDatasetWalker
    {
        public int TagCount { get; private set; } = 0;

        public void OnBeginWalk()
        {
            Console.WriteLine("開始遍歷 DICOM 標籤...");
        }

        public void OnEndWalk()
        {
            Console.WriteLine("DICOM 標籤遍歷完成");
        }

        public bool OnElement(DicomElement element)
        {
            TagCount++;
            Console.WriteLine($"標籤: {element.Tag} ({element.Tag.DictionaryEntry?.Name ?? "Unknown"}) - 值: {GetElementValueAsString(element)}");
            return true;
        }

        public Task<bool> OnElementAsync(DicomElement element)
        {
            return Task.FromResult(OnElement(element));
        }

        public bool OnBeginSequence(DicomSequence sequence)
        {
            TagCount++;
            Console.WriteLine($"開始序列: {sequence.Tag} ({sequence.Tag.DictionaryEntry?.Name ?? "Unknown"})");
            return true;
        }

        public bool OnBeginSequenceItem(DicomDataset dataset)
        {
            Console.WriteLine("  序列項目開始 >>>");
            return true;
        }

        public bool OnEndSequenceItem()
        {
            Console.WriteLine("  <<< 序列項目結束");
            return true;
        }

        public bool OnEndSequence()
        {
            Console.WriteLine("序列結束");
            return true;
        }

        public bool OnBeginFragment(DicomFragmentSequence fragment)
        {
            TagCount++;
            Console.WriteLine($"開始片段: {fragment.Tag} ({fragment.Tag.DictionaryEntry?.Name ?? "Unknown"})");
            return true;
        }

        public bool OnFragmentItem(IByteBuffer item)
        {
            Console.WriteLine($"  片段項目: {item.Size} 位元組");
            return true;
        }

        public Task<bool> OnFragmentItemAsync(IByteBuffer item)
        {
            return Task.FromResult(OnFragmentItem(item));
        }

        public bool OnEndFragment()
        {
            Console.WriteLine("片段結束");
            return true;
        }

        private string GetElementValueAsString(DicomElement element)
        {
            try
            {
                return element.ToString();
            }
            catch
            {
                return "[無法顯示的值]";
            }
        }
    }
}
```

在這個程式碼內，首先宣告三個字串，分別是： [dicomFile]、[rootPath]、[dicomFilePath]，這三個字串分別代表 DICOM 檔案名稱、目前專案的根目錄路徑，以及 DICOM 檔案的完整路徑。接著，呼叫 [GetAllTags] 方法，並將 DICOM 檔案的完整路徑作為參數傳入。

主要處理的邏輯都會設計在 [GetAllTags] 方法內，這個方法將會使用 [DicomFile.Open] 方法來開啟 DICOM 檔案，得到一個 [DicomFile] 物件。接著，建立一個 [visitor] 物件，這個物件是我們自定義的 [DicomDatasetVisitor] 類別的實例。然後，使用 [DicomDatasetWalker] 類別來遍歷 DICOM 檔案中的資料集，這裡將會呼叫其 [Walk] 方法，並將我們的訪問者物件傳入，以便在遍歷過程中輸出每個標籤的資訊。最後統計出這個 Dicom 檔案內總共會有多少個 Tag。

為了要完成上述需求，我們還需要定義一個 [DicomDatasetVisitor] 類別，這個類別實作了 [IDicomDatasetWalker] 介面，並且在其中實作了多個方法來處理不同類型的 DICOM 元素，例如：一般元素、序列、片段等。這個 [IDicomDatasetWalker] 介面定義了在遍歷 DICOM 資料集時會被呼叫的多個事件處理方法，例如：當開始遍歷時、當遇到一般元素時、當遇到序列開始與結束時、當遇到片段開始與結束時等。透過這些方法，我們可以在遍歷過程中輸出每個標籤的資訊，並且統計總共有多少個標籤。以下是 [IDicomDatasetWalker] 介面的定義：

```csharp
public interface IDicomDatasetWalker
{
    void OnBeginWalk();
    bool OnElement(DicomElement element);
    Task<bool> OnElementAsync(DicomElement element);
    bool OnBeginSequence(DicomSequence sequence);
    bool OnBeginSequenceItem(DicomDataset dataset);
    bool OnEndSequenceItem();
    bool OnEndSequence();
    bool OnBeginFragment(DicomFragmentSequence fragment);
    bool OnFragmentItem(IByteBuffer item);
    Task<bool> OnFragmentItemAsync(IByteBuffer item);
    bool OnEndFragment();

    /// <summary>
    /// Handler for end of traversal.
    /// </summary>
    void OnEndWalk();
}
```

這裡主要的定義的方法 [OnElement] 是當遍歷到一般的 DICOM 元素時會被呼叫的方法，這個方法會接收一個 [DicomElement] 物件作為參數，呼叫 [GetElementValueAsString] 方法來取得元素的值，並且在方法內輸出該元素的標籤、名稱與值等資訊。其他的方法則是用來處理序列與片段等特殊類型的元素，並且在遍歷過程中輸出相關的資訊。

在 DICOM 裡，`Sequence`（序列）跟「片段」（fragment）是兩個不同層次的概念，你在程式裡看到的：

- `OnBeginSequence(DicomSequence sequence)`
- `OnBeginSequenceItem(DicomDataset dataset)`
- `OnBeginFragment(DicomFragmentSequence fragment)`
- `OnFragmentItem(IByteBuffer item)`

正好對應這兩種結構。

Sequence 在 DICOM 裡的角色，就像是一個「子資料表」或「陣列 of Dataset」，用來表示一組有結構的資料（多欄、多筆）。而 Fragment 則是用來存放壓縮影像資料的一種表示方式，通常對應到 Pixel Data 的封裝格式。這兩者在 DICOM 的資料模型中扮演著不同的角色，理解它們的差異對於正確處理 DICOM 資料非常重要。DICOM 的資料基本單位是「元素」（`DicomElement`），大部分元素是一個標籤 + 一個值，例如：病人姓名：`(0010,0010) PN`、影像寬度：`(0028,0010) US`

但有些資訊「不是一個單純值」，而是「一組有結構的資料（多欄、多筆）」——這時就會用 `SQ`（Sequence of Items）這種 VR（Value Representation）。Sequence 的角色：像是「子資料表」或「陣列 of Dataset」：一個 `DicomSequence` 對應一個 SQ element  ，例如：`(0008,1115) Referenced Series Sequence`；它裡面包含多個 Item，每個 Item 又是一個小的 `DicomDataset`，裡面有自己的標籤（`DicomElement`）與值，甚至再巢狀其他 `Sequence`。因此：Sequence 內含 **多個 Item**，每個 Item 又是一個小的 `DicomDataset`
  - 在程式裡就是 `OnBeginSequenceItem(DicomDataset dataset)` 那個 `dataset`
- 每個 Item 裡面又可以有自己的標籤（`DicomElement`）、甚至再巢狀其他 `Sequence`

可以把它類比成：

```plaintext
DicomDataset (根)
 └─ DicomSequence: "Referenced Study Sequence"
     ├─ Item #1: DicomDataset
     │   ├─ (0008,1150) Referenced SOP Class UID
     │   └─ (0008,1155) Referenced SOP Instance UID
     └─ Item #2: DicomDataset
         ├─ ...
```

在 walker 裡的對應關係是：

- `OnBeginSequence(DicomSequence sequence)`  
  → 遇到一個 SQ 元素開始
- `OnBeginSequenceItem(DicomDataset dataset)`  
  → 進入這個 SQ 裡的一個 item（類似一筆子記錄）
- `OnEndSequenceItem()`  
  → 結束這個 item
- `OnEndSequence()`  
  → 整個 SQ 結束

Fragment（片段）在 DICOM 裡的角色，則是用來存放壓縮影像資料的一種表示方式。當 DICOM 影像是以封裝（encapsulated）格式儲存時，像素資料（Pixel Data）會被切成多個 fragment，每個 fragment 就是一小段 `IByteBuffer` 資料，這些 fragment 聚集起來就是一個 `DicomFragmentSequence`。對於「像素資料」（`Pixel Data` – `(7FE0,0010)`）來說：如果影像是 **未壓縮**：`Pixel Data` 就是一塊連續的位元組陣列（單一 value 或簡單多值），如果影像是 **壓縮**（JPEG/JPEG2000/RLE 等）：標準會用「封裝（encapsulated）」格式，把一個影像 frame 的資料切成多個 fragment，每一個 fragment 就是一小段 `IByteBuffer` 資料，這一組 fragment 聚集起來，就是一個 `DicomFragmentSequence`

這裡的對應是：

- `OnBeginFragment(DicomFragmentSequence fragment)`  
  → 開始處理某個 fragment 序列（通常對應像素資料、壓縮影像）
- `OnFragmentItem(IByteBuffer item)`  
  → 單一片段（實際的一段 bytes）
- `OnEndFragment()`  
  → 這個 fragment 序列結束

可以類比成：

```plaintext
Pixel Data (7FE0,0010) - encapsulated
 └─ DicomFragmentSequence
     ├─ Fragment #1: IByteBuffer (部分 JPEG 資料)
     ├─ Fragment #2: IByteBuffer
     └─ Fragment #N: IByteBuffer
```

簡單對照表

| 概念          | 對應類型                  | 角色/用途                                    |
|---------------|---------------------------|----------------------------------------------|
| Element       | `DicomElement`            | 一般標籤 + 值                                |
| Sequence      | `DicomSequence` (SQ)      | 「多筆結構化資料」的容器（陣列 of Dataset）  |
| Sequence Item | `DicomDataset`            | Sequence 裡的一筆資料（子 Dataset）         |
| Fragment Seq. | `DicomFragmentSequence`   | 封裝壓縮影像時的片段容器                     |
| Fragment Item | `IByteBuffer`             | 單一個實際的 byte 區塊（片段本身）          |


# 執行程式

首先先來看這個專案的執行結果：

* 按下 F5 鍵或點擊「開始」按鈕來執行程式
* 底下為實際操作過程的輸出文字

```
開始遍歷 DICOM 標籤...
標籤: (0008,0005) (Specific Character Set) - 值: (0008,0005) CS Specific Character Set
標籤: (0008,0008) (Image Type) - 值: (0008,0008) CS Image Type
標籤: (0008,0012) (Instance Creation Date) - 值: (0008,0012) DA Instance Creation Date
標籤: (0008,0013) (Instance Creation Time) - 值: (0008,0013) TM Instance Creation Time
標籤: (0008,0016) (SOP Class UID) - 值: (0008,0016) UI SOP Class UID
標籤: (0008,0018) (SOP Instance UID) - 值: (0008,0018) UI SOP Instance UID
標籤: (0008,0020) (Study Date) - 值: (0008,0020) DA Study Date
標籤: (0008,0022) (Acquisition Date) - 值: (0008,0022) DA Acquisition Date
標籤: (0008,0023) (Content Date) - 值: (0008,0023) DA Content Date
標籤: (0008,0030) (Study Time) - 值: (0008,0030) TM Study Time
標籤: (0008,0032) (Acquisition Time) - 值: (0008,0032) TM Acquisition Time
標籤: (0008,0033) (Content Time) - 值: (0008,0033) TM Content Time
標籤: (0008,0060) (Modality) - 值: (0008,0060) CS Modality
標籤: (0008,1030) (Study Description) - 值: (0008,1030) LO Study Description
開始序列: (0008,1032) (Procedure Code Sequence)
  序列項目開始 >>>
標籤: (0008,0100) (Code Value) - 值: (0008,0100) SH Code Value
標籤: (0008,0102) (Coding Scheme Designator) - 值: (0008,0102) SH Coding Scheme Designator
標籤: (0008,0104) (Code Meaning) - 值: (0008,0104) LO Code Meaning
  <<< 序列項目結束
序列結束
標籤: (0008,103e) (Series Description) - 值: (0008,103e) LO Series Description
開始序列: (0008,1111) (Referenced Performed Procedure Step Sequence)
  序列項目開始 >>>
標籤: (0008,1150) (Referenced SOP Class UID) - 值: (0008,1150) UI Referenced SOP Class UID
標籤: (0008,1155) (Referenced SOP Instance UID) - 值: (0008,1155) UI Referenced SOP Instance UID
  <<< 序列項目結束
序列結束
標籤: (0010,0010) (Patient's Name) - 值: (0010,0010) PN Patient's Name
標籤: (0010,0020) (Patient ID) - 值: (0010,0020) LO Patient ID
標籤: (0010,1010) (Patient's Age) - 值: (0010,1010) AS Patient's Age
標籤: (0018,0010) (Contrast/Bolus Agent) - 值: (0018,0010) LO Contrast/Bolus Agent
標籤: (0018,0022) (Scan Options) - 值: (0018,0022) CS Scan Options
標籤: (0018,0050) (Slice Thickness) - 值: (0018,0050) DS Slice Thickness
標籤: (0018,0060) (KVP) - 值: (0018,0060) DS KVP
標籤: (0018,0088) (Spacing Between Slices) - 值: (0018,0088) DS Spacing Between Slices
標籤: (0018,0090) (Data Collection Diameter) - 值: (0018,0090) DS Data Collection Diameter
標籤: (0018,1030) (Protocol Name) - 值: (0018,1030) LO Protocol Name
標籤: (0018,1100) (Reconstruction Diameter) - 值: (0018,1100) DS Reconstruction Diameter
標籤: (0018,1120) (Gantry/Detector Tilt) - 值: (0018,1120) DS Gantry/Detector Tilt
標籤: (0018,1130) (Table Height) - 值: (0018,1130) DS Table Height
標籤: (0018,1140) (Rotation Direction) - 值: (0018,1140) CS Rotation Direction
標籤: (0018,1151) (X-Ray Tube Current) - 值: (0018,1151) IS X-Ray Tube Current
標籤: (0018,1152) (Exposure) - 值: (0018,1152) IS Exposure
標籤: (0018,1160) (Filter Type) - 值: (0018,1160) SH Filter Type
標籤: (0018,1210) (Convolution Kernel) - 值: (0018,1210) SH Convolution Kernel
標籤: (0018,5100) (Patient Position) - 值: (0018,5100) CS Patient Position
標籤: (0020,000d) (Study Instance UID) - 值: (0020,000d) UI Study Instance UID
標籤: (0020,000e) (Series Instance UID) - 值: (0020,000e) UI Series Instance UID
標籤: (0020,0011) (Series Number) - 值: (0020,0011) IS Series Number
標籤: (0020,0013) (Instance Number) - 值: (0020,0013) IS Instance Number
標籤: (0020,0032) (Image Position (Patient)) - 值: (0020,0032) DS Image Position (Patient)
標籤: (0020,0037) (Image Orientation (Patient)) - 值: (0020,0037) DS Image Orientation (Patient)
標籤: (0020,0052) (Frame of Reference UID) - 值: (0020,0052) UI Frame of Reference UID
標籤: (0020,1041) (Slice Location) - 值: (0020,1041) DS Slice Location
標籤: (0020,4000) (Image Comments) - 值: (0020,4000) LT Image Comments
標籤: (0028,0002) (Samples per Pixel) - 值: (0028,0002) US Samples per Pixel
標籤: (0028,0004) (Photometric Interpretation) - 值: (0028,0004) CS Photometric Interpretation
標籤: (0028,0010) (Rows) - 值: (0028,0010) US Rows
標籤: (0028,0011) (Columns) - 值: (0028,0011) US Columns
標籤: (0028,0030) (Pixel Spacing) - 值: (0028,0030) DS Pixel Spacing
標籤: (0028,0100) (Bits Allocated) - 值: (0028,0100) US Bits Allocated
標籤: (0028,0101) (Bits Stored) - 值: (0028,0101) US Bits Stored
標籤: (0028,0102) (High Bit) - 值: (0028,0102) US High Bit
標籤: (0028,0103) (Pixel Representation) - 值: (0028,0103) US Pixel Representation
標籤: (0028,1050) (Window Center) - 值: (0028,1050) DS Window Center
標籤: (0028,1051) (Window Width) - 值: (0028,1051) DS Window Width
標籤: (0028,1052) (Rescale Intercept) - 值: (0028,1052) DS Rescale Intercept
標籤: (0028,1053) (Rescale Slope) - 值: (0028,1053) DS Rescale Slope
標籤: (0028,2110) (Lossy Image Compression) - 值: (0028,2110) CS Lossy Image Compression
標籤: (0028,2112) (Lossy Image Compression Ratio) - 值: (0028,2112) DS Lossy Image Compression Ratio
標籤: (0040,0007) (Scheduled Procedure Step Description) - 值: (0040,0007) LO Scheduled Procedure Step Description
開始序列: (0040,0008) (Scheduled Protocol Code Sequence)
  序列項目開始 >>>
標籤: (0008,0100) (Code Value) - 值: (0008,0100) SH Code Value
標籤: (0008,0102) (Coding Scheme Designator) - 值: (0008,0102) SH Coding Scheme Designator
標籤: (0008,0104) (Code Meaning) - 值: (0008,0104) LO Code Meaning
  <<< 序列項目結束
序列結束
標籤: (0040,0009) (Scheduled Procedure Step ID) - 值: (0040,0009) SH Scheduled Procedure Step ID
標籤: (0040,0254) (Performed Procedure Step Description) - 值: (0040,0254) LO Performed Procedure Step Description
開始序列: (0040,0260) (Performed Protocol Code Sequence)
  序列項目開始 >>>
標籤: (0008,0100) (Code Value) - 值: (0008,0100) SH Code Value
標籤: (0008,0102) (Coding Scheme Designator) - 值: (0008,0102) SH Coding Scheme Designator
標籤: (0008,0104) (Code Meaning) - 值: (0008,0104) LO Code Meaning
  <<< 序列項目結束
序列結束
開始序列: (0040,0275) (Request Attributes Sequence)
  序列項目開始 >>>
標籤: (0040,0007) (Scheduled Procedure Step Description) - 值: (0040,0007) LO Scheduled Procedure Step Description
開始序列: (0040,0008) (Scheduled Protocol Code Sequence)
  序列項目開始 >>>
標籤: (0008,0100) (Code Value) - 值: (0008,0100) SH Code Value
標籤: (0008,0102) (Coding Scheme Designator) - 值: (0008,0102) SH Coding Scheme Designator
標籤: (0008,0104) (Code Meaning) - 值: (0008,0104) LO Code Meaning
  <<< 序列項目結束
序列結束
標籤: (0040,0009) (Scheduled Procedure Step ID) - 值: (0040,0009) SH Scheduled Procedure Step ID
標籤: (0040,1001) (Requested Procedure ID) - 值: (0040,1001) SH Requested Procedure ID
  <<< 序列項目結束
序列結束
標籤: (0040,1001) (Requested Procedure ID) - 值: (0040,1001) SH Requested Procedure ID
開始片段: (7fe0,0010) (Pixel Data)
  片段項目: 89124 位元組
片段結束
DICOM 標籤遍歷完成
DICOM 檔案 image-00000.dcm 總共含有 86 個標籤
```



