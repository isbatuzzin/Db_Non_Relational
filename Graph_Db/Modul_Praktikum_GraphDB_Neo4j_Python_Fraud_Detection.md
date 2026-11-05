# Modul Praktikum

# Graph-Based Fraud Detection dengan Neo4j dan Python

**Mini-project:** membangun graph
rekening--nasabah--perangkat--alamat--transaksi, menelusuri pola
hubungan mencurigakan, mengekstrak fitur graph, lalu melatih model
Machine Learning untuk menghasilkan *fraud risk score*.

-   DBMS: Neo4j Community Edition
-   Bahasa: Python 3.10+
-   Model ML: Random Forest (baseline)
-   Dataset: sintetis, disediakan dalam folder `data/`

------------------------------------------------------------------------

## 1. Capaian Pembelajaran

Setelah menyelesaikan praktikum, mahasiswa diharapkan mampu:

1.  Menjelaskan konsep node, relationship, property, dan graph
    traversal.
2.  Menyiapkan Neo4j dan menghubungkannya dengan Python.
3.  Memodelkan data nasabah, rekening, perangkat, alamat, dan transaksi
    sebagai graph.
4.  Menjalankan query Cypher untuk menemukan rekening yang berbagi
    entitas dan pola transaksi.
5.  Menghasilkan fitur berbasis graph untuk Machine Learning.
6.  Melatih dan mengevaluasi model klasifikasi fraud sederhana.
7.  Menginterpretasikan hasil *risk scoring* secara hati-hati.

## 2. Gambaran Kasus

Sebuah institusi keuangan ingin mengidentifikasi rekening yang
berpotensi terlibat dalam jaringan transaksi mencurigakan. Analisis
tabel secara terpisah dapat melewatkan hubungan tersembunyi. Graph
Database memungkinkan analis menelusuri hubungan antarrekening melalui
perangkat, alamat, nasabah, dan transaksi.

### 2.1 Entitas dan relasi

  Label/node      Contoh ID       Makna
  --------------- --------------- --------------------------
  `Customer`      C001            Nasabah
  `Account`       A001            Rekening
  `Device`        D-FRAUD-01      Perangkat yang digunakan
  `Address`       ADDR-FRAUD-01   Alamat terdaftar
  `Transaction`   T0001           Peristiwa transfer

Relasi yang dibangun:

-   `(Customer)-[:OWNS]->(Account)`
-   `(Account)-[:USES]->(Device)`
-   `(Account)-[:REGISTERED_AT]->(Address)`
-   `(Account)-[:SENT]->(Transaction)`
-   `(Transaction)-[:RECEIVED_BY]->(Account)`

### 2.2 Alur praktikum

``` text
CSV sintetis
    |
    v
Validasi dan persiapan data
    |
    v
Neo4j Graph Database
    |
    +--> Cypher: shared device / shared address
    |
    +--> Cypher: pola transfer dan konektivitas
    |
    v
Graph feature extraction dengan Python
    |
    v
Random Forest classifier
    |
    v
Fraud probability / risk score
    |
    v
Evaluasi dan interpretasi oleh analis
```

## 3. Requirement Tools dan Spesifikasi

### 3.1 Perangkat lunak

  -------------------------------------------------------------------------
  Tools                   Fungsi                  Versi/acuan
  ----------------------- ----------------------- -------------------------
  Docker Desktop + Docker Menjalankan Neo4j       Versi stabil sesuai OS
  Compose                 secara lokal            

  Neo4j Community         Graph Database          Image `neo4j:2026.10.1`
                                                  pada contoh

  Python                  Pemrosesan data dan ML  3.10 atau lebih baru

  Visual Studio Code      Editor kode             Versi stabil

  Neo4j Browser           Antarmuka query Cypher  `http://localhost:7474`

  Neo4j Python Driver     Koneksi Python--Neo4j   6.x

  pandas                  Manipulasi data         Versi kompatibel dari pip

  scikit-learn            Model dan evaluasi ML   Versi kompatibel dari pip
  -------------------------------------------------------------------------

