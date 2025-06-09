# Menyiapkan Container MySQL di Docker

1. **Download & install Docker**  
   Link download: <https://www.docker.com/products/docker-desktop/>

2. **Jalankan container di Docker** dengan perintah berikut:

   ```bash
   docker run --name mysql-axon -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=classicmodels -p 3307:3306 -d mysql:latest
   ```
    ![image](https://github.com/user-attachments/assets/754d2970-1eb2-49c0-97d3-39153dc0cad5)

   Penjelasan parameter:
   - `--name`: nama container.
   - `-e MYSQL_ROOT_PASSWORD`: password root MySQL.
   - `-e MYSQL_DATABASE`: nama database awal yang akan dibuat.
   - `-p 3307:3306`: membuka port MySQL (localhost:3307 akan diarahkan ke container:3306).
   - `-d mysql:latest`: menjalankan container MySQL versi terbaru.

3. **Cek container** dengan perintah:

   ```bash
   docker ps -a
   ```
    ![image](https://github.com/user-attachments/assets/3e6a2fa8-56ae-411b-b89f-03a1dbfbff40)

---

# Mengimpor Data ke MySQL

1. Simpan file `Axon sales - Mysql Database.sql` dan `Axon SQL.sql` ke direktori lokal.
    ![image](https://github.com/user-attachments/assets/152b6f0e-4556-45e6-bd80-8176e7a59b04)

2. Copy 2 file SQL ke dalam container dengan perintah:

   ```bash
   docker cp "Axon sales - Mysql Database.sql" mysql-axon:/Axon.sql
   docker cp "Axon SQL.sql" mysql-axon:/Axon_script.sql
   ```
    ![image](https://github.com/user-attachments/assets/4cc88872-be95-472a-81f9-d99fc456a226)

3. Masuk ke dalam container:

   ```bash
   docker exec -it mysql-axon bash
   ```
    ![image](https://github.com/user-attachments/assets/c4913299-953e-421b-8ecd-6f04a9b8edb5)

4. Jalankan MySQL di dalam container (gunakan password root):

   ```bash
   mysql -u root -p
   ```
    ![image](https://github.com/user-attachments/assets/08a7f7e5-e334-402c-a59e-b99f878f00ca)
    ![image](https://github.com/user-attachments/assets/e73b6dbd-e876-45cd-9285-4fe782c046d2)

5. Jalankan file SQL:

   ```sql
   source /Axon.sql;
   ```
    ![image](https://github.com/user-attachments/assets/7d3dbb0f-cb81-46b3-ab01-2e991533cb4a)

---

# Menyambungkan MySQL ke Power BI Desktop

1. Download dan install Power BI melalui Microsoft Store.
    ![image](https://github.com/user-attachments/assets/d73bf64e-dd01-4b02-b49e-fb45e8c9b1fa)

2. Download dan install MySQL Connector: <https://dev.mysql.com/downloads/connector/net/>
    ![image](https://github.com/user-attachments/assets/c7ae5f2e-f330-46e4-82e7-eb8d216f1a2e)

3. Buka Power BI.
    ![image](https://github.com/user-attachments/assets/d2b81086-bbd1-433f-ab1e-f1b61aca13c8)

4. Pilih **Get Data** → **MySQL database**.
    ![image](https://github.com/user-attachments/assets/7e26584f-b41b-4a1b-ae1b-0eaaaa20ec26)

5. Masukkan parameter:
   - **Server**: `localhost:3307`
   - **Database**: `classicmodels`
    ![image](https://github.com/user-attachments/assets/f6b9ea01-8314-4635-abf7-069201b33442)

6. Masukkan username `root` dan password `root`.

7. Pilih tabel yang diinginkan (contohnya: semua tabel).
    ![image](https://github.com/user-attachments/assets/ce8895a1-e0ae-4d3b-8d4a-1de5db798f85)

8. Visualisasikan data sesuai keinginan.

   Contoh visualisasi:
   - Buat **diagram lingkaran** untuk pelanggan per negara.
   - Pilih **Pie chart** di bagian Visualizations (sebelah kanan layar).
   - Pilih tabel `classicmodels.customers`.
   - Centang dan seret kolom `country` ke bagian **Legend** atau **Axis**.
   - Seret kolom `customerNumber` ke bagian **Values**, lalu ubah ke **Count**.
    ![image](https://github.com/user-attachments/assets/2a7bb0b9-90d6-447c-895e-a5e3e882daa1)
