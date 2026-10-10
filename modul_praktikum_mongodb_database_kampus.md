# MODUL PRAKTIKUM
# IMPLEMENTASI DOCUMENT DATABASE MENGGUNAKAN MONGODB

**Mata Kuliah:** Database Non-Relational  
**Studi Kasus:** Sistem Informasi Akademik Kampus  
**Database:** `kampus`  
**Collection utama:** `mahasiswa`  
**Perangkat utama:** MongoDB Server, MongoDB Shell (`mongosh`), dan MongoDB Compass

---

## 1. Tujuan Praktikum

Setelah menyelesaikan praktikum ini, mahasiswa diharapkan mampu:

1. Memahami hubungan MongoDB Server, database, collection, document, dan field.
2. Menginstal atau menyiapkan lingkungan MongoDB.
3. Mengakses MongoDB melalui `mongosh` dan MongoDB Compass.
4. Membuat database `kampus` dan collection `mahasiswa`.
5. Memasukkan, membaca, memperbarui, dan menghapus document (CRUD).
6. Menggunakan query, operator perbandingan, operator logika, query array, dan projection.
7. Mengurutkan hasil, membatasi hasil, dan menerapkan pagination sederhana.
8. Membuat index dan memeriksa rencana eksekusi query.
9. Menggunakan aggregation pipeline untuk menghasilkan ringkasan akademik.
10. Mendokumentasikan hasil praktikum dan mengevaluasi rancangan document.

## 2. Gambaran Studi Kasus

Sebuah kampus ingin menyimpan data mahasiswa dalam document database. Untuk praktikum ini, data mahasiswa disimpan dalam collection `mahasiswa` pada database `kampus`.

Setiap document dapat memiliki field berikut:

| Field | Tipe data | Keterangan |
|---|---|---|
| `nim` | String | Nomor induk mahasiswa; harus unik |
| `nama` | String | Nama mahasiswa |
| `prodi` | String | Program studi |
| `angkatan` | Number | Tahun masuk |
| `ipk` | Number | Indeks prestasi kumulatif |
| `kota` | String | Kota asal/tempat tinggal |
| `skills` | Array of String | Kumpulan keterampilan |
| `alamat` | Embedded document | Informasi alamat, jika tersedia |
| `email` | String | Email mahasiswa, jika tersedia |

Contoh document:

```javascript
{
  nim: "230001",
  nama: "Andi Pratama",
  prodi: "Informatika",
  angkatan: 2023,
  ipk: 3.65,
  kota: "Surabaya",
  skills: ["Python", "MongoDB", "SQL"],
  alamat: {
    jalan: "Jl. Merdeka No. 10",
    kota: "Surabaya",
    kodePos: "60111"
  }
}
```

**Catatan desain:** MongoDB mendukung schema flexibility, sehingga tidak setiap document wajib memiliki semua field yang sama. Namun, aplikasi tetap memerlukan standar penamaan dan validasi agar data konsisten. Pada praktikum ini, `nim` disimpan sebagai string agar format identitas tetap terjaga.

## 3. Persiapan dan Pilihan Instalasi

MongoDB memiliki beberapa komponen yang perlu dibedakan:

- **MongoDB Server (`mongod`)**: proses database yang menyimpan dan memproses data.
- **MongoDB Shell (`mongosh`)**: antarmuka command line untuk menjalankan perintah.
- **MongoDB Compass**: aplikasi GUI untuk melihat database dan document.
- **MongoDB Atlas**: layanan MongoDB di cloud; dapat digunakan sebagai alternatif server lokal.

Pilih salah satu cara berikut untuk menyiapkan MongoDB. Untuk praktikum kelas, gunakan metode yang disarankan dosen agar lingkungan mahasiswa seragam.

### 3.1 Opsi A — Instalasi melalui situs resmi

1. Buka halaman unduhan MongoDB Community Server:  
   <https://www.mongodb.com/try/download/community>
2. Pilih sistem operasi dan paket yang sesuai.
3. Unduh installer, lalu ikuti petunjuk instalasi resmi untuk sistem operasi tersebut.
4. Pastikan MongoDB Server terpasang dan servicenya berjalan.
5. Pasang MongoDB Shell (`mongosh`) jika belum tersedia:  
   <https://www.mongodb.com/try/download/shell>