**Catatan versi:** contoh Docker menggunakan tag Neo4j `2026.10.1`
sebagai versi yang ditampilkan dalam dokumentasi resmi saat modul
disusun. Jika tag tersebut tidak tersedia di registry pada saat
praktikum, gunakan tag Community yang tersedia dan kompatibel. Driver
Python Neo4j 6.x mendukung Neo4j 4.4, 5.x, 2025.x, dan 2026.x.

### 3.2 Unduh dan instalasi

1.  Docker Desktop: https://www.docker.com/products/docker-desktop/
2.  Python: https://www.python.org/downloads/
3.  Visual Studio Code: https://code.visualstudio.com/
4.  Dokumentasi Neo4j: https://neo4j.com/docs/
5.  Dokumentasi Neo4j Python Driver:
    https://neo4j.com/docs/python-manual/current/

Pada Windows, aktifkan WSL 2 jika diminta oleh Docker Desktop. Pada
macOS, gunakan Docker Desktop sesuai jenis prosesor (Apple Silicon atau
Intel). Pada Linux, instal Docker Engine dan Docker Compose plugin
sesuai distribusi.

## 4. Menyiapkan Folder Praktikum

Ekstrak paket praktikum ke folder kerja, misalnya:

``` text
graph_fraud_lab/
├── data/
│   ├── customers.csv
│   ├── accounts.csv
│   ├── devices.csv
│   ├── addresses.csv
│   └── transactions.csv
├── scripts/
│   ├── import_to_neo4j.py
│   └── train_model.py
├── docker-compose.yml
├── requirements.txt
├── .env.example
└── README.txt
```

Dataset sudah disiapkan. Tidak perlu mengunduh data eksternal.

### 4.1 Mengenal isi dataset

  -----------------------------------------------------------------------
  File                    Kolom utama             Keterangan
  ----------------------- ----------------------- -----------------------
  `customers.csv`         customer_id, name,      Data nasabah sintetis
                          kyc_status              

  `accounts.csv`          account_id,             Rekening dan label
                          customer_id, device_id, latihan
                          address_id, is_fraud    

  `devices.csv`           device_id, device_type, Data perangkat
                          os                      

  `addresses.csv`         address_id, city,       Data alamat
                          postal_area             

  `transactions.csv`      transaction_id,         Data transfer sintetis
                          source_account,         
                          target_account, amount, 
                          timestamp, channel,     
                          is_suspicious           
  -----------------------------------------------------------------------

Dataset berisi 50 rekening, 50 nasabah, dan transaksi sintetis yang
mengandung pola normal serta kelompok transaksi yang sengaja dibuat
saling terhubung. Sepuluh rekening diberi label fraud untuk kebutuhan
supervised learning. Label tersebut adalah *ground truth* buatan dalam
simulasi, bukan hasil investigasi nyata.

**Perhatian terhadap label:** kolom `is_fraud` pada `accounts.csv`
digunakan sebagai target pelatihan. Kolom tersebut tidak boleh dipakai
sebagai fitur input model. Script mengambil label sebagai `y` dan fitur
relasional/transaksi sebagai `X`.

## 5. Menjalankan Neo4j dengan Docker

Buka terminal pada folder `graph_fraud_lab`.

### Langkah 1 --- Periksa Docker

``` bash
docker --version
docker compose version
```

Jika perintah tidak dikenali, selesaikan instalasi Docker terlebih
dahulu dan pastikan Docker Desktop/daemon sedang berjalan.

### Langkah 2 --- Jalankan Neo4j

File `docker-compose.yml` sudah disediakan. Isinya:

``` yaml
services:
  neo4j:
    image: neo4j:2026.10.1
    container_name: neo4j-fraud-lab
    ports:
      - "7474:7474"
      - "7687:7687"
    environment:
      NEO4J_AUTH: neo4j/GraphLab2026!
    volumes:
      - neo4j_data:/data
      - neo4j_logs:/logs
    restart: unless-stopped
volumes:
  neo4j_data:
  neo4j_logs:
```

Jalankan:

