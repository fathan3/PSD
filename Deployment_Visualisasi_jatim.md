# Deployment & Visualisasi Interaktif (Dual-Map Studio)

Tahap **Deployment & Visualisasi Interaktif** merealisasikan hasil pemodelan sains data spasial ke dalam instrumen visual geospasial berbasis web. Visualisasi ini dirancang dalam arsitektur studio peta ganda (*Interactive Dual-Map Studio*) menggunakan pustaka **Folium** dan **Leaflet.js**, serta diintegrasikan ke dalam dasbor analitik **Streamlit Cloud**.

---

## 1. Rancang Bangun Peta 1: Studio Evaluasi Poligon Ground Truth

Peta 1 didesain khusus untuk melakukan audit visual mendalam terhadap setiap entitas geometri poligon sampel ground truth asli di Jawa Timur. Pengguna dapat mengevaluasi 300 poligon lapangan, membandingkan kelas aktual dengan kelas prediksi model *Fuzzy Logic*, serta memeriksa derajat keyakinan keanggotaannya ($\mu_{\max}$):

* **Sorotan Data Uji:** Poligon yang termasuk dalam subset pengujian (30% data uji independen) disorot dengan garis tepi berwarna biru tebal (`#0d6efd`) untuk kemudahan identifikasi visual.
* **Penanda Misklasifikasi:** Sampel data uji yang mengalami salah prediksi ditandai secara transparan dengan marker oranye bertanda seru (`exclamation-sign`).
* **Popup Analitik Komprehensif:** Setiap poligon dapat diklik untuk menampilkan kartu tabel yang memuat nama objek, status subset (Latih/Uji), status evaluasi (Benar/Salah), kelas aktual vs prediksi fuzzy, derajat keyakinan $\mu_{\max}$, kabupaten/kota, serta nilai indeks spektral biofisik (NDVI, NDWI, NDBI).
* **Legenda LULC Mengambang:** Menampilkan panduan warna 6 kelas tutupan lahan di pojok kiri bawah peta.

