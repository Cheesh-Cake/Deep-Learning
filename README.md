# Oxford-IIIT Pet Image Classification Experiment

Repositori ini berisi eksperimen klasifikasi gambar menggunakan subset dari dataset **Oxford-IIIT Pet** (10 kelas pertama). Eksperimen ini membandingkan performa berbagai strategi transfer learning menggunakan arsitektur **ResNet-18** dan **EfficientNet-B0** di bawah framework PyTorch.

# Ringkasan Proyek

Tujuan utama dari proyek ini adalah menganalisis dampak dari transfer learning, teknik augmentasi data, fine-tuning progresif, dan pemilihan arsitektur modern terhadap performa akurasi dan kecepatan konvergensi model.

# 10 Kelas yang Digunakan:
`Abyssinian`, `American Bulldog`, `American Pit Bull Terrier`, `Basset Hound`, `Beagle`, `Bengal`, `Birman`, `Bombay`, `Boxer`, `British Shorthair`.

---

# Skenario Eksperimen

Berikut adalah 5 skenario eksperimen yang diuji:

1. **B1: Baseline Scratch**
   * Arsitektur: ResNet-18 (Tanpa Bobot Pretrained/ImageNet).
   * Strategi: Melatih seluruh parameter dari awal dengan learning rate $1 \times 10^{-3}$ dan menggunakan augmentasi data.
2. **B2: Feature Extraction**
   * Arsitektur: ResNet-18 (Pretrained ImageNet).
   * Strategi: Membekukan (*freeze*) seluruh lapisan ekstraktor fitur, hanya melatih bagian Fully Connected (FC) head dengan learning rate $1 \times 10^{-3}$ dan menggunakan augmentasi data.
3. **B3: Progressive Fine-Tuning**
   * Arsitektur: ResNet-18 (Pretrained ImageNet).
   * Strategi: Membuka kunci lapisan akhir (`layer4` dengan LR $1 \times 10^{-4}$) dan FC head (dengan LR $1 \times 10^{-3}$) untuk penyesuaian halus, menggunakan augmentasi data.
4. **B4_NoAug: B3 Tanpa Augmentasi**
   * Arsitektur: ResNet-18 (Pretrained ImageNet).
   * Strategi: Menggunakan konfigurasi yang sama dengan B3, tetapi hanya menggunakan transformasi dasar (Resize & CenterCrop) tanpa adanya augmentasi acak.
5. **B5: EfficientNet-B0**
   * Arsitektur: EfficientNet-B0 (Pretrained ImageNet).
   * Strategi: Membuka kunci blok fitur paling atas (`features[8]` dengan LR $1 \times 10^{-4}$) dan kepala klasifikasi (dengan LR $1 \times 10^{-3}$) menggunakan augmentasi data.

---

# Hasil Eksperimen

Berikut adalah ringkasan performa dari masing-masing kondisi eksperimen yang dievaluasi pada *validation set* dan *test set*:

| Kondisi | Deskripsi | Trainable Params | Total Params | Epoch Konvergen | Akurasi Val (%) | Test Acc (%) | Macro F1 (%) |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **B1** | Baseline Scratch | 11,181,642 | 11,181,642 | 25 | 57.29 | 54.49 | 53.13 |
| **B2** | Feature Extraction | 5,130 | 11,181,642 | 20 | 93.22 | 93.36 | 93.41 |
| **B3** | Progressive Fine-Tuning | 8,398,858 | 11,181,642 | 10 | 93.90 | 92.36 | 92.24 |
| **B4_NoAug** | B3 Tanpa Augmentasi | 8,398,858 | 11,181,642 | 6 | 92.88 | 94.02 | 93.97 |
| **B5** | EfficientNet-B0 | 424,970 | 4,020,358 | 23 | 92.54 | 91.36 | 91.26 |

# Temuan Utama:
* **Pentingnya Pretrained Weights:** Melatih model dari awal (**B1**) menghasilkan akurasi yang sangat rendah (~54% Test Acc) dibandingkan dengan model yang memanfaatkan transfer learning (akurasi >91%).
* **Efisiensi Parameter:** **B2** (hanya melatih 5,130 parameter) mampu bersaing sangat ketat dengan model fine-tuning penuh, menjadikannya pilihan paling efisien jika komputasi terbatas.
* **Dampak Augmentasi (B3 vs B4_NoAug):** Menghilangkan augmentasi pada **B4_NoAug** mempercepat waktu konvergensi (hanya 6 epoch) dengan akurasi pengujian yang sedikit lebih tinggi, namun rentan terhadap overfitting jika dilatih lebih lama.
* **Efisiensi Arsitektur:** **B5** (EfficientNet-B0) menghasilkan performa klasifikasi yang sangat kompetitif dengan total parameter jauh lebih ringkas dibanding ResNet-18.

---

# Visualisasi Kurva Loss (B1, B2, B3)

Grafik berikut menunjukkan perbandingan laju penurunan *training loss* dan *validation loss* antara skenario melatih dari awal (**B1**), ekstraksi fitur (**B2**), dan fine-tuning progresif (**B3**):

![Loss Comparison](loss_comparison_B1_B2_B3.png)

---

# Analisis Kesalahan (Failure Analysis)

Evaluasi model terbaik pada *test set* menunjukkan total **23 kesalahan klasifikasi dari 301 sampel**. Di bawah ini adalah beberapa contoh gambar di mana model gagal memprediksi kelas dengan benar beserta tingkat kepercayaan (*confidence score*)-nya:

![Failure Analysis](failure_analysis_samples.png)

Beberapa kesalahan klasifikasi terjadi karena kemiripan visual yang sangat tinggi antar ras hewan peliharaan tertentu (seperti kemiripan antara ras kucing *Abyssinian* dengan *Bengal*, atau ras anjing *American Bulldog* dengan *Boxer* / *Beagle*).
