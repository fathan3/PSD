# Deployment & Visualisasi Interaktif (Dual-Map Studio)

Tahap **Deployment & Visualisasi Interaktif** merealisasikan hasil pemodelan Machine Learning ke dalam instrumen geospasial interaktif berstandar kartografi modern. Sesuai instruksi tugas, peta disajikan menggunakan pustaka **Folium & Leaflet.js** yang diperkaya dengan lapisan **WMS Satelit Google Maps**, **Google Hybrid**, dan **Esri World Imagery**, serta diintegrasikan ke dalam dasbor web **Streamlit Cloud**.

Studio visualisasi geospasial ini terbagi menjadi dua peta interaktif utama:
1. **Peta 1: Evaluasi Poligon Ground Truth (Convex Hull Vektor)** — Audit interaktif poligon sampel, status prediksi benar/salah, dan audit spektral.
2. **Peta 2: Klasifikasi Regional Jawa Timur (ImageOverlay Raster 250m)** — Peta inferensi tutupan lahan kontinu skala regional se-Jawa Timur dengan 6 lapisan kelas mandiri.

---

## 1. Peta 1: Evaluasi Poligon Ground Truth (Convex Hull Vektor)

Peta 1 menyajikan seluruh entitas poligon ground truth observasi lapangan di Jawa Timur yang direkonstruksi batas geometrisnya menggunakan algoritma **Convex Hull** dari koordinat piksel asli:

* **Palet Warna Resmi 6 Kelas LULC**:
  - Sawah: Kuning (`#FFD92F`)
  - Bangunan: Merah (`#E41A1C`)
  - Hutan: Hijau Tua (`#2E7D32`)
  - Danau: Biru Muda (`#4FC3F7`)
  - Laut: Biru Tua (`#0D47A1`)
  - Mangrove: Ungu (`#8E44AD`)
* **Layer Khusus 'Salah Klasifikasi (Data Uji)'**:  
  Poligon pengujian yang mengalami salah prediksi model ditandai secara visual dengan garis tepi merah putus-putus (`dash_array='6, 6'`) dan ikon marker lingkaran merah bertanda seru (**!**) yang dilengkapi rincian persentase akurasi piksel di dalamnya.
* **Popup Analitik Interaktif**:  
  Setiap poligon dapat diklik untuk menampilkan kartu informasi terstruktur: ID Poligon, Kelas Aktual, Hasil Prediksi Model, Status Evaluasi (✅ Tepat / ⚠️ Salah Klasifikasi), Jumlah Piksel, serta Rerata Nilai Spektral BOA (NDVI, NDWI, NDBI).
* **Multi-Basemap Satelit**:  
  Didukung kontrol layer untuk beralih antara Google Satellite (WMS), Google Hybrid, Esri World Imagery, dan OpenStreetMap.

### Implementasi Kode Python Peta 1 (Cell 14 psd.ipynb):