6. (Opsional tetapi disarankan) Pasang MongoDB Compass:  
   <https://www.mongodb.com/try/download/compass>

Nama menu dan langkah detail dapat berbeda sesuai versi sistem operasi serta paket MongoDB yang dipilih. Ikuti dokumentasi resmi untuk versi yang digunakan.

### 3.2 Opsi B — macOS menggunakan Homebrew

Jika Homebrew sudah tersedia, terminal dapat digunakan untuk memasang MongoDB Community Edition. Perintah berikut merupakan pola instalasi yang umum; periksa dokumentasi resmi jika nama formula atau prosedurnya berubah.

```bash
brew tap mongodb/brew
brew install mongodb-community
```

Jalankan service MongoDB dengan perintah yang sesuai dengan formula yang terpasang, misalnya:

```bash
brew services start mongodb-community
```

Jika perintah tersebut tidak cocok dengan versi formula, periksa instruksi instalasi MongoDB untuk macOS di dokumentasi resmi:
<https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-os-x/>

### 3.3 Opsi C — Windows

1. Unduh MongoDB Community Server untuk Windows dari situs resmi.
2. Jalankan file installer `.msi`.
3. Ikuti wizard instalasi. Jika tersedia, pilih opsi pemasangan sebagai Windows Service agar server dapat berjalan sebagai layanan.
4. Pasang `mongosh` jika belum termasuk dalam instalasi.
5. Buka Command Prompt atau PowerShell baru.
6. Uji apakah shell tersedia:

```powershell
mongosh --version
```

Jika muncul pesan bahwa `mongosh` tidak dikenali, periksa apakah MongoDB Shell telah dipasang dan direktori executable sudah masuk ke `PATH`. Lihat panduan resmi:
<https://www.mongodb.com/docs/mongodb-shell/install/>

### 3.4 Opsi D — Ubuntu/Linux

Langkah instalasi MongoDB di Linux bergantung pada distribusi dan versinya. Gunakan instruksi resmi yang sesuai, bukan mencampur repository dari distribusi atau versi yang berbeda:

<https://www.mongodb.com/docs/manual/administration/install-on-linux/>

Setelah instalasi, periksa status service sesuai sistem operasi. Pada sistem berbasis systemd, contoh perintah yang umum adalah:

```bash
sudo systemctl status mongod
```

Jika service belum berjalan, perintah untuk memulainya pada sistem berbasis systemd biasanya:

```bash
sudo systemctl start mongod
```

Nama service dan mekanisme service dapat berbeda. Ikuti dokumentasi untuk distribusi yang digunakan.

### 3.5 Opsi E — Docker (alternatif untuk praktikum)

Gunakan Docker hanya jika Docker Engine/Desktop telah terpasang. Perintah berikut membuat container MongoDB bernama `mongodb-praktikum` dan memetakan port lokal `27017`.

```bash
docker run -d \
  --name mongodb-praktikum \
  -p 27017:27017 \
  mongo:latest
```

Periksa container:

```bash
docker ps
docker logs mongodb-praktikum
```

Hubungkan shell jika image yang digunakan menyediakan `mongosh`:

```bash
docker exec -it mongodb-praktikum mongosh
```

**Penting:** contoh Docker ini ditujukan untuk latihan lokal, bukan deployment produksi. Container di atas tidak mengatur autentikasi maupun penyimpanan persisten khusus. Untuk data yang perlu dipertahankan setelah container dihapus, konfigurasi volume dan keamanan dengan benar. Jika nama container sudah dipakai, pilih nama lain atau periksa container yang ada sebelum menghapus apa pun.

## 4. Verifikasi Instalasi dan Koneksi

### 4.1 Periksa versi `mongosh`

Buka Terminal, Command Prompt, atau PowerShell:

```bash
mongosh --version
```

Jika versi tampil, shell tersedia. Jika tidak, pastikan `mongosh` sudah terpasang dan dapat ditemukan melalui `PATH`.

### 4.2 Hubungkan ke server lokal

Untuk instalasi lokal dengan konfigurasi standar:

```bash
mongosh "mongodb://127.0.0.1:27017"
```

Atau, jika koneksi default sudah dikonfigurasi:

```bash
mongosh
```

Jika berhasil, prompt shell akan muncul. Jika gagal:

1. Pastikan MongoDB Server sudah berjalan.
2. Periksa host dan port koneksi.
3. Pastikan tidak ada konfigurasi jaringan atau firewall yang menghalangi koneksi.
4. Jika memakai Docker, pastikan container aktif dan port telah dipetakan.
5. Jika memakai Atlas, gunakan connection string Atlas, bukan URI lokal.

### 4.3 Perintah awal di shell

Jalankan satu per satu:

```javascript
db
show dbs
show collections
```

Perintah `db` menampilkan database yang sedang dipilih. `show dbs` menampilkan database yang telah tersimpan dan dapat diakses. Database yang baru dipilih tetapi belum berisi data belum tentu tampil pada `show dbs`.

## 5. Membuat Database dan Collection

### 5.1 Memilih database `kampus`

Di `mongosh`, jalankan:

```javascript
use kampus
```

Perintah ini memilih database bernama `kampus`. Database biasanya benar-benar muncul dalam daftar setelah ada data yang disimpan atau collection dibuat dengan data.

Periksa database aktif:

```javascript
db
```

Output yang diharapkan:

```text
kampus
```

### 5.2 Membuat collection `mahasiswa`

```javascript
db.createCollection("mahasiswa")
```

Jika collection sudah ada, MongoDB dapat menampilkan pesan bahwa namespace sudah ada. Periksa collection:

```javascript
show collections
```

Output yang diharapkan mencakup:

```text
mahasiswa
```

MongoDB juga dapat membuat collection secara otomatis ketika document pertama dimasukkan. Dalam praktikum ini collection dibuat secara eksplisit agar mahasiswa memahami langkahnya.

## 6. Memasukkan Data Mahasiswa (Create)

### 6.1 Memasukkan satu document dengan `insertOne()`

```javascript
db.mahasiswa.insertOne({
  nim: "230001",
  nama: "Andi Pratama",
  prodi: "Informatika",
  angkatan: 2023,
  ipk: 3.65,
  kota: "Surabaya",
  skills: ["Python", "MongoDB", "SQL"],
  alamat: {
    jalan: "Jl. Merdeka No. 10",
    kota: "Surabaya",
    kodePos: "60111"
  }
})
```

Periksa hasilnya:

```javascript
db.mahasiswa.find({ nim: "230001" })
```

MongoDB biasanya membuat field `_id` secara otomatis jika tidak diberikan. Field `_id` merupakan identifier unik document.

### 6.2 Memasukkan banyak document dengan `insertMany()`

Masukkan data berikut. Jalankan blok ini satu kali untuk menghindari duplikasi data saat mengulang praktikum.

```javascript
db.mahasiswa.insertMany([
  {
    nim: "230002",
    nama: "Budi Santoso",
    prodi: "Sistem Informasi",
    angkatan: 2023,
    ipk: 3.45,
    kota: "Bandung",
    skills: ["Java", "SQL"]
  },
  {
    nim: "230003",
    nama: "Citra Lestari",
    prodi: "Informatika",
    angkatan: 2022,
    ipk: 3.80,
    kota: "Surabaya",
    skills: ["Python", "MongoDB", "Data Science"]
  },
  {
    nim: "230004",
    nama: "Dewi Anggraini",
    prodi: "Teknik Komputer",
    angkatan: 2023,
    ipk: 3.25,
    kota: "Semarang",
    skills: ["C", "IoT"]
  },
  {
    nim: "230005",
    nama: "Eko Saputra",
    prodi: "Informatika",
    angkatan: 2021,
    ipk: 3.55,
    kota: "Malang",
    skills: ["JavaScript", "Python"]
  },
  {
    nim: "230006",
    nama: "Fitri Handayani",
    prodi: "Sistem Informasi",
    angkatan: 2022,
    ipk: 3.90,
    kota: "Jakarta",
    skills: ["SQL", "Data Analysis"]
  },
  {
    nim: "230007",
    nama: "Galih Prakoso",
    prodi: "Teknik Komputer",
    angkatan: 2021,
    ipk: 3.10,
    kota: "Surabaya",
    skills: ["C++", "Linux"]
  },
  {
    nim: "230008",
    nama: "Hana Putri",
    prodi: "Informatika",
    angkatan: 2024,
    ipk: 3.70,
    kota: "Yogyakarta",
    skills: ["Python", "Machine Learning"]
  },
  {
    nim: "230009",
    nama: "Indra Wijaya",
    prodi: "Sistem Informasi",
    angkatan: 2024,
    ipk: 3.35,
    kota: "Malang",
    skills: ["SQL", "UI/UX"]
  },
  {
    nim: "230010",
    nama: "Jihan Permata",
    prodi: "Informatika",
    angkatan: 2022,
    ipk: 3.95,
    kota: "Bandung",
    skills: ["Python", "MongoDB", "AI"]
  }
])
```

