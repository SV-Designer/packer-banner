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
| Figma MCP | 讀結構、寫內容、上傳圖 | **無法繼續**，停下請使用者確認 Figma 桌面版有開、Figma MCP 已登入 |
| macOS `sips` | 壓縮素材、驗尺寸 | 系統內建，通常都有 |

⚠️ **動 `use_figma` 前一定要先載入 `/figma-use` skill**（官方硬性要求）。

✅ **Figma MCP 走雲端，Claude 這端不需要使用者開桌機版** —— MCP 走 `https://mcp.figma.com/mcp`，
程式在 Figma 伺服器上跑。（這也正是「讀不到本機字型」的成因，見〈硬限制一〉。）
✅ **模版只用雲端有的字重**（`Black`／`Bold`，使用者拍板），填字不代換、畫面即最終樣子。
⇒ **G2 開桌機版純粹是讓使用者看稿與調顏色**，不是為了修字重。
（唯一例外＝模版日後又被加入本機專屬字重時，才會重新出現落差，見〈硬限制一〉。）

## 四個停點（G1／G2 可自動略過，G3／G4 一定要等使用者）

| # | 時機 | 提示語 | 可以不停的條件 |
|---|---|---|---|
| **G1** | 讀完信 | 「本次製作幾月官網？」 | **信件內文有提到月份**，或**使用者觸發時就講了**（例：「做 10 月派歌官網」）→ 直接用，不問 |
| **G2** | 內容全部套進 Figma 後 | （不是提問，是動作）**Claude 直接開啟 Figma 桌機版並跳到當月 page** | **偵測到 Figma 已在執行** → 不重開 |
| **G3** | G2 之後 | 「**下一步**：1. 請手動調整顏色 ＆ 核對內容，確認無誤打 OK 就跑輸出」 | ⛔ **無條件停** |
| **G4** | 輸出完成後 | 「**下一步**：1. 確認圖檔 OK（已存：`<$DELIV_ROOT/{月}月 絕對路徑>`），確認 OK 後移動到 `<$FINAL_ROOT 絕對路徑>` 中」 | ⛔ **無條件停**，OK 後才搬移＋收尾 |

> 📌 **G3 使用者會調的是「顏色」，不是字重**（使用者更正）。
> 主要是 `BG-color`／`BG-shadow` 要配合當月封面配色。
> ✅ **不必再提醒他看字型**（模版已對齊雲端字重，見上方先決條件段與步驟 8）。
> 📌 **G3 他也可能重排速報藝人的視覺順序** —— 那是他的環節，**不影響流程**，事後不用回頭改檔名或流水號。

---

# 步驟

## 0. 建當月工作區

🔴 **月份不能從信件取得**（實測：主旨與內文完全沒有月份字樣）。

**G1 的自動偵測順序（前兩項命中就不要問）**：
1. **使用者觸發語裡有月份** —— 例「做 10 月派歌官網」、「做派歌官網圖 10月」→ 直接用
2. **信件內文有月份字樣** —— 正則 `([0-9]{1,2}|[一二三四五六七八九十]+)\s*月` 掃過內文
3. 兩者都沒有 → ⛔ **停下問「本次製作幾月官網？」**，附上來信日期當參考

🚫 **不可用「來信日期次月」之類的規則自己推定**（9 月那次信裡完全沒寫月份，是 設計端 自己定的）。

拿到月份才建資料夾，`$M` 格式固定 `10月`。

```bash
# ⬇️ 這三個路徑請換成你自己的（公開版已去識別化）
PROJ="${PROJ_ROOT:?請設定專案資料夾絕對路徑}"
DELIV="${DELIV_ROOT:?請設定輸出暫存夾絕對路徑}"      # G4 前成品先放這
FINAL="${FINAL_ROOT:?請設定最終交付夾絕對路徑}"      # G4 OK 後搬到這
M="10月"   # ← 換成當月

mkdir -p "$PROJ/$M"                                        # ← 文案定稿.md 的家，別漏
mkdir -p "$PROJ/素材暫存/$M"/{速報,推薦,案例,_上傳用}
mkdir -p "$DELIV/$M"/{1-發行速報,2-發行推薦,3-發行案例}
```

