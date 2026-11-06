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
