# 使用者資料持久化說明

## 概述

本 LINE Bot 使用 **JSON 檔案儲存**來持久化使用者資料，確保伺服器重啟後使用者不需要重新設定個人資料。

## 儲存機制

### 📁 資料目錄
- **位置**: `data/` 目錄
- **檔案**:
  - `user_profiles.json` - 使用者個人資料（姓名、年齡、性別、糖尿病類型等）
  - `user_modes.json` - 使用者模式設定（知識庫模式 vs 個人模式）

### 🔄 自動儲存機制
- **觸發時機**: 每次更新使用者資料或切換模式時自動儲存
- **啟動載入**: 應用程式啟動時自動從 JSON 檔案載入所有使用者資料
- **即時同步**: 資料變更立即寫入檔案，無需手動操作

### 🛡️ 資料保護
- `data/` 目錄已加入 `.gitignore`，不會上傳到 GitHub
- 使用者個人資訊完全保存在部署環境的本地檔案系統
- JSON 格式使用 UTF-8 編碼，支援中文字元

## 資料結構範例

### user_profiles.json
```json
{
  "U1234567890abcdef": {
    "name": "王先生",
    "age": "45",
    "gender": "男性",
    "diabetes_type": "第2型",
    "complications": ["視網膜病變", "腎臟病變"],
    "education_level": "大學",
    "current_medications": ["Metformin", "胰島素"]
  },
  "U9876543210fedcba": {
    "name": "李小姐",
    "age": "32",
    "gender": "女性",
    "diabetes_type": "第1型",
    "complications": [],
    "education_level": "研究所",
    "current_medications": ["長效胰島素", "速效胰島素"]
  }
}
```

### user_modes.json
```json
{
  "U1234567890abcdef": "knowledge",
  "U9876543210fedcba": "personal"
}
```

## 部署注意事項

### Zeabur 部署
- ✅ **自動創建**: `data/` 目錄會在首次運行時自動創建
- ✅ **持久化**: Zeabur 環境支援檔案系統持久化
- ⚠️ **備份建議**: 定期備份 `data/` 目錄以防資料遺失

### 本地開發
1. 首次運行會看到: `ℹ️ No existing user profiles file found, starting fresh`
2. 使用者設定資料後會看到: `💾 Saved X user profiles to data/user_profiles.json`
3. 重啟應用程式後會看到: `✅ Loaded X user profiles from data/user_profiles.json`

## 資料管理

### 查看資料
```bash
# 查看所有使用者資料
cat data/user_profiles.json

# 查看使用者模式設定
cat data/user_modes.json
```

### 備份資料
```bash
# 創建備份
cp -r data/ data_backup_$(date +%Y%m%d)/

# 或壓縮備份
tar -czf data_backup_$(date +%Y%m%d).tar.gz data/
```

### 還原資料
```bash
# 從備份還原
cp -r data_backup_20250110/ data/
```

### 清除所有資料（謹慎使用）
```bash
rm -rf data/
```

## 常見問題

### Q: 伺服器重啟後使用者資料會遺失嗎？
**A**: 不會！所有資料都儲存在 JSON 檔案中，重啟後會自動載入。

### Q: 使用者資料會上傳到 GitHub 嗎？
**A**: 不會！`data/` 目錄已加入 `.gitignore`，不會被版本控制。

### Q: 如何遷移到其他伺服器？
**A**: 複製整個 `data/` 目錄到新伺服器即可。

### Q: 支援多少使用者？
**A**: JSON 檔案可支援數千個使用者。如需更大規模，建議升級到資料庫（SQLite、PostgreSQL 等）。

### Q: onboarding_state 為什麼不持久化？
**A**: Onboarding 流程是臨時性的對話狀態，不需要持久化。使用者完成設定後會自動儲存到 `user_profiles.json`。

## 未來升級選項

如果使用者數量增長，可考慮升級到：
- **SQLite**: 單檔案資料庫，無需額外服務
- **PostgreSQL**: 專業級關聯式資料庫
- **Redis**: 快取層 + 持久化儲存
- **MongoDB**: NoSQL 文件資料庫

當前 JSON 方案適合中小型應用（<1000 使用者），簡單可靠且易於維護。
