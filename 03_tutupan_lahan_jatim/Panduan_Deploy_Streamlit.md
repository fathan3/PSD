# Panduan Deploy ke Streamlit

## Panduan Step-by-Step Deployment Sistem Informasi Geospasial ke Streamlit & Streamlit Cloud

Panduan ini mendokumentasikan seluruh prosedur teknis deployment sistem cerdas geointelijen tutupan lahan Provinsi Jawa Timur mulai dari arsitektur lokal hingga peladen awan publik (*cloud deployment*).

### Ringkasan Eksekutif & Metadata Deployment
* **Nama Sistem:** Sistem Informasi Geografis (SIG) Tutupan Lahan Jawa Timur
* **Basis Platform Web:** Streamlit Web Framework (`streamlit>=1.30.0`)
* **Arsitektur Model:** Ensemble Soft-Voting Classifier (Random Forest + HistGradientBoosting)
* **Layanan Cloud Hosting:** **Streamlit Community Cloud** (PaaS Linux Container)
* **Integrasi CI/CD:** GitHub Continuous Deployment via Repositori Publik
* **Tautan Aplikasi Live Publik:** [sig-tutupan-lahan-jawa-timur.streamlit.app](https://sig-tutupan-lahan-jawa-timur.streamlit.app/)
* **Repositori GitHub:** [github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur](https://github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur)

---

## 1. Arsitektur Sistem & Strategi Rendering Geospasial

### 1.1 Mengapa Memilih Streamlit untuk Aplikasi Geospasial?
Streamlit merupakan framework aplikasi web berbasis Python murni yang sangat efisien untuk sains data dan analisis geospasial karena:
1. **Ekosistem Python Native:** Menghubungkan secara langsung pustaka komputasi numerik (`NumPy`, `Pandas`, `Scikit-Learn`, `Joblib`) dengan antarmuka web interaktif tanpa memerlukan backend terpisah seperti Flask, FastAPI, atau Django.
2. **Reaktivitas Cepat:** Setiap perubahan parameter filter di antarmuka memicu rendering ulang yang cepat dengan mekanisme pemodelan data caching (`@st.cache_data` dan `@st.cache_resource`).
3. **Kompatibilitas Komponen Geospasial:** Mendukung embedding peta HTML interaktif yang dihasilkan oleh Leaflet/Folium secara mulus.

---

### 1.2 Strategi Rendering Peta: Mengapa IFrame Mandiri Lebih Unggul?
Dalam menampilkan peta interaktif Folium di Streamlit, terdapat dua pendekatan:

| Parameter Evaluasi | Modul Pihak Ketiga (`streamlit-folium`) | Peta HTML Mandiri via `components.v1.html` *(Dipilih)* |
|:---|:---|:---|
| **Stabilitas Rendering** | Rentan terjadi re-rendering loop setiap kali peta digeser atau di-zoom. | **Sangat Stabil**, berkas HTML di-render sebagai dokumen mandiri terisolasi di dalam sandbox iframe. |
| **Kinerja Memori** | Mengirimkan seluruh event GeoJSON bolak-balik antara frontend dan server. | **Sangat Ringan**, interaksi peta dieksekusi langsung di browser klien tanpa membebani server backend. |
| **Dependensi Tambahan** | Memerlukan instalasi paket eksternal tambahan. | **Nol Dependensi Tambahan**, merupakan fungsi bawaan inti dari Streamlit standard library. |

---

### 1.3 Alur Kerja Pipeline Data ke Web

Berikut adalah siklus alur data dari citra satelit mentah hingga disajikan di browser pengguna:

```
+------------------------------------+
| Citra Satelit Sentinel-2A Level-2A |
| (Copernicus CDSE BOA Reflectance)  |
+------------------------------------+
                  │
                  ▼
+------------------------------------+
| Ekstraksi Vektor & Rekayasa Fitur  |
| (NumPy, Pandas: B02-B11, 4 Indeks) |
+------------------------------------+
                  │
                  ▼
+------------------------------------+
| Model Ensemble Soft-Voting (RF+HGB)|
| (Akurasi 89.06%, Macro F1 81.21%)  |
+------------------------------------+
                  │
                  ▼
+------------------------------------+
| Generator Peta Folium & WMS Google |
| (peta_evaluasi.html, regional.html)|
+------------------------------------+
                  │
                  ▼
+------------------------------------+
| Dasbor Web Interaktif (app.py)     |
| (Streamlit Localhost:8501)         |
+------------------------------------+
                  │
                  ▼
+------------------------------------+
| Deployment Streamlit Cloud (Linux) |
| (sig-tutupan-lahan-jawa-timur.app) |
+------------------------------------+
```

---

## 2. Inventarisasi Aset & Struktur Direktori Proyek

Sebelum mempublikasikan aplikasi ke lingkungan produksi, seluruh file struktur proyek diorganisasikan secara rapi:

### Tabel Audit Kelengkapan Berkas:

| Kategori | Nama Berkas / Direktori | Peran & Fungsi dalam Sistem |
|:---|:---|:---|
| **Aplikasi Utama** | `app.py` | Kode sumber utama antarmuka pengguna, navigasi tab, filter data, dan visualisasi. |
| **Dependensi** | `requirements.txt` | Daftar pustaka Python yang wajib diinstal otomatis oleh peladen Streamlit Cloud. |
| **Konfigurasi Tema** | `.streamlit/config.toml` | Konfigurasi palet warna visual bawaan sistem (Cobalt Blue theme). |
| **Peta Interaktif 1** | `peta_evaluasi_poligon.html` | Berkas peta mandiri evaluasi poligon ground truth (Convex Hull) dan penanda salah klasifikasi. |
| **Peta Interaktif 2** | `peta_regional_jawatimur.html` | Berkas peta mandiri klasifikasi regional Jawa Timur (ImageOverlay 6 layer mandiri) di atas Google Satellite WMS, Hybrid, dan Esri Imagery. |
| **Artefak Model** | `hasil_evaluasi/model_ensemble.joblib` | Berkas model Machine Learning Ensemble Soft-Voting terlatih. |
| **Dataset Evaluasi** | `hasil_evaluasi/evaluasi_poligon_ground_truth.csv` | Tabel data poligon evaluasi ground truth berisi koordinat, kelas aktual, prediksi, dan akurasi. |

---

## 3. Rancang Bangun Kode Aplikasi Antarmuka (`app.py`)

Aplikasi `app.py` mengimplementasikan fitur-fitur analitik spasial utama:

1. **Pengaturan Tata Letak Lebar Penuh (*Wide Mode Layout*):**
   ```python
   import streamlit as st
   st.set_page_config(
       page_title="Geointelijen Tutupan Lahan Jawa Timur | Sentinel-2A & Machine Learning",
       layout="wide",
       initial_sidebar_state="expanded"
   )
   ```
2. **Kustomisasi Tema Visual (*Custom CSS Theme*):**
   Menyuntikkan style CSS modern dengan tipografi bersih, badge status, dan kartu metrik KPI berbayang lembut.
3. **Sidebar Konsol Spasial:**
   Menyajikan navigasi kontrol, ringkasan metadata citra satelit Sentinel-2A, serta legenda interaktif 6 kelas tutupan lahan dengan palet warna resmi:
   * **Sawah:** `#FFD92F` (Kuning)
   * **Bangunan:** `#E41A1C` (Merah)
   * **Hutan:** `#2E7D32` (Hijau Tua)
   * **Danau:** `#4FC3F7` (Biru Muda)
   * **Laut:** `#0D47A1` (Biru Tua)
   * **Mangrove:** `#8E44AD` (Ungu)
4. **Struktur Tab Dasbor Analitik:**
   * **Tab Peta Geospasial Interaktif:** Pemilih mode antara Peta 1 (Studio Evaluasi Poligon Ground Truth) dan Peta 2 (Klasifikasi Regional Raster 250m dengan 6 layer ImageOverlay mandiri) dengan iframe responsif dan multi-basemap WMS Google Satellite.
   * **Tab Analisis Statistik & Evaluasi:** Visualisasi metrik model (Akurasi 89.06%, Macro F1 81.21%), heatmap Confusion Matrix, dan Feature Importance.
   * **Tab Eksplorasi Data & Unduh Hasil:** Tabel filter berdasarkan Kabupaten/Kota dan kelas tutupan lahan dengan tombol unduh CSV.

---

## 4. Konfigurasi Dependensi & Tema Sistem

### 4.1 File `requirements.txt`
File `requirements.txt` memberi tahu pengelola paket otomatis di Streamlit Cloud pustaka apa saja yang wajib dipasang:

```text
streamlit>=1.30.0
pandas>=2.0.0
numpy>=1.24.0
pillow>=9.5.0
matplotlib>=3.7.0
scikit-learn>=1.2.0
joblib>=1.3.0
folium>=0.14.0
```

### 4.2 File `.streamlit/config.toml`
File konfigurasi tema tingkat sistem mengatur palet warna dasar agar konsisten di seluruh perangkat pengguna:

```toml
[theme]
primaryColor = "#2563eb"              # Electric Cobalt Blue (Warna Aksen Tombol & Slider)
backgroundColor = "#ffffff"           # Latar Belakang Putih Bersih
secondaryBackgroundColor = "#f8fafc"  # Abu-abu Slate Terang Lembut untuk Sidebar & Tab
textColor = "#0f172a"                 # Teks Obsidian Kontras Tinggi
font = "sans serif"

[server]
maxUploadSize = 200
enableCORS = false
enableXsrfProtection = true
```

---

## 5. Pengujian & Eksekusi di Komputer Lokal (*Local Run*)

Sebelum mempublikasikan repositori ke GitHub, dilakukan pengujian fungsional pada lingkungan pengembangan lokal:

### 5.1 Perintah Eksekusi Terminal
Jalankan perintah berikut pada terminal PowerShell:

```bash
streamlit run app.py
```

### 5.2 Alamat Akses Lokal
Server Streamlit secara otomatis mengaktifkan endpoint:
* **Local URL:** `http://localhost:8501`
* **Network URL:** `http://192.168.x.x:8501` (dapat diakses dari gawai lain pada jaringan Wi-Fi lokal yang sama)

### 5.3 Checklist Pengujian Fungsional Lokal (*Local QA Checklist*):
1. [x] **Hero Banner & KPI Metrics:** Menampilkan metrik akurasi 89.06% dan Macro F1 81.21% secara akurat.
2. [x] **Sidebar Konsol Spasial:** Menampilkan 6 kelas tutupan lahan dengan kotak indikator warna resmi yang tepat.
3. [x] **Tab 1 (Peta):** Mampu berganti secara mulus antara Peta 1 (Poligon Evaluasi Ground Truth) dan Peta 2 (Klasifikasi Regional Raster 250m ImageOverlay).
4. [x] **Tab 2 (Evaluasi):** Grafik Confusion Matrix dan Feature Importance termuat tanpa error.
5. [x] **Tab 3 (Eksplorasi):** Fitur pemfilteran poligon dan tabel data responsif dengan tombol unduh CSV.

---

## 6. Alur Deployment ke Streamlit Community Cloud (Step-by-Step)

Streamlit Community Cloud memungkinkan aplikasi Python disajikan secara publik dan gratis di seluruh dunia dengan integrasi *Continuous Deployment* (CI/CD) langsung dari GitHub.

### Tahap 1: Sinkronisasi Repositori GitHub
Pastikan seluruh file proyek telah di-commit dan di-push ke cabang utama (`main`) di GitHub:

```bash
git add .
git commit -m "feat: complete geospatial land cover classification app with ensemble model"
git push origin main
```

### Tahap 2: Akses Streamlit Community Cloud
1. Buka peramban web dan kunjungi portal resmi [share.streamlit.io](https://share.streamlit.io/).
2. Masuk menggunakan autentikasi akun GitHub Anda (**fathan3**).
3. Izinkan hak akses integrasi repositori publik (*OAuth authorization*).

### Tahap 3: Pembuatan Aplikasi Baru (*Create App*)
1. Klik tombol **"Create app"** di sudut kanan atas dasbor.
2. Pilih opsi **"Yup, I have an app"**.
3. Lengkapi formulir parameter deployment berikut:
   * **Repository:** `fathan3/SIG-Tutupan-Lahan-Jawa-Timur`
   * **Branch:** `main`
   * **Main file path:** `app.py`
   * **App URL (Custom Subdomain):** `sig-tutupan-lahan-jawa-timur`  
     *(menghasilkan domain khusus `https://sig-tutupan-lahan-jawa-timur.streamlit.app`)*
4. Klik tombol **Deploy!**

### Tahap 4: Proses Build Otomatis di Cloud
Server container Linux Streamlit Cloud akan secara otomatis:
1. Mengkloning repositori dari GitHub: `git clone ...`
2. Memasang lingkungan virtual terisolasi menggunakan paket manajer `uv`.
3. Membaca dan menginstal seluruh pustaka yang didefinisikan pada `requirements.txt`.
4. Menjalankan server aplikasi di port 8501 dengan proxy SSL/HTTPS otomatis.

---

## 7. Verifikasi Live Testing & Akses Publik Aplikasi

Aplikasi geointelijen tutupan lahan Jawa Timur telah berhasil dipublikasikan secara live dan beroperasi penuh:

* **Tautan Aplikasi Produksi:** [sig-tutupan-lahan-jawa-timur.streamlit.app](https://sig-tutupan-lahan-jawa-timur.streamlit.app/)
* **Repositori Kode Sumber GitHub:** [github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur](https://github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur)
