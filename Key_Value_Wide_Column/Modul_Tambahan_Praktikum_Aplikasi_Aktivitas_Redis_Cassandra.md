# Modul Tambahan Praktikum: Aplikasi Aktivitas Pengguna dengan Redis dan Cassandra

## 1. Tujuan Praktikum

Setelah menyelesaikan praktikum, mahasiswa mampu:

1. Menghubungkan aplikasi Python dengan Redis dan Cassandra.
2. Membuat dan memeriksa session pengguna di Redis dengan masa berlaku (TTL).
3. Mencatat event pengguna ke Cassandra.
4. Menampilkan riwayat event terbaru untuk pengguna tertentu.
5. Menjelaskan alasan Redis digunakan untuk session sementara dan Cassandra untuk riwayat aktivitas.

## 2. Gambaran aplikasi

Aplikasi berbasis terminal (CLI) bernama **Activity Tracker**. Alur sederhananya:

1. Pengguna memasukkan ID pengguna untuk login.
2. Aplikasi membuat session di Redis dengan TTL.
3. Pengguna memilih aktivitas, misalnya melihat produk atau menambahkan barang ke keranjang.
4. Setiap aktivitas dicatat ke Cassandra.
5. Pengguna dapat melihat riwayat aktivitasnya.
6. Pengguna logout; session Redis dihapus, sedangkan riwayat di Cassandra tetap tersimpan.

### Pembagian tanggung jawab

| Komponen | Data | Alasan |
|---|---|---|
| Redis | Session aktif: user ID, role, waktu login | Akses cepat dan TTL untuk data sementara |
| Cassandra | Event: pengguna, waktu, jenis event, deskripsi | Menyimpan riwayat dan membaca event berdasarkan pengguna |
| Python CLI | Alur aplikasi dan antarmuka | Menghubungkan kedua database |

Session dan riwayat event adalah dua jenis data berbeda. Menghapus session saat logout **tidak** menghapus riwayat aktivitas pengguna.

## 3. Kebutuhan

- Docker Desktop/Engine dan Docker Compose.
- Python 3.10 atau 3.11 disarankan.
- Terminal (PowerShell, Windows Terminal, macOS Terminal, atau Linux shell).
- Modul utama sudah dijalankan sehingga Redis dan Cassandra tersedia.

Port host yang digunakan:
- Redis: `127.0.0.1:6379`
- Cassandra: `127.0.0.1:9042`

Jika port berbeda pada `docker-compose.yml`, sesuaikan variabel koneksi pada kode.

## 4. Siapkan layanan database

### Langkah 1 — Jalankan Redis dan Cassandra

Buka terminal pada folder yang berisi `docker-compose.yml` dari modul utama:

```bash
docker compose up -d
docker compose ps
```

Tunggu Cassandra selesai startup. Periksa log bila perlu:

```bash
docker compose logs -f cassandra
```

### Langkah 2 — Siapkan tabel Cassandra

Masuk ke CQL shell:

```bash
docker compose exec cassandra cqlsh
```

Jalankan CQL berikut. Tabel ini sama dengan tabel yang diperkenalkan di modul utama; `IF NOT EXISTS` membuat perintah aman dijalankan kembali.

```sql
CREATE KEYSPACE IF NOT EXISTS activity_lab
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};

CREATE TABLE IF NOT EXISTS activity_lab.activity_by_user (
    user_id text,
    event_time timestamp,
    event_type text,
    description text,
    PRIMARY KEY ((user_id), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);
```

Keluar:

```sql
EXIT;
```

**Mengapa model ini?** Aplikasi perlu menampilkan aktivitas terbaru berdasarkan `user_id`. Karena `user_id` adalah partition key, query riwayat menyertakan `WHERE user_id = ...`. `event_time` menjadi clustering column dan mengurutkan event terbaru lebih dahulu.

### Langkah 3 — Uji layanan

```bash
docker compose exec redis redis-cli PING
docker compose exec cassandra cqlsh -e "DESCRIBE KEYSPACES;"
```

Redis seharusnya membalas `PONG`. Cassandra harus menampilkan keyspace `activity_lab`.

## 5. Buat folder aplikasi

Buat folder `activity-tracker` di dalam folder praktikum:

```text
praktikum-nosql/
├── docker-compose.yml
└── activity-tracker/
    ├── app.py
    └── requirements.txt
```

Aplikasi Python dijalankan dari komputer host, sedangkan Redis dan Cassandra tetap berjalan di Docker. Dengan cara ini mahasiswa dapat melihat dengan jelas koneksi aplikasi ke kedua layanan.

## 6. Pasang library Python

Buat file `activity-tracker/requirements.txt`:

```text
redis>=5.0,<7.0
cassandra-driver>=3.29,<4.0
```

Buka terminal di folder `activity-tracker`.

### Windows PowerShell

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Jika PowerShell menolak aktivasi environment, jalankan perintah berikut pada terminal PowerShell yang sedang digunakan, lalu ulangi aktivasi:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

### macOS/Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## 7. Buat program aplikasi

Buat file `activity-tracker/app.py` dengan isi berikut:

```python
import json
import os
import uuid
from datetime import datetime, timezone

import redis
from cassandra.cluster import Cluster


# Konfigurasi koneksi. Nilai default cocok untuk Docker Compose modul utama.
REDIS_HOST = os.getenv("REDIS_HOST", "127.0.0.1")
REDIS_PORT = int(os.getenv("REDIS_PORT", "6379"))
CASSANDRA_HOST = os.getenv("CASSANDRA_HOST", "127.0.0.1")
CASSANDRA_PORT = int(os.getenv("CASSANDRA_PORT", "9042"))
SESSION_TTL_SECONDS = int(os.getenv("SESSION_TTL_SECONDS", "1800"))

KEYSPACE = "activity_lab"
TABLE = "activity_by_user"


def connect_redis():
    client = redis.Redis(
        host=REDIS_HOST,
        port=REDIS_PORT,
        decode_responses=True,
        socket_connect_timeout=5,
    )
    client.ping()
    return client


def connect_cassandra():
    cluster = Cluster([CASSANDRA_HOST], port=CASSANDRA_PORT)
    session = cluster.connect(KEYSPACE)
    return cluster, session


def create_session(redis_client, user_id):
    """Simpan session sementara dan kembalikan session ID."""
    session_id = str(uuid.uuid4())
    session_key = f"session:{session_id}"

    session_data = {
        "user_id": user_id,
        "role": "mahasiswa",
        "login_at": datetime.now(timezone.utc).isoformat(),
    }

    redis_client.set(
        session_key,
        json.dumps(session_data),
        ex=SESSION_TTL_SECONDS,
    )
    return session_id


def get_session(redis_client, session_id):
    """Baca session. None berarti session tidak ada atau sudah kedaluwarsa."""
    if not session_id:
        return None

    raw_data = redis_client.get(f"session:{session_id}")
    if raw_data is None:
        return None

    return json.loads(raw_data)


def delete_session(redis_client, session_id):
    if session_id:
        redis_client.delete(f"session:{session_id}")


def record_event(cassandra_session, user_id, event_type, description):
    """Catat event baru ke Cassandra."""
    event_time = datetime.now(timezone.utc)

    cassandra_session.execute(
        f"""
        INSERT INTO {TABLE}
        (user_id, event_time, event_type, description)
        VALUES (%s, %s, %s, %s)
        """,
        (user_id, event_time, event_type, description),
    )


def get_event_history(cassandra_session, user_id, limit=20):
    """Ambil event terbaru milik satu pengguna."""
    return cassandra_session.execute(
        f"""
        SELECT user_id, event_time, event_type, description
        FROM {TABLE}
        WHERE user_id = %s
        LIMIT %s
        """,
        (user_id, limit),
    )


def print_history(cassandra_session, user_id):
    rows = get_event_history(cassandra_session, user_id, limit=20)

    print(f"\\nRiwayat aktivitas untuk {user_id}:")
    print("-" * 72)
    found = False

    for row in rows:
        found = True
        timestamp = row.event_time.isoformat(sep=" ", timespec="seconds")
        print(f"{timestamp} | {row.event_type:<14} | {row.description}")

    if not found:
        print("Belum ada aktivitas.")
    print("-" * 72)


def main():
    redis_client = None
    cluster = None
    cassandra_session = None
    session_id = None

    try:
        print("Menghubungkan ke Redis dan Cassandra...")
        redis_client = connect_redis()
        cluster, cassandra_session = connect_cassandra()
        print("Koneksi berhasil. Activity Tracker siap.\\n")

        user_id = input("Masukkan ID pengguna untuk login (contoh U1001): ").strip()
        if not user_id:
            print("ID pengguna tidak boleh kosong.")
            return

        session_id = create_session(redis_client, user_id)
        print(f"Login berhasil. Session ID: {session_id}")
        print(f"Session berlaku {SESSION_TTL_SECONDS} detik.\\n")

        # Catat login sebagai event pertama.
        record_event(cassandra_session, user_id, "LOGIN", "Pengguna masuk")
        print("Event LOGIN dicatat ke Cassandra.")

        while True:
            print("\\n=== MENU ACTIVITY TRACKER ===")
            print("1. Periksa session Redis")
            print("2. Catat event: melihat produk")
            print("3. Catat event: tambah ke keranjang")
            print("4. Tampilkan riwayat aktivitas")
            print("5. Logout dan keluar")

            choice = input("Pilih menu [1-5]: ").strip()

            if choice == "1":
                current_session = get_session(redis_client, session_id)
                if current_session:
                    ttl = redis_client.ttl(f"session:{session_id}")
                    print("Session aktif:", current_session)
                    print(f"Sisa TTL: {ttl} detik")
                else:
                    print("Session tidak ditemukan atau sudah kedaluwarsa.")

            elif choice == "2":
                product_id = input("ID produk yang dilihat: ").strip()
                if product_id:
                    record_event(
                        cassandra_session,
                        user_id,
                        "VIEW_PRODUCT",
                        f"Melihat produk {product_id}",
                    )
                    print("Event VIEW_PRODUCT tersimpan.")
                else:
                    print("ID produk tidak boleh kosong.")

            elif choice == "3":
                product_id = input("ID produk yang ditambahkan: ").strip()
                if product_id:
                    record_event(
                        cassandra_session,
                        user_id,
                        "ADD_CART",
                        f"Menambahkan produk {product_id} ke keranjang",
                    )
                    print("Event ADD_CART tersimpan.")
                else:
                    print("ID produk tidak boleh kosong.")

            elif choice == "4":
                print_history(cassandra_session, user_id)

            elif choice == "5":
                record_event(
                    cassandra_session,
                    user_id,
                    "LOGOUT",
                    "Pengguna keluar",
                )
                delete_session(redis_client, session_id)
                print("Logout berhasil. Session Redis dihapus.")
                print("Riwayat event tetap tersimpan di Cassandra.")
                break

            else:
                print("Pilihan tidak dikenal. Masukkan angka 1 sampai 5.")

    except redis.RedisError as error:
        print(f"Kesalahan koneksi/operasi Redis: {error}")
    except Exception as error:
        print(f"Terjadi kesalahan aplikasi/database: {error}")
    finally:
        if cassandra_session is not None:
            cassandra_session.shutdown()
        if cluster is not None:
            cluster.shutdown()
        if redis_client is not None:
            redis_client.close()


if __name__ == "__main__":
    main()
```

