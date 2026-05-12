# 09 — Phục Hồi Sau Mất Điện

---

## 9.1 Vấn Đề Đặc Thù Chăn Nuôi

Mất điện trong chuồng trại nguy hiểm hơn nhà kính vì:

| Tình huống | Hậu quả nếu không phục hồi đúng |
|-----------|--------------------------------|
| Quạt tắt giữa trưa nắng | NH₃ tích tụ, nhiệt độ tăng → gà/heo stress, chết hàng loạt |
| Đèn sưởi tắt ban đêm | Gà con, heo con bị lạnh → tiêu chảy, chết |
| Máy cho ăn bỏ ca | Vật nuôi đói → giảm tăng trọng |
| Van nước tắt | Mất nước uống → stress nhiệt |

---

## 9.2 Giải Pháp

Lưu toàn bộ trạng thái vào **flash (Preferences)** sau mỗi thay đổi. Khi khởi động lại:
1. Đọc trạng thái từ flash
2. Kết nối WiFi + đồng bộ NTP
3. **Ưu tiên 1**: Bật quạt thông gió ngay (không chờ NTP — an toàn NH₃)
4. **Ưu tiên 2**: Tính toán trạng thái cần phục hồi (sưởi, đèn, cho ăn)

---

## 9.3 Dữ Liệu Lưu Flash

```cpp
struct TrangThaiLuu {
  // Quạt — ưu tiên phục hồi ngay
  int  soQuatDangChay;       // 0–6
  bool bomPhunDangBat;
  bool bomTamDangBat;

  // Sưởi
  bool suoiDangBat;
  bool suoiBaoGom[4];

  // Đèn chiếu sáng
  bool denDangBat;

  // Cho ăn — theo dõi ca bị lỡ
  uint32_t thoiGianTatDien;  // Unix timestamp lúc mất điện
  bool     caChoAnDaChay[3]; // 3 ca trong ngày đã chạy chưa

  // Rèm cửa
  uint8_t trangThaiRem[4];   // 0=dừng, 1=mở, 2=đóng
};
```

---

## 9.4 Luồng Phục Hồi Khi Khởi Động

```
Khởi động CB2S
      │
      ▼
Đọc TrangThaiLuu từ flash
      │
      ├─── NGAY LẬP TỨC (không chờ NTP):
      │    Bật lại số quạt như trước mất điện
      │    → Đảm bảo thông gió, tránh NH₃ tích tụ
      │
      ▼
Kết nối WiFi + NTP
      │
      ▼
Tính thời gian mất điện:
  elapsed = unixNow - thoiGianTatDien
      │
      ├─── Sưởi:
      │    Đọc nhiệt độ hiện tại → tự điều chỉnh theo cfgSuoi
      │    (không cần biết trước bật hay tắt)
      │
      ├─── Đèn chiếu sáng:
      │    Tính theo lịch quang chu kỳ + giờ hiện tại
      │    Bật/tắt đúng theo lịch
      │
      ├─── Cho ăn:
      │    Kiểm tra ca nào bị lỡ trong thời gian mất điện
      │    Nếu lỡ ca sáng hoặc trưa → chạy bù ngay
      │    Nếu lỡ ca chiều (>18h) → bỏ qua, không chạy bù
      │
      └─── Rèm cửa:
           Phục hồi vị trí → tự điều chỉnh theo nhiệt độ hiện tại
      │
      ▼
Publish MQTT: livestock/{id}/alert/dien
  {"su_kien":"phuc_hoi","thoi_gian_mat_phut": X}
      │
      ▼
Vận hành bình thường
```

---

## 9.5 Xử Lý Ca Cho Ăn Bị Lỡ

```
Ví dụ: Mất điện lúc 10:30, có điện lại lúc 13:00
  - Ca 06:00: Đã chạy trước khi mất điện → OK
  - Ca 11:00: BỊ LỠ (mất điện lúc 10:30)
             → Khi phục hồi: Chạy bù ngay lập tức
  - Ca 16:00: Chưa đến giờ → Sẽ chạy bình thường

Ví dụ: Mất điện lúc 15:30, có điện lại lúc 20:00
  - Ca 16:00: BỊ LỠ → Đã quá 18:00 → KHÔNG chạy bù
             (vật nuôi không ăn muộn quá buổi tối)
  - Ca 06:00 hôm sau → Chạy bình thường
```

---

## 9.6 Mất Điện Ngắn (<5 Phút)

```
Nếu thời gian mất điện < 5 phút:
  - Quạt: Bật lại như cũ
  - Sưởi: Đọc nhiệt độ, tự điều chỉnh
  - Đèn: Theo lịch hiện tại
  - Cho ăn: Không chạy bù (ca không bị lỡ)
  - Rèm: Giữ nguyên vị trí trước
  → Phục hồi trong vòng 30 giây sau khi có điện
```

---

## 9.7 Ưu Tiên Phục Hồi (Không Chờ NTP)

> **Quan trọng với chăn nuôi**: Nếu không kết nối được NTP (mất Internet), hệ thống VẪN phục hồi thông gió và sưởi — không bỏ vật nuôi trong điều kiện nguy hiểm.

```
Không có NTP:
  ✅ Quạt: Bật theo nhiệt độ đọc được (không cần NTP)
  ✅ Sưởi: Bật theo nhiệt độ đọc được (không cần NTP)
  ❌ Đèn chiếu sáng: Không thể tính quang chu kỳ → Giữ trạng thái trước
  ❌ Cho ăn: Không biết giờ → Không chạy bù, chờ đủ giờ khi NTP sync
  ✅ Rèm: Điều chỉnh theo nhiệt độ (không cần NTP)
```

---

## 9.8 Lưu Ý Đặc Biệt

| Tình huống | Xử lý |
|-----------|-------|
| Mất điện khi đang chạy máy cho ăn | Motor dừng ngay, không chạy bù (thức ăn đã ra một phần) |
| Mất điện khi rèm đang mở dở | Rèm giữ nguyên vị trí (không cơ cấu đóng tự động) |
| Mất điện kéo dài >4h vào ban đêm mùa đông | Gà con có thể chết lạnh — cần nguồn UPS cho đèn sưởi |
| Flash bị hỏng | Khởi động bình thường, bật quạt ngay, log lỗi MQTT |

---

## 9.9 Khuyến Nghị UPS Cho Chuồng Trại

| Thiết bị | Cần UPS? | Lý do |
|---------|:--------:|-------|
| CB2S + MCP23017 + Relay | ✅ Bắt buộc | 5VDC ~15W — UPS mini là đủ |
| Đèn sưởi gà con/heo con | ✅ Khuyến nghị | Tắt đèn sưởi 1h đêm lạnh = hao vật nuôi |
| Quạt thông gió | ⚠️ Tùy | Mất điện <30 phút ít ảnh hưởng |
| Bơm phun sương | ❌ Không cần | |
| Máy cho ăn | ❌ Không cần | |

**UPS khuyến nghị cho bộ điều khiển:**
- Công suất: 500VA / 300W
- Thời gian dự phòng: 2–4 giờ cho tải 20W (CB2S + relay)
- Giá: ~800.000 – 1.200.000đ

---

*[← Tự Động Hóa](08_tu-dong-hoa.md) | [Lắp Đặt →](10_lap-dat.md)*
