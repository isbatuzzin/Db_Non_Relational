# OUTLINE DETAIL PERTEMUAN 2

# Database Classification dan NoSQL Paradigm

## Mata Kuliah

Database Non-Relational

## Program Studi

Sains Data

## Bobot

3 SKS / 3 jam pelajaran

## Pertemuan

Pertemuan ke-2

------------------------------------------------------------------------

# 1. Tujuan Pembelajaran

Setelah mengikuti pertemuan ini, mahasiswa mampu:

1.  Menjelaskan alasan database diklasifikasikan berdasarkan model data
    dan workload.
2.  Menjelaskan karakteristik utama paradigma database non-relational:
    -   Key-Value
    -   Document
    -   Wide-Column
    -   Graph
    -   Search/Analytics
    -   Time-Series
    -   Vector/AI-oriented database
3.  Membandingkan karakteristik, kekuatan, keterbatasan, use case, dan
    anti-use case dari setiap paradigma.
4.  Menghubungkan karakteristik data dengan kebutuhan workload.
5.  Menjelaskan secara konseptual hubungan database selection dengan
    access pattern dan query pattern.
6.  Mengenali CAP theorem sebagai konsep dasar distributed database.
7.  Mengidentifikasi teknologi yang sesuai untuk suatu skenario data.
8.  Membuat decision matrix untuk memilih paradigma/database yang sesuai
    berdasarkan kebutuhan.

------------------------------------------------------------------------

# 2. Keterkaitan dengan Dokumen Acuan

Dokumen acuan menetapkan bahwa mahasiswa perlu mampu mengklasifikasikan
database berdasarkan model:

-   Key-Value
-   Document
-   Wide-Column
-   Graph
-   Search/Analytics
-   Time-Series
-   Vector/AI-oriented database

Dokumen acuan juga menekankan bahwa mahasiswa tidak diarahkan pada
pemahaman bahwa "NoSQL lebih baik daripada SQL", tetapi bahwa:

> Database harus dipilih berdasarkan karakteristik data dan workload.

Dokumen acuan memberikan contoh teknologi:

  Paradigma / Kebutuhan      Teknologi
  -------------------------- -----------------------
  Key-Value                  Redis
  Document                   MongoDB
  Wide-Column                Apache Cassandra
  Graph                      Neo4j
  Search/Analytics           OpenSearch
  Cloud Key-Value/Document   Amazon DynamoDB
  Cloud Wide-Column          Google Cloud Bigtable

Selain itu, dokumen acuan memasukkan Time-Series serta
Vector/AI-oriented database sebagai klasifikasi yang perlu dikenalkan,
terutama dalam konteks kebutuhan data modern dan Data Science.

------------------------------------------------------------------------

# 3. Posisi Pertemuan 2 dalam Alur Mata Kuliah

Pertemuan pertama membahas evolusi database, RDBMS, SQL, dan alasan
munculnya NoSQL.

Pertemuan kedua berfungsi sebagai **peta besar dunia database
non-relational** sebelum mahasiswa mempelajari masing-masing teknologi
secara lebih mendalam.

Alur berpikir:

**Data Characteristics**

↓

**Workload**

↓

**Database Model**

↓

**Database Technology**

↓

**Use Case**

↓

**Implementation**

Pertemuan ini belum bertujuan membuat mahasiswa menjadi ahli MongoDB,
Redis, Cassandra, Neo4j, atau OpenSearch.

Tujuan utamanya adalah membangun kemampuan:

> **"Jika saya memiliki suatu masalah data, database model apa yang
> paling tepat dan mengapa?"**

------------------------------------------------------------------------

# 4. Alokasi Waktu 3 Jam

Karena mata kuliah berbobot 3 SKS dan tiga jam digunakan untuk teori
sekaligus praktikum, pembagian waktu yang disarankan:

  Sesi        Kegiatan                                             Durasi
  ----------- ------------------------------------------- ---------------
  1           Apersepsi dan review Pertemuan 1                   10 menit
  2           Konsep database classification                     15 menit
  3           Key-Value                                          15 menit
  4           Document                                           15 menit
  5           Wide-Column                                        15 menit
  6           Graph                                              15 menit
  7           Search/Analytics                                   15 menit
  8           Time-Series                                        10 menit
  9           Vector/AI-oriented database                        10 menit
  10          Istirahat/transisi singkat                          5 menit
  11          Perbandingan model dan use case                    15 menit
  12          CAP theorem sebagai pengantar                      10 menit
  13          Demonstrasi teknologi                              15 menit
  14          Praktikum klasifikasi dan decision matrix          20 menit
  15          Percobaan/eksperimen sederhana                     15 menit
  16          Diskusi hasil, refleksi, dan penutup               10 menit
  **Total**                                                 **180 menit**