### Penjelasan bagian penting

- `create_session()` menyimpan JSON sebagai string Redis dengan key `session:<session_id>` dan TTL 1.800 detik (30 menit).
- `get_session()` membaca session dari Redis. Nilai `None` menandakan key tidak ada atau sudah kedaluwarsa.
- `delete_session()` menghapus session saat logout.
- `record_event()` memasukkan event ke Cassandra.
- `get_event_history()` membaca event untuk satu `user_id`, sesuai partition key tabel.
- `print_history()` menampilkan hasil query dalam bentuk daftar aktivitas.

> **Catatan:** Session ID dibuat acak, bukan memakai `user_id` sebagai key. Dengan demikian, session pengguna tidak saling menimpa ketika pengguna login dari lebih dari satu perangkat/sesi.

## 8. Jalankan aplikasi

Pastikan terminal berada di folder `activity-tracker` dan virtual environment aktif.

```bash
python app.py
```

Contoh alur:

```text
Menghubungkan ke Redis dan Cassandra...
Koneksi berhasil. Activity Tracker siap.

Masukkan ID pengguna untuk login (contoh U1001): U1001
Login berhasil. Session ID: <session-id-acak>
Session berlaku 1800 detik.

Event LOGIN dicatat ke Cassandra.
```

Coba menu berikut:

1. Pilih `1` untuk memeriksa session.
2. Pilih `2`, masukkan `P101` untuk mencatat event melihat produk.
3. Pilih `3`, masukkan `P101` untuk mencatat event tambah keranjang.
4. Pilih `4` untuk melihat riwayat event.
5. Pilih `5` untuk logout.

Saat logout, aplikasi mencatat event `LOGOUT` ke Cassandra dan menghapus session Redis.

## 9. Verifikasi data langsung di database

### 9.1 Periksa session di Redis

Selama aplikasi masih berjalan dan belum logout, buka terminal kedua:

```bash
docker compose exec redis redis-cli --scan --pattern "session:*"
```

Salin key session yang ditemukan, lalu periksa nilainya:

```bash
docker compose exec redis redis-cli GET "session:<session-id-acak>"
docker compose exec redis redis-cli TTL "session:<session-id-acak>"
```

Ganti `<session-id-acak>` dengan ID session yang tampil saat login. TTL menunjukkan sisa waktu session. Setelah logout, key tersebut seharusnya tidak lagi ditemukan.

### 9.2 Periksa event di Cassandra

```bash
docker compose exec cassandra cqlsh
```

```sql
SELECT user_id, event_time, event_type, description
FROM activity_lab.activity_by_user
WHERE user_id = 'U1001'
LIMIT 20;
```

Hasilnya memuat event `LOGIN`, `VIEW_PRODUCT`, `ADD_CART`, dan `LOGOUT` jika seluruh langkah telah dilakukan. Event terbaru ditampilkan lebih dahulu.

Keluar dari `cqlsh`:

```sql
EXIT;
```

## 10. Percobaan TTL session

1. Jalankan aplikasi dan login.
2. Catat session ID yang ditampilkan.
3. Periksa TTL melalui Redis CLI.
4. Untuk menguji kedaluwarsa tanpa menunggu 30 menit, hentikan aplikasi dan jalankan ulang dengan TTL pendek.

### Windows PowerShell

```powershell
$env:SESSION_TTL_SECONDS="30"
python app.py
```

### macOS/Linux

```bash
SESSION_TTL_SECONDS=30 python app.py
```

Setelah login, tunggu lebih dari 30 detik. Pada menu pemeriksaan session, aplikasi akan menyatakan session tidak ditemukan atau kedaluwarsa. Perlu diperhatikan bahwa menu aplikasi masih berjalan; TTL Redis tidak otomatis menghentikan program Python. Dalam aplikasi web nyata, middleware atau pemeriksaan session harus menolak request ketika session sudah tidak valid.

Untuk kembali ke TTL default, tutup terminal tersebut atau hapus variabel environment yang telah disetel.


## 11. Tugas Mahasiswa

1. Jalankan aplikasi untuk pengguna `U1001` dan `U1002`. Catat minimal tiga event untuk masing-masing pengguna.
2. Buktikan session tersimpan di Redis dan riwayat aktivitas tersimpan di Cassandra.
3. Uji TTL 30 detik. Lampirkan hasil TTL sebelum dan setelah kedaluwarsa.
4. Logout, lalu buktikan session hilang dari Redis tetapi event tetap ada di Cassandra.
5. Modifikasi aplikasi agar pengguna dapat mencatat event `SEARCH` dengan deskripsi kata kunci pencarian.
6. Jelaskan mengapa session tidak disimpan di Cassandra dan mengapa riwayat event tidak hanya disimpan di Redis.


## 12. Membersihkan praktikum

Hentikan database dari folder yang berisi `docker-compose.yml`:

```bash
docker compose down
```

Perintah tersebut mempertahankan volume data. Untuk menghapus seluruh data lab secara permanen:

```bash
docker compose down -v
```

**Peringatan:** `docker compose down -v` menghapus volume Redis dan Cassandra. Jalankan hanya jika data praktikum tidak lagi diperlukan.