``` bash
docker compose up -d
```

Periksa status:

``` bash
docker compose ps
docker logs neo4j-fraud-lab
```

Tunggu sampai log menunjukkan Neo4j siap menerima koneksi.

### Langkah 3 --- Login ke Neo4j Browser

Buka:

``` text
http://localhost:7474
```

Gunakan:

  Parameter     Nilai
  ------------- -------------------------------------------------------
  Connect URL   `neo4j://localhost:7687` atau `bolt://localhost:7687`
  Username      `neo4j`
  Password      `GraphLab2026!`
  Database      `neo4j`

Jika diminta mengganti password pada konfigurasi lain, gunakan password
baru yang sama dengan konfigurasi `.env` pada langkah Python. Password
contoh ini hanya untuk lab lokal, jangan digunakan pada sistem produksi.

### Langkah 4 --- Uji koneksi dengan Cypher

Jalankan di Neo4j Browser:

``` cypher
RETURN "Neo4j siap digunakan" AS status;
```

Jika hasil tampil, database sudah berjalan.

## 6. Menyiapkan Python Environment

### 6.1 Periksa Python

``` bash
python --version
```

Jika sistem menggunakan perintah `python3`:

``` bash
python3 --version
```

Pastikan Python 3.10 atau lebih baru.

### 6.2 Buat virtual environment

**Windows PowerShell:**

``` powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

Jika PowerShell menolak aktivasi karena kebijakan eksekusi, ikuti
petunjuk resmi Python/PowerShell untuk mengizinkan skrip aktivasi pada
sesi pengguna, atau gunakan Command Prompt:

``` cmd
.venv\Scripts\activate.bat
```

**macOS/Linux:**

``` bash
python3 -m venv .venv
source .venv/bin/activate
```

Terminal yang aktif biasanya menampilkan prefix `(.venv)`.

### 6.3 Instal library

``` bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Library yang digunakan:

-   `neo4j`: driver resmi untuk koneksi ke Neo4j.
-   `python-dotenv`: membaca konfigurasi `.env`.
-   `pandas`: mengolah hasil query menjadi DataFrame.
-   `scikit-learn`: melatih dan mengevaluasi Random Forest.
-   `joblib`: menyimpan model hasil pelatihan.

### 6.4 Buat file konfigurasi `.env`

Salin `.env.example` menjadi `.env`.

**Windows PowerShell:**

``` powershell
Copy-Item .env.example .env
```

**macOS/Linux:**

``` bash
cp .env.example .env
```

Isi `.env`:

``` dotenv
NEO4J_URI=bolt://localhost:7687
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=GraphLab2026!
NEO4J_DATABASE=neo4j
```

Jangan membagikan file `.env` yang berisi password. Pada proyek nyata,
simpan secret dengan pengelolaan kredensial yang aman.

## 7. Memahami Model Graph Sebelum Import

Perhatikan representasi berikut:

``` text
(Customer C001)-[:OWNS]->(Account A001)
(Account A001)-[:USES]->(Device D-FRAUD-01)
(Account A001)-[:REGISTERED_AT]->(Address ADDR-FRAUD-01)

(Account A001)-[:SENT]->(Transaction T0001)
(Transaction T0001)-[:RECEIVED_BY]->(Account A002)
```

Pada model ini, transaksi direpresentasikan sebagai node karena
transaksi memiliki atribut sendiri: ID, nominal, waktu, channel, dan
label simulasi. Model lain dapat menggunakan relationship langsung jika
kebutuhan dan atributnya lebih sederhana.

## 8. Import Dataset ke Neo4j Menggunakan Python

Script `scripts/import_to_neo4j.py` akan:

1.  Membaca seluruh CSV.
2.  Membuat uniqueness constraint untuk ID.
3.  Membuat node Customer, Account, Device, Address, dan Transaction.
4.  Membuat relationship antarentitas.
5.  Menggunakan `MERGE` agar proses import dapat dijalankan ulang tanpa
    menggandakan node dengan ID yang sama.

### Langkah 1 --- Jalankan script

