import streamlit as st
import folium
import geopandas as gpd
import json
import requests
from shapely.geometry import Point
from streamlit_folium import st_folium
from io import BytesIO
import zipfile
import tempfile

# === Titolo ===
st.title("Estrazione Comuni BDN")

# === Riga 1: Latitudine e Longitudine ===
col1, col2 = st.columns(2)
with col1:
    lat = st.number_input("Latitudine", value=42.358628, format="%.6f", key="lat")
with col2:
    lon = st.number_input("Longitudine", value=13.811097, format="%.6f", key="lon")

# === Riga 2: Min e Max distanza ===
col3, col4 = st.columns(2)
with col3:
    min_km = st.number_input("Distanza minima (km)", value=20, key="min_km")
with col4:
    max_km = st.number_input("Distanza massima (km)", value=50, key="max_km")

# === Inizializza session_state ===
if "gdf_anello" not in st.session_state:
    st.session_state["gdf_anello"] = None
if "comuni_anello" not in st.session_state:
    st.session_state["comuni_anello"] = None

# === Bottoni: Cerca Comuni + Reset ===
col_btn1, col_btn2 = st.columns([1, 1])
with col_btn1:
    cerca = st.button("🔍 Cerca Comuni")
with col_btn2:
    reset = st.button("🔁 Reset")

if cerca:
    # 1. Crea anello
    buffer_min = min_km * 1000
    buffer_max = max_km * 1000

    punto = gpd.GeoSeries([Point(lon, lat)], crs="EPSG:4326").to_crs(epsg=32633)
    buffer_inner = punto.buffer(buffer_min)
    buffer_outer = punto.buffer(buffer_max)
    anello = buffer_outer.difference(buffer_inner)
    gdf_anello = gpd.GeoDataFrame(geometry=anello, crs="EPSG:32633").to_crs(epsg=4326)

    st.session_state["gdf_anello"] = gdf_anello

    # 2. Query comuni entro 50km
    geometry_esri = {
        "x": lon,
        "y": lat,
        "spatialReference": {"wkid": 4326}
    }

    url = "https://services7.arcgis.com/8tIzt6yXOZrB60gX/ArcGIS/rest/services/Limiti_Amministrativi_BDN/FeatureServer/0/query"
    params = {
        "geometryType": "esriGeometryPoint",
        "geometry": json.dumps(geometry_esri),
        "inSR": 4326,
        "spatialRel": "esriSpatialRelIntersects",
        "distance": buffer_max,
        "units": "esriSRUnit_Meter",
        "outFields": "*",
        "returnGeometry": "true",
        "f": "geojson"
    }

    response = requests.get(url, params=params)
    response.raise_for_status()
    gdf_50 = gpd.read_file(json.dumps(response.json()))

    # 3. Filtra comuni che intersecano l’anello
    comuni_anello = gdf_50[gdf_50.intersects(gdf_anello.unary_union)]
    st.session_state["comuni_anello"] = comuni_anello

    st.toast(f"Trovati {len(comuni_anello)} comuni nell’anello {min_km}–{max_km} km 🎯")

if reset:
    st.session_state["gdf_anello"] = None
    st.session_state["comuni_anello"] = None
    st.rerun()

# === Tabs: Mappa e Tabella ===
tab1, tab2 = st.tabs(["🗺️ Mappa", "🧾 Tabella"])

with tab1:
    m = folium.Map(location=[lat, lon], zoom_start=8, width="100%", height="400")
    folium.Marker([lat, lon], tooltip="Punto iniziale", icon=folium.Icon(color="red")).add_to(m)

    if st.session_state["gdf_anello"] is not None:
        folium.GeoJson(
            st.session_state["gdf_anello"],
            name="Anello",
            style_function=lambda x: {
                "fillColor": "#ffbf00",
                "color": "#ff0000",
                "weight": 2,
                "fillOpacity": 0.1,
            },
            tooltip=f"Anello tra {min_km} e {max_km} km"
        ).add_to(m)

    if st.session_state["comuni_anello"] is not None:
        folium.GeoJson(
            st.session_state["comuni_anello"],
            name="Comuni",
            style_function=lambda x: {
                "fillColor": "#3388ff",
                "color": "#000000",
                "weight": 1,
                "fillOpacity": 0.4,
            },
            tooltip=folium.features.GeoJsonTooltip(
                fields=["ISTAT", "COMUNE", "PROVINCIA", "REGIONE"],
                aliases=["ISTAT", "Comune:", "Provincia:", "Regione:"],
            )
        ).add_to(m)

    folium.LayerControl().add_to(m)
    st_folium(m, width=700, height=450)

with tab2:
    st.subheader("Comuni nell'anello")
    comuni = st.session_state.get("comuni_anello")
    if comuni is not None and not comuni.empty:
        df_tabella = comuni[["ISTAT", "COMUNE", "PROVINCIA", "REGIONE"]].sort_values("COMUNE").reset_index(drop=True)
        st.dataframe(df_tabella)
    else:
        st.info("Nessun comune trovato o anello non ancora generato.")

# === Esportazioni finali ===
st.markdown("---")
colx1, colx2 = st.columns(2)

with colx1:
    if st.session_state["comuni_anello"] is not None:
        df = st.session_state["comuni_anello"]
        campi = ["ISTAT", "COMUNE", "PROVINCIA", "REGIONE"]
        excel_bytes = BytesIO()
        df[campi].to_excel(excel_bytes, index=False)
        st.download_button("📥 Esporta Excel", data=excel_bytes.getvalue(), file_name="comuni_anello.xlsx")


with colx2:
    if st.session_state["comuni_anello"] is not None:
        gdf = st.session_state["comuni_anello"]
        campi = ["ISTAT", "COMUNE", "PROVINCIA", "REGIONE", "geometry"]
        gdf_export = gdf[campi]

        with tempfile.TemporaryDirectory() as tmpdir:
            shapefile_path = f"{tmpdir}/comuni_anello.shp"
            gdf_export.to_file(shapefile_path)

            zip_buffer = BytesIO()
            with zipfile.ZipFile(zip_buffer, "w") as z:
                for ext in [".shp", ".shx", ".dbf", ".prj", ".cpg"]:
                    z.write(f"{tmpdir}/comuni_anello{ext}", f"comuni_anello{ext}")
            zip_buffer.seek(0)

            st.download_button("📥 Esporta Shapefile (.zip)", data=zip_buffer.getvalue(), file_name="comuni_anello.zip")

