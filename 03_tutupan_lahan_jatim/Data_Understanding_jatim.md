# Data Understanding: Citra Satelit Sentinel-2A & Fitur Spektral

Tahap **Data Understanding** berfokus pada eksplorasi sumber data citra penginderaan jauh (*remote sensing*), spesifikasi saluran multispektral satelit Sentinel-2A Level-2A, pendefinisian taksonomi 6 kelas tutupan lahan wilayah Jawa Timur, serta formulasi rekayasa 4 indeks komposit spektral biofisik.

---

## 1. Spesifikasi Citra Satelit Sentinel-2A Level-2A

Dataset dikoleksi dari sensor optik multispektral MSI (*MultiSpectral Instrument*) wahana antariksa **Sentinel-2A** milik Badan Antariksa Eropa (*European Space Agency* / ESA) produk **Level-2A (Bottom-of-Atmosphere / BOA Surface Reflectance)** melalui platform resmi **Copernicus Data Space Ecosystem (CDSE)**.

Citra produk Level-2A telah terkoreksi secara radiometrik dan atmosferik menggunakan algoritma kalibrasi *Sen2Cor*. Nilai piksel mencerminkan nilai reflektansi permukaan bumi sebenarnya yang terbebas dari bias hamburan molekul gas atmosfer (*Rayleigh scattering*) dan aerosol.

### Tabel Spesifikasi Saluran Spektral Sentinel-2A MSI:

| Band | Panjang Gelombang Pusat ($\lambda$ nm) | Resolusi Spasial (m) | Karakteristik Spektral & Fungsi Fisis |
| :--- | :---: | :---: | :--- |
| **B01** | 443 | 60 | *Coastal aerosol*, pemantauan partikel aerosol atmosfer dan pesisir |
| **B02** | 490 | 10 | **Blue**: Penetrasi kolom air dangkal dan hamburan Rayleigh atmosfer |
| **B03** | 560 | 10 | **Green**: Puncak pantulan klorofil vegetasi tampak; pembeda air dan vegetasi |
| **B04** | 665 | 10 | **Red**: Serapan klorofil tanaman fotosintetik; kontras vegetasi vs tanah |
| **B05** | 705 | 20 | *Vegetation Red Edge 1*, batas tepi serapan klorofil daun |
| **B06** | 740 | 20 | *Vegetation Red Edge 2*, analisis status nutrisi dan kanopi tanaman |
| **B07** | 783 | 20 | *Vegetation Red Edge 3*, transisi sensitif ke pantulan inframerah dekat |
| **B08** | 842 | 10 | **NIR (Near Infrared)**: Pantulan tinggi oleh struktur mesofil daun; diserap mutlak oleh air |
| **B8A** | 865 | 20 | *Narrow NIR*, kalibrasi uap air dan analisis biomassa kanopi |
| **B09** | 945 | 60 | *Water vapour*, deteksi kandungan uap air di kolom atmosfer |
| **B11** | 1610 | 20 (resample 10m) | **SWIR-1**: Sensitif terhadap kelembaban daun & tanah; kunci deteksi atap/bangunan |
| **B12** | 2190 | 20 | *SWIR-2*, deteksi geologi, batuan, dan kandungan mineral tanah |

Fitur input utama saluran reflektansi dasar yang digunakan dalam analisis ini adalah 5 saluran reflektansi: **B02 (Blue), B03 (Green), B04 (Red), B08 (NIR), dan B11 (SWIR-1)**.

---

## 2. Formulasi Matematis Indeks Spektral (Fitur Komposit)

Untuk memaksimalkan daya pisah (*separability*) antar-objek spasial pada ruang fitur multidimensi, dihitung 4 indeks komposit fisis berbasis rasio pita spektral:

### 1. Normalized Difference Vegetation Index (NDVI)
$$\text{NDVI} = \frac{\text{B08} - \text{B04}}{\text{B08} + \text{B04}}$$
* **Tujuan Fisis**: Mengukur kehijauan, viabilitas klorofil, dan kerapatan kanopi daun aktif fotosintesis. Nilai tinggi ($>0.50$) mengindikasikan vegetasi lebat (hutan lindung atau hamparan sawah subur), sedangkan air bernilai negatif.

