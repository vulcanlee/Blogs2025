# 巨明 CanWellBeing SMART on FHIR 應用系統介紹

名稱：SMART on FHIR 身體組成 AI 推論 App（L3 CT）應用說明文件

本 App 為**醫護端** SMART on FHIR 應用，用於快速取得**腹部 L3 橫切面 CT** 的身體組成分析結果（骨骼肌/脂肪等），協助臨床評估肌少症風險、肌肉品質（肌肉脂肪變性）與營養/預後相關指標。

## SMART on FHIR App 示範影片

![](./Mega_Radiance_SMART_on_FHIR_App.mp4)

# 使用情境
1. 使用者使用 Standalone 方式啟動這個系統
2. 系統將會進行 OAuth2 授權碼取得
3. 接著需要進行身分驗證
4. 完成後，就會取得 Access Token
5. 使用者輸入病歷號，便會到 FHIR Server 上查詢病人基本資料與身高、體重
6. 使用者選擇或上傳 L3 CT DICOM 影像
7. 點擊「AI 推論」按鈕，系統會呼叫後端推論服務
8. 推論完成後，系統會顯示結果：原始 CT 影像、分割覆蓋圖與指標數值表



