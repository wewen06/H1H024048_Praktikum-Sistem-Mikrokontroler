Nama: Wendy Virtus  
NIM: H1H024048  
Shift Awal: B  
Shift Akhir: B  

---

## 3.5 Pertanyaan Praktikum
### 1. Jelaskan proses dari input keyboard hingga LED menyala/mati! 
Pengguna mengetik karakter di Serial Monitor → komputer mengirim data melalui USB ke chip UART Arduino → data masuk ke buffer serial Arduino → Serial.available() mendeteksi ada data masuk → Serial.read() membaca satu karakter → program mengevaluasi kondisi if-else: jika '1' maka digitalWrite(PIN_LED, HIGH) → arus mengalir ke LED → LED menyala. Jika '0' maka digitalWrite(PIN_LED, LOW) → LED mati.

---