Pastikan: - Docker Neo4j aktif. - Virtual environment aktif. - File
`.env` sudah benar.

Jalankan:

``` bash
python scripts/import_to_neo4j.py
```

Contoh pesan sukses:

``` text
Import selesai: 50 customer, 50 account, ... transaksi.
```

Jumlah transaksi aktual mengikuti file `transactions.csv`.

### Langkah 2 --- Periksa jumlah node

Jalankan di Neo4j Browser:

``` cypher
MATCH (n)
RETURN labels(n) AS label, count(n) AS jumlah
ORDER BY label;
```

### Langkah 3 --- Periksa relationship

``` cypher
MATCH ()-[r]->()
RETURN type(r) AS relationship, count(r) AS jumlah
ORDER BY relationship;
```

### Langkah 4 --- Tampilkan graph

``` cypher
MATCH (c:Customer)-[:OWNS]->(a:Account)
OPTIONAL MATCH (a)-[r1:USES]->(d:Device)
OPTIONAL MATCH (a)-[r2:REGISTERED_AT]->(x:Address)
RETURN c, a, r1, d, r2, x
LIMIT 30;
```

Klik tampilan Graph pada Neo4j Browser untuk melihat node dan hubungan.

## 9. Eksperimen 1 --- Menemukan Shared Device

**Tujuan:** menemukan dua rekening berbeda yang menggunakan perangkat
yang sama.

``` cypher
MATCH (a:Account)-[:USES]->(d:Device)<-[:USES]-(b:Account)
WHERE a.account_id < b.account_id
RETURN a.account_id AS account_1,
       d.device_id AS shared_device,
       b.account_id AS account_2
ORDER BY shared_device;
```

**Interpretasi:**

-   Jika dua rekening menggunakan satu device, keduanya memiliki
    hubungan melalui perangkat tersebut.
-   Perangkat yang digunakan banyak rekening dapat menjadi sinyal untuk
    investigasi.
-   Perangkat keluarga, perangkat bersama, atau perangkat publik dapat
    menghasilkan hubungan yang sah.

**Pertanyaan praktikum:**

1.  Berapa pasangan rekening yang berbagi perangkat?
2.  Perangkat mana yang menghubungkan rekening terbanyak?
3.  Apakah semua rekening yang berbagi perangkat patut dianggap fraud?
    Jelaskan.

## 10. Eksperimen 2 --- Menemukan Shared Address

**Tujuan:** menemukan rekening yang memiliki alamat terdaftar sama.

``` cypher
MATCH (a:Account)-[:REGISTERED_AT]->(x:Address)<-[:REGISTERED_AT]-(b:Account)
WHERE a.account_id < b.account_id
RETURN a.account_id AS account_1,
       x.address_id AS shared_address,
       b.account_id AS account_2
ORDER BY shared_address;
```

Untuk mencari alamat yang terhubung ke lebih dari satu rekening:

``` cypher
MATCH (a:Account)-[:REGISTERED_AT]->(x:Address)
WITH x, collect(a.account_id) AS accounts
WHERE size(accounts) > 1
RETURN x.address_id AS address,
       size(accounts) AS account_count,
       accounts
ORDER BY account_count DESC;
```

**Pertanyaan praktikum:**

1.  Alamat mana yang memiliki jumlah rekening terbanyak?
2.  Apa perbedaan shared address dan bukti kepemilikan bersama?
3.  Mengapa alamat yang sama perlu diperiksa bersama informasi lain?

## 11. Eksperimen 3 --- Menganalisis Pola Transfer

**Tujuan:** melihat aliran dana antar rekening.

### 11.1 Daftar transaksi terbesar

``` cypher
MATCH (s:Account)-[:SENT]->(t:Transaction)-[:RECEIVED_BY]->(r:Account)
RETURN t.transaction_id AS transaction,
       s.account_id AS source,
       r.account_id AS target,
       t.amount AS amount,
       t.timestamp AS timestamp
ORDER BY amount DESC
LIMIT 15;
```

### 11.2 Rekening dengan transaksi keluar terbanyak

