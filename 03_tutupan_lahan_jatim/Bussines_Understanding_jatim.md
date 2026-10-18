# Business Understanding: Tutupan Lahan Jawa Timur

Tahap **Business Understanding** merupakan fase pertama dan fondasi terpenting dalam kerangka kerja metodologi *Cross-Industry Standard Process for Data Mining* (CRISP-DM). Pada tahap ini, proyek diorientasikan pada pemahaman mendalam mengenai urgensi spasial, ekologi, dinamika alih fungsi lahan di Provinsi Jawa Timur, serta perumusan sasaran sains data spasial yang terukur dan aplikatif bagi pembuat kebijakan (*stakeholders*).

---

## 1. Latar Belakang & Urgensi Analisis Spasial Jawa Timur

Provinsi Jawa Timur memegang posisi strategis sebagai lumbung pangan nasional utama Republik Indonesia sekaligus episentrum pertumbuhan ekonomi dan industri manufaktur di kawasan timur Nusantara. Berdasarkan data Badan Pusat Statistik (BPS), Jawa Timur menyumbang lebih dari $17\%$ total produksi beras nasional dan memasok kebutuhan logistik pangan bagi belasan provinsi mitra di Indonesia Timur.

Namun, pesatnya laju pertumbuhan demografi, pembangunan koridor infrastruktur jalan tol, serta ekspansi kawasan industri memicu tekanan alih fungsi lahan (*land use change*) yang sangat agresif di berbagai kabupaten/kota:

1. **Konversi Lahan Sawah Produktif (*Agricultural Land Loss*):**  
   Hamparan sawah irigasi teknis di sentra pertanian (seperti Lamongan, Bojonegoro, Jember, Lumajang, dan Banyuwangi) menghadapi ancaman konversi menjadi kawasan pergudangan, perumahan komersial, dan sentra manufaktur. Fenomena ini berisiko melemahkan kedaulatan pangan regional dan nasional.
2. **Aglomerasi Perkotaan & Penjalaran Urban (*Urban Sprawl*):**  
   Kawasan metropolitan **Gerbangkertosusila** (Gresik, Bangkalan, Mojokerto, Surabaya, Sidoarjo, Lamongan) mengalami lonjakan luas permukaan kedap air (*impervious surface*). Kondisi ini mempersempit Ruang Terbuka Hijau (RTH), memicu fenomena pulau panas (*Urban Heat Island*), serta memperbesar debit limpasan air hujan yang memperparah potensi banjir perkotaan.
3. **Degradasi Hutan Lindung & Sabuk Pesisir Mangrove:**  
   Tekanan perambahan hutan di lereng pegunungan vulkanik serta konversi mangrove di sepanjang Pantai Utara (Pantura), muara sungai besar, dan Pantai Timur Surabaya (Pamurbaya) menjadi tambak tak berizin memicu degradasi biodiversitas pesisir, ancaman abrasi laut, dan hilangnya penyerap karbon biru (*blue carbon sinks*).

Pemantauan tutupan lahan secara terestrial (survei manual ke lapangan) memerlukan anggaran yang sangat besar, sumber daya manusia yang masif, dan memakan waktu berbulan-bulan. Oleh karena itu, penerapan penginderaan jauh (*remote sensing*) berbasis satelit optik multispektral **Sentinel-2A** dipadukan dengan algoritma **Machine Learning** menjadi instrumen esensial dalam tata kelola wilayah modern berbasis bukti saintifik empiris (*evidence-based policy*).

---

## 2. Tujuan Analisis Land Use and Land Cover (LULC)

Tujuan utama pelaksanaan analisis LULC Provinsi Jawa Timur ini mencakup 5 target strategis:

1. **Identifikasi Akurat Objek Spasial**: Mengklasifikasikan 6 kelas tutupan lahan acuan nasional standar geospasial Indonesia:
   - **Sawah**: Lahan pertanian basah beririgasi teknis maupun non-teknis.
   - **Bangunan**: Kawasan permukiman, industri, gedung perkotaan, dan infrastruktur terbangun.
   - **Hutan / Lahan Hijau**: Tutupan kanopi pohon rapat, hutan lindung pegunungan, dan vegetasi permanen.
   - **Danau / Waduk**: Badan air tawar pedalaman, ranu alami, dan waduk buatan pengendali banjir/irigasi.
   - **Laut / Perairan Terbuka**: Area kelautan lepas dan perairan pesisir pantai.
   - **Mangrove**: Komunitas vegetasi halofit di zona transisi pasang surut air laut (*intertidal zone*).