```python
# ==============================================================================
# 6.1 PETA 1: EVALUASI POLIGON SAMPEL & DERAJAT KEANGGOTAAN FUZZY (INTERAKTIF INLINE)
# ==============================================================================
pusat_jatim = [-7.75, 112.75]
peta1 = folium.Map(location=pusat_jatim, zoom_start=9, tiles="CartoDB positron", control_scale=True)

folium.TileLayer(
    tiles="https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
    attr="Tiles &copy; Esri World Imagery",
    name="Citra Satelit Esri",
    max_zoom=19
).add_to(peta1)

folium.TileLayer(
    tiles="https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}",
    attr="Google Hybrid",
    name="Google Hybrid (Satelit + Label)",
    max_zoom=20
).add_to(peta1)

grup_kelas = {k: folium.FeatureGroup(name=f"Kelas: {k}", show=True) for k in SUMBER_DATA.keys()}
grup_uji = folium.FeatureGroup(name="Sorotan Data Uji (Outline Biru)", show=True)
grup_salah = folium.FeatureGroup(name="Salah Prediksi (Marker Oranye)", show=True)

for _, r in df_evaluasi.iterrows():
    pts = r["koordinat_points"]
    warna = SUMBER_DATA[r["kelas"]]["warna"]
    is_test = (r["subset"] == "Uji")
    is_wrong = (r["evaluasi_status"] == "Salah" and is_test)
    
    popup_html = f"""
    <div style='font-family:Arial, sans-serif; font-size:12px; width:260px;'>
        <h4 style='margin:0 0 6px 0; color:#1a1a1a;'>{r['nama_objek']}</h4>
        <span style='background:#0d6efd; color:#fff; padding:2px 6px; border-radius:3px; font-size:10px;'>{r['subset']}</span>
        <span style='background:{'#28a745' if r['evaluasi_status']=='Benar' else '#dc3545'}; color:#fff; padding:2px 6px; border-radius:3px; font-size:10px;'>{r['evaluasi_status']}</span>
        <table style='width:100%; margin-top:8px; border-collapse:collapse;'>
            <tr><td><b>Kelas Asli</b></td><td>: {r['kelas']}</td></tr>
            <tr><td><b>Prediksi Fuzzy</b></td><td>: {r['kelas_prediksi']}</td></tr>
            <tr><td><b>Keyakinan (μ)</b></td><td>: {r['keyakinan_max']*100:.2f}%</td></tr>
            <tr><td><b>Kab/Kota</b></td><td>: {r['kab_kota']}</td></tr>
            <tr><td><b>NDVI / NDWI</b></td><td>: {r['NDVI']:.3f} / {r['NDWI']:.3f}</td></tr>
            <tr><td><b>NDBI</b></td><td>: {r['NDBI']:.3f}</td></tr>
        </table>
    </div>
    """
    
    lat_lon_pts = [[p[1], p[0]] for p in pts]
    
    poly = folium.Polygon(
        locations=lat_lon_pts,
        color="#333333",
        weight=1.0,
        fill=True,
        fill_color=warna,
        fill_opacity=0.75,
        tooltip=f"{r['poligon_id']} | Asli: {r['kelas']} | Pred: {r['kelas_prediksi']} ({r['keyakinan_max']*100:.1f}%)",
        popup=folium.Popup(popup_html, max_width=300)
    )
    poly.add_to(grup_kelas[r["kelas"]])
    
    if is_test:
        folium.Polygon(
            locations=lat_lon_pts,
            color="#0d6efd",
            weight=2.5,
            fill=False
        ).add_to(grup_uji)
        
    if is_wrong:
        folium.Marker(
            location=[r["lat_centroid"], r["lon_centroid"]],
            icon=folium.Icon(color="orange", icon="exclamation-sign"),
            tooltip=f"SALAH: {r['poligon_id']} (Asli: {r['kelas']}, Pred: {r['kelas_prediksi']})"
        ).add_to(grup_salah)

for g in grup_kelas.values():
    g.add_to(peta1)
grup_uji.add_to(peta1)
grup_salah.add_to(peta1)

legenda_html = """<div style="position:fixed; bottom:30px; left:30px; z-index:9999; background:white;
padding:12px 16px; border:1px solid #ccc; border-radius:6px; font-family:Arial; font-size:12px; box-shadow:0 2px 6px rgba(0,0,0,0.2);">
<b>Legenda 6 Kelas Tutupan Lahan (LULC)</b><hr style="margin:6px 0;">
<div><span style="display:inline-block;width:14px;height:14px;background:#74c476;margin-right:6px;border:1px solid #444;"></span>Sawah</div>
<div><span style="display:inline-block;width:14px;height:14px;background:#de2d26;margin-right:6px;border:1px solid #444;"></span>Bangunan</div>
<div><span style="display:inline-block;width:14px;height:14px;background:#006d2c;margin-right:6px;border:1px solid #444;"></span>Lahan Hijau</div>
<div><span style="display:inline-block;width:14px;height:14px;background:#41b6c4;margin-right:6px;border:1px solid #444;"></span>Danau</div>
<div><span style="display:inline-block;width:14px;height:14px;background:#08519c;margin-right:6px;border:1px solid #444;"></span>Perairan Terbuka</div>
<div><span style="display:inline-block;width:14px;height:14px;background:#8c510a;margin-right:6px;border:1px solid #444;"></span>Mangrove</div>
</div>"""
peta1.get_root().html.add_child(folium.Element(legenda_html))
folium.LayerControl(collapsed=False).add_to(peta1)

peta1.save("peta_evaluasi_poligon_fuzzy.html")
print("✓ Peta 1 berhasil disimpan ke file 'peta_evaluasi_poligon_fuzzy.html'.")
```

### Hasil Peta 1: Studio Evaluasi Poligon Ground Truth

Berikut adalah visualisasi interaktif langsung dari Peta 1:

<iframe 
    src="peta_evaluasi_poligon_fuzzy.html" 
    width="100%" 
    height="650" 
    frameborder="0" 
    style="border: 1px solid #cbd5e1; border-radius: 8px; width: 100%; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);">
</iframe>

*(Catatan: Anda dapat berinteraksi langsung dengan peta, memperbesar/memperkecil zoom, mengklik poligon untuk melihat informasi atribut popup dan derajat keyakinan fuzzy, memeriksa poligon uji bergaris tepi biru tebal serta marker peringatan oranye untuk misklasifikasi, dan memilih layer basemap citra satelit pada kontrol kanan atas).*

---

## 2. Rancang Bangun Peta 2: Klasifikasi Tutupan Lahan Skala Regional Jawa Timur (ImageOverlay)

Peta 2 merupakan visualisasi spasial makro yang menyajikan hasil inferensi spasial klasifikasi tutupan lahan (*Land Use and Land Cover* - LULC) skala regional untuk seluruh daratan dan pesisir Provinsi Jawa Timur secara kontinu menggunakan teknik **ImageOverlay** di atas citra satelit resolusi tinggi.