``` cypher
MATCH (a:Account)-[:SENT]->(t:Transaction)
RETURN a.account_id AS account,
       count(t) AS outgoing_transactions,
       sum(t.amount) AS total_outgoing
ORDER BY outgoing_transactions DESC
LIMIT 10;
```

### 11.3 Rekening dengan transaksi masuk terbanyak

``` cypher
MATCH (t:Transaction)-[:RECEIVED_BY]->(a:Account)
RETURN a.account_id AS account,
       count(t) AS incoming_transactions,
       sum(t.amount) AS total_incoming
ORDER BY incoming_transactions DESC
LIMIT 10;
```

### 11.4 Menelusuri pola transfer dua langkah

``` cypher
MATCH (a:Account)-[:SENT]->(:Transaction)-[:RECEIVED_BY]->(b:Account),
      (b)-[:SENT]->(:Transaction)-[:RECEIVED_BY]->(c:Account)
WHERE a <> b AND b <> c AND a <> c
RETURN a.account_id AS first_account,
       b.account_id AS intermediary,
       c.account_id AS final_account
LIMIT 30;
```

**Interpretasi:** pola A → B → C menunjukkan aliran dana melalui
perantara. Pola ini dapat muncul pada aktivitas normal maupun
mencurigakan; konteks nominal, waktu, frekuensi, profil nasabah, dan
hasil investigasi tetap diperlukan.

## 12. Eksperimen 4 --- Menghitung Graph Features

Pada tahap ini, Graph Database digunakan sebagai sumber fitur untuk
Machine Learning.

Fitur yang diekstrak:

  -----------------------------------------------------------------------
  Fitur                               Definisi
  ----------------------------------- -----------------------------------
  `shared_device_accounts`            Jumlah rekening lain yang berbagi
                                      perangkat

  `shared_address_accounts`           Jumlah rekening lain yang berbagi
                                      alamat

  `outgoing_count`                    Jumlah transaksi keluar

  `outgoing_amount`                   Total nominal transaksi keluar

  `incoming_count`                    Jumlah transaksi masuk

  `incoming_amount`                   Total nominal transaksi masuk
  -----------------------------------------------------------------------

Script `scripts/train_model.py` menjalankan query fitur langsung ke
Neo4j, membentuk DataFrame, kemudian menggunakan fitur tersebut untuk
melatih model.

### 12.1 Jalankan script ML

``` bash
python scripts/train_model.py
```

Script akan:

1.  Membaca fitur dari graph.
2.  Memisahkan fitur (`X`) dan label (`y`).
3.  Membagi data train dan test secara stratified.
4.  Melatih Random Forest dengan `class_weight="balanced"`.
5.  Menghitung classification report, confusion matrix, PR-AUC, dan
    ROC-AUC.
6.  Menyimpan `outputs_risk_scores.csv` dan
    `fraud_random_forest.joblib`.

### 12.2 Memahami proses feature extraction

Bagian penting dalam script:

``` python
FEATURE_QUERY = """
MATCH (a:Account)
OPTIONAL MATCH (a)-[:USES]->(d:Device)<-[:USES]-(otherD:Account)
WHERE otherD <> a
WITH a, count(DISTINCT otherD) AS shared_device_accounts
OPTIONAL MATCH (a)-[:REGISTERED_AT]->(x:Address)<-[:REGISTERED_AT]-(otherA:Account)
WHERE otherA <> a
WITH a, shared_device_accounts, count(DISTINCT otherA) AS shared_address_accounts
OPTIONAL MATCH (a)-[:SENT]->(outTx:Transaction)
WITH a, shared_device_accounts, shared_address_accounts,
     count(outTx) AS outgoing_count, coalesce(sum(outTx.amount),0) AS outgoing_amount
OPTIONAL MATCH (inTx:Transaction)-[:RECEIVED_BY]->(a)
RETURN a.account_id AS account_id, a.is_fraud AS label,
       shared_device_accounts, shared_address_accounts,
       outgoing_count, outgoing_amount,
       count(inTx) AS incoming_count,
       coalesce(sum(inTx.amount),0) AS incoming_amount
"""
```

