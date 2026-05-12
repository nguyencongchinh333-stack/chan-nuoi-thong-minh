# 03 — Phần Cứng & Thông Số Kỹ Thuật

---

## 3.1 Bộ Điều Khiển (Giống Hệ Thống Nhà Kính)

### CB2S — BK7231N

| Thông số | Giá trị |
|----------|---------|
| Chip | BK7231N |
| WiFi | 802.11 b/g/n **2.4GHz** (không hỗ trợ 5GHz) |
| Flash / RAM | 2MB / 256KB |
| Điện áp | 3.3VDC / ~250mA |
| Framework | LibreTuya Arduino |
| Platform | NextFarm Livestock qua MQTT |

### GPIO Phân Công

| GPIO | Chức năng | Ghi chú |
|------|-----------|---------|
| GPIO0 | I2C SDA | Pull-up 4.7kΩ |
| GPIO1 | I2C SCL | Pull-up 4.7kΩ |
| GPIO4 | UART TX → MH-Z19B | CO₂ sensor |
| GPIO5 | UART RX ← MH-Z19B | CO₂ sensor |
| GPIO8 | DS18B20 | Pull-up 4.7kΩ, dự phòng |
| GPIO9 | Nút chế độ | INPUT_PULLUP |
| GPIO10 | LED xanh AUTO | Qua R 220Ω |
| GPIO11 | LED đỏ THỦ CÔNG | Qua R 220Ω |
| GPIO12 | AJ-SR04M ECHO | Qua phân áp 5V→3.3V |
| GPIO13 | AJ-SR04M TRIG | OUTPUT |
| GPIO14 | HX711 DT (cân) | |
| GPIO26 | HX711 SCK (cân) | |

### I2C Devices Map

| Địa chỉ | Thiết bị |
|---------|---------|
| 0x20 | MCP23017 #1 (relay kênh 0–15) |
| 0x21 | MCP23017 #2 (relay kênh 16–31) |
| 0x27 | OLED SH1106 1.3" |
| 0x44 | SHT30 điểm 1 (đầu chuồng) |
| 0x45 | SHT30 điểm 2 (giữa chuồng) — địa chỉ thay đổi qua ADDR pin |
| 0x46 | SHT30 điểm 3 (cuối chuồng) |
| 0x48 | ADS1115 (ADC analog) |

---

## 3.2 Cảm Biến Đặc Thù Chăn Nuôi

### SHT30 — Nhiệt Độ + Độ Ẩm

| Thông số | Giá trị |
|----------|---------|
| Giao tiếp | I2C |
| Độ chính xác nhiệt độ | ±0.3°C |
| Độ chính xác độ ẩm | ±2%RH |
| Điện áp | 3.3V |
| Số điểm đo | **3 điểm** (đầu / giữa / cuối chuồng) |
| Lý do 3 điểm | Chuồng dài có thể chênh lệch 3–8°C theo chiều dài |

### SEN0321 — Cảm Biến NH₃ (Amoniac)

| Thông số | Giá trị |
|----------|---------|
| Giao tiếp | Analog → ADS1115 |
| Dải đo NH₃ | 10–300 ppm |
| Điện áp | 5V |
| Thời gian làm ấm | **24–48h** (bắt buộc trước khi dùng) |
| Hiệu chỉnh | Ngoài trời sạch (NH₃ ≈ 0) |
| Ngưỡng cảnh báo | 25 ppm |
| Ngưỡng nguy hiểm | 50 ppm |

> **Tại sao NH₃ quan trọng?**
> NH₃ > 25ppm: Gà/Heo giảm ăn, tăng trọng chậm, dễ bệnh hô hấp
> NH₃ > 50ppm: Nguy hiểm tính mạng vật nuôi và người chăn nuôi

### MH-Z19B — CO₂

| Thông số | Giá trị |
|----------|---------|
| Giao tiếp | UART 9600bps hoặc PWM |
| Dải đo | 400–5000 ppm |
| Điện áp | 5V / 150mA |
| Thời gian làm ấm | 3 phút |
| Kết nối | TX→GPIO5, RX→GPIO4 |

