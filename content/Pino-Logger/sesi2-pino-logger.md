---
Title: "Menyiapkan Proyek dan Instalasi Pino"
Weight: 2
---

Di sesi ini, kita akan mempersiapkan "tempat kerja" kita dan memasang "alat pencatat" Pino ke dalamnya.  
Ingat analogi kendaraan? Sekarang kita akan menyiapkan kendaraan kita dan memasang perangkat untuk mencatat apa saja yang terjadi di dalamnya.

---

## Mempersiapkan Proyek Express JS Anda

Anda sudah mengunggah file `API-Authentication.zip` yang berisi proyek Express.js. Ini adalah proyek "kendaraan" kita.

Disini, saya memakai node js versi **_v24.2.0_**

### Langkah-langkah Persiapan Proyek:

- **Ekstrak File Proyek:**

  Pertama, pastikan Anda sudah mengekstrak file API-Authentication.zip ke folder di komputer Anda. Misalnya, Anda bisa mengekstraknya ke folder bernama pino-auth-app.

  ```plain
  pino-auth-app/
  ├── index.js
  ├── solution.js
  ├── package.json
  ├── package-lock.json
  └── views/
      └── index.ejs
  ```

- **Masuk ke Direktori Proyek:**

  Buka terminal (Command Prompt/PowerShell di Windows, Terminal di macOS/Linux) dan navigasikan ke folder tempat Anda mengekstrak proyek tersebut.

  ```bash
  cd pino-auth-app
  ```

  (Ganti `pino-auth-app` dengan nama folder proyek Anda jika berbeda).

- **Instal Dependensi Proyek:**

  Proyek Express.js ini memiliki beberapa "suku cadang" yang sudah ditentukan di file package.json (seperti express, axios, ejs). Kita perlu memasang semua suku cadang ini agar proyek bisa berjalan. Karena Anda ingin menggunakan pnpm, gunakan perintah ini:

  ```bash
  pnpm install
  ```

  Perintah ini akan membaca `package.json` dan mengunduh serta memasang semua dependensi yang diperlukan ke dalam folder `node\_modules`.

## Menyiapkan File .env

File `.env` (singkatan dari _environment_) ini seperti "catatan rahasia" untuk konfigurasi aplikasi kita.
Di sini kita bisa menyimpan informasi penting yang mungkin berbeda antara satu komputer dengan komputer lain, atau antara mode pengembangan dan mode produksi, seperti nomor _port_ aplikasi atau kunci API.

Dengan begitu, kita tidak perlu mengubah kode utama saat ada perubahan konfigurasi.

- **Buat File .env:**
  Di dalam folder utama proyek Anda (pino-auth-app/), buat file baru dengan nama .env. Pastikan ada titik di depannya.

- **Isi File .env:**  
   Tambahkan baris-baris berikut ke dalam file .env yang baru Anda buat:

  ```plain
  PORT=3000
  NODE_ENV=development
  LOG_ENABLE=true
  ```

  **Penjelasan:**

  - **PORT=3000:** Ini adalah nomor _port_ di mana aplikasi web kita akan berjalan. Anda bisa mengubahnya jika `3000` sudah terpakai di komputer Anda.

  - **NODE_ENV=development:** Ini adalah variabel yang menandakan bahwa aplikasi kita berjalan dalam mode pengembangan. Nanti, ini bisa kita ubah menjadi `production` saat aplikasi siap digunakan banyak orang. Perbedaan mode ini bisa mempengaruhi bagaimana Pino mencatat log-nya.

  - **LOG_ENABLE=true:** Ini adalah sakelar khusus yang akan kita buat sendiri untuk dengan mudah mengaktifkan atau menonaktifkan logging Pino di aplikasi kita. Jika `true`, logging akan aktif. Jika `false`, logging akan dinonaktifkan.

  > [!tip] **Penting:**
  > Untuk bisa membaca nilai dari file `.env` ini di kode JavaScript kita, kita perlu pustaka `dotenv`.
  > Kabar baiknya, di proyek Anda ( `API-Authentication.zip/package.json`), sudah ada entri `type: "module"`. Kita akan pastikan nanti `dotenv` kita import di `index.js` agar variabel lingkungan bisa dibaca.

## Menjalankan Proyek (Verifikasi)

Setelah semua dependensi terinstal dan file `.env` siap, mari kita coba jalankan aplikasi untuk memastikan semuanya beres.

