# 🚀 Cloud Ingestion via Google Colab (Free GPU)

```ps1
Compress-Archive -Path "core", "api.py", "ingest.py", "requirements.txt" -DestinationPath "pede_colab.zip" -Force
```

Karena proses pengubahan PDF menjadi vektor matematika (Embedding) membutuhkan komputasi yang berat, menjalankannya di laptop/CPU biasa akan memakan waktu lama (sekitar 5-7 menit per artikel).

Untuk memproses ratusan hingga ribuan jurnal secara **kilat (hanya dalam hitungan detik/menit)**, kita bisa meminjam GPU gratis (NVIDIA T4) dari Google Colab.

Ikuti **5 Langkah Cepat** berikut untuk melakukannya:

---

### Langkah 1: Buka Google Colab & Aktifkan GPU
1. Buka [Google Colab](https://colab.research.google.com/) dan buat **Notebook baru**.
2. Di bilah menu atas, klik **Runtime > Change runtime type**.
3. Pilih **T4 GPU** pada bagian *Hardware accelerator*, lalu klik **Save**.

### Langkah 2: Upload Source Code ke Colab
Untuk mempermudah, Anda bisa menggunakan file zip yang sudah di-generate secara lokal (`pede_colab.zip`). File ini berisi kode inti (bersih dari *database* lokal yang berat).

1. Di panel sebelah kiri layar Colab, klik ikon **Folder (Files)**.
2. Tarik dan lepas (*Drag & Drop*) file **`pede_colab.zip`** ke area panel tersebut.
3. Buat *Code Cell* baru (+ Code) di tengah layar, *copy-paste* perintah ini, lalu jalankan (klik tombol Play):
```bash
!unzip pede_colab.zip
```

### Langkah 3: Masukkan Jurnal (PDF)
Setelah diekstrak, Anda akan melihat file proyek PEDE di sebelah kiri.
1. Buat folder baru bernama `papers`.
2. Tarik dan lepas (*Drag & Drop*) semua file jurnal (PDF) yang ingin Anda masukkan ke dalam *Vector Database* ke dalam folder `papers` tersebut.

### Langkah 4: Jalankan Instalasi & Proses Ingestion
Buat *Code Cell* baru, *copy-paste* perintah ini, dan jalankan secara berurutan:

```bash
# 1. Install semua library yang dibutuhkan (termasuk Qdrant & PyMuPDF)
!pip install -r requirements.txt

# 2. Jalankan mesin pemakan PDF (Super Cepat dengan GPU T4)
!python ingest.py ./papers/
```
*Tunggu hingga log di bawah cell menunjukkan `INGESTION COMPLETE`.*

### Langkah 5: Download Hasil Database-nya!
Database *Vector* Qdrant yang sudah jadi akan tersimpan rapi di dalam folder lokal `qdrant_db`. Mari kita *zip* folder tersebut agar mudah di-*download* ke laptop Anda.

Buat *Code Cell* terakhir dan jalankan:
```bash
!zip -r qdrant_db_selesai.zip ./qdrant_db
```

1. *Refresh* panel file di kiri.
2. Klik kanan pada file `qdrant_db_selesai.zip` yang baru muncul, lalu klik **Download**.
3. Ekstrak folder `qdrant_db` tersebut kembali ke dalam direktori proyek lokal (laptop) Anda.
4. Jalankan `python api.py` di laptop Anda, dan seketika Golang Anda terhubung ke pangkalan data skala besar yang diproses via Cloud! ⚡️

---

> **💡 FAQ:** Mengapa tidak meng-install Qdrant Docker di Colab?
> Karena kita menggunakan mode `Embedded Qdrant` berbasis *local path* (`QdrantClient(path="./qdrant_db")`). Mode ini membuat Python bertindak sendiri sebagai *database engine* tanpa memerlukan Docker/Server eksternal. Sangat cocok untuk arsitektur Cloud-to-Local seperti ini!
