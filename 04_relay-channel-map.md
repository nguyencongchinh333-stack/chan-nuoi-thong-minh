# 04 — Bản Đồ Kênh Relay

---

## 4.1 Tổng Quan

| Thông tin | Giá trị |
|-----------|---------|
| Tổng kênh vật lý | 32 (2 × MCP23017 × 16) |
| Kênh sử dụng | **31 kênh** |
| Kênh dự phòng | **1 kênh** (kênh 15) |
| Logic | **Active LOW** — 0V = BẬT relay |

---

## 4.2 Bảng 31 Kênh Chi Tiết

| Kênh | Define | Thiết Bị | Điện Áp | MCP | GPIO |
|:----:|--------|---------|---------|-----|------|
| **0** | `K_BOM` | Bơm chính cấp nước | 220VAC | #1 0x20 | GPA0 |
| **1** | `K_QUAT1` | Quạt thông gió 1 | 220VAC | #1 0x20 | GPA1 |
| **2** | `K_QUAT2` | Quạt thông gió 2 | 220VAC | #1 0x20 | GPA2 |
| **3** | `K_QUAT3` | Quạt thông gió 3 | 220VAC | #1 0x20 | GPA3 |
| **4** | `K_QUAT4` | Quạt thông gió 4 | 220VAC | #1 0x20 | GPA4 |
| **5** | `K_QUAT5` | Quạt thông gió 5 | 220VAC | #1 0x20 | GPA5 |
| **6** | `K_QUAT6` | Quạt thông gió 6 | 220VAC | #1 0x20 | GPA6 |
| **7** | `K_BOM_PHUN` | Bơm phun sương | 220VAC | #1 0x20 | GPA7 |
| **8** | `K_BOM_TAM` | Bơm tấm làm mát | 220VAC | #1 0x20 | GPB0 |
| **9** | `K_SUOI1` | Đèn sưởi 1 | 220VAC | #1 0x20 | GPB1 |
| **10** | `K_SUOI2` | Đèn sưởi 2 | 220VAC | #1 0x20 | GPB2 |
| **11** | `K_SUOI3` | Đèn sưởi 3 | 220VAC | #1 0x20 | GPB3 |
| **12** | `K_SUOI4` | Đèn sưởi 4 | 220VAC | #1 0x20 | GPB4 |
| **13** | `K_DEN_A` | Đèn chiếu sáng khu A | 220VAC | #1 0x20 | GPB5 |
| **14** | `K_DEN_B` | Đèn chiếu sáng khu B | 220VAC | #1 0x20 | GPB6 |
| **15** | `K_SPARE` | **Dự phòng** | — | #1 0x20 | GPB7 |
| **16** | `K_VAN_N1` | Van nước uống khu 1 | 24VAC | #2 0x21 | GPA0 |
| **17** | `K_VAN_N2` | Van nước uống khu 2 | 24VAC | #2 0x21 | GPA1 |
| **18** | `K_VAN_N3` | Van nước uống khu 3 | 24VAC | #2 0x21 | GPA2 |
| **19** | `K_VAN_N4` | Van nước uống khu 4 | 24VAC | #2 0x21 | GPA3 |
| **20** | `K_REM1_MO` | Rèm cửa 1 — MỞ | 220VAC | #2 0x21 | GPA4 |
| **21** | `K_REM1_DONG` | Rèm cửa 1 — ĐÓNG | 220VAC | #2 0x21 | GPA5 |
| **22** | `K_REM2_MO` | Rèm cửa 2 — MỞ | 220VAC | #2 0x21 | GPA6 |
| **23** | `K_REM2_DONG` | Rèm cửa 2 — ĐÓNG | 220VAC | #2 0x21 | GPA7 |
| **24** | `K_REM3_MO` | Rèm cửa 3 — MỞ | 220VAC | #2 0x21 | GPB0 |
| **25** | `K_REM3_DONG` | Rèm cửa 3 — ĐÓNG | 220VAC | #2 0x21 | GPB1 |
| **26** | `K_REM4_MO` | Rèm cửa 4 — MỞ | 220VAC | #2 0x21 | GPB2 |
| **27** | `K_REM4_DONG` | Rèm cửa 4 — ĐÓNG | 220VAC | #2 0x21 | GPB3 |
| **28** | `K_CHOAN1` | Máy cho ăn 1 | 220VAC | #2 0x21 | GPB4 |
| **29** | `K_CHOAN2` | Máy cho ăn 2 | 220VAC | #2 0x21 | GPB5 |
| **30** | `K_DEN_C` | Đèn chiếu sáng khu C | 220VAC | #2 0x21 | GPB6 |
| **31** | `K_DEN_D` | Đèn chiếu sáng khu D | 220VAC | #2 0x21 | GPB7 |

---

## 4.3 Phân Nhóm Chức Năng

```
K0          BƠM CHÍNH
K1–K6       QUẠT THÔNG GIÓ × 6  (bật dần theo nhiệt độ + NH₃)
K7          BƠM PHUN SƯƠNG
K8          BƠM TẤM LÀM MÁT
K9–K12      ĐÈN SƯỞI × 4        (bật/tắt theo nhiệt độ mục tiêu)
K13–K14     ĐÈN CHIẾU SÁNG A+B  (quang chu kỳ)
K15         DỰ PHÒNG
K16–K19     VAN NƯỚC UỐNG × 4   (24VAC NC)
K20–K27     RÈM CỬA × 4         (2 chiều MỞ/ĐÓNG)
K28–K29     MÁY CHO ĂN × 2
K30–K31     ĐÈN CHIẾU SÁNG C+D  (quang chu kỳ)
```

---

## 4.4 An Toàn Rèm Cửa 2 Chiều

```
⚠️ KHÔNG bật K_REM1_MO và K_REM1_DONG cùng lúc → chập mạch

Logic bắt buộc:
  1. setRelay(K_REMx_MO,   OFF)
  2. setRelay(K_REMx_DONG, OFF)
  3. delay(500ms)
  4. setRelay(K_REMx_...,  ON)  ← chiều cần thiết
```

---

*[← Kiến Trúc](02_kien-truc.md) | [Đấu Nối →](05_dau-noi-dien.md)*
