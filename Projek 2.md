# 1. 	Arsitektur Bisnis

i)   	Tujuan aplikasi

Meningkatkan kemampuan pengguna dalam berbahasa Jawa melalui sistem pembelajaran bertingkat (level-based), mirip seperti Duolingo.

ii) 	Aktivitas bisnis utama

(1)   Manajemen konten pembelajaran (CRUD Level & Soal)

(2)   Proses pembelajaran interaktif

(3)   Monitoring progress pengguna

(4)   Penilaian otomatis hasil latihan

iii)	Aktor

(1)   Admin: mengelola level & soal

(2)   User: belajar bahasa Jawa dan menyelesaikan soal

# 2. 	Arsitektur Layanan

Modul ini merupakan implementasi layanan backend (server actions) berbasis Next.js App Router + MongoDB, yang menangani proses CRUD untuk entitas Lesson. Modul ini dirancang untuk mendukung fitur manajemen lesson pada aplikasi pembelajaran, khususnya bagian dashboard admin.

1. Dependensi
    1. (1) MongoDB sebagai database utama.
    2. Zod untuk validasi skema data (createLessonSchema, updateLessonSchema).
    3. Next.js Cache (revalidatePath) untuk menyegarkan halaman setelah mutasi data.
    4. Model: LessonModel (Mongoose) berdasarkan interface ILesson.
2. Struktur Layanan
    - getLessons(): Promise<Lesson[]>
        - Fungsi: Mengambil seluruh data Lesson dari database, diurutkan berdasarkan order.
        - Proses:
            - Melakukan koneksi ke database.
            - Mengambil seluruh lesson dan melakukan sort ascending pada order.
            - Mapping hasil query menjadi bentuk yang sesuai dengan Lesson type.

    - getLesson(lessonId: string): Promise<Lesson | null>
        - Fungsi: Mengambil seluruh data Lesson dari database, diurutkan berdasarkan order.
        - Proses:
            - Koneksi ke database.
            - Query LessonModel.findOne() berdasarkan ID.
            - Jika ditemukan, dikembalikan dalam bentuk object Lesson; jika tidak, return null.
  
3. Layanan Mutasi
    - createLesson(data: CreateLessonInput)
        - Fungsi: Menambahkan lesson baru ke database.
        - Validasi: Menggunakan createLessonSchema (Zod).
        - Proses:
            - Validasi input menggunakan Zod.
            - Generate ID baru dengan timestamp.
            - Simpan data baru ke database.
            - Revalidasi path /dashboard/lessons untuk menyegarkan tampilan.
            - Return success dan detail lesson.
    - updateLesson(lessonId: string, data: UpdateLessonInput)
        - Fungsi: Memperbarui lesson berdasarkan ID.
        - Validasi: Menggunakan updateLessonSchema.
        - Proses:
            - Validasi input.
            - Jika lesson ditemukan dan berhasil diupdate:
            - Return data lesson dan success: true
            - Revalidasi path
            - Jika gagal, return success: false.
    - deleteLesson(lessonId: string)
        - Fungsi: Menghapus lesson dari database berdasarkan ID.
        - Proses:
            - Query findOneAndDelete untuk menghapus.
            - Jika ditemukan dan dihapus, return success: true.
            - Revalidasi path.
            - Jika tidak ditemukan, return success: false.
            - 

# 3. 	Arsitektur Aplikasi

Komponen Utama :

- Frontend :
    - Teknologi: Next.js dengan React.js dan TypeScript.
    - Tanggung Jawab: Membangun UI yang interaktif untuk pembelajaran bahasa Jawa, menangani input user, dan berkomunikasi dengan server actions untuk mengambil dan mengirim data ke database.
    - Struktur:
        - components/: Folder untuk komponen UI reusable (forms, buttons, tables).
        - app/: Folder untuk routing menggunakan App Router Next.js (dashboard, lessons).
        - actions/: Folder untuk server actions yang menangani logika bisnis (lessons.ts, challenges.ts).
        - types/: Folder untuk definisi TypeScript interfaces (lesson.ts).
        - lib/: Folder untuk utilities dan konfigurasi (schemas.ts, mongodb.ts).
    - Alur Interaksi: User berinteraksi dengan form atau tabel => state pada React diperbarui => server action dipanggil => data diproses di server-side.
