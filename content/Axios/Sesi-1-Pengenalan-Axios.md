---
Title: "Pengenalan Axios"
Weight: 1
---

## Axios – Si Kurir Super Cepat

Bayangkan Anda memiliki sebuah toko online. Ketika ada pelanggan yang ingin melihat daftar produk, atau ingin melakukan pembayaran, toko Anda perlu "mengirim" atau "menjemput" informasi dari gudang data (server).

Nah, di dunia pemrograman, proses "mengirim" dan "menjemput" informasi ini dilakukan melalui permintaan HTTP (Hypertext Transfer Protocol).

Axios adalah seperti "kurir super cepat" yang tugasnya khusus mengantarkan dan menjemput "paket" data (permintaan HTTP dan respons HTTP) antara aplikasi Anda dengan server.

## Beberapa hal penting tentang Axios:

- **Berbasis Promise:** Axios dibuat berdasarkan "Promise". Ini berarti, ketika Anda meminta Axios untuk mengirim atau menjemput data, Axios akan "berjanji" untuk memberikan hasilnya (berhasil atau gagal) nanti. Anda tidak perlu menunggu di tempat, cukup siapkan apa yang harus dilakukan setelah "paket" sampai atau jika ada masalah.

- **Isomorfik (Bisa di Mana Saja):** Axios ini sangat fleksibel karena bersifat "isomorfik". Artinya, kode Axios yang Anda tulis bisa berjalan baik di browser (misalnya, di website yang Anda buka) maupun di lingkungan Node.js (misalnya, di server atau aplikasi backend Anda) dengan basis kode yang sama5. Ini membuat pengembangan jadi lebih efisien.

- **Cara Kerja di Balik Layar:**
  - Di browser, Axios menggunakan sesuatu yang namanya XMLHttpRequests untuk berkomunikasi. Ini adalah teknologi standar browser untuk melakukan permintaan jaringan.
  - Di sisi server (Node.js), Axios menggunakan modul http asli dari Node.js.

## Fitur-fitur Axios – Kehebatan Si Kurir Ini

Kurir super cepat ini punya banyak kelebihan yang membuat pekerjaannya jadi sangat efisien:

- **Membuat Permintaan dari Browser:** Axios bisa mengirim permintaan HTTP langsung dari browser Anda.

- **Membuat Permintaan dari Node.js:** Selain browser, Axios juga jago mengirim permintaan HTTP dari lingkungan Node.js.

- **Mendukung API Promise:** Seperti yang sudah dijelaskan, Axios sepenuhnya memanfaatkan Promise API, memudahkan Anda mengelola respons asynchronous.

- **Dapat Mengintersepsi (Mencegat) Permintaan dan Respons:**
  Axios bisa "mencegat" permintaan yang akan dikirim atau respons yang baru diterima.
  Ini berguna jika Anda ingin menambahkan sesuatu ke setiap permintaan (misalnya token otentikasi) atau memproses respons sebelum aplikasi Anda menggunakannya (misalnya, menangani error secara global).

- **Mengubah Data Permintaan dan Respons:** Anda bisa mengubah format atau isi data sebelum dikirim (permintaan) atau setelah diterima (respons).

- **Dapat Membatalkan Permintaan:** Jika Anda sudah tidak butuh lagi data yang sedang diminta, Axios memungkinkan Anda untuk membatalkan permintaan yang sedang berjalan.

- **Transformasi Otomatis ke Data JSON:** Jika server mengirimkan data dalam format JSON, Axios secara otomatis akan mengubahnya menjadi objek JavaScript yang mudah Anda gunakan. Ini sangat praktis!

- **Perlindungan Terhadap XSRF:** Untuk keamanan di sisi klien (browser), Axios memiliki dukungan untuk melindungi aplikasi Anda dari serangan Cross-Site Request Forgery (XSRF).

## Cara Menginstal Axios – Menyiapkan Si Kurir

Sebelum Anda bisa menggunakan "kurir super cepat" ini, Anda perlu "memanggil" atau menginstalnya ke project Anda. Pastikan Anda sudah menginstal Node.js terlebih dahulu.
Ada beberapa cara untuk menginstal Axios:

{{< tabs >}}
{{% tab title="npm" %}}

```bash
npm install axios
```

{{% /tab %}}
{{% tab title="pnpm" %}}

```bash
pnpm add axios
```

{{% /tab %}}
{{% tab title="yarn" %}}

```bash
 yarn add axios
```

{{% /tab %}}

{{% tab title="jsDelivr" %}}

```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

{{% /tab %}}

{{% tab title="unpkg" %}}

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

{{% /tab %}}

{{< /tabs >}}

Setelah diinstal, "kurir" Axios Anda siap untuk membantu mengirim dan menjemput data!
