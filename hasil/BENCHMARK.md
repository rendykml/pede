# Benchmark Embedding Model all-mpnet-base-v2 pada Project PEDE

## Deskripsi

Project PEDE (PDF to Embedding) digunakan untuk mengubah dokumen PDF ilmiah menjadi vector embedding yang disimpan pada Qdrant Vector Database dan digunakan untuk kebutuhan Semantic Search maupun Retrieval-Augmented Generation (RAG).

Pada implementasi awal, project menggunakan model embedding BAAI/bge-m3. Dalam eksperimen ini model embedding diganti menjadi:

```text
sentence-transformers/all-mpnet-base-v2
```

Model ini dipilih karena memiliki performa yang baik untuk semantic retrieval, ukuran model yang relatif ringan, dan banyak digunakan sebagai baseline pada berbagai implementasi RAG modern.

---

## Dataset

Pengujian dilakukan menggunakan dua artikel ilmiah internasional yang berkaitan dengan transformasi digital. Kedua dokumen dipilih karena memiliki struktur akademik yang lengkap, terdiri dari abstrak, pembahasan, referensi, tabel, dan subbab sehingga cocok untuk menguji performa chunking serta embedding pada sistem RAG.

| No  | Informasi Artikel                                                                                       |
| --- | ------------------------------------------------------------------------------------------------------- |
| 1   | **Understanding Digital Transformation: A Review and a Research Agenda**                                |
|     | Penulis: Gregory Vial                                                                                   |
|     | DOI: 10.1016/j.jsis.2019.01.003                                                                         |
|     | Jurnal: Journal of Strategic Information Systems                                                        |
|     | Jumlah Halaman: 71                                                                                      |
|     | Tahun Publikasi: 2019                                                                                   |
| 2   | **Digital Transformation of Mental Health Services**                                                    |
|     | Penulis: Raymond R. Bond, Maurice D. Mulvenna, Courtney Potts, Siobhan O'Neill, Edel Ennis, John Torous |
|     | DOI: 10.1038/s44184-023-00033-y                                                                         |
|     | Jurnal: Communications Medicine                                                                         |
|     | Jumlah Halaman: 9                                                                                       |
|     | Tahun Publikasi: 2023                                                                                   |

### Ringkasan Dataset

| Keterangan         | Nilai                               |
| ------------------ | ----------------------------------- |
| Jumlah Dokumen PDF | 2                                   |
| Total Halaman      | 80                                  |
| Bahasa Dokumen     | Inggris                             |
| Domain Penelitian  | Digital Transformation              |
| Format Dokumen     | PDF                                 |
| Tipe Dokumen       | Artikel Jurnal Ilmiah Internasional |

### Karakteristik Dataset

Dataset yang digunakan memiliki karakteristik yang cukup kompleks karena mengandung berbagai elemen dokumen ilmiah seperti:

- Abstract
- Keywords
- Pendahuluan (Introduction)
- Metodologi Penelitian
- Pembahasan
- Kesimpulan
- Referensi
- Tabel
- Caption Gambar

Karakteristik tersebut menjadikan dataset sesuai untuk mengevaluasi kemampuan sistem PEDE dalam melakukan proses konversi PDF, ekstraksi metadata, chunking, embedding, serta penyimpanan ke dalam Qdrant Vector Database.

---

## Alur Proses

Pipeline yang digunakan pada project PEDE:

```text
PDF
↓
Markdown Conversion
↓
Metadata Extraction
↓
Chunking
↓
Embedding (all-mpnet-base-v2)
↓
Qdrant Vector Database
↓
Semantic Search / RAG
```

Setiap dokumen terlebih dahulu dikonversi menjadi Markdown menggunakan pymupdf4llm. Metadata artikel kemudian diekstraksi menggunakan DOI dan CrossRef. Setelah itu isi dokumen dipecah menjadi beberapa chunk sebelum dikonversi menjadi embedding dan disimpan ke Qdrant.

---

## Konfigurasi Benchmark

