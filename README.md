# SonarToes
Yes, the anagram addict strikes again

Stora Enso

# Data
Get OSM data for Sweden in Esri Shapefile format fromn http://download.geofabrik.de/europe/sweden.html

Direct link http://download.geofabrik.de/europe/sweden-latest-free.shp.zip

Unzip zip file to a folder

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

## Test for one shapefile

`gis_osm_landuse_a_free_1.shp`

```
cd /Users/emilzegers/sampledata/sweden-latest-free.shp

rm -rf gis_osm_landuse_a_free_1.json

ogr2ogr -skipfailures -makevalid -f GeoJSONSeq gis_osm_landuse_a_free_1.json gis_osm_landuse_a_free_1.shp

mongoimport --uri mongodb://127.0.0.1:27017/osm --collection sweden.landuse --file gis_osm_landuse_a_free_1.json --type json

```

Skip using options `-explodecollections -simplify .1 -makevalid -lco COORDINATE_PRECISION=5`

Do use options `-skipfailures -makevalid` to ensure valid geometries as result.

```
wc -l gis_osm_landuse_a_free_1.json
 1031098 gis_osm_landuse_a_free_1.json
```

Explore data with MongoDB Compass / shell. Check that `{'properties.fclass': 'forest'}` is ~43% (depeneding on sample).

Create 2dsphere index, I prefer to call the first one `geometry`.

Without index commands like near fail:

` planner returned error :: caused by :: unable to find index for $geoNear query, full error: {'ok': 0.0, 'errmsg': 'error processing query: ns=osm.sweden.roads limit=5Tree: GEONEAR  field=geometry maxdist=1000 isNearSphere=0\nSort: {}\nProj: {}\n planner returned error :: caused by :: unable to find index for $geoNear query', 'code': 291, 'codeName': 'NoQueryExecutionPlans'}`

Issues

db["sweden.landuse"].deleteMany({})

Create 2dsphere index on empty collection

No longer use `--drop`

Some thingies with mongoimport... Reports wrong number of imported documents

```
8 ], [ 14.74309, 63.15218 ], [ 14.7485, 63.14822 ], [ 14.73652, 63.14187 ] ]
2022-01-22T02:57:04.524+0100    [########################] osm.sweden.landuse   311MB/311MB (100.0%)
2022-01-22T02:57:04.524+0100    imported 1031098 documents
```

But works for this purpose

```
mongoimport --version
mongoimport version: r4.0.20
git version: e2416422da84a0b63cde2397d60b521758b56d1b
Go version: go1.11.13
   os: darwin
   arch: amd64
   compiler: gc
```

```
db["sweden.landuse"].countDocuments()

db["sweden.landuse"].find({'properties.fclass': 'forest'}).count()

db["sweden.landuse"].find({'properties.fclass': 'forest','properties.name': {$ne : null}}).count()
```

```
ogr2ogr -skipfailures -makevalid -f GeoJSONSeq gis_osm_roads_free_1.json gis_osm_roads_free_1.shp

mongoimport --uri mongodb://127.0.0.1:27017/osm --collection sweden.roads --file gis_osm_roads_free_1.json --type json
```

## Run for all shapefiles

Dummy...

For s in *.shp
Convert geojsonseq

For s in *.json
Mongoimport

# Notebook

Run the notebook inside VS Code or using `python3 -m jupyter notebook`


# Notes

Hi Johannes, were you able to get some stuff together for StoraEnso? I downloaded some data for Sweden in Shapefile format from http://download.geofabrik.de/europe/sweden.html Shapefile is table with geometry so 'flat'. Could be nice to take some parts of that data and store /use with MongoDB. I did not find time yet to do it myself but can do it over the weekend if it it useful for you. Would result in a 10-20 line writeup that you could repeat yourself. Then combine with a client app like QGIS or minimal web app (like https://github.com/taatuut/AchilleaMillefolium :slightly_smiling_face:) and off you go (and don't forget Compass of course!).

Key is to convert shapefiles to geojsonseq then they are ready for upload in mongodb. Might be useful to add some explode en valid commands, and create 2dsphere index upfront.
New

johannes.brannstrom:spiral_calendar_pad:  5:55 PM
For Monday i want so show some data modeling, doesn't have to be all the Bells an whistles, but for Wednesday I think this will be great. Actually it would be great for Monday too but I have not gotten my story straight yet so it would be tight