### AJ-SR04M — Mức Thức Ăn

| Thông số | Giá trị |
|----------|---------|
| Giao tiếp | TRIG + ECHO GPIO |
| Dải đo | 2–400 cm |
| Điện áp | 5V |
| Ghi chú | ECHO 5V → cần phân áp (R1=1kΩ, R2=2kΩ) → 3.3V cho GPIO |
| Tính toán | % cám = (H_thùng - khoảng_cách) / H_thùng × 100 |

### HX711 — Cân Load Cell

| Thông số | Giá trị |
|----------|---------|
| Độ phân giải | 24-bit |
| Giao tiếp | 2 GPIO (DT + SCK) |
| Điện áp | 3.3V–5V |
| Load cell | 50kg (gà) hoặc 200kg (heo) |
| Ứng dụng | Cân mẫu 3–5 con → tính khối lượng trung bình |

### ADS1115 — ADC 16-bit

| Thông số | Giá trị |
|----------|---------|
| Kênh | 4 single-ended hoặc 2 differential |
| Độ phân giải | 16-bit |
| Giao tiếp | I2C 0x48 |
| Sử dụng | SEN0321 (AIN0), áp suất nước (AIN1) |

---

## 3.3 Thiết Bị Làm Mát

### Bơm Phun Sương Cao Áp

| Thông số | Giá trị |
|----------|---------|
| Điện áp | 220VAC |
| Công suất | 750W (0.75HP) |
| Áp suất | 30–40 bar (phun sương mù) |
| Đầu phun | INOX 0.1mm — sương mù bay hơi ngay |
| Lọc | 5 micron đầu vào (bắt buộc) |
| Hiệu quả | Giảm 5–8°C khi độ ẩm ngoài < 70% |

### Tấm Làm Mát Cellulose (Cooling Pad)

| Thông số | Giá trị |
|----------|---------|
| Kích thước | 150×60×15cm / tấm |
| Vật liệu | Cellulose đặc biệt |
| Hiệu suất bay hơi | >85% |
| Tuổi thọ | 3–5 năm (tùy chất lượng nước) |
| Nguyên lý | Không khí đi qua tấm ướt → hạ nhiệt 5–8°C |
| Lắp đặt | Đầu chuồng, đối diện quạt hút |

---

## 3.4 Van Nước Uống 24VAC

| Thông số | Giá trị |
|----------|---------|
| Model | Airtac 2W-160-15 SS304 |
| Điện áp | 24VAC NC (thường đóng) |
| DN | DN15 hoặc DN20 |
| Gioăng | Viton — chịu clo trong nước |
| Kết hợp | Flow meter YF-B7 theo dõi lượng nước uống |

---

## 3.5 Motor Rèm Cửa

| Thông số | Giá trị |
|----------|---------|
| Điện áp | 220VAC |
| Công suất | 40W (khuyến nghị) |
| Loại | Hộp số giảm tốc |
| Bảo vệ | Cầu chì 5A riêng mỗi chiều |
| Giới hạn | Công tắc hành trình IP65 × 2 |
| An toàn | Không bật 2 chiều cùng lúc (firmware bảo vệ) |

---

## 3.6 Quạt Thông Gió

| Thông số | Gói 2 (Trung bình) | Gói 3 ✅ (Khuyến nghị) | Gói 4 (Cao cấp) |
|----------|:-:|:-:|:-:|
| Model | Panasonic 50cm | KDK FV-50TH | INOX nhập 60cm |
| Điện áp | 220VAC | 220VAC | 220VAC |
| Lưu lượng | ~2.000 m³/h | ~2.500 m³/h | ~3.500 m³/h |
| Công suất | 120W | 150W | 200W |
| Cấp độ bảo vệ | IP44 | IP44 | IP55 |
| Tuổi thọ | 3–5 năm | 5–8 năm | 8–12 năm |
| **Phù hợp chuồng** | ≤200m² | ≤500m² | >500m² |

