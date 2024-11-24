# Percobaan Mengeliminasi Resource Contention dengan Menghentikan Scheduler

## Tentang Proyek
Proyek ini merupakan implementasi sistem embedded menggunakan STM32 yang mendemonstrasikan penggunaan FreeRTOS untuk mengelola multiple LED dalam lingkungan multi-threading. Program ini dirancang untuk menunjukkan konsep konkurensi dan pengelolaan sumber daya bersama (shared resources) dalam sistem embedded. Melalui implementasi multiple task yang mengakses shared resource, proyek ini mengilustrasikan bagaimana penggunaan critical section secara efektif mengeliminasi kemungkinan race condition.

## Fitur Utama
- Implementasi multi-threading dengan FreeRTOS
- Mekanisme proteksi sumber daya bersama
- Sistem indikator kesalahan menggunakan 

## Komponen Task

### 1. GreenLedTask
- Mengontrol LED Hijau
- Interval kedip: 100ms
- Mengakses shared resource dengan proteksi critical section

### 2. RedLedTask
- Mengontrol LED Merah
- Interval kedip: 500ms
- Mengakses shared resource dengan proteksi critical section

## Cara Kerja Program

### Inisialisasi Sistem
1. Melakukan konfigurasi clock sistem
2. Menginisialisasi GPIO untuk LED
3. Nonatktifkan semua LED pada awal program
4. Menginisialisasi kernel RTOS
5. Membuat dan memulai thread-thread

### Mekanisme Shared Resource
Program mengimplementasikan mekanisme proteksi sumber daya bersama dengan:
1. Menggunakan flag global (`startFlag`)
2. Implementasi critical section menggunakan `taskENTER_CRITICAL()` dan `taskEXIT_CRITICAL()`
3. LED Biru sebagai indikator akses sumber daya:
   - Menyala: Resource tersedia
   - Mati: Resource sedang digunakan

### Logika Program
1. **Pengelolaan Konkurensi**:
   - Dua task (RedLedTask dan GreenLedTask) berjalan secara concurrent
   - Masing-masing task mencoba mengakses shared resource
   - Critical section digunakan untuk menghindari race condition

2. **Mekanisme Flag**:
   - startFlag = 1: Resource tersedia
   - startFlag = 0: Resource sedang digunakan
   - Pergantian status flag menandakan akses ke resource

3. **Timing dan Delay**:
   - GreenLedTask: Delay 100ms
   - RedLedTask: Delay 500ms
   - Simulasi operasi pada shared resource: 500ms

## Konfigurasi Hardware
### Program menggunakan konfigurasi GPIO berikut:
 - GPIOE: LED Oranye dan Biru
 - GPIOB: LED Hijau dan Merah

## Catatan Pengembangan
- Program menggunakan HAL (Hardware Abstraction Layer) STM32

## Dokumentasi 
![rtos-6](https://github.com/user-attachments/assets/72ebb263-0af8-4a27-a2fa-7056fb44a0ec)

## Analisa
Pada implementasi saat ini, LED Biru tidak akan menyala untuk menandakan race condition karena penggunaan taskENTER_CRITICAL() dan taskEXIT_CRITICAL() secara efektif mencegah terjadinya akses simultan ke shared resource (startFlag). Ketika sebuah task memasuki critical section, semua interrupt di-disable dan task lain tidak dapat menginterupsi eksekusi, sehingga tidak ada kesempatan bagi dua task untuk mengakses startFlag secara bersamaan. Ini berarti kondisi dimana startFlag == 0 (yang seharusnya memicu LED Biru untuk menyala) tidak akan pernah tercapai karena setiap task dijamin menyelesaikan operasinya secara atomic sebelum task lain dapat mengakses resource tersebut.
Penggunaan critical section dalam hal ini memiliki trade-off yang signifikan. Keuntungannya adalah memberikan solusi yang sangat efektif dan straightforward untuk menghindari race condition, dengan overhead yang minimal untuk operasi-operasi singkat. Namun, menonaktifkan interrupt selama critical section berjalan bisa mempengaruhi responsivitas sistem secara keseluruhan, terutama jika ada interrupt penting yang perlu ditangani.
