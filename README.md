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

Test for one shapefile

gis_osm_landuse_a_free_1.shp

ogr2ogr -explodecollections -skipfailures -f GeoJSONSeq gis_osm_landuse_a_free_1.json gis_osm_landuse_a_free_1.shp


Run for all

For s in *.shp
Convert geojsonseq


For s in *.json
Mongoimport




# Notes

Hi Johannes, were you able to get some stuff together for StoraEnso? I downloaded some data for Sweden in Shapefile format from http://download.geofabrik.de/europe/sweden.html Shapefile is table with geometry so 'flat'. Could be nice to take some parts of that data and store /use with MongoDB. I did not find time yet to do it myself but can do it over the weekend if it it useful for you. Would result in a 10-20 line writeup that you could repeat yourself. Then combine with a client app like QGIS or minimal web app (like https://github.com/taatuut/AchilleaMillefolium :slightly_smiling_face:) and off you go (and don't forget Compass of course!).

Key is to convert shapefiles to geojsonseq then they are ready for upload in mongodb. Might be useful to add some explode en valid commands, and create 2dsphere index upfront.
New

johannes.brannstrom:spiral_calendar_pad:  5:55 PM
For Monday i want so show some data modeling, doesn't have to be all the Bells an whistles, but for Wednesday I think this will be great. Actually it would be great for Monday too but I have not gotten my story straight yet so it would be tight



