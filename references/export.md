# 步驟 11～13 細節：輸出、交付確認、收尾

> 📁 本檔的 `<SKILL_DIR>` 要換成實際路徑（＝本 skill 資料夾）再用；本檔不會自動替換。
> 從 SKILL.md 搬出的細節。規格／模版結構／node id 的細節正本仍在 `$PROJ_ROOT/CLAUDE.md`；本檔是「可直接執行的指令與程式」，兩邊對不上時以本檔（skill）為準並回頭修專案文件。

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
4. **提醒使用者兩件手動事項**（不可默默結束；回覆第一行「**下一步**：」＋編號條列）：
   - 把成品上傳 Seafile，連結貼回 Notion 頁面最上面的 callout
   - **Notion 欄位要自己填**，特別是 `訂單狀態` 設成 `已接單`

