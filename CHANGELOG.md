# Changelog

## [0.1.1] - 2026-11-21

### Changed
- skill 名稱拿掉 `claude-` 前綴，名字更短、跟其他 skill 一致。

### Fixed
- `.gitignore` 加擋 `/CLAUDE.md`：維護筆記只給本機用，避免不小心收進版控。

## v0.1.0 — 2026-09-02

首次公開發布（去識別化版）。

- 官網 banner 月更全流程 skill：Gmail 讀信 → Notion 開接單卡（只填內文）→ Figma 套版完稿 → 輸出 JPG 到交付夾
- 四個人在環中停點：G1 問月份／G2 開 Figma／G3 調顏色核對／G4 確認完稿路徑
- 內含三段機械化流程的可直接執行程式碼：素材下載壓縮命名、clone＋swapComponent、批次輸出驗尺寸
- 公司內部真值（絕對路徑、Figma fileKey、Notion data_source_id、Seafile 主機與 token、人名、信件主旨）已全部替換為佔位符，安裝方式見 README
