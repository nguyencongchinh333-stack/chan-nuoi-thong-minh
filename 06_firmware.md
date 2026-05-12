# 06 — Firmware Đầy Đủ — chan-nuoi.ino

---

## 6.1 Môi Trường Biên Dịch

```bash
# Arduino IDE 2.x + LibreTuya board package
# URL: https://github.com/libretiny-eu/libretiny/releases/latest/download/package_libretiny_index.json
# Board: BK7231N (CB2S)

# Thư viện cần cài:
arduino-cli lib install \
  "WiFiManager" "PubSubClient" "ArduinoJson" \
  "Adafruit MCP23017 Arduino Library" "Adafruit ADS1X15" \
  "UNIT_SHT30" "HX711 Arduino Library" \
  "NewPing" "LiquidCrystal I2C"
```

---

## 6.2 Cấu Hình Trước Khi Upload

```cpp
// Chỉnh sửa trước khi upload:
const char* MQTT_HOST  = "mqtt.nextfarm.vn";
const int   MQTT_PORT  = 1883;
const char* MQTT_USER  = "device";
const char* MQTT_PASS  = "secret";
const char* TZ_VN      = "ICT-7";
const char* NTP_SERVER = "pool.ntp.org";
```

---

## 6.3 Code Đầy Đủ

