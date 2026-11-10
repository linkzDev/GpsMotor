# 🛵 IoT Smart GPS Tracker & Anti-Theft Motor System

Sistem pelacak GPS dan proteksi anti-maling sepeda motor cerdas berbasis **ESP32-C3 Super Mini** (arsitektur 32-bit RISC-V) yang terintegrasi dengan **Web Dashboard interaktif (Leaflet.js)** via protokol **MQTT (HiveMQ Public Broker)**. 

Dilengkapi dengan sistem perimeter keamanan mandiri (**Hardware-Level Offline Geofencing**). Konfigurasi batas radius atau wilayah kotak disimpan langsung pada memori **Non-Volatile Storage (NVS Flash ESP32)**, sehingga motor tetap otomatis mati atau klakson meraung jika dilarikan ke luar zona aman, **meskipun koneksi Wi-Fi/internet terputus**.

---

## 🌟 Fitur Utama

- **Real-Time Live Tracking:** Pemantauan koordinat garis lintang/bujur (Latitude & Longitude), kecepatan motor ($km/h$), dan jumlah satelit GPS aktif secara berkala via MQTT.
- **Auto-Follow Tracking Mode:** Fitur kamera peta cerdas pada dashboard web yang otomatis mengunci dan membuntuti posisi motor secara dinamis.
- **Offline Hardware Geofencing (Circle & Square):** Gambar batas lingkaran atau persegi langsung dari peta web dashboard menggunakan toolbar interaktif. Batas perimeter dihitung lokal di ESP32 menggunakan formula Haversine.
- **Multi-Action Threshold Response:** Pilihan respon otomatis saat kendaraan menembus batas:
  1. *Engine Cut-Off* (Mematikan mesin seketika).
  2. *Horn Alarm Alert* (Membunyikan klakson putus-putus/intermiten).
  3. *Engine Cut-Off + Horn Alarm* (Kombinasi keduanya).
- **Manual Remote Kill Switch:** Tombol darurat di web dashboard untuk memutus kontak/pengapian motor dan mengaktifkan klakson secara manual dari jarak jauh.
- **Interactive City Navigation:** Pilihan preset kota (Medan, Jakarta, Bandung, Surabaya, Yogyakarta) untuk memudahkan penyesuaian range dan kalibrasi peta.
- **Smart Captive Portal (WiFiManager Integration):** Konfigurasi hotspot mudah via HP tanpa perlu coding ulang. Dilengkapi proteksi *timeout* (40 detik) agar fungsi pelacak tidak blocking saat kendaraan berada di jalan raya.
- **Serverless Web Architecture:** Web dashboard statis dapat langsung di-host gratis di **GitHub Pages** menggunakan koneksi WebSocket WSS (Port `8884`).

---

## 🛠️ Komponen & Skema Pinout Hardware

Rancangan pinout ini telah dioptimalkan khusus board **ESP32-C3 Super Mini** agar tidak berbenturan dengan jalur Native USB CDC, pin bootstrapping, maupun baseband radio Wi-Fi.

| Komponen Hardware | Pin Modul | Pin ESP32-C3 Super Mini | Catatan Teknis / Fungsi |
| :--- | :--- | :--- | :--- |
| **GPS U-blox Neo-6M** | VCC | **5V / VBUS** | Wajib 5V agar LDO regulator onboard modul stabil |
| | GND | **GND** | Ground bersama |
| | **TX** | **GPIO 0** | ESP32 Hardware UART1 RX (Bebas dari jalur USB D-) |
| | **RX** | **GPIO 1** | ESP32 Hardware UART1 TX |
| **Modul Relay 2-CH** | VCC | **5V / VBUS** | Daya koil relay 5V |
| | GND | **GND** | Ground bersama |
| | **IN1** | **GPIO 2** | **Relay 1: Engine Cut-Off** (Active-LOW, putus kabel pulser/starter) |
| | **IN2** | **GPIO 3** | **Relay 2: Horn Pulser** (Active-LOW, intermiten 300ms flip-flop) |
| **Buzzer Aktif** | (+) | **GPIO 10** | Indikator internal sasis / alert alarm kabin |
| | (-) | **GND** | Ground |

> **Catatan Instalasi Motor:** Gunakan modul Step-Down DC-DC Buck Converter (seperti LM2596 atau Mini MP1584EN) untuk menurunkan tegangan Aki Motor (12V) ke **5V 2A** sebelum dihubungkan ke pin VBUS/5V ESP32 dan Relay.

---

## 📡 Protokol & Topik MQTT

- **Broker:** `broker.hivemq.com`
- **Port:** `1883` (TCP Standar ESP32) | `8884` (WebSocket WSS untuk GitHub Pages)

### 1. Telemetri Status (ESP32 ➔ Web Dashboard)
**Topik:** `tracker/motor/rivaldi/telemetry`
```json
{
  "lat": 3.58524,
  "lng": 98.67560,
  "speed": 34.2,
  "sats": 8,
  "r1": 0,
  "r2": 0,
  "breach": 0
}
