# Materi Teori CPMK-3 — Data Modeling NoSQL
## Perancangan Model Data NoSQL Berdasarkan Access Pattern, Query Pattern, dan Karakteristik Data

---

# 1. Identitas dan Fokus Pertemuan

## 1.1 CPMK-3 — Data Modeling

Ketika **merancang model data NoSQL** aspek utama yang perlu dipertimbangkan:

1. **Access pattern** pola atau cara aplikasi akan mengakses data
2. **Query pattern** pola query yang akan digunakan aplikasi untuk mengambil atau memanipulasi dat
3. **Cardinality** Berapa banyak hubungan/data
4. **Data volume** Seberapa banyak data
5. **Data velocity** Seberapa cepat data masuk/berubah
6. **Consistency** Seberapa seragam data harus selalu sama
7. **Availability** Seberapa sering sistem harus tersedia
8. **Scalability** Apakah sistem mampu berkembang

Kedelapan aspek tersebut tidak berdiri sendiri. Semuanya saling berhubungan dalam menentukan bagaimana data harus disimpan, didistribusikan, dibaca, ditulis, dan dikembangkan ketika **workload sistem meningkat**.


## 1.2 Fokus Utama Pertemuan

Pada relational database, proses pemodelan sering dimulai dari entity, relationship, ERD, normalisasi, kemudian tabel. Pada NoSQL, pendekatan yang ditekankan adalah **memulai dari bagaimana aplikasi menggunakan data**.

Prinsip utama:

> **Model the data around the queries and access patterns, bukan sekadar model the data around the entities.**

Artinya, **desain data harus mempertimbangkan terlebih dahulu bagaimana aplikasi akan membaca, menulis, memperbarui, dan mencari data**.

Pertanyaan awal yang harus dijawab:

- Data apa yang paling sering dibaca?
- Data apa yang paling sering ditulis?
- Bagaimana bentuk query yang paling sering dilakukan?
- Berapa banyak data yang berhubungan dengan satu entity?
- Apakah relationship tersebut bounded atau unbounded?
- Seberapa cepat data bertambah?
- Seberapa besar kebutuhan konsistensi?
- Seberapa penting sistem tetap tersedia ketika sebagian node gagal?
- Bagaimana data berkembang ketika jumlah pengguna, transaksi, request, atau perangkat meningkat?

---

# 2. Tujuan Pembelajaran

## 2.1 Tujuan Pembelajaran Umum

Setelah mengikuti pertemuan, mahasiswa diharapkan mampu menghubungkan kebutuhan aplikasi dengan rancangan model data NoSQL.

- Mahasiswa tidak hanya dituntut menghasilkan collection, document, key-value, atau tabel wide-column, tetapi juga mampu menjelaskan **alasan teknis** di balik struktur yang dipilih.
- Mahasiswa harus mampu mempertanggungjawabkan keputusan desain berdasarkan **kebutuhan aplikasi**, bukan hanya berdasarkan bentuk ERD.

Dengan demikian, jawaban:

> "Collection ini dibuat karena entity-nya berbeda."

belum cukup.

Jawaban yang lebih tepat harus menjelaskan:

> "Collection ini dipisahkan karena memiliki access pattern, lifecycle, cardinality, frekuensi update, dan kebutuhan konsistensi yang berbeda."



---

# 3. Peta Konsep Pertemuan

Alur berpikir data modeling NoSQL:

```text
Business Requirement
        │
        ▼
Identifikasi Entity & Data
        │
        ▼
Access Pattern
        │
        ▼
Query Pattern
        │
        ├──────────────┐
        ▼              ▼
Cardinality       Volume & Velocity
        │              │
        └──────┬───────┘
               ▼
     Consistency & Availability
               │
               ▼
           Scalability
               │
               ▼
       Pilihan Model NoSQL
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
   Embedding Referencing Denormalization
       │       │        │
       └───────┼────────┘
               ▼
        Physical Data Model
               │
               ▼
        Query Validation
               │
               ▼
      Performance Evaluation
```


## Penjelasan Alur

### Tahap 1 — Business Requirement

Identifikasi fungsi bisnis yang harus dilayani.

### Tahap 2 — Entity & Data

Identifikasi data yang dibutuhkan, tetapi jangan langsung menjadikan setiap entity sebagai collection.

### Tahap 3 — Access Pattern

Identifikasi bagaimana user atau service mengakses data.

### Tahap 4 — Query Pattern

Terjemahkan access pattern menjadi bentuk query konkret.

### Tahap 5 — Cardinality

Analisis jumlah hubungan antar-data.

### Tahap 6 — Volume & Velocity

Analisis jumlah data dan kecepatan data masuk/berubah.

### Tahap 7 — Consistency & Availability

Tentukan seberapa konsisten data harus dibaca dan seberapa penting layanan tetap tersedia.

### Tahap 8 — Scalability

Tentukan bagaimana sistem menangani pertumbuhan workload.

### Tahap 9 — Pilihan Model NoSQL

Pilih document, key-value, wide-column, graph, atau pendekatan lain yang sesuai.

### Tahap 10 — Physical Data Model

Tentukan struktur field, key, partition, index, embedded data, referenced data, dan strategi distribusi.

### Tahap 11 — Query Validation

Uji semua query utama terhadap model.

### Tahap 12 — Performance Evaluation

Evaluasi latency, throughput, read/write cost, storage, distribution, dan trade-off desain.

---

# 4. Bagian A — Pengantar Data Modeling NoSQL

## 4.1 Pengertian Data Modeling

**Data modeling** adalah proses menentukan:

- data apa yang disimpan;
- bagaimana data direpresentasikan;
- bagaimana data saling berhubungan;
- bagaimana data diakses;
- bagaimana data diubah;
- bagaimana data didistribusikan;
- bagaimana struktur data mendukung kebutuhan aplikasi.

Dalam NoSQL, data modeling bukan sekadar menentukan nama collection atau field. Data modeling juga mencakup keputusan mengenai:

- bentuk dokumen;
- key;
- relationship;
- embedding;
- referencing;
- denormalization;
- partition;
- shard;
- replication;
- index;
- retention;
- dan pola query.

## 4.2 Pendekatan SQL

Pendekatan yang umum digunakan:

```text
Entity
   ↓
Relationship
   ↓
ERD
   ↓
Normalization
   ↓
Table
```

Tujuan utamanya adalah menghasilkan struktur data yang terorganisasi dan mengurangi redundancy yang tidak diperlukan.