### 6.3 Memeriksa jumlah data

```javascript
db.mahasiswa.countDocuments()
```

Karena langkah sebelumnya memasukkan 1 document lalu 9 document, jumlah awal yang diharapkan adalah **10 mahasiswa**, selama setiap blok berhasil dan belum pernah dijalankan sebelumnya.

Lihat seluruh data:

```javascript
db.mahasiswa.find()
```

Agar tampilan lebih mudah dibaca:

```javascript
db.mahasiswa.find().pretty()
```

Jika jumlah data berbeda, periksa pesan hasil insert dan apakah blok `insertMany()` pernah dijalankan sebelumnya.

## 7. Read — Membaca Document

### 7.1 Membaca semua mahasiswa

```javascript
db.mahasiswa.find()
```

### 7.2 Membaca satu mahasiswa

```javascript
db.mahasiswa.findOne({ nim: "230003" })
```

### 7.3 Mencari berdasarkan program studi

```javascript
db.mahasiswa.find({ prodi: "Informatika" })
```

### 7.4 Mencari berdasarkan kota

```javascript
db.mahasiswa.find({ kota: "Surabaya" })
```

### 7.5 Menghitung jumlah mahasiswa

```javascript
db.mahasiswa.countDocuments()
```

Jumlah mahasiswa Informatika:

```javascript
db.mahasiswa.countDocuments({ prodi: "Informatika" })
```

## 8. Update — Memperbarui Document

Gunakan filter yang spesifik. Periksa data sebelum dan sesudah update.

### 8.1 Mengubah IPK dengan `$set`

```javascript
db.mahasiswa.updateOne(
  { nim: "230001" },
  { $set: { ipk: 3.75 } }
)
```

Periksa hasilnya:

```javascript
db.mahasiswa.findOne({ nim: "230001" })
```

### 8.2 Menambahkan field email

```javascript
db.mahasiswa.updateOne(
  { nim: "230001" },
  { $set: { email: "andi@example.com" } }
)
```

Periksa hasilnya:

```javascript
db.mahasiswa.findOne({ nim: "230001" })
```

### 8.3 Menambahkan elemen array dengan `$push`

```javascript
db.mahasiswa.updateOne(
  { nim: "230001" },
  { $push: { skills: "Data Science" } }
)
```

`$push` menambahkan elemen, termasuk jika nilai yang sama sudah ada. Untuk menambahkan nilai hanya jika belum ada, pertimbangkan `$addToSet`.

```javascript
db.mahasiswa.updateOne(
  { nim: "230001" },
  { $addToSet: { skills: "MongoDB" } }
)
```

### 8.4 Memperbarui banyak document dengan `updateMany()`

Contoh menaikkan field `status` untuk semua mahasiswa angkatan 2024:

```javascript
db.mahasiswa.updateMany(
  { angkatan: 2024 },
  { $set: { status: "Aktif" } }
)
```

Contoh ini menambahkan field baru `status` pada document yang cocok. Gunakan update massal hanya jika perubahan memang dimaksudkan untuk seluruh hasil filter.

### 8.5 Memahami hasil update

MongoDB mengembalikan informasi seperti jumlah document yang cocok (`matchedCount`) dan jumlah document yang berubah (`modifiedCount`). Jika `matchedCount` bernilai 0, periksa kembali filter dan tipe data field.

## 9. Delete — Menghapus Document

> **Peringatan:** operasi delete dapat menghilangkan data. Selalu jalankan `find()` dengan filter yang sama terlebih dahulu, dan pastikan filter sesuai target.

