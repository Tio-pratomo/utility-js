---
Title: "Implementasi Dasar Pino dan Konfigurasi Log"
Weight: 3
---

Di sesi ini, kita akan mulai "menulis catatan" dengan Pino. Kita akan membuat "buku catatan" khusus untuk log kita, mengaturnya agar bisa mencatat hal-hal penting, dan melihat bagaimana hasilnya di terminal.

---

## Membuat File Konfigurasi Logger

Kita tidak ingin kode logger kita tersebar di mana-mana di aplikasi. Akan lebih rapi jika kita punya satu tempat khusus untuk mengatur bagaimana logger kita bekerja. Ini seperti membuat "ruang kontrol" untuk semua pencatatan kita.

### Langkah-langkah:

- **Buat Folder utils:**

  Di dalam folder utama proyek Anda (pino-auth-app/), buat folder baru bernama **utils**. Folder ini biasanya digunakan untuk menyimpan fungsi-fungsi pembantu atau konfigurasi yang bisa digunakan di banyak tempat.

  ```plain
  pino-auth-app/
  ├── index.js
  ├── solution.js
  ├── package.json
  ├── .env
  ├── utils/          <-- Folder baru
  └── views/
  ```

- **Buat File logger.js:**

Di dalam folder **utils** yang baru Anda buat, buat file baru dengan nama logger.js.

    ```plain
    pino-auth-app/
    ├── utils/
    │   └── logger.js   <-- File baru
    └── ...
    ```

## 2. Mengimpor Pino dan dotenv

Sekarang, di dalam file `utils/logger.js`, kita akan memanggil "alat" Pino agar bisa digunakan, dan juga memastikan kita bisa membaca variabel dari file `.env` kita.

Kode di **`utils/logger.js`:**

```js
// utils/logger.js

// Import pustaka pino
import pino from "pino";

// Membuat instance (objek) logger dari pino
// Ini adalah "buku catatan" kita
const logger = pino({
  // Konfigurasi dasar logger akan kita tambahkan di sini nanti
});

// Mengekspor logger agar bisa digunakan di bagian lain aplikasi
export default logger;
```

### Penjelasan Kode:

- `import pino from 'pino';`: Ini adalah cara modern (ES6 Modules) untuk "memanggil" pustaka Pino agar bisa kita gunakan.

- `const logger = pino({...});`: Ini adalah inti dari logger kita. Kita memanggil fungsi `pino()` untuk membuat "objek logger" baru. Objek inilah yang akan kita gunakan untuk mencatat log. Kita akan mengisi bagian `{...}` dengan pengaturan lebih lanjut.

- `export default logger;`: Kita mengekspor objek `logger` ini agar bisa "diimpor" dan digunakan di file-file JavaScript lainnya dalam proyek kita (misalnya, di `index.js`). Ini membuat kode lebih rapi dan modular.

## 3. Menggunakan Logger Dasar dan Menambahkan Objek Sebagai Payload Log

Setelah membuat objek `logger`, sekarang kita bisa mulai menggunakannya. Mari kita modifikasi file `index.js` Anda untuk menggunakan logger ini.

Modifikasi **`index.js`:**

Kita perlu melakukan beberapa perubahan di `index.js` Anda:

1. Import `dotenv` (jika belum ada) dan logger kita.
2. Gunakan logger dasar untuk mencatat saat server berjalan.
3. Tambahkan contoh log dengan objek.

Kode di **`index.js` (perubahan yang sudah ada akan kita tambahkan):**

