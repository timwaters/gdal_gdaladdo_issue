A repo to help work out what was going on with a sequence of commands to add overviews to rasters.
The **fix** was to add add `-dstnodata none` to the gdalwarp call. Thanks to Even Rouault, lead of the GDAL project for helping me with this.


```
#set up
cp treasure_island.orig.tif treasure_island.tif
rm treasure_island_warped.tif

#Apply overviews to ungeoreferenced raster
gdaladdo -r average treasure_island.tif 2 4 8 16

#Rasterise ungeoreferenced raster with the cropping mask 
gdal_rasterize -i  -b 1 -b 2 -b 3 -burn 17 -burn 17 -burn 17  20.gml -l features treasure_island.tif

#Georeference the raster
gdal_translate -a_srs '+init=epsg:4326' -of VRT treasure_island.tif temp.vrt  -gcp 100.38, 83.096, -122.377, 37.830 -gcp 430.1966, 370.08, -122.363, 37.8207 -gcp 313.69, 61.68, -122.368, 37.831
gdalwarp -rn  -dstalpha -srcnodata '17 17 17' -s_srs 'EPSG:4326' temp.vrt treasure_island_warped.tif -co TILED=YES -co COMPRESS=LZW

#Apply overviews to georeferenced raster
gdaladdo -r average treasure_island_warped.tif 2 4 8 16

#Expected - transparent areas in overviews should be 17,17,17 with alpha of 0
#Observed - transparent areas in overviews are 17,17,17 with alpha also set to 17

gdaladdo -r nearest treasure_island_warped.tif 2 4 8 16
#Expected - transparent areas in overviews should be 17,17,17 with alpha of 0
#Observed - transparent areas in overviews are 17,17,17 with alpha of 0
```
Screenshots from Gimp

With average:

 ![wrong](observed_wrong.png)
 
 
With nearest:

 ![correct](observed_nearest_correct.png)


Showing what it looks like in mapwarper
 ![](in_mapwarper.png)

 # FIX

add -dstnodata none to the gdalwarp call.
```
gdalwarp -rn  -dstalpha -srcnodata '17 17 17' -dstnodata none -s_srs 'EPSG:4326' temp.vrt treasure_island_warped.tif -co TILED=YES -co COMPRESS=LZW
```