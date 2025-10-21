# 📘 Dokumentasi beberapa istilah dalam machine learning

## 🧠 1. **Bobot (Weight)**

> **Bobot adalah nilai yang mengatur seberapa kuat pengaruh suatu fitur/input terhadap output model.**

Bayangkan kamu sedang menilai rumah:

> harga_rumah = (luas_tanah × 2 juta) + (jumlah_kamar × 10 juta) + (garasi × 5 juta)

Angka **2 juta**, **10 juta**, dan **5 juta** itulah **bobot**.

Dalam jaringan saraf (neural network):

* Bobot mengatur **hubungan antar neuron.**
* Bobot bisa **positif (meningkatkan aktivasi)** atau **negatif (mengurangi aktivasi)**.
* Saat model belajar, bobot **diubah-ubah sedikit demi sedikit** agar hasil prediksi makin mendekati kebenaran.

---

## ⚖️ 2. **Bias**

> **Bias adalah nilai tambahan yang “menggeser” hasil sebelum masuk ke fungsi aktivasi.**

Contoh sederhana:

> harga_rumah = (luas_tanah × 2 juta) + (jumlah_kamar × 10 juta) + **20 juta (bias)**

Nilai 20 juta itu tidak tergantung pada input apa pun, tapi membantu model menyesuaikan hasil prediksi agar lebih fleksibel — seperti “titik awal” atau **offset**.

Tanpa bias, semua garis regresi akan melewati titik (0,0), padahal dunia nyata jarang begitu.

---

## 🧩 3. **Gradien**

> **Gradien adalah arah dan seberapa besar perubahan yang harus dilakukan terhadap bobot dan bias agar model makin akurat.**

Secara matematis, gradien adalah **turunan (derivative)** dari fungsi loss terhadap bobot:
[
\nabla W = \frac{\partial \text{Loss}}{\partial W}
]

Tapi dalam bahasa sehari-hari:

> Gradien = “arah turun bukit paling curam”.

Bayangkan kamu berdiri di puncak bukit (tempat kesalahan model masih tinggi).
Gradien memberi tahu kamu **ke arah mana harus melangkah** agar kamu menuruni bukit (mengurangi error).
Jika kamu melangkah terlalu besar, kamu bisa “loncat ke lembah lain” — di situlah **learning rate** berperan.

---

## 🚶 4. **Learning Rate (α)**

> **Learning rate adalah ukuran langkah yang diambil model setiap kali memperbarui bobot berdasarkan gradien.**

* Kalau **terlalu besar**, kamu bisa “lewat” dari lembah optimum → model tidak stabil.
* Kalau **terlalu kecil**, kamu nyaris tidak bergerak → model lambat belajar.

Analogi:

> Kamu menuruni bukit menuju lembah paling rendah.
>
> * Langkah kecil = aman tapi lama.
> * Langkah besar = cepat tapi bisa terpeleset.

Dalam formula update bobot:
[
W_{baru} = W_{lama} - \alpha \times \nabla W
]

---

## 🔁 5. **Epoch dan Batch**

* **Epoch:** satu kali seluruh dataset digunakan untuk melatih model.
* **Batch:** potongan kecil dari dataset yang digunakan sekali update.
* **Mini-batch:** sebagian kecil data yang membuat pembelajaran lebih efisien.

Misal dataset = 1000 gambar, batch = 100 → 1 epoch = 10 batch.

---

## 🧮 6. **Loss Function**

> **Fungsi yang mengukur seberapa jauh prediksi model dari kenyataan.**

Contoh:

* Untuk regresi: Mean Squared Error (MSE)
* Untuk klasifikasi: Cross-Entropy Loss

Loss inilah “bukit” yang coba diturunkan oleh gradien.
Semakin kecil loss, semakin baik modelmu memprediksi.

---

## ⚙️ 7. **Backpropagation**

> **Proses menghitung gradien dan memperbarui bobot secara mundur dari output ke input.**

Langkah-langkah:

1. Model melakukan **forward pass** → menghasilkan prediksi.
2. Hitung **loss** antara prediksi dan label asli.
3. Lakukan **backward pass** (turunan berantai) untuk menghitung gradien tiap bobot.
4. Update bobot:
   [
   W = W - \alpha \times \text{gradien}
   ]

Proses ini diulang ribuan kali.

---

## 🌍 8. **Federated Learning (FL)**

> **Cara melatih model tanpa memindahkan data mentah dari perangkat lokal.**

Alur sederhananya:

1. Server mengirim **model global** (bobot awal) ke semua klien.
2. Tiap klien melatih modelnya pada data lokal → menghasilkan **gradien lokal**.
3. Klien mengirim **gradien (bukan datanya)** ke server.
4. Server menghitung **rata-rata (FedAvg)** dari semua gradien → memperbarui model global.
5. Ulangi siklus ini beberapa ronde.

Jadi:

* Data pengguna tetap di perangkat mereka (privasi terjaga).
* Hanya bobot atau gradien yang “keliling dunia”.

---

## ⚡ 9. **Momentum**

> **Trik untuk mempercepat konvergensi dan menghindari tersesat di lembah kecil.**

Alih-alih hanya melihat gradien terbaru, momentum juga melihat arah sebelumnya.
Ibarat bola yang meluncur di bukit, momentum membuatnya tetap bergerak walau gradien sesaat mendatar.

Formula umumnya:
[
v = \beta v + (1 - \beta)\nabla W, \quad W = W - \alpha v
]

---

## 🧭 10. **FedAvg (Federated Averaging)**

> **Strategi inti dalam federated learning yang menggabungkan bobot dari berbagai klien dengan cara dirata-rata.**

Misal ada 3 klien:

* Klien 1 punya 100 data → model A
* Klien 2 punya 200 data → model B
* Klien 3 punya 50 data → model C

Server menggabungkannya:
[
W_{global} = \frac{100W_A + 200W_B + 50W_C}{100 + 200 + 50}
]

Jadi kontribusi tiap klien sebanding dengan jumlah datanya.

---

## 🔍 Ringkasan Cepat

| Istilah                | Fungsi                                    | Analogi                                    |
| ---------------------- | ----------------------------------------- | ------------------------------------------ |
| **Bobot (Weight)**     | Menentukan seberapa besar pengaruh input  | Seperti “harga per meter”                  |
| **Bias**               | Nilai tambahan agar model lebih fleksibel | Seperti “uang muka tetap”                  |
| **Gradien**            | Arah dan besar perubahan bobot            | Arah turun bukit                           |
| **Learning Rate**      | Besarnya langkah menuruni bukit           | Ukuran langkah kaki                        |
| **Loss Function**      | Mengukur kesalahan prediksi               | Seberapa jauh dari target                  |
| **Epoch / Batch**      | Siklus pembelajaran                       | Satu kali semua data / sepotong kecil data |
| **Backpropagation**    | Cara menghitung dan memperbarui bobot     | Umpan balik ke belakang                    |
| **Momentum**           | Mempercepat konvergensi                   | Bola menuruni bukit                        |
| **Federated Learning** | Latih model tanpa kirim data              | Model keliling, bukan data                 |
| **FedAvg**             | Rata-rata bobot dari semua klien          | Gabungan pendapat dari semua peserta       |
