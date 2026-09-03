# Own-deauth-watch
# ⚡ DEAUTHER WATCH v2.0 — ESP32-S3 Super Mini (QSPI)

> **⚠️ YASAL UYARI:** Bu proje yalnızca eğitim, güvenlik araştırması ve kendi ağınızı test etme amacıyla hazırlanmıştır. İzinsiz ağlara veya cihazlara saldırmak yasa dışıdır. Kullanıcı tüm yasal sorumluluğu kabul eder.

<p align="center">
  <img src="https://img.shields.io/badge/Platform-ESP32--S3%20QSPI-blue?style=for-the-badge&logo=espressif" />
  <img src="https://img.shields.io/badge/Language-Arduino%20C%2B%2B-orange?style=for-the-badge&logo=arduino" />
  <img src="https://img.shields.io/badge/Version-2.0-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Lines-4000%2B-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-Educational%20Only-yellow?style=for-the-badge" />
</p>

ESP32-S3 Super Mini **(QSPI varyantı)** tabanlı, tek dosyada çalışan WiFi/BLE/BadUSB güvenlik araştırma platformu. 128x64 OLED ekran ve 3 buton ile tamamen bağımsız (standalone) çalışır. nRF24L01+ modülü ile 2.4 GHz spektrum analizi ve jammer özellikleri içerir.

> **QSPI Notu:** Bu varyantta GPIO 35, 36, 37 dahili flash/PSRAM'a bağlıdır ve kullanıcı tarafından **kullanılamaz**. nRF24L01+ için SPI2 (FSPI) pinleri buna göre seçilmiştir (GPIO 4/5/6/7/10).

---

## 📋 İçindekiler

