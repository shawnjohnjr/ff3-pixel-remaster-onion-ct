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
- 當時未測試存檔重讀、戰鬥、取得新職業後重跑或其他遊戲版本；後續職業與歌曲補測見下節。

其他先前踩雷：裝備欄位3為身體、4為頭部；OwnedItemData.Count 已包含裝備中的数量，不可再加 EquipCount；裝備欄位與背包可能回傳不同物件，不能只比較記憶體地址判定物品相同。

追加踩雷與最終驗證：

- 只改 OwnedJobData 的屬性，當下讀取99不代表已提交到角色資料；重新從隊伍取得角色時曾讀回1。修正方式為回寫 OwnedJobDataList，再以 OwnedCharacterClient.SetOwnedCharacter 提交。
- 調試期間曾有一次提交流程在第二名角色後返回失敗；其具體原因未確定，不能把失敗當作整隊成功。後續最終交付 CT 各連續執行兩次皆成功，並以獨立讀取流程重新取得四人資料確認結果。
- 最終回讀：四人等級99、經驗值9,999,999；各6個已解鎖職業皆99，解鎖 ID 仍為1、2、4、5、6、7。最大HP維持6613、6591、6607、6553。執行未完成時請查看 CE Lua 錯誤，重新核對整隊資料，不要以部分輸出推定完成。

## 吟遊詩人歌曲缺漏修正（2026-10-12）

問題：直接修改 Level／CurrentProficiency 不會完成職業技能學習。角色即使顯示熟練度 99，仍可能只有初始恢復歌曲。

本版 learning 表有三筆職業學習規則，均為 type_id=5、job_id=19：

| 熟練度門檻 | 歌曲 | Content ID | Ability ID |
|---|---|---|---|
| 10 | 攻擊之歌 | 358 | 108 |
| 20 | 守護之歌 | 359 | 109 |
| 30 | 破壞之歌 | 361 | 111 |

初始恢復歌曲為 Content 360／Ability 110。歌曲不是背包道具，不能透過 AddOwnedItem 學會。

修正流程：先提交已解鎖職業的熟練度，再檢查角色的 OwnedAbilityList。吟遊詩人已解鎖、角色有該職業資料且達門檻時，呼叫 OwnedCharacterClient.AddAbility(characterId, abilityId) 補入缺少的歌曲。這個函式同時處理實際持有技能及學習紀錄。最後重新取得角色，驗證 OwnedAbilityList、LearningAbilitys、既有技能、目前職業與等級，以及解鎖職業清單。新增後不可再用舊角色物件覆寫資料。

踩雷紀錄：

- 只補 LearningAbilitys 再 SetOwnedCharacter：四人的清單成功讀回 358、359、361，重跑也沒有新增，但使用者實測戰鬥仍沒有新歌。學習紀錄不等於實際可用技能。
- AddAbility 的第二參數是 Ability ID，不能傳 Content ID；傳 358 時回傳空值。依 master content 的對應改傳 108、109、111 後，第二位角色實際 OwnedAbilityList 成功讀回 358、359、360、361，各一筆。
- Lua 的數字 0 仍為真；檢查 mono 返回的 Boolean 時必須明確比對 true 或 1，不能直接 if result。

驗證結果：使用者確認第二位吟遊詩人在戰鬥歌唱選單中已出現全部四首歌曲。執行與交付 CT 相同的完整 Lua 腳本後，重新取得四位角色，確認每人 17 個已解鎖職業均為 99，三首學習歌曲各有一筆實際持有技能，既有技能、目前職業與角色等級未變；解鎖 ID 清單仍為 1、2、3、4、5、6、7、8、10、14、15、16、17、18、19、20、21。再次執行時四人新增歌曲數均為 0，所有回讀檢查通過。此次透過 Lua Engine 執行 CT 內同一份 Lua；CT XML 與腳本內容一致性另行檢查。未測存檔重讀或其他版本。
