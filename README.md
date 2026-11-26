# GitHub Developer Network: Louvain vs Leiden Analysis

Analisis struktur komunitas pada jaringan sosial pengembang GitHub (37.700 simpul, 289.003 sisi) dengan algoritma Louvain dan Leiden. Notebook menjawab empat pertanyaan:

1. Apa perbedaan hasil Louvain dan Leiden (modularity, ukuran komunitas, keterhubungan internal, waktu komputasi)?
2. Seberapa besar parameter resolusi (gamma) dan random seed memengaruhi struktur dan kestabilan komunitas?
3. Bagaimana hubungan komunitas yang ditemukan dengan kategori pengembang Web dan Machine Learning?
4. Pengguna mana yang berpotensi menjadi penghubung antarkomunitas, dan indikator apa yang mendukungnya?

Seluruh kode ada di satu notebook: `github-developer-network-louvain-leiden-analysis.ipynb`. Proyek pendamping, [github-network-community-detection-benchmark](../github-network-community-detection-benchmark), membandingkan kedua algoritma dalam 40 run pada data yang sama dan tidak membahas label pengembang.

## Dataset

GitHub Social Network (`musae_git`) dari [Stanford SNAP](https://snap.stanford.edu/data/github-social.html). Simpul adalah pengembang GitHub dan sisi adalah hubungan saling mengikuti. Tiga file dipakai: `musae_git_edges.csv`, `musae_git_target.csv`, dan `musae_git_features.json`. Kolom `ml_target` menyimpan kategori pengembang: 0 untuk Web dan 1 untuk Machine Learning. Repo ini tidak menyertakan file data.

| Properti graf | Nilai |
|---|---|
| Simpul / sisi | 37.700 / 289.003 |
| Komponen terhubung | 1 |
| Densitas | 0,000407 |
| Degree rata-rata / minimum / maksimum | 15,33 / 1 / 9.458 |

Distribusi degree berekor panjang. Notebook menampilkannya dalam histogram dan plot log-log.

## Metode

- **Louvain:** `community_multilevel` dari python-igraph (resolusi 1,0).
- **Leiden:** `leidenalg.find_partition` dengan `RBConfigurationVertexPartition`. Parameter `n_iterations` memakai nilai bawaan library.
- **Modularity** pada seluruh eksperimen dihitung dengan gamma = 1, sehingga angkanya bisa dibandingkan antar konfigurasi.
- **Keterhubungan internal:** setiap komunitas diambil sebagai subgraf, lalu komunitas dengan lebih dari satu komponen terhubung dihitung sebagai komunitas terpecah.
- **Pengguna penghubung:** betweenness centrality (perkiraan dengan 500 simpul sampel, seed 42) dan jumlah sisi lintas komunitas tiap pengguna.

## Hasil

### 1. Louvain vs Leiden (gamma = 1,0, seed 0)

| Metrik | Louvain | Leiden |
|---|---|---|
| Jumlah komunitas | 35 | 86 |
| Modularity | 0,4446 | 0,4585 |
| Ukuran komunitas (min / median / maks) | 3 / 6 / 7.753 | 2 / 2 / 10.067 |
| Komunitas terpecah | 3 | 0 |
| Waktu (detik) | 0,95 | 6,57 |

Leiden menghasilkan modularity lebih tinggi dan tidak punya komunitas terpecah, tetapi sekitar 7 kali lebih lambat pada percobaan ini. Jumlah komunitas Leiden yang besar berasal dari banyaknya komunitas kecil (median 2 anggota), jadi jumlah komunitas saja tidak menunjukkan kualitas partisi.

### 2. Pengaruh gamma dan random seed (Leiden)

Pengaruh gamma, satu run per nilai:

| Gamma | Jumlah komunitas | Modularity |
|---|---|---|
| 0,5 | 17 | 0,3660 |
| 1,0 | 30 | 0,4537 |
| 1,5 | 35 | 0,4457 |
| 2,0 | 45 | 0,4149 |

Pengaruh seed pada gamma = 1,0:

| Seed | Jumlah komunitas | Modularity |
|---|---|---|
| 0 | 86 | 0,4585 |
| 1 | 36 | 0,4546 |
| 2 | 26 | 0,4562 |
| 3 | 32 | 0,4584 |
| 4 | 33 | 0,4525 |

Gamma menggeser jumlah komunitas dari 17 sampai 45 (simpangan baku 11,6) dan modularity dari 0,366 sampai 0,454. Seed mengubah jumlah komunitas lebih lebar, dari 26 sampai 86 (simpangan baku 24,5), sementara modularity tetap stabil (simpangan baku 0,0025). Kualitas partisi bertahan antar seed, tetapi jumlah dan ukuran komunitasnya berubah.

### 3. Komunitas dan kategori pengembang

Partisi yang dipakai: Leiden dengan gamma = 1,0 (31 komunitas, modularity 0,4544). Sebelas komunitas terbesar menampung 99,75% simpul. Enam yang terbesar:

| Komunitas | Anggota | Web | ML | Persentase ML |
|---|---|---|---|---|
| 0 | 8.574 | 8.158 | 416 | 4,85% |
| 1 | 7.945 | 2.224 | 5.721 | 72,01% |
| 2 | 6.854 | 5.522 | 1.332 | 19,43% |
| 3 | 5.380 | 4.507 | 873 | 16,23% |
| 4 | 2.813 | 2.668 | 145 | 5,15% |
| 5 | 2.629 | 2.154 | 475 | 18,07% |

Komposisi kategori berbeda jelas antar komunitas. Komunitas 0 dan 4 hampir seluruhnya pengembang Web (sekitar 95%). Komunitas 1 adalah satu-satunya komunitas besar yang didominasi pengembang Machine Learning (72%), padahal pengembang ML hanya 25,8% dari seluruh jaringan. Komunitas ini menampung 5.721 dari 9.739 pengembang ML, atau sekitar 59%. Komunitas kecil (3 sampai 22 anggota) menunjukkan persentase ekstrem seperti 100%, tetapi ukurannya terlalu kecil untuk ditafsirkan.

### 4. Pengguna penghubung antarkomunitas

Lima pengguna dengan betweenness tertinggi:

| ID pengguna | Betweenness | Komunitas | Sisi lintas komunitas |
|---|---|---|---|
| 27803 | 0,2543 | 1 | 5.040 |
| 31890 | 0,2459 | 2 | 5.974 |
| 19222 | 0,0576 | 0 | 1.560 |
| 13638 | 0,0380 | 3 | 1.572 |
| 35773 | 0,0364 | 0 | 1.465 |

Pengguna 27803 dan 31890 menonjol di kedua indikator: betweenness-nya jauh di atas pengguna lain, dan masing-masing punya ribuan sisi yang menghubungkan komunitas berbeda. Notebook menampilkan sepuluh teratas.

## Keterbatasan

- Eksperimen gamma hanya satu run per nilai tanpa seed tetap. Karena seed saja sudah mengubah jumlah komunitas dari 26 sampai 86, selisih jumlah komunitas antar gamma belum bisa dipisahkan dari variasi seed.
- Analisis kategori (bagian 3) dan pengguna penghubung (bagian 4) memakai satu partisi Leiden tanpa seed tetap. Hasilnya bisa berbeda saat notebook dijalankan ulang.
- Leiden memakai `n_iterations` bawaan, jadi tidak dijalankan sampai konvergen. Waktu 6,57 detik tidak sebanding dengan run konvergen di proyek benchmark.
- Louvain dan Leiden dibandingkan lewat satu run di bagian 1. Perbandingan dengan banyak seed ada di proyek benchmark.
- Betweenness dihitung dengan perkiraan 500 simpul sampel, bukan perhitungan penuh.
- Dua pengguna teratas juga termasuk pengguna dengan degree tertinggi, sehingga jumlah sisi lintas komunitas sebagian mencerminkan efek hub. Rasio sisi lintas komunitas terhadap degree (sekitar 71% untuk pengguna 27803 dan 63% untuk 31890) memberi gambaran yang lebih seimbang.
- Hubungan komunitas dengan kategori bersifat deskriptif. Notebook tidak menjalankan uji statistik.

## Cara menjalankan

1. Unduh tiga file data dari halaman SNAP di atas, lalu letakkan di satu folder.
2. Buka notebook di Google Colab atau Jupyter, lalu atur `DATA_DIR` ke folder tersebut (bawaan: `/content/`).
3. Jalankan semua sel berurutan. Sel pertama memasang `python-igraph` dan `leidenalg`. Library lain: NumPy, pandas, NetworkX, Matplotlib, scikit-learn.

## Struktur repo

```
github-developer-network-louvain-leiden-analysis/
├── github-developer-network-louvain-leiden-analysis.ipynb
└── README.md
```
