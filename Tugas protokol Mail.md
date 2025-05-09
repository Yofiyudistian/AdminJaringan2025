1. Protokol mail (SMTP, POP3, IMAP, POP3S)

**SMTP (Simple Mail Transfer Protocol)**

- **Fungsi:** Mengirim email dari client ke mail server, atau antar mail server.
- **Port default:** 25 (umum), 587 (untuk TLS), 465 (untuk SSL)
- **Arah komunikasi:** Dari pengirim ke server, atau antar server.
- **Catatan:** SMTP tidak digunakan untuk mengambil/membaca email.

**POP3 (Post Office Protocol version 3)**

- **Fungsi:** Mengambil email dari server ke client.
- **Port default:** 110 (tanpa enkripsi), 995 (untuk SSL/TLS)
- **Catatan:** email diunduh ke perangkat pengguna sehingga dapat diakses tanpa koneksi internet setelah diunduh.
- **Ciri khas:**
  - Email diunduh ke perangkat dan biasanya dihapus dari server.
  - Cocok jika hanya menggunakan satu perangkat untuk email.

**IMAP (Internet Message Access Protocol)**

- **Fungsi:** Mengakses dan mengelola email langsung di server secara real time.
- **Port default:** 143 (tanpa enkripsi), 993 (TLS/SSL)
- **Ciri khas:**
  - Email tetap di server.
  - Sinkronisasi penuh antar banyak peran

**POP3S (POP3 Secure)**

- **Fungsi:** POP3 dengan keamanan SSL/TLS.
- **Port default:** 995
- **Bedanya dengan POP3:** Data dienkripsi untuk keamanan selama transmisi.

1. informasi mail server dalam sebuah domain

informasi mail server dalam sebuah domain disebut **MX Record** (Mail Exchange Record). Ini adalah bagian dari DNS (Domain Name System) yang menunjukkan server mana yang menangani email untuk domain tersebut.

**MX Record (Mail Exchange Record)** adalah jenis catatan dalam sistem DNS (Domain Name System) yang menunjukkan **server mana yang bertanggung jawab untuk menerima email** yang dikirim ke alamat email di domain tertentu.

## Fungsi Utama MX Record

- Mengarahkan email yang masuk ke **mail server** yang benar.
- Bisa mendukung **beberapa server email** untuk **backup/failover**.
- Memberikan **prioritas** untuk menentukan server mana yang dicoba lebih dulu.

## Catatan Penting

- MX record **harus menunjuk ke nama host (bukan langsung ke IP)**.
- Nama host tersebut **harus memiliki A record atau AAAA record** (agar bisa diubah jadi IP).
- MX record diperlukan agar email dari luar bisa dikirim ke domain kamu.

![image](https://github.com/user-attachments/assets/f27c3720-2546-49f0-845d-ba782d937d98)


menggambarkan dua pengguna email (User A dan User B) yang berkomunikasi melalui email via internet. Email dikirim dari sisi klien User A, diproses melalui berbagai komponen, lalu dikirimkan ke server User B dan diterima oleh User B. Diagram ini mencerminkan cara kerja protokol seperti SMTP, POP3, dan IMAP di balik layar.

## 💡 Komponen-Komponen Utama

### 1\. **UA (User Agent)**

- Ini adalah aplikasi yang digunakan pengguna untuk menulis, mengirim, dan membaca email. Contohnya: Outlook, Thunderbird, Gmail (web).
- Di sisi User A: UA digunakan untuk membuat dan mengirim email.
- Di sisi User B: UA digunakan untuk menerima dan membaca email.

### 2\. **Spool**

- Tempat penyimpanan sementara untuk email yang akan dikirim (outgoing queue).
- Ketika User A menulis email dan menekan “Send”, email itu akan disimpan di spool dulu sebelum dikirim oleh MTA.

### 3\. **Mailboxes**

- Tempat email yang diterima (inbox) disimpan.
- User B membaca email dari mailbox ini melalui UA.

### 4\. **Alias Expander (Alias exp.)**

- Bertugas untuk memetakan **alias email** (misalnya: <support@domain.com>) ke alamat asli atau grup penerima.
- Bisa menggunakan informasi dari **Database** untuk melakukan ekspansi (misalnya ke banyak penerima jika itu mailing list).

### 5\. **Database**

- Menyimpan informasi pengguna, konfigurasi email, alias, filter, dsb.
- Digunakan oleh alias expander untuk memutuskan ke mana email sebenarnya harus dikirim.

### 6\. **MTA (Mail Transfer Agent)**

- Komponen utama yang bertugas **mengirimkan email antar server** menggunakan protokol **SMTP**.
- Di sisi pengirim:
  - MTA pertama menangani pengambilan dari spool.
  - MTA kedua bertugas menghubungkan ke internet dan mengirim email ke MTA penerima.
- Di sisi penerima:
  - MTA menerima email dari internet dan menyerahkannya ke mailbox penerima.

### 7\. **Internet**

- Jalur komunikasi antar MTA dari domain berbeda.
- SMTP digunakan di sini untuk mentransfer email dari server User A ke server User B.

## 📬 **Alur Pengiriman Email dari User A ke User B:**

1. **User A** menulis email di UA (user agent).
2. Email dikirim ke **spool** (outbox lokal) untuk sementara.
3. **Alias expander** mengecek apakah alamat tujuan adalah alias, lalu menggunakan **Database** untuk memetakan alamat ke tujuan sebenarnya.
4. Email dikirim ke **MTA (client side)** untuk diproses.
5. MTA ini mengirim email ke MTA berikutnya (server side) yang terhubung ke internet.
6. Melalui **internet**, email dikirim ke **MTA di domain User B**.
7. MTA server User B menerima email, melewatkannya ke **alias expander** untuk mengecek alias tujuan (jika ada).
8. Alias expander menyimpan email di **mailbox** penerima.
9. **User B** menggunakan UA-nya untuk mengambil dan membaca email dari mailbox.

## 📘 Catatan Tambahan

- **SMTP** digunakan antar MTA untuk pengiriman email.
- **POP3/IMAP** akan digunakan oleh UA milik User B untuk mengambil email dari mailbox (tidak digambarkan langsung di sini, tapi UA dan Mailboxes-nya mengimplikasikan ini).
- Komponen di kiri (User A) mencerminkan **pengiriman email**, sedangkan komponen di kanan (User B) mencerminkan **penerimaan email**.
