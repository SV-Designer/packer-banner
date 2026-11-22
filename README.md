# packer-banner

官網 banner 月更的 Claude Code skill：Gmail 讀需求信 → Notion 開接單卡（只填內文）→ Figma 套版完稿 → 輸出 JPG 到交付夾。

**觸發詞**：「做派歌官網圖」

## 這是去識別化的公開版

原始版本含公司內部真值。本版已把下列項目換成佔位符，**安裝後必須自行補上**：

| 佔位符 | 意思 |
|---|---|
| `$PROJ_ROOT` | 你的專案資料夾絕對路徑（放素材暫存、文案定稿、`_scripts/`） |
| `$DELIV_ROOT` | 成品 JPG 的輸出暫存夾絕對路徑（G4 確認前先放這） |
| `$FINAL_ROOT` | 最終交付夾絕對路徑（G4 確認 OK 後整個月份夾搬過去；不存在會停下，不自動建） |
| `<FIGMA_FILE_KEY>` | Figma 檔案 key |
| `<NOTION_DATA_SOURCE_ID>` | Notion 接單資料庫的 data_source_id |
| `<SEAFILE_HOST>` | 素材下載用的 Seafile 主機 |
| `<上個月接單卡網址>` | 用來當範本／備援查 data_source_id |

## ⚠️ 這支 skill 不是自足的

`SKILL.md` 是**流程正本**（步驟順序、防呆、可直接抄的程式碼）。
模版結構、node id、字重對應表、文案整理規則的正本在**專案自己的 `CLAUDE.md`**，刻意不複製進 skill（避免同一套規則養兩份而分岔）。
⇒ 直接裝上去不會馬上能跑，要先照上表補齊路徑，並自備一份專案 `CLAUDE.md`。

## 需要的 MCP

Gmail、Notion、Figma（含 `use_figma`／`upload_assets`）。

## 安裝

```bash
git clone <this repo> ~/.claude/skills/packer-banner
```

## 授權

MIT
