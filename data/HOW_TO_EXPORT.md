# Exporting the six data files (Overpass Turbo)

Go to https://overpass-turbo.eu. For each area, zoom the map so that only Kibera (or only Mathare) fills the screen, paste a query, click Run, then Export > GeoJSON > "download". Rename the file as shown and put it in this data/ folder.

## Water  (water_kibera.geojson, water_mathare.geojson)
[out:json][timeout:60];
(
  nwr["amenity"="drinking_water"]({{bbox}});
  nwr["man_made"="water_tap"]({{bbox}});
  nwr["man_made"="water_well"]({{bbox}});
  nwr["amenity"="water_point"]({{bbox}});
);
out center;

## Toilets  (toilets_kibera.geojson, toilets_mathare.geojson)
[out:json][timeout:60];
nwr["amenity"="toilets"]({{bbox}});
out center;

## Waste  (waste_kibera.geojson, waste_mathare.geojson)
[out:json][timeout:60];
(
  nwr["amenity"="waste_disposal"]({{bbox}});
  nwr["amenity"="waste_transfer_station"]({{bbox}});
  nwr["amenity"="recycling"]({{bbox}});
  nwr["waste"]({{bbox}});
);
out center;

Check the tags: open https://taginfo.openstreetmap.org, search each key and confirm which one Map Kibera used for waste and water points. If a layer comes back empty, widen the query.
Note the date you export. Put it in CONFIG.exportDate in index.html.
Test locally: run `python3 -m http.server` in the folder above data/ and open http://localhost:8000 (opening index.html by double-click will not load the data).

## Walking-distance analysis (buildings and street network)

Run each query at https://overpass-turbo.eu, click Run, then Export, then GeoJSON, then download. Save under the file name shown. Run Kibera and Mathare separately because the exports are large. If a query times out, split the box in half and save the parts as _a and _b.

### buildings_kibera.geojson
[out:json][timeout:240];
way["building"](-1.328,36.767,-1.300,36.810);
out geom;

### roads_kibera.geojson
[out:json][timeout:240];
way["highway"]["highway"!~"motorway|motorway_link|trunk|trunk_link|proximity|construction|raceway|bus_guideway|platform|corridor|elevator|steps_none"](-1.328,36.767,-1.300,36.810);
out geom;

### buildings_mathare.geojson
[out:json][timeout:240];
way["building"](-1.270,36.842,-1.257,36.862);
out geom;

### roads_mathare.geojson
[out:json][timeout:240];
way["highway"]["highway"!~"motorway|motorway_link|trunk|trunk_link|proximity|construction|raceway|bus_guideway|platform|corridor|elevator|steps_none"](-1.270,36.842,-1.257,36.862);
out geom;


## Services comparison (Kibera vs higher-income suburbs)
Run in Overpass Turbo for each area (change the bbox or area name), export GeoJSON, and send both files:
```
[out:json][timeout:60];
(
 nwr["amenity"~"hospital|clinic|doctors|pharmacy|school|kindergarten|toilets|drinking_water|police"]({{bbox}});
 nwr["healthcare"]({{bbox}});
);
out center;
```
Suggested areas: Kibera, Lavington, Kilimani, Karen, Muthaiga, Runda. OSM completeness differs by area, so treat counts as a lower bound.


## Round 2: files for the "walk to water" and "last checked" chapters

Go to https://overpass-turbo.eu. Zoom so only Kibera (then only Mathare) fills the screen, paste, click Run.

### A. Facilities with edit dates (raw data, not GeoJSON)
Query:
[out:json][timeout:180];
(
  nwr["amenity"="drinking_water"]({{bbox}});
  nwr["man_made"="water_tap"]({{bbox}});
  nwr["man_made"="water_well"]({{bbox}});
  nwr["amenity"="water_point"]({{bbox}});
  nwr["amenity"="toilets"]({{bbox}});
  nwr["amenity"~"waste_disposal|waste_transfer_station|recycling"]({{bbox}});
);
out meta center;

