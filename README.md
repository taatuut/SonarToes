# SonarToes
What's in a repo name? So far not too much, this repo is just another exercise with geospatial data and tools.
For those cases when you don't have a proper product name at hand, just let Hermes, the ancient Greek god of science, art, speech, eloquence, writing and a few things more be your guide to come up with something. This time it was SonarToes, thanks Hermes.

# Data
Get OSM data for Sweden in Esri Shapefile format fromn http://download.geofabrik.de/europe/sweden.html

Direct link http://download.geofabrik.de/europe/sweden-latest-free.shp.zip

Unzip zip file to a folder.

# Prereqs
Assumes installed:

* gdal ogr command line tool
* QGIS
* MongoDB Compass
* MongoDB local and Atlas
* MongoDB Database tools
* Python 3

# Tools of choice
ogrinfo
ogr2ogr
mongoimport

# Steps

## Test for single shapefile
Test with `gis_osm_landuse_a_free_1.shp`

Use `GeoJSONSeq` as format for easy import into MongoDB creating one document per feature (can be MULTI* geometries).

```
cd /Users/emilzegers/sampledata/sweden-latest-free.shp

rm -rf gis_osm_landuse_a_free_1.json

ogr2ogr -skipfailures -makevalid -f GeoJSONSeq gis_osm_landuse_a_free_1.json gis_osm_landuse_a_free_1.shp

mongoimport --uri mongodb://127.0.0.1:27017/osm --collection sweden.landuse --file gis_osm_landuse_a_free_1.json --type json
```

Don't use options `-explodecollections -simplify .1 -lco COORDINATE_PRECISION=5`. It is not necessary to split MULTI* geometries with `-explodecollections` as MongoDB can handle these structures. Using `-simplify .1 -lco COORDINATE_PRECISION=5` distorts the data too much resulting in less accurate and less appealing visualisations.

Do use options `-skipfailures -makevalid` to ensure valid geometries as result.

```
wc -l gis_osm_landuse_a_free_1.json
 1031098 gis_osm_landuse_a_free_1.json
```

Create a `2dsphere` index upfront on `geometry` field (assuming geojson), you can give it any name but why not use just `geometry`.

Without index commands like near fail:

` planner returned error :: caused by :: unable to find index for $geoNear query, full error: {'ok': 0.0, 'errmsg': 'error processing query: ns=osm.sweden.roads limit=5Tree: GEONEAR  field=geometry maxdist=1000 isNearSphere=0\nSort: {}\nProj: {}\n planner returned error :: caused by :: unable to find index for $geoNear query', 'code': 291, 'codeName': 'NoQueryExecutionPlans'}`

Use a `mongoimport` version that continues on errors.

```
mongoimport --version
mongoimport version: r4.0.20
git version: e2416422da84a0b63cde2397d60b521758b56d1b
Go version: go1.11.13
   os: darwin
   arch: amd64
   compiler: gc
```

Explore data with MongoDB Compass / shell. Check that `{'properties.fclass': 'forest'}` is ~43% (depeneding on sample).

Can import some geojson documents and then run `db["sweden.landuse"].deleteMany({})` as q&d way to have the collection structure in place.

Some thingies with `mongoimport`... It reports the wrong number of actual imported documents (reports total, including skipped ones)

But works for this purpose.

```
db["sweden.landuse"].countDocuments()
db["sweden.landuse"].find({'properties.fclass': 'forest'}).count()
db["sweden.landuse"].find({'properties.fclass': 'forest','properties.name': {$ne : null}}).count()
```

Idem roads

```
ogr2ogr -skipfailures -makevalid -f GeoJSONSeq gis_osm_roads_free_1.json gis_osm_roads_free_1.shp

mongoimport --uri mongodb://127.0.0.1:27017/osm --collection sweden.roads --file gis_osm_roads_free_1.json --type json
```

## Run for all shapefiles
Dummy code...

```
For s in *.shp
Convert geojsonseq

For s in *.json
Mongoimport
```

# Notebook
Run the notebook inside VS Code or using `python3 -m jupyter notebook`

# Notes
An (Esri) Shapefile is a files (.shp, .shx, .dbf and a few more) dataset containing a table (so 'flat') structure with one column containing one type of geometric feature.

Process with ogr2ogr and import in MongoDB. Then combine with a client app like QGIS or minimal web app (like https://github.com/taatuut/AchilleaMillefolium :-) ) and off you go. And don't forget Compass of course! Add some 'new' attributes live in Compass and show ion reload in QGIS (note that teh MongoDB plugin for QGIS actually downloads the data so you need a full refresh... Be aware of that with large datasets.)
