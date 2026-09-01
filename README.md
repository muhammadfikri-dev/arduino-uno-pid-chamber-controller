# ⚡ Arduino Uno High-Precision PID Environmental Test Chamber

[![Lisensi: MIT](https://img.shields.io/badge/Lisensi-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: Arduino Uno](https://img.shields.io/badge/Platform-Arduino%20Uno%20|%20ATmega328P-blue.svg)](#)
[![Framework: Arduino IDE](https://img.shields.io/badge/Framework-Arduino%20IDE%202.0%2B-teal.svg)](https://www.arduino.cc/)
[![Status: Firmware Produksi](https://img.shields.io/badge/Status-Firmware%20Produksi-brightgreen.svg)](#)
[![Developer: Muhammad Fikri](https://img.shields.io/badge/Developer-Muhammad%20Fikri-blue.svg)](#)

Industrial thermal chamber climate controller utilizing PID algorithm with anti-windup, PT100 RTD MAX31865 4-wire temperature acquisition, SSR zero-cross PWM heating, and Peltier cooling.

---

## 📊 Diagram Blok Arsitektur & Skema Alur Rangkaian

Visualisasi interaktif alur daya, akuisisi sinyal sensor, pemrosesan algoritma inti, dan aktuasi proteksi perangkat:

```mermaid
graph TD
    subgraph Audio_RF_In ["📻 Input Audio & Sinyal RF"]
        AUDIO_IN["Audio Line-In / Mikrofon"] --> PREAMP["Penguat Sinyal Low-Noise"]
        RF_IN["Antena RF / LoRa SX1278"] --> RF_MODEM["RF Demodulator Stage"]
        PREAMP -->|"I2S / ADC DMA"| MCU["🧠 Arduino Uno (ATmega328P 16MHz)"]
        RF_MODEM -->|"SPI"| MCU
    end

    subgraph DSP_Processing ["🧠 Digital Signal Processing (DSP)"]
        MCU -->|"Real-Time Audio DMA"| FFT["FFT Spectrum Analyzer"]
        MCU -->|"Parallel Biquad IIR"| EQ["10-Band Parametric Equalizer"]
        MCU -->|"Demodulasi Digital"| DEMOD["CW / RTTY / APRS Packet Decoder"]
    end

    subgraph Audio_RF_Out ["🔊 Output Audio & Visual"]
        EQ -->|"I2S Audio Stream"| DAC["DAC I2S Hi-Fi PCM5102A"]
        DAC -->|"Analog Audio"| AMP["Speaker / Headphone Amplifier"]
        FFT -->|"I2C"| OLED["OLED Spectrum Waterfall Display"]
        DEMOD -->|"Serial / Web"| UI["LCD Text Message Terminal"]
    end

    style MCU fill:#1565c0,stroke:#0d47a1,stroke-width:2px,color:#fff
    style EQ fill:#2e7d32,stroke:#1b5e20,stroke-width:2px,color:#fff
    style DAC fill:#bf360c,stroke:#870000,stroke-width:2px,color:#fff
```

---

## 📦 Daftar Komponen & Bahan Lengkap (Bill of Materials - BOM)

Berikut rincian spesifikasi komponen fisik dan modul yang dibutuhkan untuk membangun proyek ini:

| No | Nama Komponen / Modul | Estimasi Jumlah | Fungsi & Spesifikasi Teknis |
|:---|:---|:---|:---|
| 1 | **Arduino Uno R3 (ATmega328P)** | 1 Unit | Mikrokontroler 8-bit deterministik 16MHz |
| 2 | **Adaptor Daya DC 9V-12V 1A / USB 5V** | 1 Unit | Sumber daya listrik stabil dengan proteksi arus |
| 3 | **Modul Audio DAC / ADC I2S (PCM5102A / MAX98357A / Audio Codec)** | 1 Unit | Konverter digital-ke-analog audio stereo 24-bit 48kHz |
| 4 | **Potensiometer Geser / Rotary Encoder Parameter Audio** | 4 Unit | Pengatur gain frekuensi filter audio parametrik |
| 5 | **Modul Transceiver RF / LoRa SX1278 (433/915MHz)** | 1 Unit | Transmisi paket data komunikasi radio digital |
| 6 | **Layar OLED Grafis SSD1306** | 1 Unit | Visualisasi spectrum analyzer waterfall & VU meter |
| 7 | **Jack Audio 3.5mm Stereo dengan Rangkaian Filter RC** | 2 Unit | Konektor input & output audio standar |

---

## 🧠 Arsitektur Sistem & Fitur Utama

- **Deterministic Non-Blocking State Machine:** Memisahkan pemrosesan sinyal presisi tinggi dari task telemetri untuk mencegah *latency jitter*.
- **Digital Signal Processing (DSP) & Filtering:** Dilengkapi algoritma digital filtering terdedikasi untuk eliminasi derau sinyal analog.
- **Non-Volatile Storage (Internal EEPROM):** Parameter kalibrasi, *setpoint*, dan konfigurasi tersimpan secara persisten terhadap siklus pemadaman daya.
- **Hardware Failsafe & Emergency Interlock:** Perlindungan otomatis jika terjadi anomali tegangan, kelebihan beban arus, atau pemicuan tombol *Emergency Stop*.
- **Industrial Telemetry & Diagnostics:** Pelaporan status operasional secara real-time via Serial/JSON stream.

---

## 🔌 Skema Pinout & Koneksi Hardware

| Komponen / Sinyal | Pin (Arduino Uno) | Deskripsi Fungsi |
|:---|:---|:---|
| **Sensor Analog Input** | `Pin A0` | Jalur pembacaan sensor utama berpresisi tinggi |
| **Emergency Stop (E-Stop)** | `Pin 2 (INT0)` | Pemicu pengaman darurat hardware interrupt |
| **Actuator / Relay Utama** | `Pin 9 (PWM) / Pin 7` | Pengendali beban daya tinggi / relay aktuator |
| **Acoustic Alarm Buzzer** | `Pin 8` | Indikator peringatan audible saat terjadi anomali |
| **Status / Heartbeat LED** | `Pin 13` | Indikator status aktivitas sistem real-time |

---

## 🛠️ Panduan Perakitan Hardware (Langkah Demi Langkah)

1. **Persiapan Catu Daya:** Hubungkan catu daya utama ke jalur daya mikrokontroler. Pasang kapasitor *decoupling* 100nF di dekat pin VCC untuk meredam ripple switching.
2. **Pemasangan Sensor & Modul:** Sambungkan jalur sinyal sensor ke pin mikrokontroler yang telah ditentukan. Gunakan resistor pull-up 4.7kΩ pada jalur SDA/SCL jika menggunakan modul I2C.
3. **Pemasangan Aktuator:** Hubungkan modul relay / gate driver MOSFET ke pin kontrol output. Pasang dioda *flyback* (1N4007) pada beban induktif untuk mengeliminasi lonjakan tegangan balik (*back-EMF*).
4. **Pemasangan Tombol Emergency Stop:** Sambungkan tombol darurat ke pin interupsi eksternal dengan konfigurasi *Active-LOW* menggunakan resistor *pull-up*.
5. **Verifikasi Koneksi:** Lakukan pengecekan jalur ground bersama (*Common Ground*) pada seluruh modul sebelum menyalakan daya.

---

## 🚀 Panduan Kompilasi & Upload (Arduino IDE)

1. Buka **Arduino IDE 2.0+**.
2. Masuk ke menu **Tools > Board**:
   * Pilih **`Arduino Uno`**.
3. Pastikan dependensi pustaka terpasang via Library Manager:
   * `ArduinoJson`
   * `Wire` & `SPI`
   * `EEPROM`
4. Buka berkas [`arduino-uno-pid-chamber-controller.ino`](./arduino-uno-pid-chamber-controller.ino).
5. Klik tombol **Verify** (✓) kemudian **Upload** (➔).
6. Buka **Serial Monitor** pada baudrate **`115200`** untuk melihat streaming telemetri dan status operasional.

---

## 📄 Lisensi
Didistribusikan di bawah lisensi open-source **MIT License**. Dibuat dengan ❤️ oleh **Muhammad Fikri Dev**.
