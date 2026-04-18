Nama: Wendy Virtus  
NIM: H1H024048  
Shift Awal: B  
Shift Akhir: B  

---

## 3.5 Pertanyaan Praktikum
### 1. Jelaskan proses dari input keyboard hingga LED menyala/mati! 
Pengguna mengetik karakter di Serial Monitor → komputer mengirim data melalui USB ke chip UART Arduino → data masuk ke buffer serial Arduino → `Serial.available()` mendeteksi ada data masuk → `Serial.read()` membaca satu karakter → program mengevaluasi kondisi `if-else`: jika '1' maka `digitalWrite(PIN_LED, HIGH)` → arus mengalir ke LED → LED menyala. Jika '0' maka `digitalWrite(PIN_LED, LOW)` → LED mati.

---

### 2. Mengapa digunakan Serial.available() sebelum membaca data? Apa yang terjadi jika baris tersebut dihilangkan? 
`Serial.available()` digunakan untuk memastikan ada data yang tersedia di buffer serial sebelum membaca. Jika baris tersebut dihilangkan, `Serial.read()` tetap dipanggil setiap iterasi `loop()` meskipun tidak ada data masuk. Dalam kondisi ini `Serial.read()` mengembalikan nilai -1 (yang diinterpretasikan sebagai karakter tidak valid), sehingga pesan "Perintah tidak dikenal" akan muncul terus-menerus di Serial Monitor tanpa berhenti, membuat output menjadi sangat berantakan dan tidak dapat digunakan.

---

### 3. Modifikasi program agar LED berkedip (blink) ketika menerima input '2' dengan kondisi jika ‘2’ aktif maka LED akan terus berkedip sampai perintah selanjutnya diberikan dan berikan penjelasan disetiap baris kode nya dalam bentuk README.md!

```cpp
#include <Arduino.h>

// ── 1. Definisi pin LED ──────────────────────────────────────────────────────
// LED dihubungkan ke Pin 8 Arduino melalui resistor 220Ω
const int PIN_LED = 8;

// ── 2. Variabel mode dan timing blink ────────────────────────────────────────
// blinkMode: true = LED sedang dalam mode berkedip, false = mode normal
bool blinkMode = false;

// Menyimpan waktu terakhir LED berubah state saat mode blink aktif
unsigned long lastBlinkTime = 0;

// Interval kedip: LED ON selama 500ms, OFF selama 500ms
const unsigned long blinkInterval = 500;

// State LED saat ini untuk toggle (true = ON, false = OFF)
bool ledState = false;

// ── 3. setup() ───────────────────────────────────────────────────────────────
void setup() {
  // Inisialisasi komunikasi Serial dengan baud rate 9600 bps
  // Harus sama dengan baud rate di Serial Monitor
  Serial.begin(9600);

  // Tampilkan petunjuk penggunaan ke Serial Monitor saat program mulai
  Serial.println("Ketik '1' untuk LED ON, '0' untuk LED OFF, '2' untuk LED Blink");

  // Set Pin 8 sebagai OUTPUT agar bisa mengirim sinyal HIGH/LOW ke LED
  pinMode(PIN_LED, OUTPUT);
}

// ── 4. loop() ────────────────────────────────────────────────────────────────
void loop() {

  // ── Bagian A: Cek dan baca perintah dari Serial Monitor ──────────────────
  // Serial.available() > 0 berarti ada data baru di buffer serial
  if (Serial.available() > 0) {

    // Baca satu karakter dari buffer serial
    char data = Serial.read();

    // Evaluasi karakter yang diterima
    if (data == '1') {
      // Perintah '1': matikan mode blink, nyalakan LED
      blinkMode = false;                  // Nonaktifkan mode blink
      digitalWrite(PIN_LED, HIGH);        // Nyalakan LED
      Serial.println("LED ON");           // Kirim konfirmasi ke Serial Monitor

    } else if (data == '0') {
      // Perintah '0': matikan mode blink, matikan LED
      blinkMode = false;                  // Nonaktifkan mode blink
      digitalWrite(PIN_LED, LOW);         // Matikan LED
      Serial.println("LED OFF");          // Kirim konfirmasi ke Serial Monitor

    } else if (data == '2') {
      // Perintah '2': aktifkan mode blink
      blinkMode = true;                   // Aktifkan mode blink
      lastBlinkTime = millis();           // Catat waktu mulai blink
      Serial.println("LED BLINK ON");     // Kirim konfirmasi ke Serial Monitor

    } else if (data != '\n' && data != '\r') {
      // Karakter selain '0', '1', '2', newline, dan carriage return:
      // → tampilkan pesan error (filter \n dan \r agar tidak muncul spam)
      Serial.println("Perintah tidak dikenal");
    }
  }

  // ── Bagian B: Eksekusi blink menggunakan millis() (non-blocking) ─────────
  // Blok ini hanya aktif jika blinkMode == true
  if (blinkMode) {
    // millis() mengembalikan waktu (ms) sejak Arduino menyala
    // Cek apakah sudah lewat 500ms dari kedip terakhir
    if (millis() - lastBlinkTime >= blinkInterval) {

      // Update waktu referensi terakhir ke waktu sekarang
      lastBlinkTime = millis();

      // Toggle state LED: jika sebelumnya ON → jadi OFF, dan sebaliknya
      ledState = !ledState;

      // Terapkan state baru ke LED
      digitalWrite(PIN_LED, ledState ? HIGH : LOW);
    }
  }
  // CATATAN: Karena menggunakan millis() bukan delay(), program tidak pernah
  // "berhenti" → loop() terus berulang sangat cepat sehingga Serial.available()
  // selalu diperiksa → perintah baru bisa diterima kapan saja, termasuk saat
  // LED sedang dalam mode blink.
}
```

