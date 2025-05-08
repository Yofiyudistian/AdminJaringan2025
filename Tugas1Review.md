# **WORKSHOP ADMINISTRASI JARINGAN**

**NAMA:** YOFI YUDISTIAN  
**NRP:** 3123600010  
**KELAS:** 2 D4 TEKNIK INFORMATIKA A  

#

## **Analisis File http.cap dengan Wireshark**

### **1. Analisis HTTP Traffic**

#### **a. IP Server dan Client**
![Untitled](https://github.com/user-attachments/assets/3237d54a-80a5-4e7f-a26e-d4dabe896a65)
![Untitled2](https://github.com/user-attachments/assets/46fc04f5-e460-4874-8341-be64eadc8abf)


- **IP Server:** `65.208.228.223`
- **IP Client:** `145.254.160.237`
- **IP Iklan:** `216.239.59.99` (terdeteksi sebagai iklan berdasarkan informasi di Wireshark pada kolom info yang mencantumkan "ads").

#### **b. Versi HTTP**
![Untitled](https://github.com/user-attachments/assets/3237d54a-80a5-4e7f-a26e-d4dabe896a65)
Untuk mengetahui versi HTTP yang digunakan, kita dapat memfilter dengan `http` di Wireshark. Pada informasi yang muncul, terlihat bahwa versi yang digunakan adalah **HTTP/1.1**.

#### **c. Waktu Client Mengirim Request**
![Untitled](https://github.com/user-attachments/assets/3237d54a-80a5-4e7f-a26e-d4dabe896a65)
Untuk melihat waktu saat client mengirim request, cari paket dengan metode **GET**, **POST**, atau **PUT**. Dalam contoh ini, digunakan paket nomor `4`, yang menunjukkan bahwa waktu pengiriman request adalah **0.911310 detik**.

#### **d. Waktu Server Menerima HTTP Request**
![Untitled3](https://github.com/user-attachments/assets/292d0199-9700-4285-a97d-3c729dc518b0)
![Untitled4](https://github.com/user-attachments/assets/13f3c2c9-a0b9-4c76-ac6a-9486a6f42d85)


Waktu server menerima HTTP request dapat ditemukan dengan melihat **Response in Frame**. Dalam kasus ini, request yang dikirim oleh client pada paket nomor `4` diproses dan selesai pada paket nomor `38`, dengan waktu **4.846969 detik**.

#### **e. Waktu yang Dibutuhkan untuk Transfer dan Response**
Waktu total dihitung dengan rumus:

```plaintext
Waktu transfer dan response = Waktu response - Waktu request
                           = 4.846969 - 0.911310
                           = 3.935659 detik
```
Jadi, total waktu yang dibutuhkan untuk transfer dan response adalah **3.935659 detik**.

---

## **2. Deskripsi Gambar pada Slide**
![Untitled5](https://github.com/user-attachments/assets/0ed5875f-a94a-47ac-b566-2fb077f57a4e)


Pada gambar, terlihat **node-to-node communication** yang terjadi pada lapisan **Data Link (Layer 2)** dalam **OSI Model**. Proses ini menggunakan **MAC (Media Access Control)** untuk identifikasi, dengan protokol seperti **Ethernet, WiFi, atau Frame Relay**.

- **Router ke Router:** Menggunakan protokol **Data Link** untuk komunikasi antar node. Biasanya menggunakan mac. protokol yang dipakai ethernet, wifi, atau frame relay.
- **Host to Host:** Terjadi di **Transport Layer (Layer 4)** dengan menggunakan alamat **IP**. Protokol yang digunakan antara lain **IP, ICMP, TCP, dan UDP**. Biasanya terjadi antara komputer client dan komputer server.
- **Process to Process:** Terjadi di **Transport dan Application Layer (Layer 4 & 7)** dengan menggunakan **port number** untuk komunikasi antar aplikasi, serta memanfaatkan protokol **TCP dan UDP** untuk pengiriman data dari komputer klien ke server.

---

## **3. Rangkuman Tahapan Komunikasi Menggunakan TCP**

### **a. TCP Connection Establishment (Three-Way Handshake)**
Sebelum dua perangkat bertukar data, mereka harus membangun koneksi melalui **Three-Way Handshake**:

1. **SYN (Synchronize):** Klien mengirim paket `SYN` ke server untuk memulai koneksi, berisi nomor urut awal (**Sequence Number**).
2. **SYN-ACK (Synchronize-Acknowledge):** Server menerima `SYN` dan membalas dengan `SYN-ACK`, serta mengirimkan nomor urutnya sendiri.
3. **ACK (Acknowledge):** Klien mengirim paket `ACK` untuk mengonfirmasi koneksi. Setelah ini, data dapat mulai ditransfer.

### **b. Data Transfer**
Setelah koneksi terjalin, proses pengiriman data dapat berlangsung:

- **Segmentasi dan Pengiriman:** Data dipecah menjadi **segmen TCP**, dengan setiap segmen memiliki **Sequence Number** untuk menjaga urutan pengiriman.
- **Acknowledge & Retransmission:** Penerima mengirim `ACK` untuk mengonfirmasi penerimaan data. Jika pengirim tidak menerima `ACK`, maka data akan dikirim ulang (**Retransmission**).
- **Flow Control & Congestion Control:**
  - **Flow Control** memastikan pengiriman tidak melebihi kapasitas penerima (**Window Size**).
  - **Congestion Control** mengatur kecepatan pengiriman untuk menghindari kemacetan jaringan.

### **c. TCP Connection Termination (Four-Way Handshake)**
Untuk menutup koneksi, digunakan **Four-Way Handshake**:

1. **FIN (Finish):** Host pertama mengirim `FIN` untuk meminta terminasi koneksi.
2. **ACK:** Host kedua mengakui permintaan dengan `ACK`, tetapi masih bisa mengirim data.
3. **FIN:** Setelah selesai mengirim data, host kedua mengirim `FIN` untuk menutup koneksi.
4. **ACK:** Host pertama mengirim `ACK` terakhir untuk mengonfirmasi terminasi. Pada langkah ini, koneksi benar-benar ditutup.

---

