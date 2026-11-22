---
name: packer-banner
description: >-
  派歌（Packer）官網 banner 月更全流程：Gmail 讀需求信 → 在 Notion 開頁面填內文（欄位由使用者手動填）→
  Figma 套版完稿 → 輸出 JPG 到交付夾。觸發詞：「做派歌官網圖」（可直接說「做 10 月派歌官網圖」）。
  需要 Gmail MCP、Notion MCP、Figma MCP（含 use_figma／upload_assets）。
---

# 派歌官網 banner 月更

每月把派歌版權管理部的需求信，變成 Notion 接單卡＋一批官網 banner。

## 🔴 開跑前必讀（正本在專案文件，不在這裡）

**先讀 `$PROJ_ROOT/CLAUDE.md`。**

本 skill **刻意不複製**模版結構、node id、字重對應表、文案整理規則——那些的正本在專案文件。
理由：同一套規則養兩份必然分岔（模版重製過，id 會全換）。
**本檔只負責「照什麼順序做、每步的防呆是什麼」。**
本 skill 是套版，不走「出 3 個設計變體」那類設計流程。

## 先決條件

| 需要 | 用途 | 沒有的話 |
|---|---|---|
| Gmail MCP | 讀需求信 | 請使用者把信件內文貼進來 |
| Notion MCP | 寫接單卡內文 | **僅限 MCP 失效時**：改成產出文字給使用者手動貼（正常情況 Claude 直接寫內文） |
| Figma MCP | 讀結構、寫內容、上傳圖 | **無法繼續**，停下請使用者確認 Figma MCP 已登入（桌面版是 G2 使用者看稿調色用，不是前置條件） |
| macOS `sips` | 壓縮素材、驗尺寸 | 系統內建，通常都有 |

⚠️ **動 `use_figma` 前一定要先載入 `/figma-use` skill**（官方硬性要求）。

✅ Figma MCP 走雲端，**Claude 這端不需要使用者開桌面版**；G2 開桌面版純粹是讓使用者看稿與調顏色（背景見 `${CLAUDE_SKILL_DIR}/references/figma-build.md` 開頭）。

📁 **路徑變數**（使用者自己設定，見 README）：`$PROJ_ROOT`＝專案資料夾、`$DELIV_ROOT`＝輸出暫存夾（G4 前）、`$FINAL_ROOT`＝最終交付夾（G4 OK 後搬過去）。

📖 **reference 分工**（每步動手前先讀對應那份，裡面是可直接抄的指令與程式碼）。下表路徑會自動展開，直接讀；references 檔內文寫的 `<SKILL_DIR>` 代表本 skill 資料夾，照著換成實際路徑再執行（references 不會自動替換）：

| 步驟 | 讀這份 |
|---|---|
| 0～1 建工作區、讀信 | `${CLAUDE_SKILL_DIR}/references/mail-and-month.md` |
| 2～3 文案定稿、Notion 開頁 | `${CLAUDE_SKILL_DIR}/references/notion-card.md` |
| 4～5 Seafile 下載、壓縮命名 | `${CLAUDE_SKILL_DIR}/references/assets.md` |
| 6～10b Figma 建 page、置圖、填字、H1/H2、命名、開 Figma、G3 | `${CLAUDE_SKILL_DIR}/references/figma-build.md` |
| 11～13 輸出、G4、收尾 | `${CLAUDE_SKILL_DIR}/references/export.md` |

## 停點 G1～G4（G1／G2 可自動略過，G3／G4 一定要等使用者）

| # | 時機 | 提示語 | 可以不停的條件 |
|---|---|---|---|
| **G1** | 讀完信 | 「本次製作幾月官網？」 | **信件內文有提到月份**，或**使用者觸發時就講了**（例：「做 10 月派歌官網」）→ 直接用，不問 |
| **G2** | 內容全部套進 Figma 後 | （不是提問，是動作）**Claude 直接開啟 Figma 桌面版並跳到當月 page** | **偵測到 Figma 已在執行** → 不重開 |
| **G3** | G2 之後 | 「**下一步**：1. 請手動調整顏色 ＆ 核對內容，確認無誤打 OK 就跑輸出」 | ⛔ **無條件停** |
| **G4** | 輸出完成後 | 「**下一步**：1. 確認圖檔 OK（已存：`<$DELIV_ROOT/{月}月 絕對路徑>`），確認 OK 後移動到 `<$FINAL_ROOT 絕對路徑>` 中」 | ⛔ **無條件停**，OK 後才搬移＋收尾 |

