Nama: Wendy Virtus  
NIM: H1H024048  
Shift Awal: B  
Shift Akhir: B  

---

## 5.5 Pertanyaan Praktikum
### 1. Apakah ketiga task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!
Ketiga task tersebut berjalan secara bergantian, bukan benar-benar bersamaan dalam satu waktu. Namun karena perpindahan antar task terjadi sangat cepat, seolah-olah ketiganya berjalan bersamaan (concurrent). Mekanisme ini diatur oleh FreeRTOS Scheduler yang menggunakan metode preemptive scheduling berbasis prioritas. Karena ketiga task (TaskBlink1, TaskBlink2, dan Taskprint) memiliki prioritas yang sama (priority = 1), scheduler akan membagi waktu CPU secara round-robin di antara ketiganya. Ketika suatu task memanggil `vTaskDelay()`, task tersebut akan masuk ke kondisi blocked dan CPU akan langsung diberikan ke task lain yang siap berjalan. Dengan demikian, selama satu task menunggu delay, task lainnya dapat menggunakan CPU, sehingga sistem terasa multitasking.

---

### 2. Bagaimana cara menambahkan task keempat? Jelaskan langkahnya!
Untuk menambahkan task keempat, langkah pertama adalah mendeklarasikan prototipe fungsi task baru di bagian atas program, misalnya `void TaskKeempat(void *pvParameters);`. Selanjutnya, di dalam fungsi `setup()`, ditambahkan pemanggilan `xTaskCreate()` dengan parameter yang sesuai, seperti nama fungsi task, nama string task, ukuran stack, parameter, prioritas, dan handle. Kemudian, fungsi task keempat didefinisikan di bagian bawah program dengan struktur `while(1)` yang berisi logika yang diinginkan dan pemanggilan `vTaskDelay()` agar task tidak memborosi CPU. Pastikan ukuran stack yang dialokasikan cukup untuk kebutuhan task tersebut, dan perhatikan prioritas task agar tidak mengganggu task lain yang lebih kritis.

---

### 3. Modifikasilah program dengan menambah sensor (misalnya potensiometer), lalu gunakan nilainya untuk mengontrol kecepatan LED! Bagaimana hasilnya? Jelaskan program pada file README.md.
<img width="1200" height="1600" alt="image" src="https://github.com/user-attachments/assets/af11a152-7d9e-47ae-940e-bafb070cd2f8" />  

---

```cpp
#include <Arduino_FreeRTOS.h>
#include <semphr.h>

#define POT_PIN  A0
#define LED1_PIN  8
#define LED2_PIN  7

SemaphoreHandle_t xMutex;
volatile int blinkSpeed = 200; // nilai default delay (ms)

void TaskReadPot(void *pvParameters);
void TaskBlink1(void *pvParameters);
void TaskBlink2(void *pvParameters);
void Taskprint(void *pvParameters);

void setup() {
  Serial.begin(9600);
  pinMode(POT_PIN, INPUT);

  xMutex = xSemaphoreCreateMutex();

  xTaskCreate(TaskReadPot, "ReadPot", 128, NULL, 2, NULL); // prioritas lebih tinggi
  xTaskCreate(TaskBlink1,  "task1",   128, NULL, 1, NULL);
  xTaskCreate(TaskBlink2,  "task2",   128, NULL, 1, NULL);
  xTaskCreate(Taskprint,   "task3",   128, NULL, 1, NULL);

  vTaskStartScheduler();
}

void loop() {}

// Task membaca potensiometer dan update blinkSpeed
void TaskReadPot(void *pvParameters) {
  while (1) {
    int potValue = analogRead(POT_PIN);          // 0 - 1023
    int newSpeed = map(potValue, 0, 1023, 50, 1000); // mapping ke 50ms - 1000ms

    if (xSemaphoreTake(xMutex, portMAX_DELAY) == pdTRUE) {
      blinkSpeed = newSpeed;
      xSemaphoreGive(xMutex);
    }

    vTaskDelay(100 / portTICK_PERIOD_MS);
  }
}

// Task LED 1 - kecepatan dipengaruhi potensiometer
void TaskBlink1(void *pvParameters) {
  pinMode(LED1_PIN, OUTPUT);
  while (1) {
    int speed;
    if (xSemaphoreTake(xMutex, portMAX_DELAY) == pdTRUE) {
      speed = blinkSpeed;
      xSemaphoreGive(xMutex);
    }

    Serial.println("Task1 - LED1 Blink");
    digitalWrite(LED1_PIN, HIGH);
    vTaskDelay(speed / portTICK_PERIOD_MS);
    digitalWrite(LED1_PIN, LOW);
    vTaskDelay(speed / portTICK_PERIOD_MS);
  }
}

// Task LED 2 - kecepatan 1.5x lebih lambat dari LED1
void TaskBlink2(void *pvParameters) {
  pinMode(LED2_PIN, OUTPUT);
  while (1) {
    int speed;
    if (xSemaphoreTake(xMutex, portMAX_DELAY) == pdTRUE) {
      speed = blinkSpeed;
      xSemaphoreGive(xMutex);
    }

    Serial.println("Task2 - LED2 Blink");
    digitalWrite(LED2_PIN, HIGH);
    vTaskDelay((speed * 1.5) / portTICK_PERIOD_MS);
    digitalWrite(LED2_PIN, LOW);
    vTaskDelay((speed * 1.5) / portTICK_PERIOD_MS);
  }
}

// Task print counter + tampilkan blinkSpeed saat ini
void Taskprint(void *pvParameters) {
  int counter = 0;
  while (1) {
    int speed;
    if (xSemaphoreTake(xMutex, portMAX_DELAY) == pdTRUE) {
      speed = blinkSpeed;
      xSemaphoreGive(xMutex);
    }

    counter++;
    Serial.print("Counter: ");
    Serial.print(counter);
    Serial.print(" | Blink Speed: ");
    Serial.print(speed);
    Serial.println(" ms");

    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}
```