- Backend :
    - Teknologi: Next.js dengan Server Actions dan Mongoose ODM.
    - Tanggung Jawab: Menyediakan server actions yang dapat diakses langsung dari frontend, menangani logika bisnis untuk lessons dan challenges, melakukan validasi data menggunakan Zod schema, serta berinteraksi dengan database MongoDB.
    - Struktur:
        - actions/lessons.ts: Server actions untuk CRUD operations lessons.
        - actions/challenges.ts: Server actions untuk CRUD operations challenges.
        - lib/mongodb.ts: Konfigurasi koneksi database MongoDB dengan caching.
        - lib/schemas.ts: Validasi data menggunakan Zod untuk input validation.
        - models/Lesson.ts: Definisi model MongoDB schema menggunakan Mongoose.
    - Contoh Server Actions:
        - getLessons(): Mengambil semua lessons dengan sorting berdasarkan order.
        - getLesson(lessonId): Mengambil lesson berdasarkan ID beserta challenges-nya.
        - createLesson(data): Membuat lesson baru dengan validasi.
        - updateLesson(lessonId, data): Memperbarui lesson yang ada.
        - deleteLesson(lessonId): Menghapus lesson.
        - getChallenges(lessonId): Mengambil challenges dari lesson tertentu.
        - createChallenge(lessonId, data): Membuat challenge baru dalam lesson.
        - updateChallenge(lessonId, challengeOrder, data): Memperbarui challenge berdasarkan order.
        - deleteChallenge(lessonId, challengeOrder): Menghapus challenge dari lesson.
- Database :
    - Teknologi: MongoDB dengan Mongoose ODM.
    - Skema:
        - Lessons Collection: Menyimpan data lesson dengan embedded challenges.
        - Structure:

```basic
{
  id: "lesson_1", // Custom ID
  title: "Dasar 1",
  order: 1,
  challenges: [
    {
      question: "Makan",
      order: 1,
      instruction: "Terjemahkan kata dibawah ini:",
      options: [
        { text: "Mangan", correct: true },
        { text: "Ngombe", correct: false }
      ]
    }
  ]
}

```

- Akses: Next.js Server Actions dengan Mongoose ODM.
- Deployment: Docker container dengan seeded data dari lessons.json.
- Containerization :
    - Teknologi: Docker dan Docker Compose.
    - Struktur:
        - mongodb service: MongoDB database dengan seeding script.
        - web service: Next.js application server.
        - Network: adibasa_network untuk komunikasi antar container.
        - Volumes: mongodb_data untuk persistensi data database.
    - Data Seeding: Otomatis import data dari lessons.json saat container pertama kali dijalankan.

Alur Kerja (Contoh: Menambah Lesson Baru):

1. User membuka halaman dashboard lessons (/dashboard/lessons).
2. User mengklik tombol "Add Lesson", yang akan membuka dialog dengan komponen LessonForm.
3. User mengisi detail lesson (title, order) pada form.
4. Saat form disubmit, handleCreateLesson function dipanggil di client component.
5. Function tersebut memanggil createLesson server action dengan data lesson.
6. Server action createLesson melakukan validasi data menggunakan Zod schema (createLessonSchema).
7. Setelah validasi berhasil, server action membuat koneksi ke MongoDB menggunakan connectDB().
8. Data lesson baru disimpan ke MongoDB menggunakan Mongoose model LessonModel.
9. Server action menjalankan revalidatePath() untuk update cache Next.js.
10. Response sukses/error dikirim kembali ke client component.
11. Client component mengupdate state lokal dan menampilkan toast notification.
12. UI tabel lessons diperbarui dengan data lesson baru.

Alur Kerja (Contoh: Menambah Challenge ke Lesson):

1. User navigasi ke halaman detail lesson (/dashboard/lessons/[lessonId]).
2. User mengklik tombol "Add Challenge" untuk membuka form challenge.
3. User mengisi data challenge (question, instruction, options dengan jawaban benar/salah).
4. Form challenge disubmit dan memanggil createChallenge server action.
5. Server action melakukan validasi menggunakan createChallengeSchema.
6. Server action mencari lesson berdasarkan lessonId di MongoDB.
7. Challenge baru di-push ke array challenges dalam document lesson.
8. Document lesson disimpan dengan lesson.save().
9. Cache Next.js diperbarui dengan revalidatePath().
10. Response dikirim kembali dan UI diperbarui dengan challenge baru.

Keunggulan Arsitektur:

- Full-Stack TypeScript: Konsistensi tipe data dari frontend hingga database.
- Server Actions: Eliminasi kebutuhan API routes terpisah, lebih efisien.
- Embedded Documents: MongoDB dengan nested challenges mengurangi kompleksitas query.
- Docker Containerization: Easy deployment dan consistent environment.
- Automatic Data Seeding: Setup database otomatis dengan data pembelajaran bahasa Jawa.
- Zod Validation: Type-safe validation schema yang dapat digunakan di client dan server.
- Next.js Caching: Optimasi performa dengan built-in caching dan revalidation.

