# Laporan Praktikum Jaringan Komputer - Modul 3
## HTTP Protocol Analysis

### Identitas Praktikum
|------|------------|
| **Nama** | Moh Irham Maulana Al Sifri |
| **NIM** | 103072400063 |
| **Kelas** | IF-04-01 |

---

## 1. Tujuan Praktikum
Berdasarkan modul praktikum Jaringan Komputer Semester Genap 2025/2026, tujuan dari Modul 3 adalah:
- Mahasiswa dapat menginvestigasi cara kerja protokol HTTP menggunakan Wireshark.
- Mahasiswa memahami interaksi dasar HTTP GET/Response, Conditional GET, pengambilan dokumen panjang, objek tertanam (embedded objects), dan autentikasi HTTP.

---

## 2. Dasar Teori
**HTTP (Hypertext Transfer Protocol)** adalah protokol lapisan aplikasi yang digunakan untuk mendistribusikan informasi di World Wide Web. HTTP menggunakan model request-response antara klien (browser) dan server.

Beberapa aspek penting HTTP yang dianalisis dalam modul ini:
- **Basic GET/Response:** Interaksi dasar dimana klien meminta dokumen dan server merespons dengan status code (misal: 200 OK).
- **Conditional GET:** Mekanisme caching dimana klien meminta dokumen hanya jika dokumen tersebut telah dimodifikasi sejak terakhir diakses (header `If-Modified-Since`).
- **HTTP & TCP:** Dokumen besar dipecah menjadi beberapa segmen TCP ("TCP segment of a reassembled PDU").
- **Embedded Objects:** Halaman HTML yang memuat objek lain (gambar) akan memicu multiple HTTP GET requests.
- **HTTP Authentication:** Mekanisme keamanan dasar dimana kredensial dikirimkan dalam header `Authorization` (biasanya encoded Base64).

---

## 3. Langkah Kerja
Berikut adalah langkah-langkah yang dilakukan selama praktikum Modul 3:

### 3.1 Basic HTTP GET/Response Interaction
- Membersihkan cache browser.
- Menjalankan Wireshark dan mengatur filter `http`.
- Mengakses URL: `http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file1.html`.
- Menghentikan capture dan menganalisis paket HTTP GET dan OK.

### 3.2 HTTP Conditional GET/Response Interaction
- Membersihkan cache browser.
- Menjalankan Wireshark.
- Mengakses URL: `http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file2.html`.
- Melakukan refresh halaman (akses URL yang sama untuk kedua kalinya).
- Menganalisis header `If-Modified-Since` dan respons `304 Not Modified`.

### 3.3 Retrieving Long Documents
- Membersihkan cache browser.
- Mengakses URL: `http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file3.html`.
- Menganalisis respons TCP multi-paket untuk dokumen besar.

### 3.4 HTML Documents dengan Embedded Objects
- Membersihkan cache browser.
- Mengakses URL: `http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file4.html`.
- Menganalisis jumlah request HTTP GET yang terjadi (HTML + Gambar).

### 3.5 HTTP Authentication
- Membersihkan cache browser.
- Mengakses URL: `http://gaia.cs.umass.edu/wireshark-labs/protected_pages/HTTP-wireshark-file5.html`.
- Memasukkan username: `wireshark-students` dan password: `network`.
- Menganalisis header `Authorization: Basic`.

---

## 4. Hasil dan Pembahasan

### 4.1 Basic HTTP GET/Response
Pada percobaan pertama, diakses file HTML sederhana. Wireshark menangkap dua pesan utama: HTTP GET dari klien dan HTTP OK dari server.

![update](../assets/image/basic%20git.png.jpeg)
*Gambar 1: Tangkapan layar Wireshark menunjukkan paket HTTP GET dan Response 200 OK.*

**Analisis:**
- **Request:** Metode `GET`, Host `gaia.cs.umass.edu`.
- **Response:** Status Code `200 OK`, Content-Type `text/html`.
- Protokol HTTP dibawa di atas segmen TCP, datagram IP, dan frame Ethernet.

### 4.2 HTTP Conditional GET
Pada percobaan ini, halaman diakses dua kali. Akses kedua memanfaatkan cache browser.

![update](../assets/image/conditional.png.jpeg)
*Gambar 2: Tangkapan layar Wireshark menunjukkan header If-Modified-Since dan respons 304.*

**Analisis:**
- Pada request kedua, muncul header `If-Modified-Since`.
- Server merespons dengan status code `304 Not Modified`, yang berarti browser tidak perlu mengunduh ulang konten karena tidak ada perubahan. Ini menghemat bandwidth.

### 4.3 Retrieving Long Documents
Mengakses dokumen "Bill of Rights" yang cukup panjang (sekitar 4500 byte).

![Long Document](../assets/image/long%20document.png.jpeg)
*Gambar 3: Tangkapan layar Wireshark menunjukkan TCP segment of a reassembled PDU.*

**Analisis:**
- Respons HTTP tidak muat dalam satu paket TCP.
- Wireshark menampilkan keterangan `[TCP segment of a reassembled PDU]`.
- Ini menunjukkan bahwa lapisan transportasi (TCP) memecah data besar menjadi segmen-segmen kecil sebelum dikirim.

### 4.4 HTML Documents dengan Embedded Objects
Mengakses halaman HTML yang mengandung dua gambar yang disimpan di server berbeda.

![embedded object](../assets/image/embedded%20objects.png.jpeg)
*Gambar 5: Tangkapan layar Wireshark menunjukkan header Authorization Basic.*

**Analisis:**
- Request pertama ditolak server dengan status `401 Authorization Required`.
- Client mengirim ulang GET dengan header `Authorization: Basic`.
- Kredensial (username dan password) di-encode menggunakan **Base64**.
- **Keamanan:** Base64 bukan enkripsi, melainkan encoding. Siapa saja yang menangkap paket dapat mendekode kembali username dan password menggunakan decoder Base64 online. Ini menunjukkan HTTP Basic Auth tidak aman tanpa HTTPS (SSL/TLS).

## 5. Kesimpulan
Berdasarkan praktikum Modul 3 ini, dapat disimpulkan bahwa:
- **Wireshark** efektif untuk menganalisis transaksi HTTP secara detail (header, method, status code).
- **HTTP bersifat stateless**, namun mekanisme caching (Conditional GET) membantu efisiensi jaringan.
- **TCP** bertanggung jawab memecah data besar menjadi segmen-segmen (fragmentasi di lapisan transport) untuk dokumen HTML yang panjang.
- **Embedded Objects** menyebabkan terjadinya multiple HTTP request untuk satu halaman web yang kompleks.
- **HTTP Basic Authentication** tidak aman karena kredensial hanya di-encode (Base64) dan dapat dengan mudah dibaca oleh packet sniffer jika tidak menggunakan enkripsi lapisan bawah (HTTPS).

---