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

# **Implementasi Analisis Statistika Data Polutan Menggunakan Python**

Berbeda dengan pendekatan berbasis *drag-and-drop*, proyek analisis kelayakan udara Kota Sukabumi ini sepenuhnya memanfaatkan ekosistem analitik berbasis **Python (Pandas)**. Python memberikan fleksibilitas komputasi langsung setelah data satelit berhasil diunduh.

Berikut adalah sintaks perhitungan statistik deskriptif untuk data yang sudah digabungkan menjadi `Data_Polutan_Kota-Sukabumi.csv`.

## 1. Menghitung Statistika Dasar dengan Pandas

Library `pandas` memiliki fitur bawaan `.describe()` yang merangkum keseluruhan nilai Mean, Std, Min, Max, hingga kuartil secara instan.

```python
import pandas as pd

# Memuat dataset gabungan
df = pd.read_csv("Data_Polutan_Kota-Sukabumi.csv")

# Pastikan kolom date menjadi index untuk time series
df['date'] = pd.to_datetime(df['date'])
df.set_index('date', inplace=True)

# Ekstraksi statistik dasar
stat_summary = df.describe()
print(stat_summary)
```

## 2. Menghitung Skewness, Kurtosis, dan Missing Values

Metrik asimetri, keruncingan, serta pemantauan rekam yang hilang dilakukan dengan metode lanjutan `.skew()`, `.kurtosis()`, dan fungsi pengecekan nol `.isna()`.

```python
# Menghitung derajat kemencengan (Skewness)
print("=== Skewness ===")
print(df.skew(numeric_only=True))

# Menghitung derajat keruncingan (Kurtosis)
print("\n=== Kurtosis ===")
print(df.kurtosis(numeric_only=True))

# Menghitung total data yang kosong (Missing Values)
print("\n=== Missing Values ===")
print(df.isna().sum())
```

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