```js
// index.js
import express from "express";
import axios from "axios";

import logger from "./utils/logger.js"; // Import logger kustom kita

const app = express();
const port = process.env.PORT || 3000; // Gunakan port dari .env

const API_URL = "https://secrets-api.appbrewery.com/";

// TODO 1: Fill in your values for the 3 types of auth.
const yourUsername = "";
const yourPassword = "";
const yourAPIKey = "";
const yourBearerToken = "";

// Mengatur view engine
app.set("view engine", "ejs"); // Pastikan ini ada jika menggunakan EJS
app.use(express.static("public")); // Jika ada file statis seperti CSS/JS

app.get("/", (req, res) => {
  // Menggunakan logger untuk mencatat akses ke halaman utama
  logger.info('Akses ke halaman utama ("/")');
  res.render("index.ejs", { content: "API Response." });
});

// ... rute-rute lain seperti /noAuth, /basicAuth, dll.
// (Tidak perlu diubah di sesi ini, akan diubah nanti di Sesi 4)

// Contoh rute lain untuk melihat logging
app.get("/test-log", (req, res) => {
  // Menggunakan logger info
  logger.info("Rute /test-log diakses.");

  // Menggunakan logger info dengan objek sebagai payload (informasi tambahan)
  // Payload ini seperti "detail kejadian" yang ingin kita catat
  logger.info(
    { userId: 123, status: "success" },
    "Pengujian log dengan detail pengguna."
  );

  res.send("Lihat konsol terminal Anda untuk melihat log!");
});

// Contoh penggunaan logger untuk server startup
// Perubahan di bagian app.listen
app.listen(port, () => {
  // Mencatat bahwa server sudah berjalan
  logger.info(`Server berhasil berjalan di port ${port}`);
  logger.info(`Aplikasi berjalan di lingkungan: ${process.env.NODE_ENV}`);
  logger.info(`Logging aktif: ${process.env.LOG_ENABLE}`);
});
```

### Penjelasan Perubahan di **`index.js`:**

- `import logger from './utils/logger.js';`: Kita "memanggil" objek logger yang sudah kita buat di `utils/logger.js`.

- `const port = process.env.PORT \|\| 3000;`: Kita mengambil nilai `PORT` dari `.env`. Jika tidak ada, defaultnya adalah `3000`.

- `logger.info('Pesan Anda di sini');`: Ini adalah cara paling dasar untuk mencatat log dengan level `info`.

- `logger.info({ userId: 123, status: 'success' }, 'Pesan dengan detail');`: Ini menunjukkan bagaimana kita bisa menyertakan objek JSON sebagai "detail tambahan" dalam log kita.

  Objek ini disebut _payload_. Ini sangat berguna karena membuat log lebih informatif dan mudah dianalisis oleh mesin. Pino akan otomatis menggabungkan objek ini ke dalam output JSON log.

**Coba Jalankan Aplikasi Anda:**

1. Pastikan Anda sudah menyimpan perubahan di `utils/logger.js` dan `index.js`.
2. Buka terminal di direktori proyek Anda.
3. Jalankan aplikasi:

```bash
node --env-file=.env index.js

```

Atau jika pakai nodemon

```bash
pnpm run dev
```

Anda akan melihat output log di terminal, tetapi mungkin masih terlihat "mentah" dalam format JSON. Jangan khawatir, kita akan membuatnya lebih cantik di sesi selanjutnya!

Akses http://localhost:3000/ dan http://localhost:3000/test-log di browser Anda untuk memicu log.

## 4. Mengatur LOG_ENABLE untuk Mengaktifkan/Menonaktifkan Log

Kita sudah membuat variabel `LOG_ENABLE` di `.env`. Sekarang mari kita gunakan ini untuk mengontrol apakah logger kita aktif atau tidak. Ini seperti sakelar utama untuk buku catatan kita.

**Modifikasi `utils/logger.js:`**  
Kita akan menambahkan properti `enabled` ke konfigurasi `pino`.

```js
// utils/logger.js

import pino from "pino";

const logger = pino({
  // Sakelar utama untuk mengaktifkan/menonaktifkan logger
  // Jika LOG_ENABLE di .env adalah 'false', logger tidak akan mencatat apapun.
  enabled: process.env.LOG_ENABLE === "true",
  // Konfigurasi lainnya akan kita tambahkan di sini
});

export default logger;
```

**Coba Ulangi:**

1. Simpan perubahan di `utils/logger.js`.
2. Jalankan ulang aplikasi ( `node index.js`). Perhatikan output log.
3. Sekarang, ubah nilai `LOG_ENABLE` di file `.env` menjadi `false`:

Cuplikan kode

```
PORT=3000
NODE_ENV=development
LOG_ENABLE=false # Ubah menjadi false

```