Contoh:

```text
CUSTOMER
---------
customer_id
name
email

ORDER
-----
order_id
customer_id
order_date

ORDER_ITEM
----------
order_id
product_id
qty
price
```

Untuk memperoleh detail order, aplikasi dapat membutuhkan beberapa operasi relasional.

## 4.3 Pendekatan NoSQL

Pendekatan yang ditekankan:

```text
Application Requirement
   ↓
Access Pattern
   ↓
Query Pattern
   ↓
Data Structure
   ↓
Denormalization / Embedding / Referencing
   ↓
Distributed Data Model
```

Contoh pemikiran:

```text
Kebutuhan:
"Customer sering membuka detail order"

        ↓

Query:
GET order by order_id

        ↓

Keputusan:
Simpan item order bersama order

        ↓

Hasil:
Satu document dapat melayani kebutuhan detail order
```

## 4.4 Prinsip Dasar

Pertanyaan penting bukan:

> "Bagaimana saya membuat tabel dari ERD?"

tetapi:

> "Bagaimana aplikasi membaca dan menulis data?"

---

# 5. Bagian B — Perbedaan Data Modeling SQL dan NoSQL

## 5.1 Perbandingan

| Aspek | SQL | NoSQL |
|---|---|---|
| Orientasi utama | Entity dan relationship | Access/query pattern |
| Struktur | Tabel | Document, key-value, wide-column, graph |
| Normalisasi | Umumnya tinggi | Denormalisasi sering digunakan |
| Join | Umum digunakan | Sering dihindari/dioptimalkan dengan desain |
| Schema | Relatif fixed | Flexible/schema-on-read atau schema-flexible |
| Distribusi | Bisa distributed | Umumnya dirancang untuk distributed system |
| Skalabilitas | Vertical + horizontal tertentu | Horizontal scaling menjadi perhatian penting |
| Konsistensi | Strong consistency sering menjadi default | Ditentukan sesuai kebutuhan dan kemampuan sistem |
| Data duplication | Umumnya dihindari | Dapat diterima jika mendukung performa read |

## 5.2 Entity-Centric vs Query-Centric

### Entity-centric

Berpikir:

```text
Customer
Product
Order
Payment
```

Kemudian menentukan relationship.

### Query-centric

Berpikir:

```text
Apa yang harus dilakukan aplikasi?

→ Get product by product_id
→ Get orders by customer_id
→ Get order by order_id
→ Get reviews by product_id
```

Kemudian struktur data dirancang untuk memenuhi pola tersebut.

## 5.3 Mengapa NoSQL Sering Menggunakan Denormalisasi?

Dalam sistem terdistribusi, join kompleks dapat menjadi mahal atau membutuhkan beberapa operasi. Karena itu, data yang sering dibaca bersama dapat disimpan bersama.

Namun denormalisasi bukan berarti:

> "Semua data harus disalin."

Denormalisasi harus memiliki alasan berdasarkan workload.

---

# 6. Bagian C — Access Pattern

## 6.1 Definisi

**Access pattern** adalah pola bagaimana aplikasi membaca, menulis, memperbarui, atau menghapus data.

Contoh:

- pengguna membuka halaman produk;
- pengguna melihat riwayat pesanan;
- administrator mencari transaksi;
- sistem mencatat event;
- sistem membaca data sensor;
- sistem menampilkan dashboard real-time.


## 6.2 Jenis Operasi

Access pattern dapat terdiri dari:

```text
Create
Read
Update
Delete
```

Namun pada sistem modern, pola juga dapat berupa:

```text
Read latest data
Read historical range
Read by owner
Read by status
Read by category
Write event
Update state
Append event
Aggregate data
```

## 6.3 Contoh Marketplace

Kebutuhan:

1. Menampilkan detail produk.
2. Menampilkan produk berdasarkan kategori.
3. Menampilkan keranjang.
4. Menampilkan order.
5. Menampilkan detail order.
6. Mencari transaksi berdasarkan status.
7. Menampilkan review.


Dari kebutuhan tersebut dibuat access pattern matrix.

| No | Aktor | Aktivitas | Operasi | Frekuensi |
|---|---|---|---|---|
| 1 | Customer | Melihat produk | Read | Sangat tinggi |
| 2 | Customer | Melihat kategori | Read | Tinggi |
| 3 | Customer | Membuat order | Write | Tinggi |
| 4 | Customer | Melihat order | Read | Tinggi |
| 5 | Admin | Mencari order | Read | Sedang |
| 6 | Customer | Menulis review | Write | Sedang |


## 6.4 Prinsip Co-Access

Jika dua data:

```text
selalu dibaca bersama
```

maka pertimbangkan untuk menyimpannya bersama.

Sebaliknya, jika data:

```text
sering berubah
+
memiliki lifecycle berbeda
+
tidak selalu dibaca bersama
```

maka pertimbangkan untuk memisahkannya.


## 6.5 Read-Heavy dan Write-Heavy

### Read-heavy

Jika:

```text
Read = 95%
Write = 5%
```

desain dapat lebih agresif dioptimalkan untuk read.

### Write-heavy

Jika:

```text
Read = 20%
Write = 80%
```

maka perlu memperhatikan:

- write throughput;
- index overhead;
- partition distribution;
- batching;
- append pattern;
- contention.

---

# 7. Bagian D — Query Pattern

## 7.1 Definisi

**Query pattern** adalah bentuk query yang berulang dan penting bagi aplikasi.

Contoh:

```text
GET product by product_id
GET products by category
GET orders by customer_id
GET order by order_id
GET orders by customer_id + date
GET transactions by status
```


## 7.2 Dari User Story ke Query

Contoh user story:

> "Customer ingin melihat semua order miliknya berdasarkan urutan terbaru."

Diterjemahkan menjadi:

```text
Query:
customer_id = ?
ORDER BY order_date DESC
LIMIT ?
```

Maka model harus mendukung:

```text
customer_id
+
order_date
```

## 7.3 Mengapa Query Pattern Penting?

NoSQL sering merancang struktur data agar query utama dapat dilakukan dengan sedikit operasi.

Contoh relasional:

```text
Order
   JOIN OrderItem
   JOIN Product
   JOIN Customer
```

Model document dapat menyimpan informasi yang memang dibutuhkan ketika detail order dibaca.

Contoh:

