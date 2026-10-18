# Kesimpulan & Rekomendasi Kebijakan Tata Ruang

Tahap akhir metodologi CRISP-DM ini merangkum seluruh temuan saintifik empiris yang dihasilkan oleh arsitektur model **Machine Learning Ensemble**, karakteristik respon spektral citra satelit **Sentinel-2A**, serta mentransformasikannya menjadi rekomendasi kebijakan strategis penataan ruang dan perlindungan lingkungan hidup bagi Pemerintah Provinsi Jawa Timur.

---

## 1. Ringkasan Temuan Saintifik Sains Data Spasial

1. **Keunggulan Arsitektur Ensemble Soft-Voting (RF + HGB) 🏆**:  
   Hasil benchmarking komparasi tiga arsitektur model membuktikan bahwa **Ensemble Soft-Voting Classifier** yang memadukan Random Forest (bagging) dan HistGradientBoosting (boosting sekuensial) berhasil mengungguli seluruh model tunggal dengan **Akurasi Global 89.06%** dan **Macro F1-Score 81.21%** pada 57.855 piksel data uji mandiri. Mekanisme perataan probabilitas posterior secara signifikan meningkatkan ketahanan prediksi dan menaikkan presisi deteksi pada kelas minoritas Bangunan menjadi **71.93%** (recall 80.00%).

2. **Daya Pisah Biofisik Indeks Spektral (Feature Importance)**:  
   Analisis *Gini Feature Importance* membuktikan bahwa:
   * **NDWI** dan **B08 (NIR)** merupakan fitur paling dominan yang memisahkan perairan laut dan danau secara mutlak dari daratan tanpa ambiguitas (akurasi kelas Laut mencapai **99.77%**).
   * **B11 (SWIR-1)** dan **NDBI** merupakan fitur kunci pendeteksi material anorganik kawasan terbangun permukiman dan perkotaan.
   * **NDVI** dan **B04 (Red)** membedakan secara tegas stratifikasi kanopi vegetasi antara hutan lindung pegunungan, vegetasi sawah musiman, dan komunitas mangrove pesisir.

3. **Integritas Metodologi Bebas Kebocoran Spasial (*Spatial Leakage Free*)**:  
   Penerapan partisi *Spatial Group Splitting* (`GroupShuffleSplit` berbasis kelompok `poligon_id`) berhasil menjamin bahwa evaluasi model benar-benar mengukur kemampuan generalisasi pada 60 poligon mandiri baru, mematuhi Hukum Pertama Tobler dan meniadakan akurasi semu akibat autokorelasi piksel bertetangga.

4. **Visualisasi Interaktif Berstandar Kartografi**:  
   Penyajian visualisasi peta geospasial berbasis Leaflet dan Folium dengan dukungan **Google Satellite WMS**, **Google Hybrid**, dan **Esri World Imagery** menghadirkan kejelasan spasial ganda: audit poligon ground truth beserta penanda khusus misklasifikasi pada Peta 1, serta penjelajahan sebaran regional 1,37 juta piksel raster 250m dengan 6 lapisan ImageOverlay mandiri pada Peta 2.

---

## 2. Rekomendasi Kebijakan Tata Ruang Wilayah (RTRW Jawa Timur & LP2B)

Berdasarkan bukti empiris hasil klasifikasi tutupan lahan Provinsi Jawa Timur, dirumuskan tiga rekomendasi kebijakan strategis bagi pemangku kepentingan (Bappeda Provinsi Jawa Timur, Dinas Pertanian, Dinas Lingkungan Hidup, Dinas Kehutanan, dan ATR/BPN):

### 1. 🌾 Pengendalian Alih Fungsi Lahan Pertanian (Zonasi LP2B)
* **Kondisi Spasial Lapangan:**  
  Hasil klasifikasi membuktikan bahwa petak sawah irigasi teknis terbentang sangat luas di dataran aluvial Kabupaten Lamongan, Bojonegoro, Jember, Lumajang, dan Banyuwangi dengan performa deteksi model mencapai presisi **85.17%** dan F1-Score **83.70%**.
* **Rekomendasi Kebijakan Konkret:**
  * **Integrasi Peta Digital ke Portal OSS**: Menjadikan peta poligon tutupan sawah hasil klasifikasi sebagai lapisan kontrol geospasial (*geofencing layer*) di dalam sistem *Online Single Submission* (OSS) perizinan berusaha, guna memblokir otomatis izin alih fungsi lahan sawah beririgasi teknis menjadi pergudangan atau kawasan industri.
  * **Moratorium Koridor Jalan Tol Trans-Jawa**: Memperketat moratorium alih fungsi lahan sawah di sepanjang koridor jalan tol trans-Jawa (ruas Mojokerto–Jombang–Kertosono–Madiun) yang mengalami tekanan penjalaran urban (*urban sprawl*) paling intensif.
  * **Insentif Fiskal bagi Petani LP2B**: Memberikan insentif pembebasan Pajak Bumi dan Bangunan (PBB) sektor perdesaan serta subsidi benih dan pupuk bagi pemilik lahan sawah yang berkomitmen mempertahankan status sawah abadi.

---

### 2. 🌿 Sabuk Hijau Konservasi Mangrove Pesisir (*Blue Carbon Sinks*)
* **Kondisi Spasial Lapangan:**  
  Tutupan mangrove teridentifikasi secara presisi di muara-muara sungai besar dan kawasan pesisir (Wonorejo Pamurbaya Surabaya, Ujungpangkah Gresik, muara Kali Porong Sidoarjo, dan Teluk Pangpang Banyuwangi) dengan recall deteksi mencapai **86.53%**.
* **Rekomendasi Kebijakan Konkret:**
  * **Penetapan Garis Sempadan Mangrove Ketat**: Menertibkan tambak-tambak budidaya tak berizin dan reklamasi liar yang merambah zona inti sabuk hijau mangrove di pesisir utara dan timur Jawa Timur.
  * **Valuasi & Perdagangan Karbon Biru (*Blue Carbon Credit*)**: Memanfaatkan data spasial kanopi mangrove untuk inventarisasi biomassa dan cadangan karbon biru (*blue carbon*) dalam skema pendanaan iklim internasional bekerja sama dengan Kementerian Lingkungan Hidup dan Kehutanan.
  * **Sabuk Penahan Bencana Pesisir**: Melakukan restorasi bakau terintegrasi untuk memitigasi risiko abrasi air laut, gelombang pasang, dan penurunan muka tanah (*land subsidence*) di wilayah pesisir padat penduduk.

---

### 3. 🏞️ Perlindungan Badan Air Daratan (Danau, Waduk, & Ranu)
* **Kondisi Spasial Lapangan:**  
  Badan air tawar daratan utama (Waduk Karangkates Malang, Waduk Selorejo, Waduk Wonorejo Tulungagung, Waduk Sutami, dan Ranu Kumbolo) berhasil dipetakan secara akurat dengan presisi **81.24%**.
* **Rekomendasi Kebijakan Konkret:**
  * **Delimitasi Sempadan Danau/Waduk Minimal 50 Meter**: Menerapkan penegakan batas sempadan danau dan waduk minimal 50 meter dari titik pasang tertinggi, melarang pendirian bangunan permanen dan aktivitas pertanian intensif yang memicu erosi tebing.
  * **Pengendalian Eutrofikasi & Sedimentasi**: Mengendalikan limpasan pupuk kimia pertanian dari hulu yang memicu blooming eceng gondok di permukaan danau/waduk, menjaga kapasitas tampung air baku, irigasi pertanian, dan keandalan pembangkit listrik tenaga air (PLTA).
