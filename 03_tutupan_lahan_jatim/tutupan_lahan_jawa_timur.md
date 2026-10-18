# Sistem Informasi Geografis Tutupan Lahan Jawa Timur

## Klasifikasi Spasial Penggunaan & Tutupan Lahan (LULC) Berbasis Citra Satelit Optik Sentinel-2A Level-2A (CDSE openEO) & Machine Learning (Random Forest, HistGradientBoosting, & Ensemble Soft-Voting)

Dokumentasi ini menyajikan implementasi lengkap proyek pemodelan spasial klasifikasi penutup dan penggunaan lahan (*Land Use and Land Cover* - LULC) skala regional untuk Provinsi Jawa Timur berdasarkan standar kurikulum sains data geospasial dan kerangka kerja industri **CRISP-DM** (*Cross-Industry Standard Process for Data Mining*).

### Ringkasan Eksekutif & Metadata Proyek
* **Mata Kuliah / Topik:** Sistem Informasi Geografis & Sains Data Spasial — Klasifikasi Spasial Tutupan Lahan (*Land Use and Land Cover* - LULC)
* **Wilayah Analisis:** Seluruh Wilayah Provinsi Jawa Timur (Daratan Aluvial, Pegunungan Vulkanik, Zona Pesisir, dan Kepulauan)
* **Kerangka Kerja (Framework):** **CRISP-DM (Cross-Industry Standard Process for Data Mining)**
* **Platform Citra Satelit:** Sentinel-2A MSI Level-2A (*Bottom-of-Atmosphere* / BOA Surface Reflectance) dari European Space Agency (ESA) via **Copernicus Data Space Ecosystem (CDSE)**
* **Dataset Observasi:** 285.491 piksel multispektral (19 fitur komposit) dari 297 poligon mandiri ground truth di Jawa Timur
* **Metode Pemodelan Machine Learning:**
  1. *Model 1:* **Random Forest Classifier** (100 estimators, max depth 16, penyeimbang bobot kelas otomatis `class_weight='balanced'`)
  2. *Model 2:* **HistGradientBoosting Classifier** (Boosting sekuensial cepat berbasis 256 integer binning setara LightGBM)
  3. *Model 3 (Terbaik 🏆):* **Ensemble Soft-Voting Classifier (RF + HGB)** (Menggabungkan rata-rata probabilitas posterior kedua model)
