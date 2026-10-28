# Steam 成就獨立 CT

`FF3_PR_Achievements_20264899.CT` 提供五個獨立勾選項目，勾選哪一項就立即解鎖哪一項；沒有全選或自動執行腳本。

| 項目 | Steam ID |
|---|---|
| 傳奇寶物獵人（寶箱100%） | `treasure_box_367` |
| 道具獵人 | `get_hidden_item` |
| 探求的冒險者 | `area_visit` |
| 魔物圖鑑・完全攻略篇 | `picture_book_225` |
| FF3 大師！ | `all_achievement` |

## 使用

1. Steam 登入要解鎖成就的帳號，啟動 FF3 Pixel Remaster，載入存檔並停在地圖。
2. CE 連接 `FINAL FANTASY III.exe`，開啟此 CT。
3. 勾選需要的成就。勾選即執行，取消勾選不會撤銷成就；重跑先取消再勾選。
4. 在 Steam 成就頁確認結果。Lua Engine 會輸出 `ACHIEVEMENT VERIFIED`，錯誤時不要只看勾選框判定成功。

這份 CT 只設定指定 Steam 成就，不修改存檔中的開箱旗標、隱藏物品、造訪地點或圖鑑。因此 Steam 成就可能已解鎖，但遊戲內回收率仍不足100%。「FF3 大師！」只設定這一枚成就，不會連帶解鎖其他成就。沒有還原或清除成就功能。

## 實作與維護

全部 Lua 內嵌在 CT，不依賴本機路徑或外部腳本。保留 build 20264899 的 GameAssembly PE 檢查。使用 Mono collector 呼叫遊戲內 Steamworks 函式：列舉 `GetNumAchievements`／`GetAchievementName` 核對 ID，對單一 ID 呼叫 `SetAchievement`，再呼叫 `StoreStats`。兩個回傳值都必須是 true 或 1；Lua 的數字 0 仍為真，不能直接用 `if result`。

接著取得現有 `SteamSDKManager` singleton，以 `GetAllAchievementsOnPlatform` 回讀目標值為1。這是客戶端回讀；StoreStats 接受請求不等於已另外確認遠端同步，應以 Steam 成就頁為準。不要建立新的 SDK manager、重設全部統計，或遍歷所有 ID 進行解鎖。

## 驗證（2026-10-29）

Mac CrossOver／CE／Steam build 20264899。原存檔開箱數242；寶箱成就由0改為1，使用者確認看到解鎖。另四項也逐一回讀為1。

封裝後從 CT 抽出五段原樣 Lua，在同一遊戲程序中執行，確認已解鎖狀態可重複執行、各目標回讀通過。XML 結構與五個 ID 已檢查。未重新鎖定帳號成就測試首次解鎖，也未測其他版本或 Steam 離線同步。