Query menghasilkan satu baris fitur untuk setiap rekening. Label
`a.is_fraud` dipakai sebagai target, bukan sebagai fitur.

### 12.3 Memahami model

Random Forest membangun banyak decision tree dan menggabungkan hasil
prediksi. Model ini dipilih sebagai baseline karena relatif mudah
digunakan untuk data tabular dan dapat dibandingkan dengan model yang
lebih kompleks.

Probabilitas prediksi kelas fraud diperoleh melalui:

``` python
prob = model.predict_proba(X_test)[:, 1]
```

Contoh interpretasi: - `0.90`: model memperkirakan probabilitas kelas
fraud 90% menurut pola yang dipelajari. - `0.20`: model memperkirakan
probabilitas kelas fraud 20%.

Angka tersebut bukan kepastian dan tidak otomatis menjadi bukti fraud.
Kalibrasi probabilitas, kualitas label, representativitas data, dan
validasi eksternal perlu diperhatikan.

## 13. Eksperimen 5 --- Membaca Risk Score

Setelah script selesai, buka file:

``` text
outputs_risk_scores.csv
```

Kolom yang tersedia:

  Kolom                 Arti
  --------------------- -----------------------------------------
  `account_id`          ID rekening
  `actual_label`        Label simulasi sebenarnya
  `fraud_probability`   Probabilitas prediksi kelas fraud
  `predicted_alert`     Hasil klasifikasi dengan threshold 0.50

Contoh pembacaan: - Urutkan `fraud_probability` dari terbesar ke
terkecil. - Periksa apakah rekening dengan skor tinggi memiliki shared
device, shared address, atau pola transaksi yang tidak biasa. -
Bandingkan prediksi dengan `actual_label` untuk mengukur kesalahan
model.

Threshold 0.50 hanya contoh pembelajaran. Pada sistem nyata, threshold
dipilih berdasarkan validasi, biaya false positive/false negative,
kapasitas analis, dan risiko operasional.

## 14. Evaluasi Model

Jangan hanya menggunakan accuracy karena data fraud sering tidak
seimbang.

  Metrik             Pertanyaan yang dijawab
  ------------------ ---------------------------------------------------------
  Precision          Dari semua alert, berapa yang benar-benar fraud?
  Recall             Dari semua kasus fraud, berapa yang berhasil ditemukan?
  F1-score           Bagaimana keseimbangan precision dan recall?
  PR-AUC             Seberapa baik model pada kelas positif yang jarang?
  ROC-AUC            Seberapa baik model membedakan dua kelas secara umum?
  Confusion matrix   Berapa TP, FP, TN, dan FN?

### 14.1 Membaca confusion matrix

                        Prediksi legitimate   Prediksi fraud
  ------------------- --------------------- ----------------
  Aktual legitimate                      TN               FP
  Aktual fraud                           FN               TP

-   **False Positive (FP):** rekening legitimate ditandai sebagai fraud.
-   **False Negative (FN):** rekening fraud tidak terdeteksi.
-   **True Positive (TP):** rekening fraud terdeteksi.
-   **True Negative (TN):** rekening legitimate dikenali dengan benar.

### 14.2 Catatan metodologis

Split stratified acak dalam script digunakan agar praktikum baseline
dapat dijalankan pada dataset kecil. Untuk eksperimen yang lebih
realistis:

1.  Gunakan pembagian train/test berdasarkan waktu.
2.  Pastikan fitur hanya menggunakan informasi yang tersedia sebelum
    waktu prediksi.
3.  Hindari menggunakan label hasil investigasi masa depan sebagai
    fitur.
4.  Lakukan validasi pada periode atau kelompok nasabah yang berbeda.
5.  Periksa data leakage, class imbalance, dan perubahan pola fraud
    (*concept drift*).

## 15. Tugas dan Pertanyaan Analisis

1.  Buat diagram graph dari minimal 10 rekening beserta perangkat dan
    alamatnya.
