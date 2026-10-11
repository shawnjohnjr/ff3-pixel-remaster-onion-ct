# 最大 HP 9999 CT

適用 Steam build 20264899。獨立檔案 `FF3_PR_MaxHP9999_20264899.CT`，在地圖上執行一次；取消勾選不還原。修改最大 HP，不會自動補滿目前 HP。

## 實作

1. 從 UserDataManager 取得目前隊伍四名角色。
2. 讀取角色 Parameter 的 ConfirmedMaxHp 與 AddtionalMaxHp（方法名稱原本就如此拼寫）。
3. 最大 HP 小於9999時，以「現有附加值 + 9999 - 目前最大HP」計算新附加值；已達9999時跳過，避免重複累加。
4. 呼叫 set_AddtionalMaxHp，透過 OwnedCharacterData.SetParameter 和 OwnedCharacterClient.SetOwnedCharacter 提交角色資料。
5. 重新從隊伍取得角色，UpdateParameter 後檢查 ConfirmedMaxHp=9999，並確認目前HP、等級與職業不變。

使用附加最大HP欄位，避免把職業基礎HP或裝備HP當成永久成長。不要只修改暫存物件後就宣稱成功；應重新取得角色驗證。這不是無敵或鎖血腳本，受到傷害仍會扣除目前HP。

## 限制

未驗證存檔重新載入、轉職、戰鬥或其他版本。低等級或有特殊HP裝備的情境也尚未測試。

## 實測結果

已從交付 CT 載入並執行，四人最大HP由6613、6591、6607、6553變成9999。重新取得角色並更新能力後仍為9999；目前HP維持46、45、47、46，等級與職業不變。再次載入執行，四人皆為9999→9999，回讀檢查全部通過。測試環境為本版遊戲、Mac CrossOver 與 Cheat Engine。
