# Penjelasan Metrik Statistika Deskriptif

Dalam analisis data polusi udara deret waktu, ringkasan nilai-nilai metrik yang disajikan dalam bentuk tabel statistik dikenal sebagai **Statistika Deskriptif (*Descriptive Statistics*)**. Ringkasan ini sangat penting pada tahap **Exploratory Data Analysis (EDA)**. Tujuan utamanya adalah untuk memahami karakteristik distribusi data, tren, serta mengevaluasi kualitas data pemantauan sebelum data tersebut digunakan lebih lanjut pada tahap peramalan (*forecasting*) atau pemodelan *Machine Learning*.

Berdasarkan dataset pengamatan polutan udara Kota Sukabumi ($NO_2$, $CO$, $SO_2$, $O_3$), berikut adalah penjelasan terkait berbagai metrik statistik yang digunakan beserta metode perhitungan manualnya:

## 1. Min & Max
*   **Penjelasan:** Menunjukkan titik observasi terendah (Min) dan tertinggi (Max) di dalam kumpulan data. Metrik ini berguna untuk mengetahui seberapa lebar rentang ekstrem dari kualitas udara.
*   **Perhitungan Manual:** Mengurutkan seluruh data polutan dari konsentrasi terkecil hingga terbesar.
    *   $Min = X_1$ (Nilai observasi paling awal setelah diurutkan)
    *   $Max = X_n$ (Nilai observasi paling akhir)

## 2. Mean
*   **Penjelasan:** Rata-rata aritmatika dari seluruh observasi konsentrasi gas polutan. Nilai ini mewakili titik pusat konsentrasi rata-rata harian.
*   **Perhitungan Manual:**
    $$ \bar{x} = \frac{\sum_{i=1}^{n} x_i}{n} $$
    *(Total jumlah konsentrasi observasi dibagi dengan jumlah hari observasi yang tidak kosong/valid).*

## 3. Std. Deviation (Standar Deviasi)
*   **Penjelasan:** Menggambarkan seberapa tersebar titik data dari rata-ratanya (Mean). Standar deviasi yang kecil menunjukkan bahwa data konsentrasi polusi sangat stabil, sedangkan nilai yang besar menandakan fluktuasi harian yang sangat tinggi.
*   **Perhitungan Manual (Sampel):**
    $$ s = \sqrt{\frac{\sum_{i=1}^{n} (x_i - \bar{x})^2}{n-1}} $$

## 4. Variance (Varians)
*   **Penjelasan:** Nilai rata-rata dari kuadrat jarak setiap titik observasi terhadap Mean. Ini adalah bentuk kuadrat dari Standar Deviasi.
*   **Perhitungan Manual (Sampel):**
    $$ s^2 = \frac{\sum_{i=1}^{n} (x_i - \bar{x})^2}{n-1} $$

## 5. Skewness 
*   **Penjelasan:** Metrik yang mengukur ketidaksimetrisan dari distribusi data polutan.
    *   *Skewness = 0*: Kurva berpusat sempurna dan simetris (distribusi normal).
    *   *Skewness > 0 (Positif)*: Distribusi condong ke kiri dengan ekor panjang ke kanan. Mengindikasikan adanya lonjakan ekstrem (pencemaran tinggi) yang terjadi pada beberapa hari tertentu.
    *   *Skewness < 0 (Negatif)*: Distribusi condong ke kanan dengan ekor ke kiri.
*   **Perhitungan Manual (Fisher-Pearson):**
    $$ Skewness = \frac{n}{(n-1)(n-2)} \sum_{i=1}^{n} \left(\frac{x_i - \bar{x}}{s}\right)^3 $$

## 6. Kurtosis
*   **Penjelasan:** Menunjukkan tingkat keruncingan puncak distribusi dan ketebalan ekor data (bobot *outlier*). 
    *   *Kurtosis ≈ 0*: Normal (Mesokurtik).
    *   *Kurtosis > 0*: Puncak yang sangat runcing dan ekor tebal (Leptokurtik). Ini berarti banyak *outlier* atau lonjakan polusi ekstrem di Sukabumi.
    *   *Kurtosis < 0*: Puncak distribusi datar (Platikurtik).
