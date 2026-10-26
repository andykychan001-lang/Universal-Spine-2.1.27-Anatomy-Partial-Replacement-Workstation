# 🧬 Universal Spine 2.1.27 Anatomy & Partial Replacement Workstation
> **A Zero-Dependency, Non-Destructive Web Workstation for Darkest Dungeon Modding**  
> 專為《暗黑地牢》(Darkest Dungeon) 打造的全類型 `.skel` / `.atlas` 逆向分件、關節補土與局部替換工作台

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Spine Runtime](https://img.shields.io/badge/Spine_Spec-2.1.27_Compliant-00c7be.svg)]()
[![Zero Dependency](https://img.shields.io/badge/Build-Single_HTML_File-28a745.svg)]()

---

## 🎯 為什麼做這個工具？ (Motivation)

在《暗黑地牢》的 Mod 製作過程中，將一張完整的角色立繪（或 AI 生成／Photoshop 繪製的單張角色圖）轉化為符合遊戲引擎 **Spine 2.1.27** 規範的動態拆件圖集（如 `walk`、`combat`、`idle`、`attack_sk*`），往往面臨極高的門檻：
1. **封閉與過時的環境**：官方綁定極難取得的舊版 Spine 2.1.27 二進位 `.skel` 格式，手動修改 `.atlas` 座標極易引發貼圖錯位、比例崩壞或遊戲閃退。
2. **關節轉動穿幫**：直接從靜態立繪切下四肢與軀幹後，角色邁步或揮砍時關節切口會露出透明破洞。
3. **局部改圖困難**：只想替換某個技能檔（`attack_sk*`）裡的「頭部」或「軀幹」，卻不想動到原作者精心製作的刀光特效與武器圖層。

本工具以 **「單一 HTML 檔案、零外部依賴、非破壞性局部處理（Non-Destructive Pipeline）」** 為核心理念，讓創作者直接在瀏覽器中載入任何現有的母體三件套（`.png` + `.atlas` + `.skel`），透過直覺的畫筆分區與關節補土，一鍵逆向烤製出 100% 符合母體骨骼矩陣的無損圖集。

---

## ✨ 核心功能亮點 (Key Features)

* **🛡️ 最小干預與 1:1 原生無損合規 (Non-Destructive & 1:1 Native Fidelity)**
  * **預設 0 濾鏡直通**：絕不擅自抹除或侵蝕《暗黑地牢》標誌性的純黑粗描邊（Pitch-Black Ink Outlines）。
  * **嚴格遵守 Spine 2.1.27 規範**：1:1 保持母體 `orig` 尺寸與正數 `offset` 緊密裁切，不修改 `.skel` 二進位結構，徹底杜絕 Mesh 變形錯位。
* **🔒 局部替換直通模式 (Unmapped Slot Pass-Through)**
  * 支援全類型檔案（`walk`、`combat`、`camp`、`attack_sk1~12`、`fx_*`）。未被綁定修改的母體插槽（如武器、刀光、地影 `shadow`）可選擇 **「原封不動保留母體原圖」** 或 **「轉為 1×1 透明防閃退錨點」**。
* **🧱 關節白土補土與切口自動採樣上色 (Joint Putty & Auto-Seam Coloring)**
  * 內建底層白土畫筆（Putty Brush），可直接在關節轉軸處塗抹延伸補土，並一鍵吸取切口內側純淨色自動上色，完美解決邁步與出招時的關節裂縫。
  * **無損縮放母本**：白土與分區儲存在 1:1 基準母本空間並搭配 3×3 形態學平滑，來回縮放角色（70%–150%）一千次也絕不發生邊緣破碎。
* **⚡ 60FPS 零卡頓即時逆向預覽 (Zero-Lag Interactive Baking)**
  * 在右側預覽區直接按住左鍵拖曳部件貼圖或拉動微調滑桿時，採用純數學頂點包圍盒（Analytical Bounding Box）即時投射，單幀耗時 `< 0.5ms`，並即時套用母體 `.skel` 真實網格與骨骼預覽。
* **📐 支援特大貼圖樣本 (HD / Oversized Canvas Support)**
  * 可自由切換標準（`900×750`）、大型（`1400×1100`）與特大（`2000×1600`）1:1 手術台畫布，並支援展開 Setup Pose 預設隱藏的技能幀附件。
* **📦 自訂檔名與三件套一鍵打包 (.png + .atlas + .skel)**
  * 自訂匯出檔名時自動同步 `.atlas` 首行圖片索引（防止改名後出現白塊），並可連同母體 `.skel` 一鍵打包下載。

---

## 🚀 快速上手流程 (Quickstart Workflow)

1. **開啟工作台**：直接用 Chrome / Edge / Firefox 雙擊打開 `.html` 檔案（無需架設伺服器）。
2. **載入母體三件套 (Section A)**：上傳作為骨架基準的母體 `.png`、`.atlas` 與 `.skel`。
3. **載入目標角色圖**：上傳你繪製好的角色圖片，使用 **分區筆刷 (Paint Zone)** 微調軀幹與四肢分界。
4. **修補關節與對位 (Section A & B)**：
   * 切換 **🧱 白土補土畫筆** 在大腿根部或肩膀切口處塗抹補土，點擊 **🎨 對白土自動上色**。
   * 在右側 B 區直接拖曳貼圖位置或點擊 **▶ 播放拆件動作測試** 檢查關節咬合。
5. **一鍵匯出 (Export)**：輸入自訂主檔名（如 `my_hero.sprite.walk`），點擊 **📦 一鍵下載三件套** 即可直接放入 Mod 資料夾使用！