- **素材暫存**（過程檔）：`$PROJ/素材暫存/{月}月/`
- **輸出暫存**（成品 JPG，G4 前）：`$DELIV/{月}月/{1-發行速報,2-發行推薦,3-發行案例}/`
- **最終交付**（G4 OK 後搬過去）：`$FINAL/{月}月/`
- **文案定稿**：`$PROJ/{月}月/文案定稿.md`

## 1. 讀需求信　⛔ G1

```
search_threads(query='subject:派歌 官網')   →  取 threadId
get_thread(threadId, messageFormat='PLAIN_TEXT')
```

🔴 **必須讀整串** —— 死線藏在回信裡，不在原始需求信。
🚫 **不要用 `FULL_CONTENT`** —— 每封約 2.7MB（HTML body ＋ 附件），三封灌爆 context。`PLAIN_TEXT` 就夠。
💰 **這步建議外包 subagent**（`general-purpose`，`model: haiku`），主對話只收結論（連結、張數、文案、死線）。

### 🔴 三個實測踩過的坑

**① 主旨每個月都一樣、而且不含月份**
實際主旨固定是 `【<公司>】<產品>官網 每月設計需求討論`，新月份很可能**回在同一條 thread** 裡（`Re:` 一路接下去）。
⇒ **靠日期區分，不是靠主旨。** 找最新一批未處理的訊息，然後在 G1 跟使用者確認月份。

**② thread 第一封不是需求方本人**
第一封是**轉寄信**（同事A 轉寄），需求方窗口的原文包在轉寄內文裡。
⇒ `需求負責人` **不可用 thread 第一封的 sender**，要從轉寄內文的 `寄件者：` 抓原始寄件人。

**⛔ 讀完信走 G1**（先自動偵測，測不到才問）。沒有月份不要繼續——資料夾名、Notion 標題、交付路徑全都要用到。

**③ 信裡的 PNG 附件不是素材**
那張截圖是給信末溝通題用的（官網主頁六張 banner）。**素材一律走 Seafile，別下載附件。**

## 2. 整理文案 → 落檔

依專案文件〈分析一〉的推導規則整理，落檔 `$PROJ/{月}月/文案定稿.md`，格式照 `9月/文案定稿.md`。

**三件事一定要做**：
1. **簡體錯字直接改**（CN 版出現繁體字＝需求方筆誤），改了要在對話裡說一聲
2. **溝通題不寫進卡**，但要在對話裡口頭提醒使用者一句
3. **數張數**：抓**所有**「共 N 張」再加總。⚠️ **張數每月不同，一律回信件重數，不可沿用上月**
   - 🕳️ **N 是中文數字**（「共兩張」「共四張」），不是阿拉伯數字，要先轉換
   - 🕳️ **速報有兩個「共兩張」**（橫式 750×390 ＋ 方形 1000×1000 各一組），只抓第一個會少算 2 張

> 🚫 **`9月/文案定稿.md` 裡「案例只有 Title／Desc 兩層」那句是錯的**，案例是三層（Title／Song／Desc）。

## 3. 在 Notion 開新頁面（只填內文，不碰欄位）

> 🔴 **拍板：Claude 只在資料庫裡開一張新頁面、把統整好的文字寫進內文。
> 所有欄位（訂單狀態／開卡單位／急迫性／死線／配送員／需求負責人／吸吸誰）一律由使用者手動填。**
> 🚫 **不要設定任何 property**，`properties` 只帶 `Name`。

```
notion-create-pages(
  parent={type:'data_source_id',
          data_source_id:'<NOTION_DATA_SOURCE_ID>'},
  pages=[{properties:{Name:'Packer_{月}月官網'}, content:'<統整好的內文>'}]
)
```

**內文結構＝照 9 月卡抄。先現查一次拿到真正的 block 結構，不要憑印象寫**：

```
notion-fetch(id='<上個月接單卡網址>')
```

骨架：
1. 最上面一個 callout：`完稿檔：` ← **留空**，使用者最後自己補 Seafile 連結
2. `### 設計進度` 勾選清單三行（格式 `{分類} {尺寸}，簡體 xN 繁體 xN，共 N 張`）
3. 三個分類各一段：`### 一、發行速報` ＋ 🔗「圖檔連結」超連結 ＋ TW／CN 並列

