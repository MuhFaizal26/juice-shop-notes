<div align="center">

# 🚩 Challenge 2: CAPTCHA Bypass

![Risk Level](https://img.shields.io/badge/Risk-Medium-yellow?style=for-the-badge&logo=appveyor)
![Status](https://img.shields.io/badge/Status-SOLVED-brightgreen?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Broken%20Anti--Automation-blue?style=for-the-badge)

</div>

---

## 📋 Ringkasan (Summary)

| Atribut | Keterangan |
| :--- | :--- |
| **Vulnerability** | Broken Anti-Automation / Replay Attack |
| **CWE ID** | CWE-294: Authentication Bypass by Capture-replay |
| **Endpoint** | `/api/Feedbacks` |
| **Tools** | Burp Suite Community |

---

### 1. Deskripsi
Mengirimkan 10 atau lebih umpan balik pelanggan (*feedback*) dalam waktu 20 detik dengan cara melewati mekanisme anti-otomasi (CAPTCHA) aplikasi.

> **Petunjuk (Hint):**
> "Pergilah ke Balai Suara tempat para pelanggan menumpahkan keluh kesahnya. Penjaga di sana menuntut upeti berupa jawaban hitungan untuk setiap pesan yang masuk. Namun, ia tua dan lambat. Kau tak perlu memecahkan teka-tekinya sepuluh kali. Cukup temukan satu kunci jawaban yang benar, lalu gunakan kunci itu berulang-ulang untuk menyelundupkan sepuluh pesanmu sekaligus sebelum ia sempat berkedip."

### 2. Analisis
Berdasarkan teka-teki tersebut, "Balai Suara" merepresentasikan fitur *Customer Feedback*. "Upeti hitungan" adalah CAPTCHA matematika yang wajib diisi. Frasa "Penjaga tua dan lambat" mengindikasikan adanya celah keamanan pada logika validasi *backend* secara spesifik, kegagalan server untuk membatalkan token CAPTCHA setelah penggunaan pertama.

Analisis kami terhadap endpoint `/api/Feedbacks` mengungkapkan kerentanan terhadap serangan **Replay Attack**:
* **Mekanisme:** Server membuat `captchaId` dan mengharapkan jawaban matematika yang sesuai.
* **Celah:** Meskipun server memverifikasi kebenaran jawaban, server gagal menandai `captchaId` tersebut sebagai "kadaluwarsa" atau "sudah dipakai" setelah pengiriman pertama berhasil.
* **Eksploitasi:** Sebuah permintaan (*request*) yang valid (berisi `captchaId` dan jawaban yang benar) dapat ditangkap dan "dikirim ulang" (*replayed*) berkali-kali dengan cepat. Server menganggap setiap permintaan susulan tersebut sebagai pengiriman baru yang sah karena "kunci" masih dianggap benar oleh sistem yang "lambat".

---

### 3. Tahapan Eksploitasi

#### Tahap 1: Persiapan Proxy
Persiapan proxy dan target: Buka aplikasi **Burp Suite** dan pastikan fitur **Intercept** sudah aktif (*Intercept is on*). Kemudian, buka browser dan akses halaman target pada URL `http://192.168.17.133:3000/#/contact`. Pastikan browser sudah terhubung ke proxy agar lalu lintas data dapat dipantau.
<img width="1286" height="241" alt="Screenshot 2026-01-25 102344" src="https://github.com/user-attachments/assets/c667662e-bc3b-461b-8557-8ba4e5a090f4" />

#### Tahap 2: Pengambilan Sampel Valid
Kembali ke browser, isi formulir dengan komentar sembarang, rating bebas, dan selesaikan soal matematika CAPTCHA dengan jawaban yang **benar**. Klik tombol **Submit**.
<img width="955" height="915" alt="Screenshot 2026-01-25 105656" src="https://github.com/user-attachments/assets/90a34a49-8c52-4eaa-b8c5-39c76f9d6881" />

#### Tahap 3: Penyusunan Payload (Repeater)
Setelah paket POST yang valid berhasil ditangkap, jangan teruskan (*Forward*) paket tersebut. Sebaliknya, klik kanan pada area kode permintaan (*Request*) di Burp Suite untuk membuka menu konteks, lalu pilih opsi **"Send to Repeater"**.

<img width="1311" height="797" alt="Screenshot 2026-01-25 105827" src="https://github.com/user-attachments/assets/cd1f6438-4fff-4833-9d9b-ee12aa2be57e" />


#### Tahap 4: Eksekusi Serangan (Spamming)
Buka tab **Repeater** pada menu atas Burp Suite. Rincian paket sudah siap dimanipulasi. Tekan tombol **Send** di pojok kiri atas. Perhatikan panel respons di sebelah kanan (*Response*) menampilkan status `HTTP/1.1 201 Created`.

Tanpa menunggu, tekan tombol **Send** secara berulang-ulang (*spam*) sebanyak 10-20 kali dengan cepat. Konsistensi respons `201 Created` membuktikan server menerima jawaban CAPTCHA yang sama berulang kali.

<img width="1919" height="828" alt="Screenshot 2026-01-25 104923" src="https://github.com/user-attachments/assets/969e197b-d9f4-4d16-a9be-7854b325da53" />


---

### 4. 🚩 Flag
Tantangan berhasil diselesaikan. Notifikasi sukses muncul yang menandakan mekanisme CAPTCHA berhasil di-*bypass*.

<img width="876" height="205" alt="Screenshot 2026-01-24 153407" src="https://github.com/user-attachments/assets/3478259f-9cc4-46d8-9b05-99f431d540a7" />

