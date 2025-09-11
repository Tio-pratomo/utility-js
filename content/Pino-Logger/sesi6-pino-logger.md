---
Title: "Rotasi File Log (Pino Rotating File)"
Weight: 6
---

Di sesi sebelumnya, kita sudah berhasil menyimpan log ke dalam sebuah file **app.log**. Ini bagus, tapi bayangkan jika aplikasi Anda berjalan terus-menerus dan menghasilkan banyak log setiap hari.

File **app.log** akan terus membesar, dan lama-kelamaan bisa memenuhi _hard disk_ Anda! Ini seperti memiliki buku catatan yang tak ada habisnya, akhirnya bukunya terlalu tebal dan sulit dipegang.

Untuk mengatasi ini, kita butuh mekanisme **rotasi file log**. Rotasi file log adalah proses otomatis untuk:

1. **Membuat file log baru** setelah file log yang sedang aktif mencapai ukuran tertentu atau setelah jangka waktu tertentu.

2. **Memindahkan/mengganti nama** file log lama.

3. **Mengompres** (mengecilkan ukuran) file log lama untuk menghemat ruang.

4. **Menghapus file log yang sangat lama** agar tidak memenuhi penyimpanan.

Dalam sesi ini, kita akan menggunakan pustaka `pino-rotating-file` untuk melakukan tugas ini.

## Menginstal Pino Rotating File

Pertama, mari kita pasang "petugas arsip" kita, yaitu `pino-rotating-file`.

**Langkah-langkah Instalasi:**

1. Buka terminal Anda (pastikan Anda berada di direktori proyek `pino-auth-app/`).

2. Gunakan `pnpm` untuk menginstal `pino-rotating-file`:

   {{< tabs >}}
   {{% tab title="pnpm" %}}

   {{< highlight bash >}}

   pnpm add pino-rotating-file

   {{< /highlight >}}
   {{% /tab %}}

   {{% tab title="npm" %}}

   ```bash
   npm install pino-rotating-file
   ```

   {{% /tab %}}
   {{< /tabs >}}

   Ini akan menambahkan `pino-rotating-file` ke dependensi proyek Anda.

## Membuat File Konfigurasi Rotasi ( rotate-file.cjs)

**pino-rotating-file** memiliki cara konfigurasi yang sedikit berbeda. Karena pustaka ini masih menggunakan format CommonJS (bukan ES Modules seperti yang kita gunakan dengan `import/export`), kita perlu membuat file konfigurasi terpisah dengan ekstensi .cjs.

**Analogi:** Ini seperti memiliki instruksi khusus untuk petugas arsip kita, yang ditulis dalam bahasa yang berbeda dari buku catatan utama kita.

**Langkah-langkah:**

1. Di dalam folder `utils` (tempat `logger.js` berada), buat file baru dengan nama `rotate-file.cjs`.

```tree
- pino-auth-app/ | folder-open
  - ...
  - ...
  - ...
  - utils/ | folder-open
    - logger.js | fa-brands fa-js | accent
    - rotate-file.cjs   <-- File baru | fa-brands fa-js | accent
```

2. Isi File **`rotate-file.cjs`:**

```js {title="JavaScript" lineNos="true" wrap="true"}
// utils/rotate-file.cjs

// Ini adalah konfigurasi untuk pino-rotating-file
// Gunakan module.exports karena ini adalah CommonJS (.cjs)
module.exports = {
  // 'output' adalah nama file log utama yang akan dirotasi
  // Pastikan nama ini cocok dengan 'destination' di logger.js jika Anda ingin Pino mengirimkan log ke sini
  output: "app.log",

  // 'options' adalah pengaturan rotasi
  options: {
    directory: "./logs", // Folder tempat file log akan disimpan dan dirotasi
    size: "10M", // Ukuran maksimum file sebelum dirotasi (misal: '10M' = 10 Megabyte)
    // Bisa juga '10KB' (kilobyte), '1G' (gigabyte)
    interval: "1h", // Interval waktu rotasi (misal: '1h' = 1 jam)
    // Bisa juga '1d' (1 hari), '12h' (12 jam)
    count: 7, // Jumlah file log yang akan disimpan. Setelah itu, yang terlama akan dihapus.
    // Jadi akan ada app.log, app.log.1, app.log.2, ..., app.log.7
    compress: "gzip", // Kompres file log yang dirotasi menggunakan gzip (.gz)
    // Menghemat ruang penyimpanan. Bisa 'none' jika tidak ingin kompres.
  },

  // 'filter' (opsional): Anda bisa menambahkan aturan filter di sini
  // Misalnya, hanya log dengan level 'info' ke atas yang akan dirotasi
  // filter: 'info', // Uncomment ini jika ingin memfilter
};
```

Penjelasan Kode **rotate-file.cjs :**

- `module.exports = {...}` : Ini adalah sintaks CommonJS untuk mengekspor objek dari sebuah modul. Karena kita menggunakan ES Modules di bagian lain, penting untuk menggunakan ekstensi `.cjs` agar Node.js tahu cara menanganinya.

- `output: 'app.log'` : Ini menentukan nama file log utama yang akan terus ditulis oleh Pino. Ini harus sama dengan `destination` yang kita atur di `utils/logger.js`.

