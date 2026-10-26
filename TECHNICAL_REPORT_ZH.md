# 📘 Technical Report: Architecture & Algorithms

本報告記錄 **Universal Spine 2.1.27 Workstation** 在面對《暗黑地牢》封閉舊版引擎限制時，所採用的五項核心演算法與架構設計。

---

## 1. Spine 2.1.27 二進位解析與逆向仿射投射管線 (Inverse Affine Baking Pipeline)

### 1.1 問題背景
在母體 `.skel` 與 `.atlas` 中，每個部件在圖集上是經過旋轉、縮放、甚至鏡像翻轉（`flipX` / `flipY`）後局部擺放的圖片；當遊戲載入時，引擎會透過骨骼階層世界矩陣（Bone World Matrix）將圖片投射到世界座標。
若創作者在畫布（世界空間）上切下一塊角色的手臂，必須將它「逆向還原」回該插槽原本的局部 UV 角度與比例，貼回 `.png` 後才能在遊戲內正確顯示。

### 1.2 數學模型
1. **骨骼世界變換矩陣計算**：
   針對 `.skel` 內的每一根骨骼 $b$，依父子階層遞迴計算其世界座標 $(W_x, W_y)$ 與 $2 \times 2$ 旋轉縮放矩陣：
   $$M_{\text{bone}} = \begin{bmatrix} m_{00} & m_{01} \\ m_{10} & m_{11} \end{bmatrix} = \begin{bmatrix} \cos\theta \cdot s_x & -\sin\theta \cdot s_y \\ \sin\theta \cdot s_x & \cos\theta \cdot s_y \end{bmatrix}$$
   （若 `flipX` 或 `flipY` 為真，則對應列向量取反）。

2. **正向投射矩陣 $M_{\text{fwd}}$（UV 空間 $\to$ 手術台畫布空間）**：
   * **針對 `Region` 附件**：由局部偏移 $(x, y, \text{rot}, s_x, s_y)$ 與原始尺寸 $(\text{origW}, \text{origH})$ 推導出三個角點在畫布上的投影座標 $(P_0, P_1, P_2)$，建立 $3 \times 3$ 齊次仿射矩陣 $M_{\text{fwd}}$。
   * **針對 `Mesh` / `SkinnedMesh` 網格附件**：由於網格頂點受多根骨骼權重（Weights）影響，系統先計算所有頂點在 Setup Pose 下的畫布世界座標 $V_i = (X_i, Y_i)$，再與對應的材質像素座標 $U_i = (u_i \cdot \text{origW}, v_i \cdot \text{origH})$ 建立最小平方法（Least-Squares Linear Regression）超定方程組，求解最佳擬合仿射矩陣：
     $$\min_{A, B} \sum_{i=1}^{N} \| A \cdot U_i + B - V_i \|^2$$

3. **逆向烤製矩陣 $M_{\text{bake}}$（手術台畫布空間 $\to$ 圖集 UV 空間）**：
   求取 $M_{\text{fwd}}$ 的解析逆矩陣 $M_{\text{inv}} = M_{\text{fwd}}^{-1}$，並與使用者在 B 區微調的局部變換矩陣 $M_{\text{user}}(\Delta x, \Delta y, \theta, s)$ 進行矩陣乘法：
   $$M_{\text{bake}} = M_{\text{inv}} \cdot M_{\text{user}}$$
   直接將此矩陣傳入 HTML5 Canvas `uCtx.setTransform(a, b, c, d, e, f)`，即可在 GPU 硬體加速下瞬間完成無失真逆向烤製。

---

## 2. 世代零失真母本空間 (Master-Space Architecture) 與形態學抗破碎

### 2.1 問題診斷：重取樣世代失真 (Generational Loss)
早期版本將分區遮罩 `zoneMap` 與手繪白土 `puttyMap` 僅儲存在當前縮放後的畫布像素陣列中。當使用者拉動 `70% - 150%` 縮放滑桿時，若直接對前一次的陣列進行最近鄰重取樣（Nearest-Neighbor Resampling）並覆寫自身，來回拖動數次後會產生嚴重的階梯鋸齒與孔洞破碎。

