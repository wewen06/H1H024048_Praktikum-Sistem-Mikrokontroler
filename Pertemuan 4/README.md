Nama: Wendy Virtus  
NIM: H1H024048  
Shift Awal: B  
Shift Akhir: B  

---

## 4.5 Pertanyaan Praktikum
### 1. Apa fungsi perintah analogRead() pada rangkaian praktikum ini?

---

### 2. Mengapa diperlukan fungsi map() dalam program tersebut?

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

---

### 2. Apa hubungan antara nilai ADC (0–1023) dan nilai PWM (0–255)?

---

### 3. Modifikasilah program berikut agar LED hanya menyala pada rentang kecerahan sedang, yaitu hanya ketika nilai PWM berada pada rentang 50 sampai 200. Jelaskan program pada file README.md.

---

## 4.7 Pertanyaan Analisis
### 1. Bagaimana proses konversi sinyal analog menjadi digital pada Arduino? Jelaskan!

---

### 2. Faktor apa saja yang dapat mempengaruhi keakuratan pembacaan ADC?

---

### 3. Apa kendala yang mungkin terjadi saat mengintegrasikan ADC dan PWM dalam satu sistem?