------------------------------------------------------------------------

# 5. Bagian A --- Apersepsi dan Review

## Durasi

10 menit

## Tujuan

Menghubungkan materi Pertemuan 1 dengan materi Pertemuan 2.

## Pertanyaan Pemantik

Dosen dapat memulai dengan pertanyaan:

1.  Apakah semua data harus disimpan dalam tabel?
2.  Apakah semua aplikasi membutuhkan database yang sama?
3.  Jika aplikasi memiliki jutaan request per detik, apakah database
    transaksi biasa selalu menjadi pilihan terbaik?
4.  Jika data utama berbentuk JSON, apakah model tabel tetap menjadi
    pilihan pertama?
5.  Jika masalah utama adalah mencari hubungan antar-entitas, apakah
    database relational selalu paling efektif?
6.  Jika kebutuhan utama adalah pencarian teks pada jutaan dokumen,
    apakah database transactional merupakan pilihan utama?

## Pesan Kunci

> Tidak ada satu database yang optimal untuk semua workload.

------------------------------------------------------------------------

# 6. Bagian B --- Konsep Database Classification

## Durasi

15 menit

## Materi

### 6.1 Mengapa Database Perlu Diklasifikasikan?

Database dapat memiliki model data dan karakteristik workload yang
berbeda.

Klasifikasi membantu engineer menjawab:

-   Data disimpan dalam bentuk apa?
-   Bagaimana data diakses?
-   Seberapa cepat data harus diakses?
-   Seberapa besar data?
-   Seberapa cepat data masuk?
-   Bagaimana hubungan antar-data?
-   Apakah workload dominan read atau write?
-   Apakah membutuhkan full-text search?
-   Apakah membutuhkan traversal relationship?
-   Apakah membutuhkan distributed scalability?

### 6.2 Konsep Workload

Perkenalkan workload sebagai pola penggunaan database.

Contoh:

**Workload A**

-   banyak read
-   response harus sangat cepat
-   data dapat berubah relatif sering

→ kandidat: Key-Value/Redis

**Workload B**

-   data semi-structured
-   banyak JSON
-   struktur dokumen dapat bervariasi

→ kandidat: Document/MongoDB

**Workload C**

-   massive write
-   distributed
-   data terpartisi berdasarkan key

→ kandidat: Wide-Column/Cassandra

**Workload D**

-   relationship sangat kompleks
-   membutuhkan traversal

→ kandidat: Graph/Neo4j

**Workload E**

-   pencarian teks
-   log
-   search analytics

→ kandidat: Search/Analytics/OpenSearch

------------------------------------------------------------------------

# 7. Bagian C --- Key-Value Database

## Durasi

15 menit

## Konsep

Data direpresentasikan secara sederhana sebagai:

**Key → Value**

Contoh:

``` text
user:1001 → "Isbat"
session:ABC123 → "{...}"
product:1001 → "{...}"
```

## Karakteristik

-   akses berdasarkan key
-   sederhana
-   cepat
-   cocok untuk low-latency access
-   dapat digunakan untuk caching
-   session
-   counter
-   leaderboard
-   real-time state

## Teknologi

**Redis**

Perkenalkan secara konseptual beberapa data structure Redis:

-   String
-   Hash
-   List
-   Set
-   Sorted Set
-   Streams
-   Geospatial

## Use Case

-   cache
-   session management
-   real-time counter
-   leaderboard
-   real-time location

## Anti-Use Case

Diskusikan mengapa key-value kurang ideal jika kebutuhan utama adalah:

-   query kompleks berdasarkan banyak atribut
-   relationship traversal
-   full-text search kompleks

## Pertanyaan Kelas

> Jika kita hanya mengetahui `user_id` dan membutuhkan profil user
> dengan latency sangat rendah, model apa yang cocok?

------------------------------------------------------------------------

# 8. Bagian D --- Document Database

## Durasi

15 menit

## Konsep

Data disimpan sebagai document, biasanya dalam bentuk JSON/BSON atau
struktur yang menyerupainya.

Contoh:

``` json
{
  "customer_id": 101,
  "name": "Andi",
  "email": "andi@example.com",
  "address": {
    "city": "Surabaya",
    "country": "Indonesia"
  },
  "preferences": [
    "technology",
    "data science"
  ]
}
```

## Karakteristik

-   semi-structured
-   schema lebih fleksibel
-   nested data
-   array
-   cocok untuk application object
-   query berdasarkan field
-   aggregation

## Teknologi

**MongoDB**

## Use Case

-   customer profile
-   content management
-   catalog
-   application data
-   event/document data

## Anti-Use Case

Diskusikan kasus ketika:

-   transaksi sangat kompleks membutuhkan relational integrity
-   relationship sangat banyak dan traversal merupakan kebutuhan utama

## Pertanyaan

> Mengapa profil pelanggan berbentuk JSON lebih natural dimodelkan
> sebagai document dibandingkan memecah seluruh struktur menjadi banyak
> tabel?

------------------------------------------------------------------------

# 9. Bagian E --- Wide-Column Database

## Durasi

15 menit

## Konsep

Wide-column database dirancang untuk workload distributed dan data
berskala besar.

Teknologi:

**Apache Cassandra**

## Konsep Utama

-   cluster
-   node
-   partition
-   partition key
-   clustering key
-   replication
-   distributed storage
-   query-driven data modeling

## Karakteristik

-   high write throughput
-   horizontal scalability
-   distributed
-   availability
-   cocok untuk workload dengan access pattern yang jelas

## Contoh

Sensor:

``` text
sensor_id
timestamp
temperature
humidity
```

Query utama:

``` text
Ambil data sensor tertentu
pada rentang waktu tertentu
```

Maka desain partition perlu mengikuti access pattern tersebut.

## Use Case

-   sensor/IoT
-   event data
-   massive write
-   distributed operational workload

## Anti-Use Case

Kurang tepat jika kebutuhan utamanya:

-   ad-hoc query kompleks
-   relationship traversal
-   relational transaction kompleks

------------------------------------------------------------------------

# 10. Bagian F --- Graph Database

## Durasi

15 menit

## Konsep

Graph database memodelkan:

**Node + Relationship + Property**

Contoh:

``` text
Customer
   |
   | purchased
   ↓
Product
```

atau:

``` text
User
 |
 | follows
 ↓
User
```

## Teknologi

**Neo4j**

## Konsep Query

Perkenalkan secara singkat:

-   node
-   relationship
-   property
-   pattern
-   traversal
-   Cypher

## Use Case

-   social network
-   recommendation
-   fraud detection
-   knowledge graph
-   relationship analysis

## Anti-Use Case

Jika kebutuhan hanya:

-   lookup berdasarkan primary key
-   simple key-value access

maka graph database dapat menjadi overkill.

## Pertanyaan

> Jika kita ingin menemukan "teman dari teman" atau jalur hubungan
> antar-entitas, model database apa yang secara natural
> merepresentasikan masalah tersebut?

------------------------------------------------------------------------

# 11. Bagian G --- Search/Analytics Database

## Durasi

15 menit

## Teknologi

**OpenSearch**

## Konsep

Search/analytics engine digunakan ketika kebutuhan utama berkaitan
dengan:

-   full-text search
-   log analytics
-   search
-   filtering
-   aggregation
-   semantic search
-   hybrid search
-   vector search

## Contoh

Data:

``` text
Application Log
2026-09-03 10:01 ERROR payment timeout
2026-09-03 10:02 INFO payment success
2026-09-03 10:03 ERROR database timeout
```

Pertanyaan:

> "Cari semua error yang mengandung kata timeout."

Ini merupakan workload yang berbeda dari transaksi relational biasa.

## Use Case

-   application log analytics
-   observability
-   search engine
-   document search
-   semantic/hybrid search

------------------------------------------------------------------------

# 12. Bagian H --- Time-Series Database

## Durasi

10 menit

## Konsep

Time-series data memiliki dimensi waktu sebagai elemen penting.

Contoh:

``` text
timestamp
sensor_id
temperature
humidity
```

Karakteristik:

-   data datang terus-menerus
-   timestamp penting
-   write volume tinggi
-   query berdasarkan time range
-   aggregation berdasarkan waktu

## Contoh Workload

-   data sensor
-   monitoring server
-   financial time series
-   telemetry
-   application metrics

## Diskusi

Bandingkan:

**Data transaksi**

vs

**Data sensor setiap 1 detik**

Mahasiswa diminta menjelaskan mengapa workload keduanya berbeda.

------------------------------------------------------------------------

# 13. Bagian I --- Vector/AI-Oriented Database

## Durasi

10 menit

## Konsep

Database modern juga dapat digunakan untuk menyimpan dan mencari
vector/embedding.

Contoh:

``` text
Dokumen
   ↓
Embedding Model
   ↓
Vector
   ↓
Vector Search
```

## Use Case

-   semantic search
-   recommendation
-   RAG
-   similarity search
-   AI knowledge retrieval

## Hubungan dengan Database Lain

Vector dapat disimpan bersama:

-   metadata
-   document
-   graph relationship

Sehingga muncul arsitektur seperti:

**Vector + Metadata + Graph**

## Pertanyaan

> Mengapa pencarian "dokumen yang maknanya paling mirip" berbeda dengan
> pencarian menggunakan exact keyword?