* **Strategi Validasi Spasial:** *Spatial Group Splitting* (`GroupShuffleSplit` berbasis `poligon_id` unik, 80% Latih : 20% Uji) guna mematuhi Hukum Pertama Tobler dan mencegah kebocoran autokorelasi spasial (*Spatial Data Leakage*)
* **Capaian Kinerja Model Terbaik:** Akurasi Global Data Uji **89.06%**, Macro F1-Score **81.21%** pada 57.855 piksel uji dari 60 poligon mandiri
* **Visualisasi Geospasial Interaktif:** Studio peta Leaflet / Folium dengan basemap **Google Satellite (WMS)**, **Google Hybrid (Label Jalan & Kota)**, dan **Esri World Imagery**
* **Status Deployment:** Berjalan pada aplikasi web interaktif Streamlit Cloud ([sig-tutupan-lahan-jawa-timur.streamlit.app](https://sig-tutupan-lahan-jawa-timur.streamlit.app/))

---

### 🗺️ Akses Cepat Peta Geospasial Interaktif
Anda dapat mengeksplorasi peta interaktif secara langsung pada dokumen web ini atau membukanya di tab peramban baru:
* 📍 [**Peta 1: Evaluasi Poligon Ground Truth (Convex Hull Vektor)**](../peta_evaluasi_poligon.html) — Menampilkan batas poligon sampel, status prediksi akurat vs misklasifikasi, dan data spektral BOA.
* 🌐 [**Peta 2: Klasifikasi Regional Jawa Timur (Raster 250m)**](../peta_regional_jawatimur.html) — Menampilkan hasil klasifikasi skala regional se-Jawa Timur dengan 6 layer ImageOverlay mandiri di atas citra satelit Google & Esri.
* 📖 Panduan teknis arsitektur peta dan kode pembuatan lengkap dapat dipelajari pada bab [**Deployment & Visualisasi Interaktif**](Deployment_Visualisasi_jatim.md).

---

### Sistematika Dokumentasi & Alur CRISP-DM

Dokumentasi proyek Sistem Informasi Geografis (SIG) Tutupan Lahan Jawa Timur ini disusun secara komprehensif mematuhi siklus metodologi standar sains data spasial **CRISP-DM** yang terbagi ke dalam sub-bab berikut:

1. [**Business Understanding**](Bussines_Understanding_jatim.md): Merumuskan urgensi ketahanan pangan nasional dan ekologi Provinsi Jawa Timur, dinamika alih fungsi lahan (*urban sprawl*), sasaran proyek bagi *stakeholders* (Bappeda Jatim, Dinas Pertanian LP2B, DLHK, ATR/BPN), pencegahan *spatial leakage*, dan kriteria keberhasilan saintifik.
2. [**Data Understanding**](Data_Understanding_jatim.md): Membedah spesifikasi teknis 12 saluran multispektral satelit Sentinel-2A Level-2A BOA Surface Reflectance, taksonomi 6 kelas tutupan lahan Jawa Timur (Sawah, Bangunan, Hutan/Lahan Hijau, Danau/Waduk, Laut/Perairan Terbuka, Mangrove), serta formulasi matematis 4 indeks spektral komposit (NDVI, NDWI, MNDWI, NDBI).
3. [**Data Preparation**](Data_Preparation_jatim.md): Menyiapkan dependensi komputasi, memuat dataset 285.491 piksel Sentinel-2A, melakukan analisis eksplorasi data (EDA distribusi kelas dan visualisasi barplot rerata indeks spektral), serta menerapkan partisi *Spatial Group Splitting* berbasis kelompok `poligon_id` (227.636 piksel latih : 57.855 piksel uji).
4. [**Modeling**](Modeling_jatim.md): Merumuskan arsitektur dan formulasi matematis dari tiga model Machine Learning: Random Forest (`class_weight='balanced'`, Gini Impurity), HistGradientBoosting (Histogram Binning, Multinomial Log-Loss), dan Ensemble Soft-Voting Classifier (pembobotan probabilitas rata-rata), implementasi kode Python, dan penyimpanan artefak model `.joblib`.
5. [**Evaluation**](Evaluation_jatim.md): Melakukan benchmarking performa ketiga model pada 57.855 piksel data uji independen, menyajikan tabel perbandingan akurasi global dan Macro F1-Score, Classification Report lengkap Ensemble Soft-Voting, visualisasi *Confusion Matrix Heatmap* dan *Feature Importance*, serta audit transparan sampel poligon misklasifikasi pada zona ekoton transisi.
6. [**Deployment & Visualisasi Interaktif**](Deployment_Visualisasi_jatim.md): Merancang studio visualisasi geospasial interaktif berbasis Folium & Leaflet.js dengan basemap Google Satellite WMS, Google Hybrid, dan Esri World Imagery, yang menyajikan Peta 1 (Evaluasi Poligon Ground Truth berbasis Convex Hull Vektor dengan penanda khusus misklasifikasi) dan Peta 2 (Klasifikasi Tutupan Lahan Skala Regional Jawa Timur dengan 6 lapisan ImageOverlay raster 250m mandiri).
7. [**Kesimpulan & Rekomendasi Kebijakan**](Kesimpulan_Rekomendasi_jatim.md): Menyimpulkan temuan teknis model sains data serta merumuskan rekomendasi kebijakan spasial strategis bagi Pemerintah Provinsi Jawa Timur terkait perlindungan Lahan Pertanian Pangan Berkelanjutan (LP2B), pengendalian aglomerasi perkotaan Gerbangkertosusila, dan konservasi sabuk hijau mangrove pesisir.
8. [**Panduan Deploy ke Streamlit**](Panduan_Deploy_Streamlit.md): Panduan teknis langkah demi langkah (*step-by-step*) deployment aplikasi dashboard geospasial interaktif ke server lokal hingga publikasi daring di Streamlit Community Cloud via integrasi repositori GitHub.
