# 步驟 6～10b 細節：建當月 Figma page、置圖、填字、H1/H2、命名、開 Figma、G3

> 📁 本檔的 `<SKILL_DIR>` 要換成實際路徑（＝本 skill 資料夾）再用；本檔不會自動替換。
> 從 SKILL.md 搬出的細節。規格／模版結構／node id 的細節正本仍在 `$PROJ_ROOT/CLAUDE.md`；本檔是「可直接執行的指令與程式」，兩邊對不上時以本檔（skill）為準並回頭修專案文件。

## 背景：雲端 MCP 與字重

✅ **Claude 這端不需要使用者開桌面版 Figma** —— MCP 走雲端 `https://mcp.figma.com/mcp`，
程式在 Figma 伺服器上跑（所以讀不到本機字型，見專案文件〈硬限制一〉）。
✅ **模版只用雲端有的字重**（`Black`／`Bold`），填字不代換、畫面即最終樣子。
⇒ **G2 開桌面版純粹是讓使用者看稿與調顏色**，不是為了修字重。
模版若被加入本機專屬字重，才會重新出現落差（見〈硬限制一〉）。

## 背景：G3 使用者在調什麼

> 📌 **G3 使用者調的是「顏色」，不是字重**：主要是 `BG-color`／`BG-shadow` 配合當月封面配色。
> ✅ **不必提醒他看字型**（模版已對齊雲端字重，見上方與步驟 8）。
> 📌 **G3 他也可能重排速報藝人的視覺順序** —— 那是他的環節，**不影響流程**，事後不用回頭改檔名或流水號。

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

### 🔴 案例（或推薦）多於模版份數時：在當月 page 多複製一組

模版 `3.發行案例`＋`Work-3` 只有**一組**。某月案例來了**兩位**音樂人 → 在**當月 page** 再 clone 一對：
```js
const caseSec = await figma.getNodeByIdAsync('<模版 3.發行案例 id>'), w3 = await figma.getNodeByIdAsync('<模版 Work-3 id>');
const case2 = caseSec.clone(); page.appendChild(case2); case2.x = caseSec.x + 2101; case2.y = caseSec.y; case2.name = '3.發行案例-2';
const w32 = w3.clone();       page.appendChild(w32);  w32.x = w3.x + 2101;     w32.y = w3.y;     w32.name = 'Work-3-2';
const map2 = {}; walk(w3, w32, map2);          // ← 獨立的對照表
// case2 底下的 instance 用 map2 swapComponent（不是 map1，否則會連到第一組的 img）
```
- 位移 `+2101` 對齊模版 `2.發行推薦-2` 與第一份的間距
- 驗：`case2` 裡的 `img`／`Liner` instance 的 mainComponent 父層都要是 `Work-3-2`
- 🚫 **不改模版本身**（模版是共用資產）；張數一律以信件為準

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
> ✅ 切換後**文字會保留**，不用重填。
> ⚠️ 切換後**內部文字的 id 會變**（`;18:2661` → `;18:2662`），要重抓才能再改字。

### 🕳️ 實跑踩到的兩個坑

**① 長字串填完在同一支腳本裡量寬度，會丟 `The node with id "I..;18:2661" does not exist`**
腳本原子性，整段不生效。
✅ **拆兩次呼叫**：第一次只填字＋命名，第二次再用 `inst.children[0]` 量寬、切 H2。

**② 切 H2 之後還是放不下**（H2 文字寬 > 框 210px，最後一字被裁）
- 🚫 不要在 Title 裡塞 `\n` 斷兩行 —— Title instance 是固定高度，第二行會壓到 Song
- ✅ 保持單行 H2，**G3 時明確點名哪幾張被裁**，由使用者在 Figma 調
- 可給使用者的選項：A 調小字級／B 把「最新專輯」等前綴移到 Song 層

## 9. frame 命名

把當月音樂人名填進括號，CN 版用簡體。速報無括號（非單一藝人）。
🚫 **案例 frame 名稱前後的多餘空格是模版原樣，只換括號內容、別順手清掉。**

## 10. 開 Figma 給使用者看　⛔ G2

內容全部套進去後，**Claude 主動幫使用者開啟 Figma 桌面版並跳到當月 page**。

**先偵測 Figma 有沒有在跑**（在跑就不要重開，避免打斷他）：
```bash
if lsof -nP -iTCP:3845 -sTCP:LISTEN 2>/dev/null | grep -q Figma; then
  echo "Figma 已開啟，不重開"
else
  open "figma://file/<FIGMA_FILE_KEY>?node-id=<當月 page id>"
fi
```
> `figma://` 是桌面版的深層連結。若打不開，退而求其次用一般網址：
> `open "https://www.figma.com/design/<FIGMA_FILE_KEY>/?node-id=<當月 page id>"`

## 10b. 請使用者調顏色並核對　⛔ G3

**提示語**（照這個講，不要換說法）：

> **下一步**：
> 1. 需求內容都套進 Figma 了。**請手動調整顏色 ＆ 核對內容，確認無誤打 OK 就跑輸出！**

- **主要要調的是顏色** —— `BG-color`／`BG-shadow` 配合當月封面配色
- ✅ **不必提字型**（模版已對齊雲端字重）
- 📌 他可能會重排速報藝人的視覺順序——**那不影響流程**，事後不用回頭改檔名或流水號
- ⛔ **無條件停在這裡**，收到「OK」才跑步驟 11

