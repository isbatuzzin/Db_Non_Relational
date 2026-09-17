# Praktikum MongoDB — Data Modeling NoSQL
## Studi Kasus: Order E-Commerce

### 1. Tujuan Praktikum

Praktikum ini merupakan contoh implementasi sederhana **Data Modeling NoSQL menggunakan MongoDB** berdasarkan pendekatan *query-centric/data access pattern*. Fokus praktikum adalah:

1. Menyiapkan environment MongoDB.
2. Membuat database dan collection.
3. Menyiapkan data order e-commerce.
4. Menggunakan pendekatan **embedding** untuk `OrderItem`.
5. Melakukan akses data berdasarkan *access pattern* dan *query pattern*.
6. Membuat index sesuai kebutuhan query.
7. Menggunakan `explain("executionStats")` untuk melihat penggunaan index dan performa query.
8. Memahami alasan desain dokumen MongoDB yang digunakan.

---

# 2. Studi Kasus

Sebuah sistem marketplace menyimpan transaksi pelanggan. Salah satu kebutuhan utama sistem adalah:

> **Ketika pelanggan membuka halaman detail order, sistem harus dapat mengambil informasi order beserta seluruh item yang dibeli dengan cepat.**

Contoh informasi order:

- `order_id`
- `customer_id`
- tanggal order
- status order
- informasi pengiriman
- daftar produk yang dibeli
- harga produk saat transaksi
- subtotal
- total order
- informasi pembayaran

### Access Pattern Utama

Sistem membutuhkan akses data berikut:

1. Mengambil order berdasarkan `order_id`.
2. Mengambil seluruh order milik customer.
3. Mengambil order customer berdasarkan rentang/tanggal order.
4. Mencari order yang memiliki produk tertentu.

### Query Pattern

| Kebutuhan | Query Pattern |
|---|---|
| Detail order | `order_id` |
| Riwayat order customer | `customer_id` |
| Riwayat order berdasarkan tanggal | `customer_id + order_date` |
| Order yang berisi produk tertentu | `items.product_id` |

---

# 3. Alasan Menggunakan Embedding

Pada model ini, `OrderItem` disimpan sebagai array `items` di dalam dokumen `orders`.

Contoh struktur:

```text
orders
 └── order
      ├── order_id
      ├── customer_id
      ├── order_date
      ├── status
      ├── shipping
      ├── items[]
      │    ├── product_id
      │    ├── product_name
      │    ├── quantity
      │    ├── price
      │    └── subtotal
      ├── total_amount
      └── payment
```

Pendekatan ini sesuai karena detail item order biasanya dibaca bersamaan ketika pengguna membuka detail order.

Selain itu, `price` dan `subtotal` pada item merupakan **snapshot historis transaksi**. Harga produk saat ini dapat berubah, tetapi harga yang tercatat pada order harus tetap merepresentasikan harga ketika transaksi dilakukan.

---

# 4. Persiapan Environment

## 4.1 Software yang Dibutuhkan

Gunakan salah satu lingkungan berikut:

- MongoDB Community Server
- MongoDB Atlas
- `mongosh`
- MongoDB Compass (opsional)

Untuk praktikum ini, contoh akses menggunakan `mongosh`.

## 4.2 Memastikan MongoDB Berjalan

Buka terminal/command prompt:

```bash
mongosh
```

Jika koneksi berhasil, akan muncul informasi koneksi MongoDB.

---

# 5. Membuat Database

Setelah masuk ke `mongosh`, pilih database:

```javascript
use marketplace
```

Contoh hasil:

```text
switched to db marketplace
```

Perintah tersebut membuat/memilih database bernama `marketplace`.

---

# 6. Membuat Collection

Buat collection `orders`:

```javascript
db.createCollection("orders")
```

Contoh hasil:

```text
{ ok: 1 }
```

Untuk melihat collection:

```javascript
show collections
```

Hasil:

```text
orders
```

---

# 7. Menyiapkan Data

Masukkan dua contoh order:

```javascript
db.orders.insertMany([
  {
    order_id: "ORD001",
    customer_id: "CUS001",
    order_date: ISODate("2026-09-16T10:15:00Z"),
    status: "PAID",

    shipping: {
      courier: "JNE",
      service: "REG",
      address: "Surabaya"
    },

    items: [
      {
        product_id: "PROD001",
        product_name: "Laptop ABC",
        quantity: 1,
        price: 12000000,
        subtotal: 12000000
      },
      {
        product_id: "PROD002",
        product_name: "Mouse ABC",
        quantity: 1,
        price: 1200000,
        subtotal: 1200000
      }
    ],

    total_amount: 13200000,

    payment: {
      method: "Bank Transfer",
      status: "PAID"
    }
  },

  {
    order_id: "ORD002",
    customer_id: "CUS001",
    order_date: ISODate("2026-09-17T09:30:00Z"),
    status: "PROCESSING",

    shipping: {
      courier: "J&T",
      service: "REG",
      address: "Surabaya"
    },

    items: [
      {
        product_id: "PROD003",
        product_name: "Keyboard XYZ",
        quantity: 1,
        price: 550000,
        subtotal: 550000
      }
    ],

    total_amount: 550000,

    payment: {
      method: "E-Wallet",
      status: "PAID"
    }
  }
])
```

Contoh hasil:

```text
{
  acknowledged: true,
  insertedIds: {
    "0": ObjectId("..."),
    "1": ObjectId("...")
  }
}
```

> Nilai `ObjectId` dapat berbeda pada setiap komputer karena dibuat otomatis oleh MongoDB.

---

# 8. Melihat Seluruh Data

Gunakan:

```javascript
db.orders.find().pretty()
```

Contoh hasil:

```javascript
{
  _id: ObjectId("..."),
  order_id: "ORD001",
  customer_id: "CUS001",
  order_date: ISODate("2026-09-16T10:15:00Z"),
  status: "PAID",
  shipping: {
    courier: "JNE",
    service: "REG",
    address: "Surabaya"
  },
  items: [
    {
      product_id: "PROD001",
      product_name: "Laptop ABC",
      quantity: 1,
      price: 12000000,
      subtotal: 12000000
    },
    {
      product_id: "PROD002",
      product_name: "Mouse ABC",
      quantity: 1,
      price: 1200000,
      subtotal: 1200000
    }
  ],
  total_amount: 13200000,
  payment: {
    method: "Bank Transfer",
    status: "PAID"
  }
}
```

Dokumen `ORD002` memiliki struktur yang sama, tetapi hanya mempunyai satu item.

---

# 9. Mengakses Order Berdasarkan `order_id`

Ini merupakan query utama untuk halaman **Detail Order**.

```javascript
db.orders.findOne({
  order_id: "ORD001"
})
```

Contoh hasil:

```javascript
{
  _id: ObjectId("..."),
  order_id: "ORD001",
  customer_id: "CUS001",
  order_date: ISODate("2026-09-16T10:15:00Z"),
  status: "PAID",
  shipping: {
    courier: "JNE",
    service: "REG",
    address: "Surabaya"
  },
  items: [
    {
      product_id: "PROD001",
      product_name: "Laptop ABC",
      quantity: 1,
      price: 12000000,
      subtotal: 12000000
    },
    {
      product_id: "PROD002",
      product_name: "Mouse ABC",
      quantity: 1,
      price: 1200000,
      subtotal: 1200000
    }
  ],
  total_amount: 13200000,
  payment: {
    method: "Bank Transfer",
    status: "PAID"
  }
}
```

## Analisis

Satu query langsung memperoleh:

- informasi order,
- informasi customer,
- informasi pengiriman,
- seluruh item,
- harga transaksi,
- subtotal,
- total,
- informasi pembayaran.

Tidak diperlukan query tambahan untuk mengambil `OrderItem` dari collection lain.

---

# 10. Mengakses Seluruh Order Customer

Untuk mendapatkan seluruh order milik `CUS001`:

```javascript
db.orders.find({
  customer_id: "CUS001"
}).pretty()
```

Contoh hasil:

```text
ORD001
ORD002
```

Secara konsep:

```text
Customer CUS001
       |
       +---- ORD001
       |
       +---- ORD002
```

Ini merupakan contoh relationship:

```text
Customer 1 : N Order
```

Relationship tersebut dapat memiliki jumlah order yang besar sehingga order tetap menjadi collection tersendiri, bukan seluruh order di-embed ke dalam dokumen customer.

---

# 11. Mengakses Order Customer Berdasarkan Tanggal

Contoh kebutuhan:

> Ambil order milik `CUS001`, diurutkan dari order terbaru.

Query:

```javascript
db.orders.find({
  customer_id: "CUS001"
}).sort({
  order_date: -1
})
```

Contoh hasil:

```text
ORD002  2026-09-17
ORD001  2026-09-16
```

Query ini menunjukkan pentingnya mempertimbangkan **query pattern** sebelum menentukan index.

---

# 12. Mengambil Order pada Rentang Tanggal

Contoh:

```javascript
db.orders.find({
  customer_id: "CUS001",
  order_date: {
    $gte: ISODate("2026-09-16T00:00:00Z"),
    $lt: ISODate("2026-09-18T00:00:00Z")
}).pretty()
```

Query tersebut mengambil order customer `CUS001` pada periode:

```text
16 September 2026
sampai sebelum
18 September 2026
```

---

# 13. Mencari Order Berdasarkan Produk

Karena `items` di-embed, MongoDB dapat mencari produk di dalam array:

```javascript
db.orders.find({
  "items.product_id": "PROD001"
}).pretty()
```

Hasil:

```text
ORD001
```

Artinya `ORD001` memiliki item dengan:

```text
product_id = PROD001
```

---

# 14. Membuat Index

## 14.1 Index Customer

Karena query sering menggunakan:

```javascript
customer_id
```

buat index:

```javascript
db.orders.createIndex({
  customer_id: 1
})
```

Contoh hasil:

```text
customer_id_1
```

---

# 15. Compound Index

Untuk query:

```javascript
db.orders.find({
  customer_id: "CUS001"
}).sort({
  order_date: -1
})
```

index yang lebih sesuai adalah:

```javascript
db.orders.createIndex({
  customer_id: 1,
  order_date: -1
})
```

Index tersebut mendukung pola:

```text
customer_id + order_date
```

dan cocok untuk kebutuhan riwayat order customer yang diurutkan berdasarkan tanggal.

---

# 16. Melihat Index

Gunakan:

```javascript
db.orders.getIndexes()
```

Contoh hasil:

```javascript
[
  {
    v: 2,
    key: {
      _id: 1
    },
    name: "_id_"
  },
  {
    v: 2,
    key: {
      customer_id: 1
    },
    name: "customer_id_1"
  },
  {
    v: 2,
    key: {
      customer_id: 1,
      order_date: -1
    },
    name: "customer_id_1_order_date_-1"
  }
]
```

---

# 17. Mengevaluasi Query dengan `explain()`

Gunakan:

```javascript
db.orders.find({
  customer_id: "CUS001"
}).explain("executionStats")
```

MongoDB akan memberikan informasi mengenai proses eksekusi query.

Beberapa informasi penting yang dapat diperhatikan:

### `totalDocsExamined`

Jumlah dokumen yang diperiksa MongoDB.

### `totalKeysExamined`

Jumlah index key yang diperiksa.

### `nReturned`

Jumlah dokumen yang dikembalikan.

Contoh interpretasi sederhana:

```text
totalDocsExamined : 2
totalKeysExamined : 2
nReturned         : 2
```

Pada dataset kecil, perbedaan performa mungkin belum terlihat signifikan. Namun, ketika jumlah data berkembang menjadi jutaan atau puluhan juta dokumen, desain index dan access pattern menjadi jauh lebih penting.

---

# 18. Memahami Desain Collection

Model yang digunakan:

```text
orders
│
├── order_id
├── customer_id
├── order_date
├── status
├── shipping
│
├── items[]
│   ├── product_id
│   ├── product_name
│   ├── quantity
│   ├── price
│   └── subtotal
│
├── total_amount
│
└── payment
    ├── method
    └── status
```

Desain ini merupakan pendekatan **document-oriented modeling**.

---

# 19. Mengapa `OrderItem` Di-Embedding?

`OrderItem` di-embedding karena:

1. Item merupakan bagian dari order.
2. Item biasanya dibaca bersama order.
3. Detail item diperlukan ketika melihat detail transaksi.
4. Jumlah item dalam satu order relatif terbatas dibandingkan jumlah seluruh order.
5. Harga item dapat disimpan sebagai snapshot transaksi.

Contoh:

```text
ORD001
  |
  +-- Laptop ABC
  |
  +-- Mouse ABC
```

Ketika `ORD001` dibaca, seluruh item langsung tersedia.

---

# 20. Mengapa Order Tidak Di-Embedding ke Customer?

Misalnya kita membuat:

```text
customer
 └── orders[]
```