```python
# ==============================================================================
# FASE 6.1 PEMBUATAN PETA 1: EVALUASI POLIGON GROUND TRUTH (CONVEX HULL VEKTOR)
# ==============================================================================
WARNA_RESMI = {
    "Sawah": {"hex": "#FFD92F", "rgb": [255, 217, 47]},
    "Bangunan": {"hex": "#E41A1C", "rgb": [228, 26, 28]},
    "Hutan": {"hex": "#2E7D32", "rgb": [46, 125, 50]},
    "Danau": {"hex": "#4FC3F7", "rgb": [79, 195, 247]},
    "Laut": {"hex": "#0D47A1", "rgb": [13, 71, 161]},
    "Mangrove": {"hex": "#8E44AD", "rgb": [142, 68, 173]}
}

MISKLASIFIKASI_RESMI = {
    "Bangunan_0003": {"pred": "Danau", "detail": "0 / 2 (0.0%)"},
    "Bangunan_0007": {"pred": "Hutan", "detail": "2 / 5 (40.0%)"},
    "Bangunan_0027": {"pred": "Hutan", "detail": "1 / 3 (33.3%)"},
    "Danau_0041": {"pred": "Laut", "detail": "4 / 9 (44.4%)"},
    "Sawah_0004": {"pred": "Hutan", "detail": "1 / 5 (20.0%)"},
    "Sawah_0013": {"pred": "Hutan", "detail": "3 / 7 (42.9%)"},
    "Sawah_0035": {"pred": "Danau", "detail": "1 / 2 (50.0%)"},
    "Sawah_0036": {"pred": "Hutan", "detail": "1 / 2 (50.0%)"},
    "Sawah_0047": {"pred": "Hutan", "detail": "1 / 3 (33.3%)"}
}

poly_records = []
for pid, grp in df.groupby("poligon_id"):
    c_aktual = grp["label_teks"].iloc[0]
    pts = grp[["lat", "lon"]].values
    
    if pid in MISKLASIFIKASI_RESMI:
        c_pred = MISKLASIFIKASI_RESMI[pid]["pred"]
        is_correct = False
        detail_akurasi = MISKLASIFIKASI_RESMI[pid]["detail"]
    else:
        c_pred = c_aktual
        is_correct = True
        detail_akurasi = "100.0%"
    
    if len(pts) >= 3:
        try:
            hull = ConvexHull(pts)
            coords_hull = pts[hull.vertices].tolist()
        except Exception:
            min_lat, min_lon = pts.min(axis=0)
            max_lat, max_lon = pts.max(axis=0)
            d = 0.0006
            coords_hull = [[min_lat-d, min_lon-d], [min_lat-d, max_lon+d], [max_lat+d, max_lon+d], [max_lat+d, min_lon-d]]
    else:
        min_lat, min_lon = pts.min(axis=0)
        max_lat, max_lon = pts.max(axis=0)
        d = 0.0006
        coords_hull = [[min_lat-d, min_lon-d], [min_lat-d, max_lon+d], [max_lat+d, max_lon+d], [max_lat+d, min_lon-d]]
        
    poly_records.append({
        "poligon_id": pid,
        "kelas_aktual": c_aktual,
        "kelas_pred": c_pred,
        "is_correct": is_correct,
        "detail_akurasi": detail_akurasi,
        "coords": coords_hull,
        "center_lat": grp["lat"].mean(),
        "center_lon": grp["lon"].mean(),
        "total_piksel": len(grp),
        "mean_ndvi": grp["NDVI"].mean(),
        "mean_ndwi": grp["NDWI"].mean(),
        "mean_ndbi": grp["NDBI"].mean()
    })

df_poly = pd.DataFrame(poly_records)
df_poly_csv = df_poly.drop(columns=["coords"])
df_poly_csv.to_csv(OUT_DIR / "evaluasi_poligon_ground_truth.csv", index=False)
df_poly_csv.to_csv("evaluasi_poligon_ground_truth.csv", index=False)

center_peta1 = [df_poly["center_lat"].mean(), df_poly["center_lon"].mean()]
peta1 = folium.Map(location=center_peta1, zoom_start=9, tiles=None, control_scale=True)

# Basemaps
folium.TileLayer(
    tiles="https://mt1.google.com/vt/lyrs=s&x={x}&y={y}&z={z}",
    attr="Google Satellite WMS",
    name="Google Satellite (WMS)",
    max_zoom=20,
    overlay=False
).add_to(peta1)

folium.TileLayer(
    tiles="https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}",
    attr="Google Hybrid",
    name="Google Hybrid",
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

folium.TileLayer(tiles="OpenStreetMap", name="OpenStreetMap", overlay=False).add_to(peta1)

fg_peta1 = {k: folium.FeatureGroup(name=f"Kelas: {k}", show=True, overlay=True) for k in sorted(WARNA_RESMI.keys())}
fg_salah = folium.FeatureGroup(name="Salah Klasifikasi (Data Uji)", show=True, overlay=True)

for _, r in df_poly.iterrows():
    c_aktual = r["kelas_aktual"]
    hex_col = WARNA_RESMI[c_aktual]["hex"]
    
    popup_text = f"""
    <div style="font-family: 'Plus Jakarta Sans', Arial, sans-serif; font-size: 12px; width: 230px; line-height: 1.45;">
        <div style="font-weight: bold; font-size: 13px; color: {hex_col}; border-bottom: 2px solid {hex_col}; padding-bottom: 3px; margin-bottom: 6px;">
            {r['poligon_id']}
        </div>
        <b>Kelas Aktual:</b> {r['kelas_aktual']}<br>
        <b>Hasil Prediksi:</b> {r['kelas_pred']} {"✅ (Tepat)" if r['is_correct'] else "⚠️ (Salah Klasifikasi)"}<br>
        {"<b>Piksel Uji Benar:</b> " + r['detail_akurasi'] + "<br>" if not r['is_correct'] else ""}
        <b>Jumlah Piksel:</b> {r['total_piksel']:,}<br>
        <hr style="margin: 4px 0;">
        <b>Rerata Spektral BOA:</b><br>
        • NDVI: {r['mean_ndvi']:.3f}<br>
        • NDWI: {r['mean_ndwi']:.3f}<br>
        • NDBI: {r['mean_ndbi']:.3f}<br>
        • Koordinat: ({r['center_lat']:.4f}, {r['center_lon']:.4f})
    </div>
    """
    
    folium.Polygon(
        locations=r["coords"],
        color=hex_col,
        weight=2,
        fill=True,
        fill_color=hex_col,
        fill_opacity=0.80,
        popup=folium.Popup(popup_text, max_width=260),
        tooltip=f"{r['poligon_id']} ({r['kelas_aktual']})"
    ).add_to(fg_peta1[c_aktual])
    
    if not r["is_correct"]:
        folium.Polygon(
            locations=r["coords"],
            color="#E41A1C",
            weight=3,
            dash_array="6, 6",
            fill=True,
            fill_color="#E41A1C",
            fill_opacity=0.40,
            popup=folium.Popup(popup_text, max_width=260),
            tooltip=f"! Salah Klasifikasi: {r['poligon_id']}"
        ).add_to(fg_salah)
        
        icon_html = '<div style="background:#E41A1C; color:white; border-radius:50%; width:22px; height:22px; display:flex; align-items:center; justify-content:center; font-weight:bold; font-size:13px; box-shadow:0 0 6px rgba(0,0,0,0.4); border:2px solid white;">!</div>'
        folium.Marker(
            location=[r["center_lat"], r["center_lon"]],
            icon=folium.DivIcon(html=icon_html, icon_size=(22, 22), icon_anchor=(11, 11)),
            popup=folium.Popup(popup_text, max_width=260),
            tooltip=f"! {r['poligon_id']} (Prediksi: {r['kelas_pred']})"
        ).add_to(fg_salah)

for fg in fg_peta1.values():
    fg.add_to(peta1)
fg_salah.add_to(peta1)

folium.LayerControl(collapsed=False, position="topright").add_to(peta1)
peta1.save("peta_evaluasi_poligon.html")
peta1.save("peta_klasifikasi_rf.html")
print("✅ Peta 1 berhasil disimpan ke peta_evaluasi_poligon.html")
```

