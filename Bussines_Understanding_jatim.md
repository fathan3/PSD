# Business Understanding: Tutupan Lahan Jawa Timur

Tahap **Business Understanding** merupakan fondasi terpenting dalam kerangka kerja metodologi *Cross-Industry Standard Process for Data Mining* (CRISP-DM). Pada tahap ini, proyek diorientasikan pada pemahaman mendalam mengenai dinamika spasial, ekologi, dan tantangan kebijakan alih fungsi lahan di wilayah Provinsi Jawa Timur, serta perumusan sasaran sains data spasial yang terukur.

---

## 1. Latar Belakang & Urgensi Lingkungan Provinsi Jawa Timur

Provinsi Jawa Timur memegang peranan krusial sebagai lumbung pangan utama nasional Republik Indonesia sekaligus pusat pertumbuhan ekonomi dan logistik terbesar di wilayah timur Nusantara. Berdasarkan data Badan Pusat Statistik (BPS), Jawa Timur menyumbang lebih dari $17\%$ produksi beras/padi nasional dan menopang pasokan logistik pangan untuk belasan provinsi di kawasan Indonesia Timur.

Namun, pesatnya laju pertumbuhan demografi, pembangunan infrastruktur transportasi, dan ekspansi industri manufaktur memicu tekanan konversi lahan (*land conversion pressure*) yang sangat eksponensial di berbagai kabupaten dan kota:

1. **Ancaman Alih Fungsi Sawah Produktif Beririgasi Teknis:**
   Kawasan lumbung padi strategis beririgasi teknis (seperti di Kabupaten Banyuwangi, Jember, Lumajang, Lamongan, dan Sidoarjo) menghadapi ancaman konversi masif menjadi perumahan komersial, pergudangan, dan zona industri. Fenomena ini berpotensi merongrong kedaulatan pangan regional maupun nasional.
2. **Ekspansi Urban Tak Terkendali (*Urban Sprawl*):**
   Koridor aglomerasi megapolitan **Gerbangkertosusila** (Gresik, Bangkalan, Mojokerto, Surabaya, Sidoarjo, Lamongan) mengalami kenaikan proporsi permukaan kedap air (*impervious surface*) secara masif. Kondisi ini mempersempit ketersediaan Ruang Terbuka Hijau (RTH), meningkatkan fenomena pulau panas perkotaan (*Urban Heat Island*), dan memperbesar debit puncak limpasan air hujan yang memicu genangan banjir tahunan.
3. **Degradasi Ekosistem Pesisir & Mangrove:**
   Sabuk hijau bakau di sepanjang pesisir Pantai Utara (Pantura), muara sungai besar (seperti Kali Porong dan Bengawan Solo), serta Pantai Timur Surabaya (Pamurbaya) menghadapi ancaman deforestasi dan intrusi tambak ikan tak berizin. Hal ini meningkatkan risiko abrasi air laut, penurunan keanekaragaman hayati estuari, serta hilangnya rosot penyerap karbon biru (*blue carbon sinks*).

---

## 2. Rumusan Masalah Sains Data Spasial

Pemantauan tutupan lahan secara terestrial lapangan (*ground survey*) atau manual memerlukan biaya operasional yang sangat mahal, rentang waktu yang lama, dan memiliki keterbatasan fisik untuk menjangkau seluruh bentang alam Provinsi Jawa Timur yang luasnya mencapai lebih dari $47.800\text{ km}^2$. 

Penginderaan jauh (*remote sensing*) berbasis satelit optik multispektral menjadi solusi paling efektif dan objektif. Namun demikian, pendekatan klasifikasi konvensional menghadapi tantangan teknis mendasar:

* **Kelemahan Model Klasifikasi Kaku (*Crisp / Hard Classifier*):**
  Model *machine learning* konvensional seperti Decision Tree, Random Forest standar, atau SVM memaksakan setiap piksel atau entitas spasial ke dalam satu kelas tunggal secara biner ($0$ atau $1$).
