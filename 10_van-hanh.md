# 10 — Vận Hành & Xử Lý Sự Cố

---

## 10.1 Khởi Động Hàng Ngày

```
1. Kiểm tra màn hình LCD:
   - Nhiệt độ hợp lý với giờ trong ngày
   - NH₃ < 10ppm (bình thường)
   - Số quạt đang chạy phù hợp nhiệt độ
   - % cám còn đủ cho ngày hôm nay

2. Kiểm tra app NextFarm:
   - Thiết bị hiển thị "online"
   - Không có cảnh báo tồn đọng
   - Lịch cho ăn hôm nay đúng

3. Kiểm tra thực tế chuồng:
   - Quạt quay đúng chiều, không rung lắc
   - Nước uống chảy đều các khu
   - Thức ăn trong máng đầy đủ
```

---

## 10.2 Đọc Màn Hình LCD

```
Màn hình bình thường:
  AUTO  NH3:08p  28.5C    ← Chế độ AUTO, NH₃=8ppm, 28.5°C
  Quat: 4/6  DoAm:72%     ← 4/6 quạt đang chạy, độ ẩm 72%
  Den:ON  Suoi:OFF        ← Đèn chiếu sáng bật, sưởi tắt
  ChoAn:16:00  Cam: 85%   ← Ca cho ăn tiếp 16h, cám còn 85%

Màn hình cảnh báo (nhấp nháy):
  !!! CANH BAO !!!        ← Nền nhấp nháy
  NH3: 28ppm > 25ppm      ← NH₃ vượt ngưỡng
  Da bat toan bo quat     ← Hành động đã thực hiện
  Kiem tra thong gio!     ← Hướng dẫn xử lý

Màn hình lỗi nghiêm trọng (nền đỏ liên tục):
  [!!!] LOI HE THONG [!!!]
  NH3: 55ppm NGUY HIEM
  Goi ngay: 090 xxx xxxx
  Mat ket noi MQTT        ← Thêm thông tin lỗi
```

---

## 10.3 Điều Khiển Thủ Công Qua MQTT

> Chuyển sang THỦ CÔNG trước khi điều khiển tay

```bash
# Chuyển sang THỦ CÔNG
mosquitto_pub -t "livestock/{id}/che_do/set" -m '{"che_do":"thu_cong"}'

# Bật quạt 1
mosquitto_pub -t "livestock/{id}/quat/1/lenh" -m "ON"

# Bật tất cả quạt khẩn cấp
mosquitto_pub -t "livestock/{id}/quat/tat_ca/lenh" -m "ON"

# Bật đèn sưởi 1+2
mosquitto_pub -t "livestock/{id}/suoi/set" -m '{"suoi":[1,2],"trang_thai":"ON"}'

# Chạy máy cho ăn 1 trong 30 giây
mosquitto_pub -t "livestock/{id}/cho_an/lenh" -m '{"may":1,"giay":30}'

# Mở rèm cửa 1
mosquitto_pub -t "livestock/{id}/rem/1/lenh" -m "mo"

# Dừng khẩn cấp tất cả
mosquitto_pub -t "livestock/{id}/dung" -m "1"

# Quay lại AUTO
mosquitto_pub -t "livestock/{id}/che_do/set" -m '{"che_do":"tu_dong"}'
```

---

## 10.4 Cài Đặt Thông Số Tự Động

```bash
# Cài ngưỡng nhiệt độ bật quạt (6 cấp)
mosquitto_pub -t "livestock/{id}/quat/nguong/set" -m '{
  "t0":22,"t1":26,"t2":28,"t3":30,"t4":33,"t5":35}'

# Cài nhiệt độ mục tiêu sưởi (tuần 2 gà)
mosquitto_pub -t "livestock/{id}/suoi/set" -m '{
  "loai":"ga","tuan":2,"nhiet_do_muc_tieu":32}'

# Cài lịch cho ăn
mosquitto_pub -t "livestock/{id}/cho_an/lich/set" -m '{
  "lich":[
    {"gio":6,"phut":0,"giay_chay":30},
    {"gio":11,"phut":0,"giay_chay":30},
    {"gio":16,"phut":0,"giay_chay":30}
  ]}'

# Cài lịch chiếu sáng (gà đẻ)
mosquitto_pub -t "livestock/{id}/den/lich/set" -m '{
  "gio_bat":5,"phut_bat":0,"gio_tat":21,"phut_tat":0}'

# Cài ngưỡng cảnh báo NH₃
mosquitto_pub -t "livestock/{id}/camBien/nh3/nguong/set" -m '{
  "canh_bao":25,"nguy_hiem":50}'
```

---

## 10.5 Theo Dõi Qua App

