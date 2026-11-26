# Modul Praktikum: Key-Value dan Wide-Column Database dengan Redis dan Cassandra

## 1. Tujuan Praktikum

Setelah menyelesaikan modul ini, mahasiswa mampu:
- Menyiapkan Redis dan Cassandra dengan Docker.
- Menguji koneksi kedua layanan.
- Melakukan operasi dasar Redis: `SET`, `GET`, `DEL`, Hash, dan TTL.
- Membuat keyspace dan tabel Cassandra serta melakukan `INSERT`, `SELECT`, `UPDATE`, dan `DELETE`.

## 2. Skenario Praktikum

Aplikasi membutuhkan:
1. Penyimpanan session/cache sementara dengan akses berdasarkan key.
2. Penyimpanan riwayat aktivitas pengguna, lalu menampilkan aktivitas terbaru per pengguna.

Komponen:
- Redis untuk session/cache dan TTL.
- Cassandra untuk riwayat aktivitas.
- Docker Compose untuk menjalankan kedua layanan secara lokal.

## 3. Kebutuhan Sistem

### 3.1 Perangkat lunak

- Komputer Windows 10/11, macOS, atau Linux 64-bit.
- Docker Desktop pada Windows/macOS atau Docker Engine + Docker Compose pada Linux.
- Terminal: PowerShell, Windows Terminal, Terminal macOS, atau shell Linux.
- Koneksi internet saat mengunduh image Docker.
- Ruang disk kosong beberapa GB untuk image dan volume database.

### 3.2 Port

| Layanan | Port host | Keterangan |
|---|---:|---|
| Redis | 6379 | CLI/API Redis |
| Cassandra CQL | 9042 | CQL native transport |

Jika port tersebut sudah digunakan, ubah pemetaan port pada `docker-compose.yml`. Untuk praktikum lokal, jangan membuka port database ke jaringan publik.

## 4. Menyiapkan Folder Praktikum

Buat folder bernama `praktikum-nosql` dan simpan file `docker-compose.yml` berikut di dalamnya.

```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: lab-redis
    ports:
      - "127.0.0.1:6379:6379"
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis_data:/data

  cassandra:
    image: cassandra:4.1
    container_name: lab-cassandra
    ports:
      - "127.0.0.1:9042:9042"
    environment:
      MAX_HEAP_SIZE: 1G
      HEAP_NEWSIZE: 256M
    volumes:
      - cassandra_data:/var/lib/cassandra

volumes:
  redis_data:
  cassandra_data:
```

Catatan:
- File ini adalah konfigurasi untuk lab lokal satu node, bukan arsitektur produksi.
- `127.0.0.1` membatasi port host ke komputer sendiri.
- Redis menggunakan AOF untuk memperlihatkan opsi persistensi; ini bukan pengganti backup.
- Image dan versi dapat diperbarui mengikuti kompatibilitas lingkungan kelas.

## 5. Instalasi dan Menjalankan Layanan

### Langkah 1 — Pastikan Docker tersedia

Jalankan:

```bash
docker --version
docker compose version
```

Kedua perintah harus menampilkan versi. Jika tidak, instal Docker Desktop/Engine dan jalankan servicenya terlebih dahulu.

### Langkah 2 — Jalankan container

Dari folder yang berisi `docker-compose.yml`:

```bash
docker compose up -d
```

Periksa status:

```bash
docker compose ps
```

Cassandra memerlukan waktu inisialisasi lebih lama daripada Redis. Periksa log:

```bash
docker compose logs -f cassandra
```

Tunggu sampai Cassandra selesai startup dan siap menerima koneksi, lalu tekan `Ctrl+C` untuk keluar dari tampilan log. Jika belum siap, tunggu lagi beberapa saat.

### Langkah 3 — Uji Redis

```bash
docker compose exec redis redis-cli PING
```

Hasil yang diharapkan:

```text
PONG
```

### Langkah 4 — Uji Cassandra

```bash
docker compose exec cassandra cqlsh
```

Jika berhasil, prompt `cqlsh>` muncul. Jalankan:

```sql
DESCRIBE KEYSPACES;
```

Keluar dari `cqlsh` dengan:

```sql
EXIT;
```

Jika koneksi Cassandra gagal, periksa `docker compose ps`, `docker compose logs cassandra`, penggunaan RAM, dan apakah port 9042 sudah dipakai.

## 6. Percobaan A — Redis: Operasi Key-Value

### A1. Masuk ke Redis CLI

```bash
docker compose exec redis redis-cli
```

Semua perintah di bagian A1–A5 dijalankan pada prompt Redis (`127.0.0.1:6379>`), bukan pada shell sistem operasi.

### A2. Menyimpan dan membaca string

```redis
SET session:U1001 "{\"user_id\":\"U1001\",\"role\":\"mahasiswa\"}"
GET session:U1001
```

Hasil `GET` seharusnya menampilkan string JSON yang disimpan. Redis memperlakukan value tersebut sebagai string; Redis tidak otomatis mengubahnya menjadi document yang dapat dicari berdasarkan field JSON.

**Pertanyaan:** Apa yang menjadi key dan apa yang menjadi value pada contoh ini?

### A3. Mengganti nilai dan menghapus key

```redis
SET cache:produk:101 "Laptop"
GET cache:produk:101
SET cache:produk:101 "Laptop-Updated"
GET cache:produk:101
DEL cache:produk:101
GET cache:produk:101
```