1. Simpan file `.env` dan jalankan ulang aplikasi. Anda seharusnya tidak melihat pesan log dari Pino di terminal. Ini membuktikan bahwa sakelar kita berfungsi!

2. Kembalikan `LOG_ENABLE` ke `true` agar kita bisa melanjutkan.

## 5. Menampilkan Semua Log Level

Secara default, Pino mungkin hanya menampilkan log dari level `info` ke atas (warning, error, fatal). Untuk tujuan pengembangan, seringkali kita ingin melihat semua detail, termasuk `debug` dan `trace`. Ini seperti menyetel kamera pengawas kita ke mode paling sensitif agar tidak ada detail yang terlewat.

### Modifikasi **`utils/logger.js`:**

Tambahkan properti `level` ke konfigurasi `pino`. Kita akan mengaturnya berdasarkan `NODE_ENV`.

```js
// utils/logger.js

import pino from "pino";

const logger = pino({
  enabled: process.env.LOG_ENABLE === "true",
  // Atur level log minimum yang akan dicatat.
  // Jika NODE_ENV='development', kita ingin melihat semua level (debug).
  // Jika NODE_ENV='production', kita hanya ingin yang penting (info ke atas).
  level: process.env.NODE_ENV === "development" ? "debug" : "info",
  // Konfigurasi lainnya akan kita tambahkan di sini
});

export default logger;
```

**Coba Ulangi di `index.js`:**  
Untuk melihat efeknya, mari tambahkan beberapa log dengan level berbeda di `index.js`.

```
// index.js (lanjutan, tambahkan ini di bagian mana saja, misal di bawah rute /test-log)

// ... kode sebelumnya ...

app.get("/all-log-levels", (req, res) => {
  logger.trace('Ini adalah pesan TRACE, sangat detail.');
  logger.debug('Ini adalah pesan DEBUG, untuk membantu pengembangan.');
  logger.info('Ini adalah pesan INFO, informasi umum.');
  logger.warn('Ini adalah pesan WARN, potensi masalah.');
  logger.error('Ini adalah pesan ERROR, ada kesalahan!');
  logger.fatal('Ini adalah pesan FATAL, aplikasi mungkin berhenti!');
  res.send('Memicu semua level log. Lihat konsol Anda.');
});

// ... app.listen dan kode lainnya ...


```

**Coba Jalankan Aplikasi:**

1. Simpan semua perubahan.
2. Pastikan `NODE_ENV=development` di `.env`.
3. Jalankan aplikasi
4. Akses `http://localhost:3000/all-log-levels` di browser Anda.

Anda akan melihat semua level log (trace, debug, info, warn, error, fatal) muncul di terminal Anda! Jika Anda mengubah `NODE_ENV=production` di `.env`, hanya log dari level `info` ke atas yang akan muncul.

## 6. Mengubah Format Output Log (Formater)

Secara default, output Pino terlihat seperti JSON mentah. Ini sangat bagus untuk mesin, tetapi kurang ramah untuk dibaca manusia di terminal. Kita bisa "merias" tampilan log agar lebih mudah dibaca. Ini seperti mengubah tulisan tangan biasa menjadi tulisan yang dicetak rapi.

Kita akan fokus pada dua hal:

- Mengubah level log dari angka menjadi label teks (misalnya, `30` menjadi `INFO`).

- Mengubah _timestamp_ dari angka epoch menjadi format tanggal-waktu yang mudah dibaca (ISO).

### Modifikasi **`utils/logger.js`:**

Tambahkan properti `timestamp` dan `formatters` ke konfigurasi `pino`.

```js
// utils/logger.js

import pino from "pino";

const logger = pino({
  enabled: process.env.LOG_ENABLE === "true",
  level: process.env.NODE_ENV === "development" ? "debug" : "info",

  // Mengatur format timestamp menjadi ISO waktu standar
  // Contoh: 2023-10-27T10:30:00.123Z
  timestamp: pino.stdTimeFunctions.isoTime,

  // Mengatur formatters untuk mengubah tampilan output
  formatters: {
    // Fungsi untuk mengubah label level log (misalnya, 30 menjadi "INFO")
    level: (label) => ({ level: label.toUpperCase() }),
    // Anda bisa menambahkan formatters lain di sini jika dibutuhkan
  },
  // Konfigurasi lainnya (transport) akan kita tambahkan di Sesi berikutnya
});

export default logger;
```