Berbeda dengan sekadar sebaran titik koordinat, pembaruan Peta 2 mengintegrasikan raster hasil prediksi algoritma *Supervised Fuzzy Logic* ($m=2.0$) yang mencakup seluruh *Area of Interest* Jawa Timur (AOI: $111.0^\circ\text{ BT} - 114.65^\circ\text{ BT}$, $-8.85^\circ\text{ LS} - -6.75^\circ\text{ LS}$) dengan transparansi transparan $65\%$, dipadukan secara harmonis dengan titik sampel validasi lapangan per kelas serta legenda LULC mengambang (*floating widget*).

### Karakteristik & Fitur Utama Pembaruan Peta 2:
1. **Lapisan Raster Klasifikasi Regional (*ImageOverlay* Opasitas 65%):**
   * Berkas raster klasifikasi fuzzy regional (`hasil_evaluasi/peta_regional_raster.png`) dikonversi ke representasi string Base64 PNG secara *in-memory* dan diproyeksikan langsung pada koordinat batas wilayah Jawa Timur (`batas_peta`).
   * Tingkat opasitas $0.65$ dirancang presisi agar zonasi tematik tutupan lahan terlihat kontras dan tegas tanpa menutupi tekstur geomorfologi, relief pegunungan, pola jalan, dan garis pantai pada citra satelit di bawahnya.
2. **Skema Palet Warna Standar Penilaian LULC (`WARNA_REGIONAL`):**
   * <span style="display:inline-block;width:13px;height:13px;background:#FFD92F;border:1px solid #333;vertical-align:middle;margin-right:4px;"></span> **Sawah (`#FFD92F`):** Kuning keemasan, mencerminkan hamparan sawah irigasi teknis dan non-irigasi lumbung pangan Jawa Timur.
   * <span style="display:inline-block;width:13px;height:13px;background:#E41A1C;border:1px solid #333;vertical-align:middle;margin-right:4px;"></span> **Bangunan (`#E41A1C`):** Merah mencolok, memetakan kepadatan permukiman perkotaan dan perkerasan kedap air (*impervious surfaces*).
   * <span style="display:inline-block;width:13px;height:13px;background:#8E44AD;border:1px solid #333;vertical-align:middle;margin-right:4px;"></span> **Mangrove (`#8E44AD`):** Ungu pesisir, menandai sabuk hijau hutan bakau muara sungai dan pantai utara/timur.
   * <span style="display:inline-block;width:13px;height:13px;background:#2E7D32;border:1px solid #333;vertical-align:middle;margin-right:4px;"></span> **Hutan / Lahan Hijau (`#2E7D32`):** Hijau tua, mendelineasi hutan pegunungan, kawasan lindung, dan tutupan vegetasi lebat.
   * <span style="display:inline-block;width:13px;height:13px;background:#0D47A1;border:1px solid #333;vertical-align:middle;margin-right:4px;"></span> **Laut / Perairan Terbuka (`#0D47A1`):** Biru tua samudera, merepresentasikan perairan laut lepas dan selat.
   * <span style="display:inline-block;width:13px;height:13px;background:#4FC3F7;border:1px solid #333;vertical-align:middle;margin-right:4px;"></span> **Danau (`#4FC3F7`):** Biru muda / cyan cerah, memetakan waduk buatan, rawa alami, dan telaga daratan.
3. **Titik Sampel Validasi Terorganisir per Kelas (*FeatureGroup*):**
   * Titik observasi lapangan dikelompokkan ke dalam layer mandiri untuk setiap kelas tutupan lahan (`Titik Sampel: Sawah`, `Titik Sampel: Bangunan`, dll.) yang dapat dinyalakan/dimatikan sesuai kebutuhan audit pengguna.
   * Dirender menggunakan `CircleMarker` beradius 4 dengan garis tepi putih solid (`#ffffff`) dan isi warna kelas, dilengkapi popup informatif berformat kartu yang merinci: nama objek, kelas LULC bertanda warna, koordinat latitude/longitude 4 desimal, dan persentase Derajat Keyakinan Fuzzy ($\mu_{\max}$).
4. **Legenda LULC Mengambang Permanen (*Floating Responsive Legend*):**
   * Panel legenda modern berposisi tetap di sudut kiri bawah peta (`position:fixed; bottom:30px; left:30px; z-index:9999`) dengan latar belakang semi-transparan (`rgba(255,255,255,0.92)`), batas halus, dan bayangan elevasi (*box-shadow*) agar pengguna senantiasa memiliki acuan interpretasi warna tanpa mengganggu navigasi peta.