> 🔴 **案例段的文字切法（使用者拍板，Figma 與 Notion 一律對齊）**：
> 每層各一行，**不要把整段黏成一行**——
> 第 1 行拆成 `Title`（藝人＋全新單曲，**不含歌名**）與 `Song`（`〈歌名〉`），
> 信裡第 2、3 行合併成 `Desc`。
> 例：`阿明樂團全新單曲` ／ `〈歌名〉` ／ `第一句文案，` ＋ `第二句文案。`
> 完整對照表在專案文件〈分析二〉。

**交付後在對話裡提醒使用者**：欄位要自己填，特別是 `訂單狀態` 要設成 `已接單`。

## 4. 下載素材

> ⚡ **這步＋步驟 5 的機械段已腳本化**：
> `python3 $PROJ/_scripts/parse_mail.py <信件純文字檔>` 抽 token →
> `$PROJ/_scripts/seafile_fetch.sh <月份> <速報> <推薦> <案例>` 一次做完下載＋壓縮＋流水號＋`清單.md`。
> 🔴 跑完**一定要核對 `清單.md` 的 tw／cn 分界**（腳本只是照檔名切一半）。下面保留手動指令當備援。

**怎麼找連結**（實測格式）：`PLAIN_TEXT` 下超連結會變成**錨文字 ＋ 尖括號網址**，不是 markdown：

```
*一、發行速報 **圖檔連結 <https://<SEAFILE_HOST>/d/<token>/>*
```

抓法：
```
圖檔連結\s*<(https://<SEAFILE_HOST>/d/[a-f0-9]{20}/)>
```

**分類判斷**：三個連結都跟分類標題**在同一行**（`一、發行速報`／`二、發行推薦`／`三、發行案例`），逐行配對即可。
⚠️ 但別把「同一行」寫死當唯一假設——那是 Gmail HTML→純文字轉換的結果，換客戶端可能斷行。
**fallback**：以分類標題當分段界線，取該段內第一個 seafile URL。
🚫 不要用上個月的網址。

```bash
TOKEN="<分享連結 /d/ 與結尾 / 之間那 20 碼>"
# 撈檔案清單
curl -s "https://<SEAFILE_HOST>/d/$TOKEN/" \
 | grep -oE 'files/\?p=/[^"]+&dl=1' | sed 's|^files/?p=/||; s|&dl=1$||' | sort -u
# 下載單檔（enc = URL-encoded 檔名）
curl -sL -o "<檔名>" "https://<SEAFILE_HOST>/d/$TOKEN/files/?p=/<enc>&dl=1"
```

存到 `$PROJ/素材暫存/{月}月/{速報,推薦,案例}/`，並產生 `清單.md`。

**備援**：若要求登入 → **不要自己找帳密**，請使用者用瀏覽器下載。連結過期就回信要新的。

## 5. 壓縮並重新命名

原圖常 4000×4000／8MB，`upload_assets` 上限 10MB。

```bash
sips -s format jpeg -s formatOptions 88 -Z 1600 "<原檔>" \
     --out "$PROJ/素材暫存/$M/_上傳用/<新檔名>.jpg"
```

🔴 **速報藝人順序＝完全照信件，不要自己重排**（使用者拍板）
信裡第 1~4 位＝TW（`01_tw1`~`04_tw4`），第 5~8 位＝CN（`05_cn5`~`08_cn8`）。
🚫 **不要按檔名排序**（Seafile 抓下來的檔名順序與信件無關，實測錯位過就是這樣來的）。
🚫 **不要為視覺效果做判斷** —— 使用者會在 **G3 那一關自己重排**成順眼的順序，**不影響你的流程**（他改 Figma，不改檔名）。

🕳️ **一定要用流水號命名**：`upload_assets` **靠陣列順序**對應 `nodeIds`，用中文藝人名極易錯位。
命名規則見專案文件步驟 4 的表（`01_tw1`…→ `05_cn5`…→ `09_rec1`…→ `11_case`）。
**順序固定「速報 → 推薦 → 案例」**，流水號依當月實際張數順延。

## 6. 建當月 Figma page（全自動，不需要使用者動手）

🔴 **模版 `Templete` 是每月重複使用的資產，絕不直接在上面做當月內容。**

### 三步（漏第三步＝換圖沒反應）

