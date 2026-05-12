# 08 — Tự Động Hóa & Lịch Vận Hành

---

## 8.1 Thông Gió Tự Động

### Cài Đặt Ngưỡng Nhiệt Độ (6 Cấp)

| Cấp | Ngưỡng | Quạt bật | Làm mát |
|:---:|--------|---------|---------|
| 0 | < T0 | Tắt hết | Tắt |
| 1 | ≥ T1 | Q1 + Q2 | Tắt |
| 2 | ≥ T2 | Q1 + Q2 + Q3 + Q4 | Tắt |
| 3 | ≥ T3 | Q1 – Q6 (tất cả) | Tắt |
| 4 | ≥ T4 | Q1 – Q6 | Bơm phun sương |
| 5 | ≥ T5 | Q1 – Q6 | Phun sương + tấm làm mát |

**Giá trị mặc định cho gà thịt:**

| Cấp | T (°C) |
|:---:|:------:|
| T0 | 22 |
| T1 | 26 |
| T2 | 28 |
| T3 | 30 |
| T4 | 33 |
| T5 | 35 |

Cài đặt qua MQTT — thay đổi được từ app theo loại vật nuôi và tuần tuổi.

### Ưu Tiên NH₃ — Tuyệt Đối

```
NH₃ > 25ppm → Bật TẤT CẢ 6 quạt (bất kể nhiệt độ)
              → LCD nhấp nháy cảnh báo
              → MQTT alert: "NH3_CANH_BAO"

NH₃ > 50ppm → Bật tất cả quạt
              → LCD đỏ NGUY HIỂM
              → MQTT alert: "NH3_NGUY_HIEM"
              → Gửi thông báo app ngay lập tức
```

---

## 8.2 Sưởi Ấm Tự Động

### Nhiệt Độ Mục Tiêu Theo Tuần Tuổi

| Tuần | Gà con (°C) | Heo con (°C) | Ghi chú |
|:----:|:-----------:|:-----------:|---------|
| 1 | 35 | 32 | Mới nở / Mới sinh |
| 2 | 32 | 30 | |
| 3 | 29 | 28 | |
| 4 | 26 | 26 | |
| 5+ | 24 | 24 | Nhiệt độ chuồng bình thường |

**Logic điều khiển (Hysteresis ±1°C):**
```
Đặt mục tiêu = 35°C (tuần 1)
  Nhiệt độ < 34°C → BẬT đèn sưởi
  Nhiệt độ > 36°C → TẮT đèn sưởi
  34°C–36°C       → Giữ nguyên trạng thái
```

### Cài Đặt Tuần Hiện Tại

```bash
# Cài tuần tuổi 2 cho gà
mosquitto_pub -t "livestock/{id}/suoi/set" \
  -m '{"loai":"ga","tuan":2,"nhiet_do_muc_tieu":32}'
```

---

## 8.3 Chiếu Sáng — Quang Chu Kỳ

### Lịch Mặc Định

| Loại | Giờ sáng | Bật lúc | Tắt lúc |
|------|:--------:|---------|---------|
| Gà đẻ | 16h | 05:00 | 21:00 |
| Gà thịt | 23h | 04:00 | 03:00+1 |
| Gà con tuần 1 | 24h | Liên tục | — |
| Heo | 12h | 06:00 | 18:00 |

### Giao Diện Cài Đặt

```json
{
  "den": {
    "gio_bat": 5,
    "phut_bat": 0,
    "gio_tat": 21,
    "phut_tat": 0,
    "khu": [true, true, true, true]
  }
}
```

---

## 8.4 Cho Ăn Tự Động

### Lịch Mặc Định

| Ca | Giờ | Thời gian chạy motor |
|:--:|-----|:-------------------:|
| Sáng | 06:00 | 30 giây |
| Trưa | 11:00 | 30 giây |
| Chiều | 16:00 | 30 giây |

### Cài Đặt Qua MQTT

```json
{
  "cho_an": {
    "lich": [
      {"gio": 6, "phut": 0, "giay_chay": 30},
      {"gio": 11, "phut": 0, "giay_chay": 30},
      {"gio": 16, "phut": 0, "giay_chay": 30}
    ],
    "may": [true, true]
  }
}
```

### Cảnh Báo Mức Cám

| Mức | Ngưỡng | Hành động |
|-----|:------:|-----------|
| Bình thường | >30% | Không cảnh báo |
| Cần bổ sung | ≤20% | MQTT + LCD thông báo |
| Sắp hết | ≤10% | MQTT + LCD cảnh báo vàng |
| Hết | ≤5% | MQTT + LCD cảnh báo đỏ, gửi app |

---

## 8.5 Rèm Cửa Tự Động

### Theo Nhiệt Độ

```
Nhiệt độ > T_rem_mo → Mở rèm (tăng thông gió tự nhiên)
Nhiệt độ < T_rem_dong → Đóng rèm (giữ ấm)

Mặc định:
  T_rem_mo  = 30°C
  T_rem_dong = 25°C
```

### Theo Giờ

```
Ban ngày (08:00–17:00): Mở rèm
Ban đêm (17:00–08:00): Đóng rèm
```

---

## 8.6 Hiển Thị LCD & Cảnh Báo

### Màn Hình Bình Thường (LCD 20×4)

```
Dòng 1: [Chế độ] [NH₃ ppm] [Nhiệt độ °C]
Dòng 2: [Số quạt/6] [Độ ẩm %]
Dòng 3: [Đèn ON/OFF] [Sưởi ON/OFF] [Làm mát ON/OFF]
Dòng 4: [Cho ăn giờ tiếp] [Cám còn %]

Ví dụ:
  AUTO  NH3:08p  28.5C
  Quat: 4/6  DoAm:72%
  Den:ON  Suoi:OFF Mat:OFF
  ChoAn:16:00  Cam: 85%
```

### Màn Hình Cảnh Báo

```
Khi NH₃ > 25ppm:
  !!! CANH BAO !!!
  NH3: 28ppm > 25ppm
  Da bat toan bo quat
  Kiem tra thong gio!

Khi nhiệt độ cao:
  !!! CANH BAO !!!
  Nhiet do: 36.5C
  Da bat lam mat
  Kiem tra he thong!

Khi hết cám:
  [*] THONG BAO [*]
  Cam may 1: 4%
  Can bo sung ngay
  Lien he: 090 xxx
```

### MQTT Alert Topics

```
livestock/{id}/alert/nh3     ← Cảnh báo NH₃
livestock/{id}/alert/nhiet   ← Cảnh báo nhiệt độ
livestock/{id}/alert/cam     ← Cảnh báo mức cám
livestock/{id}/alert/nuoc    ← Cảnh báo nước uống
livestock/{id}/alert/dien    ← Cảnh báo mất điện / phục hồi
```

---

## 8.7 Phục Hồi Sau Mất Điện

Khi mất điện giữa chừng:
- Trạng thái thiết bị lưu vào flash (Preferences)
- Khi có điện: kết nối WiFi + NTP → phục hồi đúng trạng thái
- Quạt được bật lại ngay khi có điện (ưu tiên thông gió)
- Cho ăn và lịch chiếu sáng tiếp tục theo giờ thực

---

*[← MQTT API](07_mqtt-api.md) | [Lắp Đặt →](09_lap-dat.md)*
