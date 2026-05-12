# 09 — Hướng Dẫn Lắp Đặt

---

## 9.1 Yêu Cầu Trước Khi Lắp

### Dụng Cụ
- Tua vít dẹt + Phillips, đồng hồ vạn năng (VOM) — bắt buộc
- Kìm tuốt dây, kìm bấm đầu cos
- Máy khoan, keo silicon chống ẩm
- Mỏ hàn thiếc (hàn mạch CB2S + MCP23017)
- Băng điện, co nhiệt, nhựa thông

### Lưu Ý Môi Trường Chuồng Trại
> Chuồng trại có độ ẩm cao, khí NH₃ ăn mòn, bụi cám — yêu cầu cao hơn nhà kính:
- Tất cả đầu nối dây điện **bắt buộc bọc co nhiệt** + keo silicon
- Tủ điện **IP65** — không dùng tủ thường
- Cảm biến đặt **ngoài tủ**, có vỏ bảo vệ riêng
- Dây tín hiệu đi xa dùng **dây lưới chống nhiễu**

---

## 9.2 Bước 1 — Chuẩn Bị Tủ Điện

```
1. Gắn DIN rail 35mm (2 thanh) vào tủ IP65
2. Lắp thứ tự từ trên xuống:
   - Aptomat tổng 2P 16A ELCB (trên cùng)
   - Aptomat nhánh 1P 16A (quạt + bơm)
   - Aptomat nhánh 1P 10A (đèn sưởi + chiếu sáng + rèm + cho ăn)
   - Aptomat nhánh 1P 6A  (bơm chính)
   - Aptomat nhánh 1P 6A  (nguồn DC + 24VAC)
   - Mean Well HDR-60-5 (5VDC)
   - Biến áp 24VAC 50VA
   - Contactor bơm NC1-0910
3. Bố trí terminal block:
   - Rail 1: GND chung, 5VDC, 3.3VDC
   - Rail 2: 24VAC L, 24VAC N (van nước)
   - Rail 3: 220VAC L quạt, 220VAC N chung
   - Rail 4: 220VAC L đèn/rèm/cho ăn
   - Rail 5: Tín hiệu GPIO (cảm biến)
4. Gắn 4 relay board 8 kênh (Board 1–4)
5. Dán nhãn tất cả terminal (tên kênh + số)
```

---

## 9.3 Bước 2 — Hàn Mạch Điều Khiển

```
Perfboard CB2S + MCP23017:
  1. Cắm 2 đế DIP-28 vào perfboard
  2. Gắn MCP23017 #1: A0=GND, A1=GND, A2=GND → 0x20
  3. Gắn MCP23017 #2: A0=VCC, A1=GND, A2=GND → 0x21
  4. Hàn tụ 100nF mỗi IC (giữa VDD và GND)
  5. Hàn R 10kΩ từ RESET mỗi IC lên 3.3V
  6. Hàn 2 điện trở pull-up 4.7kΩ (SDA→3.3V, SCL→3.3V)
  7. Hàn header pin cắm CB2S
  8. Hàn các header dây ra terminal GPIO

Lưu ý chuồng trại:
  - Phủ conformal coating lên PCB sau khi hàn xong
  - Bảo vệ chống ẩm, chống NH₃ ăn mòn
```

---

## 9.4 Bước 3 — Lắp Cảm Biến

```
SHT30 (Nhiệt độ + Độ ẩm) — lắp 3 điểm:
  - Vị trí 1: Đầu chuồng (gần cửa lấy khí)
  - Vị trí 2: Giữa chuồng (vùng vật nuôi)
  - Vị trí 3: Cuối chuồng (gần quạt hút)
  - Đặt ở độ cao 1–1.5m (ngang với vật nuôi)
  - Dùng hộp nhựa có lỗ thông khí để bảo vệ cảm biến

MQ-135 (NH₃):
  - Đặt ở độ cao 0.5–1m (nơi vật nuôi thở)
  - Tránh đặt gần cửa sổ hoặc quạt
  - Cần làm ấm 24–48h trước khi hiệu chỉnh ở ngoài trời

MH-Z19B (CO₂):
  - Đặt ở độ cao 1m
  - Tránh ánh nắng trực tiếp
  - Kết nối UART qua dây 4 chân

HC-SR04 (Mức cám):
  - Gắn trên nắp thùng cám, hướng xuống
  - Khoảng cách tối thiểu từ cảm biến đến mặt cám: 5cm
  - Tính % = (chiều cao thùng - khoảng cách đo) / chiều cao thùng × 100
```

---

## 9.5 Bước 4 — Lắp Quạt Thông Gió

```
Vị trí quạt hút gió (tunnel ventilation):
  - Đặt CUỐI chuồng (tường đối diện cửa lấy khí)
  - Chiều cao: 0.5–1m từ sàn
  - Khoảng cách đều nhau theo chiều ngang

Lắp quạt:
  1. Khoét lỗ tường theo kích thước quạt
  2. Gắn khung quạt vào tường, silicon xung quanh
  3. Lắp lưới bảo vệ phía ngoài (chống chim, chuột)
  4. Đấu dây 3×1.5mm² vào terminal tủ
  5. Luồn dây qua ống PVC Ø20

Kiểm tra:
  - Cánh quạt quay đúng chiều HÚT ra ngoài
  - Không có vật cản trước mặt quạt
```

