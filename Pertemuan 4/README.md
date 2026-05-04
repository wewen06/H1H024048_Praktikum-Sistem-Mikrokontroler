Nama: Wendy Virtus  
NIM: H1H024048  
Shift Awal: B  
Shift Akhir: B  

---

## 4.5 Pertanyaan Praktikum
### 1. Apa fungsi perintah analogRead() pada rangkaian praktikum ini?
Fungsi `analogRead(pin)` digunakan untuk membaca nilai tegangan analog yang dihasilkan oleh potensiometer pada pin analog Arduino (contoh: A0). Arduino Uno memiliki ADC 10-bit, sehingga hasil pembacaannya berupa nilai integer dalam rentang 0 hingga 1023, yang merepresentasikan tegangan dari 0V hingga 5V. Pada percobaan ini, `analogRead()` membaca posisi putar potensiometer. Ketika potensiometer diputar, tegangan yang masuk ke pin analog berubah, dan `analogRead()` mengonversi perubahan tegangan tersebut menjadi angka digital yang kemudian diproses untuk menggerakkan servo motor.

```cpp
val = analogRead(potensioPin); // Membaca nilai 0–1023 dari potensiometer
```

---

### 2. Mengapa diperlukan fungsi map() dalam program tersebut?
Fungsi `map()` diperlukan karena rentang nilai ADC dan rentang sudut servo berbeda. `analogRead()` menghasilkan nilai 0–1023, sedangkan servo motor hanya menerima sudut 0°–180°. Tanpa konversi, nilai ADC tidak bisa langsung digunakan untuk menggerakkan servo.
Fungsi `map()` melakukan pemetaan linear dari satu rentang ke rentang lain:

```cpp
pos = map(val, 0, 1023, 0, 180);
// val=0    → pos=0°
// val=511  → pos=90°
// val=1023 → pos=180°
```

Tanpa `map()`, jika nilai 1023 langsung dikirim ke `myservo.write()`, servo bisa bergerak melebihi batas fisiknya dan berpotensi merusak motor.

---

### 3. Modifikasi program berikut agar servo hanya bergerak dalam rentang 30° hingga 150°, meskipun potensiometer tetap memiliki rentang ADC 0–1023. Jelaskan program pada file README.md

```cpp
#include <Servo.h>    // Library untuk mengontrol motor servo via sinyal PWM

// ── 1. Objek Servo ───────────────────────────────────────────────────────────
// Buat objek servo yang akan digunakan untuk mengontrol motor
Servo myservo;

// ── 2. Definisi Pin ──────────────────────────────────────────────────────────
const int potensioPin = A0;  // Pin analog untuk potensiometer (input ADC)
const int servoPin    = 9;   // Pin digital PWM untuk motor servo

// ── 3. Variabel ──────────────────────────────────────────────────────────────
int val = 0;   // Menyimpan nilai ADC dari potensiometer (0–1023)
int pos = 0;   // Menyimpan sudut servo hasil konversi (30–150 derajat)

// ── 4. setup() ───────────────────────────────────────────────────────────────
void setup() {
  // Hubungkan objek servo ke pin fisik servoPin
  myservo.attach(servoPin);

  // Inisialisasi komunikasi Serial untuk menampilkan data di Serial Monitor
  Serial.begin(9600);
}

// ── 5. loop() ────────────────────────────────────────────────────────────────
void loop() {

  // ── Baca nilai ADC dari potensiometer ────────────────────────────────────
  // analogRead() mengambil sampel tegangan pada A0 dan mengembalikan 0–1023
  val = analogRead(potensioPin);

  // ── Konversi ADC ke sudut servo (MODIFIKASI: 30°–150°) ───────────────────
  // map(nilai, dari_min, dari_max, ke_min, ke_max)
  // Nilai ADC 0–1023 dipetakan ke sudut 30–150 (bukan 0–180 seperti semula)
  // Akibatnya: potensiometer paling kiri → servo 30°
  //            potensiometer paling kanan → servo 150°
  pos = map(val, 0, 1023, 30, 150);

  // ── Gerakkan servo ke sudut hasil konversi ────────────────────────────────
  // myservo.write() memerintahkan servo bergerak ke sudut yang ditentukan
  myservo.write(pos);

  // ── Tampilkan data ke Serial Monitor ─────────────────────────────────────
  Serial.print("ADC Potensio: ");
  Serial.print(val);               // Nilai ADC mentah (0–1023)
  Serial.print(" | Sudut Servo: ");
  Serial.println(pos);             // Sudut servo saat ini (30–150 derajat)

  // ── Delay stabilisasi ────────────────────────────────────────────────────
  // Memberi waktu servo bergerak ke posisi baru sebelum pembacaan berikutnya
  // 15ms adalah waktu minimum yang cukup untuk servo SG90 bergerak ~1°
  delay(15);
}
```