---

## 3.7 Đèn Sưởi Hồng Ngoại

| Thông số | Bóng đỏ (Gói 1) | Ceramic (Gói 2/3) |
|----------|:-:|:-:|
| Điện áp | 220VAC E27 | 220VAC E27 |
| Công suất | 250W | 250W |
| Bức xạ | Ánh sáng đỏ 630–760nm | Hồng ngoại vô hình |
| Diện tích sưởi (từ 0.5m) | ~3m² | ~4m² |
| Tuổi thọ | ~2.000h | ~5.000h |
| Ưu điểm | Rẻ, thay dễ | Không làm chói gà, bền hơn |

**Lắp đặt:**
- Treo ở độ cao 0.5–0.7m với gà con tuần 1
- Tăng dần lên 1–1.5m theo tuần tuổi (giảm nhiệt)
- Bắt buộc có lồng bảo vệ chống vỡ + cháy bụi cám

---

## 3.8 Van Nước Uống 24VAC

| Thông số | Gói 1 (Đồng EPDM) | Gói 3 ✅ (SS304 Viton) |
|----------|:-:|:-:|
| Model | 2W-160-15 Covna | 2W-160-15 Airtac |
| Điện áp | 24VAC NC | 24VAC NC |
| DN | DN15 | DN15–DN20 |
| Gioăng | EPDM | **Viton** (chịu clo, phân bón) |
| Thân | Đồng | SS304 |
| Áp suất | 0–5 bar | 0–8 bar |
| Tuổi thọ | 1–2 năm | 5–7 năm |
| Ghi chú | Clo trong nước uống ăn mòn EPDM | Viton bền với clo, nước có khoáng |

---

## 3.9 Bơm Phun Sương Cao Áp

| Thông số | Áp thấp (Gói 1) | Áp cao ✅ (Gói 3) |
|----------|:-:|:-:|
| Áp suất | 3–5 bar | 30–40 bar |
| Đầu phun | 0.3mm (hạt lớn) | **0.1mm (sương mù)** |
| Tính năng | Làm ướt | Bay hơi ngay → Hiệu quả cao |
| Điện áp | 220VAC 0.5HP | 220VAC 0.75HP |
| Phù hợp độ ẩm | Mọi độ ẩm | Độ ẩm < 70% |
| Lọc đầu vào | 80 mesh | **5 micron** (bắt buộc) |

---

## 3.10 Máy Cho Ăn Tự Động

| Thông số | Máng kéo (Gói 1) | Vít tải INOX ✅ (Gói 3) |
|----------|:-:|:-:|
| Cơ cấu | Motor DC + xích kéo máng | Motor 220VAC + vít tải |
| Vật liệu | Nhựa + xích PE | **INOX 304** |
| Thùng chứa | Nhựa HDPE | **INOX 304 50kg** |
| Độ chính xác phân phối | Trung bình | Tốt (ổn định lượng cám/ca) |
| Vệ sinh | Tháo lắp khó | Tháo từng đoạn vít |
| Tuổi thọ | 2–3 năm | 7–10 năm |

---

## 3.11 Tấm Làm Mát Cellulose (Cooling Pad)

| Thông số | Giá trị |
|----------|---------|
| Kích thước chuẩn | 150×60×15cm |
| Vật liệu | Cellulose đặc biệt tẩm kháng khuẩn |
| Hiệu suất bay hơi | >85% |
| Độ bền | 3–5 năm (tùy chất lượng nước) |
| Áp lực nước | 0.1–0.3 bar (bơm tuần hoàn nhẹ) |
| Vị trí lắp | Đầu chuồng — đối diện quạt hút |
| Diện tích cần | = Lưu lượng quạt ÷ 3.600 (m²) |
| Vệ sinh | Xả nước bể hàng tuần, thay nước định kỳ |

---

*[← Kiến Trúc](02_kien-truc.md) | [Relay Map →](04_relay-channel-map.md)*
