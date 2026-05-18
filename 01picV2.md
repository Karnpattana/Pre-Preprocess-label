# 01picV2.ipynb — บันทึกสำคัญ

## โครงสร้าง Notebook

| Cell | หัวข้อ | หน้าที่ |
|------|--------|---------|
| Constants | CONSTANTS | tuning parameters ทั้งหมดอยู่ที่นี่ที่เดียว |
| Cell 2 | Load + Segmentation | เปิดไฟล์ด้วย file dialog, สร้าง plant_mask |
| Cell 2.5 | Illumination Normalization | Gray-world WB + CLAHE, ทับ img_bgr |
| Cell 3 | Color Features | สถิติ R,G,B,H,S,V,L,a,b บน plant pixels |
| Cell 4 | Vegetation Index | ExG, ExGR, GLI, VARI, WI |
| Cell 5 | Pixel Ratio | % Brown / Yellow / Green |
| Cell 6 | GLCM Texture | Energy, Contrast, Homogeneity, Correlation, Entropy, ASM, Dissimilarity + Skewness, Kurtosis |
| Cell 7 | Histogram Bins | การกระจายความสว่าง 8 กลุ่ม |
| Cell 8 | Edge Density | Canny + Laplacian variance |
| Cell 9 | Hu Moments | 7 ค่า shape descriptor (log-transform) |
| Cell 10 | LBP | Local Binary Pattern histogram |
| Cell 11 | Gabor Filter | 3 freq x 4 angle = 12 combinations |
| Cell 12 | FFT | High/Low frequency energy ratio |
| Cell 13 | Shape Features | Area, Perimeter, Circularity, Solidity, Extent, Aspect Ratio, Eccentricity, Compactness |
| Cell 14 | Edge Browning + Dark Spots | % ขอบใบน้ำตาล + นับจุดดำ/เน่า |
| Cell 15 | Specular Highlight | ความมันวาวของใบ (V channel) |
| Cell 17 | NGRDI | Normalized Green-Red Difference Index |
| Cell 18 | Convexity Defects | รอยหยักขอบใบ + Roughness Index |
| Cell 19 | Wavelet | Haar wavelet 3 levels (cH, cV, cD) |

---

## Bugs ที่เจอและแก้แล้ว

### 1. LBP loop variable ทับ HSV channel (Cell 10)
```python
# ❌ ก่อนแก้
for i, v in enumerate(lbp_hist_norm):   # v ทับ HSV V channel!

# ✅ หลังแก้
for i, val in enumerate(lbp_hist_norm):
```
ผลกระทบ: Cell 15 crash ด้วย IndexError: invalid index to scalar variable เพราะ v กลายเป็น float scalar

### 2. Dark Spots ใช้ global v ที่อาจถูก overwrite (Cell 14)
```python
# ❌ ก่อนแก้
v_plant_mean = np.mean(v[px])

# ✅ หลังแก้
_, _, v_ch = cv2.split(cv2.cvtColor(img_bgr, cv2.COLOR_BGR2HSV))
v_plant_mean = np.mean(v_ch[px].astype(float))
```

### 3. Specular Highlight ใช้ global v (Cell 15)
แก้เหมือน Cell 14 — re-fetch v_ch ทุกครั้งแทนการพึ่ง global

### 4. tkinter dialog ไม่ขึ้น (Cell 2)
```python
# ✅ ต้องเพิ่ม -topmost เพื่อบังคับให้ dialog ขึ้นมาหน้าสุด
root.attributes('-topmost', True)
```

---

## หลักการสำคัญของ Notebook นี้

### Plant Mask
- พื้นหลังถ่ายบนผ้าดำ → V < BG_V_THRESHOLD (30) = background
- ทุก feature คำนวณบน px = plant_mask > 0 เท่านั้น ไม่รวม background

### Illumination Normalization (Cell 2.5)
- ต้องรันก่อน Cell 3 เสมอ
- ทับ img_bgr เพื่อให้ทุก cell ถัดไปใช้ภาพที่ normalize แล้วอัตโนมัติ
- เลือกได้ระหว่าง img_clahe (default) หรือ img_gw

### Global Variables ที่ต้องระวัง
| ตัวแปร | ปัญหา | สถานะ |
|--------|-------|-------|
| v | Cell 10 loop ทับ | แก้แล้ว (เปลี่ยนเป็น val) |
| s | Cell 18 loop (s, e, f_pt, depth) ทับ HSV s channel | ยังมี — ระวังถ้าเพิ่ม cell ที่ใช้ s |
| x, y, w | Cell 6 boundingRect ทับ | ยังมี |

---

## Constants ที่ปรับได้ (อยู่ใน Constants Cell)

| Constant | ค่า | ปรับเพื่อ |
|----------|-----|----------|
| BG_V_THRESHOLD | 30 | ตัดพื้นหลัง — ขึ้น=ตัดหลวม ลง=ตัดเข้ม |
| HSV_BROWN/YELLOW/GREEN | ranges | ปรับ color detection ตามสภาพแสง |
| EDGE_THICKNESS | 20px | ความหนาแถบขอบใบสำหรับ tip burn |
| DARK_RATIO | 0.4 | 40% ของ V_mean = dark spot threshold |
| MIN_SPOT_SIZE | 20px | กรอง noise ของ dark spot |
| HIGHLIGHT_PERCENTILE | 95 | top 5% brightness = specular highlight |
| MIN_DEFECT_DEPTH_PX | 5px | ความลึกขั้นต่ำของ convexity defect |
| GLCM_LEVELS | 16 | จำนวน gray level ใน GLCM |

---

## GitHub
- Repo: https://github.com/Karnpattana/Pre-Preprocess-label.git
- Branch: main
- Commit ด้วย /cmm <path> | <message>