---

## 9.6 Bước 5 — Lắp Hệ Thống Làm Mát

```
Phun sương:
  1. Kéo ống PE cao áp 6mm dọc chuồng
  2. Lắp đầu phun cách đều nhau 3m, hướng xuống
  3. Đầu ống nối vào bơm cao áp qua bộ lọc 5 micron
  4. Van 1 chiều tránh nước chảy ngược khi tắt bơm
  5. Bình tích áp 8L (giảm dao động áp suất)

Tấm làm mát cooling pad:
  1. Lắp tấm cellulose vào khung nhôm phía ĐẦU chuồng
     (đối diện quạt hút — không khí đi qua tấm ướt → lạnh)
  2. Bể chứa nước 100L phía dưới tấm
  3. Bơm chìm tuần hoàn nước từ bể lên máng phân phối
  4. Máng phân phối chạy dọc theo mép trên tấm
  5. Phao tự động bổ sung nước khi bể hạ mức
```

---

## 9.7 Bước 6 — Lắp Rèm Cửa

```
1. Gắn motor hộp số trên khung chuồng
2. Cuộn rèm/bạt vào thanh nhôm
3. Kết nối dây rèm với trục motor
4. Lắp công tắc hành trình IP65:
   - CT MỞ: vị trí rèm mở hoàn toàn
   - CT ĐÓNG: vị trí rèm đóng hoàn toàn
5. Đấu dây motor: 3×1.5mm² qua cầu chì 5A riêng mỗi chiều
6. Đấu dây CT hành trình: 2×0.5mm² về GPIO CB2S
7. Test: bật relay MỞ → rèm kéo lên → CT kích → relay tắt
```

---

## 9.8 Bước 7 — Lắp Máy Cho Ăn

```
Máy vít tải INOX:
  1. Đặt thùng chứa cám phía trên cao
  2. Lắp vít tải INOX nghiêng 30–45° xuống máng ăn
  3. Kết nối motor với đầu vít tải
  4. Gắn HC-SR04 trên nắp thùng cám đo mức
  5. Gắn load cell dưới thùng cám (tùy chọn)
  6. Đấu dây motor 220VAC qua relay K28/K29
  7. Test: chạy motor 5 giây → kiểm tra cám ra đều
```

---

## 9.9 Bước 8 — Upload Firmware & Cấu Hình

```
Cài đặt môi trường:
  1. Arduino IDE 2.x + LibreTuya board package
  2. Thư viện: PubSubClient, ArduinoJson, Adafruit MCP23017,
     ADS1X15, SHT30, HX711, NewPing, LiquidCrystal I2C

Cấu hình trong code:
  const char* MQTT_HOST = "mqtt.nextfarm.vn";
  const char* TOPIC_PREFIX = "livestock/";

Upload CB2S:
  TX↔RX, RX↔TX, GND↔GND, 3.3V↔3.3V (UART)
  BOOT → RST → thả RST → thả BOOT → Upload
  Serial 115200 baud kiểm tra: "I2C OK - 2 MCP23017"

Cấu hình WiFi:
  Lần đầu: AP "ChuongTrai_XXXXXX" + mật khẩu "12345678"
  Kết nối AP → vào 192.168.4.1 → nhập WiFi + MQTT
```

---

## 9.10 Kiểm Tra Trước Khi Vận Hành

```
□ Đo VOM: 220VAC → vỏ tủ ≥ 1MΩ
□ Đo VOM: 24VAC rail và 220VAC rail không chạm nhau
□ Serial: "MCP23017 OK", "WiFi OK", "MQTT OK", "NTP OK"
□ Test từng kênh:
  - K1–K6: Bật relay → nghe tiếng quạt chạy
  - K7: Bật relay → bơm phun sương chạy, kiểm tra đầu phun
  - K8: Bật relay → bơm tấm làm mát chạy
  - K9–K12: Bật relay → đèn sưởi sáng
  - K13,14,30,31: Bật relay → đèn chiếu sáng sáng
  - K16–K19: Bật relay → van nước mở (nghe tiếng click)
  - K20/21: Test rèm MỞ → test ĐÓNG (quan sát delay 500ms)
  - K28/K29: Bật relay 5 giây → máy cho ăn chạy
□ Cảm biến:
  - SHT30: Serial hiện nhiệt độ + độ ẩm hợp lý
  - MQ-135: Giá trị NH₃ < 10ppm trong không khí sạch
  - HC-SR04: Đo khoảng cách thùng cám đúng
□ LCD hiển thị đúng trạng thái
□ MQTT: Nhận được livestcok/{id}/sensor trên app
□ Test cảnh báo: Che MQ-135 lại → xem app nhận alert NH₃
□ Test phục hồi mất điện: Ngắt nguồn 30 giây → cắm lại
```

---

*[← Tự Động Hóa](08_tu-dong-hoa.md) | [Vận Hành →](10_van-hanh.md)*