# 4. 	Arsitektur Infrastruktur

i)   	Teknologi Infrastruktur

(1)   Docker: Untuk containerisasi masing-masing layanan agar berjalan terisolasi.

(2)   Docker Compose: Untuk mengatur dan menjalankan beberapa container sekaligus secara terkoordinasi.

(3)   MongoDB: Layanan database non-relasional yang berjalan sebagai container.

(4)   Node.js App: Backend berjalan dalam container terpisah.

ii) 	Struktur Docker Compose

- Docker compose yaml
    
    ```basic
    services:
      mongodb:
        build: ./adibasa_mongodb
        container_name: adibasa_mongodb
        environment:
          - MONGO_INITDB_ROOT_USERNAME=admin
          - MONGO_INITDB_ROOT_PASSWORD=password123
          - MONGO_INITDB_DATABASE=adibasa
        ports:
          - "27017:27017"
        volumes:
          - mongodb_data:/data/db
        networks:
          - adibasa_network
    
      web:
          build: ./adibasa_web
          container_name: adibasa_web
          environment:
            - MONGODB_URI=mongodb://admin:password123@mongodb:27017/adibasa?authSource=admin
          ports:
            - "3000:3000"
          # volumes:
          #   - ./adibasa_web:/app:Z  # <--- Add :Z here
          #   - /app/node_modules
          user: "0:0"
          depends_on:
            - mongodb
          networks:
            - adibasa_network
    
    volumes:
      mongodb_data:
    
    networks:
      adibasa_network:
        driver: bridge
    ```
    

Analisa:

**I. services:** mendefinisikan kontainer, setiap layanan yang ingin dibuat akan dijalankan  di bawah service:

**Ii. mongodb:** merupakan layanan pertama yang akan didefinisikan

**Iii. build: ./adibasa_mongodb:** membangun image docker untuk layanan mongoDB. Docker akan mencari sebuah file bernama Dockerfile di dalam direktori ./adibasa_mongodb

**Iv. container_name: adibasa_mongodb:** menetapkan nama spesifik untuk kontainer yang akan dibuat dari image layanan mongodb ini saat dijalankan

**V. environment:** mendefinisikan variabel lingkungan yang akan diatur di dalam kontainer mongodb. Environment yang disertakan sebagai berikut:

1. MONGO_INITDB_ROOT_USERNAME=admin
2. MONGO_INITDB_ROOT_PASSWORD=password123
3. MONGO_INITDB_DATABASE=adibasa

**Vi. ports: "27017:27017":** Memetakan port 27017 pada mesin host ke port 27017 di dalam kontainer

**Vii. volumes: mongodb_data:/data/db:** mendefinisikan *volumes* yang akan digunakan oleh kontainer. Memetakan *named volume* bernama mongodb_data ke direktori /data/db di dalam kontainer mongodb

**Viii. networks: adibasa_network:** menentukan jaringan virtual Docker yang akan dihubungkan.  Menghubungkan layanan mongodb ke jaringan kustom yang bernama adibasa_network

