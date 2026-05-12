# 13 — Tính Toán Thiết Kế Hệ Thống

> Hướng dẫn tính toán để chọn đúng thiết bị cho từng quy mô chuồng trại.
> Điền số liệu chuồng của bạn vào các công thức để ra kết quả thực tế.

---

# PHẦN 1 — THÔNG TIN CHUỒNG TRẠI

| Thông số | Ký hiệu | Đơn vị | Giá trị của bạn |
|---------|:-------:|:------:|:--------------:|
| Chiều dài chuồng | L | m | ___ |
| Chiều rộng chuồng | W | m | ___ |
| Chiều cao chuồng | H | m | ___ |
| Diện tích sàn | S = L × W | m² | ___ |
| Thể tích | V = L × W × H | m³ | ___ |
| Loại vật nuôi | — | — | Gà □  Heo □  Bò □ |
| Số đầu nuôi | N | con | ___ |
| Mật độ nuôi | d = N/S | con/m² | ___ |

---

# PHẦN 2 — TÍNH SỐ QUẠT THÔNG GIÓ

## 2.1 Công Thức Tính

```
Lưu lượng cần (m³/h) = Thể tích V × Số lần đổi khí mỗi giờ (ACH)

ACH theo loại vật nuôi:
  Gà thịt mùa hè:  60–80 ACH
  Gà thịt mùa đông: 4–6 ACH (thông gió tối thiểu)
  Heo thịt mùa hè: 40–60 ACH
  Heo thịt mùa đông: 3–5 ACH

Số quạt = Lưu lượng cần ÷ Lưu lượng 1 quạt
```

## 2.2 Ví Dụ Tính

```
Chuồng gà: 50m × 12m × 3m = 1.800m³
Mùa hè, ACH = 60:
  Lưu lượng = 1.800 × 60 = 108.000 m³/h
  Quạt 50cm KDK (2.500 m³/h):
  Số quạt = 108.000 ÷ 2.500 = 43 quạt

→ Hệ thống này (6 quạt) phù hợp chuồng nhỏ ≤ 15.000 m³/h
  Tức chuồng ~500m² × 3m = 1.500m³, ACH = 10 lần/giờ

Để tăng ACH: Thêm MCP23017 → thêm kênh relay → thêm quạt
```

## 2.3 Bảng Tra Nhanh — Số Quạt 50cm Cần Cho Chuồng Gà Mùa Hè

| Diện tích (m²) | Thể tích (m³) | Lưu lượng cần (m³/h) | Số quạt 50cm |
|:-:|:-:|:-:|:-:|
| 100 | 300 | 18.000 | 8 |
| 200 | 600 | 36.000 | 15 |
| 300 | 900 | 54.000 | 22 |
| 500 | 1.500 | 90.000 | 36 |
| 1.000 | 3.000 | 180.000 | 72 |

> Hệ thống này (6 quạt × 2.500 m³/h = 15.000 m³/h) → **Chuồng ≤100m² × 3m**
> Để dùng cho chuồng lớn hơn: Mở rộng MCP23017 thêm kênh relay

---

# PHẦN 3 — TÍNH CÔNG SUẤT SƯỞI

## 3.1 Công Thức

```
Công suất sưởi (W) = V × ΔT × 0.34

Trong đó:
  V  = Thể tích chuồng (m³)
  ΔT = Chênh lệch nhiệt độ (°C) = Nhiệt độ mục tiêu - Nhiệt độ ngoài trời lạnh nhất
  0.34 = Hệ số trao đổi nhiệt không khí (W/m³/°C)
```

## 3.2 Ví Dụ Tính

```
Chuồng gà con: 10m × 5m × 2.5m = 125m³
Mục tiêu: 35°C | Nhiệt độ ngoài lạnh nhất: 15°C
ΔT = 35 - 15 = 20°C

Công suất = 125 × 20 × 0.34 = 850W

→ Dùng 4 bóng đèn sưởi 250W = 1.000W (dư 150W, đủ dùng)

Nếu chuồng có cách nhiệt tốt: Giảm hệ số xuống 0.20–0.25
```

## 3.3 Bảng Tra Nhanh

| Thể tích (m³) | ΔT = 10°C | ΔT = 15°C | ΔT = 20°C |
|:-:|:-:|:-:|:-:|
| 50 | 170W | 255W | 340W |
| 100 | 340W | 510W | 680W |
| 200 | 680W | 1.020W | 1.360W |
| 500 | 1.700W | 2.550W | 3.400W |

> Hệ thống này: 4 đèn × 250W = **1.000W** → Phù hợp chuồng ≤ 150m³, ΔT ≤ 20°C

---

# PHẦN 4 — TÍNH HỆ THỐNG LÀM MÁT

## 4.1 Tấm Làm Mát Cooling Pad

```
Diện tích tấm (m²) = Lưu lượng quạt (m³/h) ÷ Vận tốc gió qua tấm (m/h)

Vận tốc tiêu chuẩn qua tấm: 3.600–4.200 m/h (1–1.17 m/s)

Ví dụ:
  6 quạt × 2.500 m³/h = 15.000 m³/h
  Diện tích tấm = 15.000 ÷ 3.600 = 4,17m²
  → Dùng tấm 150×60×15cm (0,9m²/tấm) × 5 tấm = 4,5m²
```