### 9.1 Menghapus satu document

Periksa target:

```javascript
db.mahasiswa.find({ nim: "230004" })
```

Jika benar, hapus:

```javascript
db.mahasiswa.deleteOne({ nim: "230004" })
```

Verifikasi:

```javascript
db.mahasiswa.find({ nim: "230004" })
```

### 9.2 Menghapus banyak document

Contoh berikut menghapus mahasiswa dengan program studi Teknik Komputer:

```javascript
db.mahasiswa.find({ prodi: "Teknik Komputer" })
```

Periksa hasil terlebih dahulu. Jika memang diperlukan, jalankan:

```javascript
db.mahasiswa.deleteMany({ prodi: "Teknik Komputer" })
```

### 9.3 Perbedaan `deleteOne()` dan `deleteMany()`

- `deleteOne(filter)` menghapus paling banyak satu document yang cocok.
- `deleteMany(filter)` menghapus semua document yang cocok.
- Filter kosong `{}` pada `deleteMany({})` akan menghapus semua document dalam collection. Jangan jalankan perintah tersebut pada data yang ingin dipertahankan.

## 10. Query dan Filtering

Operator berikut digunakan untuk memilih document berdasarkan kondisi.

| Operator | Makna | Contoh kondisi |
|---|---|---|
| `$gt` | Lebih besar dari | `ipk: { $gt: 3.5 }` |
| `$gte` | Lebih besar atau sama dengan | `ipk: { $gte: 3.5 }` |
| `$lt` | Lebih kecil dari | `ipk: { $lt: 3.5 }` |
| `$lte` | Lebih kecil atau sama dengan | `ipk: { $lte: 3.5 }` |
| `$eq` | Sama dengan | `angkatan: { $eq: 2023 }` |
| `$ne` | Tidak sama dengan | `kota: { $ne: "Surabaya" }` |
| `$in` | Nilai berada dalam daftar | `prodi: { $in: ["Informatika", "Sistem Informasi"] }` |
| `$nin` | Nilai tidak berada dalam daftar | `prodi: { $nin: ["Informatika"] }` |

### 10.1 IPK minimal 3,50

```javascript
db.mahasiswa.find({ ipk: { $gte: 3.50 } })
```

### 10.2 IPK di atas 3,50

```javascript
db.mahasiswa.find({ ipk: { $gt: 3.50 } })
```

### 10.3 IPK di antara 3,00 dan 3,60

```javascript
db.mahasiswa.find({
  ipk: {
    $gte: 3.00,
    $lte: 3.60
  }
})
```

Kedua kondisi pada field `ipk` tersebut berlaku bersamaan.

### 10.4 Mahasiswa dari beberapa program studi

```javascript
db.mahasiswa.find({
  prodi: { $in: ["Informatika", "Sistem Informasi"] }
})
```

### 10.5 Operator logika `$and`

Cari mahasiswa Informatika dengan IPK minimal 3,50:

```javascript
db.mahasiswa.find({
  $and: [
    { prodi: "Informatika" },
    { ipk: { $gte: 3.50 } }
  ]
})
```

Untuk kondisi pada field berbeda, bentuk ringkas berikut juga dapat digunakan:

```javascript
db.mahasiswa.find({
  prodi: "Informatika",
  ipk: { $gte: 3.50 }
})
```

### 10.6 Operator logika `$or`

Cari mahasiswa Informatika atau Sistem Informasi:

```javascript
db.mahasiswa.find({
  $or: [
    { prodi: "Informatika" },
    { prodi: "Sistem Informasi" }
  ]
})
```

### 10.7 Query pada array

Cari mahasiswa yang memiliki skill `Python`:

```javascript
db.mahasiswa.find({ skills: "Python" })
```

Cari mahasiswa yang memiliki kedua skill `Python` dan `MongoDB`:

```javascript
db.mahasiswa.find({
  skills: { $all: ["Python", "MongoDB"] }
})
```

## 11. Projection — Memilih Field yang Ditampilkan

Projection mengatur field yang muncul pada hasil query.

```javascript
db.mahasiswa.find(
  { prodi: "Informatika" },
  { nama: 1, prodi: 1, ipk: 1, _id: 0 }
)
```

Keterangan:

- `1` berarti field ditampilkan.
- `0` berarti field tidak ditampilkan.
- `_id` secara default ikut ditampilkan, sehingga dapat disembunyikan dengan `_id: 0`.
- Umumnya, jangan mencampur inclusion dan exclusion pada field biasa dalam satu projection, kecuali pengecualian `_id`.

## 12. Sorting, Limit, dan Skip

### 12.1 Urutkan IPK dari rendah ke tinggi

```javascript
db.mahasiswa.find().sort({ ipk: 1 })
```

### 12.2 Urutkan IPK dari tinggi ke rendah

```javascript
db.mahasiswa.find().sort({ ipk: -1 })
```

### 12.3 Urutkan nama secara alfabetis

```javascript
db.mahasiswa.find().sort({ nama: 1 })
```

### 12.4 Lima mahasiswa dengan IPK tertinggi

```javascript
db.mahasiswa.find()
  .sort({ ipk: -1 })
  .limit(5)
```

### 12.5 Pagination sederhana

Lewati lima hasil pertama dan tampilkan lima hasil berikutnya:

```javascript
db.mahasiswa.find()
  .sort({ nim: 1 })
  .skip(5)
  .limit(5)
```

Menggunakan field pengurutan yang stabil, misalnya `nim` yang unik, membantu hasil pagination tetap konsisten. Pada aplikasi dengan data sangat besar, pola pagination perlu dievaluasi lebih lanjut.

## 13. Indexing

Index membantu MongoDB menemukan document untuk pola query tertentu. Index memerlukan ruang penyimpanan dan perlu diperbarui saat data berubah, sehingga index tidak sebaiknya dibuat untuk semua field tanpa alasan.

### 13.1 Melihat index

```javascript
db.mahasiswa.getIndexes()
```

Setiap collection secara default memiliki index pada `_id`.

### 13.2 Membuat index pada NIM

```javascript
db.mahasiswa.createIndex({ nim: 1 })
```

### 13.3 Membuat unique index pada NIM

Untuk memastikan setiap mahasiswa memiliki NIM yang unik:

```javascript
db.mahasiswa.createIndex(
  { nim: 1 },
  { unique: true }
)
```

Jika data yang sudah ada memiliki NIM duplikat, pembuatan unique index akan gagal. Periksa dan bersihkan data duplikat sebelum membuat index. Pada data praktikum ini, nilai `nim` dirancang unik.

### 13.4 Membuat compound index

```javascript
db.mahasiswa.createIndex({ prodi: 1, ipk: -1 })
```

Index ini berpotensi membantu query yang memfilter `prodi` dan mengurutkan `ipk` secara menurun. Manfaat aktual bergantung pada bentuk query dan distribusi data.

### 13.5 Memeriksa index

```javascript
db.mahasiswa.getIndexes()
```

### 13.6 Menganalisis query dengan `explain()`

```javascript
db.mahasiswa.find({ prodi: "Informatika" })
  .explain("executionStats")
```

Perhatikan informasi seperti `nReturned`, `totalDocsExamined`, `totalKeysExamined`, dan tahap eksekusi. Untuk dataset yang sangat kecil, perbedaan performa mungkin tidak terlihat; tujuan latihan ini adalah memahami cara mengevaluasi rencana eksekusi, bukan membuktikan bahwa semua index selalu lebih cepat.

## 14. Aggregation Pipeline

Aggregation memproses document melalui beberapa tahap (*stage*) untuk menghasilkan ringkasan atau bentuk data baru.

### 14.1 Jumlah mahasiswa per program studi

```javascript
db.mahasiswa.aggregate([
  {
    $group: {
      _id: "$prodi",
      jumlah_mahasiswa: { $sum: 1 }
    }
  },
  {
    $sort: { jumlah_mahasiswa: -1 }
  }
])
```

Keterangan:

- `$group` mengelompokkan document berdasarkan nilai `prodi`.
- `_id: "$prodi"` menentukan field pengelompokan.
- `$sum: 1` menghitung jumlah document pada setiap kelompok.
- `$sort` mengurutkan jumlah secara menurun.

### 14.2 Rata-rata IPK setiap program studi

