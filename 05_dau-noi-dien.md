# 05 — Đấu Nối Điện

---

## 5.1 Sơ Đồ Tổng Quan Nguồn

```
220VAC (Lưới điện)
       │
  [Aptomat 2P 16A ELCB 30mA]  ← Bắt buộc
       │
  ┌────┼─────────────────────────────────┐
  │    │                                 │
  ▼    ▼                                 ▼
[CB  [Mean Well      [Biến áp      [Aptomat 1P]
1P]   HDR-60-5]      24VAC 50VA]   16A + 10A + 6A
Bơm  5VDC/12A        24VAC         Quạt/Đèn/Rèm
  │       │             │
  ▼       ▼             ▼
Bơm  CB2S+MCP23017  Van nước
chính +Relay coil   uống ×4
```

---

## 5.2 CB2S ↔ MCP23017

```
CB2S GPIO0 (SDA) ──[4.7kΩ→3.3V]── MCP23017 #1 SDA ── MCP23017 #2 SDA
CB2S GPIO1 (SCL) ──[4.7kΩ→3.3V]── MCP23017 #1 SCL ── MCP23017 #2 SCL
3.3V ──────────────────────────── VDD cả 2 chip
GND  ──────────────────────────── GND cả 2 chip

MCP23017 #1: A0=GND, A1=GND, A2=GND → 0x20
MCP23017 #2: A0=VCC, A1=GND, A2=GND → 0x21
RESET cả 2: nối 3.3V qua 10kΩ (bắt buộc)
```

---

## 5.3 MCP23017 ↔ Relay Board

```
MCP23017 #1 GPA0–GPA7 → Relay Board 1 IN1–IN8  (kênh 0–7)
MCP23017 #1 GPB0–GPB7 → Relay Board 2 IN1–IN8  (kênh 8–15)
MCP23017 #2 GPA0–GPA7 → Relay Board 3 IN1–IN8  (kênh 16–23)
MCP23017 #2 GPB0–GPB7 → Relay Board 4 IN1–IN8  (kênh 24–31)

5VDC → VCC tất cả 4 board
GND  → GND tất cả 4 board
```

---

## 5.4 Đấu Nối Cảm Biến

```
SHT30 (Nhiệt độ + Độ ẩm):
  VCC → 3.3V  │  GND → GND
  SDA → GPIO0 │  SCL → GPIO1
  Địa chỉ: 0x44 (mặc định)
  Lắp 3 điểm: đầu / giữa / cuối chuồng

MQ-135 (NH₃ Amoniac):
  VCC → 5V  │  GND → GND
  AOUT → ADS1115 AIN0 (analog)
  Lưu ý: Cần làm ấm 24–48h trước khi hiệu chỉnh

MH-Z19B (CO₂):
  VCC → 5V  │  GND → GND
  TX → GPIO5 (UART RX của CB2S)
  RX → GPIO4 (UART TX của CB2S)

ADS1115 (ADC 16-bit):
  VCC → 3.3V  │  GND → GND
  SDA → GPIO0 │  SCL → GPIO1
  Địa chỉ: 0x48 (ADDR = GND)
  AIN0 ← MQ-135 AOUT
  AIN1 ← Cảm biến áp suất nước
  AIN2 ← Dự phòng
  AIN3 ← Dự phòng

HC-SR04 (Mức thức ăn):
  VCC → 5V  │  GND → GND
  TRIG → GPIO13
  ECHO → GPIO12 (qua phân áp 5V→3.3V: R1=1kΩ, R2=2kΩ)

HX711 (Cân load cell):
  VCC → 3.3V  │  GND → GND
  DT  → GPIO11
  SCK → GPIO10

DS18B20 (Nhiệt độ dự phòng):
  VCC → 3.3V  │  GND → GND
  DATA → GPIO8 + [4.7kΩ → 3.3V]

LCD 20×4 I2C:
  VCC → 5V  │  GND → GND
  SDA → GPIO0 │  SCL → GPIO1
  Địa chỉ: 0x27
```

