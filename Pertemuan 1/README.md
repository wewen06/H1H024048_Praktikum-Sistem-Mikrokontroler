Nama        : Wendy Virtus  
NIM         : H1H024048  
Shift Awal  : B  
Shift Akhir : B  

---

## 1.5 Pertanyaan Praktikum

### 1. Pada kondisi apa program masuk ke blok `if`?
Program masuk ke blok `if` ketika nilai `timeDelay <= 100`.  
Artinya, setelah LED berkedip cukup cepat (delay berkurang dari 1000 ms hingga mencapai 100 ms atau kurang), program akan menjalankan jeda 3 detik lalu mereset `timeDelay` kembali ke 1000.

---

### 2. Pada kondisi apa program masuk ke blok `else`?
Program masuk ke blok `else` ketika `timeDelay > 100`.  
Selama kondisi ini terpenuhi, setiap akhir siklus kedip akan mengurangi `timeDelay` sebesar 100 ms, sehingga LED semakin cepat berkedip.

---

### 3. Apa fungsi dari perintah `delay(timeDelay)`?
`delay(timeDelay)` berfungsi untuk menghentikan eksekusi program selama `timeDelay` milidetik.

Digunakan dua kali dalam satu siklus:
- Saat LED menyala  
- Saat LED mati  

Sehingga durasi ON dan OFF sama, dan menentukan kecepatan kedip.

---

### 4. Modifikasi Program
LED tidak langsung reset, tetapi berubah dari **cepat → sedang → mati**
```cpp
const int ledPin = 6; 
int timeDelay = 100;   // mulai dari cepat

void setup() { 
  pinMode(ledPin, OUTPUT); 
} 

void loop() { 
  // Nyalakan LED
  digitalWrite(ledPin, HIGH); 
  delay(timeDelay); 

  // Matikan LED
  digitalWrite(ledPin, LOW); 
  delay(timeDelay); 

  // Perubahan delay: dari cepat → sedang → lambat
  if (timeDelay >= 1000) { 
    // Kondisi mati (berhenti sejenak)
    delay(3000);        
    timeDelay = 100;    // kembali ke cepat
  } else { 
    timeDelay += 100;   // perlambatan bertahap
  } 
}
```

---

## 1.6 Pertanyaan Praktikum

### 1. Schematic 5 LED Running
<img width="728" height="562" alt="image" src="https://github.com/user-attachments/assets/4a590b60-d295-415c-93f1-95bcb13c621e" />

---

### 2. Bagaimana program membuat efek LED berjalan kiri ke kanan?
Loop `for` pertama menjalankan `ledPin` dari 2 hingga 7 (`ledPin++`).

Setiap iterasi:
- Menyalakan LED (`HIGH`)  
- Menunggu beberapa ms  
- Mematikan LED  

Karena urutannya naik, efeknya LED bergerak dari kiri ke kanan.

---

### 3. Bagaimana program membuat LED kembali dari kanan ke kiri?
Loop `for` kedua menjalankan `ledPin` dari 7 ke 2 (`ledPin--`).

Urutan terbalik ini membuat LED bergerak dari kanan ke kiri, menghasilkan efek bolak-balik seperti **Knight Rider**.

---