*   **Perhitungan Manual (Excess Kurtosis Sampel):**
    $$ Kurtosis = \left[ \frac{n(n+1)}{(n-1)(n-2)(n-3)} \sum \left(\frac{x_i - \bar{x}}{s}\right)^4 \right] - \frac{3(n-1)^2}{(n-2)(n-3)} $$

## 7. Overall Sum
*   **Penjelasan:** Jumlah total akumulatif nilai dari seluruh pengamatan di periode tersebut.
*   **Perhitungan Manual:**
    $$ Sum = \sum_{i=1}^{n} x_i $$

## 8. Metrik Kualitas (Missing Values & Anomali)
Dalam akuisisi data satelit Sentinel-5P, metrik ini menjadi vital karena pantauan optik sering terhalang cuaca atau awan.
*   **No. missings:** Frekuensi sel data yang kosong (NaN) karena satelit gagal merekam wilayah Kota Sukabumi akibat hambatan awan tebal.
*   **Perhitungan Manual:** Mentotalkan jumlah baris observasi yang nilainya tidak terekam/kosong (NaN).

## 9. Median
*   **Penjelasan:** Nilai tengah dari deretan pengamatan yang sudah diurutkan. Median sangat kebal terhadap adanya lonjakan polusi sesaat (*outlier*).
*   **Perhitungan Manual:** Urutkan seluruh data $X_1$ hingga $X_n$.
    *   Jumlah data ($n$) ganjil: $Median = X_{(n+1)/2}$
    *   Jumlah data ($n$) genap: $Median = \frac{X_{n/2} + X_{(n/2)+1}}{2}$

---

# **Implementasi Analisis Data Polutan: Integrasi Cloud Database ke KNIME**

Bagian ini memaparkan langkah-langkah teknis untuk menghubungkan basis data PostgreSQL yang di-*hosting* pada platform cloud Aiven, mengecek ketersediaan data via pgAdmin 4, hingga melakukan ekstraksi perhitungan statistika deskriptif menggunakan perangkat lunak analitik visual KNIME Analytics Platform.

## Langkah 1: Mendapatkan Kredensial Akses Database Aiven

Sebelum melakukan koneksi melalui *client* mana pun, kita membutuhkan rincian kredensial server.
1. Masuk ke *dashboard* atau console utama **Aiven**, kemudian pilih proyek Anda.
2. Beralih ke tab **Overview** pada layanan PostgreSQL yang sedang aktif berjalan.
3. Pada segmen **Connection information**, perhatikan dan salin beberapa parameter krusial berikut:
   * **Host:** `[Ganti dengan Host Aiven Anda]`
   * **Port:** `[Ganti dengan Port Anda]`
   * **User:** `[Ganti dengan Username Anda]`
   * **Password:** (Klik ikon *copy* atau visibilitas untuk menyalin kata sandi)
   * **SSL mode:** `require`
4. Apabila aplikasi *client* membutuhkan, pastikan Anda juga mengunduh *CA certificate* (Sertifikat SSL) yang tersedia.

![Aiven PostgreSQL Console]([Masukkan Path Foto Aiven Anda])

---

## Langkah 2: Menyiapkan Koneksi Server di pgAdmin 4

Aplikasi pgAdmin 4 difungsikan untuk memverifikasi tabel dan data secara langsung sebelum dipindahkan ke alat analitik.
1. Jalankan **pgAdmin 4**. Di panel kiri (Browser), klik kanan pada bagian **Servers** > **Register** > **Server...**
2. Di tab **General**, tuliskan nama koneksi sesuai keinginan Anda (misal: `Database Polutan Sukabumi`).
3. Berpindah ke tab **Connection**, lengkapi form sesuai dengan data dari Langkah 1:
   * **Host name/address:** Tempelkan *Host* dari Aiven.
   * **Port:** Masukkan *Port* yang sesuai.
   * **Maintenance database:** Isikan nama spesifik *database* Anda.
   * **Username:** Masukkan *User*.
   * **Password:** Tempelkan kata sandi, lalu centang **Save password?** agar tidak perlu mengetik ulang nanti.
4. Klik tombol **Save** untuk memulai koneksi ke server *cloud*.