```json
{
  "order_id": "ORD001",
  "customer_id": "CUS001",
  "customer_name": "Budi",
  "order_date": "2026-09-16",
  "items": [
    {
      "product_id": "P001",
      "product_name": "Laptop",
      "qty": 1,
      "price": 12500000
    },
    {
      "product_id": "P002",
      "product_name": "Mouse",
      "qty": 2,
      "price": 350000
    }
  ],
  "total": 13200000
}
```

fileciteturn0file0L259-L300

## 7.4 Query-First Modeling

Proses:

```text
1. Daftar query
2. Urutkan berdasarkan frekuensi dan kepentingan
3. Tentukan data yang dibutuhkan setiap query
4. Tentukan key
5. Tentukan embedding/reference
6. Tentukan index
7. Uji query
```

## 7.5 Query Matrix

| Query ID | Query | Parameter | Hasil | Frekuensi |
|---|---|---|---|---|
| Q1 | Product by ID | product_id | 1 product | Sangat tinggi |
| Q2 | Product by category | category_id | list product | Sangat tinggi |
| Q3 | Order history | customer_id | list order | Tinggi |
| Q4 | Order detail | order_id | 1 order | Sangat tinggi |
| Q5 | Review | product_id | list review | Tinggi |

---

# 8. Bagian E — Cardinality

## 8.1 Definisi

**Cardinality** menunjukkan jumlah hubungan antar-data.

Contoh:

- 1 customer → banyak order;
- 1 order → banyak order item;
- 1 product → banyak review;
- 1 sensor → sangat banyak measurement.

fileciteturn0file0L306-L317

## 8.2 One-to-One

```text
User ───── Profile
  1          1
```

Data profile dapat dipertimbangkan untuk embedding karena relationship-nya kecil dan bounded.

Contoh:

```json
{
  "user_id": "U001",
  "profile": {
    "name": "Andi",
    "email": "andi@example.com"
  }
}
```

## 8.3 One-to-Many

```text
Customer ─────< Order
    1           N
```

Tidak otomatis berarti order harus di-embed.

Pertanyaan yang harus dijawab:

- Berapa maksimum order?
- Apakah jumlahnya terus bertambah?
- Apakah order selalu dibaca bersama customer?
- Apakah order memiliki lifecycle sendiri?
- Apakah order sering dicari secara independen?

fileciteturn0file0L321-L337

## 8.4 Many-to-Many

```text
Student >────< Course
```

Relationship many-to-many sering membutuhkan referencing atau struktur tambahan.

fileciteturn0file0L339-L345

## 8.5 Bounded vs Unbounded

### Bounded

Jumlah elemen memiliki batas praktis.

Contoh:

```text
Order → OrderItem
```

Sebuah order biasanya memiliki jumlah item yang masih dapat dibatasi secara operasional.

### Unbounded

Jumlah elemen dapat terus bertambah.

Contoh:

```text
Product → Review
Customer → Order
Device → Measurement
```

Jika array tumbuh tanpa batas, embedding dapat menyebabkan document terlalu besar dan sulit dikelola.

fileciteturn0file0L347-L376

---

# 9. Bagian F — Data Volume

## 9.1 Definisi

**Data volume** adalah jumlah data yang harus disimpan dan dikelola.

Contoh:

```text
1 juta user
10 juta order
100 juta event
1 miliar log
```

fileciteturn0file0L380-L392

## 9.2 Estimasi Volume

Rumus dasar:

```text
Total Data ≈ jumlah record × ukuran rata-rata record
```

Contoh:

```text
100.000.000 event
× 500 byte/event
= 50.000.000.000 byte
≈ 50 GB
```

Jika terdapat replication factor 3:

```text
50 GB × 3 = 150 GB
```

Namun kapasitas aktual perlu memperhitungkan:

- index;
- metadata;
- storage overhead;
- backup;
- temporary data;
- growth;
- free-space requirement.

fileciteturn0file0L395-L423

## 9.3 Volume Bulanan dan Tahunan

Jika data bertambah:

```text
5 juta record/hari
```

maka:

```text
per 30 hari ≈ 150 juta record
per 365 hari ≈ 1,825 miliar record
```

Perhitungan semacam ini membantu menentukan apakah data:

- disimpan seluruhnya;
- diarsipkan;
- diagregasi;
- atau memiliki retention period.

## 9.4 Dampak Volume terhadap Model

Volume besar memengaruhi:

- partition key;
- shard key;
- indexing;
- retention;
- archival;
- distribution;
- storage cost;
- query performance.


---

# 10. Bagian G — Data Velocity

## 10.1 Definisi

**Velocity** adalah kecepatan data dibuat, masuk, diproses, dan berubah.

Contoh:

```text
10 event/detik
1.000 event/detik
100.000 event/detik
1 juta event/detik
```

fileciteturn0file0L440-L452

## 10.2 Konversi Rate

Jika:

```text
5.000 event/detik
```

maka:

```text
per menit = 5.000 × 60
           = 300.000 event

per jam   = 300.000 × 60
           = 18.000.000 event

per hari  = 18.000.000 × 24
           = 432.000.000 event
```

fileciteturn0file0L455-L468

## 10.3 Average vs Peak

Rata-rata tidak selalu menggambarkan beban aktual.

Misalnya:

```text
Average = 1.157 event/s
Peak factor = 10×
```

maka:

```text
Peak ≈ 11.570 event/s
```

fileciteturn0file0L915-L920

## 10.4 Velocity dan Hot Partition

Jika write sangat tinggi diarahkan ke satu partition:

```text
100.000 write/s
       ↓
satu partition
       ↓
hot partition
       ↓
bottleneck
       ↓
latency meningkat
```

Karena itu data velocity harus dipertimbangkan ketika memilih partition key.

---

# 11. Bagian H — Consistency

## 11.1 Definisi

**Consistency** berkaitan dengan sejauh mana data yang dibaca mencerminkan state data yang benar sesuai kebutuhan sistem.

Pertanyaan utamanya:

> "Seberapa penting agar pembacaan segera melihat nilai terbaru yang benar?"

## 11.2 Strong Consistency

Untuk contoh kasus dalam outline, kebutuhan strong consistency lebih relevan pada:

- saldo rekening;
- stok kritis;
- pembayaran;
- status transaksi finansial.

fileciteturn0file0L483-L495

Jika stok:

```text
stock = 1
```

dan dua customer mencoba membeli secara bersamaan, desain harus mencegah kondisi bisnis yang menghasilkan overselling.

