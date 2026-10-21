# RetailRocket Recommendation System dengan Neo4j

Proyek UTS mata kuliah Graf Sains Data. Notebook `Graph.ipynb` menyimpan data interaksi e-commerce RetailRocket sebagai graf di Neo4j, menghitung kemiripan pengunjung (cosine similarity), lalu membuat rekomendasi item dengan *User-Based KNN* dan mengevaluasinya dengan Hit@5 dan NDCG@5.

## Struktur proyek

```text
Project\
├──Readme
└── GSDD
    ├── Graph.ipynb
    ├── requirements.txt
    ├── img/
    │   ├── visualisasi_sebelum_similarity.png
    │   └── visualisasi_sesudah_similarity.png
    └── data/
        ├── events.csv       # disediakan sendiri
        ├── events_P1.csv    # dihasilkan notebook
        └── events_P2.csv    # dihasilkan notebook
```

## Graph.ipynb

Notebook dijalankan dari atas ke bawah. Ringkasan isinya:

| Bagian | Isi |
|---|---|
| Uji koneksi | Menguji koneksi Python ke Neo4j dengan contoh graf `Person`–`KNOWS` |
| 2.1 Data Wrangling dan EDA | Membersihkan `events.csv`, membagi menjadi P1 dan P2, mengimpor P1 ke Neo4j, memeriksa komposisi graf |
| VSM | Membentuk matriks sparse User–Item (`Mu`) dan Item–User (`Mi`), lalu membandingkan keduanya |
| 7. Similarity | Menghitung 5 visitor paling mirip dengan Cypher dan menyimpannya sebagai relasi `SIMILAR_TO` |
| 8. Sebelum dan sesudah | Menggambar graf sebelum dan sesudah relasi `SIMILAR_TO` ditambahkan |
| 2.2 Analytics | Rekomendasi KNN dan evaluasi Hit@5 serta NDCG@5 |

## Data

| File | Isi | Dibuat oleh |
|---|---|---|
| `data/events.csv` | Data asal sebelum diolah (seluruh periode) | diunduh dari [Kaggle](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset) |
| `data/events_P1.csv` | Data **training**: 18 Mei 2015 – 18 Juli 2015, diimpor ke Neo4j | Notebook |
| `data/events_P2.csv` | Data **testing**: 27 Juli 2015 – 11 Agustus 2015, kebenaran dasar evaluasi | Notebook |

Kolom: `timestamp` (epoch milidetik), `visitorid`, `event` (`view`, `addtocart`, `transaction`), `itemid`, `transactionid`.

## requirements.txt

Berisi paket Python yang dipakai notebook. Pasang dengan `pip install -r requirements.txt`.

| Paket | Fungsi |
|---|---|
| `neo4j` | Driver Python untuk terhubung ke Neo4j |
| `pandas`, `numpy` | Membaca dan membersihkan CSV, operasi numerik |
| `scipy` | Matriks sparse (CSR) dan perhitungan cosine similarity |
| `matplotlib`, `networkx` | Menggambar graf visualisasi |
| `ipykernel`, `jupyter` | Menjalankan notebook |
| `ipywidgets`, `tqdm` | Progress bar di Jupyter |

## Persiapan lingkungan

### 1. Python dan virtual environment

Disarankan Python 3.11 atau lebih baru. Virtual environment menjaga paket proyek ini tidak bercampur dengan proyek lain.

```bash
python -m venv .venv
```

```powershell
# Windows PowerShell
.venv\Scripts\Activate.ps1
```

```bash
# Linux/macOS
source .venv/bin/activate
```

```bash
pip install -r requirements.txt
```

### 3. Menjalankan notebook

```bash
jupyter notebook
```

Buka `Graph.ipynb` dan jalankan sel dari atas ke bawah. Jika koneksi benar, sel pertama mencetak `Koneksi Neo4j berhasil`.

> **Perhatian:** jika graf di database tidak sesuai CSV, notebook menghapus semua simpul `Visitor` dan `Item` lalu mengimpor ulang. Bagian 8 juga menghapus semua relasi `SIMILAR_TO`. Gunakan database khusus proyek ini.

## Hasil 2.2 Analytics

Seed acak 42, K = 5 tetangga, TOP_N = 5. Kandidat evaluasi adalah visitor dengan minimal 5 item berbeda di P1 yang punya item relevan di P2 (belum pernah dilihat di P1).

| Hal | Nilai |
|---|---|
| Ukuran kolam visitor | 1.153 |
| Visitor acuan | 137360 |
| Jumlah item acuan di P1 | 6 |
| Item relevan di P2 | 1 (item 64747) |

**K = 5 tetangga terdekat**

| Visitor | Similarity |
|---|---|
| 46195 | 0,408248 |
| 69445 | 0,408248 |
| 93681 | 0,408248 |
| 100078 | 0,408248 |
| 115912 | 0,408248 |

**Daftar rekomendasi top-5:** kosong (tidak ada item yang direkomendasikan).

| Metrik | Nilai |
|---|---|
| Hit@5 | 0 |
| NDCG@5 | 0,0000 |

Kelima tetangga seri pada 0,408248 (= 1/√6). Daftar rekomendasi kosong kemungkinan karena item milik para tetangga sudah ada di riwayat P1 visitor acuan sehingga tidak tersisa kandidat item baru. Karena itu item relevan 64747 tidak muncul dan kedua metrik bernilai 0. Hasil ini berasal dari satu visitor acuan dan belum mewakili kualitas sistem secara umum.

## Catatan

- `events.csv` harus disediakan sendiri; notebook tidak mengunduh dataset otomatis.
- Hanya sebagian kecil visitor P2 yang juga ada di P1 (kasus *cold-start*), sehingga evaluasi terbatas pada visitor dan item yang memenuhi kriteria di atas.
- Notebook adalah prototipe analisis, bukan aplikasi siap produksi.