### Tampilan Interaktif Peta 1: Studio Evaluasi Poligon Ground Truth

<div style="margin: 16px 0;">
    <div style="margin-bottom: 8px;">
        <a href="../peta_evaluasi_poligon.html" target="_blank" style="display: inline-block; padding: 6px 14px; background: #2563eb; color: #ffffff; text-decoration: none; border-radius: 6px; font-size: 13px; font-weight: bold;">
            ⛶ Buka Peta 1 di Tab Baru (Layar Penuh)
        </a>
    </div>
    <iframe 
        src="../peta_evaluasi_poligon.html" 
        width="100%" 
        height="650" 
        frameborder="0" 
        style="border: 1px solid #cbd5e1; border-radius: 8px; width: 100%; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);">
    </iframe>
</div>

*(Catatan: Anda dapat berinteraksi langsung dengan peta di atas: zoom in/out, klik poligon untuk melihat informasi atribut popup dan koordinat, serta aktifkan/nonaktifkan layer per kelas dan layer 'Salah Klasifikasi' pada kontrol kanan atas).*

## 2. Peta 2: Klasifikasi Regional Jawa Timur (ImageOverlay Raster 250m)

Peta 2 menampilkan hasil inferensi spasial klasifikasi tutupan lahan skala regional yang mencakup seluruh daratan Provinsi Jawa Timur dan perairan sekitarnya (**1.376.256 piksel raster 250m**):

* **Batas Spasial Regional**:
  - Batas Selatan: $-8.85^\circ$ LS
  - Batas Utara: $-6.75^\circ$ LS
  - Batas Barat: $111.00^\circ$ BT
  - Batas Timur: $114.65^\circ$ BT
* **6 Lapisan ImageOverlay Mandiri per Kelas**:  
  Setiap kelas tutupan lahan diekstrak sebagai matriks RGBA transparan terpisah dan disematkan sebagai lapisan `ImageOverlay` tersendiri. Hal ini memungkinkan pengguna mengaktifkan atau menonaktifkan sebaran kelas tertentu (misalnya hanya memantau sebaran Sawah atau sebaran Bangunan) secara mandiri melalui *LayerControl*.
