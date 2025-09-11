---
Title: "Menyimpan Log ke File dan Kombinasi Output"
Weight: 5
---

Sampai saat ini, log kita sudah tampil cantik di terminal. Tapi, bagaimana jika terminalnya ditutup? Semua log akan hilang! Di sesi ini, kita akan belajar bagaimana menyimpan log-log penting ini ke dalam sebuah file.

Ini seperti punya "buku catatan" yang abadi, yang bisa kita buka kapan saja untuk melihat kembali apa yang terjadi, bahkan setelah aplikasi dimatikan.

Selain itu, kita juga akan belajar bagaimana menggabungkan output: tetap menampilkan log di terminal _dan_ menyimpannya ke file secara bersamaan.

---

## Mengkonfigurasi Pino untuk Menyimpan Log ke File

Untuk menyimpan log ke file, kita akan memodifikasi konfigurasi `transport` di `utils/logger.js`. Jika sebelumnya kita hanya punya satu target ( `pino-pretty` untuk konsol), sekarang kita akan tambahkan target baru yang mengarah ke file.

**Analogi:** Bayangkan Anda memiliki pengeras suara (konsol) untuk memberitahu apa yang terjadi saat ini, dan juga seorang juru tulis (file) yang mencatat semua kejadian penting ke dalam buku.

### Modifikasi `utils/logger.js`:

Kita akan mengubah properti `transport` menjadi sebuah _array_ (daftar) target.

```js {title="JavaScript" wrap="true" lineNos="true"}
// utils/logger.js

import pino from "pino";

const logger = pino({
  enabled: process.env.LOG_ENABLE === "true",
  level: process.env.NODE_ENV === "development" ? "debug" : "info",
  timestamp: pino.stdTimeFunctions.isoTime,
  formatters: {
    level: (label) => ({ level: label.toUpperCase() }),
  },

  // *** BAGIAN BARU UNTUK KOMBINASI OUTPUT (KONSOL & FILE) ***
  transport: {
    targets: [
      // Menggunakan array untuk beberapa target output
      // Target 1: Untuk output ke konsol (dengan pino-pretty)
      {
        target: "pino-pretty",
        options: {
          colorize: true,
          translateTime: "SYS:yyyy-mm-dd HH:MM:ss",
          ignore: "pid,hostname",
        },
        level: "debug", // Di lingkungan development, semua level (debug ke atas) ke konsol
      },
      // Target 2: Untuk output ke file
      {
        target: "pino/file", // Menggunakan target 'pino/file' untuk menulis ke file
        options: {
          destination: "./logs/app.log", // Jalur ke file log utama kita
          mkdir: true, // Pino akan otomatis membuat folder 'logs' jika belum ada
          colorize: false, // Penting: Jangan beri warna di file log agar isinya bersih JSON
        },
        // Hanya log dengan level 'info' dan di atasnya yang akan disimpan ke file.
        // Ini bagus untuk production karena hanya menyimpan informasi penting dan error.
        level: "info",
      },
    ],
  },
  // ********************************************************
});

export default logger;
```

### Penjelasan Modifikasi :

- `transport: { targets: [...] }` : Ini adalah perubahan kunci. Alih-alih satu objek `transport`, kita sekarang memberikan sebuah _array_ objek. Setiap objek dalam _array_ ini adalah satu "tujuan" log.

- Target Pertama ( `pino-pretty`) : Ini adalah konfigurasi yang sama seperti di Sesi 4 untuk menampilkan log yang cantik di konsol. Kita pertahankan `level: 'debug'` agar kita bisa melihat semua detail di konsol saat pengembangan.

- Target Kedua ( `pino/file`) :

  - `target: 'pino/file'` : Ini memberi tahu Pino untuk menulis log ke sebuah file.

  - `destination: './logs/app.log'` : Kita menentukan bahwa log akan disimpan di file bernama **app.log** di dalam folder logs. Folder logs akan otomatis dibuat di _root_ proyek Anda jika belum ada (karena `mkdir: true`).

  - `mkdir: true` : Jika folder `logs` belum ada, Pino akan membuatnya secara otomatis.

  - `colorize: false` : **Ini sangat penting!** Jika Anda menyertakan warna di file log, file tersebut akan berisi karakter-karakter aneh (kode ANSI escape) yang membuat log sulit dibaca oleh program lain atau bahkan editor teks biasa. Selalu pastikan `colorize: false` saat menulis ke file.

  - `level: 'info'` : Kita mengatur bahwa ke file, hanya log dengan level `info` dan di atasnya (yaitu `info`, `warn`, `error`, `fatal`) yang akan disimpan.

    Log `debug` dan `trace` hanya akan muncul di konsol (karena `level: 'debug'` di target `pino-pretty`), ini bagus untuk memisahkan log detail pengembangan dari log penting produksi.

## Contoh Penggunaan Log untuk Menangani Error

Logging sangat, sangat penting untuk _error handling_. Ketika sebuah masalah terjadi, kita ingin tahu _apa_ yang terjadi, _di mana_, dan _mengapa_. Dengan log, kita bisa mencatat detail error tersebut.

**Analogi:** Jika ada komponen kendaraan yang rusak, kita tidak hanya ingin tahu "mesin mati", tapi juga "mesin mati karena oli bocor di silinder 3 pada RPM 5000".

Anda sudah memiliki struktur `try-catch` di rute-rute API Anda ( `/noAuth`, `/basicAuth`, dll.) dari sesi sebelumnya. Kita hanya akan memastikan penggunaan `req.log.error` di sana sudah tepat.