Then Export > Data > "download as raw OSM data".
Save as: facilities_kibera.json and facilities_mathare.json

### B. Buildings and streets (GeoJSON)
Use the four queries in the section above ("Walking-distance analysis"):
buildings_kibera.geojson, roads_kibera.geojson, buildings_mathare.geojson, roads_mathare.geojson
Each file must stay under 25 MB to upload through the GitHub website. If one is bigger, split the box in half and save as _a and _b.

Send the six new files to Claude (or put them in the data/ folder). Do not change the file names.


## Round 3: Makoko (Lagos) for the Makoko map chapter

Server tip: if a query fails with a 502 or timeout, wait a minute or switch server in Settings.

### Buildings: save as makoko_buildings.geojson (Export > Data > GeoJSON)
[out:json][timeout:180][bbox:6.4885,3.3855,6.5040,3.4020];
way["building"];
out geom;

### Places: save as makoko_places.geojson (GeoJSON)
[out:json][timeout:180][bbox:6.4885,3.3855,6.5040,3.4020];
(
  nwr["amenity"];
  nwr["shop"];
  nwr["healthcare"];
  nwr["man_made"="water_tap"];
);
out center tags;

Put both files in the data/ folder. Without them the Makoko chapter still works but shows no building map.


## Round 4: Tanzania (Ramani Huria) and Nairobi matatus

The page picks these files up automatically. Without them the chapters still work, with no data on the map.

### Dar es Salaam buildings: save as dar_buildings.geojson (Export > Data > GeoJSON)
Covers the Tandale / Magomeni / Hananasifu area. After Run, check on the map that it shows the Msimbazi valley.
[out:json][timeout:180][bbox:-6.795,39.240,-6.775,39.262];
way["building"];
out geom;

### Dar es Salaam drains: save as dar_drains.geojson (GeoJSON)
[out:json][timeout:180][bbox:-6.830,39.215,-6.760,39.290];
(
  way["waterway"~"drain|ditch|canal|stream|river|culvert"];
);
out geom;

### Nairobi matatu routes
1. Go to http://www.digitalmatatus.com/map.html and download the GTFS feed (a .zip file).
2. Send the zip to Claude as it is. Claude converts the route shapes for the map.


## Round 5: Google Open Buildings (optional outline layer)

The map already draws OpenStreetMap building footprints, coloured by walk distance. Google Open Buildings is an extra outline layer for comparison. It appears in the Layers panel only when the file below exists. I could not download it from my workspace, so this route is untested here.

1. Install the Overture Maps tool once (Python 3): `pip install overturemaps`
2. Download buildings for both areas (Overture combines Google, Microsoft and OpenStreetMap footprints):

   overturemaps download --bbox=36.765,-1.330,36.805,-1.300 -f geojson --type=building -o kibera_raw.geojson
   overturemaps download --bbox=36.835,-1.275,36.870,-1.250 -f geojson --type=building -o mathare_raw.geojson

3. Keep only the Google footprints and join the two areas. Save this as filter_google.py next to the two files and run `python filter_google.py`:

   import json
   out = []
   for name in ("kibera_raw.geojson", "mathare_raw.geojson"):
       for f in json.load(open(name))["features"]:
           src = f["properties"].get("sources") or []
           if isinstance(src, str):
               src = json.loads(src)
           if any("Google" in (s.get("dataset") or "") for s in src):
               out.append({"type": "Feature", "properties": {}, "geometry": f["geometry"]})
   json.dump({"type": "FeatureCollection", "features": out}, open("google_buildings.geojson", "w"))
   print(len(out), "Google footprints")

4. Put google_buildings.geojson in the data/ folder on GitHub. Credit: Google Open Buildings (CC BY 4.0, or ODbL where it comes through Overture), via Overture Maps Foundation.

If the file is larger than about 10 MB, send it to Claude and the footprints will be clipped to the settlement areas.