------------------------------------------------------------------------

# 14. Bagian J --- Perbandingan Paradigma

## Durasi

15 menit

Gunakan tabel berikut sebagai bahan diskusi.

  -------------------------------------------------------------------------------------------------
  Model              Cara Akses Utama            Kekuatan       Contoh Use Case      Teknologi
  ------------------ --------------------------- -------------- -------------------- --------------
  Key-Value          key lookup                  low latency    cache/session        Redis

  Document           document/field query        flexible       customer profile     MongoDB
                                                 schema                              

  Wide-Column        partition/query pattern     distributed    sensor/event         Cassandra
                                                 scale                               

  Graph              traversal                   relationship   recommendation       Neo4j

  Search/Analytics   search/filter/aggregation   text &         logs/search          OpenSearch
                                                 analytics                           

  Time-Series        time range                  temporal data  sensor/metrics       Time-series DB

  Vector/AI          similarity search           semantic       RAG/recommendation   Vector DB
                                                 retrieval                           
  -------------------------------------------------------------------------------------------------

## Prinsip Penting

Mahasiswa harus memahami bahwa tabel tersebut bukan aturan mutlak.

Pemilihan database tetap bergantung pada:

-   requirement
-   data characteristics
-   workload
-   access pattern
-   query pattern
-   volume
-   velocity
-   consistency
-   availability
-   scalability

------------------------------------------------------------------------

# 15. Bagian K --- Use Case dan Anti-Use Case

## Tujuan

Mahasiswa tidak hanya mengetahui kapan suatu database digunakan, tetapi
juga kapan database tersebut **tidak cocok**.

## Latihan Cepat

Dosen menyebutkan skenario, mahasiswa menjawab:

**Cocok / Tidak Cocok / Perlu Analisis Lebih Lanjut**

### Skenario 1

Menyimpan session login berdasarkan session ID.

Kandidat: **Redis**

### Skenario 2

Menyimpan profil pelanggan dengan struktur JSON yang dapat berkembang.

Kandidat: **MongoDB**

### Skenario 3

Menyimpan jutaan event sensor setiap hari dengan distributed write.

Kandidat: **Cassandra**

### Skenario 4

Mencari hubungan pelanggan yang melakukan transaksi dengan pelanggan
lain.

Kandidat: **Neo4j**

### Skenario 5

Mencari kata "timeout" dari jutaan application log.

Kandidat: **OpenSearch**

### Skenario 6

Mencari dokumen yang memiliki makna paling mirip dengan pertanyaan
pengguna.

Kandidat: **Vector/AI-oriented database**

------------------------------------------------------------------------

# 16. Bagian L --- CAP Theorem sebagai Pengantar

## Durasi

10 menit

## Tujuan

CAP theorem hanya diperkenalkan sebagai dasar pemahaman distributed
database.

Belum perlu masuk terlalu dalam karena pembahasan distributed database
dilakukan pada pertemuan berikutnya.

## Materi

CAP:

-   Consistency
-   Availability
-   Partition Tolerance

### Pertanyaan Pemantik

Bayangkan database memiliki:

``` text
Node A
   |
   X   Network Partition
   |
Node B
```

Apa yang terjadi jika client mengakses Node A dan Node B?

Diskusikan:

-   apakah kedua node harus selalu memberikan data yang sama?
-   apakah sistem harus tetap melayani request?
-   apa trade-off ketika network partition terjadi?

## Pesan Utama

> Pada distributed system terdapat trade-off. Database selection tidak
> hanya berbicara mengenai struktur data, tetapi juga consistency,
> availability, dan scalability.

Pembahasan detail CAP, replication, partitioning, sharding, dan eventual
consistency dilakukan pada pertemuan 9.

------------------------------------------------------------------------

# 17. Bagian M --- Demonstrasi Teknologi

## Durasi

15 menit

Demonstrasi tidak perlu mendalam.

Tujuannya memberikan **visual grounding** bahwa paradigma database
benar-benar berbeda.

## Demo 1 --- Redis

Contoh:

``` text
SET user:101 "Andi"
GET user:101
```

Tekankan:

**Key → Value**

## Demo 2 --- MongoDB

Contoh:

``` json
{
  "id": 101,
  "name": "Andi",
  "skills": ["Python", "SQL"]
}
```

Tekankan:

**Document**

## Demo 3 --- Cassandra

Tunjukkan konsep:

``` text
Partition Key
+
Clustering Key
```

Tekankan:

**Query-driven distributed data model**

## Demo 4 --- Neo4j

Tunjukkan:

``` text
(A)-[:PURCHASED]->(B)
```

Tekankan:

**Relationship**

## Demo 5 --- OpenSearch

Tunjukkan konsep:

``` text
search("database")
```

Tekankan:

**Search/Analytics**

------------------------------------------------------------------------

# 18. Bagian N --- Praktikum: Database Classification Exercise

## Durasi

20 menit

## Tujuan Praktikum

Mahasiswa berlatih memilih database berdasarkan karakteristik data dan
workload.

## Bentuk Praktikum

Mahasiswa bekerja dalam kelompok 2--3 orang.

Setiap kelompok mendapatkan 10 skenario.

------------------------------------------------------------------------

## Skenario 1 --- User Session

Sistem membutuhkan penyimpanan:

``` text
session_id
user_id
login_time
expiration
```

Kebutuhan:

-   sangat cepat
-   lookup berdasarkan session_id
-   memiliki expiration

### Pertanyaan

1.  Database model apa?
2.  Teknologi apa?
3.  Mengapa?

------------------------------------------------------------------------

## Skenario 2 --- Customer Profile

Data:

``` text
customer
address
preferences
social_media
purchase_preferences
```

Struktur dapat berubah.

### Pertanyaan

1.  Database model apa?
2.  Mengapa document database sesuai?

------------------------------------------------------------------------

## Skenario 3 --- IoT Sensor

Setiap sensor mengirim data setiap detik.

Data:

``` text
sensor_id
timestamp
temperature
humidity
```

Jumlah sensor: 100.000.

### Pertanyaan

1.  Apa workload utama?
2.  Apakah write atau read lebih dominan?
3.  Database model apa yang cocok?
4.  Mengapa Cassandra dapat menjadi kandidat?

------------------------------------------------------------------------

## Skenario 4 --- Social Network

Data:

``` text
User A follows User B
User B follows User C
User C follows User D
```

### Pertanyaan

Bagaimana mencari:

> "Siapa teman dari teman User A?"

Database model apa yang natural?

------------------------------------------------------------------------

## Skenario 5 --- Application Logs

Jutaan log:

``` text
INFO
WARNING
ERROR
DATABASE TIMEOUT
AUTHENTICATION FAILED
```

### Pertanyaan

Database apa yang cocok untuk pencarian dan analytics?

------------------------------------------------------------------------

## Skenario 6 --- Recommendation

Sistem ingin mencari:

> "Produk yang mirip dengan produk yang sedang dilihat."

### Pertanyaan

Apakah relational database saja selalu merupakan pilihan paling natural?

Apa peran vector database?

------------------------------------------------------------------------

## Skenario 7 --- Real-Time Leaderboard

Data:

``` text
player
score
rank
```

Skor berubah sangat sering.

### Pertanyaan

Database model apa yang sesuai?

------------------------------------------------------------------------

## Skenario 8 --- Knowledge Graph

Data:

``` text
Person
Organization
Project
Technology
```

Dengan banyak relationship.

### Pertanyaan

Model apa yang paling natural?

------------------------------------------------------------------------

## Skenario 9 --- Full-Text Search

Pengguna mengetik:

> "database architecture for big data"

Sistem harus menemukan dokumen yang relevan.

### Pertanyaan

Apa perbedaan workload ini dengan:

``` sql
SELECT * FROM documents
WHERE title = ...
```

------------------------------------------------------------------------

## Skenario 10 --- Cloud Massive Scale

Sistem membutuhkan database managed cloud dengan:

-   scalability
-   high availability
-   key-value/document model

### Kandidat

**Amazon DynamoDB**

Diskusikan alasan memilih managed database dibanding mengelola database
sendiri.

------------------------------------------------------------------------

# 19. Format Jawaban Praktikum

Mahasiswa mengisi:

  -----------------------------------------------------------------------------------------
            No Skenario         Data             Workload   Model     Kandidat    Alasan
                                Characteristic                        Teknologi   
  ------------ ---------------- ---------------- ---------- --------- ----------- ---------
             1 Session                                                            

             2 Customer                                                           

             3 IoT                                                                

             4 Social Network                                                     

             5 Log                                                                

             6 Recommendation                                                     

             7 Leaderboard                                                        

             8 Knowledge Graph                                                    

             9 Search                                                             

            10 Cloud Scale                                                        
  -----------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 20. Percobaan 1 --- "Satu Data, Banyak Model"

## Tujuan

Membuktikan kepada mahasiswa bahwa satu domain data dapat dimodelkan
dengan paradigma database yang berbeda, tetapi masing-masing optimal
untuk workload tertentu.

## Kasus

Gunakan data:

``` text
Customer
Product
Order
```

Contoh:

``` text
Customer 101 membeli Product A
Customer 101 membeli Product B
Customer 102 membeli Product A
```

## Percobaan

Mahasiswa diminta membayangkan data yang sama dimodelkan sebagai:

### Model 1 --- Document

``` text
Customer
  └── Orders
       └── Products
```

### Model 2 --- Graph

``` text
(Customer)-[:PURCHASED]->(Product)
```

### Model 3 --- Key-Value

``` text
customer:101 → {...}
```

## Pertanyaan

1.  Model mana yang paling mudah untuk mengambil profil customer?
2.  Model mana yang paling natural untuk mencari relationship?
3.  Model mana yang paling cepat untuk key lookup?
4.  Apakah satu model otomatis lebih baik untuk semua kebutuhan?

## Kesimpulan Eksperimen

Mahasiswa diarahkan menemukan sendiri bahwa:

> **Database model harus disesuaikan dengan workload.**

------------------------------------------------------------------------

# 21. Percobaan 2 --- Query Pattern Determines Database Model

## Tujuan

Memahami bahwa database selection dimulai dari pertanyaan yang ingin
dijawab oleh sistem.

## Langkah

Berikan data customer-product.

Kemudian berikan query:

### Query A

> Ambil profil customer berdasarkan customer_id.

### Query B

> Cari semua produk yang dibeli customer.

### Query C

> Cari pelanggan yang membeli produk yang sama.

### Query D

> Cari pelanggan yang memiliki hubungan sampai tiga level.

### Query E

> Cari dokumen berdasarkan kemiripan makna.

## Diskusi

Mahasiswa menghubungkan query dengan model:

  Query                       Model Kandidat
  --------------------------- ----------------
  Key lookup                  Key-Value
  Document retrieval          Document
  Massive distributed write   Wide-Column
  Relationship traversal      Graph
  Text search                 Search
  Time-range analytics        Time-Series
  Similarity search           Vector

## Pesan Utama

> **Jangan memilih database terlebih dahulu. Tentukan workload dan query
> pattern terlebih dahulu.**

------------------------------------------------------------------------

# 22. Percobaan 3 --- Workload Classification

## Tujuan

Mahasiswa memahami perbedaan:

-   read-heavy
-   write-heavy
-   low-latency
-   high-throughput
-   relationship-heavy
-   search-heavy
-   time-oriented
-   similarity-oriented

## Instruksi

Untuk setiap workload, mahasiswa memberikan ranking 1--3 database model
yang paling sesuai.

Contoh:

### Workload

> 1 juta read per detik berdasarkan ID.

Jawaban yang diharapkan:

**Key-Value → kandidat kuat**

### Workload

> Pencarian relationship kompleks.

**Graph → kandidat kuat**

### Workload

> Pencarian full-text.

**Search/Analytics → kandidat kuat**

### Workload

> Massive distributed write.

**Wide-Column → kandidat kuat**

------------------------------------------------------------------------

# 23. Praktikum Utama --- Membuat Decision Matrix

## Tujuan

Menghasilkan output utama pertemuan:

> **Decision Matrix Pemilihan Paradigma NoSQL**

## Kriteria

Mahasiswa memberikan skor terhadap:

1.  Key-based access
2.  Flexible schema
3.  Massive write
4.  Horizontal scalability
5.  Relationship traversal
6.  Full-text search
7.  Time-series workload
8.  Similarity/vector search
9.  Low latency
10. Distributed availability

## Contoh Format

  -----------------------------------------------------------------------------------------------
  Kriteria         Key-Value   Document   Wide-Column     Graph    Search   Time-Series    Vector
  -------------- ----------- ---------- ------------- --------- --------- ------------- ---------
  Key Lookup               5          4             4         2         2             2         2

  Flexible                 2          5             3         4         4             3         4
  Schema                                                                                

  Massive Write            4          3             5         2         4             5         3

  Relationship             1          3             1         5         2             1         3

  Full-text                1          3             1         2         5             1         4
  Search                                                                                

  Time-series              2          3             5         2         4             5         3

  Similarity               1          2             1         3         4             1         5
  Search                                                                                
  -----------------------------------------------------------------------------------------------

**Catatan:** Nilai di atas hanya contoh untuk latihan. Mahasiswa harus
memberikan justifikasi terhadap skor yang mereka berikan.

------------------------------------------------------------------------

# 24. Aktivitas Diskusi Kritis

## Pertanyaan 1

> Apakah NoSQL menggantikan SQL?

Jawaban yang diharapkan:

Tidak.

Database dipilih berdasarkan kebutuhan dan workload.

------------------------------------------------------------------------

## Pertanyaan 2

> Mengapa satu perusahaan dapat menggunakan MongoDB + Redis +
> Cassandra + Neo4j sekaligus?

Karena setiap database dapat menangani workload yang berbeda.

Konsep ini mengarah pada:

**Polyglot Persistence**

------------------------------------------------------------------------

