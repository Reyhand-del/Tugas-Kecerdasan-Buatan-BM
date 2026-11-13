# Segmentasi Polip pada Citra Kolonoskopi dengan U-Net

<!--
  TEMPLATE LAPORAN — Tugas P10 Kecerdasan Buatan (BM25-41204), Teknik Biomedis ITERA.
  Petunjuk lengkap: https://mctm.web.id/ai-bm/tugas

  Cara memakai:
  - Ganti semua teks yang diawali "✍️" dengan tulisan kelompokmu, lalu hapus tanda ✍️-nya.
  - Blok komentar seperti ini adalah petunjuk; tidak tampil di GitHub. Boleh dihapus.
  - Gambar diambil dari folder hasil/ yang dibuat otomatis oleh notebook.
  - Jangan hapus judul-judul bagian (##).
-->

**Tugas P10 — Kecerdasan Buatan (BM25-41204), Teknik Biomedis ITERA**

## Anggota kelompok

| Nama | NIM | Username GitHub | Kontribusi utama |
|---|---|---|---|
| Reyhand Shevanco | 123430114 | Reyhand-del | Menjalankan eksperimen, membuat repository |
| M.Fadhil Ananta Putra | 123430085 | Fadhil085 | Menuliskan Bagian 3 |
| Falih Akbar | 123430111 | xxx | ✍️ |

## 1. Pertanyaan

Augmentasi apa yang membantu U-Net menemukan polip pada citra kolonoskopi, dan augmentasi apa yang tidak? Kenapa?

## 2. Data & pengaturan

<!-- Isi dari Bagian 2 notebook. Pastikan CEPAT = False untuk hasil akhir. -->

- Dataset: Kvasir-SEG (1000 citra kolonoskopi + mask polip), diperkecil ke 128 × 128 piksel.
- Data uji: 200 citra (tetap). Data latih: 200 citra per ulangan.
- Model: U-Net mini (±0,5 juta parameter), 40 epoch, 3 ulangan per skenario.
- Dijalankan di: Google Colab T4 

![Contoh data](hasil/contoh_data.png)

## 3. Skenario augmentasi & hipotesis

<!-- Tulis bagian ini SEBELUM menjalankan eksperimen. Commit dulu, baru jalankan.
     Alasan klinis = kenapa variasi ini mungkin (atau tidak mungkin) terjadi pada kolonoskopi sungguhan. -->

| Skenario | Augmentasi (kode) | Alasan klinis | Hipotesis: Dice naik/turun/sama? |
|---|---|---|---|
| Tanpa augmentasi | — | Pembanding | — |
| Geometri | "Geometri (Flip)": v2.Compose([v2.RandomHorizontalFlip(p=0.5),v2.RandomVerticalFlip(p=0.5),]) | Kamera endoskop berputar di dalam usus sehingga posisi polip bisa tampak terbalik tanpa mengubah bentuk aslinya | Dice naik |
| Color Jitter | "Warna Ekstrem (ColorJitter)": v2.ColorJitter( brightness=0.4, contrast=0.4 saturation=0.4, hue=0.2) | Polip dibedakan dari dinding usus berdasarkan warna kemerahannya; perubahan warna ekstrem mengaburkan ciri jaringan ini | Dice turun |

Contoh augmentasi yang dicoba:

![Contoh augmentasi](hasil/augmentasi_flip.png)
<!-- Ganti nama file sesuai skenario kalian (lihat isi folder hasil/). -->

## 4. Hasil

<!-- Tempel isi hasil/ringkasan.md di sini. -->

| Skenario | Dice (rata-rata ± sb) | IoU (rata-rata ± sb) | Dice tiap ulangan |
|---|---|---|---|
| Tanpa augmentasi | 0.584 ± 0.025 | 0.467 ± 0.027 | 0.575, 0.618, 0.559 |
| Geometri | 0.631 ± 0.046 | 0.524 ± 0.045 | 0.568, 0.675, 0.650 |
| Color Jitter | 0.484 ± 0.073 | 0.374 ± 0.053 | 0.558, 0.510, 0.385 |

![Perbandingan Dice](hasil/perbandingan.png) 

## 5. Analisis peta kesalahan

![Peta kesalahan](hasil/peta_kesalahan.png)

<!-- Hijau = benar, merah = false positive, biru = false negative.
     Pertanyaan pemandu: Polip seperti apa yang sering terlewat (kecil? datar? warna mirip dinding usus?)
     Apa yang sering dikira polip padahal bukan? Apakah augmentasi memperbaiki kasus sulit? -->

Berdasarkan peta kesalahan (error map), hasil segmentasi menunjukkan bahwa area hijau (true positive) mendominasi polip yang menonjol dengan batas tegas, sementara area biru (false negative) banyak muncul pada polip berukuran sangat kecil maupun polip datar yang warnanya menyatu dengan dinding usus. Di sisi lain, area merah (false positive) tersebar luas pada lipatan mukosa (haustral folds) dan pantulan cahaya cairan yang secara visual menyerupai polip. Penerapan augmentasi data—khususnya Color Jitter—terbukti efektif menggeser dominasi warna biru menjadi hijau pada kasus-kasus sulit tersebut, meskipun masih menyisakan sedikit area merah akibat sensitivitas model terhadap variasi tekstur jaringan normal.

## 6. Pembahasan & kesimpulan

<!-- Bandingkan hasil dengan hipotesis di Bagian 3. Hipotesis yang ternyata salah tetap bernilai,
     asalkan dijelaskan kenapa. Perbedaan kecil (lebih kecil dari simpangan baku) belum tentu berarti. -->

Perbandingan antara data kuantitatif keseluruhan dan visualisasi kualitatif menunjukkan bahwa hasil evaluasi agregat pada 200 citra uji justru membenarkan hipotesis awal sepenuhnya. Pada skenario Geometri, rata-rata skor Dice meningkat signifikan melampaui batas simpangan baku dari 0.584 ± 0.025 menjadi 0.631 ± 0.046, membuktikan efektivitas invarian rotasi endoskop. Sebaliknya, pada skenario Color Jitter, rata-rata skor Dice keseluruhan mengalami penurunan tajam menjadi 0.484 ± 0.073 dengan variabilitas tinggi antar-ulangan (0.385–0.558), yang mengonfirmasi hipotesis bahwa distorsi warna ekstrem merusak fitur klinis kemerahan polip dan memperluas false positive secara global. Adapun kenaikan skor pada peta kesalahan sebelumnya (0.09 dan 0.47) merupakan fenomena kasus lokal pada sampel spesifik berbatas kabur, bukan cerminan performa rata-rata model di seluruh dataset uji.

## 7. Pembagian kerja

<!-- Siapa mengerjakan apa. Penilaian proses kerja dilihat dari riwayat commit tiap anggota. -->

✍️

## 8. Penggunaan AI

<!-- Wajib diisi, walau tidak memakai AI (tulis "Tidak memakai AI").
     Contoh: "Claude Code (CLI) untuk menulis skenario augmentasi dan memperbaiki error di Bagian 4.
     Draf Bagian 6 ditulis sendiri, lalu dirapikan bahasanya dengan bantuan AI." -->

✍️

## Cara menjalankan ulang

```bash
uv sync
uv run jupyter lab segmentasi_polip.ipynb
```

Atau buka notebook di Google Colab (Runtime → Change runtime type → T4 GPU).

## Referensi

- Jha, D., dkk. (2020). Kvasir-SEG: A Segmented Polyp Dataset. *MultiMedia Modeling (MMM 2020)*, 451–462.
- Ronneberger, O., Fischer, P., & Brox, T. (2015). U-Net: Convolutional Networks for Biomedical Image Segmentation. *MICCAI*. arXiv:1505.04597.
- ✍️ tambahkan referensi lain yang kalian pakai