### 2. Normalized Difference Water Index (NDWI - McFeeters, 1996)
$$\text{NDWI} = \frac{\text{B03} - \text{B08}}{\text{B03} + \text{B08}}$$
* **Tujuan Fisis**: Menegaskan keberadaan badan air terbuka daratan maupun lautan. Air memiliki reflektansi tinggi di pita Hijau (B03) dan menyerap radiasi Inframerah Dekat (B08) hampir sempurna, sehingga badan air memiliki nilai NDWI positif ($>0$), sedangkan daratan bernilai negatif.

### 3. Modified Normalized Difference Water Index (MNDWI - Xu, 2006)
$$\text{MNDWI} = \frac{\text{B03} - \text{B11}}{\text{B03} + \text{B11}}$$
* **Tujuan Fisis**: Menyempurnakan pemetaan perairan di kawasan sekitar permukiman dan perkotaan. Pita SWIR-1 (B11) menekan sinyal bising dari pantulan material bangunan, sehingga danau/waduk perkotaan terpisahkan secara tegas dari atap gedung.

### 4. Normalized Difference Built-up Index (NDBI - Zha et al., 2003)
$$\text{NDBI} = \frac{\text{B11} - \text{B08}}{\text{B11} + \text{B08}}$$
* **Tujuan Fisis**: Menyorot material anorganik kawasan terbangun (atap genteng, seng, beton, aspal). Kawasan terbangun memiliki pantulan di pita SWIR (B11) yang lebih tinggi daripada NIR (B08), sehingga NDBI bernilai positif pada perkotaan dan bernilai negatif pada vegetasi lebat maupun air.

---

## 3. Taksonomi 6 Kelas Tutupan Lahan Wilayah Jawa Timur

Berdasarkan pedoman klasifikasi penutup lahan standar nasional dan karakteristik biogeografi regional Jawa Timur, objek tutupan lahan dikelompokkan ke dalam 6 kelas acuan:

| No | Label Numerik | Nama Kelas Tutupan Lahan | Kode Warna Resmi | Karakteristik Biofisik & Sebaran Lapangan di Jawa Timur | Respon Spektral Kunci |
|:--:|:---:|:---|:---:|:---|:---|
| 1 | 1 | **Sawah** | `#FFD92F` | Kawasan pertanian padi sawah irigasi teknis di dataran aluvial (Lamongan, Bojonegoro, Jember, Lumajang, Banyuwangi). | NIR sedang–tinggi saat vegetatif; SWIR meningkat saat bera; NDVI berkisar $0.50 - 0.70$. |
| 2 | 2 | **Bangunan** | `#E41A1C` | Kawasan terbangun perkotaan Surabaya, Malang, Sidoarjo, zona industri, perumahan, perkerasan aspal/beton. | NDBI positif ($>0.10$); reflektansi SWIR (B11) & Red (B04) tinggi; NDVI rendah ($<0.20$). |
| 3 | 3 | **Hutan** | `#2E7D32` | Kanopi pohon lebat hutan lindung pegunungan vulkanik (TN Bromo Tengger Semeru, Ijen, Raung, Arjuno-Welirang). | Penyerapan kuat Red (B04); pantulan puncak NIR (B08 $>2500$); NDVI sangat tinggi ($>0.75$). |
| 4 | 4 | **Danau** | `#4FC3F7` | Badan air tawar daratan tenang, waduk irigasi/PLTA (Waduk Karangkates, Sutami, Selorejo, Wonorejo, Ranu Kumbolo). | Penyerapan mutlak inframerah; NDWI positif; reflektansi B08 dan B11 teredam. |
| 5 | 5 | **Laut** | `#0D47A1` | Perairan laut terbuka di Laut Jawa, Selat Madura, Selat Bali, dan Samudera Hindia di pesisir selatan. | Hamburan kuat pita pendek Blue (B02); penyerapan gelombang inframerah total; NDWI & MNDWI positif tinggi. |
| 6 | 6 | **Mangrove** | `#8E44AD` | Vegetasi halofit di zona intertidal pesisir pasang surut (Wonorejo Surabaya, Ujungpangkah Gresik, muara Porong, Teluk Pangpang). | Kanopi vegetasi hijau rapat (NIR tinggi) yang berlatar belakang substrat lumpur basah/air (SWIR teredam). |