## Pertanyaan 3

> Apakah semakin banyak jenis database berarti semakin baik?

Tidak.

Setiap database menambah:

-   operational complexity
-   skill requirement
-   monitoring
-   backup
-   security
-   maintenance
-   integration complexity

Maka pemilihan database harus mempertimbangkan trade-off.

------------------------------------------------------------------------

## Pertanyaan 4

> Mengapa Data Engineer harus memahami database model dan bukan hanya
> syntax query?

Karena Data Engineer harus mampu menentukan:

-   bagaimana data disimpan
-   bagaimana data diakses
-   bagaimana pipeline mengalir
-   bagaimana workload ditangani
-   bagaimana sistem diskalakan

------------------------------------------------------------------------

# 25. Studi Kasus Integratif

## Kasus

Sebuah platform e-commerce memiliki:

### Customer

-   profile
-   address
-   preference

### Transaction

-   order
-   payment
-   product

### Real-Time

-   session
-   cart
-   leaderboard/promotion counter

### Event

-   click
-   view
-   purchase

### Search

-   product search
-   log search

### Recommendation

-   product similarity
-   customer-product relationship

## Tugas Mahasiswa

Tentukan kandidat database untuk masing-masing kebutuhan.

Contoh format:

  Kebutuhan                   Kandidat       Alasan
  --------------------------- -------------- --------------------------
  Customer Profile            MongoDB        Document/semi-structured
  Session/Cache               Redis          Low-latency key-value
  Massive Event               Cassandra      Distributed/high write
  Relationship                Neo4j          Graph traversal
  Search/Log                  OpenSearch     Search/analytics
  Recommendation similarity   Vector/AI DB   Similarity search

Kemudian mahasiswa harus menjelaskan:

> Apakah semua data tersebut harus disimpan dalam satu database?

------------------------------------------------------------------------

# 26. Evaluasi Formatif

## Bentuk

Dosen memberikan 5 pertanyaan singkat pada akhir pertemuan.

### Pertanyaan 1

Sistem membutuhkan lookup berdasarkan key dengan latency sangat rendah.

**Database model?**

### Pertanyaan 2

Data pelanggan berbentuk JSON dengan nested structure.

**Database model?**

### Pertanyaan 3

Data sensor masuk dengan volume sangat besar dan membutuhkan horizontal
scalability.

**Database model?**

### Pertanyaan 4

Sistem membutuhkan pencarian relationship yang kompleks.

**Database model?**

### Pertanyaan 5

Sistem membutuhkan full-text search dan log analytics.

**Database model?**

------------------------------------------------------------------------

# 27. Exit Ticket

Sebelum kelas selesai, setiap mahasiswa menuliskan:

1.  Satu hal baru yang dipahami.
2.  Satu paradigma database yang paling menarik.
3.  Satu workload yang cocok untuk paradigma tersebut.
4.  Satu alasan mengapa database tidak boleh dipilih hanya berdasarkan
    popularitas teknologi.

------------------------------------------------------------------------

# 28. Tugas Setelah Pertemuan

## Tugas Individu

Pilih satu sistem:

-   e-commerce
-   social media
-   IoT
-   recommendation
-   fintech
-   healthcare
-   education
-   logistics
-   streaming
-   AI/RAG

Kemudian identifikasi:

1.  Minimal 5 jenis data.
2.  Karakteristik setiap data.
3.  Workload.
4.  Access pattern.
5.  Query pattern.
6.  Database model kandidat.
7.  Teknologi kandidat.
8.  Alasan pemilihan.
9.  Risiko jika menggunakan database yang tidak sesuai.

## Format Output

Maksimal 2--3 halaman.

------------------------------------------------------------------------

# 29. Rubrik Praktikum

  Aspek                                  Bobot
  --------------------------------- ----------
  Identifikasi karakteristik data          20%
  Identifikasi workload                    20%
  Pemilihan paradigma                      25%
  Justifikasi teknologi                    20%
  Decision matrix                          10%
  Diskusi/presentasi                        5%
  **Total**                           **100%**

------------------------------------------------------------------------

# 30. Indikator Keberhasilan Pertemuan

Mahasiswa dianggap mencapai tujuan pembelajaran apabila mampu:

### Level 1 --- Understand

Menjelaskan minimal tujuh paradigma database.

### Level 2 --- Classify

Mengklasifikasikan suatu workload ke database model yang sesuai.

### Level 3 --- Analyze

Menjelaskan alasan pemilihan berdasarkan karakteristik data dan
workload.

### Level 4 --- Evaluate

Membandingkan dua atau lebih database model dan menjelaskan trade-off.

### Level 5 --- Decide

Membuat decision matrix dan mengambil keputusan database selection.

------------------------------------------------------------------------

