# Translate & penjelasan paper Measuring the Effects of Non-Identical Data Distribution for Federated Visual Classification
## Translate

**Mengukur Efek Distribusi Data Non-Identik untuk Klasifikasi Visual Terfederasi**

**Penulis:**
* Tzu-Ming Harry Hsu (MIT CSAIL, stmharry@mit.edu)
* Hang Qi (Google Research, hangqi@google.com)
* Matthew Brown (Google Research, mtbr@google.com)

*(Penelitian dilakukan saat magang di Google)*
*(Preprint. Sedang ditinjau.)*

Paper: URL [https://arxiv.org/pdf/1909.06335](https://arxiv.org/pdf/1909.06335).

---

**Abstrak**

Federated Learning (Pembelajaran Terfederasi) memungkinkan model visual dilatih dengan cara yang menjaga privasi menggunakan data dunia nyata dari perangkat seluler. Mengingat sifatnya yang terdistribusi, statistik data di seluruh perangkat ini kemungkinan besar akan berbeda secara signifikan. Dalam penelitian ini, kami mengamati efek distribusi data non-identik semacam itu pada klasifikasi visual melalui Federated Learning. Kami mengusulkan cara untuk mensintesis dataset dengan rentang keidentikan (identicalness) yang berkelanjutan dan menyediakan ukuran kinerja untuk algoritma Federated Averaging. Kami menunjukkan bahwa kinerja menurun seiring semakin berbedanya distribusi, dan mengusulkan strategi mitigasi melalui momentum server. Eksperimen pada CIFAR-10 menunjukkan peningkatan kinerja klasifikasi pada berbagai tingkat non-identik, dengan akurasi klasifikasi meningkat dari 30.1% menjadi 76.9% dalam pengaturan yang paling tidak seimbang (skewed).

---

**1. Pendahuluan**

Federated Learning (FL) adalah kerangka kerja yang menjaga privasi untuk melatih model dari data pengguna terdesentralisasi yang berada di perangkat edge. Dengan algoritma Federated Averaging (FedAvg), dalam setiap putaran pembelajaran terfederasi, setiap perangkat yang berpartisipasi (juga disebut klien), menerima model awal dari server pusat, melakukan stochastic gradient descent (SGD) pada dataset lokalnya dan mengirimkan kembali gradiennya. Server kemudian mengagregasi semua gradien dari klien yang berpartisipasi dan memperbarui model awal.

Meskipun dalam pelatihan di pusat data (data-center), batch biasanya dapat diasumsikan IID (independen dan terdistribusi secara identik), asumsi ini kemungkinan tidak berlaku dalam pengaturan Federated Learning. Dalam penelitian ini, kami secara khusus mempelajari efek distribusi data non-identik pada setiap klien, dengan asumsi data diambil secara independen dari distribusi lokal yang berbeda. Kami mempertimbangkan rentang distribusi non-identik yang berkelanjutan, dan memberikan hasil empiris pada berbagai hiperparameter dan strategi optimasi.

---

**2. Karya Terkait**

Beberapa penulis telah mengeksplorasi algoritma FedAvg pada partisi data klien non-identik yang dihasilkan dari dataset klasifikasi gambar. McMahan et al. mensintesis pemisahan pengguna non-identik patologis dari dataset MNIST, mengurutkan contoh pelatihan berdasarkan label kelas dan mempartisinya menjadi shard sehingga setiap klien diberi 2 shard. Mereka menunjukkan bahwa FedAvg pada klien non-identik masih konvergen ke akurasi 99%, meskipun membutuhkan lebih banyak putaran daripada klien identik. Dengan cara pengurutan-dan-partisi yang serupa, Zhao et al. dan Sattler et al. menghasilkan partisi ekstrem pada dataset CIFAR-10, membentuk populasi yang terdiri dari total 10 klien. Pengaturan ini agak tidak realistis, karena pembelajaran terfederasi praktis biasanya melibatkan kumpulan klien yang lebih besar, dan distribusi yang lebih kompleks daripada partisi sederhana.

Penulis lain melihat distribusi data yang lebih realistis pada klien. Misalnya, Caldas et al. menggunakan Extended MNIST dengan partisi berdasarkan penulis digit, bukan hanya mempartisi berdasarkan kelas digit. Berkaitan erat dengan penelitian kami, Yurochkin et al. menggunakan distribusi Dirichlet dengan parameter konsentrasi 0.5 untuk mensintesis dataset non-identik. Kami memperluas ide ini, mengeksplorasi rentang konsentrasi α yang berkelanjutan, dengan eksplorasi rinci tentang pengaturan hiperparameter dan optimasi yang optimal.

Karya sebelumnya di sisi teoritis mempelajari konvergensi varian FedAvg dalam kondisi yang berbeda. Sahu et al. memperkenalkan istilah proksimal pada objektif klien dan membuktikan jaminan konvergensi. Li et al. menganalisis FedAvg di bawah skema sampling dan perataan (averaging) yang tepat dalam masalah yang sangat konveks (strongly convex).

---

**3. Data Klien Non-Identik Sintetis**

Dalam tugas klasifikasi visual kami, kami berasumsi pada setiap klien, contoh pelatihan diambil secara independen dengan label kelas mengikuti distribusi kategorikal atas N kelas yang diparameterisasi oleh vektor q ($q_{i}\ge0,i\in[1,N]$ dan $||q||_{1}=1$). Untuk mensintesis populasi klien non-identik, kami mengambil $q\sim Dir(\alpha p)$ dari distribusi Dirichlet, di mana p mencirikan distribusi kelas prior atas N kelas, dan $\alpha>0$ adalah parameter konsentrasi yang mengontrol keidentikan antar klien. Kami bereksperimen dengan 8 nilai untuk α untuk menghasilkan populasi yang mencakup spektrum keidentikan. Dengan $\alpha\rightarrow\infty$, semua klien memiliki distribusi yang identik dengan prior; dengan $\alpha\rightarrow0$ pada ekstrem lainnya, setiap klien hanya menyimpan contoh dari satu kelas yang dipilih secara acak.

Dalam penelitian ini, kami menggunakan dataset klasifikasi gambar CIFAR-10, yang berisi 60.000 gambar (50.000 untuk pelatihan, 10.000 untuk pengujian) dari 10 kelas. Kami menghasilkan populasi seimbang yang terdiri dari 100 klien, masing-masing menyimpan 500 gambar. Kami menetapkan distribusi prior menjadi seragam di 10 kelas, identik dengan set pengujian tempat kami melaporkan kinerja. Untuk setiap klien, dengan α tertentu, kami mengambil sampel q dan menetapkan jumlah gambar yang sesuai dari 10 kelas kepada klien tersebut. Gambar 1 mengilustrasikan populasi yang diambil dari distribusi Dirichlet dengan parameter konsentrasi yang berbeda.

* **(Gambar 1 tidak dapat ditampilkan di sini, namun keterangannya adalah sebagai berikut):**
  **Gambar 1: Populasi sintetis dengan klien non-identik.** Distribusi antar kelas direpresentasikan dengan warna berbeda.
  * (a) Urut-dan-partisi: 10 klien yang dihasilkan dari skema urut-dan-partisi, masing-masing diberi 2 kelas.
  * (b) Dirichlet, α → ∞: populasi yang dihasilkan dari Distribusi Dirichlet.
  * (c) Dirichlet, α = 100.0: populasi yang dihasilkan dari Distribusi Dirichlet.
  * (d) Dirichlet, α = 1.0: populasi yang dihasilkan dari Distribusi Dirichlet.
  * (e) Dirichlet, α → 0.0: populasi yang dihasilkan dari Distribusi Dirichlet, masing-masing 30 klien acak untuk (b-e).
  * Label sumbu horizontal: Distribusi kelas. Label sumbu vertikal: Klien.

---

**4. Eksperimen dan Hasil**

Dengan persiapan dataset di atas, kami sekarang melanjutkan untuk mengukur kinerja algoritma FedAvg standar pada berbagai distribusi mulai dari identik hingga non-identik. Kami menggunakan arsitektur CNN dan notasi yang sama seperti pada McMahan et al. kecuali bahwa weight decay 0.004 digunakan dan tidak ada jadwal penurunan learning rate (learning rate decay) yang diterapkan. Model ini bukanlah yang tercanggih (state-of-the-art) pada dataset CIFAR-10, tetapi cukup untuk menunjukkan kinerja relatif untuk tujuan investigasi kami.

FedAvg dijalankan dengan ukuran batch klien $B=64$, jumlah epoch lokal $E\in\{1,5\}$, dan fraksi pelaporan $C\in\{0.05,0.1,0.2,0.4\}$ (sesuai dengan 5, 10, 20, dan 40 klien yang berpartisipasi dalam setiap putaran tunggal) untuk total 10.000 putaran komunikasi. Kami melakukan pencarian hiperparameter pada grid learning rate klien $\eta\in\{10^{-4},3\times10^{-4},...,10^{-1},3\times10^{-1}\}$.

**4.1 Kinerja Klasifikasi dengan Distribusi Non-Identik**

Gambar 2 menunjukkan kinerja klasifikasi sebagai fungsi dari parameter konsentrasi Dirichlet α (α yang lebih besar menyiratkan distribusi yang lebih identik). Perubahan signifikan dalam akurasi uji terjadi di sekitar α rendah ketika klien mendekati hanya memiliki satu kelas. Meningkatkan fraksi pelaporan C menghasilkan keuntungan yang semakin berkurang (diminishing returns), dan peningkatan kinerja sangat marjinal untuk dataset klien yang terdistribusi secara identik. Menariknya, untuk kasus anggaran putaran optimasi tetap, menyinkronkan bobot lebih sering $(E=1)$ tidak selalu meningkatkan akurasi pada data non-identik.

Selain penurunan akurasi akhir pelatihan, kami juga mengamati error pelatihan yang lebih fluktuatif dalam kasus data yang lebih non-identik, lihat Gambar 3. Proses pelatihan dengan fraksi pelaporan kecil kesulitan untuk konvergen dalam 10.000 putaran komunikasi.

* **(Gambar 2 tidak dapat ditampilkan di sini, namun keterangannya adalah sebagai berikut):**
  **Gambar 2: Akurasi FedAvg untuk α yang berbeda.** Setiap sel dioptimalkan berdasarkan learning rate, dengan setiap learning rate dirata-ratakan dari 5 kali proses pada populasi yang berbeda di bawah α yang sama.
  * (a) Epoch Lokal E = 1. Nilai akurasi tertera dalam heatmap.
  * (b) Epoch Lokal E = 5. Nilai akurasi tertera dalam heatmap.
  * Sumbu Y: Fraksi Pelaporan C. Sumbu X: α. Label: Akurasi Uji.

* **(Gambar 3 tidak dapat ditampilkan di sini, namun keterangannya adalah sebagai berikut):**
  **Gambar 3: Kurva pembelajaran FedAvg dengan learning rate tetap.** Hasil pembelajaran terpusat (garis putus-putus) berasal dari tutorial TensorFlow. Kurva menunjukkan Akurasi Uji vs Putaran Komunikasi untuk berbagai kombinasi α, E, dan C. Sumbu X berkisar 0-10000.

* **(Gambar 4 tidak dapat ditampilkan di sini, namun keterangannya adalah sebagai berikut):**
  **Gambar 4: Akurasi uji FedAvg dalam pencarian hiperparameter.** (a-b) Fraksi pelaporan tinggi dan (c-d) rendah dari 100 klien didemonstrasikan. Akurasi acak (chance accuracy) ditunjukkan oleh garis putus-putus. Grafik menunjukkan Akurasi Uji vs Learning Rate untuk berbagai nilai α.
  * (a) Epoch Lokal E = 1, Fraksi Pelaporan C = 0.40.
  * (b) Epoch Lokal E = 5, Fraksi Pelaporan C = 0.40.
  * (c) Epoch Lokal E = 1, Fraksi Pelaporan C = 0.05.
  * (d) Epoch Lokal E = 5, Fraksi Pelaporan C = 0.05.
  * Sumbu X (Learning Rate) berkisar 10⁻⁴ hingga 10⁻¹.

**Sensitivitas Hiperparameter.** Selain mempengaruhi akurasi keseluruhan pada set pengujian, kondisi pembelajaran seperti yang ditentukan oleh C dan α memiliki efek signifikan pada sensitivitas hiperparameter. Pada ujung identik dengan α besar, rentang learning rate (sekitar dua orde magnitudo) dapat menghasilkan akurasi yang baik pada set pengujian. Namun, dengan nilai C dan α yang lebih kecil, penyesuaian (tuning) learning rate yang cermat diperlukan untuk mencapai akurasi yang baik. Lihat Gambar 4.

**4.2 Mengakumulasi Pembaruan Model dengan Momentum**

Menggunakan momentum di atas SGD telah terbukti sangat sukses dalam mempercepat pelatihan jaringan dengan akumulasi riwayat gradien untuk meredam osilasi. Hal ini tampaknya sangat relevan untuk FL di mana pihak yang berpartisipasi mungkin memiliki distribusi data yang jarang, dan memegang subset label yang terbatas. Dalam sub-bagian ini kami menguji efek momentum di server pada kinerja FedAvg.

FedAvg standar memperbarui bobot melalui $w \leftarrow w - \Delta w$, di mana $\Delta w = \sum_{k=1}^{K} \frac{n_k}{n} \Delta w_k$ ($n_k$ adalah jumlah contoh, $\Delta w_k$ adalah pembaruan bobot dari klien ke-k, dan $n = \sum_{k=1}^{K} n_k$). Untuk menambahkan momentum di server, kami malah menghitung $v \leftarrow \beta v + \Delta w$, dan memperbarui model dengan $w \leftarrow w - v$. Kami menyebut pendekatan ini FedAvgM (Federated Averaging with Server Momentum).

Dalam eksperimen, kami menggunakan Nesterov accelerated gradient dengan momentum $\beta \in \{0, 0.7, 0.9, 0.97, 0.99, 0.997\}$. Arsitektur model, ukuran batch klien B, dan learning rate η sama seperti FedAvg standar di sub-bagian sebelumnya. Learning rate dari optimizer server dijaga konstan pada 1.0.

**Efek momentum server.** Gambar 5 menunjukkan efek pembelajaran dengan data non-identik baik dengan maupun tanpa momentum server. Akurasi uji meningkat secara konsisten untuk FedAvgM dibandingkan FedAvg, dengan kinerja mendekati baseline pembelajaran terpusat (86.0%) dalam banyak kasus. Sebagai contoh, dengan $E=1$ dan $C=0.05$, kinerja FedAvgM tetap relatif konstan dan di atas 75%, sedangkan akurasi FedAvg turun drastis menjadi sekitar 35%.

* **(Gambar 5 tidak dapat ditampilkan di sini, namun keterangannya adalah sebagai berikut):**
  **Gambar 5: Kurva kinerja FedAvgM dan FedAvg untuk tingkat non-identik yang berbeda.** Data semakin non-identik ke arah kanan. Paling baik dilihat dalam warna. Grafik menunjukkan Akurasi Uji vs α untuk berbagai kombinasi E dan C, membandingkan FedAvg dan FedAvgM.
  * (a) Epoch Lokal E = 1.
  * (b) Epoch Lokal E = 5.
  * Sumbu X (α) berkisar 10⁻² hingga 10².

**Ketergantungan hiperparameter pada C dan E.** Penyesuaian hiperparameter lebih sulit untuk FedAvgM karena melibatkan hiperparameter tambahan β. Dalam Gambar 6, kami memplot akurasi terhadap learning rate efektif yang didefinisikan sebagai $\eta_{eff} = \eta / (1 - \beta)$ yang menunjukkan $\eta_{eff}$ optimal untuk setiap set kondisi pembelajaran. Terutama, ketika fraksi pelaporan C besar, pemilihan β lebih mudah dan rentang nilai dalam dua orde magnitudo menghasilkan akurasi uji yang wajar. Sebaliknya, ketika hanya beberapa klien yang melapor setiap putaran, jendela yang layak untuk $\eta_{eff}$ bisa sekecil hanya satu orde magnitudo. Untuk mencegah pembaruan klien menyimpang (diverging), kami juga harus menggunakan kombinasi learning rate absolut rendah dan momentum tinggi. Parameter epoch lokal E juga mempengaruhi pilihan learning rate. Optimasi lokal yang ekstensif meningkatkan varians pembaruan bobot klien, oleh karena itu $\eta_{eff}$ yang lebih rendah diperlukan untuk menetralkan noise.

* **(Gambar 6 tidak dapat ditampilkan di sini, namun keterangannya adalah sebagai berikut):**
  **Gambar 6: Sensitivitas akurasi uji untuk FedAvgM.** Diplot untuk $\alpha=1$. Learning rate efektif didefinisikan sebagai $\eta_{eff} = \eta / (1 - \beta)$. Ukuran proporsional dengan learning rate klien η dan titik paling berkinerja ditandai dengan crosshair. Grafik menunjukkan Akurasi Uji vs Learning Rate Efektif ($\eta_{eff}$) dengan ukuran dan warna titik yang berbeda merepresentasikan Learning Rate absolut (η).
  * (a) Epoch Lokal E = 1, Fraksi Pelaporan C = 0.40.
  * (b) Epoch Lokal E = 5, Fraksi Pelaporan C = 0.40.
  * (c) Epoch Lokal E = 1, Fraksi Pelaporan C = 0.05.
  * (d) Epoch Lokal E = 5, Fraksi Pelaporan C = 0.05.
  * Sumbu X (Learning Rate Efektif) berkisar 10⁻⁴ hingga 10¹.

---

**Referensi**

* Sebastian Caldas, Peter Wu, Tian Li, Jakub Konečnỳ, H Brendan McMahan, Virginia Smith, dan Ameet Talwalkar. Leaf: Sebuah benchmark untuk pengaturan terfederasi. preprint arXiv:1812.01097, 2018.
* Gregory Cohen, Saeed Afshar, Jonathan Tapson, dan André van Schaik. EMNIST: ekstensi MNIST untuk huruf tulisan tangan. preprint arXiv:1702.05373, 2017.
* Alex Krizhevsky, Geoffrey Hinton, et al. Mempelajari beberapa lapisan fitur dari gambar kecil. Laporan teknis, Citeseer, 2009.
* Xiang Li, Kaixuan Huang, Wenhao Yang, Shusen Wang, dan Zhihua Zhang. Tentang konvergensi FedAvg pada data non-IID. preprint arXiv:1907.02189, 2019.
* Brendan McMahan, Eider Moore, Daniel Ramage, Seth Hampson, dan Blaise Aguera y Arcas. Pembelajaran jaringan dalam yang efisien komunikasi dari data terdesentralisasi. Dalam Artificial Intelligence and Statistics, halaman 1273-1282, 2017.
* Yu Nesterov. Metode gradien untuk meminimalkan fungsi objektif komposit. 2007.
* Anit Kumar Sahu, Tian Li, Maziar Sanjabi, Manzil Zaheer, Ameet Talwalkar, dan Virginia Smith. Tentang konvergensi optimasi terfederasi dalam jaringan heterogen. preprint arXiv:1812.06127, 2018.
* Felix Sattler, Simon Wiedemann, Klaus-Robert Müller, dan Wojciech Samek. Pembelajaran terfederasi yang kuat dan efisien komunikasi dari data non-IID. preprint arXiv:1903.02891, 2019.
* Christopher J Shallue, Jaehoon Lee, Joe Antognini, Jascha Sohl-Dickstein, Roy Frostig, dan George E Dahl. Mengukur efek paralelisme data pada pelatihan jaringan saraf. preprint arXiv:1811.03600, 2018.
* TensorFlow. Jaringan saraf konvolusional tingkat lanjut. URL [https://www.tensorflow.org/tutorials/images/deep_cnn](https://www.tensorflow.org/tutorials/images/deep_cnn).
* Mikhail Yurochkin, Mayank Agarwal, Soumya Ghosh, Kristjan Greenewald, Nghia Hoang, dan Yasaman Khazaeni. Pembelajaran terfederasi nonparametrik Bayesian dari jaringan saraf. Dalam International Conference on Machine Learning, halaman 7252-7261, 2019.
* Yue Zhao, Meng Li, Liangzhen Lai, Naveen Suda, Damon Civin, dan Vikas Chandra. Pembelajaran terfederasi dengan data non-IID. preprint arXiv:1806.00582, 2018.

## PENJELASAN 1:

## **Apa Sih Masalah Utamanya? Belajar Bareng Tapi Data Beda-beda**

Jurnal ini membahas tantangan dalam **Federated Learning (FL)**, khususnya untuk tugas **klasifikasi visual** (mengenali gambar).

* **Apa itu Federated Learning (FL)?** 🤔
  Bayangkan kita mau melatih model AI (misalnya, untuk kenal jenis bunga dari foto) pakai data dari banyak *handphone* pengguna. Tapi, kita *gak mau* data pribadi pengguna (foto-foto mereka) dikirim ke server pusat karena masalah privasi. Nah, FL ini solusinya! Caranya:

  1.  Server pusat kirim model AI awal ke *handphone* pengguna (disebut **klien**).
  2.  Setiap *handphone* melatih model itu pakai data lokalnya sendiri.
  3.  *Handphone* *gak* kirim data mentah, tapi cuma kirim "pembaruan" atau "pelajaran" (gradien/perubahan bobot model) yang didapat dari data lokalnya ke server.
  4.  Server pusat menggabungkan semua "pelajaran" dari banyak *handphone* untuk memperbarui model utama. Proses ini diulang-ulang. Algoritma umum untuk ini namanya **Federated Averaging (FedAvg)**.

* **Masalahnya di mana?** 😥
  Kalau kita latih AI di pusat data, kita bisa atur datanya biar **IID (Independent and Identically Distributed)**, artinya data di setiap *batch* (kelompok data kecil untuk sekali latihan) itu mirip-mirip distribusinya (misalnya, jumlah gambar tiap kelas bunga seimbang). Tapi di FL, data di tiap *handphone* pengguna itu **sangat mungkin BEDA-BEDA** atau **Non-IID**. Contohnya:

  * Pengguna A suka foto mawar, jadi datanya kebanyakan mawar.
  * Pengguna B tinggal di daerah tropis, fotonya banyak anggrek.
  * Pengguna C cuma punya sedikit foto bunga.
  Distribusi data yang *gak identik* antar klien ini bisa bikin proses belajar AI jadi kacau dan performanya jelek.

-----

## **Tujuan Penelitian Ini: Ngukur Efek & Cari Solusi**

Penelitian ini punya dua tujuan utama:

1.  **Mengukur seberapa parah sih efek data Non-IID ini?** Mereka ingin tahu seberapa jauh penurunan performa model klasifikasi gambar kalau data antar klien semakin berbeda.
2.  **Mencari cara untuk mengurangi efek negatifnya.** Mereka mengusulkan dan menguji sebuah strategi untuk bikin FedAvg lebih tahan banting terhadap data Non-IID.

-----

## **Mensimulasikan Data Beda-beda Pakai Apa? Distribusi Dirichlet!**

Untuk bisa mengukur efek data Non-IID secara sistematis, peneliti *gak* bisa langsung pakai data asli pengguna (karena privasi tadi). Jadi, mereka bikin simulasi data Non-IID pakai dataset gambar standar, yaitu **CIFAR-10** (dataset berisi 60.000 gambar kecil dalam 10 kelas, seperti pesawat, mobil, burung, dll.).

Mereka membagi data CIFAR-10 ini ke 100 klien simulasi, masing-masing dapat 500 gambar. Nah, cara baginya ini yang penting, biar bisa dikontrol tingkat "beda-beda"-nya (non-identicalness):

* **Pakai Distribusi Dirichlet:** Bayangkan kita punya 10 kantong (mewakili 10 kelas gambar di CIFAR-10). Distribusi Dirichlet ini kayak cara "acak" untuk menentukan proporsi kelereng dari tiap kantong yang akan dimasukkan ke setiap tas klien.
  * Setiap klien `k` akan punya vektor proporsi kelas `qk` (misalnya, klien 1 punya 70% gambar kucing, 20% anjing, 10% mobil, dst.). Total proporsi harus 1 (atau 100%).
  * Proporsi `qk` ini diambil (di-*sample*) dari **Distribusi Dirichlet**, ditulis $q \sim Dir(\alpha p)$.
    * `p` itu **distribusi prior**, anggapan awal kita tentang sebaran kelas. Di sini, mereka pakai `p` yang **seragam** (uniform), artinya awalnya dianggap setiap kelas punya kemungkinan sama rata (10% untuk masing-masing dari 10 kelas). Ini penting karena data ujinya juga distribusinya seragam.
    * `α` (alpha) itu **parameter konsentrasi**. Ini yang jadi "tombol" pengatur tingkat keidentikan data antar klien.
      * Kalau `α` **besar banget** (mendekati tak hingga, $\alpha \rightarrow \infty$), semua klien akan punya distribusi kelas yang **mirip banget** dengan `p` (jadi hampir IID). Kayak kita kasih instruksi super ketat pas bagi kelereng.
      * Kalau `α` **kecil banget** (mendekati nol, $\alpha \rightarrow 0$), setiap klien kemungkinan besar **hanya akan punya data dari SATU kelas saja** yang dipilih acak. Ini kondisi Non-IID paling ekstrem. Kayak kita bagi kelerengnya asal banget, satu tas bisa cuma isi kelereng merah semua.
* **Spektrum Keidentikan:** Dengan mengubah-ubah nilai `α`, peneliti bisa menciptakan berbagai skenario, dari data yang hampir sama semua antar klien sampai data yang sangat berbeda drastis. Mereka mencoba 8 nilai `α` yang berbeda.

**(Lihat Gambar 1 di jurnal):** Gambar ini visualisasi bagus banget. Kolom (a) itu cara lama (urut-dan-partisi), tiap klien cuma dapat 2 kelas. Kolom (b) sampai (e) pakai Dirichlet dengan `α` dari besar ke kecil. Kelihatan jelas kan, makin kecil `α` (ke arah kanan), warna di tiap baris (klien) makin homogen (cuma satu atau sedikit warna), artinya datanya makin Non-IID.

-----

## **Eksperimen Pertama: Seberapa Parah Sih Kalau Pakai FedAvg Biasa?**

Peneliti lalu melatih model **Convolutional Neural Network (CNN)** standar (arsitektur mirip yang dipakai McMahan et al., 2017) pakai FedAvg biasa di atas data simulasi tadi.

* **Pengaturan Latihan (Hyperparameter):**

  * **Batch Size (B):** Ukuran kelompok data kecil yang dipakai klien untuk sekali update SGD di *handphone*-nya. Di sini $B=64$.
  * **Local Epochs (E):** Berapa kali klien melatih model pakai *seluruh* data lokalnya sebelum kirim pembaruan ke server. Mereka coba $E=1$ (latih sebentar) dan $E=5$ (latih lebih lama).
  * **Reporting Fraction (C):** Berapa persen klien yang ikut serta di setiap putaran komunikasi. Misal $C=0.1$ artinya dari 100 klien, cuma 10 yang dipilih acak untuk berkontribusi di putaran itu. Mereka coba $C=0.05, 0.1, 0.2, 0.4$.
  * **Client Learning Rate (η):** Seberapa besar "langkah" pembaruan yang dilakukan klien saat SGD. Ini dicari nilai terbaiknya dari beberapa pilihan.
  * **Weight Decay:** Teknik regulariasi untuk mencegah model jadi terlalu kompleks (overfitting) dengan memberi "hukuman" kecil pada bobot model yang besar. Nilainya 0.004. Learning rate *tidak* diturunkan selama pelatihan.
  * Total **10.000 putaran komunikasi**.

* **Hasilnya Gimana? (Lihat Gambar 2, 3, 4)**

  * **Makin Non-IID (α kecil), Makin Jelek:** Akurasi model di data uji (data yang *gak pernah* dilihat pas latihan) **turun drastis** saat `α` makin kecil, terutama saat mendekati 0. Artinya, FedAvg biasa kesulitan belajar kalau data klien beda-beda banget.
  * **Nambah Klien (C besar) Gak Selalu Efektif:** Menaikkan jumlah klien yang ikut (menaikkan C) memang sedikit membantu, tapi efeknya makin kecil, apalagi kalau datanya udah lumayan identik (α besar).
  * **Lama Latih di Klien (E) Gak Nentu:** Anehnya, melatih lebih lama di klien ($E=5$) *gak selalu* lebih baik daripada latih sebentar ($E=1$) saat datanya Non-IID, padahal intuisinya mungkin sebaliknya.
  * **Latihan Jadi Gak Stabil:** Kalau data makin Non-IID (α kecil) dan klien yang ikut sedikit (C kecil), kurva belajarnya jadi **naik-turun gak jelas (volatile)** dan **susah konvergen** (susah mencapai performa terbaiknya) bahkan setelah 10.000 putaran. (Lihat Gambar 3, kurva hijau dan abu-abu putus-putus).
  * **Sensitif Sama Learning Rate:** Kalau datanya identik (α besar), model bisa tetap bagus meskipun learning rate (η) diubah-ubah dalam rentang yang lumayan lebar (sekitar 2 orde **magnitudo** - artinya faktor 100x). Tapi kalau datanya Non-IID (α kecil) dan klien yang ikut sedikit (C kecil), kita harus **hati-hati banget** pilih learning rate yang pas. Salah sedikit, performanya bisa anjlok. (Lihat Gambar 4, kurva untuk α=0.00 bentuknya lebih "tajam").

-----

## **Solusinya: Tambahin "Momentum" di Server (FedAvgM)**

Karena FedAvg biasa kewalahan, peneliti coba modifikasi: **Federated Averaging with Server Momentum (FedAvgM)**.

* **Apa itu Momentum?** 🏃💨
  Momentum di optimasi itu mirip konsep momentum di fisika. Kalau kita dorong bola, dia gak langsung berhenti kan? Dia punya "momentum" yang bikin dia terus gerak. Dalam SGD, momentum membantu "mengingat" arah pembaruan sebelumnya.

  * **Manfaatnya:**
    1.  **Mempercepat konvergensi:** Bisa bantu model belajar lebih cepat.
    2.  **Meredam Osilasi:** Kalau pembaruan dari klien arahnya gonta-ganti terus (karena data mereka beda-beda), momentum bisa bantu "meratakan" arahnya biar lebih stabil. Ini *cocok banget* buat kasus data Non-IID di FL!

* **Gimana Cara Kerjanya di FedAvgM?**

  * FedAvg Biasa: Server langsung update model `w` pakai rata-rata pembaruan klien $\Delta w$: $w \leftarrow w - \Delta w$.
  * FedAvgM: Server punya "variabel kecepatan" `v`. Setiap putaran:
    1.  Hitung rata-rata pembaruan klien $\Delta w$ (sama kayak FedAvg).
    2.  Update "kecepatan": $v \leftarrow \beta v + \Delta w$. Di sini `β` (beta) itu **faktor momentum** (biasanya antara 0 dan 1, seringnya dekat 1 kayak 0.9). Ini artinya kecepatan baru adalah sebagian kecil dari kecepatan lama (`βv`) ditambah pembaruan sekarang ($\Delta w$). Jadi dia "ingat" arah sebelumnya.
    3.  Update model pakai "kecepatan": $w \leftarrow w - v$.
  * Mereka pakai varian momentum namanya **Nesterov Accelerated Gradient (NAG)**, yang sedikit lebih canggih tapi intinya sama. Mereka coba berbagai nilai `β`. Learning rate *server* (bukan klien) dijaga tetap 1.0.

* **Hasilnya Gimana? (Lihat Gambar 5)**

  * **Jauh Lebih Baik!** 🎉 FedAvgM secara konsisten **mengalahkan** FedAvg biasa di hampir semua skenario data Non-IID (berbagai nilai α).
  * **Lebih Tahan Banting:** Performanya *gak* turun drastis meskipun data sangat Non-IID (α kecil). Contohnya, saat $E=1, C=0.05$ (kondisi lumayan sulit), FedAvgM tetap bisa capai akurasi di atas 75%, sementara FedAvg anjlok ke sekitar 35%. Keren kan!
  * **Mendekati Terpusat:** Di banyak kasus, akurasi FedAvgM bahkan mendekati hasil kalau model dilatih secara terpusat (data dikumpulin semua, akurasi sekitar 86%).

-----

## **Tapi... Nyetel FedAvgM Agak Ribet: Peran Learning Rate Efektif**

Meskipun hasilnya bagus, FedAvgM punya satu PR lagi: nambahin parameter `β` bikin nyari kombinasi hiperparameter terbaik jadi lebih susah.

* **Learning Rate Efektif (η\<sub\>eff\</sub\>):** Peneliti pakai konsep **learning rate efektif**, $\eta_{eff} = \eta / (1 - \beta)$. Kenapa? Ini kayak gabungan efek learning rate klien (η) dan momentum server (β) jadi satu angka. Teori dan eksperimen [Shallue et al., 2018] nunjukin kalau seringkali ada nilai $\eta_{eff}$ "optimal" untuk suatu masalah, meskipun kombinasi η dan β nya bisa beda-beda.
* **Hubungan Antar Parameter (Lihat Gambar 6):**
  * **Kalau Klien Banyak (C besar):** Lebih gampang nyari η dan β yang pas. Ada rentang $\eta_{eff}$ yang lumayan lebar (sekitar 2 orde magnitudo) yang bisa kasih hasil bagus.
  * **Kalau Klien Sedikit (C kecil):** Lebih susah! Rentang $\eta_{eff}$ yang bagus jadi **sempit banget** (cuma sekitar 1 orde magnitudo). Kita harus pakai learning rate klien (η) yang **kecil** *dan* momentum (β) yang **tinggi** (dekat 1) biar pembaruan dari sedikit klien yang datanya mungkin aneh itu *gak* langsung merusak model global, tapi pelan-pelan diakumulasi lewat momentum.
  * **Kalau Latih Lama di Klien (E besar):** Karena klien ngelakuin optimasi lebih banyak secara lokal, pembaruan ($\Delta w_k$) yang mereka kirim bisa jadi lebih "beragam" atau punya varians lebih tinggi. Untuk menstabilkan ini, biasanya butuh $\eta_{eff}$ yang **lebih rendah**.

**(Lihat Gambar 6):** Gambar ini nunjukkin akurasi vs $\eta_{eff}$. Ukuran titik nunjukkin nilai η absolut, warnanya beda-beda. Kelihatan kan polanya? Ada "puncak" performa di sekitar $\eta_{eff}$ tertentu. Kalau C kecil (gambar c, d), puncaknya lebih sempit dan titik-titik terbaik cenderung punya ukuran kecil (η kecil) dan warna terang (β tinggi mendekati 1).

-----

## **Kesimpulan Singkat**

Jadi, intinya jurnal ini nunjukkin:

1.  **Data Non-IID itu masalah serius** di Federated Learning, bisa bikin performa model anjlok dan latihannya gak stabil, apalagi kalau cuma sedikit klien yang aktif per putaran.
2.  **Mensimulasikan Non-IID pakai Distribusi Dirichlet dengan parameter α** itu cara yang bagus untuk mempelajari efek ini secara sistematis.
3.  **Menambahkan Momentum di Server (FedAvgM)** adalah solusi yang efektif banget untuk melawan efek negatif data Non-IID, bikin performa lebih stabil dan jauh lebih baik, bahkan mendekati performa latihan terpusat. 💪
4.  Tapi, **penyetelan hiperparameter** (khususnya learning rate dan momentum) jadi lebih krusial di FedAvgM, terutama kalau jumlah klien yang berpartisipasi sedikit. Konsep *learning rate efektif* bisa membantu proses ini.

## PENJELASAN 2:

## **Bagaimana Klien Belajar? SGD Jawabannya**

Sebelum ke rumus server, kita pahami dulu apa yang terjadi di *handphone* klien. Setiap klien yang terpilih di satu putaran akan melatih model yang diterima dari server pakai data lokalnya. Proses latihan ini biasanya pakai **Stochastic Gradient Descent (SGD)** atau variannya.

* **SGD itu apa?** 🧐
  SGD itu cara umum untuk melatih model AI. Bayangkan model AI itu kayak orang buta yang lagi di gunung (permukaan *error*) dan mau cari jalan turun ke lembah (titik *error* terendah).
  1.  Dia ambil *satu langkah kecil* (ini diatur sama **learning rate**, η).
  2.  Arah langkahnya ditentukan oleh *kemiringan* (gradien) di tempat dia berdiri sekarang, tapi dihitung hanya dari *sebagian kecil data* (**stochastic** / acak, yaitu **batch** data). Dia melangkah ke arah *turunan paling curam*.
  3.  Dia ulang terus langkah ini sampai nemu lembah (atau cukup dekat).
  Di FL, setiap klien melakukan SGD ini beberapa kali (sebanyak **Local Epochs**, E) pakai data lokalnya. Hasil akhirnya adalah "arah" atau "pembaruan" (disebut **gradien** atau **perubahan bobot**, $\Delta w_k$) yang menurut klien tersebut paling bagus untuk menurunkan *error* berdasarkan data lokalnya.

-----

## **Rumus Update Server FedAvg (Yang Biasa)**

Di akhir latihan lokal, klien terpilih akan kirim pembaruan bobot ($\Delta w_k$) ke server. Server lalu menggabungkan semua pembaruan ini untuk update model global `w`. Rumusnya begini:

$\Delta w = \sum_{k=1}^{K} \frac{n_k}{n} \Delta w_k$

* **Apa artinya?** 🤔
  * `w`: Bobot model AI global saat ini (misalnya, angka-angka parameter di dalam model CNN).
  * `K`: Jumlah klien yang berpartisipasi di putaran ini (misalnya, 10 klien).
  * `k`: Indeks untuk klien ke-k (dari 1 sampai K).
  * $\Delta w_k$: Pembaruan bobot yang dihitung dan dikirim oleh klien ke-k. Ini adalah hasil SGD lokal di *handphone* klien k.
  * $n_k$: Jumlah data (contoh gambar) yang dimiliki klien ke-k yang dipakai untuk latihan.
  * $n$: Total jumlah data dari *semua* klien yang berpartisipasi di putaran itu ($n = \sum_{k=1}^{K} n_k$).
  * $\frac{n_k}{n}$: Ini adalah **bobot rata-rata** (weighted average). Artinya, pembaruan dari klien yang punya data lebih banyak ($n_k$ besar) akan punya pengaruh **lebih besar** dalam menentukan pembaruan global $\Delta w$. Masuk akal kan? Klien yang "belajar" dari lebih banyak data, pendapatnya lebih didengar.
  * $\sum_{k=1}^{K}$: Simbol sigma ini artinya "jumlahkan semua" dari klien k=1 sampai K.
  * $\Delta w$: Ini adalah hasil **rata-rata terbobot** dari semua pembaruan klien. Inilah "pembaruan global" yang akan dipakai server.

Setelah $\Delta w$ dihitung, server update model globalnya:

$w \leftarrow w - \Delta w$

* **Apa artinya?** 🤔

  * Tanda panah ($\leftarrow$) artinya "diperbarui menjadi".
  * Bobot model global yang baru (`w` baru) adalah bobot lama (`w` lama) **dikurangi** pembaruan global ($\Delta w$). Kenapa dikurangi? Karena SGD (dan $\Delta w_k$) biasanya menghitung arah *kenaikan* error, jadi kita bergerak ke arah *sebaliknya* untuk *menurunkan* error.

* **Contoh Sederhana FedAvg:**
  Anggap model kita cuma punya 1 bobot `w`, dan nilainya sekarang `w = 10`. Ada 2 klien ikut putaran ini:

  * Klien 1 punya data $n_1 = 60$, hasil latihannya $\Delta w_1 = +0.5$ (menurut klien 1, bobot perlu ditambah 0.5).
  * Klien 2 punya data $n_2 = 40$, hasil latihannya $\Delta w_2 = -0.1$ (menurut klien 2, bobot perlu dikurangi 0.1).
  Total data $n = n_1 + n_2 = 60 + 40 = 100$.
  Hitung pembaruan global $\Delta w$:
  $\Delta w = \frac{n_1}{n} \Delta w_1 + \frac{n_2}{n} \Delta w_2 = \frac{60}{100}(+0.5) + \frac{40}{100}(-0.1)$
  $\Delta w = (0.6)(0.5) + (0.4)(-0.1) = 0.3 - 0.04 = +0.26$
  Server update model global:
  $w \leftarrow w - \Delta w = 10 - (+0.26) = 9.74$
  Jadi, bobot model global baru adalah 9.74.

-----

## **Rumus Update Server FedAvgM (Dengan Momentum)**

FedAvgM menambahkan konsep "momentum" di sisi server untuk membuat update lebih stabil, terutama saat data klien Non-IID.

Rumusnya jadi dua langkah:

1.  **Update "Kecepatan" (Velocity):**
  $v \leftarrow \beta v + \Delta w$
2.  **Update Bobot Model:**
  $w \leftarrow w - v$

* **Apa artinya?** 🤔

  * `v`: Ini variabel baru di server, namanya **velocity** atau "kecepatan pembaruan". Dia menyimpan semacam "rata-rata bergerak" (moving average) dari pembaruan-pembaruan sebelumnya. Awalnya, `v` biasanya 0.
  * `β` (beta): Ini **faktor momentum**. Nilainya antara 0 dan 1.
    * Kalau $\beta = 0$, rumusnya jadi $v \leftarrow 0 \cdot v + \Delta w = \Delta w$, dan $w \leftarrow w - v = w - \Delta w$. Ini **sama persis** kayak FedAvg biasa! Jadi FedAvg itu kasus spesial FedAvgM dengan $\beta=0$.
    * Kalau $\beta$ dekat 1 (misal 0.9), artinya `v` baru akan **sangat dipengaruhi** oleh `v` lama ($\beta v$ jadi besar) dan **sedikit dipengaruhi** oleh $\Delta w$ sekarang. Ini bikin update `w` jadi lebih *smooth*, gak gampang goyang hanya karena $\Delta w$ di satu putaran kebetulan aneh. Dia "ingat" arah update sebelumnya.
  * $\Delta w$: Ini **sama persis** kayak $\Delta w$ di FedAvg biasa (rata-rata terbobot pembaruan klien).
  * **Langkah 1:** Kecepatan baru `v` adalah kombinasi dari kecepatan lama (dikalikan faktor `β`) ditambah pembaruan rata-rata klien saat ini ($\Delta w$).
  * **Langkah 2:** Bobot model `w` diperbarui dengan *mengurangi* kecepatan `v`, bukan $\Delta w$ langsung.

* **Nesterov Accelerated Gradient (NAG):** Jurnal ini menyebutkan pakai varian NAG. NAG itu sedikit lebih pintar dari momentum standar. Intinya, sebelum menghitung gradien ($\Delta w$), dia "intip" dulu sedikit ke arah momentum (`v`) akan membawa bobot, baru hitung gradien di titik "intipan" itu. Ini seringkali bantu konvergen lebih cepat. Tapi konsep dasarnya tetap pakai akumulasi momentum `v`.

* **Contoh Sederhana FedAvgM:**
  Pakai skenario yang sama: `w = 10`. Klien 1 ($\Delta w_1 = +0.5, n_1=60$), Klien 2 ($\Delta w_2 = -0.1, n_2=40$). Jadi $\Delta w = +0.26$.
  Misalkan ini putaran pertama, jadi `v` lama = 0. Kita pakai momentum $\beta = 0.9$.

  1.  Update kecepatan `v`:
  $v \leftarrow \beta v + \Delta w = (0.9)(0) + (+0.26) = +0.26$
  2.  Update bobot `w`:
  $w \leftarrow w - v = 10 - (+0.26) = 9.74$
  Di putaran pertama, hasilnya sama kayak FedAvg.

  Sekarang, **putaran kedua**. Anggap bobot *baru* jadi `w = 9.74`.
  Misal di putaran ini, klien yang terpilih (mungkin beda, mungkin sama) menghasilkan rata-rata pembaruan baru $\Delta w_{\text{baru}} = -0.5$ (arahnya beda jauh!). Kecepatan *lama* dari putaran sebelumnya adalah `v = +0.26`.

  * **Kalau pakai FedAvg biasa:**
  $w \leftarrow w - \Delta w_{\text{baru}} = 9.74 - (-0.5) = 10.24$ (Bobotnya langsung "loncat" balik arah).
  * **Kalau pakai FedAvgM (lanjutan):**
  1.  Update kecepatan `v`:
  $v \leftarrow \beta v + \Delta w_{\text{baru}} = (0.9)(+0.26) + (-0.5)$
  $v = 0.234 - 0.5 = -0.266$
  2.  Update bobot `w`:
  $w \leftarrow w - v = 9.74 - (-0.266) = 9.74 + 0.266 = 10.006$
  Lihat bedanya! Dengan momentum, meskipun pembaruan klien sekarang ($\Delta w_{\text{baru}}$) arahnya negatif kuat, bobot `w` *gak* langsung loncat jauh ke arah negatif. Dia masih "tertarik" sedikit oleh momentum positif dari putaran sebelumnya, jadi bergeraknya lebih *smooth* (dari 9.74 cuma jadi 10.006, gak langsung ke 10.24). Ini yang bantu meredam osilasi dari data Non-IID.

-----

## **Istilah Teknis Lainnya**

* **Weight Decay:** Seperti disebut sebelumnya, ini teknik **regularisasi**. Tujuannya mencegah **overfitting** (model terlalu hafal data latihan sampai gak bisa generalisasi ke data baru). Caranya, saat update bobot, kita tambahkan "hukuman" kecil sebanding dengan besarnya bobot itu sendiri. Jadi, model "didorong" untuk pakai bobot yang nilainya kecil-kecil saja, biar gak terlalu kompleks. Nilai 0.004 itu menentukan seberapa kuat hukumannya.
* **Learning Rate Efektif (η\<sub\>eff\</sub\> = η / (1 - β)):** Ini cara mengukur "kekuatan" update total per putaran saat pakai momentum. Kalau β=0 (tanpa momentum), η\<sub\>eff\</sub\> = η. Kalau β=0.9, η\<sub\>eff\</sub\> = η / (1-0.9) = η / 0.1 = 10η. Artinya, dengan momentum 0.9, langkah efektifnya jadi 10x lebih besar dibanding cuma pakai learning rate η saja, karena ada akumulasi dari langkah sebelumnya. Konsep ini bantu pas nyari kombinasi η dan β yang pas.
* **Magnitude (Orde Magnitudo):** Ini cara simpel bilang "kelipatan 10". Kalau dibilang rentang learning rate-nya 2 orde magnitudo, artinya rentangnya sekitar 10² = 100 kali lipat (misalnya dari 0.001 sampai 0.1). Kalau rentangnya 1 orde magnitudo, ya sekitar 10 kali lipat (misal dari 0.01 sampai 0.1).
* **Fungsi Proksimal & Fungsi Konveks Kuat:** Istilah ini muncul di bagian "Karya Terkait" saat membahas *analisis teori konvergensi* algoritma FL lain.
  * **Konveks Kuat (Strongly Convex):** Ini properti matematis dari beberapa fungsi *loss* (fungsi yang mengukur error model). Kalau fungsi *loss*-nya konveks kuat, secara teori lebih gampang dijamin kalau SGD akan konvergen ke satu titik minimum global (lembah terdalam). Analisisnya jadi lebih mudah.
  * **Fungsi Proksimal:** Ini semacam "tambahan" yang dimasukkan ke fungsi *loss* di sisi klien. Tujuannya untuk menjaga agar model lokal klien *gak* terlalu jauh berbeda dari model global server selama latihan lokal. Ini bisa bantu konvergensi, terutama di data Non-IID.
  **Penting:** Jurnal ini *menyebut* karya lain yang pakai konsep ini untuk *membuktikan* konvergensi secara teori, tapi jurnal ini *tidak* pakai fungsi proksimal atau mengasumsikan konveks kuat dalam *eksperimen* FedAvg dan FedAvgM mereka. Fokus mereka lebih ke hasil empiris/eksperimental.

-----
