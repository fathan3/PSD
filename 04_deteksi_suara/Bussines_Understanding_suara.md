# Business Understanding: Sistem Deteksi Kebisingan Laboratorium

Tahap **Business Understanding** merupakan fase fondasi dalam siklus metodologi *Cross-Industry Standard Process for Data Mining* (CRISP-DM). Pada tahap ini, ruang lingkup proyek difokuskan pada analisis kebutuhan operasional, identifikasi kelemahan sistem konvensional, penentuan tujuan bisnis dan teknis yang terukur, serta pemetaan pemangku kepentingan (*stakeholders*) di lingkungan laboratorium akademik.

---

## 1. Latar Belakang & Urgensi Lingkungan Akademik

Laboratorium di lingkungan Departemen Teknik Informatika, Fakultas Teknik, Universitas Trunojoyo Madura (UTM) merupakan fasilitas komputasi vital yang digunakan setiap hari untuk berbagai aktivitas krusial, antara lain:
* **Praktikum Pemrograman & Rekayasa Perangkat Lunak:** Membutuhkan konsentrasi tinggi untuk analisis algoritma, penulisan sintaks, dan pelacakan *bug* (*debugging*).
* **Riset & Komputasi Kecerdasan Buatan:** Pelatihan model pembelajaran mendalam (*deep learning*) dan analisis sains data berskala besar.
* **Asistensi Tugas Besar & Bimbingan Skripsi/Tugas Akhir:** Sesi konsultasi mendalam antara dosen pembimbing, asisten laboratorium, dan mahasiswa.

Suasana ruangan yang kondusif, hening, dan tertib merupakan prasyarat mutlak untuk menjaga produktivitas kognitif, daya ingat kerja, dan fokus mental seluruh sivitas akademika di dalam laboratorium.

```
       +-------------------------------------------------------------+
       |   Tantangan Suasana Laboratorium Teknik Informatika UTM     |
       +-------------------------------------------------------------+
                                      |
         +----------------------------+----------------------------+
         |                                                         |
         v                                                         v
+-------------------------------+         +-------------------------------+
|  Kondisi Normal / Wajar       |         |  Kondisi Bising Mengganggu    |
|  - Ketikan keyboard & mouse   |         |  - Obrolan santai kelompok    |
|  - Diskusi berbisik tertib    |         |  - Candaan / tawa lepas       |
|  - Dengung AC & server lab    |         |  - Nada bicara meninggi       |
|  - Gerakan kursi sesaat       |         |  - Gaduh berulang terus-menerus|
+-------------------------------+         +-------------------------------+
         |                                                         |
         v                                                         v
   [STATUS: AMAN]                                            [STATUS: ALARM!]
```

Namun pada realitasnya, dinamika interaksi mahasiswa saat praktikum sering kali menimbulkan kebisingan yang melampaui batas wajar kenyamanan akustik. Kegaduhan ini umumnya bersumber dari:
1. Obrolan santai di luar konteks materi praktikum yang merembet antar-meja praktikan.
2. Diskusi penyelesaian tugas kelompok yang nadanya semakin meninggi dan lepas kendali.
3. Candaan atau reaksi spontan yang menimbulkan tawa berkepanjangan.

---

## 2. Rumusan Masalah (*Problem Statement*)

Dalam menertibkan suasana laboratorium, terdapat dua pendekatan yang selama ini umum digunakan, namun keduanya memiliki kelemahan mendasar:

### A. Keterbatasan Pengawasan Manual oleh Asisten Laboratorium
1. **Beban Konsentrasi Terbagi (*Cognitive Overload*):** Asisten laboratorium memikul tanggung jawab teknis utama untuk mendampingi puluhan mahasiswa dalam memecahkan kendala sintaks *coding*, logika program, dan penginstalan perangkat lunak. Asisten tidak dapat terus-menerus memantau tingkat kegaduhan setiap sudut ruangan.
2. **Subjektivitas Penilaian:** Penilaian manusia terhadap batasan "berisik" sangat bersifat relatif, bergantung pada tingkat kelelahan dan batas toleransi individu asisten pada saat itu. Hal ini sering kali memicu rasa enggan menegur atau bahkan perdebatan dengan mahasiswa yang merasa volume suaranya masih dalam batas toleransi.
3. **Ketiadaan Rekam Jejak Objektif:** Tidak ada data historis yang dapat dijadikan rujukan evaluasi oleh Kepala Laboratorium untuk memetakan kelas atau jadwal praktikum mana yang memiliki tingkat kedisiplinan rendah.