Amati bahwa `SET` dengan key yang sama mengganti nilai sebelumnya. Setelah `DEL`, `GET` menghasilkan `(nil)`.

### A4. Menyimpan struktur menggunakan Hash

```redis
HSET profile:U1001 nama "Andi" prodi "Informatika" angkatan "2023"
HGETALL profile:U1001
HGET profile:U1001 nama
HSET profile:U1001 prodi "Sistem Informasi"
HGETALL profile:U1001
```

Amati bahwa Hash menyimpan beberapa field di bawah satu key. Ini merupakan struktur data Redis, bukan berarti Redis berubah menjadi document database.

### A5. Menguji TTL

```redis
SET otp:U1001 "481205" EX 30
TTL otp:U1001
GET otp:U1001
```

Tunggu sekitar 30 detik, kemudian jalankan:

```redis
TTL otp:U1001
GET otp:U1001
```

Hasil TTL akan menurun. Setelah kedaluwarsa, key tidak lagi tersedia dan `GET` menghasilkan `(nil)`. Nilai TTL dapat berbeda tergantung waktu ketika perintah dijalankan.

Keluar dari Redis CLI:

```redis
QUIT
```

### A6. Pemeriksaan key untuk latihan

Untuk dataset kecil di lab, gunakan:

```bash
docker compose exec redis redis-cli --scan --pattern "profile:*"
```

Hindari menjadikan `KEYS *` sebagai kebiasaan pada server produksi karena operasi tersebut dapat mengganggu layanan pada dataset besar.

## 7. Percobaan B — Cassandra: Membuat Model Data

### B1. Masuk ke `cqlsh`

```bash
docker compose exec cassandra cqlsh
```

### B2. Membuat keyspace

Jalankan:

```sql
CREATE KEYSPACE IF NOT EXISTS activity_lab
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};
```

Gunakan keyspace:

```sql
USE activity_lab;
```

**Penjelasan:** `SimpleStrategy` dengan replication factor 1 digunakan untuk lab satu node. Ini bukan konfigurasi produksi multi-data-center dan tidak menyediakan salinan redundan pada node lain.

### B3. Membuat tabel aktivitas

```sql
CREATE TABLE IF NOT EXISTS activity_by_user (
    user_id text,
    event_time timestamp,
    event_type text,
    description text,
    PRIMARY KEY ((user_id), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);
```

Interpretasi primary key:
- `user_id` adalah partition key.
- `event_time` adalah clustering column.
- Aktivitas milik pengguna yang sama dikelompokkan pada partisi yang sama.
- Data dalam partisi diurutkan berdasarkan `event_time` secara menurun.

### B4. Memasukkan data contoh

Gunakan timestamp ISO-8601 dengan zona UTC (`Z`) seperti di bawah. Cassandra akan menyimpan waktu sebagai timestamp; tampilannya dapat mengikuti format klien.

```sql
INSERT INTO activity_by_user
(user_id, event_time, event_type, description)
VALUES ('U1001', '2026-10-10T08:00:00Z', 'LOGIN', 'Pengguna masuk');

INSERT INTO activity_by_user
(user_id, event_time, event_type, description)
VALUES ('U1001', '2026-10-10T08:05:00Z', 'VIEW_PRODUCT', 'Melihat produk P101');

INSERT INTO activity_by_user
(user_id, event_time, event_type, description)
VALUES ('U1001', '2026-10-10T08:10:00Z', 'ADD_CART', 'Menambahkan produk P101 ke keranjang');

INSERT INTO activity_by_user
(user_id, event_time, event_type, description)
VALUES ('U1002', '2026-10-10T09:00:00Z', 'LOGIN', 'Pengguna masuk');

INSERT INTO activity_by_user
(user_id, event_time, event_type, description)
VALUES ('U1002', '2026-10-10T09:15:00Z', 'SEARCH', 'Mencari laptop');

INSERT INTO activity_by_user
(user_id, event_time, event_type, description)
VALUES ('U1003', '2026-10-10T10:00:00Z', 'LOGIN', 'Pengguna masuk');
```

### B5. Membaca semua data untuk satu pengguna

```sql
SELECT user_id, event_time, event_type, description
FROM activity_by_user
WHERE user_id = 'U1001'
LIMIT 20;
```

Hasil yang diharapkan: tiga event milik `U1001`, dengan event terbaru lebih dahulu sesuai clustering order.

Coba pengguna lain:

```sql
SELECT user_id, event_time, event_type, description
FROM activity_by_user
WHERE user_id = 'U1002';
```


### B7. Memperbarui data

```sql
UPDATE activity_by_user
SET description = 'Melihat produk P101 (diperbarui)'
WHERE user_id = 'U1001'
  AND event_time = '2026-10-10T08:05:00Z';
```

Periksa hasil:

```sql
SELECT * FROM activity_by_user
WHERE user_id = 'U1001';
```

### B8. Menghapus satu event

```sql
DELETE FROM activity_by_user
WHERE user_id = 'U1001'
  AND event_time = '2026-10-10T08:00:00Z';
```

Periksa kembali:

```sql
SELECT * FROM activity_by_user
WHERE user_id = 'U1001';
```

### B9. Mengamati schema

```sql
DESCRIBE KEYSPACES;
DESCRIBE TABLE activity_lab.activity_by_user;
```

Keluar:

```sql
EXIT;
```