## 11.3 Eventual Consistency

Dalam konteks tertentu, beberapa data dapat mentoleransi keterlambatan sementara, misalnya:

- jumlah like;
- counter statistik;
- feed;
- data rekomendasi;
- beberapa jenis cache.

fileciteturn0file0L496-L502

Contoh:

```text
User A melakukan like
       ↓
replica/event processor
       ↓
counter diperbarui beberapa saat kemudian
```

Jika kebutuhan bisnis mengizinkan keterlambatan tersebut, eventual consistency dapat dipertimbangkan.

## 11.4 Pertanyaan Desain Consistency

Untuk setiap data tanyakan:

1. Apakah data harus langsung konsisten?
2. Berapa lama keterlambatan yang masih dapat diterima?
3. Apa konsekuensi jika user membaca data yang belum terbaru?
4. Apakah transaksi membutuhkan atomicity?
5. Apakah consistency harus berlaku lintas beberapa record?

fileciteturn0file0L504-L512

---

# 12. Bagian I — Availability

## 12.1 Definisi

**Availability** adalah kemampuan sistem untuk tetap melayani request ketika terjadi gangguan.

Contoh cluster:

```text
Node A
Node B
Node C
```

Jika Node A gagal, node lain diharapkan dapat melanjutkan layanan sesuai mekanisme sistem.


## 12.2 Faktor Availability

Faktor yang perlu diperhatikan:

- replication;
- failover;
- quorum;
- replica placement;
- network partition;
- recovery;
- backup;
- disaster recovery.



## 12.3 Replication

Secara konseptual:

```text
Primary / Leader
       │
 ┌─────┼─────┐
 ▼     ▼     ▼
R1     R2     R3
```

Tujuan replication antara lain menyediakan salinan data sehingga kegagalan node tidak otomatis menghilangkan seluruh salinan.

## 12.4 Trade-Off Availability

Availability tinggi dapat membutuhkan:

- replication lebih banyak;
- infrastructure lebih besar;
- mekanisme conflict resolution;
- kemungkinan penerimaan eventual consistency pada skenario tertentu.

fileciteturn0file0L543-L550

---

# 13. Bagian J — Scalability

## 13.1 Definisi

**Scalability** adalah kemampuan sistem untuk menangani pertumbuhan beban.

Pertumbuhan dapat berupa:

- jumlah user;
- request;
- write;
- data;
- query;
- node;
- region.


## 13.2 Vertical Scaling

```text
Server kecil
    ↓
CPU/RAM/storage lebih besar
    ↓
Server lebih besar
```

Kelebihan konseptual:

- arsitektur relatif sederhana.

Keterbatasan:

- ada batas kapasitas hardware;
- dapat menghasilkan single-system dependency jika tidak disertai redundancy.

## 13.3 Horizontal Scaling

```text
Node A
Node B
Node C
Node D
```

Beban dan data dapat didistribusikan ke lebih banyak node sesuai kemampuan sistem.

## 13.4 Scalability Harus Dirancang dari Awal

Jika sistem awal:

```text
1 juta record
```

kemudian menjadi:

```text
1 miliar record
```

maka model yang tadinya bekerja dapat mengalami:

- partition terlalu besar;
- index membesar;
- query semakin lambat;
- biaya storage meningkat;
- write hotspot;
- maintenance semakin berat.

---

# 14. Bagian K — Embedding vs Referencing

# 14.1 Embedding

**Embedding** berarti data yang berhubungan erat disimpan di dalam document yang sama.

Contoh:

```json
{
  "product_id": "P001",
  "name": "Laptop ABC",
  "price": 12500000,
  "seller": {
    "seller_id": "S001",
    "name": "Toko ABC"
  }
}
```

### Kelebihan

- read lebih sederhana;
- mengurangi kebutuhan join/application-side lookup;
- cocok untuk data yang dibaca bersama.

### Kekurangan

- duplication;
- update lebih kompleks jika embedded data sering berubah;
- document dapat membesar.

## 14.2 Kapan Embedding Cocok?

Pertimbangkan embedding jika:

1. Data hampir selalu dibaca bersama.
2. Relationship relatif bounded.
3. Data embedded jarang berubah.
4. Ukuran document masih terkontrol.
5. Query utama memang berorientasi pada parent document.

## 14.3 Referencing

**Referencing** berarti data disimpan terpisah dan dihubungkan menggunakan identifier.

Order:

```json
{
  "order_id": "ORD001",
  "customer_id": "CUS001"
}
```

Customer:

```json
{
  "customer_id": "CUS001",
  "name": "Budi"
}
```

### Kelebihan

- mengurangi duplication;
- cocok untuk relationship besar/unbounded;
- update entity dapat dilakukan di satu tempat.

### Kekurangan

- membutuhkan query tambahan;
- dapat meningkatkan latency;
- aplikasi melakukan lookup/join-like operation.

fileciteturn0file0L647-L657

## 14.4 Decision Matrix

| Pertanyaan | Embedding | Referencing |
|---|---|---|
| Selalu dibaca bersama? | Cocok | Kurang ideal |
| Relationship bounded? | Cocok | Tidak wajib |
| Relationship unbounded? | Berisiko | Lebih sesuai |
| Sering berubah independen? | Berisiko | Cocok |
| Mengutamakan read sederhana? | Cocok | Kurang sederhana |
| Mengurangi duplication? | Kurang | Cocok |

---

# 15. Bagian L — Denormalization

## 15.1 Pengertian

**Denormalization** adalah pengulangan data yang dilakukan secara sengaja untuk mendukung performa atau pola akses.

Contoh:

```json
{
  "order_id": "ORD001",
  "customer_id": "CUS001",
  "customer_name": "Budi",
  "customer_city": "Jakarta"
}
```

Data customer dapat tetap memiliki collection master sendiri.

## 15.2 Tujuan

Query:

```text
Get order detail
```

dapat langsung memperoleh:

```text
order
+
customer display information
```

tanpa lookup tambahan.


## 15.3 Risiko Denormalization

Jika customer berubah:

```text
Customer.name = "Budi Santoso"
```

data yang telah diduplikasi mungkin masih:

```text
"Budi"
```

Maka harus ditentukan apakah data tersebut merupakan:

1. **snapshot historis**, atau
2. **data yang harus selalu mengikuti master**.

## 15.4 Denormalization yang Sehat