```cpp
// ════════════════════════════════════════════════════════════════
// HỆ THỐNG CHĂN NUÔI THÔNG MINH — NextFarm v1.0
// Chip: CB2S (BK7231N) | Framework: LibreTuya + Arduino
// Hardware: 2× MCP23017 + 31 relay
//
// Tính năng:
//   - 6 quạt thông gió (6 cấp theo nhiệt độ + NH₃)
//   - Hệ thống làm mát (phun sương + tấm cooling pad)
//   - 4 đèn sưởi (theo nhiệt độ mục tiêu + tuần tuổi)
//   - 4 đèn chiếu sáng (quang chu kỳ)
//   - 4 van nước uống 24VAC
//   - 4 rèm cửa 2 chiều
//   - 2 máy cho ăn tự động
//   - Cảm biến: SHT30 ×3, NH₃, CO₂, cân, mức cám
//   - LCD 20×4 hiển thị trạng thái + cảnh báo
//   - MQTT → NextFarm Livestock Platform
// ════════════════════════════════════════════════════════════════

#include <WiFi.h>
#include <WiFiManager.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include <Preferences.h>
#include <Wire.h>
#include <Adafruit_MCP23X17.h>
#include <Adafruit_ADS1X15.h>
#include <SoftwareSerial.h>
#include <HX711.h>
#include <NewPing.h>
#include <LiquidCrystal_I2C.h>
#include <esp_task_wdt.h>
#include <time.h>

// ── Cấu hình server ──────────────────────────────────────────────
const char* MQTT_HOST  = "mqtt.nextfarm.vn";
const int   MQTT_PORT  = 1883;
const char* MQTT_USER  = "device";
const char* MQTT_PASS  = "secret";
const char* NTP_SERVER = "pool.ntp.org";
const char* TZ_VN      = "ICT-7";
#define WDT_TIMEOUT 30

// ── GPIO CB2S ────────────────────────────────────────────────────
#define PIN_SDA      0
#define PIN_SCL      1
#define PIN_CO2_TX   4
#define PIN_CO2_RX   5
#define PIN_DS18B20  8
#define PIN_BTN      9
#define PIN_LED_AUTO 10
#define PIN_LED_TAY  11
#define PIN_HC_ECHO  12
#define PIN_HC_TRIG  13
#define PIN_HX_DT    14
#define PIN_HX_SCK   26

// ── Kênh Relay ───────────────────────────────────────────────────
#define K_BOM        0
#define K_QUAT1      1
#define K_QUAT2      2
#define K_QUAT3      3
#define K_QUAT4      4
#define K_QUAT5      5
#define K_QUAT6      6
#define K_BOM_PHUN   7
#define K_BOM_TAM    8
#define K_SUOI1      9
#define K_SUOI2     10
#define K_SUOI3     11
#define K_SUOI4     12
#define K_DEN_A     13
#define K_DEN_B     14
// 15 = dự phòng
#define K_VAN_N1    16
#define K_VAN_N2    17
#define K_VAN_N3    18
#define K_VAN_N4    19
#define K_REM1_MO   20
#define K_REM1_DONG 21
#define K_REM2_MO   22
#define K_REM2_DONG 23
#define K_REM3_MO   24
#define K_REM3_DONG 25
#define K_REM4_MO   26
#define K_REM4_DONG 27
#define K_CHOAN1    28
#define K_CHOAN2    29
#define K_DEN_C     30
#define K_DEN_D     31

// ── Hằng số ─────────────────────────────────────────────────────
#define SO_QUAT       6
#define SO_SUOI       4
#define SO_DEN        4
#define SO_VAN        4
#define SO_REM        4
#define SO_CHOAN      2
#define CHIEU_THUNG_CM 100   // Chiều cao thùng cám (cm)

// ── Enum ─────────────────────────────────────────────────────────
enum CheDo    { TU_DONG, THU_CONG };
enum TrangThaiRem { REM_DUNG, REM_MO, REM_DONG };

// ── Cấu hình tự động ────────────────────────────────────────────
struct CauHinhQuat {
  float t0, t1, t2, t3, t4, t5;  // Ngưỡng nhiệt độ 6 cấp
  float nh3_bat_het;              // NH₃ bật hết quạt
};

struct CauHinhSuoi {
  float nhiet_do_muc_tieu;
  float hysteresis;
  int   tuan_tuoi;
  bool  suoi_bao_gom[SO_SUOI];
};

struct LichChoAn {
  uint8_t gio, phut, giay_chay;
  bool    may[SO_CHOAN];
};

struct CauHinhDen {
  uint8_t gio_bat, phut_bat;
  uint8_t gio_tat, phut_tat;
  bool    khu[SO_DEN];
};

// ── Hardware objects ─────────────────────────────────────────────
Adafruit_MCP23X17 mcp1, mcp2;
Adafruit_ADS1115  ads;
HX711             hx711;
NewPing           sonar(PIN_HC_TRIG, PIN_HC_ECHO, 400);
LiquidCrystal_I2C lcd(0x27, 20, 4);
SoftwareSerial    co2Serial(PIN_CO2_RX, PIN_CO2_TX);
WiFiClient        net;
PubSubClient      mqtt(net);
Preferences       prefs;

// ── Biến toàn cục ───────────────────────────────────────────────
String      deviceID;
CheDo       cheDo = TU_DONG;

CauHinhQuat cfgQuat  = {22, 26, 28, 30, 33, 35, 25.0};
CauHinhSuoi cfgSuoi  = {35.0, 1.0, 1, {true,true,true,true}};
CauHinhDen  cfgDen   = {5, 0, 21, 0, {true,true,true,true}};
LichChoAn   lichAn[3] = {
  {6, 0, 30, {true,true}},
  {11, 0, 30, {true,true}},
  {16, 0, 30, {true,true}}
};

float       nhietDo[3]   = {28.0, 28.0, 28.0};
float       doAm[3]      = {70.0, 70.0, 70.0};
float       nh3Ppm       = 0;
float       co2Ppm       = 800;
float       khoiLuong    = 0;
float       mucCam[2]    = {100.0, 100.0};
int         quatDangChay = 0;
bool        suoiDangBat  = false;
bool        lamMatPhun   = false;
bool        lamMatTam    = false;
bool        denChieuSang = false;
TrangThaiRem trangThaiRem[SO_REM] = {REM_DUNG,REM_DUNG,REM_DUNG,REM_DUNG};

unsigned long lastSensor    = 0;
unsigned long lastLcd       = 0;
unsigned long lastPublish   = 0;
unsigned long lastSchedule  = 0;
unsigned long lastBtn       = 0;

bool canhBaoNH3    = false;
bool canhBaoNhiet  = false;
unsigned long lastCanhBao = 0;

// ════════════════════════════════════════════════════════════════
//  RELAY
// ════════════════════════════════════════════════════════════════
void setRelay(int k, bool on) {
  if (k < 0 || k > 31) return;
  bool v = on ? HIGH : LOW;
  if (k < 16) mcp1.digitalWrite(k,      v);
  else        mcp2.digitalWrite(k - 16, v);
}

void tatTatCaRelay() {
  for (int i = 0; i < 16; i++) {
    mcp1.digitalWrite(i, LOW);
    mcp2.digitalWrite(i, LOW);
  }
}

// ════════════════════════════════════════════════════════════════
//  RÈM CỬA 2 CHIỀU
// ════════════════════════════════════════════════════════════════
int kRemMo  (int i) { return K_REM1_MO   + i * 2; }
int kRemDong(int i) { return K_REM1_DONG + i * 2; }

void dieuKhienRem(int i, TrangThaiRem tt) {
  setRelay(kRemMo(i),   false);
  setRelay(kRemDong(i), false);
  delay(500);
  if (tt == REM_MO)   setRelay(kRemMo(i),   true);
  if (tt == REM_DONG) setRelay(kRemDong(i), true);
  trangThaiRem[i] = tt;
  const char* s = (tt==REM_MO)?"mo":(tt==REM_DONG)?"dong":"dung";
  String topic = "livestock/" + deviceID + "/rem/" + i + "/state";
  mqtt.publish(topic.c_str(), s, true);
}

// ════════════════════════════════════════════════════════════════
//  THÔNG GIÓ — Theo nhiệt độ + NH₃
// ════════════════════════════════════════════════════════════════
float nhietDoTB() {
  return (nhietDo[0] + nhietDo[1] + nhietDo[2]) / 3.0f;
}

void kiemTraThongGio() {
  if (cheDo != TU_DONG) return;
  float t = nhietDoTB();
  int soQuat = 0;
  bool batPhun = false, batTam = false;

  // Ưu tiên NH₃
  if (nh3Ppm >= cfgQuat.nh3_bat_het) {
    soQuat = SO_QUAT;
    guiCanhBaoNH3();
  } else if (t >= cfgQuat.t5) {
    soQuat = SO_QUAT; batPhun = true; batTam = true;
  } else if (t >= cfgQuat.t4) {
    soQuat = SO_QUAT; batPhun = true;
  } else if (t >= cfgQuat.t3) {
    soQuat = SO_QUAT;
  } else if (t >= cfgQuat.t2) {
    soQuat = 4;
  } else if (t >= cfgQuat.t1) {
    soQuat = 2;
  }

  for (int i = 0; i < SO_QUAT; i++)
    setRelay(K_QUAT1 + i, i < soQuat);

  if (batPhun != lamMatPhun) { setRelay(K_BOM_PHUN, batPhun); lamMatPhun = batPhun; }
  if (batTam  != lamMatTam)  { setRelay(K_BOM_TAM,  batTam);  lamMatTam  = batTam;  }

  quatDangChay = soQuat;
}

// ════════════════════════════════════════════════════════════════
//  SƯỞi ẤM — Theo nhiệt độ mục tiêu + tuần tuổi
// ════════════════════════════════════════════════════════════════
float nhietDoMucTieuTheoTuan(int tuan) {
  if (tuan <= 1) return 35.0;
  if (tuan == 2) return 32.0;
  if (tuan == 3) return 29.0;
  if (tuan == 4) return 26.0;
  return 24.0;
}

void kiemTraSuoi() {
  if (cheDo != TU_DONG) return;
  float t = nhietDoTB();
  float mt = cfgSuoi.nhiet_do_muc_tieu;
  float hy = cfgSuoi.hysteresis;
  bool canBat = false;

  if (t < mt - hy) canBat = true;
  else if (t > mt + hy) canBat = false;
  else canBat = suoiDangBat;

  if (canBat != suoiDangBat) {
    suoiDangBat = canBat;
    for (int i = 0; i < SO_SUOI; i++)
      if (cfgSuoi.suoi_bao_gom[i]) setRelay(K_SUOI1 + i, canBat);
  }
}

// ════════════════════════════════════════════════════════════════
//  CHIẾU SÁNG — Quang chu kỳ
// ════════════════════════════════════════════════════════════════
void kiemTraChieuSang() {
  if (cheDo != TU_DONG) return;
  time_t now = time(nullptr);
  struct tm* t = localtime(&now);
  int ph = t->tm_hour * 60 + t->tm_min;
  int phBat = cfgDen.gio_bat * 60 + cfgDen.phut_bat;
  int phTat = cfgDen.gio_tat * 60 + cfgDen.phut_tat;

  bool canBat;
  if (phBat < phTat) canBat = (ph >= phBat && ph < phTat);
  else               canBat = (ph >= phBat || ph < phTat);

  if (canBat != denChieuSang) {
    denChieuSang = canBat;
    int ks[] = {K_DEN_A, K_DEN_B, K_DEN_C, K_DEN_D};
    for (int i = 0; i < SO_DEN; i++)
      if (cfgDen.khu[i]) setRelay(ks[i], canBat);
  }
}

// ════════════════════════════════════════════════════════════════
//  CHO ĂN TỰ ĐỘNG
// ════════════════════════════════════════════════════════════════
void kiemTraLichChoAn() {
  if (cheDo != TU_DONG) return;
  time_t now = time(nullptr);
  struct tm* t = localtime(&now);
  if (t->tm_sec > 5) return;

  for (int l = 0; l < 3; l++) {
    if (lichAn[l].gio == t->tm_hour && lichAn[l].phut == t->tm_min) {
      for (int m = 0; m < SO_CHOAN; m++) {
        if (lichAn[l].may[m]) {
          setRelay(K_CHOAN1 + m, true);
        }
      }
      delay(lichAn[l].giay_chay * 1000UL);
      for (int m = 0; m < SO_CHOAN; m++) setRelay(K_CHOAN1 + m, false);

      String msg = "{\"ca\":" + String(l) + ",\"giay\":" + String(lichAn[l].giay_chay) + "}";
      mqtt.publish(("livestock/"+deviceID+"/cho_an/done").c_str(), msg.c_str());
    }
  }
}

// ════════════════════════════════════════════════════════════════
//  CẢM BIẾN
// ════════════════════════════════════════════════════════════════
void docSHT30() {
  // Đọc 3 SHT30 tại địa chỉ 0x44, 0x45, 0x46
  // Code đơn giản hóa — thực tế dùng thư viện SHT30
  // nhietDo[0..2] và doAm[0..2] sẽ được cập nhật
}

void docNH3() {
  int16_t raw = ads.readADC_SingleEnded(0);
  // Chuyển đổi raw → ppm (cần hiệu chỉnh thực tế)
  nh3Ppm = (raw / 32767.0f) * 100.0f;  // Placeholder
}

void docCO2() {
  byte cmd[9] = {0xFF, 0x01, 0x86, 0x00, 0x00, 0x00, 0x00, 0x00, 0x79};
  co2Serial.write(cmd, 9);
  delay(100);
  if (co2Serial.available() >= 9) {
    byte buf[9];
    co2Serial.readBytes(buf, 9);
    if (buf[0] == 0xFF && buf[1] == 0x86) {
      co2Ppm = (float)((buf[2] << 8) | buf[3]);
    }
  }
}

void docMucCam() {
  for (int m = 0; m < SO_CHOAN; m++) {
    int dist = sonar.ping_cm();
    if (dist > 0 && dist <= CHIEU_THUNG_CM) {
      mucCam[m] = (float)(CHIEU_THUNG_CM - dist) / CHIEU_THUNG_CM * 100.0f;
    }
  }
}

void docCamBien() {
  docSHT30();
  docNH3();
  docCO2();
  docMucCam();

  // Publish sensor data
  StaticJsonDocument<256> doc;
  JsonArray ta = doc.createNestedArray("nhiet_do");
  JsonArray ha = doc.createNestedArray("do_am");
  for (int i = 0; i < 3; i++) { ta.add(nhietDo[i]); ha.add(doAm[i]); }
  doc["nhiet_do_tb"] = nhietDoTB();
  doc["nh3_ppm"]     = nh3Ppm;
  doc["co2_ppm"]     = co2Ppm;
  char buf[256]; serializeJson(doc, buf);
  mqtt.publish(("livestock/"+deviceID+"/sensor").c_str(), buf);

  // Publish mức cám
  StaticJsonDocument<64> docCam;
  docCam["may1_phan_tram"] = mucCam[0];
  docCam["may2_phan_tram"] = mucCam[1];
  docCam["canh_bao"] = (mucCam[0] < 20.0f || mucCam[1] < 20.0f);
  char bufCam[64]; serializeJson(docCam, bufCam);
  mqtt.publish(("livestock/"+deviceID+"/cam").c_str(), bufCam);
}

// ════════════════════════════════════════════════════════════════
//  CẢNH BÁO
// ════════════════════════════════════════════════════════════════
void guiCanhBaoNH3() {
  if (millis() - lastCanhBao < 300000) return;  // Không gửi quá 1 lần/5 phút
  lastCanhBao = millis();
  String muc = (nh3Ppm >= 50.0f) ? "nguy_hiem" : "canh_bao";
  String msg = "{\"gia_tri\":" + String(nh3Ppm, 1) +
               ",\"nguong\":25,\"muc\":\"" + muc + "\"}";
  mqtt.publish(("livestock/"+deviceID+"/alert/nh3").c_str(), msg.c_str());
}

void kiemTraCanhBao() {
  if (mucCam[0] < 5.0f || mucCam[1] < 5.0f) {
    mqtt.publish(("livestock/"+deviceID+"/alert/cam").c_str(),
      "{\"muc\":\"het\"}");
  } else if (mucCam[0] < 20.0f || mucCam[1] < 20.0f) {
    mqtt.publish(("livestock/"+deviceID+"/alert/cam").c_str(),
      "{\"muc\":\"sap_het\"}");
  }
}

// ════════════════════════════════════════════════════════════════
//  LCD
// ════════════════════════════════════════════════════════════════
bool canhBaoHienTai = false;

void capNhatLCD() {
  if (nh3Ppm >= 25.0f || nhietDoTB() > 38.0f) {
    // Màn hình cảnh báo
    lcd.clear();
    lcd.setCursor(0, 0); lcd.print("!!! CANH BAO !!!");
    if (nh3Ppm >= 25.0f) {
      lcd.setCursor(0, 1);
      lcd.print("NH3: "); lcd.print((int)nh3Ppm); lcd.print("ppm > 25ppm");
      lcd.setCursor(0, 2); lcd.print("Da bat toan bo quat");
      lcd.setCursor(0, 3); lcd.print("Kiem tra thong gio!");
    } else {
      lcd.setCursor(0, 1);
      lcd.print("Nhiet do: "); lcd.print(nhietDoTB(), 1); lcd.print("C");
      lcd.setCursor(0, 2); lcd.print("Da bat he lam mat");
      lcd.setCursor(0, 3); lcd.print("Kiem tra he thong!");
    }
    return;
  }

  // Màn hình bình thường
  time_t now = time(nullptr);
  struct tm* t = localtime(&now);

  lcd.setCursor(0, 0);
  lcd.print(cheDo == TU_DONG ? "AUTO" : "TAY ");
  lcd.print("  NH3:");
  lcd.print((int)nh3Ppm);
  lcd.print("p  ");
  lcd.print(nhietDoTB(), 1);
  lcd.print("C");

  lcd.setCursor(0, 1);
  lcd.print("Quat:");
  lcd.print(quatDangChay);
  lcd.print("/");
  lcd.print(SO_QUAT);
  lcd.print("  DoAm:");
  lcd.print((int)doAm[1]);
  lcd.print("%");

  lcd.setCursor(0, 2);
  lcd.print("Den:");
  lcd.print(denChieuSang ? "ON " : "OFF");
  lcd.print(" Suoi:");
  lcd.print(suoiDangBat ? "ON " : "OFF");

  lcd.setCursor(0, 3);
  lcd.print("ChoAn:  :   Cam:");
  lcd.print((int)min(mucCam[0], mucCam[1]));
  lcd.print("%");
}

// ════════════════════════════════════════════════════════════════
//  SAVE / LOAD CONFIG
// ════════════════════════════════════════════════════════════════
void saveConfig() {
  prefs.begin("cn", false);
  prefs.putBytes("quat",   &cfgQuat,  sizeof(cfgQuat));
  prefs.putBytes("suoi",   &cfgSuoi,  sizeof(cfgSuoi));
  prefs.putBytes("den",    &cfgDen,   sizeof(cfgDen));
  prefs.putBytes("lican",  lichAn,    sizeof(lichAn));
  prefs.putUChar("chedom", (uint8_t)cheDo);
  prefs.end();
}

void loadConfig() {
  prefs.begin("cn", true);
  prefs.getBytes("quat",   &cfgQuat,  sizeof(cfgQuat));
  prefs.getBytes("suoi",   &cfgSuoi,  sizeof(cfgSuoi));
  prefs.getBytes("den",    &cfgDen,   sizeof(cfgDen));
  prefs.getBytes("lican",  lichAn,    sizeof(lichAn));
  cheDo = (CheDo)prefs.getUChar("chedom", (uint8_t)TU_DONG);
  prefs.end();
}

// ════════════════════════════════════════════════════════════════
//  MQTT CALLBACK
// ════════════════════════════════════════════════════════════════
void mqttCallback(char* topic, byte* payload, unsigned int len) {
  payload[len] = 0;
  String t(topic), msg((char*)payload);
  StaticJsonDocument<512> doc;
  if (msg.length() > 2) deserializeJson(doc, msg);

  // Chế độ
  if (t.endsWith("/che_do/set")) {
    const char* m = doc["che_do"] | "tu_dong";
    cheDo = (!strcmp(m, "thu_cong")) ? THU_CONG : TU_DONG;
    if (cheDo == THU_CONG) tatTatCaRelay();
    saveConfig();
    digitalWrite(PIN_LED_AUTO, cheDo == TU_DONG);
    digitalWrite(PIN_LED_TAY,  cheDo == THU_CONG);
    return;
  }

  // Dừng khẩn cấp
  if (t.endsWith("/dung")) {
    tatTatCaRelay();
    quatDangChay = 0; suoiDangBat = false; lamMatPhun = false;
    return;
  }

  // Ngưỡng quạt
  if (t.endsWith("/quat/nguong/set")) {
    cfgQuat.t0 = doc["t0"] | 22.0f;
    cfgQuat.t1 = doc["t1"] | 26.0f;
    cfgQuat.t2 = doc["t2"] | 28.0f;
    cfgQuat.t3 = doc["t3"] | 30.0f;
    cfgQuat.t4 = doc["t4"] | 33.0f;
    cfgQuat.t5 = doc["t5"] | 35.0f;
    cfgQuat.nh3_bat_het = doc["nh3_bat_het"] | 25.0f;
    saveConfig(); return;
  }

  // Cài sưởi
  if (t.endsWith("/suoi/set")) {
    cfgSuoi.nhiet_do_muc_tieu = doc["nhiet_do_muc_tieu"] | 35.0f;
    cfgSuoi.hysteresis        = doc["hysteresis"]        | 1.0f;
    cfgSuoi.tuan_tuoi         = doc["tuan"]              | 1;
    if (cfgSuoi.tuan_tuoi > 0)
      cfgSuoi.nhiet_do_muc_tieu = nhietDoMucTieuTheoTuan(cfgSuoi.tuan_tuoi);
    saveConfig(); return;
  }

  // Cài đèn
  if (t.endsWith("/den/lich/set")) {
    cfgDen.gio_bat  = doc["gio_bat"]  | 5;
    cfgDen.phut_bat = doc["phut_bat"] | 0;
    cfgDen.gio_tat  = doc["gio_tat"]  | 21;
    cfgDen.phut_tat = doc["phut_tat"] | 0;
    saveConfig(); return;
  }

  // Cài cho ăn
  if (t.endsWith("/cho_an/lich/set")) {
    JsonArray la = doc["lich"];
    for (int i = 0; i < 3 && i < (int)la.size(); i++) {
      lichAn[i].gio       = la[i]["gio"]      | 6;
      lichAn[i].phut      = la[i]["phut"]     | 0;
      lichAn[i].giay_chay = la[i]["giay_chay"]| 30;
    }
    saveConfig(); return;
  }

  // Cho ăn thủ công
  if (t.endsWith("/cho_an/lenh")) {
    if (cheDo != THU_CONG) {
      mqtt.publish(("livestock/"+deviceID+"/loi").c_str(), "{\"ma\":\"DANG_TU_DONG\"}");
      return;
    }
    int may  = doc["may"]  | 1;
    int giay = doc["giay"] | 30;
    if (may >= 1 && may <= SO_CHOAN) {
      setRelay(K_CHOAN1 + may - 1, true);
      delay(giay * 1000UL);
      setRelay(K_CHOAN1 + may - 1, false);
    }
    return;
  }

  // Điều khiển rèm tay
  if (t.indexOf("/rem/") > 0 && t.endsWith("/lenh")) {
    if (cheDo != THU_CONG) {
      mqtt.publish(("livestock/"+deviceID+"/loi").c_str(), "{\"ma\":\"DANG_TU_DONG\"}");
      return;
    }
    int idx = t.charAt(t.indexOf("/rem/") + 5) - '1';
    TrangThaiRem tt = (msg=="mo") ? REM_MO : (msg=="dong") ? REM_DONG : REM_DUNG;
    if (idx >= 0 && idx < SO_REM) dieuKhienRem(idx, tt);
    return;
  }

  // Điều khiển quạt tay
  if (t.indexOf("/quat/") > 0 && t.endsWith("/lenh")) {
    if (cheDo != THU_CONG) return;
    if (t.indexOf("tat_ca") > 0) {
      bool bat = (msg == "ON");
      for (int i = 0; i < SO_QUAT; i++) setRelay(K_QUAT1 + i, bat);
    } else {
      int i = t.charAt(t.indexOf("/quat/") + 6) - '1';
      if (i >= 0 && i < SO_QUAT) setRelay(K_QUAT1 + i, msg == "ON");
    }
    return;
  }
}

// ════════════════════════════════════════════════════════════════
//  MQTT KẾT NỐI
// ════════════════════════════════════════════════════════════════
void connectMQTT() {
  String lwt = "livestock/" + deviceID + "/status";
  while (!mqtt.connected()) {
    if (mqtt.connect(deviceID.c_str(), MQTT_USER, MQTT_PASS,
                     lwt.c_str(), 1, true, "offline")) {
      mqtt.publish(lwt.c_str(), "online", true);
      mqtt.subscribe(("livestock/" + deviceID + "/#").c_str());
    } else {
      esp_task_wdt_reset(); delay(3000);
    }
  }
}

// ════════════════════════════════════════════════════════════════
//  SETUP
// ════════════════════════════════════════════════════════════════
void setup() {
  Serial.begin(115200);
  co2Serial.begin(9600);
  esp_task_wdt_init(WDT_TIMEOUT, true);
  esp_task_wdt_add(NULL);

  Wire.begin(PIN_SDA, PIN_SCL);
  if (!mcp1.begin_I2C(0x20)) Serial.println("ERR: MCP1");
  if (!mcp2.begin_I2C(0x21)) Serial.println("ERR: MCP2");
  for (int i = 0; i < 16; i++) {
    mcp1.pinMode(i, OUTPUT); mcp1.digitalWrite(i, LOW);
    mcp2.pinMode(i, OUTPUT); mcp2.digitalWrite(i, LOW);
  }

  ads.begin(0x48);
  hx711.begin(PIN_HX_DT, PIN_HX_SCK);
  lcd.init(); lcd.backlight();
  lcd.setCursor(0, 0); lcd.print("ChuongTrai v1.0");
  lcd.setCursor(0, 1); lcd.print("Dang khoi dong...");

  pinMode(PIN_BTN,     INPUT_PULLUP);
  pinMode(PIN_LED_AUTO, OUTPUT);
  pinMode(PIN_LED_TAY,  OUTPUT);

  loadConfig();
  digitalWrite(PIN_LED_AUTO, cheDo == TU_DONG);
  digitalWrite(PIN_LED_TAY,  cheDo == THU_CONG);

  WiFiManager wm;
  String apName = "ChuongTrai_" + String((uint32_t)ESP.getEfuseMac(), HEX);
  wm.autoConnect(apName.c_str(), "12345678");

  uint8_t m[6]; WiFi.macAddress(m);
  deviceID = "";
  for (int i = 0; i < 6; i++) { if(m[i]<16) deviceID+="0"; deviceID+=String(m[i],HEX); }

  configTzTime(TZ_VN, NTP_SERVER);
  mqtt.setServer(MQTT_HOST, MQTT_PORT);
  mqtt.setCallback(mqttCallback);
  mqtt.setBufferSize(512);
  connectMQTT();

  lcd.clear();
  lcd.setCursor(0, 0); lcd.print("San sang!");
}

// ════════════════════════════════════════════════════════════════
//  LOOP
// ════════════════════════════════════════════════════════════════
void loop() {
  esp_task_wdt_reset();
  if (!mqtt.connected()) connectMQTT();
  mqtt.loop();

  // Cảm biến: mỗi 30 giây
  if (millis() - lastSensor > 30000) {
    lastSensor = millis();
    docCamBien();
    if (cheDo == TU_DONG) {
      kiemTraThongGio();
      kiemTraSuoi();
      kiemTraChieuSang();
    }
    kiemTraCanhBao();
  }

  // Lịch cho ăn + chiếu sáng: mỗi 60 giây
  if (millis() - lastSchedule > 60000) {
    lastSchedule = millis();
    if (cheDo == TU_DONG) {
      kiemTraLichChoAn();
      kiemTraChieuSang();
    }
  }

  // LCD: mỗi 2 giây
  if (millis() - lastLcd > 2000) {
    lastLcd = millis();
    capNhatLCD();
  }

  // Nút vật lý: giữ 2 giây đổi chế độ
  if (millis() - lastBtn > 200) {
    lastBtn = millis();
    if (digitalRead(PIN_BTN) == LOW) {
      unsigned long t0 = millis();
      while (digitalRead(PIN_BTN) == LOW && millis() - t0 < 2000) esp_task_wdt_reset();
      if (millis() - t0 >= 2000) {
        cheDo = (cheDo == TU_DONG) ? THU_CONG : TU_DONG;
        if (cheDo == TU_DONG) tatTatCaRelay();
        saveConfig();
        digitalWrite(PIN_LED_AUTO, cheDo == TU_DONG);
        digitalWrite(PIN_LED_TAY,  cheDo == THU_CONG);
        mqtt.publish(("livestock/"+deviceID+"/che_do").c_str(),
                     cheDo == TU_DONG ? "tu_dong" : "thu_cong", true);
      }
    }
  }
}
```

---

*[← Đấu Nối](05_dau-noi-dien.md) | [MQTT API →](07_mqtt-api.md)*
