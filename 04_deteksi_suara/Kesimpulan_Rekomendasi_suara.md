# Kesimpulan & Rekomendasi: Sintesis Ilmiah dan Pengembangan Masa Depan

Bab ini menyajikan sintesis komprehensif atas temuan saintifik dan capaian teknis proyek **Sistem Peringatan Kebisingan Cerdas (*Smart Noise Alert System*)** untuk Laboratorium Departemen Teknik Informatika Universitas Trunojoyo Madura (UTM), membandingkan keunggulan sistem dengan sensor konvensional, serta merumuskan rekomendasi strategis bagi pengembangan perangkat keras cerdas (*Edge AI / IoT*) di masa mendatang.

---

## 1. Sintesis Temuan Utama Proyek CRISP-DM

Melalui penerapan metodologi berstandar industri sains data *Cross-Industry Standard Process for Data Mining* (CRISP-DM), proyek ini telah berhasil menyelesaikan seluruh siklus hidup pengembangan sistem:

1. **Rekayasa Fitur Akustik Multidimensi:**  
   Transformasi sinyal audio mentah deret waktu ($80.000\text{ sampel/segmen}$ pada $16\text{ kHz}$) berhasil diringkas ke dalam $32\text{ fitur akustik dasar}$ yang merepresentasikan daya energi fisik (RMS), pergeseran spektrum frekuensi (Spectral Centroid), dinamika perlintasan nol (Zero Crossing Rate), serta pemetaan formulan anatomi saluran vokal manusia (MFCC 1–13).
2. **Pencegahan Kebocoran Data (*Anti Data Leakage*):**  
   Penerapan partisi *Stratified Split* ($80\% : 20\%$) secara ketat pada level file rekaman asli (sebelum proses segmentasi *chunk*) berhasil mencegah model menghafal lingkungan akustik rekaman yang sama. Hal ini menjamin bahwa seluruh metrik evaluasi yang dilaporkan benar-benar mencerminkan kemampuan generalisasi model terhadap rekaman audio baru.
3. **Efisiensi Komputasi melalui Seleksi Fitur (ANOVA F-Score):**  
   Pengujian signifikansi One-Way ANOVA F-Score terbukti mampu memangkas dimensi dari $32\text{ fitur}$ menjadi **$14\text{ Fitur Terbaik Terpilih}$** (reduksi dimensi sebesar $> 56\%$). Seleksi fitur ini terbukti mengeliminasi multikolinearitas dan derau frekuensi tinggi tanpa mengorbankan akurasi klasifikasi.
4. **Keandalan Model Support Vector Machine (SVM):**  
   Penggunaan kernel non-linier RBF dipadukan dengan standardisasi fitur (`StandardScaler`), penyeimbang kelas seimbang (`class_weight='balanced'`), serta kalibrasi probabilitas kontinu *Platt Scaling* menghasilkan akurasi data uji sebesar **$88.65\% \approx 89\%$** dan Macro F1-Score sebesar **$0.89$**.
5. **Akurasi Tinggi pada Evaluasi Level File Rekaman Penuh ($92.50\%$):**  
   Ketika probabilitas inferensi diagregasikan ke level file rekaman penuh, performa sistem meningkat menjadi **$92.50\%$** ($37$ dari $40$ file rekaman uji asli diprediksi tepat), membuktikan bahwa sistem sangat stabil untuk pemantauan jangka panjang.
6. **Kalibrasi Ambang Batas Optimal ($0.6$):**  
   Penyetelan ambang batas keputusan pada nilai $0.6$ terbukti menjadi titik ekuilibrium ideal yang berhasil menekan angka *False Alarm* hingga tersisa $7.3\%$ saja pada data uji, dengan tetap menjaga sensitivitas tangkapan keramaian (*Recall*) sebesar $84.3\%$.
7. **Kinerja Operasional Sangat Cepat (*Ultra-Low Latency*):**  
   Waktu pemrosesan rata-rata sebesar **$41.21\text{ ms}$** per segmen audio $5\text{ detik}$ (hanya $4.1\%$ dari batas toleransi $1.000\text{ ms}$) membuktikan kesiapan sistem untuk berjalan secara *real-time* di laboratorium tanpa menimbulkan jeda (*lag*).

---

## 2. Keunggulan Komparatif: Machine Learning vs Sensor Desibel (dB) Konvensional

Implementasi pengenalan pola audio berbasis Machine Learning menghadirkan lompatan teknologi yang signifikan dibandingkan alat ukur kebisingan berbasis ambang amplitudo analog:

| Parameter Perbandingan | Sensor Desibel (dB) Konvensional | Sistem Cerdas SVM Audio (Proyek Ini) |
| :--- | :--- | :--- |
| **Prinsip Kerja Deteksi** | Mengukur lonjakan amplitudo gelombang sesaat (*Peak Amplitude*). | Menganalisis spektrum frekuensi vokal manusia dan energi berkelanjutan (*MFCC + RMS + ZCR*). |
| **Resistansi *False Alarm*** | **Sangat Buruk.** Kursi bergeser, pintu tertutup keras, atau batuk sesaat langsung memicu alarm. | **Sangat Baik ($92\%$ Presisi).** Kebal terhadap benturan sesaat karena tidak memiliki pola formulan suara manusia. |
| **Pembeda Karakter Suara** | Tidak mampu membedakan dengung kipas angin/AC dari suara manusia. | Mampu membedakan desau mekanik pendingin dari kerumunan orang berbicara aktif. |
| **Penyesuaian Ambang Batas** | Manual melalui potensiometer putar fisik yang tidak terkalibrasi. | Berbasis probabilitas matematis terkalibrasi (*Platt Scaling Sigmoid*) yang dapat disetel via perangkat lunak. |
| **Integrasi Perangkat Lunak** | Terisolasi pada modul elektronik analog lokal. | Terintegrasi penuh dengan server web Flask, logging data, dan sistem notifikasi jaringan. |

---

## 3. Dampak Operasional bagi Laboratorium Informatika UTM

Penerapan sistem peringatan kebisingan ini membawa manfaat manajerial dan akademik yang nyata:
* **Penghapusan Beban Pengawasan Ganda:** Asisten laboratorium tidak perlu lagi membagi perhatian antara membimbing kendala *coding* praktikan dan menegur keributan ruangan.
* **Transparansi & Reduksi Friksi Antar-Sivitas:** Penilaian kebisingan yang dilakukan secara otomatis oleh sistem kecerdasan buatan menghilangkan potensi prasangka subjektif atau rasa tersinggung saat praktikan ditegur.
* **Peningkatan Kualitas Belajar:** Suasana praktikum yang tertib secara langsung berkorelasi dengan peningkatan pemahaman logika algoritma dan penurunan kesalahan sintaks pemrograman mahasiswa.

---

## 4. Rekomendasi Pengembangan Masa Depan (*Future Work*)

Untuk meningkatkan kapabilitas sistem ke tingkat kesiapan implementasi industri yang lebih tinggi, direkomendasikan rencana aksi berikut:

```
                          Rencana Pengembangan Masa Depan
                                        |
       +--------------------------------+--------------------------------+
       |                                |                                |
       v                                v                                v
+----------------------+     +----------------------+     +----------------------+
| 1. Perangkat Edge IoT|     | 2. Adaptive Noise    |     | 3. Smart Webhook     |
|   Raspberry Pi /     |     |   Dynamic Profiling  |     |   Bot Telegram Lab   |
|   ESP32-S3 + I2S Mic |     |   (Auto-Kalibrasi)   |     |   & Dashboard Audit  |
+----------------------+     +----------------------+     +----------------------+
```

1. **Implementasi Perangkat Keras Komputasi Edge (*Edge AI / IoT Hardware*):**
   * Mengemas model `model_noise.pkl` ke dalam perangkat mikrokomputer berspesifikasi hemat daya seperti **Raspberry Pi 4 / 5** atau mikrokontroler **ESP32-S3 (via TinyML)** yang dilengkapi dengan mikrofon digital I2S presisi tinggi (seperti modul **INMP441**).
   * Menjadikan sistem sebagai perangkat mandiri (*standalone IoT box*) yang dapat digantung di plafon tengah ruangan laboratorium tanpa membutuhkan komputer desktop induk.
2. **Profil Suara Latar Dinamis (*Adaptive Ambient Noise Profiling*):**
   * Mengembangkan fitur kalibrasi latar otomatis yang mempelajari tingkat kebisingan dasar (*noise floor*) ruangan saat kondisi kosong di pagi hari, sehingga ambang batas probabilitas dapat beradaptasi secara dinamis terhadap perubahan musim (misalnya saat hujan lebat di luar ruangan).
3. **Integrasi Notifikasi Bot Telegram & Logging Historis:**
   * Menghubungkan server Flask dengan webhook bot Telegram pengurus laboratorium untuk mengirimkan rekaman log waktu (*timestamp*) ketika terjadi insiden kegaduhan berulang.
   * Membangun dasbor analitik deret waktu untuk memetakan jam-jam praktikum yang paling sering mengalami pelanggaran ketertiban sebagai bahan evaluasi rapat akademik jurusan.