---

## 4.6 Pertanyaan Praktikum
### 1. Jelaskan mengapa LED dapat diatur kecerahannya menggunakan fungsi analogWrite()!
LED sejatinya hanya mengenal dua kondisi: menyala (HIGH) atau mati (LOW). Namun dengan teknik PWM (Pulse Width Modulation), Arduino dapat mengatur kecerahan LED secara bertahap.
Cara kerjanya: `analogWrite()` menghasilkan sinyal digital yang menyala dan mati sangat cepat dalam satu periode waktu. Perbandingan waktu HIGH terhadap total periode disebut duty cycle. Karena pergantian ini terlalu cepat untuk ditangkap mata manusia, LED seolah-olah terlihat redup atau terang secara proporsional.

```cpp
Duty cycle 0%   → LED mati total
Duty cycle 50%  → LED setengah terang
Duty cycle 100% → LED terang penuh
```

Contoh dari README:

```cpp
analogWrite(LED_PIN, 127); // ~50% duty cycle → LED setengah terang
analogWrite(LED_PIN, 255); // 100% duty cycle → LED terang penuh
```

---

### 2. Apa hubungan antara nilai ADC (0–1023) dan nilai PWM (0–255)?
Nilai ADC dan nilai PWM memiliki hubungan yang bersifat linear dan dihubungkan melalui fungsi `map()`. ADC pada Arduino memiliki resolusi 10-bit sehingga menghasilkan nilai 0 hingga 1023, sedangkan PWM pada Arduino memiliki resolusi 8-bit sehingga hanya menerima nilai 0 hingga 255. Dengan menggunakan `map(nilaiADC, 0, 1023, 0, 255)`, setiap kenaikan sekitar 4 pada nilai ADC setara dengan kenaikan 1 pada nilai PWM. Hubungan proporsional ini memungkinkan potensiometer mengontrol kecerahan LED secara halus, di mana semakin besar putaran potensiometer maka semakin besar nilai ADC yang terbaca dan semakin terang pula cahaya LED yang dihasilkan.

---

### 3. Modifikasilah program berikut agar LED hanya menyala pada rentang kecerahan sedang, yaitu hanya ketika nilai PWM berada pada rentang 50 sampai 200. Jelaskan program pada file README.md.

```cpp
#include <Arduino.h>  // Library dasar Arduino

// ── 1. Definisi Pin ──────────────────────────────────────────────────────────
const int potPin = A0;   // Pin analog potensiometer (input ADC)
const int ledPin = 9;    // Pin digital PWM untuk LED (~9 pada Arduino Uno)

// ── 2. Variabel ──────────────────────────────────────────────────────────────
int nilaiADC = 0;   // Menyimpan nilai ADC dari potensiometer (0–1023)
int pwm      = 0;   // Menyimpan nilai PWM hasil konversi (0–255)

// ── 3. setup() ───────────────────────────────────────────────────────────────
void setup() {
  // Set pin LED sebagai OUTPUT agar bisa menerima sinyal PWM
  pinMode(ledPin, OUTPUT);

  // Inisialisasi komunikasi Serial 9600 bps untuk Serial Monitor
  Serial.begin(9600);
}

// ── 4. loop() ────────────────────────────────────────────────────────────────
void loop() {

  // ── Baca nilai ADC dari potensiometer ────────────────────────────────────
  // analogRead() mengembalikan 0–1023 sesuai tegangan pada A0
  nilaiADC = analogRead(potPin);

  // ── Konversi ADC ke PWM ───────────────────────────────────────────────────
  // Petakan nilai ADC (0–1023) ke nilai PWM (0–255) secara linear
  // ADC 10-bit → PWM 8-bit: setiap kenaikan ±4 ADC ≈ ±1 PWM
  pwm = map(nilaiADC, 0, 1023, 0, 255);

  // ── MODIFIKASI: Batasi output hanya pada rentang PWM 50–200 ─────────────
  // Kondisi: LED hanya menyala jika pwm >= 50 DAN pwm <= 200
  if (pwm >= 50 && pwm <= 200) {
    // Dalam rentang → kirim nilai PWM ke LED (kecerahan sesuai posisi potensiometer)
    analogWrite(ledPin, pwm);
  } else {
    // Di luar rentang (pwm < 50 atau pwm > 200) → LED dimatikan
    analogWrite(ledPin, 0);
  }

  // ── Tampilkan data ke Serial Monitor ─────────────────────────────────────
  Serial.print("ADC: ");
  Serial.print(nilaiADC);      // Nilai ADC mentah
  Serial.print(" | PWM: ");
  Serial.print(pwm);           // Nilai PWM setelah konversi
  Serial.print(" | Status: ");
  // Tampilkan status LED berdasarkan kondisi rentang
  if (pwm >= 50 && pwm <= 200) {
    Serial.println("ON");
  } else {
    Serial.println("OFF (di luar rentang)");
  }

  // ── Delay stabilisasi sistem ──────────────────────────────────────────────
  // 50ms cukup untuk menstabilkan pembacaan dan tampilan Serial Monitor
  delay(50);
}
```