Pastikan Kode di **index.js Anda seperti ini (ini sudah kita lakukan di Sesi 4):**

```js {title="JavaScript" wrap="true" lineNos="true"}
// index.js (Pastikan bagian ini sudah ada)

// ... kode sebelumnya ...

app.get("/noAuth", async (req, res) => {
  try {
    const result = await axios.get(API_URL + "/random");
    req.log.info("API /random berhasil diakses.");
    res.render("index.ejs", { content: JSON.stringify(result.data) });
  } catch (error) {
    // Mencatat error dengan detailnya
    // Kita bisa menyertakan objek 'error' itu sendiri, dan properti lain
    req.log.error(
      {
        message: error.message, // Pesan error
        code: error.code, // Kode error (misal ECONNREFUSED)
        stack: error.stack, // Stack trace untuk lokasi error
        url: req.originalUrl, // URL yang diakses saat error
        method: req.method, // Metode HTTP saat error
      },
      "Gagal mengakses API /random"
    );
    res.status(404).send(error.message);
  }
});

// ... rute-rute lain dengan pola try-catch dan req.log.error serupa ...

// Middleware penanganan error global (sudah ada dari Sesi 4)
app.use((err, req, res, next) => {
  // req.log tersedia di sini jika error terjadi dalam konteks request
  // Jika tidak ada req (misal error di luar request HTTP), gunakan logger biasa
  const logFn = req && req.log ? req.log.error : logger.error;
  logFn(
    {
      error_name: err.name,
      error_message: err.message,
      error_stack: err.stack,
      request_url: req.originalUrl,
      request_method: req.method,
    },
    "Terjadi kesalahan tak terduga di server!"
  );
  res.status(500).send("Ada yang salah di server kami!");
});
```

### Penjelasan :

- Dengan menggunakan `req.log.error({ ... }, 'pesan')`, kita bisa menyertakan objek `error` itu sendiri beserta detail lain yang relevan seperti `error.message`, `error.code`, atau `error.stack` (yang menunjukkan baris kode mana yang menyebabkan error). Ini sangat membantu dalam melacak akar masalah.

## Mengkombinasikan Output Log ke Konsol dan File

Dengan konfigurasi `transport` dalam bentuk _array_ seperti yang sudah kita lakukan di `utils/logger.js`, kita secara otomatis sudah mengkombinasikan output ke konsol dan file.

**Bagaimana cara kerjanya?**

- Ketika Anda menjalankan aplikasi dalam mode `development` ( `NODE_ENV=development`) :

  - Log dengan level `trace`, `debug`, `info`, `warn`, `error`, `fatal` akan muncul di **konsol terminal** (difasilitasi oleh `pino-pretty`, berwarna dan rapi).

  - Log dengan level `info`, `warn`, `error`, `fatal` akan disimpan ke file `**logs/app.log**` (dalam format JSON mentah).

- Ketika Anda menjalankan aplikasi dalam mode `production` ( `NODE_ENV=production`) :

  - Hanya log dengan level `info`, `warn`, `error`, `fatal` yang akan muncul di **konsol terminal** (karena `level` utama logger kita sudah `info`).

  - Log dengan level `info`, `warn`, `error`, `fatal` akan disimpan ke file `**logs/app.log**`.

**Coba Jalankan Aplikasi Anda:**

1. Pastikan semua perubahan di `utils/logger.js` sudah disimpan.
2. Pastikan `NODE\_ENV=development` dan `LOG_ENABLE=true` di `.env`.
3. Buka terminal di direktori proyek Anda.
4. Jalankan aplikasi:

```bash {title="bash"}
node index.js
```

1. Akses berbagai rute di browser Anda ( `http://localhost:3000/`, `/test-log`, `/all-log-levels`, dan rute API seperti `/noAuth`).

   - Anda akan melihat log yang rapi dan berwarna di **terminal**.

   - Sekarang, buka folder **pino-auth-app/logs/** di _file explorer_ Anda. Anda akan menemukan file `app.log`.
     Buka file ini dengan editor teks (seperti VS Code, Notepad++, Sublime Text).

   Anda akan melihat baris-baris log dalam format JSON. Setiap baris adalah satu objek JSON lengkap.

Contoh isi **app.log** (JSON):

```json {title="json" wrap="true"}
{"level":30,"time":1700000000000,"pid":12345,"hostname":"YourPC","msg":"Akses ke halaman utama (\"/\")"}
{"level":30,"time":1700000000001,"pid":12345,"hostname":"YourPC","userId":123,"status":"success","msg":"Pengujian log dengan detail pengguna."}
{"level":50,"time":1700000000002,"pid":12345,"hostname":"YourPC","error":{"message":"Request failed with status code 404"},"url":"/noAuth","method":"GET","msg":"Gagal mengakses API /random"}
```

> [!note] Catatan
> `time` di JSON akan berupa angka (bukan format waktu yyyy/mm/hh : hh:mm:sss) jika Anda tidak menggunakan `translateTime` di `pino/file` option.
> Tapi karena kita hanya menggunakan `pino-pretty` untuk `translateTime`, di file akan tetap angka, yang memang format standarnya Pino.

---

Selamat! Anda telah menyelesaikan Sesi 5. Anda sekarang tidak hanya bisa mencatat log yang cantik di konsol, tetapi juga menyimpannya dengan rapi ke dalam sebuah file, memastikan tidak ada informasi penting yang terlewatkan.