> 📌 G3 使用者調的是**顏色**（`BG-color`／`BG-shadow`），不是字重；他也可能重排速報視覺順序，**不影響流程**。

---

# 步驟（骨架）

| # | 做什麼 | 關鍵規則 |
|---|---|---|
| **0** | 建當月工作區（`$PROJ/$M`、`素材暫存/$M/{速報,推薦,案例,_上傳用}`、`$DELIV/$M/{1-發行速報,2-發行推薦,3-發行案例}`） | 月份偵測：觸發語 → 信件內文正則 → 都沒有才問。🚫 不用來信日期推定 |
| **1** | 讀需求信 ⛔ G1 | `search_threads('subject:派歌 官網')` → `get_thread(PLAIN_TEXT)`，**讀整串**；建議外包 subagent（`general-purpose`，`model: haiku`） |
| **2** | 整理文案 → `$PROJ/{月}月/文案定稿.md` | 簡體錯字直接改並說一聲；溝通題不進卡但口頭提醒；張數回信件重數 |
| **3** | Notion「今晚我想來點」開新頁面（**使用者已貼自開卡網址 → 不開新卡，`notion-update-page` `replace_content` 只寫內文**，見 `${CLAUDE_SKILL_DIR}/references/notion-card.md`） | `properties` **只帶 `Name`**；內文照上個月卡骨架（先 `notion-fetch` 現查）；案例三層切法 |
| **4** | 下載 Seafile 素材 | 優先 `_scripts/parse_mail.py` ＋ `seafile_fetch.sh`；跑完核對 `清單.md` 的 tw／cn 分界 |
| **5** | 壓縮＋流水號命名 | `sips -Z 1600 q88`；速報順序**完全照信件**；順序固定「速報 → 推薦 → 案例」 |
| **6** | 建當月 Figma page | 用 id `18:386` 定位模版 → clone section → **建對照表＋`swapComponent`** → 驗 Artist 接線 → `get_metadata` 重取 id |
| **7** | 置入圖片 | `upload_assets(..., scaleMode='FILL')`，目標＝元件本身 |
| **8** | 填文字 | 替代字型保險邏輯；一次 ≤ 6 節點；名稱白名單；案例 CN `Desc` 固定寬 210 |
| **8b** | Title H1／H2 | 量**內部文字**寬 > `Context` 寬（220）就切 H2，切後文字 id 會變 |
| **9** | frame 命名 | 括號填當月音樂人（CN 簡體）；速報無括號；案例前後空格保留 |
| **10** | 開 Figma ⛔ G2 | `lsof -iTCP:3845` 偵測，已開不重開；否則 `open "figma://file/<FIGMA_FILE_KEY>?node-id=<page id>"` |
| **10b** | 請使用者調顏色核對 ⛔ G3 | 提示語固定：「**下一步**：1. 請手動調整顏色 ＆ 核對內容，確認無誤打 OK 就跑輸出！」 |
| **11** | 輸出 | 優先 `_scripts/export_banners.sh <月份> <tsv>`；`get_screenshot` → PNG → `sips` JPEG **q100**；檔名 rstrip；逐檔驗尺寸 |
| **12** | 交付確認 ⛔ G4 → 搬移 | 提示「**下一步**：1. 確認圖檔 OK（已存：`$DELIV_ROOT/{月}月/`），確認 OK 後移動到 `$FINAL_ROOT` 中」；收到 OK 才 `mv` 到 `$FINAL_ROOT/{月}月/`（目的地不存在就停下回報） |
| **13** | 收尾 | 張數三方驗算、繁簡逐字比對、CN variant 檢查、回覆第一行「**下一步**：」列兩件手動事項 |

> 專案文件 `$PROJ_ROOT/CLAUDE.md` 的〈執行手冊〉與本骨架重疊，可能已過期，**指令以本 skill 的 references 為準**；模版結構／node id／字重表／文案規則以專案文件〈分析一〉〈分析二〉為準。