```javascript
db.mahasiswa.aggregate([
  {
    $group: {
      _id: "$prodi",
      rata_rata_ipk: { $avg: "$ipk" }
    }
  },
  {
    $sort: { rata_rata_ipk: -1 }
  }
])
```

### 14.3 IPK tertinggi dan terendah per program studi

```javascript
db.mahasiswa.aggregate([
  {
    $group: {
      _id: "$prodi",
      ipk_tertinggi: { $max: "$ipk" },
      ipk_terendah: { $min: "$ipk" }
    }
  },
  {
    $sort: { _id: 1 }
  }
])
```

### 14.4 Pipeline gabungan: filter, group, dan sort

Hitung jumlah, rata-rata IPK, IPK tertinggi, dan IPK terendah per program studi untuk mahasiswa dengan IPK minimal 3,00.

```javascript
db.mahasiswa.aggregate([
  {
    $match: {
      ipk: { $gte: 3.00 }
    }
  },
  {
    $group: {
      _id: "$prodi",
      jumlah_mahasiswa: { $sum: 1 },
      rata_rata_ipk: { $avg: "$ipk" },
      ipk_tertinggi: { $max: "$ipk" },
      ipk_terendah: { $min: "$ipk" }
    }
  },
  {
    $sort: { rata_rata_ipk: -1 }
  },
  {
    $project: {
      _id: 0,
      prodi: "$_id",
      jumlah_mahasiswa: 1,
      rata_rata_ipk: 1,
      ipk_tertinggi: 1,
      ipk_terendah: 1
    }
  }
])
```

Alur pipeline:

1. `$match`: memilih mahasiswa dengan IPK minimal 3,00.
2. `$group`: menghitung ringkasan untuk setiap program studi.
3. `$sort`: mengurutkan program studi berdasarkan rata-rata IPK.
4. `$project`: mengatur field keluaran.

## 15. Menggunakan MongoDB Compass

Bagian ini memperkenalkan akses GUI. Tampilan menu dapat sedikit berbeda menurut versi Compass.

### 15.1 Membuka koneksi

1. Jalankan MongoDB Compass.
2. Untuk server lokal standar, masukkan connection string:

   ```text
   mongodb://127.0.0.1:27017
   ```

3. Klik **Connect**.
4. Jika memakai autentikasi atau Atlas, gunakan connection string dan kredensial yang sesuai. Jangan membagikan password atau connection string rahasia.

### 15.2 Melihat database dan collection

1. Pilih database `kampus`.
2. Pilih collection `mahasiswa`.
3. Buka tab **Documents** untuk melihat data.
4. Gunakan filter untuk melakukan pencarian. Contoh filter JSON:

   ```json
   { "prodi": "Informatika" }
   ```

5. Untuk IPK minimal 3,50, gunakan:

   ```json
   { "ipk": { "$gte": 3.5 } }
   ```

### 15.3 Insert dan edit document

1. Pada collection `mahasiswa`, pilih fungsi menambah document.
2. Masukkan document JSON yang valid.
3. Simpan document.
4. Untuk mengubah document, buka document yang sesuai dan gunakan fungsi edit.
5. Pastikan hasil perubahan benar dengan menjalankan pencarian berdasarkan `nim`.

### 15.4 Aggregations dan indexes

1. Buka tab **Aggregations** untuk menyusun pipeline stage.
2. Coba stage `$match` dan `$group` yang telah dipelajari.
3. Buka tab **Indexes** untuk melihat atau membuat index.
4. Bandingkan hasil GUI dengan hasil yang diperoleh melalui `mongosh`.

## 16. Reset Data untuk Mengulang Praktikum (Opsional)

Gunakan hanya jika ingin mengulang dari awal dan yakin data dalam collection `mahasiswa` boleh dihapus.

Periksa database aktif dan collection terlebih dahulu:

```javascript
db
show collections
db.mahasiswa.countDocuments()
```

Untuk menghapus hanya collection praktikum:

```javascript
db.mahasiswa.drop()
```

Setelah itu, buat ulang collection:

```javascript
db.createCollection("mahasiswa")
```

**Perhatian:** `drop()` menghapus collection beserta document dan index di dalamnya. Jangan menjalankan reset pada database yang berisi data penting.

## 17. Tugas Praktikum

