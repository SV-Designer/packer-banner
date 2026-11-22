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

`SKILL.md` 是**流程骨架**（步驟順序、停點、防呆清單），每步可直接抄的指令與程式碼拆在 `references/`：

| 檔 | 步驟 |
|---|---|
| `references/mail-and-month.md` | 0～1 建當月工作區、讀需求信（三個路徑變數在這裡設） |
| `references/notion-card.md` | 2～3 文案定稿、Notion 開頁（使用者自開卡就只寫內文） |
| `references/assets.md` | 4～5 Seafile 下載、壓縮與流水號命名 |
| `references/figma-build.md` | 6～10b Figma 建 page、置圖、填字、H1/H2、命名、開 Figma、G3 |
| `references/export.md` | 11～13 輸出、G4 確認後搬到最終交付夾、收尾 |

模版結構、node id、字重對應表、文案整理規則的正本在**專案自己的 `CLAUDE.md`**，刻意不複製進 skill（避免同一套規則養兩份而分岔）。
⇒ 直接裝上去不會馬上能跑，要先照最上面的佔位符表補齊路徑（`SKILL.md` 與 `references/*.md` 都要換），並自備一份專案 `CLAUDE.md`。

路徑變數可在 shell 設定檔先 `export PROJ_ROOT=… DELIV_ROOT=… FINAL_ROOT=…`；沒設的話步驟 0 會直接報錯停下，不會寫到錯的地方。

## 需要的 MCP

Gmail、Notion、Figma（含 `use_figma`／`upload_assets`）。

## 安裝

```bash
git clone <this repo> ~/.claude/skills/packer-banner
```

## 授權

MIT