5. **Dukungan Multi-Basemap Satelit Resolusi Tinggi:**
   * **Esri Satellite (Default):** Citra satelit global beresolusi sangat tinggi dari Esri World Imagery yang tajam dan jernih.
   * **Google Hybrid (Opsional):** Citra satelit Google dipadukan dengan label toponimi nama kota, jalan raya, dan batas administratif.

```python
# ==============================================================================
# 6.2 PETA 2: KLASIFIKASI TUTUPAN LAHAN SKALA REGIONAL JAWA TIMUR (IMAGEOVERLAY)
# Inferensi Spasial Supervised Fuzzy Logic (m=2.0) di Atas Citra Satelit Esri & Google
# ==============================================================================
import base64
import io
import os
from PIL import Image
import folium
from folium.raster_layers import ImageOverlay

AOI = [111.0, -8.85, 114.65, -6.75]  # [lon_w, lat_s, lon_e, lat_n]
lat_s, lon_w = -8.85, 111.0
lat_n, lon_e = -6.75, 114.65
batas_peta = [[lat_s, lon_w], [lat_n, lon_e]]
pusat_peta = [(lat_s + lat_n) / 2, (lon_w + lon_e) / 2]

WARNA_REGIONAL = {
    "Sawah": "#FFD92F",
    "Bangunan": "#E41A1C",
    "Mangrove": "#8E44AD",
    "Hutan": "#2E7D32",
    "Laut": "#0D47A1",
    "Danau": "#4FC3F7"
}

# 1. Muat Raster Klasifikasi Regional Supervised Fuzzy
raster_path = "hasil_evaluasi/peta_regional_raster.png"
if os.path.exists(raster_path):
    img_raster = Image.open(raster_path)
    buf = io.BytesIO()
    img_raster.save(buf, format="PNG", optimize=True)
    b64_raster = "data:image/png;base64," + base64.b64encode(buf.getvalue()).decode("utf-8")
else:
    b64_raster = None

# 2. Inisialisasi Peta Interaktif Folium
peta2 = folium.Map(location=pusat_peta, zoom_start=8, tiles=None, control_scale=True)

# Lapisan Basemap Esri Satellite (Default)
folium.TileLayer(
    tiles="https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
    attr="Tiles &copy; Esri",
    name="Esri Satellite",
    max_zoom=19,
    overlay=False
).add_to(peta2)

# Lapisan Basemap Google Hybrid
folium.TileLayer(
    tiles="https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}",
    attr="Google Hybrid",
    name="Google Hybrid",
    max_zoom=20,
    overlay=False,
    show=False
).add_to(peta2)

# 3. Lapisan Citra Klasifikasi Supervised Fuzzy (ImageOverlay Opasitas 65%)
if b64_raster:
    ImageOverlay(
        image=b64_raster,
        bounds=batas_peta,
        opacity=0.65,
        name="Hasil Klasifikasi Fuzzy (Skala Regional)",
        zindex=5
    ).add_to(peta2)

# 4. Tambahkan Titik Sampel Validasi per Kelas
label_map_reg = {
    "Sawah": "Sawah",
    "Bangunan": "Bangunan",
    "Lahan Hijau": "Hutan",
    "Mangrove": "Mangrove",
    "Perairan Terbuka": "Laut",
    "Danau": "Danau"
}

for nama_key, warna in WARNA_REGIONAL.items():
    grup = folium.FeatureGroup(name=f"Titik Sampel: {nama_key}", show=False)
    sub = df_evaluasi[df_evaluasi["kelas"].map(label_map_reg) == nama_key]
    for _, r in sub.iterrows():
        folium.CircleMarker(
            location=[r["lat_centroid"], r["lon_centroid"]],
            radius=4,
            color="#ffffff",
            weight=1,
            fill=True,
            fill_color=warna,
            fill_opacity=1.0,
            tooltip=f"{r['poligon_id']} ({nama_key})",
            popup=folium.Popup(
                f"<div style='font-family:sans-serif;font-size:12px;min-width:180px;'>"
                f"<b>{r['nama_objek']}</b><br>"
                f"Kelas: <b style='color:{warna};'>{nama_key}</b><br>"
                f"Koordinat: {r['lat_centroid']:.4f}, {r['lon_centroid']:.4f}<br>"
                f"Keyakinan Fuzzy &mu;: <b>{r['keyakinan_max']*100:.1f}%</b></div>",
                max_width=250
            )
        ).add_to(grup)
    grup.add_to(peta2)

# 5. Legenda LULC (Format Sesuai Template Penilaian)
item_legenda = "".join(
    f'<div style="margin:2px 0"><span '
    f'style="display:inline-block;width:14px;height:14px;background:{WARNA_REGIONAL[k]};border:1px '
    f'solid #333;margin-right:6px;vertical-align:middle"></span>{k}</div>'
    for k in WARNA_REGIONAL.keys()
)
legenda_html = (
    f'<div style="position:fixed;bottom:30px;left:30px;z-index:9999;background:rgba(255,255,255,0.92);'
    f'padding:10px 14px;border:1px solid #888;border-radius:6px;font:12px sans-serif;box-shadow:0 2px 6px rgba(0,0,0,0.2);">'
    f'<b>Legenda LULC</b><hr style="margin:5px 0">{item_legenda}</div>'
)
peta2.get_root().html.add_child(folium.Element(legenda_html))

# 6. Kontrol Layer Folium
folium.LayerControl(collapsed=False).add_to(peta2)

# 7. Simpan ke Berkas HTML
peta2.save("peta_regional_jawatimur_fuzzy.html")
print("✓ Peta 2 berhasil disimpan ke 'peta_regional_jawatimur_fuzzy.html'.")

# 8. Render Langsung di Output Sel Notebook
display(peta2)
```