2. **Pencegahan Kebocoran Data Spasial (*Spatial Data Leakage Prevention*)**:  
   Membagi dataset latih dan uji berbasis kelompok `poligon_id` unik (*Spatial Group Splitting*) menggunakan modul `GroupShuffleSplit`, memastikan seluruh piksel data uji berasal dari lokasi poligon yang benar-benar mandiri dan belum pernah dilihat model pada fase pelatihan (mematuhi Hukum Pertama Tobler).

3. **Komparasi Model Menghasilkan Akurasi Tertinggi**:  
   Melakukan benchmarking saintifik terhadap 3 arsitektur model Machine Learning:
   - **Model 1: Random Forest Classifier** (Baseline standar penginderaan jauh dengan pembobotan seimbang `class_weight='balanced'`).
   - **Model 2: HistGradientBoosting Classifier** (Algoritma gradient boosting berbasis kuantisasi histogram setara LightGBM).
   - **Model 3: Ensemble Soft-Voting Classifier (RF + HGB)** (Menggabungkan rata-rata probabilitas posterior kedua model guna menghasilkan akurasi global dan Macro F1-Score tertinggi).

4. **Visualisasi Interaktif Berstandar Kartografi**:  
   Menyajikan hasil klasifikasi ke dalam peta interaktif berbasis Leaflet dan Folium yang didukung multi-basemap: **Google Satellite WMS**, **Google Hybrid**, dan **Esri World Imagery**.

5. **Dukungan Kebijakan Tata Ruang Berbasis Data**:  
   Menyediakan lapisan spasial presisi tinggi untuk instrumen kebijakan strategis Pemprov Jawa Timur: penetapan peta zonasi **LP2B (Lahan Pertanian Pangan Berkelanjutan)**, perlindungan sempadan danau/waduk, serta zonasi konservasi sabuk mangrove pesisir.

---

## 3. Pemangku Kepentingan (*Stakeholders*) & Penerima Manfaat

Sistem informasi geospasial ini dirancang untuk melayani berbagai institusi pemangku kebijakan:

* **Badan Perencanaan Pembangunan Daerah (Bappeda) Provinsi Jawa Timur:**  
  Sebagai data intelijen spasial untuk mengevaluasi kesesuaian implementasi Rencana Tata Ruang Wilayah (RTRW) Provinsi Jawa Timur dan mendeteksi laju alih fungsi lahan lintas kabupaten/kota.
* **Dinas Pertanian dan Ketahanan Pangan Provinsi Jawa Timur:**  
  Sebagai basis pengawasan penegakan batas Lahan Pertanian Pangan Berkelanjutan (LP2B) guna mencegah konversi ilegal sawah irigasi teknis melalui integrasi perizinan *Online Single Submission* (OSS).
* **Dinas Lingkungan Hidup dan Dinas Kehutanan Provinsi Jawa Timur:**  
  Sebagai instrumen monitoring restorasi ekosistem mangrove pesisir, pengawasan cadangan karbon biru (*blue carbon*), dan pemantauan perambahan kawasan hutan lindung pegunungan.
* **Kementerian Agraria dan Tata Ruang / Badan Pertanahan Nasional (ATR/BPN):**  
  Sebagai data komparasi pemutakhiran Peta Penggunaan Tanah (PT) skala regional yang objektif dan terbarukan secara berkala.

---

## 4. Kriteria Keberhasilan Sains Data (*Data Science Success Criteria*)

Kriteria evaluasi keberhasilan proyek sains data ini diukur secara kuantitatif dan kualitatif:

| Parameter Evaluasi | Target Kriteria Keberhasilan | Realisasi Capaian Proyek | Status |
| :--- | :---: | :---: | :---: |
| **Akurasi Global Data Uji (*Overall Accuracy*)** | $\ge 85.0\%$ | **89.06%** (pada 57.855 piksel uji mandiri) | ✅ Terlampaui |
| **Macro F1-Score (Evaluasi Setara Seluruh Kelas)** | $\ge 78.0\%$ | **81.21%** (Macro Average F1) | ✅ Terlampaui |
| **Presisi Kelas Minoritas (Bangunan)** | $\ge 70.0\%$ | **71.93%** (Presisi Bangunan pada data uji) | ✅ Terlampaui |
| **Sensitivitas Deteksi Mangrove** | $\ge 80.0\%$ | **86.53%** (Recall Mangrove pada data uji) | ✅ Terlampaui |
| **Integritas Metodologi Spasial** | Bebas *Spatial Data Leakage* | `GroupShuffleSplit` berbasis kelompok `poligon_id` unik | ✅ Terpenuhi |
| **Aksesibilitas & Visualisasi Peta** | Multi-basemap WMS Satelit | Google Satellite WMS, Google Hybrid, & Esri Imagery di Folium | ✅ Terpenuhi |