# 31. Konsep yang Harus Ditekankan Dosen

Ada lima pesan utama yang harus terus diulang selama pertemuan:

### Pesan 1

> **NoSQL bukan berarti database tanpa struktur.**

### Pesan 2

> **Tidak ada satu database yang terbaik untuk semua kebutuhan.**

### Pesan 3

> **Workload menentukan database model.**

### Pesan 4

> **Query/access pattern harus dipahami sebelum melakukan data
> modeling.**

### Pesan 5

> **Database selection adalah architectural decision, bukan sekadar
> keputusan teknis memilih produk.**

------------------------------------------------------------------------

# 32. Hubungan dengan Kompetensi Data Engineer

Pertemuan ini menjadi dasar kompetensi Data Engineer dalam:

-   memahami berbagai jenis data store
-   memahami data characteristics
-   memahami workload
-   memilih data storage technology
-   memahami distributed data store
-   memahami scalability
-   memahami availability
-   memahami performance
-   memahami trade-off
-   memahami polyglot persistence

Alur kompetensi:

**Requirement**

↓

**Data Characteristics**

↓

**Workload**

↓

**Access Pattern**

↓

**Database Model**

↓

**Technology Selection**

↓

**Data Modeling**

Materi data modeling secara mendalam akan dilanjutkan pada **Pertemuan
3**.

------------------------------------------------------------------------

# 33. Penutup Pertemuan

Dosen menutup dengan pertanyaan utama:

> **"Jika besok Anda diberi sebuah proyek dengan data baru, apakah Anda
> akan langsung memilih MongoDB, Redis, Cassandra, atau Neo4j?"**

Jawaban yang diharapkan:

> **Tidak.**

Mahasiswa harus terlebih dahulu bertanya:

1.  Apa requirement-nya?
2.  Data seperti apa?
3.  Bagaimana workload-nya?
4.  Apa access pattern-nya?
5.  Query apa yang dibutuhkan?
6.  Bagaimana volume dan velocity?
7.  Bagaimana kebutuhan consistency?
8.  Bagaimana availability?
9.  Bagaimana scalability?
10. Baru kemudian memilih database model dan teknologi.

Dengan demikian, mahasiswa mulai membangun pola pikir:

> **"Problem First, Database Later."**

------------------------------------------------------------------------

# 34. Referensi Materi

## Berdasarkan Dokumen Acuan

Dokumen acuan mata kuliah menetapkan:

-   Key-Value
-   Document
-   Wide-Column
-   Graph
-   Search/Analytics
-   Time-Series
-   Vector/AI-oriented database

serta teknologi:

-   Redis
-   MongoDB
-   Apache Cassandra
-   Neo4j
-   OpenSearch
-   Amazon DynamoDB
-   Google Cloud Bigtable

Dokumen acuan juga menekankan database selection berdasarkan
karakteristik data dan workload.

## Referensi Teknologi

-   MongoDB Documentation
-   Redis Documentation
-   Apache Cassandra Documentation
-   Neo4j Documentation
-   OpenSearch Documentation
-   Amazon DynamoDB Documentation
-   Google Cloud Bigtable Documentation

------------------------------------------------------------------------

# 35. Ringkasan Alur Mengajar 180 Menit

``` text
REVIEW PERTEMUAN 1
       ↓
Mengapa database perlu diklasifikasikan?
       ↓
WORKLOAD
       ↓
┌───────────────┬────────────────┐
│               │                │
Key-Value     Document       Wide-Column
Redis         MongoDB        Cassandra
│               │                │
└───────────────┴────────────────┘
       ↓
Graph → Neo4j
       ↓
Search → OpenSearch
       ↓
Time-Series
       ↓
Vector/AI
       ↓
Perbandingan Model
       ↓
CAP Theorem
       ↓
Demonstrasi
       ↓
Praktikum 10 Skenario
       ↓
Percobaan Workload
       ↓
Decision Matrix
       ↓
DISKUSI
       ↓
EXIT TICKET
       ↓
PERTEMUAN 3:
DATA MODELING BERBASIS ACCESS PATTERN
```

------------------------------------------------------------------------

# 36. Hasil Akhir yang Dikumpulkan Mahasiswa

Pada akhir pertemuan mahasiswa menghasilkan:

1.  Tabel klasifikasi database.
2.  Analisis 10 skenario workload.
3.  Hasil percobaan "Satu Data, Banyak Model".
4.  Hasil workload classification.
5.  Decision matrix pemilihan paradigma NoSQL.
6.  Kesimpulan database selection untuk satu studi kasus.

Output utama pertemuan:

> **Decision Matrix Pemilihan Paradigma NoSQL berdasarkan Data
> Characteristics dan Workload.**
