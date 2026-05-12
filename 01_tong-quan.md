# 01 — Tổng Quan & Định Vị Trong NextFarm Ecosystem

---

## 1.1 NextFarm Platform — Module Livestock

NextFarm hỗ trợ chăn nuôi trong tab **IoT → Devices & Solutions → Livestock**. Cùng hệ sinh thái với trồng trọt, cho phép quản lý tập trung nhiều loại mô hình nông nghiệp trên 1 platform.

### Phân Loại Giải Pháp

| Nhóm | Quy mô | Giải pháp |
|------|--------|-----------|
| **Nhỏ / Tự lắp** | <500m², chuồng gia đình | Tuya / Shelly — vài trăm ngàn/thiết bị |
| **Vừa / DIY** | 200–1000m² | **Dự án này** — CB2S DIY Controller |
| **Lớn / Chuyên nghiệp** | >1000m², trang trại công nghiệp | Nextfarm Pro Livestock |

---

## 1.2 Áp Dụng Cho Các Loại Vật Nuôi

| Loại | Đặc Thù | Phù Hợp |
|------|---------|---------|
| **Gà công nghiệp** | Thông gió 24/7, quang chu kỳ, sưởi gà con | ✅ Rất phù hợp |
| **Heo** | Làm mát mạnh, phun sương, kiểm soát NH₃ | ✅ Rất phù hợp |
| **Bò sữa** | Quạt làm mát, chiếu sáng, vắt sữa tự động | ✅ Phù hợp |
| **Cá / Tôm** | Sục khí, bơm tuần hoàn | ⚠️ Cần điều chỉnh thêm |

---

## 1.3 Tính Năng Hệ Thống

### Điều Khiển Thiết Bị (31 Relay)
| Hệ thống | Thiết bị | Kênh |
|---------|---------|:----:|
| Thông gió | 6 quạt hút gió | 6 |
| Làm mát | Bơm phun sương + bơm tấm cooling pad | 2 |
| Sưởi ấm | 4 cụm đèn hồng ngoại | 4 |
| Chiếu sáng | 4 khu đèn LED | 4 |
| Nước uống | 4 van solenoid 24VAC | 4 |
| Rèm cửa | 4 rèm × 2 chiều | 8 |
| Cho ăn | 2 máy vít tải | 2 |
| Bơm chính | 1 bơm cấp nước | 1 |

### Cảm Biến Đầu Vào
| Cảm biến | Mục đích |
|---------|---------|
| Nhiệt độ + độ ẩm (3 điểm) | Điều khiển quạt, sưởi, làm mát |
| NH₃ Amoniac | Bật quạt khẩn cấp khi >25ppm |
| CO₂ | Giám sát chất lượng không khí |
| Cân load cell | Theo dõi tăng trọng |
| Mức thức ăn (ultrasonic) | Cảnh báo hết cám |
| Flow meter nước | Theo dõi lượng uống |

### Tự Động Hóa
- Thông gió theo nhiệt độ (6 cấp độ) + NH₃ (khẩn cấp)
- Làm mát phun sương khi > 33°C
- Sưởi theo nhiệt độ mục tiêu (tuỳ tuần tuổi)
- Quang chu kỳ chiếu sáng (lịch tùy chỉnh)
- Cho ăn theo lịch + cân trọng lượng
- Phục hồi sau mất điện bằng NTP

---

## 1.4 So Sánh Chi Phí

| | **DIY Controller (dự án này)** | Giải pháp Pro |
|--|--|--|
| Bộ điều khiển | **~2.585.000đ** | 20–50 triệu |
| Lắp đặt | Tự lắp | Chuyên nghiệp |
| Tùy biến | Hoàn toàn | Hạn chế |
| Bảo hành | Tự bảo trì | Có SLA |

---

## 1.5 Ngưỡng Môi Trường Quan Trọng

### Nhiệt Độ Tối Ưu

| Vật nuôi | Tối ưu | Nguy hiểm |
|---------|:------:|:---------:|
| Gà con tuần 1 | 33–35°C | <30°C / >38°C |
| Gà thịt 4+ tuần | 20–26°C | >30°C |
| Gà đẻ | 18–24°C | >28°C |
| Heo con | 28–32°C | <26°C |
| Heo thịt | 18–24°C | >27°C |
| Bò sữa | 10–20°C | >24°C |

### Ngưỡng Khí Độc

| Khí | Bình thường | Cảnh báo | Nguy hiểm |
|-----|:-----------:|:--------:|:---------:|
| NH₃ (Amoniac) | <10 ppm | **25 ppm** | >50 ppm |
| CO₂ | <2.000 ppm | **3.000 ppm** | >5.000 ppm |
| H₂S | <0.5 ppm | **5 ppm** | >20 ppm |

---

## 1.6 Giới Hạn Hiện Tại

| Giới hạn | Giải pháp |
|----------|-----------|
| Không đo pH nước uống | Thêm cảm biến pH analog qua ADS1115 |
| Không đo trọng lượng tự động toàn đàn | Load cell cân mẫu thủ công |
| Phụ thuộc Internet (NTP + MQTT) | Shelly cho giải pháp offline |
| Tối đa 31 kênh (phiên bản này) | Thêm MCP23017 để mở rộng |

---

*[← README](README.md) | [Kiến Trúc →](02_kien-truc.md)*
