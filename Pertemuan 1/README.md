Nama        : Wendy Virtus
NIM         : H1H024048
Shift Awal  : B
Shift Akhir : B

1.5 Pertanyaan Praktikum
1. Pada kondisi apa program masuk ke blok if?
   Program masuk ke blok if ketika nilai timeDelay <= 100. Artinya, setelah LED berkedip cukup cepat (delay sudah berkurang dari 1000ms hingga mencapai 100ms atau kurang), program akan masuk ke blok tersebut — menjalankan jeda 3 detik lalu mereset timeDelay kembali ke 1000.
2. Pada kondisi apa program masuk ke blok else?
   Program masuk ke blok else ketika timeDelay > 100. Selama kondisi itu terpenuhi, setiap akhir siklus kedip akan mengurangi timeDelay sebesar 100ms, sehingga LED semakin cepat berkedip dari waktu ke waktu.
3. Apa fungsi dari perintah delay(timeDelay)?
   delay(timeDelay) berfungsi untuk menghentikan eksekusi program selama timeDelay milidetik. Digunakan dua kali dalam satu siklus: pertama saat LED menyala, kedua saat LED mati — sehingga lamanya LED ON dan OFF sama persis, dan bersama-sama menentukan kecepatan kedip.
4. Jika program yang dibuat memiliki alur mati → lambat → cepat → reset (mati), ubah menjadi LED tidak langsung reset → tetapi berubah dari cepat → sedang → mati dan berikan penjelasan disetiap baris kode nya dalam bentuk README.md!

   const int ledPin = 6; // LED ada di pin 6 pada Arduino 
   int timeDelay = 1000; // jeda 1 detik

   void setup() {
     pinMode(ledPin, OUTPUT); // Mengatur pin LED sebagai output
   }

   void loop() {
     // Nyalakan LED
     digitalWrite(ledPin, HIGH);
     delay(timeDelay); // LED menyala sesuai waktu delay

     // Matikan LED
     digitalWrite(ledPin, LOW); delay(timeDelay); // LED mati sesuai waktu delay

     // Logika perubahan kecepatan
     if (timeDelay <= 100) {
       // Jika sudah sangat cepat
       timeDelay = 500; // ubah ke kecepatan sedang
     }
     else if (timeDelay == 500) {
       // Jika sudah kecepatan sedang
       digitalWrite(ledPin, LOW); // pastikan LED mati
       delay(3000); // jeda 3 detik (mati)
       timeDelay = 1000; // kembali ke awal (lambat)
     }
     else {
       // Jika masih dalam proses percepatan
       timeDelay -= 100; // kurangi delay (lebih cepat)
     }
   }

1.6 Pertanyaan Praktikum
1. Schematic 5 LED Running
   
   <img width="413" height="529" alt="image" src="https://github.com/user-attachments/assets/ee0820a2-2642-4945-877f-19b1ff5a8cae" />
   
2. Bagaimana program membuat efek LED berjalan kiri ke kanan?
   Loop for pertama menjalankan ledPin dari 2 naik ke 7 (ledPin++). Setiap iterasi menyalakan satu pin dengan digitalWrite(ledPin, HIGH), menunggu timer ms, lalu mematikannya. Karena hanya satu LED menyala pada satu waktu dan urutannya naik, efek visualnya adalah cahaya berjalan dari kiri (pin 2) ke kanan (pin 7).
3. Bagaimana program membuat LED kembali dari kanan ke kiri?
   Loop for kedua menjalankan ledPin dari 7 turun ke 2 (ledPin--). Dengan urutan terbalik, LED yang menyala berpindah dari pin 7 kembali ke pin 2 — sehingga efeknya adalah LED berjalan dari kanan ke kiri, menciptakan gerakan bolak-balik seperti efek Knight Rider.
4. Buatkan program agar LED menyala tiga LED kanan dan tiga LED kiri secara bergantian dan berikan penjelasan disetiap baris kode nya dalam bentuk README.md!

   int timer = 500; // waktu delay (semakin besar semakin lambat)

   void setup() {
     // inisialisasi pin 2 sampai 7 sebagai output
     for (int ledPin = 2; ledPin < 8; ledPin++) {
       pinMode(ledPin, OUTPUT); // set pin sebagai output
     }
   }

   void loop() {
     // nyalakan 3 led kiri
     digitalWrite(2, HIGH); // LED kiri 1 nyala
     digitalWrite(3, HIGH); // LED kiri 2 nyala
     digitalWrite(4, HIGH); // LED kiri 3 nyala
     digitalWrite(5, LOW); // LED kanan mati
     digitalWrite(6, LOW);
     digitalWrite(7, LOW);
     delay(timer); // tunggu

     // nyalakan 3 led kanan
     digitalWrite(2, LOW); // LED kiri mati
     digitalWrite(3, LOW);
     digitalWrite(4, LOW);
     digitalWrite(5, HIGH); // LED kanan 1 nyala
     digitalWrite(6, HIGH); // LED kanan 2 nyala
     digitalWrite(7, HIGH); // LED kanan 3 nyala
     delay(timer); // tunggu
   }