## 4.2 Phun Sương

```
Số đầu phun = Chiều dài chuồng (m) ÷ Khoảng cách đầu phun (3m)

Lưu lượng bơm cần (L/h) = Số đầu phun × Lưu lượng 1 đầu phun

Đầu phun 0.1mm (20 bar): ~6–8 L/h/đầu
Đầu phun 0.3mm (5 bar):  ~30–40 L/h/đầu

Ví dụ chuồng 50m:
  Số đầu phun = 50 ÷ 3 = 17 đầu
  Bơm cao áp (0.1mm): 17 × 7 = 119 L/h ≈ 2 L/phút
  → Bơm 0.5HP (30 L/phút) là quá dư — dùng bơm nhỏ hơn 0.25HP
```

## 4.3 Hiệu Quả Làm Mát Theo Độ Ẩm

| Độ ẩm ngoài trời | Hiệu quả làm mát |
|:-:|:-:|
| <40% | Giảm 8–10°C |
| 40–60% | Giảm 5–7°C |
| 60–80% | Giảm 2–4°C |
| >80% | Không hiệu quả |

> Việt Nam mùa hè độ ẩm 70–90% → Phun sương ít hiệu quả → Tấm làm mát hiệu quả hơn

---

# PHẦN 5 — TÍNH CHIẾU SÁNG

## 5.1 Cường Độ Ánh Sáng Yêu Cầu

| Vật nuôi | Lux yêu cầu | Giai đoạn |
|---------|:-----------:|---------|
| Gà con | 20–40 lux | Tuần 1–2 (cần thấy máng ăn, uống) |
| Gà thịt | 5–20 lux | Từ tuần 3 (kích thích ăn không cần quá sáng) |
| Gà đẻ | 10–30 lux | Kích thích đẻ |
| Heo | 50–100 lux | Sinh trưởng bình thường |

## 5.2 Công Thức Tính Số Bóng Đèn

```
Số bóng = (Lux yêu cầu × Diện tích S) ÷ (Quang thông 1 bóng × Hiệu suất)

Hiệu suất bố trí: 0.5–0.7 (tổn thất do phản xạ, vị trí)
LED 40W ≈ 4.000–5.000 lumen

Ví dụ chuồng gà 100m², yêu cầu 20 lux:
  Số bóng = (20 × 100) ÷ (4.500 × 0.6)
           = 2.000 ÷ 2.700
           = 0.74 → Làm tròn lên 1 bóng/khu
  4 khu → 4 bóng LED 40W (mỗi bóng phủ 25m²)
```

---

# PHẦN 6 — TÍNH MẬT ĐỘ VẬT NUÔI

## 6.1 Mật Độ Tối Đa Khuyến Nghị

| Vật nuôi | Mật độ tiêu chuẩn | Mật độ tối đa |
|---------|:-:|:-:|
| Gà thịt tuần 1–3 | 20–25 con/m² | 30 con/m² |
| Gà thịt tuần 4–7 | 12–15 con/m² | 20 con/m² |
| Gà đẻ chuồng lồng | 400–450 cm²/con | — |
| Gà đẻ chuồng nền | 5–7 con/m² | 8 con/m² |
| Heo con (<30kg) | 0.3–0.4 m²/con | — |
| Heo thịt (30–100kg) | 0.6–0.8 m²/con | — |

## 6.2 Số Con Tối Đa Theo Diện Tích

| Diện tích (m²) | Gà thịt (tuần 5) | Gà đẻ nền | Heo thịt |
|:-:|:-:|:-:|:-:|
| 50 | 650–750 | 250–350 | 60–80 |
| 100 | 1.300–1.500 | 500–700 | 120–160 |
| 200 | 2.600–3.000 | 1.000–1.400 | 240–320 |
| 500 | 6.500–7.500 | 2.500–3.500 | 600–800 |

---

# PHẦN 7 — CHECKLIST THIẾT KẾ

```
□ Tính thể tích chuồng V = L × W × H = ___ m³
□ Tính lưu lượng quạt cần = V × ACH = ___ m³/h
□ Chọn số quạt = Lưu lượng ÷ Lưu lượng 1 quạt = ___
□ Tính công suất sưởi = V × ΔT × 0.34 = ___ W
□ Chọn số bóng sưởi = ___ bóng × ___ W
□ Tính diện tích tấm làm mát = Lưu lượng ÷ 3.600 = ___ m²
□ Số đầu phun sương = L ÷ 3 = ___
□ Số bóng đèn chiếu sáng = (Lux × S) ÷ (Lumen × 0.6) = ___
□ Mật độ vật nuôi = N ÷ S = ___ con/m² (trong ngưỡng cho phép?)
□ Số relay cần = (quạt) + (sưởi) + (đèn) + (van) + (rèm×2) + (cho ăn) = ___
   Nếu > 31: Thêm MCP23017 (16 kênh/chip, ~32.000đ/chip)
```

---

*[← Lịch Chăn Nuôi](12_lich-chan-nuoi.md) | [← README](README.md)*
