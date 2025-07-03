# 🗺️ Estrazione Comuni BDN

Questa applicazione Streamlit consente di estrarre i comuni italiani presenti all'interno di un anello geografico definito da una **distanza minima e massima** (in km) rispetto a un punto fornito dall'utente.

L'app si basa su interrogazioni al **Feature Service dei limiti amministrativi BDN** e fornisce i risultati su mappa interattiva, tabella dati ed esportazione in Excel o Shapefile.

## 🚀 Funzionalità

- Inserimento **coordinate geografiche** (latitudine, longitudine)
- Selezione delle **distanze min e max** per creare un **buffer ad anello**
- Visualizzazione su **mappa Folium** del punto e dei comuni estratti
- **Tabella interattiva** dei comuni con campi ISTAT, Comune, Provincia, Regione
- **Esportazione** dei risultati in:
  - 📥 Excel (`.xlsx`)
  - 📥 Shapefile (`.zip`)
- Pulsante di **reset** per azzerare la sessione

## 🧪 Librerie utilizzate

- [Streamlit](https://streamlit.io)
- [Geopandas](https://geopandas.org/)
- [Folium](https://python-visualization.github.io/folium/)
- [Shapely](https://shapely.readthedocs.io/)
- [Requests](https://docs.python-requests.org/)
- [streamlit-folium](https://github.com/randyzwitch/streamlit-folium)

## 📦 Installazione

Clona il progetto:

```bash
git clone https://github.com/tuo-username/estrazione-comuni-bdn.git
cd estrazione-comuni-bdn
