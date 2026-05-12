# 12 — Lịch Chăm Sóc Theo Tuần Tuổi

> Tài liệu tham khảo để cài đặt ngưỡng tự động trong firmware.
> Các thông số cài vào MQTT từ app theo từng giai đoạn.

---

# PHẦN 1 — GÀ THỊT (Broiler) — 42–49 Ngày

## 1.1 Bảng Thông Số Theo Tuần

| Tuần | Ngày | Nhiệt độ (°C) | Độ ẩm (%RH) | Đèn (h/ngày) | Thức ăn | Nước uống |
|:----:|:----:|:-------------:|:-----------:|:------------:|---------|-----------|
| 1 | 1–7 | **33–35** | 60–70 | 23–24 | Cám khởi động (21% protein) | Ad libitum |
| 2 | 8–14 | **30–32** | 60–70 | 20–23 | Cám khởi động | Ad libitum |
| 3 | 15–21 | **27–29** | 60–65 | 18–20 | Cám tăng trưởng (19% protein) | Ad libitum |
| 4 | 22–28 | **25–27** | 60–65 | 16–18 | Cám tăng trưởng | Ad libitum |
| 5 | 29–35 | **23–25** | 55–65 | 16 | Cám vỗ béo (17% protein) | Ad libitum |
| 6 | 36–42 | **22–24** | 55–65 | 16 | Cám vỗ béo | Ad libitum |
| 7 | 43–49 | **21–23** | 55–65 | 16 | Cám vỗ béo | Ad libitum |

**Cài đặt MQTT theo tuần (ví dụ tuần 2):**
```bash
mosquitto_pub -t "livestock/{id}/suoi/set" -m \
  '{"loai":"ga_thit","tuan":2,"nhiet_do_muc_tieu":31}'
```

---

## 1.2 Ngưỡng Quạt Theo Giai Đoạn

| Giai đoạn | T1 (°C) | T2 (°C) | T3 (°C) | T4 (°C) | T5 (°C) |
|-----------|:-------:|:-------:|:-------:|:-------:|:-------:|
| Tuần 1 | 36 | 38 | 40 | 42 | 44 |
| Tuần 2–3 | 33 | 35 | 37 | 39 | 41 |
| Tuần 4–5 | 28 | 30 | 32 | 34 | 36 |
| Tuần 6–7 | 26 | 28 | 30 | 32 | 34 |

> Tuần 1: Quạt ít bật hơn để giữ ấm — chỉ bật khi quá nóng

---

## 1.3 Trọng Lượng Mục Tiêu

| Tuần | Trọng lượng TB (g) | Tăng trọng/tuần (g) |
|:----:|:-----------------:|:-------------------:|
| 1 | 170–190 | 130–150 |
| 2 | 450–500 | 280–310 |
| 3 | 900–1.000 | 450–500 |
| 4 | 1.500–1.700 | 600–700 |
| 5 | 2.100–2.400 | 600–700 |
| 6 | 2.700–3.100 | 600–700 |
| 7 | 3.200–3.800 | 500–700 |

---

# PHẦN 2 — GÀ ĐẺ TRỨNG (Layer) — Vòng Đời

## 2.1 Giai Đoạn Úm (0–6 Tuần)

| Tuần | Nhiệt độ (°C) | Đèn (h/ngày) | Ghi chú |
|:----:|:-------------:|:------------:|---------|
| 1 | 33–35 | 23–24 | Úm gà con |
| 2 | 30–32 | 20 | |
| 3 | 27–29 | 18 | |
| 4–6 | 24–26 | 16 | Chuyển sang chuồng lớn |

## 2.2 Giai Đoạn Hậu Bị (7–18 Tuần)

| Tuần | Nhiệt độ (°C) | Đèn (h/ngày) | Lý do |
|:----:|:-------------:|:------------:|-------|
| 7–12 | 22–26 | **8–10** | Hạn chế đèn để phát triển cơ thể |
| 13–16 | 20–24 | **10–12** | Chuẩn bị sinh sản |
| 17–18 | 18–22 | **14** | Kích thích tuyến sinh dục |

## 2.3 Giai Đoạn Đẻ Trứng (19+ Tuần)

| Thông số | Giá trị | Ghi chú |
|---------|---------|---------|
| Nhiệt độ tối ưu | **18–24°C** | >28°C giảm đẻ rõ rệt |
| Chiếu sáng | **16h/ngày** | Bật 05:00, tắt 21:00 |
| Cường độ ánh sáng | 10–30 lux | Đủ để thấy, không quá sáng |
| Độ ẩm | 60–70%RH | |

**Lịch tăng dần đèn (kích thích đẻ):**
```
Tuần 17: 14h → Tuần 18: 15h → Tuần 19+: 16h
Tăng 30 phút/tuần — không tăng đột ngột
```

---

# PHẦN 3 — HEO THỊT — 16–20 Tuần

## 3.1 Bảng Thông Số Theo Giai Đoạn

| Giai đoạn | Tuần | Trọng lượng (kg) | Nhiệt độ (°C) | Độ ẩm (%RH) | Thức ăn |
|-----------|:----:|:----------------:|:-------------:|:-----------:|---------|
| Heo con tập ăn | 1–4 | 5–15 | **28–32** | 65–75 | Cám tập ăn (20% protein) |
| Heo choai | 5–8 | 15–35 | **24–28** | 60–70 | Cám choai (18% protein) |
| Heo thịt giai đoạn 1 | 9–12 | 35–65 | **22–26** | 60–70 | Cám thịt 1 (16% protein) |
| Heo thịt giai đoạn 2 | 13–16 | 65–95 | **20–24** | 55–65 | Cám thịt 2 (14% protein) |
| Heo thịt giai đoạn 3 | 17–20 | 95–120 | **18–22** | 55–65 | Cám thịt 3 (12–13% protein) |