Pengujian dilakukan dengan beberapa variasi ukuran chunk dan overlap untuk melihat pengaruhnya terhadap jumlah chunk yang dihasilkan serta efisiensi proses ingest.

| Chunk Size | Overlap |
| ---------- | ------- |
| 256        | 20      |
| 512        | 50      |
| 512        | 100     |
| 1024       | 200     |

---

## Hasil Benchmark

| Chunk Size | Overlap | Paper 1 Chunk | Paper 2 Chunk | Collection Total | Waktu P1 (s) | Waktu P2 (s) |
| ---------- | ------- | ------------- | ------------- | ---------------- | ------------ | ------------ |
| 256        | 20      | 908           | 257           | 793              | 189.7        | 47.2         |
| 512        | 50      | 501           | 140           | 457              | 186.7        | 49.0         |
| 512        | 100     | 509           | 144           | 469              | 193.4        | 52.4         |
| 1024       | 200     | 255           | 75            | 241              | 201.3        | 52.0         |

---

## Analisis Hasil

Hasil benchmark menunjukkan bahwa ukuran chunk memiliki pengaruh yang sangat signifikan terhadap jumlah chunk yang dihasilkan.

Pada konfigurasi 256/20, jumlah chunk meningkat drastis hingga 793 chunk. Hal ini menunjukkan bahwa dokumen dipecah menjadi bagian yang sangat kecil sehingga informasi menjadi lebih granular. Konfigurasi ini berpotensi meningkatkan ketepatan pencarian untuk pertanyaan yang sangat spesifik, namun menghasilkan ukuran database yang lebih besar.

Pada konfigurasi 512/50, jumlah chunk turun menjadi 457 chunk. Jumlah ini masih cukup detail untuk kebutuhan semantic retrieval namun jauh lebih efisien dibanding konfigurasi 256/20.

Ketika overlap ditingkatkan menjadi 100 pada ukuran chunk yang sama (512), jumlah chunk hanya meningkat sedikit menjadi 469 chunk. Perbedaan ini menunjukkan bahwa penambahan overlap yang terlalu besar tidak memberikan manfaat signifikan namun tetap meningkatkan jumlah data yang harus diproses.

Konfigurasi 1024/200 menghasilkan 241 chunk. Jumlah chunk yang lebih sedikit membuat database lebih ringan, tetapi setiap chunk memuat konteks yang lebih besar sehingga presisi retrieval pada pertanyaan yang sangat spesifik dapat menurun.

---

## Evaluasi Model all-mpnet-base-v2

Model all-mpnet-base-v2 berhasil diintegrasikan ke dalam project PEDE tanpa perubahan besar pada pipeline yang sudah ada.

Karakteristik model:

- Arsitektur: MPNet
- Dimensi embedding: 768
- Tipe: Dense Embedding
- Fokus: Semantic Search dan Information Retrieval

Keunggulan yang diperoleh:

- Proses embedding berjalan stabil.
- Seluruh dokumen berhasil diproses tanpa kegagalan.
- Cocok untuk pencarian semantik pada dokumen ilmiah.
- Lebih ringan dibanding model embedding besar seperti BGE-M3 atau RoBERTa Large.

---

## Kesimpulan

Penggantian model embedding dari BAAI/bge-m3 menjadi sentence-transformers/all-mpnet-base-v2 berhasil dilakukan pada project PEDE.

Berdasarkan hasil benchmark, ukuran chunk yang terlalu kecil menghasilkan jumlah chunk yang sangat besar sehingga meningkatkan kebutuhan penyimpanan dan proses embedding. Sebaliknya, ukuran chunk yang terlalu besar dapat mengurangi granularitas informasi yang dibutuhkan pada proses retrieval.

Konfigurasi yang paling seimbang pada pengujian ini adalah:

- Embedding Model: sentence-transformers/all-mpnet-base-v2
- Chunk Size: 512
- Chunk Overlap: 50

Konfigurasi tersebut memberikan keseimbangan terbaik antara jumlah chunk, efisiensi penyimpanan, dan kualitas representasi dokumen untuk kebutuhan Semantic Search maupun Retrieval-Augmented Generation (RAG).