### 2.2 解決方案
1. **雙層空間分離**：建立永遠固定在 `Scale = 1.0`（100% 原始尺寸）的無損母本 `masterZoneMap` 與 `masterPuttyMap`。
2. **雙向座標映射**：當使用者在任意縮放倍率 $S$ 下塗抹畫筆（半徑 $r$）時，除了更新當前視圖陣列外，同步透過逆向縮放映射寫入 1:1 母本（半徑 $r / S$）。
3. **3×3 形態學閉運算平滑（Morphological Closing Filter）**：每當縮放倍率 $S \neq 1.0$ 時，系統一律從 1:1 母本重新投射，並自動執行一次 $3 \times 3$ 鄰域形態學濾波（若空白像素周圍有 $\ge 5$ 個同區白土像素則自動填補裂縫；若孤立像素周圍 $\le 1$ 個鄰居則剔除噪點），徹底根除縮放造成的邊緣破碎。

---

## 3. 60FPS 零卡頓互動渲染管線 (Analytical Bounding Box vs. Lazy Pixel Scan)

### 3.1 效能瓶頸分析
在 HTML5 Canvas 中，呼叫 `getImageData(0, 0, W, H)` 會強制觸發 GPU 到 CPU 的記憶體同步讀取（Readback）。若在滑鼠拖曳貼圖（`onmousemove` 每秒觸發 60~120 次）時，對多個部件執行全畫布像素掃描來計算緊密裁切邊界（Tight-Trim），每秒掃描超過上億個像素會導致主執行緒嚴重阻塞（單幀耗時 $> 80\text{ms}$）。

### 3.2 雙階段渲染優化（效能提升 >100 倍）
1. **高速互動階段 (`fastInteractive = true`)**：
   在拖曳滑鼠或滑動滑桿的過程中，搭配 `requestAnimationFrame` 節流鎖，**完全跳過 `getImageData` 像素掃描**，直接利用已知的部件矩陣與頂點座標進行純數學包圍盒計算（耗時 $< 0.5\text{ms}$），實現絲滑的 60FPS 即時預覽。
2. **精確收斂階段 (`fastInteractive = false`)**：
   僅在放開滑鼠（`onmouseup` / `onchange`）或點擊匯出按鈕時，才執行一次精確像素掃描與正數 `offset` 緊密裁切（Tight-Trim），兼顧極致流暢度與最終圖集空間利用率。

---

## 4. 引擎防閃退安全島機制 (1×1 Transparent Safe Island)

### 4.1 閃退與雜訊成因
《暗黑地牢》的 `.atlas` 預設啟用 `filter: Linear,Linear`（雙線性插值過濾）。當母體中某些未使用的插槽被縮小為 `1×1` 透明像素並指向 `(0, 0)` 時：
* 若 `(0, 0)` 緊鄰其他有效部件的邊緣像素，GPU 在進行 $2 \times 2$ 雙線性取樣時會吸取到相鄰部件的顏色，在角色身上拉出詭異的彩色雜訊線條（Texture Bleeding）。
* 若完全移除 `.atlas` 內的插槽項目，`.skel` 動畫在讀取該插槽時會因找不到索引而直接引發遊戲閃退（CTD）。

### 4.2 工程解法
系統在產生 C 區最終圖集時，強制於左上角 `(0, 0) ~ (7, 7)` 劃設一塊 **`8×8` 的絕對透明安全島（Safe Island）**，在匯出前以 `putImageData` 寫入純透明像素（`RGBA = 0,0,0,0`），並將所有有效部件強制從 `X = 12` 之後開始排版。所有未綁定且需隱藏的插槽統一指向安全島正中央的 **`xy: 2, 2`（`size: 1, 1`）**，四周被至少 2px 的純透明護城河包圍，100% 杜絕雙線性溢色與缺件閃退。