```bash
# Xem trạng thái tổng hợp
mosquitto_sub -t "livestock/{id}/trang_thai"

# Xem dữ liệu cảm biến
mosquitto_sub -t "livestock/{id}/sensor"

# Xem tất cả cảnh báo
mosquitto_sub -t "livestock/{id}/alert/#"

# Xem trạng thái cân
mosquitto_sub -t "livestock/{id}/can"

# Theo dõi mức cám
mosquitto_sub -t "livestock/{id}/cam"
```

---

## 10.6 Xử Lý Sự Cố

### Quạt Không Chạy Khi NH₃ Cao

| Triệu chứng | Nguyên nhân | Cách xử lý |
|------------|------------|------------|
| LCD hiện NH₃ cao nhưng quạt không bật | Firmware bị treo | Reset thiết bị, kiểm tra watchdog |
| Relay click nhưng quạt không chạy | Cầu chì quạt đứt | Thay cầu chì, kiểm tra aptomat |
| NH₃ cao nhưng cảm biến báo thấp | Cảm biến MQ-135 lỗi | Hiệu chỉnh lại ngoài trời |

### Nhiệt Độ Không Chính Xác

| Triệu chứng | Nguyên nhân | Cách xử lý |
|------------|------------|------------|
| 3 điểm SHT30 chênh nhau >5°C | Bình thường do gradient | Dùng giá trị trung bình hoặc cao nhất |
| SHT30 báo sai lệch nhiều | Cảm biến bị bụi cám | Vệ sinh lỗ thông khí của hộp cảm biến |
| Giá trị dao động liên tục | Nhiễu I2C | Kiểm tra pull-up 4.7kΩ, rút ngắn dây |

### Rèm Cửa Không Hoạt Động

| Triệu chứng | Nguyên nhân | Cách xử lý |
|------------|------------|------------|
| Motor không chạy | Cầu chì 5A đứt | Thay cầu chì |
| Rèm chạy không dừng | CT hành trình hỏng | Thay công tắc hành trình |
| Rèm chạy ngược chiều | Đấu nhầm MỞ/ĐÓNG | Hoán đổi dây relay K_MO và K_DONG |

### Máy Cho Ăn Không Chạy

| Triệu chứng | Nguyên nhân | Cách xử lý |
|------------|------------|------------|
| Motor không khởi động | Cám đông cứng, motor kẹt | Tháo vít tải, thông tắc |
| Chạy nhưng cám không ra | Vít tải bị nghẹt | Tháo thông, giảm lượng cám mỗi ca |
| Cảm biến mức cám sai | HC-SR04 bị bụi | Lau sạch mặt cảm biến |

### LCD Không Hiển Thị

| Triệu chứng | Nguyên nhân | Cách xử lý |
|------------|------------|------------|
| LCD tối | Mất nguồn 5V | Kiểm tra nguồn Mean Well |
| LCD sáng nhưng không chữ | I2C lỗi, địa chỉ sai | Kiểm tra địa chỉ 0x27, pull-up |
| LCD hiện ký tự lạ | Khởi tạo sai | Reset thiết bị |

---

## 10.7 Bảo Trì Định Kỳ

| Tần suất | Công việc |
|----------|-----------|
| **Hàng ngày** | Kiểm tra LCD, mức cám, nước uống |
| **Hàng tuần** | Lau sạch lưới quạt (bụi cám bám nhanh), vệ sinh đầu phun sương |
| **Hàng tháng** | Kiểm tra xiết terminal, kiểm tra đấu nối rèm cửa |
| **Mỗi lứa nuôi** | Vệ sinh toàn bộ chuồng, kiểm tra cảm biến, hiệu chỉnh MQ-135 |
| **6 tháng** | Kiểm tra dây điện, thay lọc Y-strainer van nước, bảo dưỡng motor |

---

## 10.8 Mã Lỗi MQTT

| Mã lỗi | Ý nghĩa | Cách xử lý |
|--------|---------|-----------|
| `DANG_TU_DONG` | Gửi lệnh tay khi đang AUTO | Chuyển sang THỦ CÔNG |
| `NH3_CANH_BAO` | NH₃ > 25ppm | Kiểm tra thông gió |
| `NH3_NGUY_HIEM` | NH₃ > 50ppm | Kiểm tra ngay, gọi kỹ thuật |
| `NHIET_DO_CAO` | Nhiệt độ vượt ngưỡng nguy hiểm | Kiểm tra làm mát |
| `CAM_HET` | Mức cám < 5% | Bổ sung thức ăn ngay |
| `MCP_LOI` | MCP23017 không phản hồi | Kiểm tra I2C, pull-up |
| `WIFI_MAT` | Mất kết nối WiFi | Kiểm tra router |
| `NTP_LOI` | Không đồng bộ thời gian | Kiểm tra Internet |

---

*[← Lắp Đặt](09_lap-dat.md) | [Báo Giá →](11_bang-gia.md)*