**① 建新 page ＋ clone section**
```js
const work = await figma.getNodeByIdAsync('18:386');   // 模版 Templete page（🚫 用 id 定位，別用名稱——另有 Templete(2025)）
await work.loadAsync();
const page = figma.createPage();
page.name = '10月';                                    // ← 當月
for (const sec of work.children) {
  const c = sec.clone();
  page.appendChild(c);
  c.x = sec.x; c.y = sec.y;
}
```
> ⚠️ **會 clone 出 8 個 section**（成品 4 ＋ 元件庫 4 成對），不是 4 個。

**②③ 建對照表 ＋ 重新指向**（已實測可用，直接抄）
```js
const map = {};
function walk(a, b) {                       // 依「結構位置」配對
  if (a.type === 'COMPONENT') map[a.id] = b.id;
  const ac = a.children, bc = b.children;
  if (!ac || !bc || ac.length !== bc.length) return;
  for (let i = 0; i < ac.length; i++) walk(ac[i], bc[i]);
}
const src = work.children, dst = page.children;
for (let i = 0; i < src.length; i++) walk(src[i], dst[i]);

for (const inst of page.findAllWithCriteria({ types: ['INSTANCE'] })) {
  const mc = await inst.getMainComponentAsync();
  if (!mc) continue;
  const newId = map[mc.id];
  if (!newId) continue;                     // 沒對照＝共用元件，維持不動
  inst.swapComponent(await figma.getNodeByIdAsync(newId));
}
```
🚫 **不可靠名稱比對** —— 名稱會重複（`Cover` 在 `Work-2` 和 `Work-2-2` 都有），靠名稱一定對錯。
✅ **共用元件自動被跳過**：它們不在 `map` 裡，所以 `if (!newId) continue` 就自然維持共用，不必特判。

> 🕳️ **為什麼需要第三步**：逐一 clone section 時，副本 instance 的 `mainComponent` **仍指向母版**的元件。
> 不重新指向的話，你在副本換圖畫面不會變（它連的是母版），反而得去改母版才有效——完全違背做副本的目的。
> instance 分三種：指向母版（要換）、指向 `Component`（維持）、`mainComponent.parent === null`（遠端／已刪元件，也維持）。
> 🚫 **各類數量每次都不一樣**，**絕不可寫死當驗收條件**，一律現場數。

**例外＝共用元件不重新指向**：指向 `Component` page 的 `Label-*`／`Button-了解詳情`，
這些是跨月共用的 UI，**維持共用是正確的**。

### 🔴 clone 完先驗「Artist 接線」

模版的四個速報 frame 各自掛著 `Type=1`~`Type=8`，**每個編號應該剛好用一次**。
曾實測抓到 `速報CN_1000x1000` 右下格重複掛 `Type=6`、`Type=8` 完全沒被用到
→ 結果是**同一位藝人出現兩次、另一位整個消失**。
🕳️ **這種錯眼睛極難抓**（重複的是圖片不是文字，而且四張圖看起來都「有內容」），一定要用程式數。

```js
// 對每個速報 frame，列出各格掛的 Type 並找重複
const fb = f.absoluteBoundingBox;
const used = [];
for (const n of f.findAllWithCriteria({ types: ['INSTANCE'] })) {
  const mc = await n.getMainComponentAsync();
  if (mc && /^Type=\d/.test(mc.name)) used.push(mc.name);
}
const dup = used.filter((v, i) => used.indexOf(v) !== i);   // 非空＝有重複，停下回報
```

**正確接線**（使用者修正後實查）：
| frame | 各格 |
|---|---|
| `速報TW_1000x1000` | `1 / 2 / 3 / 4` |
| `速報CN_1000x1000` | `5 / 6 / 7 / 8` |
| `速報TW_750x390`／`速報CN_750x390` | 上排 `1,2,5,6`、下排 `3,4,7,8` |

⚠️ **抓到重複不要自己改模版** —— 模版是每月共用資產，回報使用者由他決定。

### 做完一定要重取 id

```
get_metadata(fileKey, nodeId=<新 page id>)
```
🔴 **絕不沿用專案文件〈分析二〉的 id** —— 那些只屬於模版 `Templete`，clone 出來的全部是新的。

## 7. 置入圖片

```
upload_assets(fileKey, count=N, nodeIds=[...], scaleMode='FILL')
→ 回傳 N 個 submitUrl → 逐一 POST
```
```bash
curl -F "file=@<檔>.jpg;type=image/jpeg" "<submitUrl>"
```

