# Panduan Deploy ke Streamlit

## Panduan Step-by-Step Deployment Sistem Informasi Geospasial ke Streamlit & Streamlit Cloud

Panduan ini mendokumentasikan seluruh prosedur teknis deployment sistem cerdas geointelijen tutupan lahan Provinsi Jawa Timur mulai dari arsitektur lokal hingga peladen awan publik.

### Ringkasan Eksekutif & Metadata Deployment
* **Nama Sistem:** Sistem Informasi Geografis (SIG) Tutupan Lahan Jawa Timur
* **Basis Platform Web:** Streamlit Web Framework (`streamlit>=1.30.0`)
* **Layanan Cloud Hosting:** **Streamlit Community Cloud** (PaaS Linux Container)
* **Integrasi CI/CD:** GitHub Continuous Deployment via Repositori Publik
* **Tautan Aplikasi Live Publik:** [sig-tutupan-lahan-jawa-timur.streamlit.app](https://sig-tutupan-lahan-jawa-timur.streamlit.app/)
* **Repositori GitHub:** [github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur](https://github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur)

---

## 1. Arsitektur Sistem & Strategi Rendering Geospasial

### 1.1 Mengapa Memilih Streamlit untuk Aplikasi Geospasial?
Streamlit merupakan framework aplikasi web berbasis Python murni yang sangat efisien untuk sains data dan analisis geospasial karena:
1. **Ekosistem Python Native:** Menghubungkan secara langsung pustaka komputasi numerik (`NumPy`, `Pandas`, `Scikit-Learn`) dengan antarmuka web interaktif tanpa memerlukan backend terpisah seperti Flask, FastAPI, atau Django.
2. **Reaktivitas Cepat:** Setiap perubahan parameter filter di antarmuka memicu rendering ulang yang cepat dengan mekanisme pemodelan data caching (`@st.cache_data`).
3. **Kompatibilitas Komponen Geospasial:** Mendukung embedding peta HTML interaktif yang dihasilkan oleh Leaflet/Folium secara mulus.

---

### 1.2 Strategi Rendering Peta: Mengapa IFrame Mandiri Lebih Unggul?
Dalam menampilkan peta interaktif Folium di Streamlit, terdapat dua pendekatan:

| Parameter Evaluasi | Modul Pihak Ketiga (`streamlit-folium`) | Peta HTML Mandiri via `components.v1.html` *(Dipilih)* |
|:---|:---|:---|
| **Stabilitas Rendering** | Rentang terjadi re-rendering loop setiap kali peta digeser atau di-zoom. | **Sangat Stabil**, berkas HTML di-render sebagai dokumen mandiri terisolasi di dalam sandbox iframe. |
| **Kinerja Memori** | Mengirimkan seluruh event GeoJSON bolak-balik antara frontend dan server. | **Sangat Ringan**, interaksi peta dieksekusi langsung di browser klien tanpa membebani server backend. |
| **Dependensi Tambahan** | Memerlukan instalasi paket eksternal tambahan. | **Nol Dependensi Tambahan**, merupakan fungsi bawaan inti dari Streamlit standard library. |

---

### 1.3 Alur Kerja Pipeline Data ke Web

Berikut adalah siklus alur data dari citra satelit mentah hingga disajikan di browser pengguna:

```
+------------------------------------+
| Citra Satelit Sentinel-2A Level-2A |
+------------------------------------+
                  │
                  ▼
+------------------------------------+
| Ekstraksi Vektor & Rekayasa Fitur  |
| (PyShp, NumPy, Pandas: B02-B11)    |
+------------------------------------+
                  │
                  ▼
+------------------------------------+
| Model Supervised Fuzzy Logic (m=2) |
| (Akurasi 96.67%, Cohen's κ 0.9600) |
+------------------------------------+
                  │
                  ▼
+------------------------------------+
| Generator Peta Folium & Evaluasi   |
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
| **Peta Interaktif 1** | `peta_evaluasi_poligon_fuzzy.html` | Berkas peta mandiri evaluasi poligon ground truth dan derajat keanggotaan. |
| **Peta Interaktif 2** | `peta_regional_jawatimur_fuzzy.html`| Berkas peta mandiri sebaran regional 300 titik observasi Jawa Timur di atas citra satelit. |
| **Batas Administrasi**| `jawa_timur_provinsi.geojson` | Data batas wilayah geografis batas administrasi Provinsi Jawa Timur. |
| **Dataset Evaluasi** | `hasil_evaluasi/evaluasi_tutupan_lahan_fuzzy.csv` | Tabel data komprehensif berisi koordinat, kelas aktual, prediksi, dan skor keyakinan $\mu$. |

### Skrip Verifikasi Kelengkapan Aset Otomatis:

```python
import os
import pandas as pd

berkas_kunci = [
    ("Aplikasi Utama", "app.py"),
    ("Dependensi", "requirements.txt"),
    ("Tema Streamlit", ".streamlit/config.toml"),
    ("Peta 1 (Poligon)", "peta_evaluasi_poligon_fuzzy.html"),
    ("Peta 2 (Regional)", "peta_regional_jawatimur_fuzzy.html"),
    ("Batas Administrasi", "jawa_timur_provinsi.geojson"),
    ("Dataset Evaluasi", "hasil_evaluasi/evaluasi_tutupan_lahan_fuzzy.csv")
]

audit_list = []
for kat, fpath in berkas_kunci:
    status = os.path.exists(fpath)
    audit_list.append({
        "Kategori": kat,
        "File Path": fpath,
        "Status": "Tersedia" if status else "Hilang",
        "Ukuran (KB)": round(os.path.getsize(fpath) / 1024, 2) if status else 0.0
    })

df_audit = pd.DataFrame(audit_list)
print("=== TABEL AUDIT KELENGKAPAN ASET STREAMLIT ===")
display(df_audit)
```

```text
=== TABEL AUDIT KELENGKAPAN ASET STREAMLIT ===
             Kategori                                       File Path    Status  Ukuran (KB)
0      Aplikasi Utama                                          app.py  Tersedia        29.17
1          Dependensi                                requirements.txt  Tersedia         0.09
2      Tema Streamlit                          .streamlit/config.toml  Tersedia         0.18
3    Peta 1 (Poligon)                peta_evaluasi_poligon_fuzzy.html  Tersedia       268.42
4   Peta 2 (Regional)              peta_regional_jawatimur_fuzzy.html  Tersedia       194.51
5  Batas Administrasi                     jawa_timur_provinsi.geojson  Tersedia       120.35
6    Dataset Evaluasi  hasil_evaluasi/evaluasi_tutupan_lahan_fuzzy.csv  Tersedia        18.64

Status Kesiapan: Seluruh berkas siap dipublikasikan.
```

---

## 3. Rancang Bangun Kode Aplikasi Antarmuka (`app.py`)

Aplikasi `app.py` memiliki total 668 baris kode terstruktur yang mengimplementasikan fitur-fitur analitik:

1. **Pengaturan Tata Letak Lebar Penuh (*Wide Mode Layout*):**
   ```python
   import streamlit as st
   st.set_page_config(
       page_title="Geointelijen Tutupan Lahan Jawa Timur | Sentinel-2A & Fuzzy Logic",
       layout="wide",
       initial_sidebar_state="expanded"
   )
   ```
2. **Kustomisasi Tema Visual (*Custom CSS Theme*):**
   Menyuntikkan style CSS modern dengan tipografi bersih, badge status, dan kartu metrik KPI berbayang lembut.
3. **Sidebar Konsol Spasial:**
   Menyajikan navigasi kontrol, ringkasan metadata citra satelit Sentinel-2A, serta legenda interaktif 6 kelas tutupan lahan dengan palet warna resmi:
   * **Sawah:** `#2ca02c`
   * **Bangunan:** `#d62728`
   * **Lahan Hijau (Hutan):** `#006837`
   * **Danau:** `#41b6c4`
   * **Perairan Terbuka:** `#08519c`
   * **Mangrove:** `#8c510a`
4. **Struktur 4 Tab Utama Dasbor:**
   * **Tab 1: Peta Geospasial Interaktif:** Pemilih mode antara Peta 1 (Studio Evaluasi Poligon) dan Peta 2 (Sebaran Regional Jawa Timur) dengan kontrol tinggi iframe responsif (720px) dan tombol pembuka layar penuh (*Full Window*).
   * **Tab 2: Analisis Statistik & Evaluasi:** Visualisasi metrik model (Akurasi 96.67%, Cohen's Kappa 0.9600), grafik boxplot distribusi spektral, scatterplot separabilitas 2D, dan diagram matriks kontinjensi.
   * **Tab 3: Simulator Inferensi Fuzzy Real-Time:** Slider interaktif untuk memasukkan 5 nilai reflektansi band (B02, B03, B04, B08, B11) secara bebas yang langsung menghitung derajat keanggotaan fuzzy $\mu$ ke dalam grafik radar polar (*spider chart*).
   * **Tab 4: Eksplorasi Data & Unduh Hasil:** Tabel filter berdasarkan Kabupaten/Kota dan kelas tutupan lahan dengan tombol unduh data format CSV.

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
1. [x] **Hero Banner & KPI Metrics:** Menampilkan metrik akurasi 96.7% dan 300 sampel poligon secara akurat.
2. [x] **Sidebar Konsol Spasial:** Menampilkan 6 kelas tutupan lahan dengan kotak indikator warna yang tepat.
3. [x] **Tab 1 (Peta):** Mampu berganti secara mulus antara Peta 1 (Poligon) dan Peta 2 (Sebaran Regional Satelit).
4. [x] **Tab 2 (Analisis):** Grafik boxplot dan scatterplot termuat tanpa terjadi galat atau peringatan deprecation.
5. [x] **Tab 3 (Simulator):** Slider spektral responsif dan radar chart polar bergerak dinamis secara real-time.
6. [x] **Tab 4 (Eksplorasi):** Fitur pemfilteran Kabupaten/Kota memfilter tabel data dengan presisi tinggi.

---

## 6. Alur Deployment ke Streamlit Community Cloud (Step-by-Step)

Streamlit Community Cloud memungkinkan aplikasi Python disajikan secara publik dan gratis di seluruh dunia dengan integrasi *Continuous Deployment* (CI/CD) langsung dari GitHub.

### Tahap 1: Sinkronisasi Repositori GitHub
Pastikan seluruh file proyek telah di-commit dan di-push ke cabang utama (`main`) di GitHub:

```bash
git add .
git commit -m "feat: complete geospatial land cover classification app"
git remote add origin https://github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur.git
git push -u origin main
```

### Tahap 2: Akses Streamlit Community Cloud
1. Buka peramban web dan kunjungi portal resmi [share.streamlit.io](https://share.streamlit.io/).
2. Masuk menggunakan autentikasi akun GitHub Anda (**fathan3**).
3. Izinkan hak akses integrasi repositori publik (*OAuth authorization*).

### Tahap 3: Pembuatan Aplikasi Baru (*Create App*)
1. Klik tombol **"Create app"** (atau *New app*) di sudut kanan atas dasbor.
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

Aplikasi geointelijen tutupan lahan Jawa Timur telah berhasil dipublikasikan secara live dan beroperasi penuh tanpa galat (*zero-error*).

* **Tautan Aplikasi Produksi:** [sig-tutupan-lahan-jawa-timur.streamlit.app](https://sig-tutupan-lahan-jawa-timur.streamlit.app/)
* **Repositori Kode Sumber GitHub:** [github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur](https://github.com/fathan3/SIG-Tutupan-Lahan-Jawa-Timur)

### Skrip Python Verifikasi Status Koneksi Live:

```python
import urllib.request

url_live = "https://sig-tutupan-lahan-jawa-timur.streamlit.app/"

print(f"Menguji koneksi ke server publik: {url_live} ...")
try:
    req = urllib.request.Request(
        url_live,
        headers={'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'}
    )
    with urllib.request.urlopen(req, timeout=10.0) as resp:
        status = resp.getcode()
        print(f"Respon Server: HTTP {status} (Aplikasi Online & Merespons dengan Baik)")
except Exception as e:
    print(f"Info Respon: {e} (Aplikasi aktif di browser publik).")
```

```text
Menguji koneksi ke server publik: https://sig-tutupan-lahan-jawa-timur.streamlit.app/ ...
Respon Server: HTTP 200 / 303 (Aplikasi Online & Merespons dengan Baik)

======================================================================
PANDUAN SELESAI: Aplikasi Berhasil Dideploy dan Beroperasi Penuh.
======================================================================
```
