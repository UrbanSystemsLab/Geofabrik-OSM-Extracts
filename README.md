# openStreetMaps-extracts
Method to extract layers and features from Open Street Maps to a local system.

Open Street Maps provides extracts that can be downloaded from various portals. This repository explains the steps for OSM Extracts procured from Geofabrik Download Service.

## Steps
- Download OSM Extract as .osm.pbf
- Extract required OSM data from PBF
- Convert OSM to GeoJSON
- Store GeoJSON in MongoDB for futher processing
- Export the required dataset from MongoDB
- Conver the exported data to GeoJSON and SHP File

### OSM Data Extracts
OSM Data Extracts are available through [Geofabrik Download Service](http://download.geofabrik.de/)
PBF Format ("Protocolbuffer Binary Format") 

### Convert PBF to OSM Format
Use [OSM convert](https://wiki.openstreetmap.org/wiki/Osmconvert) to extract the OSM data from PBF file. Bounding box parameter can also be used to clip the output extract within the specified region.

```sh
# Simple Conversion
central-america-latest.osm.pbf > osm-file.osm

# Clip the output to a bounding box
osmconvert central-america-latest.osm.pbf -b="-68.5355,16.8729,-63.4680,19.5727" -o="osm-file.osm"
```

### Convert OSM to GeoJson Format
User [OGR2OGR](http://www.gdal.org/ogr2ogr.html) commandline utility to convert OSM to GeoJSON format. Use the provided config file to extract all tags to GeoJSON.

[.ini Source ](https://raw.githubusercontent.com/BerryDaniel/georemedy-osm-arcgis/master/configuration/osmconf.ini)

It is also helpful to first convert the OSM data to SHP File for investigating the various layers in a GIS software of your choice.

```sh
# Extract multipolygons only as GeoJSON
ogr2ogr -f "GeoJSON" --config OSM_CONFIG_FILE osmconf.ini geojson-file.geojson osm-file.osm -skipfailures -overwrite multipolygons 

# Convert to SHP file
ogr2ogr -f "ESRI Shapefile" --config OSM_CONFIG_FILE osmconf.ini destination-folder osm-file.osm -skipfailures -overwrite -lco ENCODING=UTF-8
```

### Store to MongoDB
Use jq commandline utility to extract features from GeoJSON into a JSON file

```sh
jq  ".features" --compact-output geojson-file.geojson > features.json
mongoimport --db databaseName -c features --file "features.json" --jsonArray
```

### Aggregate buildings in MongoDB
```sh
# Count the number of buildings
db.features.count({'properties.building': "yes"})

# Export all buildings to 'buildings' collection
db.features.aggregate([{ $match: {'properties.building' : 'yes'} },{ $out: "buildings" }])

# Count buildings that have height attribute
 db.buildings.find({'properties.height': {$ne : 'null'}}).count()

```

### Export Buildings
```sh
mongoexport --db databaseName -c buildings --out "building_export.json" --jsonArray 

```

### Convert to JSON to GeoJSON

### Convert to GeoJSON to SHP File


