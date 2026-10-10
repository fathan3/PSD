# Deployment & Visualisasi Interaktif (Dual-Map Studio)

Tahap **Deployment & Visualisasi Interaktif** merealisasikan hasil pemodelan sains data spasial ke dalam instrumen visual geospasial berbasis web. Visualisasi ini dirancang dalam arsitektur studio peta ganda (*Interactive Dual-Map Studio*) menggunakan pustaka **Folium** dan **Leaflet.js**, serta diintegrasikan ke dalam dasbor analitik **Streamlit Cloud**.

---

## 1. Rancang Bangun Peta 1: Studio Evaluasi Poligon Ground Truth

Peta 1 didesain khusus untuk melakukan audit visual terhadap setiap entitas geometri poligon sampel ground truth. Pengguna dapat mengevaluasi poligon lapangan dan membandingkan kelas aktual dengan kelas prediksi model serta memeriksa skor keyakinan keanggotaannya:

```python
# Inisialisasi Peta Folium Evaluasi Poligon
pusat_jatim = [-7.75, 112.75]
peta1 = folium.Map(
    location=pusat_jatim,
    zoom_start=8,
    tiles=None,
    control_scale=True
)

# Menambahkan Basemap Citra Satelit Resolusi Tinggi
folium.TileLayer(
    tiles="https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}",
    attr="Google Satellite Hybrid",
    name="Google Hybrid (Satelit & Label Jalan)",
    max_zoom=20,
    overlay=False
).add_to(peta1)

folium.TileLayer(
    tiles="https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
    attr="Tiles &copy; Esri World Imagery",
    name="Esri World Imagery",
    max_zoom=19,
    overlay=False
).add_to(peta1)

# Render Layer Poligon Evaluasi dengan Warna Interaktif
for kls, meta in SUMBER_DATA.items():
    sub_df = df_evaluasi[df_evaluasi["kelas"] == kls]
    fg = folium.FeatureGroup(name=f"Poligon Uji: {kls} ({len(sub_df)} poligon)", show=True)
    
    for _, r in sub_df.iterrows():
        status_tepat = r["benar"]
        warna_garis = "#22c55e" if status_tepat else "#ef4444"
        
        popup_html = f"""
        <div style="font-family:sans-serif;font-size:12px;min-width:210px;line-height:1.5;">
            <b style="font-size:13px;color:#1e293b;">{r['nama_objek']}</b><br>
            <div style="margin:4px 0;padding:2px 6px;background:#f1f5f9;border-radius:4px;display:inline-block;">
                Kelas Aktual: <b style="color:{meta['warna']};">{kls}</b>
            </div><br>
            Prediksi Model: <b>{r['kelas_prediksi']}</b> ({'Tepat' if status_tepat else 'Ambiguitas'})<br>
            Keyakinan Fuzzy &mu;: <b>{r['keyakinan_max']*100:.1f}%</b><br>
            NDVI: <b>{r['NDVI']:.3f}</b> | NDBI: <b>{r['NDBI']:.3f}</b><br>
            Wilayah: <b>{r.get('kab_kota', 'Jawa Timur')}</b>
        </div>
        """
        
        # Gambar Poligon Geometri
        folium.Polygon(
            locations=r["koordinat_points"],
            color=warna_garis,
            weight=2.5,
            fill=True,
            fill_color=meta["warna"],
            fill_opacity=0.65,
            tooltip=f"{r['nama_objek']} ({kls}) | Keyakinan: {r['keyakinan_max']*100:.1f}%",
            popup=folium.Popup(popup_html, max_width=280)
        ).add_to(fg)
        
    fg.add_to(peta1)

folium.LayerControl(collapsed=False, position="topright").add_to(peta1)
peta1.save("peta_evaluasi_poligon_fuzzy.html")
print("Peta 1 berhasil disimpan ke file 'peta_evaluasi_poligon_fuzzy.html'.")
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

*(Catatan: Anda dapat berinteraksi langsung dengan peta, memperbesar/memperkecil zoom, mengklik poligon untuk melihat informasi atribut popup dan derajat keyakinan fuzzy, serta memilih layer basemap citra satelit pada kontrol kanan atas).*

---

## 2. Rancang Bangun Peta 2: Sebaran Spasial Regional Jawa Timur

Peta 2 menyajikan gambaran makro sebaran 300 titik observasi geospasial di seluruh penjuru Provinsi Jawa Timur yang dilengkapi dengan layer batas wilayah administrasi provinsi:

```python
# Inisialisasi Peta Sebaran Spasial Regional
peta2 = folium.Map(
    location=[-7.75, 112.75],
    zoom_start=8,
    tiles=None,
    control_scale=True
)

# Menambahkan Basemap Satelit
folium.TileLayer(
    tiles="https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}",
    attr="Google Satellite Hybrid",
    name="Google Hybrid (Satelit & Label)",
    max_zoom=20,
    overlay=False
).add_to(peta2)

