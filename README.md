# data visualization tussendtijds opdracht

Datasets:
- [Dataset verkeersongevallen](https://statbel.fgov.be/nl/open-data/geolocalisatie-van-de-verkeersongevallen-2017-2024)
- [Dataset basisschool](https://portaal-stadantwerpen.opendata.arcgis.com/datasets/206e2532b06d42a2868d7b9bd607bcf7_255/explore?location=51.257092%2C4.398900%2C13.21)
- [Dataset zone 30](https://portaal-stadantwerpen.opendata.arcgis.com/datasets/53d50c711cc04e1aaaf2d2d3c41afe37_254/explore?location=51.256783%2C4.389100%2C10.95)

Doel:
- Waar ongevallen het vaakst voorkomen in de buurt van scholen, (sorteren op uur)
- Of er een verschil is tussen straten mét en zonder zone 30
- Eventueel heatmaps of interactieve kaarten maken om patronen te tonen.

Deze Python notebook, `opdracht.ipynb`, analyseert verkeersongevallen met voetgangers in Antwerpen tijdens de ochtend- en avondspits. De notebook bevat stappen om data in te lezen, te filteren en te visualiseren op een kaart. Hieronder vind je een overzicht van de inhoud en de specifieke uitdagingen die in de notebook worden aangepakt.

### **Inhoud van de Notebook**

De analyse volgt deze stappen:

1.  **Initialisatie en Imports:**
    * De benodigde libraries worden geïmporteerd, waaronder `folium` voor kaarten, `pyspark` voor dataverwerking, en `pyproj` voor coördinatentransformaties.
    * Een Spark-sessie wordt opgezet om grote datasets efficiënt te verwerken.

2.  **Data Inlezen:**
    * **Ongevallen:** Een dataset (`OPENDATA_MAP_2017-2024.txt`) met ongevallengegevens wordt ingelezen als een Spark DataFrame.
    * **Scholen:** Een bestand met locaties van basisscholen (`basisonderwijs.csv`) wordt ingeladen.
    * **Zone 30:** GeoJSON-data van zone 30-gebieden in Antwerpen wordt ingelezen en omgezet naar `shapely` polygonen voor ruimtelijke analyse.

3.  **Data Filteren:**
    * De ongevallendata wordt gefilterd op de provincie Antwerpen.
    * Er wordt specifiek gekeken naar ongevallen waarbij voetgangers betrokken zijn (`TX_ROAD_USR_TYPE1_NL` of `TX_ROAD_USR_TYPE2_NL` is "Voetganger").
    * De tijdstippen worden beperkt tot de spitsuren: 08u-10u en 15u-18u (specifiek de uren 08, 09, 15, 16, en 17).
    * Na filtering blijven er 1927 relevante ongevallen over.

4.  **Locatiebepaling en Transformatie (Moeilijkheid):**
    * **Coördinatensystemen:** De originele datasets gebruiken waarschijnlijk het Lambert-72 coördinatensysteem (EPSG:31370) of een ander projectiesysteem (EPSG:3857), terwijl voor visualisatie op de kaart (met Folium/Leaflet) breedte- en lengtegraden (WGS84, EPSG:4326) nodig zijn.
    * **Transformatie:** Er wordt een `Transformer` uit de `pyproj` library gebruikt om de coördinaten om te zetten (`lambert_to_latlon` functie). Dit is een kritieke stap omdat de kaartlagen anders niet correct over elkaar zouden vallen.
    * **Afstandsberekening:** Er is een functie `haversine` geïmplementeerd om de afstand tussen twee punten op aarde (in meters) te berekenen, rekening houdend met de kromming van de aarde. Dit wordt gebruikt om te bepalen of een ongeval in de buurt van een school heeft plaatsgevonden (bijvoorbeeld binnen 100 meter).

5.  **Ruimtelijke Analyse en Visualisatie:**
    * Het script controleert voor elk ongeval of het binnen een zone 30 valt (met behulp van de `within` functie van `shapely`) en of het in de buurt van een school is.
    * **Kaart:** De resultaten worden gevisualiseerd op een interactieve kaart.
        * **Cirkels:** Ongevallen worden weergegeven als cirkels. De kleur geeft waarschijnlijk aan of het ongeval in een zone 30 viel (oranje) of daarbuiten (rood). Zwarte cirkels worden mogelijk gebruikt voor de locaties van scholen of specifieke ongevallen.
    * De kaart toont ook de polygonen van de zone 30-gebieden.

### **Specifieke Uitdagingen**

* **Coördinaten Omzetten:** Een van de grootste technische uitdagingen in dit script is het correct interpreteren en converteren van de coördinaten uit de bronbestanden. De datasets gebruiken verschillende projecties (EPSG:31370 en EPSG:3857), die niet direct compatibel zijn met standaard webkaarten (EPSG:4326). Het gebruik van `pyproj` en de `Transformer` klasse is essentieel om deze vertaalslag te maken, waarbij ook rekening gehouden moet worden met het feit dat sommige coördinaten als strings met komma's in plaats van punten worden aangeleverd (bijv. `replace(',', '.')`).
* **Ruimtelijke Queries:** Het bepalen of een punt (ongeval) binnen een onregelmatige veelhoek (zone 30) valt, vereist specifieke geometrische berekeningen. De `shapely` library wordt hiervoor gebruikt, maar dit kan rekenintensief zijn bij grote hoeveelheden data.
* **Data Formaat:** Het correct parsen van de CSV-bestanden met specifieke delimiters (`|`) en quoting karakters is noodzakelijk om de data bruikbaar te maken in Spark.