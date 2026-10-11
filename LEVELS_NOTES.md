# 等級與職業 CT 維護筆記

適用 Steam build 20264899。兩個 CT 都是獨立完整腳本，載入存檔後在地圖執行一次，取消勾選不會撤銷變更。請另存新欄位。

## Lv99

`FF3_PR_Level99_20264899.CT` 處理目前隊伍四人。角色經驗值目標為本版 exp_table 的 level 99 門檻 9,999,999。

有效流程：`OwnedCharacterClient.AddExp` → FF3 專用 `StatusUpProvider.Execution` → `SetOwnedCharacter` → `UpdateParameter`，最後重新讀取等級與最大 HP。Execution 使用固定亂數種子 12345，battleSkillUpInformation 傳入 null；本次測試可正常完成角色升級。這不是 HP9999 修改器，HP 成長依遊戲處理器與目前職業計算。

踩雷紀錄：

- 只呼叫 AddExp：經驗值增加，但第一人仍 Lv3、最大 HP46。AddExp 回傳角色物件不代表完成升級。
- 只呼叫 UpdateParameter，或把 AddExp 回傳物件交给 SetOwnedCharacter：都沒有完成升級。
- 必須呼叫 `Serial.FF3.Management.StatusUpProvider.Execution` 才完成原生升級結算，無須先打怪。
- 不應單憑經驗值或 CT 勾選成功判定升級成功，應檢查角色等級與成長後能力。

## 已解鎖職業熟練度99

`FF3_PR_UnlockedJobs99_20264899.CT` 讀取 UserDataManager.ReleasedJobs，再遍歷四人的 OwnedJobDataList，只修改解鎖清單中且已存在的職業資料。

本版職業經驗表 Lv99 門檻為 9702；設定 CurrentProficiency=9702、Level=99。腳本確認角色等級與目前職業不變，並比對前後解鎖 ID 清單。不建立未解鎖職業、不提前解鎖職業。取得新職業後可重跑。

## 實測紀錄與限制

測試環境：Mac CrossOver、Cheat Engine、同一已載入存檔。

- 有效升級腳本讓四人由 Lv3 升到 Lv99，EXP 各9,999,999，最大 HP 分別為 6613、6591、6607、6553。
- 職業 CT 實際成功執行，四人各6個已解鎖職業，共24筆達99；解鎖清單仍為1、2、4、5、6、7。
- 未測試存檔重讀、戰鬥、取得新職業後重跑或其他遊戲版本。不能把這些列為已驗證。

其他先前踩雷：裝備欄位3為身體、4為頭部；OwnedItemData.Count 已包含裝備中的数量，不可再加 EquipCount；裝備欄位與背包可能回傳不同物件，不能只比較記憶體地址判定物品相同。

追加踩雷與最終驗證：

- 只改 OwnedJobData 的屬性，當下讀取99不代表已提交到角色資料；重新從隊伍取得角色時曾讀回1。修正方式為回寫 OwnedJobDataList，再以 OwnedCharacterClient.SetOwnedCharacter 提交。
- 調試期間曾有一次提交流程在第二名角色後返回失敗；其具體原因未確定，不能把失敗當作整隊成功。後續最終交付 CT 各連續執行兩次皆成功，並以獨立讀取流程重新取得四人資料確認結果。
- 最終回讀：四人等級99、經驗值9,999,999；各6個已解鎖職業皆99，解鎖 ID 仍為1、2、4、5、6、7。最大HP維持6613、6591、6607、6553。執行未完成時請查看 CE Lua 錯誤，重新核對整隊資料，不要以部分輸出推定完成。