### Hasil Peta 2: Klasifikasi Tutupan Lahan Skala Regional Jawa Timur

Berikut adalah visualisasi interaktif langsung dari Peta 2 yang telah dimutakhirkan:

<iframe 
    src="peta_regional_jawatimur_fuzzy.html" 
    width="100%" 
    height="650" 
    frameborder="0" 
    style="border: 1px solid #cbd5e1; border-radius: 8px; width: 100%; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);">
</iframe>

*(Catatan: Anda dapat mengaktifkan/menonaktifkan layer raster hasil klasifikasi fuzzy dan titik sampel validasi tiap kelas melalui LayerControl di pojok kanan atas, mengubah basemap satelit antara Esri dan Google Hybrid, mengklik titik sampel untuk melihat kartu keyakinan fuzzy $\mu_{\max}$, serta membaca panduan warna pada Legenda LULC di pojok kiri bawah).*

---

## 3. Integrasi & Status Kesiapan Aplikasi Dasbor Streamlit (`app.py`)

Seluruh visualisasi peta HTML interaktif, model inferensi, dataset evaluasi, dan modul simulasi diintegrasikan ke dalam antarmuka aplikasi web berbasis **Streamlit**:

```python
# ==============================================================================
# 6.3 VERIFIKASI INTEGRASI DASHBOARD STREAMLIT (app.py) & DEPLOYMENT CLOUD
# ==============================================================================
print("=== STATUS ARSITEKTUR SISTEM INFORMASI GEOGRAFIS STREAMLIT ===")
print(f"- app.py            : {'Tersedia ✓' if os.path.exists('app.py') else 'Belum ada ✗'}")
print(f"- requirements.txt  : {'Tersedia ✓' if os.path.exists('requirements.txt') else 'Belum ada ✗'}")
print(f"- config.toml       : {'Tersedia ✓' if os.path.exists('.streamlit/config.toml') else 'Belum ada ✗'}")
print(f"- Dataset Evaluasi  : {'Tersedia ✓' if os.path.exists('hasil_evaluasi/evaluasi_tutupan_lahan_fuzzy.csv') else 'Belum ada ✗'}")
print(f"- Peta 1 (Poligon)  : {'Tersedia ✓' if os.path.exists('peta_evaluasi_poligon_fuzzy.html') else 'Belum ada ✗'}")
print(f"- Peta 2 (Regional) : {'Tersedia ✓' if os.path.exists('peta_regional_jawatimur_fuzzy.html') else 'Belum ada ✗'}")

print("\nUntuk menjalankan aplikasi di server lokal:")
print("  streamlit run app.py")

print("\nAplikasi telah dideploy dan dapat diakses publik secara online di:")
print("  https://sig-tutupan-lahan-jawa-timur.streamlit.app/")
```

```text
=== STATUS ARSITEKTUR SISTEM INFORMASI GEOGRAFIS STREAMLIT ===
- app.py            : Tersedia ✓
- requirements.txt  : Tersedia ✓
- config.toml       : Tersedia ✓
- Dataset Evaluasi  : Tersedia ✓
- Peta 1 (Poligon)  : Tersedia ✓
- Peta 2 (Regional) : Tersedia ✓
```

### Akses Server Aplikasi:
* **Eksekusi di Komputer Lokal:**
  ```bash
  streamlit run app.py
  ```
* **Akses Publik di Cloud:**  
  Aplikasi beroperasi secara aktif di internet melalui tautan: [sig-tutupan-lahan-jawa-timur.streamlit.app](https://sig-tutupan-lahan-jawa-timur.streamlit.app/)