folium.TileLayer(
    tiles="https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
    attr="Tiles &copy; Esri World Imagery",
    name="Esri World Imagery",
    max_zoom=19,
    overlay=False
).add_to(peta2)

# Menambahkan Lapisan Batas Administrasi Provinsi Jawa Timur (GeoJSON)
if os.path.exists('jawa_timur_provinsi.geojson'):
    with open('jawa_timur_provinsi.geojson', 'r', encoding='utf-8') as f:
        gj_prov = json.load(f)
    grup_batas = folium.FeatureGroup(name="Batas Provinsi Jawa Timur", show=True)
    folium.GeoJson(
        gj_prov,
        style_function=lambda x: {
            'color': '#ffffff',
            'weight': 1.8,
            'dashArray': '4 4',
            'fillColor': '#ffffff',
            'fillOpacity': 0.03
        },
        tooltip="Batas Administrasi Provinsi Jawa Timur"
    ).add_to(grup_batas)
    grup_batas.add_to(peta2)

# Menambahkan Marker Centroid untuk 300 Poligon
for kls, meta in SUMBER_DATA.items():
    sub_df = df_dataset[df_dataset["kelas"] == kls]
    fg = folium.FeatureGroup(name=f"Sebaran: {kls} ({len(sub_df)} titik)", show=True)
    for _, r in sub_df.iterrows():
        folium.CircleMarker(
            location=[r["lat_centroid"], r["lon_centroid"]],
            radius=6,
            color="#ffffff",
            weight=1.5,
            fill=True,
            fill_color=meta["warna"],
            fill_opacity=0.9,
            tooltip=f"{r['nama_objek']} ({kls})",
            popup=folium.Popup(
                f"<b>{r['nama_objek']}</b><br>Kelas: {kls}<br>Kab/Kota: {r.get('kab_kota', '-')}",
                max_width=220
            )
        ).add_to(fg)
    fg.add_to(peta2)

folium.LayerControl(collapsed=False, position="topright").add_to(peta2)
peta2.save("peta_regional_jawatimur_fuzzy.html")
print("Peta 2 berhasil disimpan ke 'peta_regional_jawatimur_fuzzy.html'.")
```

### Hasil Peta 2: Sebaran Spasial Regional Jawa Timur

Berikut adalah visualisasi interaktif langsung dari Peta 2:

<iframe 
    src="peta_regional_jawatimur_fuzzy.html" 
    width="100%" 
    height="650" 
    frameborder="0" 
    style="border: 1px solid #cbd5e1; border-radius: 8px; width: 100%; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);">
</iframe>

*(Catatan: Anda dapat mengklik marker lingkaran observasi untuk melihat nama objek lapangan, kelas tutupan lahan, dan kabupaten/kota pengamatan).*

---

## 3. Integrasi & Status Kesiapan Aplikasi Dasbor Streamlit (`app.py`)

Seluruh visualisasi peta HTML interaktif, model inferensi, dataset evaluasi, dan modul simulasi diintegrasikan ke dalam antarmuka aplikasi web berbasis **Streamlit**:

```python
print("=== STATUS ARSITEKTUR SISTEM INFORMASI GEOGRAFIS STREAMLIT ===")
print(f"- app.py            : {'Tersedia' if os.path.exists('app.py') else 'Belum ada'}")
print(f"- requirements.txt  : {'Tersedia' if os.path.exists('requirements.txt') else 'Belum ada'}")
print(f"- config.toml       : {'Tersedia' if os.path.exists('.streamlit/config.toml') else 'Belum ada'}")
print(f"- Dataset Evaluasi  : {'Tersedia' if os.path.exists('hasil_evaluasi/evaluasi_tutupan_lahan_fuzzy.csv') else 'Belum ada'}")
print(f"- Peta 1 (Poligon)  : {'Tersedia' if os.path.exists('peta_evaluasi_poligon_fuzzy.html') else 'Belum ada'}")
print(f"- Peta 2 (Regional) : {'Tersedia' if os.path.exists('peta_regional_jawatimur_fuzzy.html') else 'Belum ada'}")
```

```text
=== STATUS ARSITEKTUR SISTEM INFORMASI GEOGRAFIS STREAMLIT ===
- app.py            : Tersedia
- requirements.txt  : Tersedia
- config.toml       : Tersedia
- Dataset Evaluasi  : Tersedia
- Peta 1 (Poligon)  : Tersedia
- Peta 2 (Regional) : Tersedia
```

### Akses Server Aplikasi:
* **Eksekusi di Komputer Lokal:**
  ```bash
  streamlit run app.py
  ```
* **Akses Publik di Cloud:**  
  Aplikasi beroperasi secara aktif di internet melalui tautan: [sig-tutupan-lahan-jawa-timur.streamlit.app](https://sig-tutupan-lahan-jawa-timur.streamlit.app/)