1.7 Pertanyaan Analisis
1. Uraian Hasil Setiap Percobaan
   - Percobaan 1 — LED Blink dengan Percepatan Bertahap. Program menggunakan satu LED pada pin 6 dengan variabel timeDelay yang berkurang 100 ms setiap siklus melalui blok else, sehingga LED berkedip semakin cepat dari 1000 ms hingga 100 ms, lalu jeda 3 detik dan reset. Pada modifikasi, logika dibalik: timeDelay dimulai dari 100 ms dan bertambah 100 ms tiap siklus hingga mencapai 1000 ms, menghasilkan pola visual Cepat → Sedang → Mati → Reset.
   - Percobaan 2 — LED Running. Program menggunakan 6 LED pada pin 2–7 yang diinisialisasi sekaligus dengan satu loop for. Dua loop berurutan — satu menaik, satu menurun — menciptakan efek titik cahaya berjalan maju mundur. Pada modifikasi, LED dibagi menjadi kelompok kiri (pin 2–4) dan kanan (pin 5–7) yang dinyalakan bergantian secara bersamaan per kelompok, menghasilkan efek dua blok cahaya yang bergantian menyala kiri dan kanan.
2. Pengaruh Struktur Perulangan terhadap Jalannya Program
   Struktur perulangan for dan while memungkinkan eksekusi instruksi berulang secara efisien tanpa menulis kode yang sama berkali-kali. Dalam praktikum ini, loop for mempersingkat inisialisasi enam pin menjadi tiga baris kode saja, sekaligus menentukan perilaku fisik LED melalui arah iterasinya — loop menaik menggerakkan LED ke kanan, loop menurun membaliknya ke kiri. Perbedaan mendasarnya, for dipakai saat jumlah iterasi sudah diketahui (rentang pin pasti), sedangkan while lebih tepat untuk kondisi yang berubah secara dinamis seperti pembacaan sensor. Secara keseluruhan, perulangan adalah mekanisme utama yang membuat program Arduino berjalan kontinu dan terstruktur.
3. Cara Kerja Percabangan (if-else) dalam Menentukan Kondisi LED
   Percabangan if-else bekerja sebagai pengambil keputusan dengan mengevaluasi kondisi boolean, lalu mengeksekusi salah satu dari dua blok kode sesuai hasilnya. Dalam praktikum ini, if (timeDelay <= 100) dievaluasi setiap akhir siklus kedip — jika terpenuhi, program mereset timeDelay; jika tidak, program menguranginya 100 ms. Penempatannya setelah siklus selesai memastikan tidak ada kedip yang terpotong di tengah jalan. Secara umum, if-else menjadi jembatan antara data masukan (nilai variabel atau sensor) dan aksi keluaran (nyala/mati LED), sehingga Arduino dapat merespons kondisi yang berbeda dengan perilaku yang berbeda pula.
4. Kombinasi Perulangan dan Percabangan untuk Sistem yang Responsif
   Perulangan dan percabangan saling melengkapi dalam membangun sistem Arduino yang responsif: perulangan menjamin program terus memantau kondisi secara kontinu, sementara percabangan memastikan reaksi yang tepat saat kondisi berubah. Dalam praktikum ini kombinasi keduanya sudah terlihat — loop for mengurus eksekusi LED secara berurutan, sementara if-else mengelola logika perubahan kecepatan. Pada sistem yang lebih kompleks seperti lampu otomatis berbasis LDR atau alarm suhu, pola yang sama diterapkan: loop membaca nilai sensor secara periodik, if-else memutuskan aksi berdasarkan nilai tersebut. Agar sistem benar-benar responsif, penggunaan millis() lebih disarankan daripada delay(), karena delay() memblokir pembacaan input selama jeda berlangsung, sedangkan millis() memungkinkan program memantau kondisi dan mengelola waktu secara bersamaan tanpa jeda paksa.