Pendekatan tersebut dapat menjadi masalah jika customer memiliki order dalam jumlah sangat besar.

Relationship:

```text
Customer 1 : N Order
```

memiliki potensi pertumbuhan yang besar.

Karena itu, dalam contoh ini:

```text
customers
orders
```

dipisahkan, sedangkan:

```text
Order
 └── items[]
```

menggunakan embedding.

---

# 21. Hubungan dengan Konsep Cardinality

Contoh relationship:

| Relationship | Cardinality | Pendekatan |
|---|---|---|
| Customer → Order | 1:N | Collection terpisah |
| Order → OrderItem | 1:N | Embed |
| Customer → Cart | 1:1 | Dapat di-embed/terpisah sesuai access pattern |
| Product → Review | 1:N | Sebaiknya collection terpisah jika tidak terbatas |

Hal penting dalam NoSQL:

> **Tidak semua relationship 1:N harus diperlakukan sama.**

Keputusan embedding atau referencing harus mempertimbangkan:

- cardinality,
- pertumbuhan data,
- access pattern,
- query pattern,
- lifecycle,
- frekuensi update,
- consistency,
- kebutuhan scalability.

---

# 22. Access Pattern vs Query Pattern

### Access Pattern

Menjelaskan **data apa yang ingin diakses oleh aplikasi**.

Contoh:

```text
User membuka detail order
```

### Query Pattern

Menjelaskan **bagaimana database mengambil data tersebut**.

Contoh:

```javascript
db.orders.findOne({
  order_id: "ORD001"
})
```

Jadi:

```text
Access Pattern
      ↓
Detail order
      ↓
Query Pattern
      ↓
findOne({ order_id: "ORD001" })
      ↓
Data Order + Items
```

---

# 23. Prinsip Query-Centric Modeling

Model MongoDB sebaiknya tidak hanya dimulai dari:

```text
Entity apa yang ada?
```

Tetapi juga:

```text
Data apa yang paling sering dibaca?
Bagaimana data dicari?
Bagaimana data ditulis?
Seberapa besar cardinality?
Seberapa cepat data bertambah?
Bagaimana kebutuhan consistency?
Bagaimana kebutuhan availability?
Bagaimana data akan scale?
```

Untuk kasus ini:

```text
Requirement
     ↓
Detail Order harus cepat
     ↓
Access Pattern
     ↓
order_id
     ↓
Query Pattern
     ↓
findOne(order_id)
     ↓
Order + Items
     ↓
Embedding
```

---

# 24. Latihan Praktikum

Lakukan tugas berikut menggunakan database `marketplace`.

## Latihan 1 — Tambahkan Order

Tambahkan order baru:

```text
order_id       : ORD003
customer_id    : CUS002
status         : PAID
product        : PROD004
product_name   : Monitor XYZ
quantity       : 2
price          : 2500000
subtotal       : 5000000
total_amount   : 5000000
```

Gunakan:

```javascript
db.orders.insertOne({
  // lengkapi sendiri
})
```

---

## Latihan 2 — Cari Order

Cari order:

```text
ORD003
```

Gunakan:

```javascript
db.orders.findOne({
  // lengkapi
})
```

---

## Latihan 3 — Cari Order Customer

Cari seluruh order:

```text
CUS002
```

Gunakan:

```javascript
db.orders.find({
  // lengkapi
})
```

---

## Latihan 4 — Cari Produk

Cari seluruh order yang mengandung:

```text
PROD004
```

Gunakan:

```javascript
db.orders.find({
  // lengkapi
})
```

---

## Latihan 5 — Sorting

Tampilkan seluruh order `CUS001` dari yang terbaru.

Gunakan:

```javascript
db.orders.find({
  // lengkapi
}).sort({
  // lengkapi
})
```

---

## Latihan 6 — Index

Buat compound index untuk:

```text
customer_id + order_date
```

Gunakan:

```javascript
db.orders.createIndex({
  // lengkapi
})
```

---

## Latihan 7 — Explain

Jalankan:

```javascript
db.orders.find({
  customer_id: "CUS001"
}).explain("executionStats")
```

Kemudian identifikasi:

1. `totalDocsExamined`
2. `totalKeysExamined`
3. `nReturned`
4. apakah query menggunakan index?

---



> **Dalam MongoDB, model data sebaiknya dirancang berdasarkan bagaimana aplikasi mengakses data.**