Setiap duplication harus memiliki alasan:

```text
Query tertentu
    ↓
membutuhkan field tertentu
    ↓
field diduplikasi
    ↓
query lebih sederhana
```

Tetapi harus tersedia strategi untuk menangani perubahan.

---

# 16. Bagian M — Framework Analisis Data Modeling

Gunakan urutan:

```text
1. Identifikasi business requirement
          ↓
2. Identifikasi entity/data
          ↓
3. Daftar access pattern
          ↓
4. Daftar query pattern
          ↓
5. Analisis cardinality
          ↓
6. Hitung volume
          ↓
7. Hitung velocity
          ↓
8. Tentukan consistency requirement
          ↓
9. Tentukan availability requirement
          ↓
10. Tentukan scalability requirement
          ↓
11. Pilih jenis NoSQL
          ↓
12. Tentukan embedding/reference
          ↓
13. Tentukan partition/shard strategy
          ↓
14. Tentukan index
          ↓
15. Uji terhadap query
```

## 16.1 Prinsip "Jangan Langsung Membuat Collection"

Kesalahan umum:

```text
Entity → Collection
```

Lebih tepat:

```text
Requirement
→ Access Pattern
→ Query Pattern
→ Cardinality
→ Workload
→ Model
```

Satu entity dapat:

- di-embed;
- direference;
- didenormalisasi;
- disimpan pada model read-optimized;
- atau tidak menjadi collection mandiri.

---

# 17. Studi Kasus Utama — Platform E-Commerce Marketplace

## 17.1 Skenario

Marketplace memiliki:

- 10 juta customer;
- 500 ribu seller;
- 50 juta product;
- jutaan order per hari;
- puluhan juta product views per hari;
- review produk;
- cart;
- payment;
- order tracking.

Sistem harus menangani:

- pembacaan katalog sangat tinggi;
- transaksi order;
- pencarian order;
- riwayat transaksi;
- monitoring status pengiriman.

fileciteturn0file0L753-L775

---

# 18. Business Requirement Marketplace

Sistem harus dapat:

1. Menampilkan detail produk dengan cepat.
2. Menampilkan produk berdasarkan kategori.
3. Menampilkan produk berdasarkan seller.
4. Menyimpan shopping cart.
5. Membuat order.
6. Menampilkan detail order.
7. Menampilkan riwayat order customer.
8. Menampilkan review produk.
9. Menampilkan status pengiriman.
10. Menangani pertumbuhan transaksi secara horizontal.

fileciteturn0file0L779-L791

## 18.1 Identifikasi Entity

Entity utama:

```text
Customer
Seller
Product
Category
Cart
Order
OrderItem
Payment
Shipment
Review
```

Tetapi:

> **Entity tidak otomatis sama dengan collection.**

Struktur akhir harus ditentukan berdasarkan access pattern.

fileciteturn0file0L796-L815

---

# 19. Access Pattern Marketplace

| ID | Access Pattern | Operasi | Prioritas |
|---|---|---|---|
| AP01 | Detail product | Read | Sangat tinggi |
| AP02 | Product by category | Read | Sangat tinggi |
| AP03 | Product by seller | Read | Tinggi |
| AP04 | Customer cart | Read/Write | Tinggi |
| AP05 | Create order | Write | Sangat tinggi |
| AP06 | Order detail | Read | Sangat tinggi |
| AP07 | Customer order history | Read | Tinggi |
| AP08 | Product reviews | Read | Tinggi |
| AP09 | Update shipment status | Write | Tinggi |
| AP10 | Query order by status | Read | Sedang |

fileciteturn0file0L819-L832

## 19.1 Interpretasi

AP01 dan AP02 sangat read-heavy.

AP05 sangat penting pada write path.

AP07 membutuhkan query berdasarkan customer.

AP08 berpotensi memiliki cardinality besar.

AP10 membutuhkan struktur yang mendukung pencarian berdasarkan status dan waktu.

---

# 20. Query Pattern Marketplace

Query utama:

```text
Q1:
Cari product berdasarkan product_id.

Q2:
Cari product berdasarkan category_id.

Q3:
Cari semua order milik customer_id.

Q4:
Cari detail order berdasarkan order_id.

Q5:
Cari cart berdasarkan customer_id.

Q6:
Cari review berdasarkan product_id.

Q7:
Cari shipment berdasarkan order_id.

Q8:
Cari order berdasarkan customer_id + order_date.

Q9:
Cari order berdasarkan status + order_date.
```

fileciteturn0file0L836-L868

## 20.1 Dampak terhadap Model

Dari query tersebut terlihat bahwa key dan index tidak dapat dipilih secara acak.

Contoh:

```text
Q8:
customer_id + order_date
```

menunjukkan kebutuhan terhadap struktur yang mendukung:

```text
customer_id
+
order_date
```

Sedangkan:

```text
Q6:
product_id
```

menunjukkan bahwa review harus dapat ditemukan secara efisien berdasarkan product.

---

# 21. Analisis Cardinality Marketplace

| Relationship | Cardinality | Implikasi |
|---|---:|---|
| Customer → Order | 1:N | Banyak, hati-hati jika embedded |
| Order → OrderItem | 1:N | Dapat embedded jika bounded |
| Product → Review | 1:N | Unbounded, pertimbangkan collection terpisah |
| Customer → Cart | 1:1 | Dapat dipisahkan atau embedded |
| Seller → Product | 1:N | Bergantung pola akses |
| Order → Shipment | 1:1 / 1:N tergantung bisnis | Perhatikan lifecycle |
| Product → Category | N:1 / N:M | Bergantung struktur kategori |

fileciteturn0file0L871-L881

---

# 22. Analisis Volume Marketplace

Data contoh:

```text
Customer       = 10 juta
Seller         = 500 ribu
Product        = 50 juta
Order          = 5 juta/hari
Review         = 2 juta/hari
Product View   = 100 juta/hari
```

Product view memiliki volume jauh lebih tinggi daripada customer master.

Karena karakteristik workload berbeda, product view tidak seharusnya diperlakukan dengan model yang sama seperti master customer.

fileciteturn0file0L885-L900

## 22.1 Perbedaan Master dan Event

### Master data

Contoh:

```text
Customer
Seller
Product
```

Karakteristik:

- relatif stabil;
- update tidak setinggi event;
- query berdasarkan identifier atau atribut tertentu.

### Event data

Contoh:

```text
Product View
Click
Search
Tracking Event
```

Karakteristik:

- append-heavy;
- volume tinggi;
- velocity tinggi;
- memiliki timestamp;
- membutuhkan retention/aggregation.

---

# 23. Analisis Velocity Marketplace

Contoh:

```text
Product View = 100 juta event/hari
```

Rata-rata:

```text
100.000.000 / 86.400
≈ 1.157 event/detik
```

Jika peak = 10×:

```text
≈ 11.570 event/detik
```

fileciteturn0file0L904-L920

Maka perlu mempertimbangkan:

- distributed writes;
- partitioning;
- batching;
- buffering/event streaming;
- retention;
- hot partition prevention.

fileciteturn0file0L923-L930

---

# 24. Analisis Consistency Marketplace

| Data | Kebutuhan Consistency | Alasan |
|---|---|---|
| Customer profile | Sedang | Perubahan tidak selalu kritis |
| Product description | Sedang | Tidak selalu berubah |
| Product price | Tinggi | Berpengaruh pada transaksi |
| Stock | Tinggi | Risiko overselling |
| Order | Sangat tinggi | Status transaksi harus dapat dipercaya |
| Payment | Sangat tinggi | Data finansial |
| Review count | Dapat lebih longgar | Counter dapat ditangani terpisah |
| Product view | Dapat eventual | Statistik tidak selalu harus real-time |

fileciteturn0file0L934-L945

## 24.1 Prinsip

Tidak semua data dalam satu aplikasi harus mempunyai level consistency yang sama.

Misalnya:

```text
Payment
→ consistency sangat penting

Product View
→ keterlambatan sementara dapat ditoleransi dalam konteks statistik
```

---

# 25. Analisis Availability Marketplace

Layanan yang perlu diperhatikan:

```text
Product Catalog
Cart
Order
Shipment Tracking
```

Availability penting karena kegagalan layanan dapat mengganggu pengalaman pengguna.

Model konseptual:

```text
Application
     │
     ▼
Load Balancer
     │
 ┌───┼───┐
 ▼   ▼   ▼
DB1 DB2 DB3
```

Dengan replication dan failover yang sesuai.

fileciteturn0file0L949-L975

---

# 26. Contoh Model Document Product

```json
{
  "_id": "P001",
  "name": "Laptop ABC",
  "category_id": "C001",
  "seller_id": "S001",
  "price": 12500000,
  "stock": 100,
  "attributes": {
    "brand": "ABC",
    "ram": "16GB",
    "storage": "1TB SSD"
  },
  "rating_summary": {
    "average": 4.7,
    "count": 1250
  },
  "status": "ACTIVE"
}
```

Data yang dibutuhkan untuk product detail ditempatkan bersama.

## 26.1 Analisis

Field:

```text
_id
name
category_id
seller_id
price
stock
attributes
rating_summary
status
```

memiliki fungsi yang berbeda.

### Identifier

```text
_id
```

digunakan untuk mengidentifikasi product.

### Relationship reference

```text
category_id
seller_id
```

menghubungkan product dengan entity lain.

### Embedded data

```text
attributes
rating_summary
```

dapat dibaca sebagai bagian dari product detail.

---

# 27. Contoh Model Document Order

```json
{
  "_id": "ORD001",
  "customer_id": "CUS001",
  "order_date": "2026-09-16T10:15:00Z",
  "status": "PAID",
  "shipping": {
    "address": "Jakarta",
    "courier": "XYZ"
  },
  "items": [
    {
      "product_id": "P001",
      "product_name": "Laptop ABC",
      "qty": 1,
      "unit_price": 12500000,
      "subtotal": 12500000
    },
    {
      "product_id": "P002",
      "product_name": "Mouse ABC",
      "qty": 2,
      "unit_price": 350000,
      "subtotal": 700000
    }
  ],
  "total": 13200000,
  "payment": {
    "method": "BANK_TRANSFER",
    "status": "PAID"
  }
}
```


## 27.1 Mengapa OrderItem Di-Embed?

Ketika user membuka detail order, item hampir selalu dibutuhkan bersamaan.

Selain itu:

```text
unit_price
subtotal
product_name
```

dapat menjadi bagian dari snapshot historis transaksi.

Harga product master dapat berubah:

```text
Harga sekarang = Rp13.000.000
```

tetapi order lama mungkin menggunakan:

```text
Harga saat transaksi = Rp12.500.000
```

Karena itu harga transaksi tidak seharusnya bergantung pada harga product master saat ini.

fileciteturn0file0L1044-L1048

---

# 28. Contoh Model Review

```json
{
  "_id": "REV001",
  "product_id": "P001",
  "customer_id": "CUS001",
  "rating": 5,
  "comment": "Produk sangat baik",
  "created_at": "2026-09-16T12:00:00Z"
}
```

Query utama:

```text
product_id + created_at
```

Karena jumlah review dapat sangat besar, review lebih tepat dipikirkan sebagai data yang memiliki lifecycle dan volume sendiri daripada array yang terus membesar di dalam product.

---

# 29. Contoh Model Customer Order History

Kebutuhan:

```text
Get orders by customer_id
```

Dapat menggunakan representation:

```json
{
  "customer_id": "CUS001",
  "order_date": "2026-09-16T10:15:00Z",
  "order_id": "ORD001",
  "status": "PAID",
  "total": 13200000
}
```

Tujuannya mengoptimalkan query riwayat order tanpa membaca seluruh detail order.

Ini adalah contoh **read-optimized / denormalized model**.


---

# 30. Evaluasi Model dengan Query

Setiap desain harus diuji terhadap query.

| Query | Data yang Dibaca | Target |
|---|---|---|
| Product detail | Product document | 1 document |
| Customer order history | Order summary | Banyak record terurut |
| Order detail | 1 order document | 1 document |
| Product review | Review partition | Range query |
| Cart | Cart document | 1 document |
| Shipment | Shipment/order | 1 atau beberapa record |

## 30.1 Query Validation

Contoh:

```text
Q1: Product by ID
```

Jika membutuhkan:

```text
1 lookup
```

model relatif sederhana.

Jika:

```text
Q1
→ product
→ seller
→ category
→ rating
→ review
→ promotion
```

harus dipertanyakan apakah model terlalu normalized untuk workload tersebut.

## 30.2 Prinsip

Jika query penting membutuhkan terlalu banyak lookup, desain perlu dievaluasi kembali.