---

## 5.5 Van Nước Uống 24VAC (Kênh 16–19)

```
Biến áp 24VAC:
  L (24V) → Busbar L chung
  N (0V)  → Busbar N chung

Mỗi van (ví dụ kênh 16):
  Relay K16 COM → Busbar L 24V
  Relay K16 NO  → Van N1 dây 1
  Van N1 dây 2  → Busbar N 24V

Van loại NC → khi relay bật = van mở
              khi mất điện  = van đóng (an toàn)
```

---

## 5.6 Quạt Thông Gió 220VAC (Kênh 1–6)

```
Aptomat 1P 16A → Busbar L quạt

Mỗi quạt:
  Relay KQ1 COM → Busbar L quạt
  Relay KQ1 NO  → Quạt L
  Quạt N        → Busbar N 220V
  Quạt PE       → Tiếp địa

Lưu ý: 6 quạt chạy cùng lúc tối đa ~12A (quạt 50cm 2A/cái)
→ Cần aptomat 16A, dây tải 2.5mm²
```

---

## 5.7 Bơm Phun Sương & Tấm Làm Mát (Kênh 7–8)

```
Bơm phun sương (K7):
  Relay K7 COM → Aptomat 1P 6A → 220VAC L
  Relay K7 NO  → Bơm L
  Bơm N → 220V N  │  Bơm PE → Tiếp địa

Bơm tấm làm mát (K8):
  Relay K8 COM → 220VAC L
  Relay K8 NO  → Bơm L
  Bơm N → 220V N
```

---

## 5.8 Rèm Cửa 2 Chiều (Kênh 20–27)

```
Rèm cửa 1 (kênh 20, 21):
  Relay K20 (MỞ):
    COM → Cầu chì 5A → 220VAC L
    NO  → Motor rèm 1 dây MỞ

  Relay K21 (ĐÓNG):
    COM → Cầu chì 5A riêng → 220VAC L
    NO  → Motor rèm 1 dây ĐÓNG

  Motor N  → 220V N chung
  Motor PE → Tiếp địa

  Công tắc hành trình (nếu có):
    CT MỞ:  nối song song với GPIO — khi kích → dừng motor
    CT ĐÓNG: nối song song với GPIO — khi kích → dừng motor
```

---

## 5.9 Đèn Sưởi (Kênh 9–12)

```
Aptomat 1P 10A → Busbar L đèn sưởi

Mỗi đèn sưởi:
  Relay K9 COM → Busbar L đèn sưởi
  Relay K9 NO  → Đui đèn → Bóng 250W
  Đui trung tính → Busbar N 220V

Lưu ý: 4 đèn × 250W = 1000W = 4.5A → Aptomat 10A là đủ
```

---

## 5.10 Máy Cho Ăn (Kênh 28–29)

```
Motor 220VAC vít tải:
  Relay K28 COM → Aptomat 1P 6A → 220VAC L
  Relay K28 NO  → Motor L
  Motor N  → 220V N
  Motor PE → Tiếp địa
```

---

## 5.11 Lưu Ý An Toàn

> ⚠️ NGUY HIỂM — Điện 220VAC có thể gây chết người

```
✅ Ngắt aptomat trước khi đấu dây
✅ ELCB 30mA ở đầu tổng — bắt buộc
✅ Tiếp địa toàn bộ vỏ kim loại, bơm, tủ
✅ Dây 220VAC ≥1.5mm², quạt/bơm dùng 2.5mm²
✅ Rèm cửa: cầu chì 5A riêng mỗi chiều
✅ Đo VOM kiểm tra cách điện trước khi cấp điện
✅ Môi trường chuồng ẩm ướt: tất cả đấu nối phải bọc kín, dùng co nhiệt
```

---

*[← Relay Map](04_relay-channel-map.md) | [Firmware →](06_firmware.md)*
