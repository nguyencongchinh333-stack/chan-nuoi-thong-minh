# 07 — MQTT API

---

## 7.1 Cấu Hình Kết Nối

| Thông số | Giá trị |
|----------|---------|
| Broker | `mqtt.nextfarm.vn` |
| Topic gốc | `livestock/{deviceID}/` |
| Device ID | MAC address WiFi (tự động) |
| LWT | `livestock/{id}/status` → `"offline"` |

---

## 7.2 Sơ Đồ Topics

```
livestock/{deviceID}/
├── status              ← online / offline
├── trang_thai          ← Trạng thái hệ thống
├── sensor              ← Nhiệt độ, độ ẩm, NH₃, CO₂
├── can                 ← Trọng lượng vật nuôi
├── cam                 ← Mức thức ăn %
├── nuoc                ← Lưu lượng nước uống
├── che_do              ← tu_dong / thu_cong
│
├── alert/
│   ├── nh3             ← Cảnh báo NH₃
│   ├── nhiet           ← Cảnh báo nhiệt độ
│   ├── cam             ← Cảnh báo mức cám
│   └── dien            ← Mất điện / phục hồi
│
└── [Nhận lệnh]:
    ├── che_do/set
    ├── quat/{i}/lenh
    ├── quat/tat_ca/lenh
    ├── quat/nguong/set
    ├── suoi/set
    ├── den/lich/set
    ├── cho_an/lich/set
    ├── cho_an/lenh
    ├── rem/{i}/lenh
    ├── van_nuoc/{i}/lenh
    └── dung
```

---

## 7.3 Trạng Thái Hệ Thống

**Topic:** `livestock/{id}/trang_thai` — publish mỗi 5 giây

```json
{
  "che_do": "tu_dong",
  "quat_dang_chay": 4,
  "tong_quat": 6,
  "suoi": false,
  "lam_mat": false,
  "den": true,
  "rem": ["mo","mo","dong","dong"],
  "cho_an_tiep": "16:00"
}
```

---

## 7.4 Dữ Liệu Cảm Biến

**Topic:** `livestock/{id}/sensor` — publish mỗi 30 giây

```json
{
  "nhiet_do": [28.5, 29.1, 27.8],
  "do_am": [72, 74, 70],
  "nhiet_do_tb": 28.5,
  "nh3_ppm": 12,
  "co2_ppm": 1850,
  "timestamp": 1747123456
}
```

**Topic:** `livestock/{id}/cam` — publish mỗi 60 giây

```json
{
  "may1_phan_tram": 85,
  "may2_phan_tram": 72,
  "canh_bao": false
}
```

---

## 7.5 Cảnh Báo

**Topic:** `livestock/{id}/alert/nh3`

```json
{
  "gia_tri": 28,
  "nguong": 25,
  "muc": "canh_bao",
  "hanh_dong": "bat_tat_ca_quat",
  "thoi_gian": "2026-05-12T10:30:00"
}
```

**Topic:** `livestock/{id}/alert/cam`

```json
{
  "may": 1,
  "phan_tram": 8,
  "muc": "sap_het",
  "thoi_gian": "2026-05-12T14:00:00"
}
```

---

## 7.6 Cài Đặt Ngưỡng Quạt

**Topic:** `livestock/{id}/quat/nguong/set`

```json
{
  "t0": 22,
  "t1": 26,
  "t2": 28,
  "t3": 30,
  "t4": 33,
  "t5": 35,
  "nh3_bat_het": 25
}
```

---

## 7.7 Cài Đặt Sưởi

**Topic:** `livestock/{id}/suoi/set`

```json
{
  "loai": "ga",
  "tuan": 2,
  "nhiet_do_muc_tieu": 32,
  "hysteresis": 1.0,
  "suoi_bao_gom": [1, 2, 3, 4]
}
```

---

## 7.8 Cài Đặt Chiếu Sáng

**Topic:** `livestock/{id}/den/lich/set`

```json
{
  "loai": "ga_de",
  "gio_bat": 5,
  "phut_bat": 0,
  "gio_tat": 21,
  "phut_tat": 0,
  "khu": [true, true, true, true]
}
```

---

## 7.9 Cài Đặt Cho Ăn

**Topic:** `livestock/{id}/cho_an/lich/set`

```json
{
  "lich": [
    {"gio": 6, "phut": 0, "giay_chay": 30, "may": [1, 2]},
    {"gio": 11, "phut": 0, "giay_chay": 30, "may": [1, 2]},
    {"gio": 16, "phut": 0, "giay_chay": 30, "may": [1, 2]}
  ]
}
```

**Cho ăn thủ công ngay:**

**Topic:** `livestock/{id}/cho_an/lenh`

```json
{"may": 1, "giay": 30}
```

---

## 7.10 Điều Khiển Rèm Cửa

**Topic:** `livestock/{id}/rem/{1-4}/lenh`

Payload: `"mo"` / `"dong"` / `"dung"`

---

## 7.11 Mã Lỗi

| Mã | Ý nghĩa |
|----|---------|
| `DANG_TU_DONG` | Gửi lệnh tay khi đang AUTO |
| `NH3_CANH_BAO` | NH₃ > ngưỡng cảnh báo |
| `NH3_NGUY_HIEM` | NH₃ > ngưỡng nguy hiểm |
| `NHIET_DO_CAO` | Nhiệt độ vượt mức nguy hiểm |
| `CAM_HET` | Mức cám < 5% |
| `MCP_LOI` | MCP23017 không phản hồi |

---

*[← Firmware](06_firmware.md) | [Tự Động Hóa →](08_tu-dong-hoa.md)*
