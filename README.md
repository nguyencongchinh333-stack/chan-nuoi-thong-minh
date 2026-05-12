# NextFarm DIY Controller — Chăn Nuôi Thông Minh v1.0
### Hệ Thống Điều Khiển Chuồng Trại Tự Động — Tích Hợp NextFarm Platform

---

## Dự Án Này Là Gì

Bộ điều khiển phần cứng **tự lắp (DIY)** cho chuồng trại chăn nuôi sử dụng chip Tuya CB2S (BK7231N) + 2× MCP23017, điều khiển **31 relay** cho toàn bộ thiết bị: quạt thông gió, hệ thống làm mát, sưởi ấm, chiếu sáng, nước uống, rèm cửa, máy cho ăn — kết nối **NextFarm Platform** qua MQTT.

---

## Vị Trí Trong NextFarm Ecosystem

```
NextFarm Platform — IoT → Livestock
│
├── Tuya (CB2S) ← Dự án này
├── Shelly (MQTT local, offline)
└── Nextfarm Pro Livestock
```

---

## Tóm Tắt Hệ Thống

| Thông số | Giá trị |
|----------|---------|
| Vi điều khiển | CB2S (BK7231N) — Tuya chip, LibreTuya |
| Kênh relay | 31 kênh (32 vật lý) |
| Phù hợp | Chuồng gà / heo / bò 200–1000m² |
| Nguồn | 5VDC (logic) + 24VAC (van) + 220VAC (quạt/bơm/đèn/motor) |
| Chế độ | AUTO (tự động) / THỦ CÔNG |

## Thiết Bị Điều Khiển

| Thiết bị | SL | Kênh | Điện áp |
|----------|----|:----:|---------|
| Bơm chính | 1 | 1 | 220VAC |
| Quạt thông gió | 6 | 6 | 220VAC |
| Bơm phun sương làm mát | 1 | 1 | 220VAC |
| Bơm tấm làm mát | 1 | 1 | 220VAC |
| Đèn sưởi hồng ngoại | 4 | 4 | 220VAC |
| Đèn chiếu sáng | 4 | 4 | 220VAC |
| Van nước uống | 4 | 4 | 24VAC |
| Rèm cửa (2 chiều MỞ/ĐÓNG) | 4 | 8 | 220VAC |
| Máy cho ăn tự động | 2 | 2 | 220VAC |
| **Tổng** | **27** | **31** | |

---

## Mục Lục Tài Liệu

| # | File | Nội dung |
|---|------|---------|
| 01 | [Tổng Quan & Platform](01_tong-quan.md) | NextFarm Livestock, định vị, so sánh |
| 02 | [Kiến Trúc Hệ Thống](02_kien-truc.md) | Sơ đồ khối, LCD cảnh báo, luồng điều khiển |
| 03 | [Phần Cứng & Thông Số](03_phan-cung.md) | Thông số kỹ thuật thiết bị điều khiển + actuator |
| 04 | [Bản Đồ Relay](04_relay-channel-map.md) | 31 kênh relay chi tiết |
| 05 | [Đấu Nối Điện](05_dau-noi-dien.md) | Sơ đồ đấu nối từng thiết bị |
| 06 | [Firmware](06_firmware.md) | Code đầy đủ CB2S LibreTuya |
| 07 | [MQTT API](07_mqtt-api.md) | Giao thức NextFarm Livestock |
| 08 | [Tự Động Hóa](08_tu-dong-hoa.md) | Logic điều khiển, ngưỡng, lịch, cảnh báo LCD |
| 09 | [Phục Hồi Mất Điện](09_phuc-hoi-mat-dien.md) | UPS, ưu tiên phục hồi, ca ăn bị lỡ |
| 10 | [Lắp Đặt](10_lap-dat.md) | 9 bước lắp đặt chuồng trại + checklist |
| 11 | [Vận Hành & Debug](10_van-hanh.md) | Đọc LCD, lệnh MQTT, xử lý sự cố, bảo trì |
| 12 | [Báo Giá Chi Tiết](11_bang-gia.md) | Giá TL + TT từng kênh, 4 gói |
| 13 | [Lịch Chăn Nuôi](12_lich-chan-nuoi.md) | Lịch theo tuần (gà thịt, gà đẻ, heo) + nhật ký |
| 14 | [Tính Toán Thiết Kế](13_tinh-toan-thiet-ke.md) | Số quạt, công suất sưởi, tấm làm mát, chiếu sáng |

---

## Chi Phí Tham Khảo (Gói 3 Khuyến Nghị)

| Phần | Giá TL | Giá TT |
|------|-------:|-------:|
| Bộ điều khiển | 2.585.000đ | 3.254.000đ |
| Bộ động lực 31 kênh | 31.400.000đ | 36.218.000đ |
| Cảm biến + màn hình | 960.000đ | 1.117.000đ |
| **Tổng phần cứng** | **34.945.000đ** | **40.589.000đ** |

---

*Phiên bản: 1.0 | Cập nhật: 2026-05-12*