🕳️ **目標節點就是元件本身**（`Type=1`~`Type=8`、`Cover`、`img`），fill 掛在 COMPONENT 節點上，不是子層。
🚫 `createImageAsync` 在 `use_figma` 裡不支援，只能走 `upload_assets`。

## 8. 填文字

用「硬限制一」的替代字型邏輯。**一次 6 個節點以內**，分批呼叫。
`use_figma` 一次最多 10 個邏輯操作；腳本是**原子性**的（失敗完全不執行，不會留半套）。

> ✅ **模版本身已對齊雲端字重**（使用者拍板）：`Templete` 的 14 處
> `Noto Sans TC ExtraBold` → `Black`、`SemiBold` → `Bold` 已直接改在模版上，
> **所以正常情況不會再發生代換，畫面即最終樣子，G3 不必再檢查字重。**
> 下面的替代邏輯**保留當保險**（模版日後又被加入本機字重時仍能跑完，不會整段失敗）。
>
> 🕳️ **改字重時不要 `loadFontAsync` 舊字型** —— 雲端載不到 `ExtraBold`／`SemiBold`，
> 那行會直接丟錯讓整段不執行。**只載入新字重再 `setRangeFontName` 就好**（實測）。
>
> 📌 另有 14 處 `PingFang SC Semibold`，全在共用元件 `Label-*`／`Button-了解詳情` 內，
> 內容每月不變、不會被改字 → **刻意不動**（改了會波及 `Component` page 的其他用途）。

### 替代字型（實測版，直接抄）

```js
const SUB = (f) => {
  let fam = f.family, style = f.style;
  if (/^PingFang HK$/i.test(fam)) fam = 'Noto Sans HK';
  else if (/^PingFang (TC|SC)$/i.test(fam)) fam = 'Noto Sans TC';
  else if (/^Noto Sans CJK (TC|SC|HK)$/i.test(fam)) fam = 'Noto Sans TC';
  if (/extra\s*bold|heavy/i.test(style)) style = 'Black';        // ← 雲端沒有 ExtraBold
  else if (/semi\s*bold|demi\s*bold/i.test(style)) style = 'Bold';
  return { family: fam, style };
};
async function put(id, text) {
  const n = await figma.getNodeByIdAsync(id);
  for (const s of n.getStyledTextSegments(['fontName'])) {
    try { await figma.loadFontAsync(s.fontName); }
    catch (e) {
      const sub = SUB(s.fontName);
      await figma.loadFontAsync(sub);
      n.setRangeFontName(s.start, s.end, sub);
    }
  }
  n.characters = text;
}
```
> **雲端 `Noto Sans TC` 只有**：Black／Bold／DemiLight／Light／Medium／Regular／Thin。
> 沒有 ExtraBold、沒有 SemiBold —— 漏掉對應會直接丟 `could not be loaded` 而整段不執行。

### 🕳️ 三個實測踩過的坑

**① 模版裡留著上個月的文案**
模版是從上個月成品做出來的，`Title`／`Song`／`Desc` 都還有舊內容（例：案例的 Title 停在上個月的藝人名）。
⇒ **每個文字節點都要明確覆寫**，不可「看起來對就跳過」。殘留的畫面完全正常，只有逐字比對抓得到。

**② `Title` 是 instance，文字在它內部**
節點 id 長這樣：`I55:399;18:2661`（開頭 `I`、中間 `;`）。
🚫 **不可用「id 含 `;`」當排除條件**（會連 `Title` 一起濾掉）。
✅ 改用**名稱白名單**：`['Title','Song','Desc','Text-TW','Text-CN']`。

**③ 案例 CN 的 `Desc` 用兩個空格分隔、不是換行**
直接塞字會被拉成一長條（實測 398px）。填完要固定寬度才會斷成兩行：
```js
n.textAutoResize = 'HEIGHT';
n.resize(210, n.height);   // 目標 210×48，與模版一致
```

## 8b. Title 的 H1／H2 自動切換

`Title` 元件有變體屬性 **`Property 1`**，值 `H1`／`H2`。**預設一律 H1**。

**規則**：填完字後量寬度，**Title 內部文字寬度 > 它所在 `Context` 的寬度（220）就切 H2**。