**Ix. web:** untuk menjalankan layanan kedua dengan aplikasi web [next.js](http://next.js/)

**X. build: ./adibasa_web:** membangun image Docker untuk layanan web menggunakan Dockerfile yang ada di direktori ./adibasa_web

**Xi. container_name: adibasa_web:** menetapkan nama spesifik untuk kontainer yang akan dibuat dari image layanan Web ini saat dijalankan

**Xii.  environment:** mendefinisikan variabel lingkungan yang akan diatur di dalam kontainer Web. Environment yang disertakan sebagai berikut: MONGODB_URI=mongodb://admin:password123@mongodb:27017/adibasa?authSource=admin

**Xiii. ports: "3000:3000":** Memetakan port 3000 pada mesin host ke port 3000 di dalam kontainer web

**Xiv. depends_on:** mongodb:

**Xv. networks: - adibasa_network:** Menghubungkan layanan web ke jaringan kustom

Docker file mongodb

# ← use the official Mongo image as base

```basic
FROM mongo:7.0

# ← copy your JSON and the import script into the init folder
COPY lessons.json                         /docker-entrypoint-initdb.d/lessons.json
COPY init-seed.sh                         /docker-entrypoint-initdb.d/01-init-seed.sh

# ← ensure your shell-script is executable
RUN chmod +x /docker-entrypoint-initdb.d/01-init-seed.sh

```

Analisa :

- Line “**from mongo:7.0”** merupakan image resmi yang digunakan sebagai basis container. Dengan image ini sudah menyertakan tools seperti mongo, mongodb, dan mongoimport
- Folder **docker-entrypoint-initdb.d** merupakan lokasi khusus otomatis yang dijalankan oleh image mongo: saat pertamakali container dibuat khususnya ketika database belum ada
- File **01-init-seed.sh** akan di eksekusi otomatis saat namanya diawali angka 01, membuat urutannya jelas dan ekstensi **.sh** sendiri dikenali sebagai script sheel yang bisa dijalankan
- Jadi saat container mongodb di jalankan adalah :
    - Membuat database adibasa karena ada **MONGO_INITDB_DATABASE=adibasa**
    - Mengesekusi semua .js, .sh dan .josn yang ada di **/docker-entrypoint-initdb.d/** secara berurutan
    - Script **01-init-seed.sh** akan menunggu (sleep 2) dan menjalankan mongoimport untuk mengimport file **lesson.json** ke lokasi lessons.
    
- Docker file nextjs

# syntax=docker.io/docker/dockerfile:1

```basic
# syntax=docker.io/docker/dockerfile:1

FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* .npmrc* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci --verbose; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
# ENV NEXT_TELEMETRY_DISABLED=1

RUN \
  if [ -f yarn.lock ]; then yarn run build; \
  elif [ -f package-lock.json ]; then npm run build; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm run build; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
# Uncomment the following line in case you want to disable telemetry during runtime.
# ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT=3000

# server.js is created by next build from the standalone output
# https://nextjs.org/docs/pages/api-reference/config/next-config-js/output
ENV HOSTNAME="0.0.0.0"
CMD ["node", "server.js"]
```

Analisa:

- menggunakan image / template yang dibutuhkan untuk menjalankan didalam container dari node 18 alpine lalu dijadikan base
- Masuk ke tahap install depedecies dengan mengambil base yang sebelumnya sudah dibuat dan diubah jadi deps. Dengan tujuan untuk memisahkan setiap tahapan agar imagenya tidak membengkak jadi ratusan mb
- RUN apk add --no-cache libc6-compat untuk menjalankan dengan tanpa menyimpan cahce apk dan dengan paket libc6-compat untuk solusi jika build eror
- WORKDIR /app untuk mengeksekusi dari /app bukan dari root
- Lalu menyali file dari host ke docker image saat proses build dengan menyediakan 3 lock agar kompatibel dengan 3 package yarn, npm, pnpm
- Kemudian di run sesuai dengan paket manajer yang sesuai dengan lock
- Selanjutnya ke tahap build sama seperti tahap install disini memakai base yang dibuat pertama kali
- Lalu mengambil hasil install depedencies yaitu node_modules dan disalin ke tahap build agar tidak perlu install ulang
- COPY . . untuk menyalin semua file dari direktori lokal ke kontainer docker agar projeknya bisa dibuild dikontainer
- Lanjut ke tahap produksi sama seperti tahap sebelumnya
- ENV NODE_ENV=production agar dijalankan dalam production
- RUN addgroup --system --gid 1001

```basic
RUN adduser --system --uid 1001 nextjs
```

Untuk membuat grup & user khusus dalam kontainer agar tidak menjalankan diroot

- Lalu menyalin hasil dari tahap build dari builder
- Mengatur kontainer agar berjalan sebagai user dengan USER nextjs
- Lalu mengatur kontainer agar bisa menerima koneksi dari port 3000 dan mengatur variabel port env di 3000
- ENV HOSTNAME="0.0.0.0" agar environtment bisa diakses dari semua ip
- CMD ["node", "[server.js](http://server.js/)"] ketika node js dijalankan file [server.js](http://server.js/) yang dieksekusi
    
    iv)	Alur Kerja Deployment (via Docker Compose)
    
    (1)   Jalankan docker-compose up --build.
    
    (2)   Container mongo-db akan memulai terlebih dahulu dan diperiksa kesehatannya.
    
    (3)   Setelah mongo-db siap, app-backend akan berjalan dan langsung mengakses database.
    
    (4)   Semua data akan disimpan di volume mongo-data, terisolasi dan persistent.
    

# 5. 	Tampilan UI
  Tampilan websitenya: 
        ![image](https://github.com/user-attachments/assets/598345fd-fbed-40c4-b8a6-a47b6468bc45)
        ![image](https://github.com/user-attachments/assets/e4486b5e-d507-4459-84c5-308128e84166)