---

## 5.6 Pertanyaan Praktikum
### 1. Apakah kedua task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!
Kedua task (`read_data` dan `display`) berjalan secara bergantian, dikendalikan oleh FreeRTOS Scheduler. Mekanisme komunikasinya menggunakan Queue (`xQueueCreate`), yaitu sebuah buffer FIFO yang menjadi perantara antar task. Task `read_data` bertugas mengisi data ke dalam queue menggunakan `xQueueSend()`, lalu memanggil `vTaskDelay()` sehingga ia masuk ke kondisi blocked. Pada saat itulah CPU beralih ke task `display` yang menunggu data dari queue menggunakan `xQueueReceive()` dengan parameter `portMAX_DELAY`, artinya ia akan terus menunggu (blocked) sampai ada data tersedia. Ketika data berhasil diterima, task `display` memproses dan mencetak data ke Serial Monitor, lalu kembali menunggu. Pola ini membuat kedua task saling bersinkronisasi melalui queue secara efisien.

---

### 2. Apakah program ini berpotensi mengalami race condition? Jelaskan!
Program ini tidak berpotensi mengalami race condition secara signifikan karena komunikasi antar task sudah menggunakan mekanisme Queue yang disediakan oleh FreeRTOS. Queue bersifat thread-safe, artinya FreeRTOS menjamin bahwa operasi `xQueueSend()` dan `xQueueReceive()` bersifat atomic dan tidak dapat diinterupsi di tengah jalan oleh task lain. Dengan demikian, tidak akan terjadi kondisi di mana kedua task mengakses data yang sama secara bersamaan tanpa sinkronisasi. Race condition biasanya terjadi ketika dua task mengakses variabel global secara langsung tanpa mekanisme proteksi seperti mutex atau semaphore. Karena program ini menggunakan queue sebagai satu-satunya media berbagi data, risiko race condition dapat dihindari.

---

### 3. Modifikasilah program dengan menggunakan sensor DHT sesungguhnya sehingga informasi yang ditampilkan dinamis. Bagaimana hasilnya? Jelaskan program pada file README.md.
<img width="1920" height="1080" alt="Screenshot 2026-05-11 205213" src="https://github.com/user-attachments/assets/0c0f4f47-af7e-4125-ae01-843303ebd8b3" />

---