---

## 🕳️ 已知地雷／必檢查（全部實測踩過，細節與程式在對應 reference）

**讀信與文案**
- 主旨每月一樣且不含月份、新月份常回在同一 thread → **靠日期區分**
- thread 第一封是轉寄信 → `需求負責人` 從轉寄內文 `寄件者：` 抓
- 🚫 不用 `FULL_CONTENT`（每封約 2.7MB 灌爆 context）
- 信裡 PNG 附件不是素材，素材一律走 Seafile
- 張數：「共 N 張」是**中文數字**；速報有**兩個**「共兩張」，要抓全部再加總
- 🚫 `9月/文案定稿.md` 說案例只有兩層是錯的，案例是三層（Title／Song／Desc）

**素材**
- 🚫 不按檔名排序、不為視覺重排 —— 實測會錯位
- 信裡若明列 TW／CN 兩組（1000×1000 段），`順序.txt` 照分組寫，不照八人清單
- 🕳️ 一定用流水號命名：`upload_assets` 靠陣列順序對應 `nodeIds`
- Seafile 要求登入 → 不自己找帳密；🚫 不用上個月的網址

**Figma**
- 🔴 模版 `Templete` 不直接做當月；🚫 用名稱定位會抓到 `Templete(2025)`
- clone 出 **8 個** section（成品 4＋元件庫 4）
- 🕳️ 漏 `swapComponent` ＝換圖沒反應；🚫 不靠名稱配對；重新指向數量**每次不同、不可寫死當驗收**
- 🔴 Artist 接線：每個速報 frame `Type=1~8` 各用一次，**用程式數重複**；抓到重複不自己改模版、回報使用者
- 🔴 絕不沿用專案文件〈分析二〉的 id，clone 後一律重取
- 🚫 `createImageAsync` 不支援，只能 `upload_assets`
- 🕳️ 改字重不要 `loadFontAsync` 舊字型（雲端沒 ExtraBold／SemiBold 會整段失敗）
- 模版殘留上月文案 → **每個文字節點都明確覆寫**
- 🚫 不用「id 含 `;`」排除節點（會濾掉 Title），改用名稱白名單 `['Title','Song','Desc','Text-TW','Text-CN']`
- H1/H2：量 instance 寬永遠不成立（固定 196），**要量內部文字**
- `PingFang SC Semibold` 14 處在共用元件內，**刻意不動**
- 🔴 案例／推薦人數多於模版份數 → 在**當月 page** 多 clone 一對 section＋Work，用**獨立對照表** swap（見 figma-build 步驟 6）
- 🕳️ 長 Title 填字與量寬要**拆兩次呼叫**，否則丟 `node ... does not exist`；H2 仍放不下 → 單行保留、G3 點名，🚫 不塞 `\n`

**輸出**
- `get_screenshot` 只回短效 PNG 網址（視同機密，立刻下載、不落檔）
- `formatOptions` 要 **100** 不是 95（95 只有慣例 30–39% 大小）
- 檔名要 rstrip 結尾空白，但 frame 名本身的空格不去 Figma 清
- 陣列迴圈一律包 `bash -c '...'`（預設 zsh，`${!arr[@]}` 會 bad substitution）

---

## 停止條件（跑完自查）

- [ ] 交付夾三個子夾都有檔，總張數＝信件張數
- [ ] `sips` 逐檔驗過，像素與檔名吻合
- [ ] Notion 頁面在「今晚我想來點」（Claude 新開，或寫進使用者自開的那張），內文完整，全庫只有一張當月卡（欄位由使用者手動填）
- [ ] `文案定稿.md` 已落檔
- [ ] 簡體錯字已改並口頭說明；溝通題已口頭提醒
- [ ] G4 已收到 OK，成品已從 `$DELIV_ROOT/{月}月/` 搬到 `$FINAL_ROOT/{月}月/`，兩項手動提醒已送出

## 卡關規則

**同一步驟卡兩次就停下**，跟使用者簡述問題（使用者問細節才給錯誤原文），不要硬試第三次。
常見卡點與已知解都在專案文件的硬限制與「🕳️」標記處，先去那裡找。