### Penjelasan Modifikasi:

- `timestamp: pino.stdTimeFunctions.isoTime`: Menginstruksikan Pino untuk menggunakan format ISO 8601 untuk stempel waktu. Ini adalah format standar yang mudah dipahami dan diurutkan.
- `formatters.level: (label) => ({ level: label.toUpperCase() })`: Ini adalah sebuah fungsi. Ketika Pino ingin menampilkan level log (misalnya, `info` yang secara internal adalah angka `30`), fungsi ini akan mengambil labelnya (misalnya, `info`) dan mengubahnya menjadi huruf kapital ( `INFO`). Hasilnya adalah objek `{ level: "INFO" }` yang kemudian akan dimasukkan ke dalam log.

**Coba Jalankan Aplikasi:**

1. Simpan perubahan di `utils/logger.js`.
2. Jalankan aplikasi: `node index.js`.
3. Akses `http://localhost:3000/` atau `http://localhost:3000/test-log` atau `http://localhost:3000/all-log-levels`.

Sekarang Anda akan melihat output log di terminal yang memiliki timestamp dalam format ISO dan level log dalam huruf kapital (misalnya, `INFO`, `DEBUG`). Ini sudah terlihat lebih rapi!

## Tambahan materi

Jika anda ingin, timestamp nya lebih mudah lagi keterbacaannya di console atau file, kita bisa gunakan internationalization seperti ini :

```js
import pino from "pino";

function getFormattedLocalTime() {
  const now = new Date();
  const options = {
    day: "2-digit",
    month: "2-digit",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    hour12: false,
  };
  // Use 'en-GB' locale which formats as DD/MM/YYYY, HH:mm, then replace separators.
  return new Intl.DateTimeFormat("en-GB", options)
    .format(now)
    .replace(",", "")
    .replace(/\//g, "-");
}

const logger = pino({
  enabled: process.env.LOG_ENABLE === "true",
  level: process.env.NODE_ENV === "development" ? "debug" : "info",
  timestamp: () => `,"time":"${getFormattedLocalTime()}"`,
  formatters: {
    level: (label) => ({ level: label.toUpperCase() }),
  },
});

export default logger;
```

### Penjelasan pada `getFormattedLocalTime() { ... }`:

- Fungsi ini bertujuan untuk menghasilkan timestamp dengan format DD-MM-YYYY HH:mm.

- `const now = new Date();`: Membuat objek Date baru yang merepresentasikan waktu saat ini.

- `const options = { … }`: Mendefinisikan opsi format untuk Intl.DateTimeFormat. Ini menentukan bahwa kita ingin hari, bulan, tahun, jam, dan menit
  ditampilkan dalam format dua digit, dan menggunakan format 24 jam (hour12: false).

- `return new Intl.DateTimeFormat('en-GB', options).format(now).replace(',', '').replace(/\//g, '-');`:

  - **new Intl.DateTimeFormat('en-GB', options):** Membuat formatter tanggal dan waktu internasional.
    Lokal **en-GB** (english-Great Britain) digunakan karena secara default menghasilkan format DD/MM/YYYY.

  - **.format(now):** Memformat objek Date saat ini sesuai dengan opsi yang diberikan. Hasilnya akan seperti 25/07/2025, 14:30.

  - **.replace(',', ''):** Menghapus koma yang memisahkan tanggal dan waktu.

  - **.replace(/\//g, '-'):** Mengganti semua garis miring (/) dengan tanda hubung (-) untuk mendapatkan format DD-MM-YYYY.

  ***

Selamat! Anda telah menyelesaikan Sesi 3. Anda sekarang sudah bisa membuat objek logger kustom, mengontrolnya dengan variabel lingkungan, melihat semua level log, dan bahkan mempercantik format output dasar di terminal.  
Di Sesi 4, kita akan membawa "kecantikan" output log ini ke tingkat selanjutnya dengan `pino-pretty` dan juga otomatis mencatat setiap permintaan HTTP yang masuk dengan `pino-http`. Tetap semangat!