- [Donanım Gereksinimleri](#-donanım-gereksinimleri)
- [Pin Bağlantıları](#-pin-bağlantıları)
- [Kurulum](#-kurulum)
- [Özellikler](#-özellikler)
- [Menü Yapısı](#-menü-yapısı)
- [WiFi Saldırıları](#-wifi-saldırıları)
- [BLE Saldırıları](#-ble-saldırıları)
- [BadUSB Modülü](#-badusb-modülü)
- [Pasif Analiz Araçları](#-pasif-analiz-araçları)
- [Termal Yönetim](#-termal-yönetim)
- [Ayarlar](#-ayarlar)
- [Katkıda Bulunma](#-katkıda-bulunma)

---

## 🔧 Donanım Gereksinimleri

| Bileşen | Model | Notlar |
|---|---|---|
| Mikrodenetleyici | ESP32-S3 Super Mini **QSPI** | USB-C, native USB HID, OPI PSRAM |
| OLED Ekran | SSD1306 128×64 | I2C, 0x3C adresi |
| RF Modülü | nRF24L01+ (PA+LNA önerilir) | SPI2 (FSPI) — GPIO 4/5/6 |
| Butonlar | Anlık basma (3 adet) | UP / DOWN / SELECT |
| Güç | LiPo / USB-C | — |

> **Not:** nRF24L01 modülü opsiyoneldir. BLE JAM, BT Classic JAM ve WiFi Channel JAM özellikleri için gereklidir. Diğer tüm özellikler yalnızca ESP32-S3 ile çalışır.
>
> **⚠️ QSPI Pin Kısıtlaması:** GPIO 35 (MOSI), 36 (SCK), 37 (MISO) bu varyantta dahili QSPI flash'a ayrılmıştır. Bu pinleri SPI veya başka amaçlarla **kesinlikle kullanmayın** — yapılması flash corruption'a veya bootloop'a yol açar.

---

## 📌 Pin Bağlantıları

### Butonlar
| Buton | GPIO |
|---|---|
| UP | GPIO 1 |
| DOWN | GPIO 2 |
| SELECT | GPIO 3 |

### OLED (I2C)
| OLED | GPIO |
|---|---|
| SDA | GPIO 8 |
| SCL | GPIO 9 |

### nRF24L01+ (FSPI / SPI2)

| nRF24 | GPIO | Notlar |
|---|---|---|
| CE | GPIO 7 | — |
| CSN | GPIO 10 | — |
| SCK | GPIO 4 | FSPI SCK |
| MOSI | GPIO 5 | FSPI MOSI |
| MISO | GPIO 6 | FSPI MISO |

> **⚠️ GPIO 35/36/37 kullanmayın!** QSPI varyantında bu pinler dahili flash'a bağlıdır. Kod varsayılan `SPI.begin()` yerine `SPIClass nrfSPI(FSPI)` kullanır — bu tam da bu çakışmayı önlemek içindir.

---

## 🚀 Kurulum

### 1. Arduino IDE Ayarları

**Board Manager URL ekle:**
```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

**Board seç:**
- `ESP32S3 Dev Module`
- Flash Mode: **QIO 80MHz**
- Flash Size: **4MB** (veya 8MB — modeline göre)
- USB CDC On Boot: **Enabled**
- USB Mode: **USB-OTG (TinyUSB)**
- PSRAM: **OPI PSRAM**

> **QSPI + PSRAM:** Bu board OPI PSRAM destekler. `ps_calloc` ile büyük buffer'lar (AP listesi, handshake kayıtları) heap yerine PSRAM'a alınır. PSRAM'ı Arduino IDE'den **OPI PSRAM** olarak seçmezseniz `ps_calloc` normal heap'e düşer, bellek hatası alabilirsiniz.

### 2. Gerekli Kütüphaneler

Arduino IDE Library Manager'dan yükle:

```
Adafruit GFX Library
Adafruit SSD1306
RF24 by TMRh20
```

### 3. Derleme ve Yükleme

```bash
# Dosyayı aç
deauther_watch.ino

# Board: ESP32S3 Dev Module
# Port: COMx / /dev/ttyACM0
# Upload Speed: 921600

# Upload butonuna bas
```

---

## ✨ Özellikler

### WiFi
- ✅ AP + Station taraması (aktif/pasif)
- ✅ Deauth saldırısı (tek AP / tüm AP'ler)
- ✅ Beacon flood (sahte SSID yayını)
- ✅ Probe request flood
- ✅ EAPOL/Handshake yakalama (4-way handshake detection)
- ✅ Deauth saldırısı tespiti (IDS modu)
- ✅ Paket monitörü (canlı frame sayacı)
- ✅ Kanal analizi (1-13 arası tüm kanallar)
- ✅ RSSI izleyici (grafik görünüm)
- ✅ MAC Spoofer (rastgele / özel)
- ✅ OUI tanıma (30 üretici)
- ✅ PMF korumalı AP tespiti

### BLE / Bluetooth
- ✅ BLE cihaz tarama
- ✅ BLE Advertising jammer (kanal 37/38/39)
- ✅ BT Classic spektrum jammer (79 kanal sweep)
- ✅ BT Deauth (advertising saldırısı)
- ✅ Android cihaz spam (sahte BLE bildirimleri)
- ✅ iOS popup spam (Apple vicinity protocol)

### BadUSB (HID)
- ✅ Türkçe Q klavye tam desteği (AltGr, özel karakterler)
- ✅ **SysInfo Popup** — Sistem bilgilerini PowerShell ile açar
- ✅ **Admin Kullanıcı Ekle** — Gizli admin hesabı oluşturur
- ✅ **WiFi Şifrelerini Al** — Kayıtlı WiFi profillerini dump eder
- ✅ **Defender Kapat** — Windows Defender'ı registry ile devre dışı bırakır
- ✅ **AMSI Bypass** — PowerShell script execution kilidini kaldırır
- ✅ **Lock Screen** — Ekranı kilitler (Win+L)

### Analiz & Pasif
- ✅ Canlı paket monitörü (tip bazlı istatistik)
- ✅ Kanal analizi (her kanalda frame yoğunluğu)
- ✅ RSSI grafik izleyici (ring buffer, gerçek zamanlı)
- ✅ Handshake / EAPOL sniffer (MAC + BSSID kaydı)
- ✅ Deauth saldırısı dedektörü (reason code analizi)
- ✅ OUI tabanlı üretici tanıma

### Sistem
- ✅ Termal yönetim (75°C'de nRF24 durdurma)
- ✅ TX Rate ayarı (1/5/10/50/100 pkt/s)
- ✅ Kanal seçimi (1-13)
- ✅ MAC Spoofer
- ✅ PSRAM destekli büyük buffer'lar
- ✅ NVS / Preferences ile kalıcı ayarlar
- ✅ Saat (RTC simülasyonu)
- ✅ Sistem bilgisi ekranı (CPU, RAM, chip rev)

---

## 🗂️ Menü Yapısı

```
DEAUTHER WATCH v2.0
├── SCAN
│   ├── AP+ST       → AP ve Station taraması
│   ├── AP Only     → Yalnızca AP taraması
│   └── ST Only     → Yalnızca Station taraması
├── SHOW
│   ├── APs         → Bulunan AP listesi
│   ├── Stations    → Bulunan istasyon listesi
│   ├── SSIDs       → Özel SSID listesi
│   └── Probes      → Probe request listesi
├── ATTACK          → Seçili AP'ye saldırı menüsü
│   ├── DEAUTH      → Deauth flood
│   ├── BEACON      → Beacon flood
│   ├── PROBE       → Probe flood
│   ├── DEAUTH+BCN  → Kombine saldırı
│   ├── HANDSHK SNIFF → EAPOL/Handshake yakalama
│   ├── WiFi CH JAM → nRF24 ile kanal jammer
│   └── [START/STOP]
├── BLE ATTACK
│   ├── SCAN        → BLE cihaz tarama
│   ├── BLE JAM     → BLE advertising jammer
│   ├── BT CLASSIC JAM → BT Classic spektrum jammer
│   ├── BT DEAUTH   → BLE deauth
│   ├── ANDROID SPAM
│   └── IOS SPAM
├── PKT MONITOR     → Canlı paket istatistiği
├── CH ANALYZER     → Kanal yoğunluk analizi
├── RSSI TRACKER    → Sinyal gücü grafiği
├── DEAUTH DET.     → Deauth saldırısı dedektörü
├── CLOCK           → Saat
├── BAD USB
│   ├── SysInfo     → Sistem bilgisi popup
│   ├── AddAdmin    → Gizli admin ekle
│   ├── WiFiPass    → WiFi şifrelerini dump et
│   ├── DefenderOff → Windows Defender kapat
│   ├── AMSIBypass  → AMSI bypass
│   └── Lock Screen → Ekranı kilitle
└── SETTINGS
    ├── MAC SPOOFER
    ├── TX Rate
    ├── Channel
    ├── SSID List
    └── System Info
```

---

## 📡 WiFi Saldırıları

### Deauth Flood
Seçili AP'ye veya tüm taranmış AP'lere IEEE 802.11 deauthentication frame'leri gönderir. PMF (Protected Management Frames) aktif AP'ler bu saldırıya karşı dirençlidir — kod bu durumu tespit eder ve ekranda gösterir.

### Beacon Flood
Özel SSID listesinden veya otomatik üretilmiş SSID'lerle sahte AP beacon'ları yayar. Çevre cihazlarda ağ listesini kirletir.

### Handshake Sniffer
Promiscuous mod ile 4-way EAPOL handshake'leri pasif olarak yakalar. BSSID, client MAC ve zaman damgası kaydedilir.

### Deauth Detector
Ortamda deauth frame'leri tespit edip kaynak MAC ve reason code'u gösterir. IDS (Intrusion Detection) amaçlı kullanım içindir.

---

## 📶 BLE Saldırıları

### BLE JAM (nRF24 ile)
BLE advertising kanallarına (2402, 2426, 2480 MHz) nRF24L01+ ile yoğun gürültü gönderir. Yakın çevredeki BLE reklamlarını bozar.

### BT Classic JAM (nRF24 ile)
2402–2480 MHz arasındaki 79 Bluetooth kanalını sürekli sweep eder. Her kanalda rastgele payload ile BT FHSS senkronizasyonunu bozar.

### Android / iOS Spam
Sahte BLE advertising packet'leri ile Android cihazlara Bluetooth eşleşme bildirimi, iOS cihazlara Apple proximity popup'ı gönderir.

---

## 💀 BadUSB Modülü

ESP32-S3'ün native USB HID özelliği kullanılarak bilgisayara otomatik klavye girişi simüle edilir. **Türkçe Q klavye layoutu tam desteklenir** (AltGr kombinasyonları, ğ, ü, ş, ı, ö, ç dahil).

### Payload Listesi

| # | Payload | Açıklama |
|---|---|---|
| 0 | SysInfo | PowerShell ile CPU, RAM, disk, işletim sistemi bilgilerini popup gösterir |
| 1 | AddAdmin | Gizli bir lokal yönetici hesabı oluşturur |
| 2 | WiFiPass | Kayıtlı tüm WiFi profillerini ve şifrelerini netsh ile dump eder |
| 3 | DefenderOff | Windows Defender'ı registry üzerinden kalıcı olarak devre dışı bırakır |
| 4 | AMSIBypass | PowerShell AMSI'yi bypass ederek script çalıştırma kısıtlamasını kaldırır |
| 5 | LockScreen | Win+L kısayoluyla ekranı kilitler |

> **Not:** BadUSB yalnızca hedef sistemin kilidinin açık ve USB HID girişlerini kabul ettiği durumlarda çalışır.

---

## 🌡️ Termal Yönetim

ESP32-S3'ün dahili sıcaklık sensörü ile gerçek zamanlı termal koruma:

| Eşik | Davranış |
|---|---|
| > 75°C | nRF24 TX durdurulur |
| < 65°C | nRF24 TX otomatik devam eder |
| Her 2 sn | Sıcaklık ölçümü |

Dual mod (Deauth + nRF Jam aynı anda aktifken) PA seviyesi düşürülür (`RF24_PA_LOW`), tek mod da `RF24_PA_HIGH` ile maksimum güçle çalışır.

---

## ⚙️ Ayarlar

| Ayar | Seçenekler |
|---|---|
| TX Rate | 1 / 5 / 10 / 50 / 100 paket/sn |
| WiFi Kanal | 1 – 13 |
| MAC Spoofer | Rastgele üret / Manuel gir |
| SSID Listesi | Özel SSID yönetimi |

Ayarlar `Preferences` (NVS) ile kalıcı olarak saklanır ve cihaz yeniden başlatıldığında korunur.

---

## 📦 Proje Yapısı

```
deauther_watch.ino      ← Tek dosya, ~4000 satır, tüm özellikler
```

Tüm proje tek `.ino` dosyasında barındırılmaktadır. Bölümler şu sırayla organize edilmiştir:

```
1. Pin & Sabit Tanımları
2. nRF24 & BLE Jammer Değişkenleri
3. Termal Yönetim
4. OUI Tablosu (PROGMEM)
5. BadUSB Modülü (HID + Payloadlar)
6. Veri Yapıları (AP, Station, Handshake, BLE Device)
7. AppMode Enum & Durum Değişkenleri
8. Fonksiyon Prototipleri
9. BLE Callback'leri
10. WiFi Promiscuous Callback (paket analizi)
11. Handshake / EAPOL Parser
12. Menü Builder'ları
13. Mod Geçiş Motoru (changeMode)
14. Ekran Çizim Fonksiyonları
15. Ana Loop (setup / loop)
```

---

## 📚 Kütüphane Bağımlılıkları

| Kütüphane | Sürüm | Kaynak |
|---|---|---|
| Arduino ESP32 | ≥ 2.0.17 | Espressif |
| Adafruit GFX | ≥ 1.11 | Adafruit |
| Adafruit SSD1306 | ≥ 2.5 | Adafruit |
| RF24 | ≥ 1.4.8 | TMRh20 |
| Wire | Built-in | ESP32 |
| WiFi | Built-in | ESP32 |
| BLEDevice / BLEScan | Built-in | ESP32 |
| USB / USBHIDKeyboard | Built-in | ESP32 Arduino Core |
| Preferences | Built-in | ESP32 |

---

## 🤝 Katkıda Bulunma

Pull request'ler açıktır. Büyük değişiklikler için önce bir issue açmanız önerilir.

1. Fork'layın
2. Feature branch oluşturun (`git checkout -b feature/yeni-ozellik`)
3. Commit'leyin (`git commit -m 'Yeni özellik: ...'`)
4. Push'layın (`git push origin feature/yeni-ozellik`)
5. Pull Request açın

---

## ⚖️ Lisans & Yasal

Bu proje **yalnızca eğitim ve kendi sistemlerinizi test etme** amacıyla paylaşılmıştır.

- Kendi ağınız ve cihazlarınız dışında kullanmak **yasadışıdır**
- Proje yazarları herhangi bir kötüye kullanımdan sorumlu tutulamaz
- Birçok ülkede yetkisiz ağ saldırısı ciddi hukuki yaptırımlara tabidir

---

<p align="center">
  <b>ESP32-S3 Super Mini • SSD1306 OLED • nRF24L01+ • BadUSB HID</b><br/>
  <i>Eğitim amaçlı güvenlik araştırma platformu</i>
</p>