Namun "sedikit lookup" bukan satu-satunya tujuan. Perlu mempertimbangkan:

- ukuran document;
- write amplification;
- consistency;
- duplication;
- update frequency;
- storage.

---

# 31. Bagian N — Studi Kasus Kedua: Sistem IoT Monitoring

## 31.1 Skenario

Sistem:

```text
10.000 device
100 sensor/device
1 measurement/10 detik
```

Jumlah measurement per menit:

```text
10.000 × 100 × 6
= 6.000.000 measurement/menit
```

Jumlah per hari:

```text
6.000.000 × 60 × 24
= 8.640.000.000 measurement/hari
```

Ini merupakan contoh workload dengan volume dan velocity sangat tinggi.


## 31.2 Access Pattern IoT

Kebutuhan:

1. Ambil data terbaru device.
2. Ambil data sensor device selama 1 jam.
3. Ambil data sensor selama 24 jam.
4. Ambil agregasi per menit.
5. Ambil agregasi per jam.
6. Cari device berdasarkan lokasi.
7. Deteksi anomaly.

Query penting:

```text
device_id + sensor_id + timestamp range
```

## 31.3 Contoh Measurement Document

```json
{
  "device_id": "DEV001",
  "sensor_id": "TEMP001",
  "timestamp": "2026-09-16T10:15:00Z",
  "temperature": 27.4,
  "humidity": 72.5
}
```


## 31.4 Time Bucket

Jangan hanya berpikir:

```text
device_id
```

tetapi pertimbangkan:

```text
device_id + time bucket
```

Contoh:

```text
device_id = DEV001
date = 2026-09-16
```

Tujuannya menghindari partition yang terus membesar tanpa batas.

## 31.5 Pelajaran dari Kasus IoT

Kasus ini memperlihatkan hubungan langsung antara:

```text
Velocity
+
Volume
+
Time-series query
+
Partitioning
+
Retention
```

Jika data disimpan tanpa mempertimbangkan time dimension, query historical range dan pertumbuhan partition dapat menjadi masalah.

---

# 32. Bagian O — Studi Kasus Ketiga: Social Media Feed

## 32.1 Skenario

Platform memiliki:

- jutaan pengguna;
- jutaan post;
- banyak follower;
- pengguna sering membuka home feed;
- post memiliki like dan comment;
- jumlah interaksi sangat tinggi.

## 32.2 Access Pattern

```text
Get home feed by user
Get post by post_id
Get comments by post_id
Get posts by author
Record like
Record comment
```

## 32.3 Cardinality Challenge

Satu user dapat memiliki:

```text
followers = 10
followers = 1.000
followers = 1.000.000
```

Artinya cardinality dan distribution harus diperhatikan.

Model yang sesuai untuk user dengan sedikit follower tidak otomatis sesuai untuk user dengan jutaan follower.

## 32.4 Pelajaran

Social media menunjukkan bahwa **skew** atau distribusi yang tidak merata dapat sama pentingnya dengan jumlah data rata-rata.

Jika satu account menghasilkan traffic jauh lebih tinggi dibandingkan account lain, desain partition dan write strategy harus mempertimbangkan kondisi tersebut.

---

# 33. Bagian P — Pemilihan Jenis NoSQL

## 33.1 Document Database

Contoh penggunaan:

- product catalog;
- customer profile;
- order document;
- content management.

Model:

```text
Document
 ├── field
 ├── field
 └── nested document
```


Ciri pemodelan:

- struktur nested;
- field yang fleksibel;
- cocok untuk aggregate yang sering dibaca sebagai satu unit.

## 33.2 Key-Value

Cocok untuk:

- session;
- cache;
- shopping cart;
- lookup sederhana.

Model:

```text
key → value
```

Contoh:

```text
cart:CUS001 → {...}
```


Kekuatan utama adalah lookup berdasarkan key.

## 33.3 Wide-Column

Cocok untuk:

- event;
- time-series;
- telemetry;
- high-volume write;
- distributed workloads.

Model konseptual:

```text
partition key
      +
clustering/sort key
      +
columns
```

fileciteturn0file0L1288-L1305

Kasus IoT merupakan contoh yang sesuai dengan pola:

```text
device_id
+
sensor_id
+
time
```

## 33.4 Graph Database

Cocok untuk:

- social network;
- recommendation;
- fraud relationship;
- network topology.

Model:

```text
Node ── Edge ── Node
```


Contoh:

```text
(User A)
   │ follows
   ▼
(User B)
   │ likes
   ▼
(Post C)
```

Model graph menempatkan relationship sebagai bagian penting dari model.

## 33.5 Prinsip Pemilihan

Jangan memilih:

```text
"NoSQL → MongoDB"
```

secara otomatis.

Gunakan:

```text
Workload
+
Query
+
Cardinality
+
Volume
+
Velocity
+
Consistency
+
Availability
+
Scalability
→
Model/technology choice
```

---

# 34. Bagian Q — Partition Key dan Sharding

## 34.1 Konsep

Dalam sistem distributed, data perlu didistribusikan ke node.

Contoh:

```text
partition_key = customer_id
```


## 34.2 Cardinality Partition Key

Partition key sebaiknya memiliki cardinality yang cukup untuk mendistribusikan data.

Contoh buruk secara konseptual:

```text
country = Indonesia
```

Jika mayoritas data memiliki nilai tersebut:

```text
Indonesia
   ↓
partition sangat besar
   ↓
hot partition
```


## 34.3 Faktor Pemilihan Partition Key

Harus mempertimbangkan:

- cardinality;
- distribution;
- query pattern;
- write distribution;
- data growth.


## 34.4 Sharding

Secara konseptual:

```text
Dataset
   │
   ├── Shard 1
   ├── Shard 2
   ├── Shard 3
   └── Shard 4
```

Tujuan:

- mendistribusikan storage;
- mendistribusikan workload;
- memungkinkan horizontal scaling.

Tetapi sharding juga menimbulkan kebutuhan:

- routing;
- balancing;
- rebalancing;
- distributed query;
- monitoring.

---

# 35. Bagian R — Indexing

## 35.1 Prinsip

Index dibuat berdasarkan query yang benar-benar diperlukan.

Contoh:

```text
Query:
customer_id + order_date
```

Maka index/ordering perlu mendukung pola tersebut.


## 35.2 Mengapa Tidak Semua Field Di-index?

Index dapat:

- menambah storage;
- memperbesar biaya write;
- meningkatkan maintenance;
- tidak digunakan oleh query.

## 35.3 Index vs Partition Key

Secara konseptual:

```text
Partitioning
→ menentukan lokasi/distribusi data

Index
→ membantu menemukan data dalam struktur penyimpanan
```

Keduanya tidak sama dan dapat memiliki konsekuensi berbeda terhadap distributed workload.

---

# 36. Bagian S — Anti-Pattern Data Modeling NoSQL

## 36.1 Anti-Pattern 1 — Membuat NoSQL Persis Seperti SQL

Contoh:

```text
Customer
Order
OrderItem
Product
Category
```

kemudian setiap query memerlukan banyak join/lookup.

Masalah:

- query kompleks;
- latency meningkat;
- distributed join mahal.


### Perbaikan

Mulai dari query:

```text
Get Order Detail
```

kemudian tentukan data yang harus tersedia dalam satu read path.

---

## 36.2 Anti-Pattern 2 — Embedding Tidak Terbatas

Contoh:

```json
{
  "customer_id": "C001",
  "orders": [
    "... jutaan order ..."
  ]
}
```

Masalah:

- document membesar;
- update sulit;
- read/write menjadi berat.


### Perbaikan

Gunakan relationship terpisah untuk data unbounded dan buat model read-optimized jika diperlukan.

---

## 36.3 Anti-Pattern 3 — Mengabaikan Write Pattern

Contoh:

```text
100.000 write/detik
        ↓
satu partition
        ↓
Hot partition
        ↓
Bottleneck
        ↓
Latency meningkat
```


### Perbaikan

Evaluasi:

- distribution;
- partition key;
- time bucket;
- write batching;
- data ingestion architecture.

---

## 36.4 Anti-Pattern 4 — Mengabaikan Data Growth

Model bekerja pada:

```text
1 juta record
```

tetapi gagal pada:

```text
1 miliar record
```

### Perbaikan

Lakukan capacity planning:

```text
Current
+
Expected Growth
+
Peak
+
Retention
+
Replication
```

---

## 36.5 Anti-Pattern 5 — Denormalization Tanpa Strategi Update

Data yang sama disalin:

```text
Collection A
Collection B
Collection C
Collection D
```

Jika data berubah, harus tersedia mekanisme sinkronisasi yang jelas.

### Perbaikan

Tentukan:

- source of truth;
- data yang bersifat snapshot;
- data yang harus sinkron;
- update propagation;
- toleransi stale data.

---

# 37. Bagian T — Trade-Off Data Modeling

Tidak ada satu model NoSQL yang selalu paling benar.

Desain adalah trade-off.

```text
Read Performance
      ↕
Write Performance

Consistency
      ↕
Availability / Latency

Duplication
      ↕
Query Simplicity

Storage
      ↕
Performance

Flexibility
      ↕
Governance
```

## 37.1 Read vs Write

Denormalisasi dapat membuat read sederhana:

```text
1 query
```

tetapi update dapat menjadi:

```text
1 perubahan
→ update beberapa representation
```

## 37.2 Consistency vs Availability/Latency

Kebutuhan consistency tinggi dapat membutuhkan koordinasi lebih besar.

Pada workload tertentu, sistem mungkin memilih mekanisme yang mengutamakan availability dan menerima keterlambatan sementara.

Keputusan harus didasarkan pada kebutuhan bisnis.

## 37.3 Duplication vs Query Simplicity

```text
Data duplicate
      ↓
lebih banyak storage
      ↓
read lebih sederhana
```

Sebaliknya:

```text
Data normalized
      ↓
lebih sedikit duplication
      ↓
query dapat membutuhkan lookup tambahan
```

## 37.4 Prinsip Evaluasi

Mahasiswa harus mampu menjawab:

> **"Mengapa struktur ini dipilih untuk query tersebut?"**

bukan sekadar:

> **"Saya memilih MongoDB/Cassandra karena NoSQL."**

---



# 49. Checklist Perancangan NoSQL

Sebelum menyatakan model selesai, gunakan checklist berikut.

## Requirement

- [ ] Business requirement sudah jelas.
- [ ] Aktor dan fungsi utama sudah diidentifikasi.
- [ ] Data yang dibutuhkan sudah diidentifikasi.

## Access Pattern

- [ ] Read pattern sudah diidentifikasi.
- [ ] Write pattern sudah diidentifikasi.
- [ ] Update pattern sudah diidentifikasi.
- [ ] Delete/retention pattern sudah dipertimbangkan.

## Query

- [ ] Query utama sudah ditulis.
- [ ] Parameter query sudah jelas.
- [ ] Frekuensi query diketahui.
- [ ] Query kritis sudah diprioritaskan.

## Cardinality

- [ ] One-to-one diidentifikasi.
- [ ] One-to-many diidentifikasi.
- [ ] Many-to-many diidentifikasi.
- [ ] Bounded/unbounded relationship diketahui.

## Volume & Velocity

- [ ] Record/day dihitung.
- [ ] Record/month dihitung.
- [ ] Record/year dihitung.
- [ ] Average rate dihitung.
- [ ] Peak rate dipertimbangkan.
- [ ] Data growth dipertimbangkan.

## Consistency

- [ ] Data yang membutuhkan consistency tinggi diidentifikasi.
- [ ] Data yang dapat mentoleransi eventual consistency diidentifikasi.
- [ ] Atomicity/transaksi dipertimbangkan.

## Availability

- [ ] Failure scenario diidentifikasi.
- [ ] Replication dipertimbangkan.
- [ ] Failover dipertimbangkan.
- [ ] Backup/recovery dipertimbangkan.

## Scalability

- [ ] Vertical scaling dipertimbangkan.
- [ ] Horizontal scaling dipertimbangkan.
- [ ] Partitioning/sharding dipertimbangkan.
- [ ] Growth scenario diuji.

## Data Structure

- [ ] Embedding decision memiliki alasan.
- [ ] Referencing decision memiliki alasan.
- [ ] Denormalization memiliki alasan.
- [ ] Partition key memiliki alasan.
- [ ] Index memiliki alasan.

## Validation

- [ ] Semua query utama dapat dilayani.
- [ ] Tidak ada unbounded document yang tidak terkendali.
- [ ] Tidak ada hot partition yang jelas.
- [ ] Update consistency sudah dipertimbangkan.
- [ ] Trade-off telah didokumentasikan.

---