![Konfigurasi Session Manager pgAdmin]([Masukkan Path Foto pgAdmin Anda])

---

## Langkah 3: Memeriksa Ketersediaan Data di pgAdmin 4

Setelah terkoneksi, langkah selanjutnya adalah meninjau data mentah untuk memastikan strukturnya sudah benar.
1. Melalui panel kiri pgAdmin, rentangkan *tree* menu server Anda menuju Databases > `[Nama Database Anda]` > Schemas > `public` > Tables > `[Nama Tabel Anda]`.
2. Klik kanan pada tabel tersebut, lalu navigasikan ke **View/Edit Data** > **All Rows**.
3. Pastikan kolom-kolom penting seperti waktu pengamatan (`date`) dan nilai polutan (`no2`, `co`, `so2`, `o3`) muncul dengan benar.
4. Pada peninjauan ini, sel data yang bernilai `[null]` adalah hal yang sangat wajar. Nilai ini nantinya akan ditangani sebagai *missing values*.

![Tampilan Data Polutan di pgAdmin]([Masukkan Path Foto Data pgAdmin Anda])

---

## Langkah 4: Membangun Alur Kerja (Workflow) di KNIME Analytics Platform

Sekarang kita beralih ke KNIME untuk mengambil data dari database dan menghitung metrik statistik secara otomatis.
1. Buka **KNIME Analytics Platform** dan buatlah lembar *workflow* baru.
2. Dari panel *Node Repository*, cari dan tarik (*drag-and-drop*) *node* berikut ke area *workspace*:
   * **PostgreSQL Connector:** Untuk membuat jembatan koneksi ke server Aiven.
   * **DB Table Selector:** Untuk membidik tabel data spesifik di dalam *database*.
   * **DB Reader:** Untuk mengonversi tabel *database* ke dalam bentuk memori data internal KNIME.
   * **Statistics:** Untuk memproses perhitungan metrik matematis dari data tersebut.
3. Hubungkan setiap *node* secara berurutan sesuai daftar di atas (dari Connector hingga Statistics).
4. **Konfigurasi Node:**
   * Klik ganda **PostgreSQL Connector**, masukkan detail *Hostname*, *Port*, *Database name*, dan *Credentials* persis seperti langkah Aiven sebelumnya.
   * Klik ganda **DB Table Selector**, lalu arahkan untuk memilih skema `public` dan tabel polutan Anda.
5. Klik kanan pada node **DB Reader** dan pilih **Execute**. Lampu indikator hijau akan menyala jika data berhasil dimuat.

![Alur Kerja Database dan Statistik di KNIME]([Masukkan Path Foto Node KNIME Anda])

---

## Langkah 5: Mengeksekusi Output Statistika Deskriptif

Tahap pungkasan adalah menjalankan mesin analitik statistik di dalam KNIME.
1. Klik kanan pada node **Statistics**, lalu pilih **Execute**.
2. Setelah lampu berubah hijau, klik kanan lagi pada node **Statistics** dan pilih **Statistics View** (atau klik ikon kaca pembesar).
3. Sebuah jendela berisi tabel matriks akan terbuka. Anda dapat meninjau:
   * **Min, Max, Mean:** Rentang konsentrasi harian serta rata-ratanya.
   * **Std. deviation & Variance:** Analisis volatilitas tingkat polusi.
   * **Skewness & Kurtosis:** Deteksi asimetri kurva dan keberadaan *outlier*.
   * **No. missings:** Jumlah rekaman sensor yang bolong/gagal terekam.
   * **Histogram:** Grafik sederhana dari penyebaran nilainya.

![Tabel Hasil Output Node Statistics]([Masukkan Path Foto Output Statistik KNIME Anda])

## 3. Penjelasan Distribusi Polutan Kota Sukabumi

Berdasarkan ekstraksi dataset historis satelit terhadap langit Kota Sukabumi dari Agustus 2025 s.d. Agustus 2026 (total 366 hari), berikut narasinya:

1. **O3 (Ozon)**
   Data ozon memiliki ketersediaan paling stabil di antara polutan lain, yaitu hanya terdapat **17 *missing values***. Rata-rata (mean) paparan O3 di wilayah Sukabumi adalah sekitar 0.116. Karena persebaran distribusinya cukup stabil, standar deviasinya sangat kecil. Skewness menunjukkan asimetri positif yang sangat landai, artinya variasi harian Ozon Sukabumi relatif konsisten tanpa banyak lonjakan kejutan.

2. **CO (Karbon Monoksida)**
   Rekam data Karbon Monoksida menunjukkan **168 observasi hilang**. Rata-rata kadar CO di atmosfer Sukabumi berada pada kisaran 0.030 dengan varians yang sangat kecil. Nilai kurtosis dari observasi CO yang rendah menandakan karakteristik platikurtik, di mana frekuensi lonjakan ekstrem gas buangan pembakaran tidak terjadi terlalu signifikan di kota ini.

3. **SO2 (Sulfur Dioksida)**
   Satelit Sentinel kehilangan **247 hari** pantauan (hanya 119 hari valid) yang kemungkinan besar disebabkan tebalnya awan di atas Jawa Barat pada periode hujan. Mean dan Median untuk SO2 nyaris menyentuh angka nol, menandakan bahwa wilayah Sukabumi secara umum sangat bersih dari senyawa pembakaran fosil/batu bara, meskipun pada beberapa observasi menunjukkan sedikit *outlier* yang tecermin dari ekor distribusi.

4. **NO2 (Nitrogen Dioksida)**
   Dari total setahun kalender, NO2 kehilangan sebagian besar observasinya hingga **304 data kosong**, sehingga hanya menyisakan sekitar 62 hari data valid. Rata-rata kadar emisi kendaraan berat ini sangat kecil (mendekati 0.00004), namun dari segelintir observasi valid tersebut, terdeteksi beberapa *outlier* atas yang terkonfirmasi oleh deteksi *Isolation Forest* pada tahap sebelumnya. Ekor distribusinya (kurtosis) lebih runcing dibanding polutan lain karena adanya variasi harian mendadak.

---

### Perhitungan Manual Ozon (O3)

Sebagai rujukan matematis, berikut merupakan contoh penerapan perhitungan manual komputasi untuk kolom pengamatan $O_3$ (Ozon) di Sukabumi.

1. **Standar Deviasi**

$$ s = \sqrt{\frac{\sum_{i=1}^{n} (x_i - \bar{x})^2}{n-1}} $$

Diketahui:
- $n$ adalah jumlah baris valid (karena terdapat 17 missing values dari total 366 hari, $n = 366 - 17 = 349$)
- $\bar{x}$ (rata-rata $O_3$) diasumsikan $\approx 0.116$

$$
s &= \sqrt{\frac{\sum_{i=1}^{349} (x_i - 0.116)^2}{349-1}}\\
\\
s &\approx 0.0025
$$

2. **Variansi**

Variansi dapat dihitung secara langsung dengan mengkuadratkan Standar Deviasi ($s$).

$$
v &= s^2\\
v &= 0.0025^2\\
v &= 0.00000625
$$

3. **Skewness**

$$ Skewness = \frac{n}{(n-1)(n-2)} \sum_{i=1}^{n} \left(\frac{x_i - \bar{x}}{s}\right)^3 $$

Rumus ini dikomposisikan dalam dua blok konstanta dan penjumlahan momen:

$$ 
A &= \frac{n}{(n-1)(n-2)} = \frac{349}{(348)(347)} = \frac{349}{120756} \approx 0.00289\\
\\
B &= \sum_{i=1}^{349}\left(\frac{x_i-0.116}{0.0025}\right)^3
$$

Jika nilai total penyimpangan dipangkat tiga ($B$) menghasilkan misal 153.2:

$$
Skewness &= A \times B\\
Skewness &= 0.00289 \times 153.2 \approx 0.442
$$
*(Menandakan asimetri positif lemah).*

4. **Overall Sum**

Jumlah kumulatif dari 349 nilai pengamatan valid Ozon selama setahun pemantauan.

$$
OS &= \sum_{i=1}^{n}x_i \\
OS &= 0.115637 + 0.113963 + 0.118218 + \dots + x_{349} \\
OS &\approx 40.484
$$