### 4. Modifikasi Program
LED menyala tiga di kanan dan tiga di kiri secara bergantian
```cpp
// Variabel untuk mengatur kecepatan kedip (dalam milidetik)
// Semakin besar nilainya, semakin lambat pergantian LED
int timer = 500;

void setup() {
  // Gunakan loop for untuk menginisialisasi semua pin 2 sampai 7 sebagai OUTPUT
  // OUTPUT berarti Arduino akan mengirim tegangan (menyalakan/mematikan LED)
  for (int ledPin = 2; ledPin <= 7; ledPin++) {
    pinMode(ledPin, OUTPUT);
  }
}

void loop() {

  // ════════════════════════════════════════
  // FASE 1: Nyalakan LED KIRI (pin 2, 3, 4)
  // ════════════════════════════════════════

  // Loop untuk menyalakan tiga LED di sisi kiri (pin 2 sampai 4)
  for (int ledPin = 2; ledPin <= 4; ledPin++) {
    // Kirim sinyal HIGH (5V) ke pin → LED menyala
    digitalWrite(ledPin, HIGH);
  }

  // Pastikan LED kanan (pin 5, 6, 7) dalam kondisi mati
  // agar tidak ada tumpang tindih saat fase pergantian
  for (int ledPin = 5; ledPin <= 7; ledPin++) {
    // Kirim sinyal LOW (0V) ke pin → LED mati
    digitalWrite(ledPin, LOW);
  }

  // Tahan kondisi ini selama 'timer' milidetik
  // LED kiri menyala, LED kanan mati, selama durasi ini
  delay(timer);

  // ════════════════════════════════════════
  // FASE 2: Nyalakan LED KANAN (pin 5, 6, 7)
  // ════════════════════════════════════════

  // Loop untuk menyalakan tiga LED di sisi kanan (pin 5 sampai 7)
  for (int ledPin = 5; ledPin <= 7; ledPin++) {
    // Kirim sinyal HIGH (5V) ke pin → LED menyala
    digitalWrite(ledPin, HIGH);
  }

  // Matikan LED kiri (pin 2, 3, 4) agar tidak menyala bersamaan
  for (int ledPin = 2; ledPin <= 4; ledPin++) {
    // Kirim sinyal LOW (0V) ke pin → LED mati
    digitalWrite(ledPin, LOW);
  }

  // Tahan kondisi ini selama 'timer' milidetik
  // LED kanan menyala, LED kiri mati, selama durasi ini
  delay(timer);

  // Setelah delay selesai, loop() akan dipanggil ulang otomatis oleh Arduino
  // → kembali ke FASE 1 → efek bergantian berjalan terus-menerus
}
```
---

## 1.7 Pertanyaan Analisis

### 1. Uraian Hasil Setiap Percobaan
- **Percobaan 1 — LED Blink dengan Percepatan Bertahap**  
  Program menggunakan satu LED pada pin 6 dengan variabel `timeDelay` yang berkurang 100 ms setiap siklus, sehingga LED berkedip semakin cepat dari 1000 ms hingga 100 ms, lalu reset.  
  Pada modifikasi, `timeDelay` dimulai dari 100 ms dan bertambah hingga 1000 ms, menghasilkan pola **cepat → sedang → mati → reset**.

- **Percobaan 2 — LED Running**  
  Program menggunakan 6 LED pada pin 2–7 dengan dua loop (`naik` dan `turun`) sehingga LED bergerak bolak-balik.  
  Pada modifikasi, LED dibagi menjadi dua kelompok:
  - Kiri (pin 2–4)  
  - Kanan (pin 5–7)  
  Menyala secara bergantian.

---

### 2. Pengaruh Struktur Perulangan terhadap Jalannya Program
Struktur perulangan (`for` dan `while`) memungkinkan eksekusi berulang secara efisien.

- `for` digunakan saat jumlah iterasi sudah pasti (misalnya pin 2–7)  
- `while` digunakan untuk kondisi dinamis  

Loop juga menentukan arah gerakan LED:
- Naik → kiri ke kanan  
- Turun → kanan ke kiri  

---

### 3. Cara Kerja Percabangan (`if-else`)
Percabangan digunakan untuk mengambil keputusan berdasarkan kondisi.

Contoh:
- `if (timeDelay <= 100)` → reset delay  
- `else` → kurangi delay  

Struktur ini memastikan LED tidak terputus di tengah siklus dan tetap berjalan sesuai logika.

---

### 4. Kombinasi Perulangan dan Percabangan
Perulangan dan percabangan saling melengkapi:

- **Loop** → menjalankan program terus-menerus  
- **If-else** → menentukan aksi berdasarkan kondisi  

Dalam praktikum:
- Loop mengatur urutan LED  
- If-else mengatur perubahan kecepatan  

Untuk sistem yang lebih responsif, disarankan menggunakan `millis()` daripada `delay()`, karena:
- `delay()` bersifat blocking  
- `millis()` memungkinkan multitasking (membaca input sambil berjalan)

---