### B. Kegagalan Sensor Desibel (dB) Konvensional (*Sound Level Meter*)
Penggunaan alat ukur kenyaringan suara konvensional berbasis sensor desibel (*Sound Level Meter* / mikrofon sensor amplitudo analog) terbukti tidak efektif karena sangat rentan terhadap **False Alarm (Peringatan Palsu)**:
* **Sensitivitas Amplitudo Sesaat:** Sensor dB hanya mengukur puncak energi fisik gelombang suara (*amplitude peak*). Bunyi benturan sesaat berenergi tinggi—seperti kursi tersenggol, pintu tertutup keras, buku jatuh ke meja, atau batuk sesaat praktikan—akan langsung memicu lonjakan desibel tinggi yang menyebabkan alarm menyala, padahal ruangan sebenarnya sangat kondusif.
* **Ketidakmampuan Membedakan Karakter Suara:** Sensor dB tidak mampu membedakan apakah suara berenergi tinggi tersebut berasal dari suara dengung mesin pendingin (AC), putaran kipas kabinet *server* laboratorium, atau suara obrolan kerumunan manusia.

### Urgensi Solusi *Machine Learning*:
Oleh karena itu, diperlukan pergeseran paradigma dari sekadar pengukuran keras-lemahnya amplitudo sesaat ke **Pengenalan Pola Akustik (*Acoustic Pattern Recognition*) berbasis Machine Learning**. Sistem harus mampu mengekstraksi fitur spektral dan formulan saluran vokal manusia untuk mengenali secara cerdas apakah sebuah segmen audio merepresentasikan **kebisingan percakapan ramai** atau **suara wajar lingkungan laboratorium**.

---

## 3. Tujuan Bisnis & Sasaran Teknis Proyek

### A. Tujuan Bisnis (*Business Objectives*)
1. **Mewujudkan Pengawasan Kebisingan Otomatis & Objektif:** Menggantikan peran teguran manual asisten dengan sistem cerdas yang memantau tingkat keramaian secara kontinu 24/7 tanpa henti.
2. **Mengeliminasi *False Alarm*:** Memastikan alarm peringatan hanya aktif apabila terjadi kegaduhan percakapan manusia yang nyata, dan tetap hening saat terjadi bunyi benturan sesaat atau dengung mekanis pendingin udara.
3. **Meningkatkan Efisiensi dan Ketertiban Praktikum:** Menciptakan lingkungan belajar yang disiplin di mana asisten lab dapat fokus 100% pada pendampingan materi akademik.

### B. Sasaran Teknis Sains Data (*Data Science Objectives*)
1. **Membangun Pipeline Ekstraksi Fitur Digital Signal Processing (DSP):** Mengonversi sinyal deret waktu audio mentah (*raw audio waveform*) 16 kHz menjadi 32 fitur akustik representatif (energi RMS, Spectral Centroid, Zero Crossing Rate, dan koefisien MFCC 1–13).
2. **Menjamin Integritas Validasi (*Anti Data Leakage*):** Menerapkan partisi *Stratified Split* 80% Latih : 20% Uji pada level file rekaman asli sebelum proses pemotongan *chunk* 5 detik, mencegah model menghafal akustik ruangan dari file rekaman yang sama.
3. **Mengoptimalkan Komputasi via Seleksi Fitur (ANOVA F-Score):** Memangkas fitur akustik yang redundan dan bising dari 32 fitur menjadi **14 Fitur Terbaik Terpilih** guna menjamin latensi inferensi yang sangat rendah tanpa mengorbankan akurasi.
4. **Melatih Model Support Vector Machine (SVM) dengan Estimasi Probabilitas:** Memanfaatkan Kernel RBF, parameter regularisasi $C=3$, dan pembobotan kelas seimbang (`class_weight='balanced'`) dengan kalibrasi *Platt Scaling* untuk menghasilkan probabilitas kontinu tingkat kebisingan.
5. **Kalibrasi Ambang Batas Keputusan (*Threshold Calibration*):** Menyetel ambang batas pemicu alarm pada probabilitas **0.6 (60%)** untuk menyeimbangkan presisi tinggi dengan *recall* yang optimal.
6. **Ekapsulasi & Deployment:** Mengemas model ke dalam artefak terdistribusi `model_noise.pkl` dan merancang antarmuka web Flask `app.py` yang terhubung langsung dengan mikrofon ruangan.

---

## 4. Pemangku Kepentingan (*Stakeholders*) & Penerima Manfaat

Proyek sistem peringatan kebisingan cerdas ini dirancang untuk memberikan manfaat nyata bagi berbagai pihak di lingkungan Universitas Trunojoyo Madura:

| No | Pemangku Kepentingan | Peran & Kebutuhan Operasional | Manfaat Penerapan Sistem |
| :---: | :--- | :--- | :--- |
| 1 | **Asisten Laboratorium** | Mendampingi jalannya sesi praktikum dan menjaga ketertiban teknis ruangan. | Terbebas dari beban menegur keributan secara berulang, dapat fokus membimbing *coding*, dan memiliki landasan objektif jika harus memberi sanksi praktikan. |
| 2 | **Kepala Laboratorium Komputasi** | Pengelola manajerial fasilitas dan penjamin mutu kegiatan akademik laboratorium. | Memperoleh standar operasional berbasis teknologi cerdas, meningkatkan citra dan profesionalisme laboratorium informatika UTM. |
| 3 | **Dosen Pengampu Mata Kuliah** | Penanggung jawab kurikulum dan proses transfer pengetahuan. | Suasana kelas kondusif meningkatkan daya serap materi dan efisiensi waktu penyampaian modul praktikum. |
| 4 | **Mahasiswa Praktikan** | Pengguna fasilitas yang melaksanakan kegiatan belajar dan komputasi. | Memperoleh hak kenyamanan belajar yang adil, tidak terganggu oleh kebisingan kelompok lain, dan terlatih untuk berdisiplin di tempat kerja profesional. |

---

## 5. Metodologi CRISP-DM Terperinci

Proyek ini dilaksanakan secara ketat berdasarkan tahapan siklus industri data mining **CRISP-DM**:

1. **Business Understanding (Bab Ini):** Merumuskan urgensi suasana laboratorium, identifikasi kelemahan sensor konvensional, penentuan tujuan bisnis, dan kriteria keberhasilan.
2. **Data Understanding:** Pengumpulan dan eksplorasi 200 file rekaman audio di lingkungan laboratorium (100 Berisik vs 100 Tenang), identifikasi variabilitas noise, serta visualisasi boxplot statistik fitur kunci.
3. **Data Preparation:** Pembersihan jeda hening (*silence removal* `top_db=25`), segmentasi sinyal audio menjadi *chunk* 5 detik (80.000 sampel), ekstraksi 32 fitur akustik dasar menggunakan `librosa`, serta implementasi pembagian data *Stratified Anti-Data Leakage*.
4. **Feature Selection:** Pengujian signifikansi fitur menggunakan One-Way ANOVA F-Score pada data latih, analisis *trade-off* akurasi vs jumlah fitur, dan perumusan 14 fitur optimal.
5. **Modeling:** Pembangunan pipeline Scikit-Learn yang mengintegrasikan `StandardScaler` dan `SVC(kernel='rbf', C=3, class_weight='balanced', probability=True)`.
6. **Evaluation:** Validasi menyeluruh pada 370 sampel uji independen, kalibrasi *decision threshold* 0.6 via Precision-Recall curve, analisis Confusion Matrix, evaluasi agregasi level file (92.5%), Permutation Feature Importance, dan *latency benchmark* real-time.
7. **Deployment:** Pelatihan ulang model final pada 100% dataset, penyimpanan artefak `model_noise.pkl`, uji verifikasi inferensi sampel suara, serta arsitektur integrasi mikrofon web Flask.

---

## 6. Kriteria Keberhasilan Operasional (*Success Criteria*)

Keberhasilan sistem ini diukur melalui 3 dimensi evaluasi yang ketat:

### A. Kriteria Kinerja Sains Data (Algoritmik)
* **Akurasi Global Data Uji:** $\ge 85\%$ pada sampel *chunk* data uji yang berasal dari file rekaman independen. (Capaian Aktual: **88.65% ~ 89%**).
* **Sensitivitas / Recall Kelas Berisik:** $\ge 80\%$ guna memastikan sebagian besar keramaian nyata berhasil dijaring. (Capaian Aktual: **84.27%**).
* **Presisi Kelas Berisik:** $\ge 90\%$ guna memastikan ketika alarm menyala, probabilitas bahwa ruangan memang benar-benar berisik sangat tinggi. (Capaian Aktual: **91.46%**).
* **Akurasi Agregasi Level File:** $\ge 90\%$ pada file rekaman penuh. (Capaian Aktual: **92.50%**).

### B. Kriteria Kinerja Operasional (*Real-Time Constraints*)
* **Latensi Inferensi (*End-to-End Latency*):** Total durasi ekstraksi 14 fitur akustik ditambah prediksi model per segmen audio 5 detik harus berada **jauh di bawah 1.000 ms (1 detik)** agar alarm responsif. (Capaian Aktual: **41.21 ms**, 95th Percentile: **49.48 ms**).
* **Pencegahan *Training-Serving Skew*:** Modul ekstraksi fitur yang digunakan pada pelatihan notebook dan inferensi web wajib seragam 100% menggunakan arsitektur `features.py`.

### C. Kriteria Penerimaan Pengguna (*User Acceptance*)
* Asisten laboratorium tidak perlu melakukan konfigurasi ambang batas secara manual setiap hari.
* Indikator peringatan visual (lampu merah berkedip) dan audio alarm diaktifkan secara otomatis dan proporsional.

