# Data Understanding: Citra Satelit Sentinel-2A & Spektral

Tahap **Data Understanding** berfokus pada eksplorasi sumber data citra penginderaan jauh (*remote sensing*), pemahaman karakteristik spektral gelombang elektromagnetik, pendefinisian taksonomi kelas tutupan lahan wilayah studi, serta formulasi rekayasa fitur biofisik yang diturunkan dari citra satelit.

---

## 1. Spesifikasi Citra Satelit Sentinel-2A MSI Level-2A

Data spektral dalam penelitian geospasial ini bersumber dari satelit konstelasi **Sentinel-2A** milik Badan Antariksa Eropa (*European Space Agency* / ESA) melalui program Copernicus. Satelit ini membawa sensor *Multi-Spectral Instrument* (MSI) dengan orbit heliosinkron (*sun-synchronous*) pada ketinggian 786 km dan waktu lintas ulang (*revisit time*) 5 hari (bersama Sentinel-2B).

Produk citra yang digunakan adalah **Sentinel-2A Level-2A (Bottom-of-Atmosphere / BOA Surface Reflectance)**. Produk ini telah melalui kalibrasi radiometrik tingkat lanjut serta koreksi atmosferik otomatis menggunakan algoritma **Sen2Cor**. Nilai piksel pada produk Level-2A telah merepresentasikan nilai reflektansi permukaan tanah yang sebenarnya, bebas dari distorsi hamburan Rayleigh maupun absorpsi uap air di atmosfer atas.

### Karakteristik Band Multispektral Utama yang Diekstraksi:

| Band Satelit | Nama Saluran Spektral | Panjang Gelombang Pusat ($\lambda$) | Resolusi Spasial | Karakteristik Biofisik & Respon Objek |
|:---:|:---|:---:|:---:|:---|
| **B02** | Biru (*Blue*) | $490\text{ nm}$ | $10\text{ meter}$ | Memiliki penetrasi tinggi pada kolom air jernih dan peka terhadap partikel tersuspensi. Sangat baik untuk memisahkan perairan dari daratan. |
| **B03** | Hijau (*Green*) | $560\text{ nm}$ | $10\text{ meter}$ | Menangkap puncak pantulan spektrum tampak oleh pigmen klorofil tanaman sehat. Menjadi pembeda utama perairan jernih terhadap vegetasi darat. |
| **B04** | Merah (*Red*) | $665\text{ nm}$ | $10\text{ meter}$ | Merupakan zona serapan (*absorption pit*) maksimum oleh molekul klorofil-a. Menghasilkan kontras tajam antara biomassa tanaman dan tanah terbuka atau material buatan. |
| **B08** | Inframerah Dekat (*NIR*) | $842\text{ nm}$ | $10\text{ meter}$ | Dipantulkan secara masif oleh struktur spons sel mesofil internal daun vegetasi yang sehat (*green plateau*). Sebaliknya, radiasi NIR diserap hampir sempurna oleh badan air murni. |
| **B11** | Inframerah Gelombang Pendek 1 (*SWIR-1*) | $1610\text{ nm}$ | $20\text{ m} \to 10\text{ m}$ | Sangat peka terhadap kadar kelembapan tanah, kandungan air dalam kanopi biomassa, serta pantulan mineral silikat pada beton/atap kawasan perkotaan. |

---

## 2. Taksonomi 6 Kelas Tutupan Lahan Acuan di Jawa Timur

Berdasarkan kondisi biofisik wilayah Provinsi Jawa Timur dan pedoman klasifikasi penutup lahan SNI 7645:2010, objek tutupan lahan dikelompokkan ke dalam 6 kelas representatif seimbang:

| No | Kelas Tutupan Lahan | Kode | Karakteristik Wilayah & Objek Lapangan | Respon Spektral Utama |
|:--:|:--------------------|:----:|:---------------------------------------|:----------------------|
| 1 | **Sawah** | 1 | Lahan persawahan padi irigasi teknis dan tadah hujan (Kab. Banyuwangi, Jember, Lumajang, Bojonegoro). | Pantulan NIR sedang–tinggi saat fase vegetatif/generatif; pantulan SWIR meningkat saat fase bera/pasca panen. |
| 2 | **Bangunan** | 2 | Kawasan terbangun metropolitan Surabaya, perumahan padat, kawasan industri, dan perkerasan aspal/beton. | Pantulan SWIR (B11) & Red (B04) sangat tinggi; NDBI positif; nilai NDVI rendah ($< 0.20$). |
| 3 | **Lahan Hijau (Hutan)**| 3 | Kawasan hutan lindung primer/sekunder dan tutupan vegetasi lebat pegunungan (TN Bromo Tengger Semeru, Ijen, Raung). | Absorpsi kuat Red (B04); pantulan puncak NIR (B08); NDVI sangat tinggi ($> 0.75$). |
| 4 | **Danau (Ranu/Waduk)** | 4 | Badan air tawar daratan tenang (Ranu Kumbolo Semeru, Waduk Selorejo Malang, Waduk Sutami, Telaga Sarangan Magetan). | Penyerapan radiasi inframerah mutlak; pantulan NIR & SWIR sangat rendah mendekati nol; NDWI bernilai positif. |
| 5 | **Perairan Terbuka** | 5 | Laut pesisir utara (Laut Jawa), Selat Madura, dan Samudera Hindia di pesisir selatan Jawa Timur. | Dominasi hamburan spektrum pendek Blue (B02); penyerapan gelombang inframerah total; NDWI dan MNDWI sangat tinggi. |
| 6 | **Mangrove** | 6 | Ekosistem hutan bakau pesisir pasang surut (Wonorejo Surabaya, Ujungpangkah Gresik, muara Porong). | Kanopi hijau rapat (NIR tinggi) yang teredam oleh latar lumpur basah/air pasang surut (SWIR relatif rendah). |

---

## 3. Rekayasa Fitur 9 Indeks Reflektansi Spektral

Untuk meningkatkan daya pisah (*separability*) antar-kelas pada ruang fitur multidimensi, data reflektansi dasar diekspansi menjadi 9 atribut fitur utama yang mencakup 5 band reflektansi permukaan serta 4 indeks spektral ternormalisasi:

1. **NDVI (Normalized Difference Vegetation Index):**
   Mengukur kerapatan biomassa hijau dan kesehatan vegetasi berbasis kontras serapan klorofil di pita Merah (*Red*) dan hamburan di pita Inframerah Dekat (*NIR*):
   $$\text{NDVI} = \frac{\text{B08} - \text{B04}}{\text{B08} + \text{B04} + \epsilon}$$

2. **NDWI (Normalized Difference Water Index - McFeeters, 1996):**
   Mendeteksi keberadaan badan air permukaan terbuka dengan memaksimalkan pantulan pita Hijau (*Green*) dan meminimalkan serapan pita Inframerah Dekat (*NIR*):
   $$\text{NDWI} = \frac{\text{B03} - \text{B08}}{\text{B03} + \text{B08} + \epsilon}$$

3. **MNDWI (Modified Normalized Difference Water Index - Xu, 2006):**
   Meningkatkan kemampuan pemisahan badan air dari lahan terbangun dengan menggantikan pita NIR menggunakan pita Inframerah Gelombang Pendek (*SWIR-1 / B11*):
   $$\text{MNDWI} = \frac{\text{B03} - \text{B11}}{\text{B03} + \text{B11} + \epsilon}$$

4. **NDBI (Normalized Difference Built-up Index - Zha et al., 2003):**
   Menonjolkan kawasan permukiman terbangun dan perkerasan kedap air berbasis tingginya reflektansi material silikat dan beton pada pita SWIR dibandingkan NIR:
   $$\text{NDBI} = \frac{\text{B11} - \text{B08}}{\text{B11} + \text{B08} + \epsilon}$$

*(Konstanta stabilitas numerik $\epsilon = 10^{-9}$ ditambahkan pada penyebut pecahan guna menghindari terjadinya galat pembagian dengan nol).*
