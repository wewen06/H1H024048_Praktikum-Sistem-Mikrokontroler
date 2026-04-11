Nama        : Wendy Virtus  
NIM         : H1H024048  
Shift Awal  : B  
Shift Akhir : B  

---

## 2.5 Pertanyaan Praktikum

### 1. Gambarkan rangkaian schematic yang digunakan pada percobaan!
![Schematic](https://github.com/user-attachments/assets/5364bd69-553a-42a6-8993-76464487624b)

---

### 2. Apa yang terjadi jika nilai `num` lebih dari 15?
Program akan mengakses indeks array `digitPattern[num]` yang berada di luar batas array (karena array hanya berisi indeks 0–15). Hal ini disebut **array out-of-bounds**.

Akibatnya:
- Program membaca nilai memori yang tidak terdefinisi  
- Tampilan Seven Segment menjadi acak/tidak terprediksi  
- Dalam kondisi ekstrem dapat menyebabkan program crash  

---

### 3. Apakah program ini menggunakan common cathode atau common anode? Jelaskan!
Program ini menggunakan konfigurasi **common cathode**.

Alasannya:
- Nilai `1` (HIGH) digunakan untuk menyalakan segmen  
- Pada common cathode, semua kaki katoda terhubung ke GND  
- LED menyala saat Arduino memberikan tegangan HIGH  
- Arus mengalir dari pin → resistor → LED → GND  

---

### 4. Modifikasi program agar tampilan berjalan dari F ke 0

#### Kode Program
```cpp
#include <Arduino.h>

// ── 1. Definisi pin segmen ────────────────────────────────────────────────
// Array ini memetakan segmen (a,b,c,d,e,f,g,dp) ke pin Arduino
// Urutan: a=7, b=6, c=5, d=11, e=10, f=8, g=9, dp=4
const int segmentPins[8] = {7, 6, 5, 11, 10, 8, 9, 4};

// ── 2. Tabel pola bit karakter 0–F ───────────────────────────────────────
// Format: {a, b, c, d, e, f, g, dp}
// 1 = ON (HIGH), 0 = OFF (LOW)
byte digitPattern[16][8] = {
  {1,1,1,1,1,1,0,0},  // 0
  {0,1,1,0,0,0,0,0},  // 1
  {1,1,0,1,1,0,1,0},  // 2
  {1,1,1,1,0,0,1,0},  // 3
  {0,1,1,0,0,1,1,0},  // 4
  {1,0,1,1,0,1,1,0},  // 5
  {1,0,1,1,1,1,1,0},  // 6
  {1,1,1,0,0,0,0,0},  // 7
  {1,1,1,1,1,1,1,0},  // 8
  {1,1,1,1,0,1,1,0},  // 9
  {1,1,1,0,1,1,1,0},  // A
  {0,0,1,1,1,1,1,0},  // b
  {1,0,0,1,1,1,0,0},  // C
  {0,1,1,1,1,0,1,0},  // d
  {1,0,0,1,1,1,1,0},  // E
  {1,0,0,0,1,1,1,0}   // F
};

// ── 3. Fungsi untuk menampilkan angka ────────────────────────────────────
void displayDigit(int num) {
  for (int i = 0; i < 8; i++) {
    digitalWrite(segmentPins[i], digitPattern[num][i]);
  }
}

// ── 4. Setup ─────────────────────────────────────────────────────────────
void setup() {
  for (int i = 0; i < 8; i++) {
    pinMode(segmentPins[i], OUTPUT);
  }
}

// ── 5. Loop utama ────────────────────────────────────────────────────────
void loop() {
  // Menampilkan dari F (15) ke 0
  for (int i = 15; i >= 0; i--) {
    displayDigit(i);
    delay(1000);
  }
}
```

#### Penjelasan Singkat
- `segmentPins` → menentukan pin untuk tiap segmen  
- `digitPattern` → pola nyala LED untuk angka 0–F  
- `displayDigit()` → menampilkan angka ke seven segment  
- `setup()` → inisialisasi pin sebagai OUTPUT  
- `loop()` → menampilkan angka dari **F ke 0** secara berulang  

---

## 2.6 Pertanyaan Praktikum

### 1. Gambarkan rangkaian schematic yang digunakan pada percobaan!
<img width="981" height="716" alt="image" src="https://github.com/user-attachments/assets/ac4f81a8-7c49-4836-82fc-35136cdd2223" />

---

### 2. Mengapa pada push button digunakan mode INPUT_PULLUP pada Arduino Uno? Apa keuntungannya dibandingkan rangkaian biasa? 
Mode INPUT_PULLUP mengaktifkan resistor pull-up internal Arduino sehingga pin membaca HIGH secara default saat tombol tidak ditekan, dan LOW saat tombol ditekan (terhubung ke GND). Keuntungannya dibanding rangkaian biasa adalah tidak memerlukan resistor pull-up eksternal tambahan, sehingga rangkaian lebih sederhana, hemat komponen, dan lebih stabil terhadap noise.

---

### 3. Jika salah satu LED segmen tidak menyala, apa saja kemungkinan penyebabnya dari sisi hardware maupun software?
- Dari sisi hardware: kabel jumper lepas atau salah sambung, resistor 220Ω putus atau nilainya salah, LED segmen rusak, atau kaki common cathode tidak terhubung ke GND dengan benar.
- Dari sisi software: nilai bit pada array digitPattern salah (bit terbalik), nomor pin pada array segmentPins[] tidak sesuai wiring fisik, atau pin tidak diinisialisasi sebagai OUTPUT di dalam setup().

---

### 4. Modifikasi rangkaian dan program dengan dua push button yang berfungsi sebagai penambahan (increment) dan pengurangan (decrement) pada sistem counter dan berikan penjelasan disetiap baris kode nya dalam bentuk README.md!

#### Kode Program
```cpp
#include <Arduino.h>

// ── 1. Pin mapping Seven Segment ────────────────────────────────────────────
// Urutan: a=7, b=6, c=5, d=11, e=10, f=8, g=9, dp=4
const int segmentPins[8] = {7, 6, 5, 11, 10, 8, 9, 4};

// ── 2. Pin Push Button ───────────────────────────────────────────────────────
const int btnUp   = 2;   // tombol INCREMENT (tambah)
const int btnDown = 3;   // tombol DECREMENT (kurang)

// ── 3. Tabel pola bit karakter 0–F ──────────────────────────────────────────
// {a, b, c, d, e, f, g, dp} → 1=menyala, 0=mati
byte digitPattern[16][8] = {
  {1,1,1,1,1,1,0,0},  // 0
  {0,1,1,0,0,0,0,0},  // 1
  {1,1,0,1,1,0,1,0},  // 2
  {1,1,1,1,0,0,1,0},  // 3
  {0,1,1,0,0,1,1,0},  // 4
  {1,0,1,1,0,1,1,0},  // 5
  {1,0,1,1,1,1,1,0},  // 6
  {1,1,1,0,0,0,0,0},  // 7
  {1,1,1,1,1,1,1,0},  // 8
  {1,1,1,1,0,1,1,0},  // 9
  {1,1,1,0,1,1,1,0},  // A
  {0,0,1,1,1,1,1,0},  // b
  {1,0,0,1,1,1,0,0},  // C
  {0,1,1,1,1,0,1,0},  // d
  {1,0,0,1,1,1,1,0},  // E
  {1,0,0,0,1,1,1,0}   // F
};

// ── 4. Variabel state counter dan edge detection ─────────────────────────────
int currentDigit   = 0;      // nilai counter saat ini (0–15)
bool lastUpState   = HIGH;   // state btnUp pada iterasi sebelumnya
bool lastDownState = HIGH;   // state btnDown pada iterasi sebelumnya

// ── 5. Fungsi displayDigit ───────────────────────────────────────────────────
// Mengirim pola bit karakter ke-num ke semua pin segmen
void displayDigit(int num) {
  for (int i = 0; i < 8; i++) {
    digitalWrite(segmentPins[i], digitPattern[num][i]);
  }
}

// ── 6. setup() ───────────────────────────────────────────────────────────────
void setup() {
  // Set 8 pin segmen sebagai OUTPUT
  for (int i = 0; i < 8; i++) {
    pinMode(segmentPins[i], OUTPUT);
  }

  // Set kedua pin tombol sebagai INPUT_PULLUP
  // → pin membaca HIGH saat tombol tidak ditekan, LOW saat ditekan
  pinMode(btnUp,   INPUT_PULLUP);
  pinMode(btnDown, INPUT_PULLUP);

  // Tampilkan nilai awal (0) saat program pertama kali jalan
  displayDigit(currentDigit);
}

// ── 7. loop() ────────────────────────────────────────────────────────────────
void loop() {
  // Baca state kedua tombol di iterasi ini
  bool upState   = digitalRead(btnUp);
  bool downState = digitalRead(btnDown);

  // ── Deteksi penekanan btnUp (falling edge: HIGH → LOW) ──
  // Kondisi: sebelumnya HIGH dan sekarang LOW = tombol baru ditekan
  if (lastUpState == HIGH && upState == LOW) {
    currentDigit++;                          // tambah nilai counter
    if (currentDigit > 15) currentDigit = 0; // wrap-around: setelah F kembali ke 0
    displayDigit(currentDigit);              // perbarui tampilan
  }

  // ── Deteksi penekanan btnDown (falling edge: HIGH → LOW) ──
  if (lastDownState == HIGH && downState == LOW) {
    currentDigit--;                           // kurangi nilai counter
    if (currentDigit < 0) currentDigit = 15; // wrap-around: sebelum 0 kembali ke F
    displayDigit(currentDigit);               // perbarui tampilan
  }

  // Simpan state saat ini sebagai state "sebelumnya" untuk iterasi berikutnya
  lastUpState   = upState;
  lastDownState = downState;
}
```

---

## 2.7 Pertanyaan Analisis

### 1. Uraikan hasil tugas pada praktikum yang telah dilakukan pada setiap percobaan!
- Pada Percobaan 2A, sistem berhasil menampilkan karakter heksadesimal 0 sampai F pada Seven Segment secara otomatis menggunakan array pola bit (`digitPattern`) dan perulangan `for loop` dengan delay 1 detik. Tampilan berjalan berurutan tanpa error dan sesuai dengan spesifikasi modul.
- Pada Percobaan 2B, sistem berhasil mengintegrasikan Push Button sebagai input digital untuk mengendalikan counter secara manual. Setiap penekanan tombol akan menaikkan nilai angka pada Seven Segment dari 0 hingga F, kemudian kembali ke 0.
- Selain itu, penggunaan teknik edge detection terbukti efektif untuk mencegah pembacaan ganda akibat bouncing pada tombol, sehingga sistem berjalan stabil tanpa glitch.

---

### 2. Bagaimana prinsip kerja dari Seven Segment Display dalam menampilkan angka dan karakter? 
- Seven Segment Display terdiri dari 7 segmen LED (a–g) dan satu titik desimal (dp) yang dapat dikendalikan secara individual melalui pin GPIO Arduino.
- Setiap karakter (angka 0–9 dan huruf A–F) ditampilkan dengan cara menyalakan kombinasi segmen tertentu sesuai pola yang diinginkan.
- Pola tersebut disimpan dalam array `digitPattern`, di mana setiap elemen berisi nilai biner (1 dan 0) untuk menentukan segmen mana yang menyala atau mati.
- Pada konfigurasi common cathode, segmen akan menyala ketika diberikan logika HIGH (1), karena arus mengalir dari pin Arduino menuju GND melalui LED.
- Dengan mengirimkan pola bit ke masing-masing pin segmen menggunakan `digitalWrite()`, maka karakter dapat ditampilkan sesuai dengan tabel pola yang telah ditentukan.

---

### 3. Jelaskan bagaimana sistem counter bekerja pada program tersebut! 
- Sistem counter bekerja dengan dua metode berbeda sesuai percobaan:
  - Percobaan 2A (Otomatis):
    Counter berjalan menggunakan perulangan `for loop` dari 0 hingga 15 (0–F) dengan delay 1 detik. Setiap iterasi akan memanggil fungsi `displayDigit()` untuk memperbarui tampilan Seven Segment secara berurutan.
  - Percobaan 2B (Manual):
    Counter dikendalikan oleh Push Button menggunakan teknik edge detection. Saat terjadi transisi dari HIGH ke LOW (tombol ditekan), nilai counter (`currentDigit`) akan bertambah satu.
- Jika nilai counter melebihi 15, maka akan di-reset ke 0, sehingga membentuk siklus 0 → F → kembali ke 0.
- Setiap perubahan nilai counter langsung diikuti dengan pemanggilan fungsi `displayDigit()` sehingga tampilan selalu sesuai dengan nilai terbaru.

---