---

### 4. Tentukan apakah menggunakan delay() atau milis()! Jelaskan pengaruhnya terhadap sistem
Sebaiknya menggunakan millis(). Fungsi `delay()` memblokir seluruh eksekusi program selama durasi yang ditentukan, artinya selama LED berkedip, program tidak bisa membaca input baru dari Serial Monitor sama sekali. Akibatnya, perintah '1' atau '0' yang dikirim saat mode blink aktif akan diabaikan hingga delay selesai. Sebaliknya, `millis()` bersifat non-blocking, program terus berputar di `loop()` dengan sangat cepat, sehingga `Serial.available()` selalu diperiksa dan perintah baru bisa direspons kapan saja, termasuk di tengah-tengah siklus kedip.

---

## 3.6 Pertanyaan Praktikum
### 1. Jelaskan bagaimana cara kerja komunikasi I2C antara Arduino dan LCD pada rangkaian tersebut! 
Arduino (sebagai master) menginisialisasi bus I2C melalui library `Wire.h.` Saat hendak mengirim data ke LCD, Arduino mengeluarkan sinyal START pada bus SDA/SCL, lalu mengirimkan alamat 7-bit LCD (0x27) beserta bit R/W (Write = 0). LCD (sebagai slave) yang mengenali alamatnya merespons dengan sinyal ACK. Arduino kemudian mengirimkan data atau perintah byte demi byte melalui jalur SDA yang disinkronisasi oleh sinyal clock pada SCL. Setelah selesai, Arduino mengeluarkan sinyal STOP. Seluruh proses ini dikelola secara transparan oleh library `LiquidCrystal_I2C` sehingga programmer cukup memanggil `lcd.print()`.

---

### 2. Apakah pin potensiometer harus seperti itu? Jelaskan yang terjadi apabila pin kiri dan pin kanan tertukar! 
Pin potensiometer harus tepat. Jika kaki kiri (GND) dan kaki kanan (5V) tertukar, tegangan pada kaki tengah (output) akan bergerak terbalik: saat diputar ke kiri nilai ADC akan mendekati maksimum (1023) dan saat diputar ke kanan akan mendekati minimum (0). Akibatnya, bar level di LCD akan bergerak berlawanan arah dengan putaran potensiometer, semakin diputar ke kanan, bar justru semakin pendek.

---

### 3. Modifikasi gabungan UART + I2C + README.md:

```cpp
#include <Wire.h>               // Library untuk komunikasi I2C
#include <LiquidCrystal_I2C.h>  // Library untuk LCD dengan modul I2C backpack
#include <Arduino.h>

// ── 1. Inisialisasi objek LCD ────────────────────────────────────────────────
// Parameter: alamat I2C (0x27), jumlah kolom (16), jumlah baris (2)
// Ganti 0x27 dengan 0x20 jika LCD tidak tampil (tergantung chip backpack)
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ── 2. Pin potensiometer ─────────────────────────────────────────────────────
// Kaki tengah potensiometer terhubung ke A0 sebagai input analog
const int pinPot = A0;

// ── 3. setup() ───────────────────────────────────────────────────────────────
void setup() {
  // Inisialisasi komunikasi Serial (UART) dengan baud rate 9600 bps
  // Data akan dikirim ke Serial Monitor melalui USB
  Serial.begin(9600);

  // Inisialisasi LCD: reset, set mode 4-bit via I2C backpack
  lcd.init();

  // Nyalakan backlight LCD agar layar terlihat
  lcd.backlight();
}

// ── 4. loop() ────────────────────────────────────────────────────────────────
void loop() {

  // ── Baca nilai ADC dari potensiometer ────────────────────────────────────
  // analogRead() mengembalikan nilai 0–1023 (ADC 10-bit)
  int nilai = analogRead(pinPot);

  // ── Hitung nilai turunan ──────────────────────────────────────────────────
  // Konversi ADC ke tegangan: V = (ADC / 1023) × 5.0
  float volt = (nilai / 1023.0) * 5.0;

  // Konversi ADC ke persentase: persen = (ADC / 1023) × 100
  int persen = (int)((nilai / 1023.0) * 100);

  // Mapping nilai ADC (0–1023) ke panjang bar (0–16 karakter)
  // map(value, fromLow, fromHigh, toLow, toHigh)
  int panjangBar = map(nilai, 0, 1023, 0, 16);

  // ── Output 1: Serial Monitor (UART) ──────────────────────────────────────
  // Tampilkan data dalam satu baris dengan format yang diminta
  Serial.print("ADC: ");
  Serial.print(nilai);           // Nilai ADC mentah (0–1023)
  Serial.print("  ");
  Serial.print(persen);          // Nilai persentase
  Serial.print("%  |  ");
  Serial.print("Volt: ");
  Serial.print(volt, 2);         // Tegangan dalam Volt (2 desimal)
  Serial.print(" V  |  ");
  Serial.print("Persen: ");
  Serial.print(persen);
  Serial.println("%");           // Cetak + newline, pindah baris berikutnya

  // ── Output 2: LCD I2C – Baris 1 (nilai ADC dan persentase) ───────────────
  // setCursor(kolom, baris): mulai dari kolom 0, baris 0 (baris pertama)
  lcd.setCursor(0, 0);
  lcd.print("ADC:");             // Label tetap
  lcd.print(nilai);              // Nilai ADC
  lcd.print(" ");
  lcd.print(persen);             // Persentase
  lcd.print("%   ");             // Spasi untuk menghapus sisa karakter lama

  // ── Output 2: LCD I2C – Baris 2 (bar level) ──────────────────────────────
  // setCursor(kolom, baris): mulai dari kolom 0, baris 1 (baris kedua)
  lcd.setCursor(0, 1);

  // Cetak karakter blok (ASCII 255) sebanyak panjangBar
  for (int i = 0; i < 16; i++) {
    if (i < panjangBar) {
      lcd.print((char)255);      // Karakter blok penuh = bagian bar yang aktif
    } else {
      lcd.print(" ");            // Spasi = bagian bar yang kosong
    }
  }

  // ── Delay sebelum pembacaan berikutnya ───────────────────────────────────
  // 200ms = LCD diperbarui 5 kali per detik, cukup responsif tanpa flicker
  delay(200);
}
```
---

### 4. Lengkapi table berikut berdasarkan pengamatan pada Serial Monitor 
| ADC | Volt (V) | Persen (%) |
| :-: | :------: | :--------: |
| 1   | 0.00 V   | 0%         |
| 21  | 0.10 V   | 2%         |
| 49  | 0.24 V   | 5%         |
| 74  | 0.36 V   | 7%         |
| 96  | 0.47 V   | 9%         |

---

## 3.7 Pertanyaan Analisis
### 1. Sebutkan dan jelaskan keuntungan dan kerugian menggunakan UART dan I2C!
- UART, keuntungan: sangat sederhana, tidak perlu alamat, komunikasi langsung dua arah (full duplex), cocok untuk debug via Serial Monitor. Kerugian: hanya point-to-point (dua perangkat), tidak ada konfirmasi penerimaan (ACK), setiap koneksi butuh jalur TX dan RX tersendiri.
- I2C, keuntungan: banyak perangkat (multi-slave) pada hanya dua jalur (SDA dan SCL), setiap slave memiliki alamat unik, ada mekanisme ACK untuk konfirmasi. Kerugian: lebih lambat dari SPI, rawan konflik jika dua perangkat memiliki alamat sama, membutuhkan library tambahan.

---

### 2. Bagaimana peran alamat I2C pada LCD (misalnya 0x27 vs 0x20)? Berikan penjelasan! 
Alamat I2C berfungsi seperti nomor rumah di sebuah jalan. Arduino mengirimkan alamat tujuan di awal setiap transmisi, hanya perangkat dengan alamat yang cocok yang merespons, perangkat lain diam. LCD dengan chip backpack NXP menggunakan alamat 0x27, sedangkan chip TI menggunakan 0x20. Jika alamat di kode tidak sesuai dengan alamat fisik LCD, maka tidak ada slave yang merespons dan layar LCD tidak menampilkan apapun.

---

### 3. Jika UART dan I2C digabung dalam satu sistem (misalnya input dari Serial Monitor, output ke LCD), bagaimana alur kerja sistem tersebut? Bagaimana Arduino mengelola dua protokol sekaligus?
Input dari Serial Monitor diterima Arduino melalui hardware USART (UART) via kabel USB → diproses di `loop()` → hasilnya dikirimkan ke dua output bersamaan: Serial Monitor melalui `Serial.println()` (UART) dan LCD melalui `lcd.print()` (I2C via hardware TWI). Arduino dapat mengelola keduanya secara bersamaan karena UART dan I2C menggunakan hardware yang sepenuhnya terpisah, UART menggunakan modul USART pada ATmega328P, sementara I2C menggunakan modul TWI (Two-Wire Interface), sehingga keduanya tidak saling mengganggu dan dapat berjalan dalam satu `loop()` yang sama.