- **Jalankan Aplikasi:**

  Di terminal, pastikan Anda masih berada di direktori proyek (pino-auth-app/), lalu jalankan perintah ini:

  ```bash
  node --env-file=.env index.js
  ```

  Jika tidak ada error, Anda akan melihat output di terminal yang mirip dengan:

  ```bash
  Server is running on port 3000
  ```

  (Pesan ini akan kita buat lebih "cantik" dengan Pino nanti!)
  Jika ingin lebih mudah pakai nodemon, jalankan perintah :

  ```bash
  pnpm add -D nodemon
  ```

  Kemudian, tulis di bagian script :

  ```json
  "dev": "nodemon --exec \"node --env-file=.env index.js\""
  ```

- **Verifikasi di Browser:**

  Buka web browser Anda (Chrome, Firefox, dll.) dan ketik alamat: http://localhost:3000/  
   Anda seharusnya melihat tampilan halaman web dari proyek `API-Authentication` Anda. Jika ya, berarti aplikasi Anda sudah berjalan dengan baik.  
   Tekan `Ctrl + C` di terminal untuk menghentikan aplikasi.

### Mengenai perintah `node --env-file=.env index.js`

Ini adalah pendekatan yang jauh lebih sederhana dan dapat diandalkan. Ini merupakan cara alternatif dari penggunaan **_dotenv_** . Bayangkan Anda memberikan instruksi kepada Node.js, bahkan sebelum node js membuka buku resep (index.js).

1. Tahap 0: Persiapan Lingkungan

   - Saat Anda menjalankan `node --env-file=.env index.js`, hal pertama yang Node.js lakukan adalah membaca file .env.

   - Ia memuat SEMUA variabel dari file itu ke dalam `process.env`. Proses ini terjadi sebelum satu baris pun dari kode aplikasi Anda (termasuk `import`) dieksekusi.

2. Tahap 1 & 2: Mengumpulkan Bahan & Menyiapkan `logger`

   - Sekarang, Node.js mulai membaca index.js dan melihat import logger...

   - Ia melompat ke tempat anda menginisiasi logger buatan anda misalnya di **utils/logger.js**, untuk menyiapkannya.

   - Saat kode `const logger = pino(...)` pada **utils/logger.js** dijalankan, ia memeriksa process.env.LOG_ENABLE.
     Karena Tahap 0 sudah selesai, nilai process.env.LOG_ENABLE sekarang adalah string 'true' (atau 'false').

   - Logger pun dibuat dengan konfigurasi yang benar sejak awal.

**Tabel Perbandingan :**  
| Aspek | Cara Lama (`import 'dotenv'`) | Cara Baru (`--env-file`) |
|:-----------------------|:-------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------|
| **Kapan env di-load?** | Saat baris `dotenv.config()` dieksekusi di dalam kode | **Sebelum** baris kode pertama aplikasi dijalankan |
| **Ketergantungan** | Bergantung pada urutan import yang benar (sangat rapuh) | Tidak bergantung pada urutan kode sama sekali |
| **Penyebab Masalah** | Contoh: `import logger` dieksekusi sebelum `dotenv.config()`, sehingga logger tidak bisa mengakses env | Masalah tidak ada. Variabel env sudah tersedia sebelum eksekusi kode |
| **Kebersihan Kode** | Membutuhkan baris `import` dan `config()` di kode aplikasi (rentan error/cara lama) | Tidak perlu kode `dotenv` sama sekali. Aplikasi lebih bersih |
| **Rekomendasi** | Dianggap cara lama dan lebih rentan error. | Praktik terbaik modern. Lebih andal dan jelas |

## Menginstal Pino (dan Pino-HTTP)

Sekarang, saatnya memasang "alat pencatat" utamanya. Kita akan menginstal Pino dan juga `pino-http`, sebuah "tambahan" khusus Pino yang akan mempermudah kita mencatat setiap permintaan masuk ke aplikasi web kita.

- **Instal Pino dan Pino-HTTP:**

  Di terminal, pastikan Anda masih di direktori proyek, lalu jalankan perintah pnpm install untuk menambahkan Pino dan Pino-HTTP:

  ```bash
  pnpm add pino pino-http
  ```

  - **pino:** Pustaka utama Pino.
  - **pino-http:** Middleware khusus untuk Express.js yang memudahkan pencatatan permintaan HTTP.

  Anda bisa memverifikasi bahwa kedua paket ini sudah terinstal dengan melihat bagian `dependencies` di file `package.json` Anda. Akan ada entri untuk `pino` dan `pino-http`.

---

Kita sudah berhasil menyiapkan proyek, menginstal dependensi, membuat file `.env` untuk konfigurasi, dan memasang Pino serta `pino-http`.
Di sesi selanjutnya, kita akan mulai mengkonfigurasi Pino dan menggunakannya untuk mencatat log pertama kita!