🔴 **量的是「內部文字」，不是 instance 本身**：
instance 寬度是固定的 196，**文字溢出不會把它撐大**，所以 `inst.width > inst.parent.width`
**永遠不會成立**，Title 會靜默被裁掉（實測案例只印出「阿明樂團全新單」，尾字被吃掉）。

```js
const inst = await figma.getNodeByIdAsync(TITLE_INSTANCE_ID);
const txt  = await figma.getNodeByIdAsync(TITLE_TEXT_ID);       // I<inst>;18:2661
if (txt.width > inst.parent.width) inst.setProperties({ 'Property 1': 'H2' });
```
> ✅ 切換後**文字會保留**，不用重填（實測：`阿明樂團全新單曲` 257px → 切 H2 後 210px）。
> ⚠️ 切換後**內部文字的 id 會變**（`;18:2661` → `;18:2662`），要重抓才能再改字。

## 9. frame 命名

把當月音樂人名填進括號，CN 版用簡體。速報無括號（非單一藝人）。
🚫 **案例 frame 名稱前後的多餘空格是模版原樣，只換括號內容、別順手清掉。**

## 10. 開 Figma 給使用者看　⛔ G2

內容全部套進去後，**Claude 主動幫使用者開啟 Figma 桌機版並跳到當月 page**。

**先偵測 Figma 有沒有在跑**（在跑就不要重開，避免打斷他）：
```bash
if lsof -nP -iTCP:3845 -sTCP:LISTEN 2>/dev/null | grep -q Figma; then
  echo "Figma 已開啟，不重開"
else
  open "figma://file/<FIGMA_FILE_KEY>?node-id=<當月 page id>"
fi
```
> `figma://` 是桌機版的深層連結。若打不開，退而求其次用一般網址：
> `open "https://www.figma.com/design/<FIGMA_FILE_KEY>/?node-id=<當月 page id>"`

## 10b. 請使用者調顏色並核對　⛔ G3

**提示語**（照這個講，不要換說法）：

> **下一步**：
> 1. 需求內容都套進 Figma 了。**請手動調整顏色 ＆ 核對內容，確認無誤打 OK 就跑輸出！**

- **主要要調的是顏色** —— `BG-color`／`BG-shadow` 配合當月封面配色
- ✅ **不必提字型**（模版已對齊雲端字重）
- 📌 他可能會重排速報藝人的視覺順序——**那不影響流程**，事後不用回頭改檔名或流水號
- ⛔ **無條件停在這裡**，收到「OK」才跑步驟 11

## 11. 輸出

> ⚡ **已腳本化**：先對每個 frame 取 `get_screenshot` 拿 `image_url`，
> 寫成 TSV（每行 `image_url⇥frame名⇥1|2|3`，第三欄是交付子夾），然後
> `$PROJ/_scripts/export_banners.sh <月份> <對照表.tsv>`
> —— 下載、轉 JPEG(100)、rstrip 檔名、逐檔驗尺寸、不合格自動刪掉並回非零退出碼，一次做完。
> 🚫 `image_url` 短效且視同機密，拿到就立刻跑，不要落檔。下面保留手動指令當備援。

🔴 **`get_screenshot` 只產 PNG，而且只回一個短效網址** —— 不會直接給你 JPEG，也不會存檔。
所以是**三步**：取網址 → `curl` 下載 PNG → `sips` 轉 JPEG。

```
get_screenshot(fileKey, nodeId=<frame id>, maxDimension=<該 frame 長邊>, contentsOnly=true)
→ 回傳 {"image_url": "https://www.figma.com/api/mcp/asset/....png", "width":750, "height":390}
```

`maxDimension` 設成長邊（750×390 設 750、1000×1000 設 1000），輸出像素就精準吻合。
⚠️ 網址**短效、視同機密**，拿到就立刻下載。

```bash
OUT="$DELIV/$M/2-發行推薦"
curl -sL -o /tmp/shot.png "<image_url>"
sips -s format jpeg -s formatOptions 100 /tmp/shot.png --out "$OUT/<檔名>.jpg"
```

> 🕳️ **要跑陣列迴圈（批次下載／批次上傳）一律包 `bash -c '...'`** ——
> 這台機器的預設 shell 是 zsh，`${!arr[@]}` 之類的 bash 語法會直接噴 `bad substitution`。