Kerjakan tugas berikut pada database `kampus` dan collection `mahasiswa`. Jika memakai data yang telah dimodifikasi pada tahap CRUD, catat perubahan tersebut dalam laporan.

### Tugas A — Instalasi dan struktur

1. Jelaskan sistem operasi dan metode instalasi yang digunakan.
2. Tunjukkan bukti bahwa `mongosh` berhasil terhubung.
3. Buat database `kampus` dan collection `mahasiswa`.
4. Jelaskan perbedaan database, collection, document, dan field.

### Tugas B — CRUD

1. Masukkan minimal 10 document mahasiswa.
2. Tampilkan semua mahasiswa.
3. Cari satu mahasiswa berdasarkan NIM.
4. Ubah IPK seorang mahasiswa.
5. Tambahkan field `email` pada salah satu document.
6. Tambahkan skill baru ke array `skills`.
7. Hapus satu document berdasarkan NIM.
8. Lampirkan hasil sebelum dan sesudah setiap perubahan.

### Tugas C — Query dan sorting

Tuliskan query dan lampirkan hasil untuk setiap pertanyaan berikut:

1. Cari mahasiswa dengan IPK minimal 3,50.
2. Cari mahasiswa dari program studi Informatika.
3. Cari mahasiswa angkatan 2023.
4. Cari mahasiswa dengan IPK antara 3,00 dan 3,60.
5. Cari mahasiswa dari Surabaya.
6. Cari mahasiswa yang memiliki skill Python.
7. Cari mahasiswa Informatika dengan IPK minimal 3,50.
8. Tampilkan hanya `nama`, `prodi`, dan `ipk`.
9. Urutkan mahasiswa berdasarkan IPK tertinggi.
10. Tampilkan lima mahasiswa dengan IPK tertinggi.

### Tugas D — Index dan aggregation

1. Tampilkan index yang tersedia.
2. Buat unique index pada `nim`.
3. Buat compound index pada `prodi` dan `ipk`.
4. Jalankan `explain("executionStats")` untuk query berdasarkan prodi.
5. Hitung jumlah mahasiswa per prodi.
6. Hitung rata-rata IPK per prodi.
7. Cari IPK tertinggi dan terendah per prodi.
8. Buat pipeline yang menggabungkan `$match`, `$group`, dan `$sort`.

### Tugas E — Analisis

Jawab secara singkat:

1. Mengapa MongoDB disebut document database?
2. Apa perbedaan JSON dan BSON?
3. Mengapa NIM cocok dibuat sebagai unique index?
4. Apa manfaat dan risiko schema flexibility?
5. Kapan embedding cocok digunakan? Berikan contoh dari document mahasiswa.
6. Mengapa index tidak sebaiknya dibuat pada semua field?
7. Apa informasi yang dapat dibaca dari `explain("executionStats")`?
8. Apa manfaat aggregation untuk laporan akademik?

## 18. Format Laporan Praktikum

Susun laporan dalam format berikut:

1. **Judul praktikum**
2. **Tujuan**
3. **Lingkungan praktikum** — sistem operasi, versi MongoDB, metode koneksi
4. **Langkah instalasi dan verifikasi**
5. **Rancangan document mahasiswa**
6. **Implementasi database dan collection**
7. **Hasil CRUD**
8. **Hasil query, projection, dan sorting**
9. **Hasil indexing dan `explain()`**
10. **Hasil aggregation**
11. **Jawaban analisis**
12. **Kesimpulan**

Sertakan screenshot atau salinan output terminal yang relevan. Jangan menyertakan password, token, atau connection string yang mengandung kredensial rahasia.


## 19. Referensi Resmi

Gunakan dokumentasi resmi untuk petunjuk instalasi yang sesuai dengan sistem operasi dan versi MongoDB:

- MongoDB Community Server: <https://www.mongodb.com/try/download/community>
- MongoDB Shell (`mongosh`): <https://www.mongodb.com/docs/mongodb-shell/>
- Panduan instalasi `mongosh`: <https://www.mongodb.com/docs/mongodb-shell/install/>
- Dokumentasi MongoDB Manual: <https://www.mongodb.com/docs/manual/>
- MongoDB Compass: <https://www.mongodb.com/try/download/compass>
- MongoDB Atlas: <https://www.mongodb.com/atlas>

---