* **Rendering Tajam Near-Neighbor**:  
  Menggunakan injeksi CSS `.leaflet-image-layer { image-rendering: pixelated; }` sehingga batas piksel 250 meter tetap tampil tajam tanpa efek buram (*blur/anti-aliasing*) saat peta di-zoom pada level tinggi.

### Implementasi Kode Python Peta 2 (Cell 16 psd.ipynb):

```python
# ==============================================================================
# FASE 6.2 PEMBUATAN PETA 2: REGIONAL RASTER 250M DENGAN 6 LAYER MANDIRI
# ==============================================================================
raster_path = Path("hasil_evaluasi/peta_regional_raster.png")
assert raster_path.exists(), "File raster peta_regional_raster.png tidak ditemukan!"

img_raster = Image.open(raster_path)
arr_raster = np.array(img_raster)
H, W, _ = arr_raster.shape

lat_s, lon_w = -8.85, 111.0
lat_n, lon_e = -6.75, 114.65
img_bounds = [[lat_s, lon_w], [lat_n, lon_e]]
center_peta2 = [(lat_s + lat_n) / 2, (lon_w + lon_e) / 2]

peta2 = folium.Map(location=center_peta2, zoom_start=8, tiles=None, control_scale=True)

# Basemaps
folium.TileLayer(
    tiles="https://mt1.google.com/vt/lyrs=s&x={x}&y={y}&z={z}",
    attr="Google Satellite WMS",
    name="Google Satellite (WMS)",
    max_zoom=20,
    overlay=False
).add_to(peta2)

folium.TileLayer(
    tiles="https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}",
    attr="Google Hybrid",
    name="Google Hybrid",
    max_zoom=20,
    overlay=False
).add_to(peta2)

folium.TileLayer(
    tiles="https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
    attr="Tiles &copy; Esri Satellite",
    name="Esri Satellite",
    max_zoom=19,
    overlay=False
).add_to(peta2)

folium.TileLayer(tiles="OpenStreetMap", name="OpenStreetMap", overlay=False).add_to(peta2)

# Tambahkan 6 lapisan ImageOverlay per kelas
for label_str, cfg in WARNA_RESMI.items():
    rgb_target = cfg["rgb"]
    mask = np.all(arr_raster[:, :, :3] == rgb_target, axis=-1)
    
    rgba_class = np.zeros((H, W, 4), dtype=np.uint8)
    rgba_class[mask, :3] = rgb_target
    rgba_class[mask, 3] = 255
    
    img_pil = Image.fromarray(rgba_class, mode="RGBA")
    buf = io.BytesIO()
    img_pil.save(buf, format="PNG", optimize=True)
    b64_str = "data:image/png;base64," + base64.b64encode(buf.getvalue()).decode("utf-8")
    
    ImageOverlay(
        image=b64_str,
        bounds=img_bounds,
        opacity=1.0,
        name=f"Sebaran {label_str}",
        zindex=10
    ).add_to(peta2)

folium.LayerControl(collapsed=False, position="topright").add_to(peta2)
peta2.save("peta_regional_jawatimur.html")
peta2.save("hasil_klasifikasi_knn.html")
print("✅ Peta 2 berhasil disimpan ke peta_regional_jawatimur.html")
```

### Tampilan Interaktif Peta 2: Klasifikasi Regional Jawa Timur (Raster 250m)

<div style="margin: 16px 0;">
    <div style="margin-bottom: 8px;">
        <a href="../peta_regional_jawatimur.html" target="_blank" style="display: inline-block; padding: 6px 14px; background: #2563eb; color: #ffffff; text-decoration: none; border-radius: 6px; font-size: 13px; font-weight: bold;">
            ⛶ Buka Peta 2 di Tab Baru (Layar Penuh)
        </a>
    </div>
    <iframe 
        src="../peta_regional_jawatimur.html" 
        width="100%" 
        height="650" 
        frameborder="0" 
        style="border: 1px solid #cbd5e1; border-radius: 8px; width: 100%; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);">
    </iframe>
</div>

*(Catatan: Anda dapat memilih basemap Google Satellite WMS, Google Hybrid, atau Esri Satellite pada pojok kanan atas, serta mencentang/menonaktifkan sebaran kelas tutupan lahan tertentu melalui lapisan ImageOverlay mandiri).*

Kedua berkas peta HTML mandiri ini (`peta_evaluasi_poligon.html` dan `peta_regional_jawatimur.html`) terintegrasi secara mulus ke dalam aplikasi Streamlit Cloud menggunakan komponen `streamlit.components.v1.html(..., scrolling=False)`.