**檔名＝Figma frame 名 ＋ `.jpg`**，但要先 **`rstrip` 掉結尾空白**。

模版的 frame 名本來就是交付檔名，但**案例的 frame 名結尾有一個空格**（`案例TW(阿明樂團)_ 750x390 `）。
- ✅ **frame 名前後的空格照留**（模版原樣，不要去 Figma 清）
- 🔴 **但檔名不能帶結尾空格** —— 直接接 `.jpg` 會變成 `..._ 750x390 .jpg`，與交付慣例不符
- 正確檔名＝`案例TW(阿明樂團)_ 750x390.jpg`（`_` 後那格保留，`.jpg` 前那格拿掉）

```bash
name=$(echo "$frameName" | sed 's/[[:space:]]*$//')   # 只去結尾空白
```

> 🕳️ **`formatOptions` 要用 100，不是 95**（實測）：
> 品質 95 產出的檔只有交付慣例的 **30–39%** 大小；品質 100 是 105–118%，才對得上。

**分流**：`1.發行速報` → `1-發行速報/`；`2.發行推薦` 與 `2.發行推薦-2` → `2-發行推薦/`；`3.發行案例` → `3-發行案例/`

**輸出後必驗**：
```bash
for f in "$DELIV/$M"/*/*.jpg; do
  echo "$f"; sips -g pixelWidth -g pixelHeight "$f" | tail -2
done
```
逐檔比對檔名裡的尺寸，不吻合就重匯。

## 12. 交付確認　⛔ G4

**提示語**（照這個講）：

> **下一步**：
> 1. 確認圖檔 OK（已存：`$DELIV_ROOT/{月}月/`），確認 OK 後移動到 `$FINAL_ROOT` 中

⛔ **無條件停**，收到「OK」才搬移，再做收尾。

**收到 OK 後搬移**：
```bash
SRC="$DELIV/$M"; DST="$FINAL"
[ -d "$DST" ] || { echo "[!] 目的地 $DST 不存在，停下回報使用者（不自己建）"; exit 1; }
[ -e "$DST/$M" ] && { echo "[!] $DST/$M 已存在，停下問使用者（不覆蓋）"; exit 1; }
mv "$SRC" "$DST/" && find "$DST/$M" -name '*.jpg' | wc -l
```
> 🔴 目的地不存在（交付夾可能被搬走或改名）→ **停下回報**，🚫 不 `mkdir` 建新夾。目的地已有同月資料夾時**不要覆蓋**，停下問使用者。搬完回報新路徑與張數。

## 13. 收尾

1. **張數三方驗算**：信件張數 ＝ Notion 頁面「設計進度」加總 ＝ Figma 成品 frame 數
2. **繁簡逐字比對**，特別檢查藝人名（例：藍鯨樂隊→蓝鲸乐队、飛鳥計畫→飞鸟计画）與書名號內作品名
3. 檢查 CN 版的 `Label` 與 `Button-了解詳情` 有沒有用到 CN variant
4. 派 **`verifier`** 對照原始需求信驗收
5. **提醒使用者兩件手動事項**（不可默默結束；回覆第一行「**下一步**：」＋編號條列）：
   - 把成品上傳 Seafile，連結貼回 Notion 頁面最上面的 callout
   - **Notion 欄位要自己填**，特別是 `訂單狀態` 設成 `已接單`

---

## 停止條件（跑完自查）

- [ ] 交付夾三個子夾都有檔，總張數＝信件張數
- [ ] `sips` 逐檔驗過，像素與檔名吻合
- [ ] Notion 頁面建在「今晚我想來點」，內文完整（欄位由使用者手動填）
- [ ] `文案定稿.md` 已落檔
- [ ] 簡體錯字已改並口頭說明；溝通題已口頭提醒
- [ ] G4 已收到 OK，成品已從 `$DELIV_ROOT/{月}月/` 搬到 `$FINAL_ROOT/{月}月/`，兩項手動提醒已送出

## 卡關規則

**同一步驟卡兩次就停下**，跟使用者簡述問題（使用者問細節才給錯誤原文），不要硬試第三次。
常見卡點與已知解都在專案文件的三個硬限制與「🕳️」標記處，先去那裡找。
