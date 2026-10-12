# Sistem Informasi Geografis Tutupan Lahan Jawa Timur

## Pendekatan Supervised Fuzzy Logic Berbasis Citra Satelit Sentinel-2A Level-2A dan Analisis Kebijakan Tata Ruang

Dokumentasi ini menyajikan implementasi lengkap proyek pemodelan spasial klasifikasi penutup lahan (*Land Use and Land Cover* - LULC) skala regional untuk Provinsi Jawa Timur.

### Ringkasan Eksekutif & Metadata Proyek
* **Mata Kuliah:** Penambangan Data Spasial (PSD) / Proyek Sains Data Spasial
* **Topik Utama:** Klasifikasi Spasial Tutupan Lahan (*Land Use and Land Cover* - LULC) Skala Regional
* **Wilayah Studi:** Provinsi Jawa Timur, Indonesia
* **Kerangka Kerja (Framework):** **CRISP-DM (Cross-Industry Standard Process for Data Mining)**
* **Metode Utama:** *Supervised Fuzzy Distance-Based Classifier* (Logika Fuzzy / Maximum Membership $m=2.0$)
* **Platform Citra Satelit:** Sentinel-2A MSI Level-2A (*Bottom-of-Atmosphere* / BOA Surface Reflectance)
* **Dataset Ground Truth:** 300 Poligon Vektor Lapangan (6 Kelas Seimbang @ 50 Poligon)
* **Capaian Model:** Akurasi Data Uji **96.67%**, Cohen's Kappa **0.9600** (*Almost Perfect Agreement*)
* **Status Deployment:** Aktif & Beroperasi di Streamlit Cloud ([sig-tutupan-lahan-jawa-timur.streamlit.app](https://sig-tutupan-lahan-jawa-timur.streamlit.app/))

---

### Sistematika Dokumentasi & Alur CRISP-DM

Dokumentasi proyek Sistem Informasi Geografis (SIG) Tutupan Lahan Jawa Timur ini disusun secara komprehensif mematuhi siklus metodologi standar sains data spasial **CRISP-DM** yang terbagi ke dalam sub-bab berikut:

1. [**Business Understanding**](Bussines_Understanding_jatim.md): Merumuskan latar belakang urgensi pangan dan ekologi Jawa Timur, rumusan masalah batas kabur (*spatial vagueness*), sasaran proyek bagi *stakeholders* (Bappeda, Dinas Pertanian LP2B, DLHK), kriteria keberhasilan saintifik, dan alasan fundamental pemilihan Logika Fuzzy.
2. [**Data Understanding**](Data_Understanding_jatim.md): Membedah spesifikasi teknis band multispektral satelit Sentinel-2A Level-2A (B02, B03, B04, B08, B11), taksonomi 6 kelas tutupan lahan Jawa Timur (Sawah, Bangunan, Lahan Hijau, Danau, Perairan Terbuka, Mangrove), serta formulasi rekayasa 9 fitur indeks spektral biofisik (NDVI, NDWI, MNDWI, NDBI).
3. [**Data Preparation**](Data_Preparation_jatim.md): Mempersiapkan lingkungan kerja komputasi, strategi mitigasi bahaya kebocoran autokorelasi spasial (*Tobler's First Law*), ekstraksi 300 poligon Shapefile ground truth, penyusunan dataset fitur, analisis eksplorasi data (EDA distribusi & separabilitas ruang spektral 2D), serta partisi data latih-uji tersatifikasi (70% : 30%).
4. [**Modeling**](Modeling_jatim.md): Merumuskan formulasi matematis *Supervised Fuzzy Distance Classifier* ($m=2.0$), menyajikan simulasi perhitungan manual langkah demi langkah dengan angka riil, pembuktian verifikasi kode Python, implementasi kelas model *from-scratch* berbasis NumPy, dan proses pelatihan model.
5. [**Evaluation**](Evaluation_jatim.md): Menguji keandalan model pada 90 poligon uji independen, visualisasi *Confusion Matrix Heatmap*, analisis laporan klasifikasi (*precision, recall, f1-score*), analisis derajat keyakinan fuzzy ($\mu_{\max}$), serta audit transparan sampel yang mengalami misklasifikasi pada zona transisi.
6. [**Deployment & Visualisasi Interaktif**](Deployment_Visualisasi_jatim.md): Merancang studio visualisasi geospasial interaktif (*dual-map studio*) berbasis Folium & Leaflet.js yang menyajikan evaluasi poligon ground truth (Peta 1) dan peta klasifikasi tutupan lahan skala regional kontinu berbasis *ImageOverlay* raster logika fuzzy (Peta 2) di atas citra satelit Esri World Imagery & Google Hybrid, serta integrasi dashboard aplikasi web Streamlit Cloud.
7. [**Kesimpulan & Rekomendasi Kebijakan**](Kesimpulan_Rekomendasi_jatim.md): Menyimpulkan temuan teknis model sains data serta merumuskan rekomendasi kebijakan spasial strategis bagi Pemerintah Provinsi Jawa Timur terkait perlindungan Lahan Pertanian Pangan Berkelanjutan (LP2B), pengendalian aglomerasi perkotaan Gerbangkertosusila, dan konservasi sabuk hijau mangrove pesisir.
8. [**Panduan Deploy ke Streamlit**](Panduan_Deploy_Streamlit.md): Panduan teknis langkah demi langkah (*step-by-step*) deployment aplikasi dashboard geospasial interaktif ke server lokal hingga publikasi daring di Streamlit Community Cloud via integrasi repositori GitHub.