---

## 4.7 Pertanyaan Analisis
### 1. Bagaimana proses konversi sinyal analog menjadi digital pada Arduino? Jelaskan!
Proses konversi sinyal analog menjadi digital pada Arduino berlangsung melalui beberapa tahap. Pertama adalah tahap sampling, di mana Arduino mengambil sampel tegangan sesaat dari pin analog pada interval waktu tertentu. Kemudian nilai tegangan tersebut ditahan sementara agar stabil selama proses konversi berlangsung. Selanjutnya pada tahap kuantisasi, tegangan yang ditahan dibandingkan dengan tegangan referensi yang secara default bernilai 5V pada Arduino Uno. Karena Arduino Uno menggunakan ADC 10-bit, tegangan 0 hingga 5V dibagi menjadi 1024 tingkatan. Sebagai contoh, tegangan 2.5V akan menghasilkan nilai ADC sekitar 511 karena (2.5 / 5.0) × 1023 = 511. Terakhir, hasil kuantisasi tersebut diubah menjadi nilai integer yang dapat langsung dibaca dan diproses oleh program Arduino.

---

### 2. Faktor apa saja yang dapat mempengaruhi keakuratan pembacaan ADC?
Terdapat beberapa faktor yang dapat mempengaruhi keakuratan pembacaan ADC pada Arduino. Pertama adalah ketidakstabilan tegangan referensi, di mana jika tegangan referensi berfluktuasi maka seluruh skala pembacaan ikut bergeser sehingga hasil menjadi tidak akurat. Kedua adalah noise elektromagnetik yang berasal dari komponen lain di sekitar rangkaian, yang dapat mencemari sinyal analog yang masuk ke pin ADC. Ketiga adalah keterbatasan resolusi ADC itu sendiri, karena Arduino Uno hanya memiliki resolusi 10-bit sehingga ada batas minimum perubahan tegangan yang bisa dideteksi yaitu sekitar 4.9mV per step. Selain itu, impedansi sumber sinyal yang terlalu tinggi dapat memperlambat pengisian kapasitor internal ADC sehingga menghasilkan pembacaan yang tidak tepat. Kualitas fisik komponen seperti potensiometer yang sudah aus atau kotor juga dapat menghasilkan nilai ADC yang tidak stabil, dan perubahan suhu lingkungan turut mempengaruhi karakteristik komponen analog secara keseluruhan.

---

### 3. Apa kendala yang mungkin terjadi saat mengintegrasikan ADC dan PWM dalam satu sistem?
Terdapat beberapa kendala yang mungkin muncul ketika ADC dan PWM digunakan bersamaan dalam satu sistem. Salah satu kendala utama adalah interferensi sinyal, di mana sinyal PWM yang switching dengan cepat dapat menciptakan noise elektromagnetik yang mengganggu pembacaan ADC di pin analog terdekat sehingga nilai yang terbaca menjadi tidak stabil. Selain itu, pengaturan timing dan delay juga menjadi tantangan karena jika delay terlalu besar maka respons sistem menjadi lambat, namun jika terlalu kecil maka ADC tidak memiliki waktu yang cukup untuk membaca nilai yang stabil. Ketidaksesuaian resolusi antara ADC yang menghasilkan 0–1023 dan PWM yang hanya menerima 0–255 juga mengharuskan adanya konversi yang tepat menggunakan `map()`, dan jika konversi ini salah maka output PWM akan keliru. Kendala lain adalah beban berlebih pada pin PWM, di mana jika perangkat yang dikontrol menarik arus terlalu besar maka tegangan supply Arduino bisa drop dan mempengaruhi akurasi ADC. Kombinasi dari berbagai faktor tersebut juga dapat menyebabkan efek jitter, yaitu servo bergetar kecil atau LED berkedip, seperti yang disebutkan dalam spesifikasi percobaan.