- `options` : Objek ini berisi semua aturan untuk rotasi:

  - `directory: './logs'` : Folder tempat semua file log (yang aktif dan yang sudah dirotasi) akan disimpan. Pastikan ini cocok dengan jalur yang Anda gunakan di `utils/logger.js`.

  - `size: '10M'` dan `interval: '1h'` : Ini adalah dua pemicu rotasi. Log akan dirotasi jika **salah satu** dari kondisi ini terpenuhi:

    - Ukuran file `app.log` mencapai 10 Megabyte.
    - Sudah 1 jam sejak rotasi terakhir atau sejak file dibuat (terlepas dari ukurannya).

  - `count: 7` : Ini berarti sistem akan menyimpan 7 file log terkompresi yang sudah dirotasi, ditambah 1 file `app.log` yang aktif. Jadi total 8 file. Ketika file ke-8 dibuat, file yang paling lama akan dihapus.

  - `compress: 'gzip'` : Setelah file dirotasi, ia akan dikompres menggunakan algoritma `gzip` dan diberi ekstensi `.gz`. Ini sangat efektif untuk menghemat ruang penyimpanan.

## Mengintegrasikan Konfigurasi Rotasi ke package.json

`pino-rotating-file` tidak langsung terintegrasi dengan logger kita di `logger.js`. Sebaliknya, ia dijalankan sebagai proses terpisah yang "mendengarkan" output log dari aplikasi kita dan melakukan rotasi.

**Analogi:** Aplikasi kita tetap menulis ke file `app.log` seperti biasa. Sementara itu, ada "petugas arsip" ( `pino-rotating-file`) yang secara berkala memeriksa file `app.log`. Jika sudah terlalu besar atau sudah waktunya, petugas arsip ini akan menyalin file `app.log` ke `app.log.1.gz`, lalu mengosongkan `app.log` agar aplikasi bisa menulis lagi dari awal.

Kita akan menambahkan _script_ baru di `package.json` untuk menjalankan proses `pino-rotating-file`.

Modifikasi **`package.json`:**

Tambahkan sebuah _script_ baru di bawah bagian `"scripts"` Anda.

```json {title="json" wrap="true"}
{
  "name": "5.4-api-authentication",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js", // Script untuk menjalankan aplikasi utama
    "log:rotate": "pino-rotate-file --config utils/rotate-file.cjs" // Script baru
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "axios": "^1.4.0",
    "ejs": "^3.1.9",
    "express": "^4.18.2",
    "pino": "^9.0.0", // Pastikan pino sudah ada
    "pino-http": "^9.0.0", // Pastikan pino-http sudah ada
    "pino-pretty": "^11.0.0", // Pastikan pino-pretty sudah ada
    "pino-rotating-file": "^2.0.0" // Pastikan pino-rotating-file sudah ada
  }
}
```

**Penjelasan Modifikasi:**

- `"log:rotate": "pino-rotate-file --config utils/rotate-file.cjs"` : Ini adalah _script_ baru. Ketika Anda menjalankan `pnpm run log:rotate`, perintah ini akan memanggil `pino-rotate-file` dan menyuruhnya menggunakan konfigurasi yang kita buat di `utils/rotate-file.cjs`.

**Bagaimana cara kerjanya sekarang?**  
Karena `pino-rotating-file` berjalan sebagai proses terpisah, Anda perlu menjalankan **dua terminal terpisah** untuk melihat efek rotasi:

1. **Terminal 1 (Aplikasi):** Jalankan aplikasi Express.js Anda seperti biasa. Ini akan menulis log ke `app.log`.

```bash {title="bash"}
pnpm start
```

(Atau `node index.js`)

2. **Terminal 2 (Rotasi Log):** Jalankan _script_ rotasi log. Ini akan "mengawasi" file `app.log` dan melakukan rotasi sesuai konfigurasi.

```bash {title="bash"}
pnpm run log:rotate
```

**Coba Jalankan dan Amati:**

1. Pastikan semua perubahan di `utils/rotate-file.cjs` dan `package.json` sudah disimpan.

2. Buka **dua terminal terpisah** di direktori proyek Anda.

3. Di **Terminal 1**, jalankan: `pnpm start`

4. Di **Terminal 2**, jalankan: `pnpm run log:rotate`

5. Akses berbagai rute di browser Anda ( `http://localhost:3000/`, `/test-log`, `/all-log-levels`, dan rute API). Buat banyak log!

6. Anda bisa sengaja memicu banyak log (misalnya, membuat loop singkat yang mencatat log terus-menerus di rute `/test-log`) atau mengatur `size` dan `interval` di `rotate-file.cjs` menjadi sangat kecil (misal: `size: '1KB'`, `interval: '5s'`) untuk melihat efek rotasi lebih cepat.

7. Perhatikan folder `logs/` Anda. Anda akan melihat:

   - `app.log`: File log aktif yang sedang ditulis.

   - app.log.1.gz, app.log.2.gz, dst.: File-file log yang sudah dirotasi dan dikompres.  
      Anda akan melihat file-file lama terkompresi dan file yang terlalu lama akan dihapus sesuai dengan count yang Anda atur.

---

Selamat! Anda telah menyelesaikan Sesi 6, sesi terakhir dari pembelajaran Pino Logger kita. Anda sekarang memiliki pemahaman yang komprehensif tentang:

- Apa itu Pino dan mengapa logging penting.
- Konsep log level dan cara mengaturnya.
- Cara menyiapkan proyek dan menginstal Pino.
- Implementasi dasar Pino, termasuk mencatat log dengan payload objek.
- Membuat output log di konsol menjadi cantik dengan `pino-pretty`.
- Otomatis mencatat permintaan HTTP dengan `pino-http`.
- Menyimpan log ke file dan menggabungkan output ke konsol dan file.
- Mengelola file log agar tidak membesar dengan rotasi menggunakan `pino-rotating-file`.

Dengan pengetahuan ini, Anda sudah siap untuk mengimplementasikan logging yang efektif dan efisien di aplikasi Node.js Anda! Jika ada pertanyaan lebih lanjut, silakan tanyakan.