```cpp
/*
 * DHT Sensor + Simulasi Task (Non-blocking dengan millis())
 *
 * Disesuaikan untuk Wokwi — menggantikan FreeRTOS Queue dengan
 * pendekatan millis()-based agar kompatibel dengan simulator.
 *
 * Perilaku identik dengan versi FreeRTOS:
 *   - "Task" read_data  : membaca sensor tiap 2000 ms
 *   - "Task" display    : menampilkan data ke Serial Monitor
 *                         hanya jika ada data baru (flag newData)
 *
 * Wiring (sesuai diagram.json):
 *   DHT SDA (DATA) -> Arduino pin 2
 *   DHT VCC        -> 5V
 *   DHT GND        -> GND
 */

#include <DHT.h>

// ── Konfigurasi sensor ──────────────────────────────────────────
#define DHTPIN   2        // Sesuai koneksi di diagram.json
#define DHTTYPE  DHT22    // Sesuai atribut "type":"DHT22" di diagram.json

DHT dht(DHTPIN, DHTTYPE);

// ── Struktur data (sama seperti versi FreeRTOS) ─────────────────
struct readings {
  float temp;   // Suhu dalam derajat Celsius
  float h;      // Kelembaban relatif dalam persen
  bool  valid;  // Flag: true jika pembacaan berhasil
};

// ── "Queue" sederhana: satu slot data + flag ketersediaan ────────
readings queueSlot;        // Slot penyimpanan data (setara queue size=1)
bool     newData = false;  // Flag pengganti xQueueOverwrite/Receive

// ── Timing untuk simulasi task ───────────────────────────────────
unsigned long lastReadTime = 0;
const unsigned long READ_INTERVAL = 2000;  // ms — sama dengan vTaskDelay(2000)

// ───────────────────────────────────────────────────────────────
void setup() {
  Serial.begin(9600);   // Baud rate sesuai serialMonitor di diagram.json
  dht.begin();

  Serial.println(F("=== DHT Monitor dimulai ==="));
  Serial.println(F("Menunggu pembacaan pertama..."));
  Serial.println(F("--------------------"));
}

// ───────────────────────────────────────────────────────────────
// Simulasi Task 1 — read_data
// Dipanggil dari loop(), aktif setiap READ_INTERVAL ms
// Setara: xQueueOverwrite(my_queue, &data) + vTaskDelay(2000)
// ───────────────────────────────────────────────────────────────
void task_read_data() {
  unsigned long now = millis();
  if (now - lastReadTime < READ_INTERVAL) return;  // Belum waktunya
  lastReadTime = now;

  readings data;
  data.h    = dht.readHumidity();
  data.temp = dht.readTemperature();  // Celsius
  data.valid = !(isnan(data.h) || isnan(data.temp));

  // Timpa slot (setara xQueueOverwrite — tidak peduli apakah sudah dibaca)
  queueSlot = data;
  newData   = true;
}

// ───────────────────────────────────────────────────────────────
// Simulasi Task 2 — display
// Dipanggil dari loop(), hanya aktif saat ada data baru
// Setara: xQueueReceive(my_queue, &data, portMAX_DELAY)
// ───────────────────────────────────────────────────────────────
void task_display() {
  if (!newData) return;   // Tidak ada data baru, lewati
  newData = false;        // Konsumsi data (setara setelah Receive berhasil)

  readings data = queueSlot;

  if (data.valid) {
    Serial.print(F("Suhu      : "));
    Serial.print(data.temp, 1);
    Serial.println(F(" °C"));

    Serial.print(F("Kelembaban: "));
    Serial.print(data.h, 1);
    Serial.println(F(" %"));

    // Hitung Heat Index (indeks panas)
    float hi = dht.computeHeatIndex(data.temp, data.h, false);
    Serial.print(F("Heat Index: "));
    Serial.print(hi, 1);
    Serial.println(F(" °C"));

    Serial.println(F("--------------------"));
  } else {
    Serial.println(F("ERROR: Gagal membaca sensor DHT!"));
    Serial.println(F("Periksa kabel dan koneksi sensor."));
    Serial.println(F("--------------------"));
  }
}

// ───────────────────────────────────────────────────────────────
void loop() {
  task_read_data();   // "Task 1" — cek & baca sensor
  task_display();     // "Task 2" — tampilkan jika ada data baru
}
```

---

## 5.7 Pertanyaan Analisis
### 1. Jelaskan perbedaan antara loop() pada Arduino biasa dengan sistem yang menggunakan RTOS! 
Pada Arduino biasa tanpa RTOS, fungsi `loop()` adalah inti dari seluruh eksekusi program. Semua task atau pekerjaan dikerjakan secara sekuensial di dalam satu loop tunggal, sehingga setiap operasi harus menunggu operasi sebelumnya selesai terlebih dahulu. Hal ini menyulitkan pengelolaan banyak pekerjaan yang membutuhkan waktu berbeda-beda. Sebaliknya, pada sistem yang menggunakan RTOS, setiap pekerjaan dipecah menjadi task-task terpisah yang masing-masing memiliki siklus eksekusinya sendiri. Fungsi `loop()` tidak lagi menjadi pusat kendali, melainkan hanya tempat inisialisasi atau bahkan dibiarkan kosong, karena scheduler RTOS yang mengambil alih pengelolaan waktu dan eksekusi seluruh task secara concurrent.

---

### 2. Mengapa fungsi loop() dibiarkan kosong?
Fungsi `loop()` dibiarkan kosong karena setelah `vTaskStartScheduler()` dipanggil di dalam `setup()`, kendali penuh atas CPU diserahkan kepada FreeRTOS Scheduler. Scheduler inilah yang bertanggung jawab menjalankan, menjeda, dan berpindah antar task sesuai prioritas dan kondisi masing-masing task. Jika ada kode di dalam `loop()`, kode tersebut sebenarnya juga akan berjalan sebagai bagian dari idle task, namun menempatkan logika program di sana bertentangan dengan prinsip arsitektur RTOS yang memisahkan setiap fungsi ke dalam task-tasknya sendiri. Mengosongkan `loop()` juga mencegah konflik antara eksekusi Arduino default dengan manajemen task oleh RTOS.

---

### 3. Apa insight utama yang Anda pelajari dari praktikum ini?
Insight utama dari praktikum ini adalah pemahaman bahwa RTOS memungkinkan sistem embedded untuk mengelola banyak pekerjaan sekaligus secara terstruktur dan terprediksi, sesuatu yang tidak dapat dilakukan dengan mudah menggunakan pendekatan bare-metal Arduino biasa. Dengan adanya scheduler, setiap task dapat dijalankan sesuai prioritas dan waktu yang telah ditentukan tanpa saling mengganggu. Selain itu, mekanisme komunikasi antar task menggunakan queue mengajarkan pentingnya sinkronisasi data agar tidak terjadi kesalahan akses memori bersama. Secara keseluruhan, RTOS memberikan fondasi yang kuat untuk membangun sistem embedded yang kompleks, modular, dan andal, terutama untuk aplikasi yang membutuhkan respons real-time terhadap berbagai kejadian secara bersamaan.
