# 如何新增裝備

## 腳本位置

CT 是 UTF-8 XML。使用文字編輯器開啟，找到 ID 為 `3` 的 `CheatEntry`，其 `AssemblerScript` 是終盤裝備新增腳本。也可在 Cheat Engine 右鍵該項目編輯腳本。

腳本中的清單每項為 `{content ID, '英文識別名稱', 目標總數}`。英文名稱只用於日誌，不決定物品種類。

```lua
local items={
 {70,'Excalibur',4},
 {71,'Ragnarok',4},
 {76,'Masamune',4},
 {83,'Elder Staff',4},
 {88,'Omnirod',4},
 {161,'Ribbon',4},
 {178,'Black Robe',4},
 {179,'White Robe',4},
 {188,'Crystal Mail',4},
 {199,'Protect Ring',4},
 {201,'Crystal Gloves',4},
 {128,'Shuriken',99}
}
```

新增已確認的物品時，在清單加入同格式一列，注意逗號分隔。不要猜 ID，或把武器／防具資料表自身的 ID 當作 content ID。

## 查出正確 ID

本版 Unity 資料位於遊戲目錄 `FINAL FANTASY III_Data/StreamingAssets/aa/StandaloneWindows64/`。以 Unity 資料工具（例如 UnityPy）讀取 master 與 message bundle 中的 TextAsset。

1. 在 message 的 `system_en` 或 `system_zht` 找裝備名稱及訊息鍵。
2. 在 master 的 `content` 表，依 `mes_id_name` 找對應列。
3. 該列的 `id` 才是新增物品需要的 content ID。
4. `type_value` 指向 weapon 或 armor 表的資料，用於核對屬性、可裝備職業等，不是新增物品的 ID。

Bundle 名称帶有版本雜湊，更新後應重新定位。部分匯出的 CSV 使用字面上的 `\r\n`，解析前需轉成真正換行。訊息名稱可能帶有 `<IC_SRD>` 等圖示標記。

例如本版諸神黃昏的 content ID 是 71，weapon 資料的 ID 是 22；傳入新增 API 的是 71。

## 新增機制

腳本透過 Cheat Engine Mono/IL2CPP 功能動態尋找方法，呼叫遊戲原本的物品管理功能：

- `UserDataManager.Instance()`：取得目前遊戲資料。
- `SearchOwnedItem(System.Int32)`：依 content ID 查詢物品。
- `OwnedItemData.get_Count()`：總持有量，包含已裝備數量。
- `OwnedItemClient.AddOwnedItem(System.Int32,System.Int32)`：加入差額。

計算式為 `max(0, 目標數量 - 目前總數)`。不要再把 `get_EquipCount()` 加到 `get_Count()`，否則會重複計算已裝備物品。此選項不呼叫 `SwitchJob` 或 `SetEquip`。

方法有多載，解析時必須比對完整參數型別。CT 的 `[ENABLE]` 執行一次；`[DISABLE]` 不撤銷物品，也不會持續鎖定數量。

## 測試與發布

1. 先備份存檔並停在地圖；從 CT 載入新增選項測試，不能只測外部 Lua 原型。
2. 記錄執行前後數量，讀取遊戲回傳名稱，核對是否為目標物品。
3. 再執行一次，應維持目標數量；本來超過目標時應保留原數。
4. 補裝備測試和職業穿戴、戰鬥、存檔重讀是不同測試，文件只記錄確實完成的部分。
5. 發布 CT 和文件，不上傳個人存檔、遊戲 DLL、metadata 或完整擷取資料。

## 遊戲更新

目前僅針對 Steam build 20264899。腳本檢查 GameAssembly.dll 的 PE timestamp `1738594894` 與 SizeOfImage `39026688`，這是相容性檢查，並非完整檔案雜湊。更新後不能只刪除檢查：應重新確認 content ID、方法签名、數量語意，再完成遊戲測試後更新版本檢查。

本版已整理的 [裝備 ID 對照表](EQUIPMENT_IDS.md) 可直接查繁中／英文名稱；也提供 [CSV](equipment_ids.csv)。請使用 `content_id`，不是 `weapon_or_armor_id`。