---

## 3.2 Ngưỡng Nhiệt Độ Quạt Heo

| Giai đoạn | T1 | T2 | T3 (phun sương) | NH₃ bật hết |
|-----------|:--:|:--:|:---------------:|:-----------:|
| Heo con | 34 | 36 | 38 | 20 ppm |
| Heo choai | 29 | 31 | 33 | 25 ppm |
| Heo thịt | 26 | 28 | 30 | 25 ppm |

> Heo nhạy cảm với NH₃ hơn gà — ngưỡng cảnh báo heo con 20ppm (thấp hơn gà)

---

## 3.3 Trọng Lượng Xuất Bán

| Mục đích | Trọng lượng | Tuần nuôi |
|---------|:-----------:|:---------:|
| Heo choai | 30–35 kg | 8–10 tuần |
| Heo thịt tiêu chuẩn | 95–100 kg | 16–18 tuần |
| Heo thịt nặng | 110–120 kg | 18–20 tuần |

---

# PHẦN 4 — CÀI ĐẶT MQTT THEO TỪNG GIAI ĐOẠN

## 4.1 Script Cài Đặt Gà Thịt Tuần 1

```bash
# Nhiệt độ sưởi
mosquitto_pub -t "livestock/{id}/suoi/set" -m \
  '{"loai":"ga_thit","tuan":1,"nhiet_do_muc_tieu":34,"hysteresis":1}'

# Ngưỡng quạt
mosquitto_pub -t "livestock/{id}/quat/nguong/set" -m \
  '{"t0":28,"t1":36,"t2":38,"t3":40,"t4":42,"t5":44,"nh3_bat_het":25}'

# Chiếu sáng 23h/ngày
mosquitto_pub -t "livestock/{id}/den/lich/set" -m \
  '{"gio_bat":4,"phut_bat":0,"gio_tat":3,"phut_tat":0}'

# Cho ăn 4 lần/ngày (gà con)
mosquitto_pub -t "livestock/{id}/cho_an/lich/set" -m \
  '{"lich":[
    {"gio":6,"phut":0,"giay_chay":20,"may":[true,true]},
    {"gio":10,"phut":0,"giay_chay":20,"may":[true,true]},
    {"gio":14,"phut":0,"giay_chay":20,"may":[true,true]},
    {"gio":18,"phut":0,"giay_chay":20,"may":[true,true]}
  ]}'
```

## 4.2 Script Cài Đặt Gà Thịt Tuần 5

```bash
mosquitto_pub -t "livestock/{id}/suoi/set" -m \
  '{"loai":"ga_thit","tuan":5,"nhiet_do_muc_tieu":24,"hysteresis":1}'

mosquitto_pub -t "livestock/{id}/quat/nguong/set" -m \
  '{"t0":20,"t1":26,"t2":28,"t3":30,"t4":32,"t5":34,"nh3_bat_het":25}'

# Chiếu sáng 16h
mosquitto_pub -t "livestock/{id}/den/lich/set" -m \
  '{"gio_bat":5,"phut_bat":0,"gio_tat":21,"phut_tat":0}'

# Cho ăn 3 lần/ngày
mosquitto_pub -t "livestock/{id}/cho_an/lich/set" -m \
  '{"lich":[
    {"gio":6,"phut":0,"giay_chay":45,"may":[true,true]},
    {"gio":11,"phut":0,"giay_chay":45,"may":[true,true]},
    {"gio":16,"phut":0,"giay_chay":45,"may":[true,true]}
  ]}'
```

---

# PHẦN 5 — BẢNG THEO DÕI HÀNG NGÀY

## 5.1 Nhật Ký Kiểm Tra Sáng

```
Ngày: ___/___/______    Ca: Sáng □  Trưa □  Chiều □

MÔI TRƯỜNG:
  Nhiệt độ đo: ___°C (điểm 1) ___°C (điểm 2) ___°C (điểm 3)
  Độ ẩm: ___%
  NH₃: ___ ppm   (phải < 25 ppm)
  CO₂: ___ ppm   (phải < 3000 ppm)

THIẾT BỊ:
  □ Quạt chạy đủ (số quạt: ___)
  □ Đèn chiếu sáng đúng lịch
  □ Đèn sưởi bật/tắt đúng
  □ Nước uống chảy bình thường (4 khu)
  □ Máy cho ăn chạy đúng giờ

VẬT NUÔI:
  Số con kiểm tra: ___
  Trọng lượng TB: ___ g/kg
  Màu phân: Bình thường □  Bất thường □ (mô tả: _______)
  Số con chết: ___ (nguyên nhân: _______)
  Hành vi: Bình thường □  Lờ đờ □  Không ăn □

THỨC ĂN:
  Mức cám máy 1: ___%   Máy 2: ___%
  Cần bổ sung: Có □  Không □

GHI CHÚ: _______________________________________________
```

## 5.2 Bảng Theo Dõi Tuần

| Ngày | Nhiệt độ TB | NH₃ max | Số con chết | Trọng lượng mẫu | Ghi chú |
|:----:|:-----------:|:-------:|:-----------:|:---------------:|---------|
| T2 | | | | | |
| T3 | | | | | |
| T4 | | | | | |
| T5 | | | | | |
| T6 | | | | | |
| T7 | | | | | |
| CN | | | | | |
| **TB tuần** | | | | | |

---

*[← Báo Giá](11_bang-gia.md) | [Tính Toán Thiết Kế →](13_tinh-toan-thiet-ke.md)*