2.  Temukan tiga perangkat dengan jumlah rekening terbanyak.
3.  Temukan tiga alamat dengan jumlah rekening terbanyak.
4.  Identifikasi tiga rekening dengan transaksi keluar terbesar.
5.  Jelaskan satu pola transfer berantai yang ditemukan.
6.  Jelaskan bagaimana shared device dan shared address menjadi fitur
    ML.
7.  Bandingkan precision dan recall model.
8.  Identifikasi contoh false positive dan false negative.
9.  Jelaskan mengapa graph relationship tidak dapat langsung dianggap
    sebagai bukti fraud.
10. Usulkan dua fitur graph tambahan untuk meningkatkan deteksi.


## 16. Troubleshooting

### A. Docker tidak berjalan

Periksa Docker Desktop/daemon, lalu:

``` bash
docker compose ps
docker logs neo4j-fraud-lab
```

### B. Tidak dapat membuka Neo4j Browser

Pastikan container aktif dan port `7474` tidak digunakan aplikasi lain.

### C. Python gagal terhubung ke Neo4j

Periksa: - Neo4j berjalan. - URI menggunakan `bolt://localhost:7687`. -
Username dan password pada `.env` benar. - Port `7687` tidak diblokir
firewall.

### D. ModuleNotFoundError

Pastikan virtual environment aktif dan jalankan:

``` bash
pip install -r requirements.txt
```

### E. Constraint atau data sudah ada

Script menggunakan `MERGE` dan constraint `IF NOT EXISTS`, sehingga
dapat dijalankan ulang. Jika ingin mengulang dari kondisi kosong, hapus
data database lab dengan hati-hati:

``` cypher
MATCH (n)
DETACH DELETE n;
```

Jangan jalankan perintah penghapusan tersebut pada database yang berisi
data penting.

### F. Password berubah

Sesuaikan `NEO4J_PASSWORD` dalam `.env` dengan password yang benar-benar
berlaku di database. Mengubah nilai environment pada container yang
sudah memiliki volume database tidak otomatis mengganti password
database yang sudah tersimpan.

## 17. Menghentikan Praktikum

Untuk menghentikan container tanpa menghapus volume data:

``` bash
docker compose stop
```

Untuk menjalankan kembali:

``` bash
docker compose start
```

Untuk menghapus container dan jaringan tetapi mempertahankan volume:

``` bash
docker compose down
```

Untuk menghapus seluruh data volume lab secara permanen:

``` bash
docker compose down -v
```

Perintah terakhir akan menghapus database praktikum. Pastikan tidak ada
data yang masih dibutuhkan.

## 18. Kriteria Keberhasilan Praktikum

  No.   Kriteria                            Bukti
  ----- ----------------------------------- -----------------------------------
  1     Neo4j berjalan                      Screenshot Neo4j Browser
  2     Python terhubung ke Neo4j           Log koneksi/import
  3     Graph berhasil dibangun             Visualisasi node dan relationship
  4     Query shared entity berhasil        Hasil Cypher
  5     Pola transfer berhasil dianalisis   Hasil query transfer
  6     Graph features berhasil diekstrak   DataFrame/CSV fitur
  7     Model ML berhasil dilatih           Classification report
  8     Risk score berhasil dihasilkan      `outputs_risk_scores.csv`
  9     Interpretasi hasil dibuat           Laporan analisis

## Referensi Resmi

1.  Neo4j Python Driver Manual --- Installation:
    https://neo4j.com/docs/python-manual/current/install/
2.  Neo4j Python Driver Manual --- Build applications with Python:
    https://neo4j.com/docs/python-manual/current/
3.  Neo4j Operations Manual --- Docker:
    https://neo4j.com/docs/operations-manual/current/docker/
4.  Neo4j Cypher Manual --- Constraints:
    https://neo4j.com/docs/cypher-manual/current/schema/constraints/
5.  Neo4j Cypher Manual --- Indexes:
    https://neo4j.com/docs/cypher-manual/current/indexes/
6.  scikit-learn --- User Guide:
    https://scikit-learn.org/stable/user_guide.html