* **Masalah Batas Kabur Spasial (*Spatial Vagueness & Mixed Pixels*):**
  Di alam nyata, tutupan lahan bersifat kontinu dan dinamis. Batas antara sawah basah beririgasi dengan badan rawa/danau, atau antara perairan payau dengan kanopi mangrove pesisir, membentuk zona transisi alami (*ecotone*). Memaksakan klasifikasi biner diskret pada area percampuran ini menyebabkan galat klasifikasi (*misclassification*) yang tinggi dan mengabaikan informasi gradasi biofisik lingkungan.

---

## 3. Sasaran Proyek & Pemangku Kepentingan (*Stakeholders*)

### Sasaran Utama Proyek:
1. **Membangun Model Klasifikasi Spasial Berbasis Logika Fuzzy Terawasi (*Supervised Fuzzy Logic Classifier*):**
   Mengembangkan model klasifikasi multi-kelas yang mampu memetakan derajat keanggotaan spektral $\mu \in [0, 1]$ pada setiap kelas tutupan lahan untuk merepresentasikan spektrum gradasi objek alam secara transparan.
2. **Menerapkan Metodologi Validasi Spasial Bebas Kebocoran Data (*Spatial Autocorrelation Leakage Prevention*):**
   Menerapkan agregasi poligon centroid mandiri agar pemisahan data latih dan uji tidak terdistorsi oleh korelasi spasial lokal (Hukum Pertama Tobler).
3. **Menyediakan Antarmuka Sistem Informasi Geospasial Interaktif (*Interactive Dual-Map Studio*):**
   Mendeploy platform dasbor analitik berbasis web yang dapat diakses secara publik dan *real-time* oleh pembuat kebijakan dan masyarakat luas.

### Pemangku Kepentingan (*Stakeholders*) yang Menerima Manfaat:
* **Badan Perencanaan Pembangunan Daerah (Bappeda) Provinsi Jawa Timur:**
  Sebagai instrumen pemantauan kepatuhan pelaksanaan Rencana Tata Ruang Wilayah (RTRW) Provinsi Jawa Timur 2023–2043 dan evaluasi neraca perubahan penggunaan lahan.
* **Dinas Pertanian dan Ketahanan Pangan Provinsi Jawa Timur:**
  Sebagai alat pengawasan geointelijen penegakan zonasi Lahan Pertanian Pangan Berkelanjutan (LP2B) untuk melindungi sawah irigasi teknis dari alih fungsi ilegal.
* **Dinas Lingkungan Hidup dan Dinas Kehutanan:**
  Sebagai platform pemantauan restorasi sabuk hijau mangrove pesisir, deteksi deforestasi kawasan hutan lindung, dan inventarisasi cadangan karbon alami.
* **Kementerian Agraria dan Tata Ruang / Badan Pertanahan Nasional (ATR/BPN):**
  Sebagai data acuan pemutakhiran Peta Penggunaan Tanah (PT) skala regional berbasis citra satelit terkini.

---

## 4. Kriteria Keberhasilan Sains Data (*Data Science Success Criteria*)

Keberhasilan proyek sains data spasial ini dinilai berdasarkan indikator capaian kuantitatif dan kualitatif berikut:

* **Akurasi Klasifikasi Data Uji (*Overall Accuracy*):** $\ge 90.0\%$ pada data uji poligon independen.
* **Indeks Keselarasan Cohen's Kappa ($\kappa$):** $\ge 0.85$ (termasuk ke dalam kategori keselarasan *Almost Perfect Agreement*).
* **Derajat Keyakinan Model (*Confidence Score* $\mu_{\max}$):** Rata-rata derajat keanggotaan maksimum $\mu_{\max} > 0.80$ pada sampel terprediksi benar, menunjukkan ketegasan keputusan defuzzifikasi.
* **Separabilitas Ruang Spektral:** Mampu memisahkan secara jelas sebaran multivariat antara vegetasi hutan lebat, vegetasi sawah musiman, kanopi mangrove, perairan darat/laut, dan kawasan permukiman terbangun.
* **Kesiapan Akses Operasional:** Seluruh pipeline sistem berhasil dipublikasikan ke peladen awan (*cloud deployment*) dan dapat diakses publik dengan waktu muat responsif.
