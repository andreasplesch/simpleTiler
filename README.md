# simpleTiler
gdal workflow/scripts to produce tiles for any format including ers which is raw binary

strategy is produce giant map first. Then cut into tiles and build pyramids.

1) determine base zoom level z0 according to resolution of source map: for example for 1km resolution base (highest zoom) is 8

2) use gdalwarp to project to spherical mercator and also conform to tile rectangle:

- for global coverage use these bounds min, max: -20037508.342789244 -20037508.342789244 20037508.342789244 20037508.342789244
- size of output map in pixels is 256 * 2 ^ zoom on each side, for level 256 * 256 = 65536

```gdalwarp -t_srs EPSG:3857 -te -20037508.342789244 -20037508.342789244 20037508.342789244 20037508.342789244 -ts 65536 65536 ../topo30.ers topo30epsg3857z8.ers```

3) use gdal_retile to tile and build pyramids

``` gdal_retile.py -v -targetDir tiles -of ERS -ps 256 256 -levels 8 -useDirForEachRow topo30epsg3857z8.ers ```

4) rename directories: 
- 0 is highest zoom, so rename to highest zoom - dirname
- second level is Y, so need to rearrange and rename
- names are basename_yyy_xxx.ers : x and yformatted to three digits (leading 0s).
- indexes start at 1
- remove leading 0's
```  for a in */*/*; do b=`echo $a | sed 's/_0*/_/g'`; mv $a $b; done ```
-  look into python script to avoid creating leading 0's

4b) populate XYZ hierarchy:
- z/x/y.ers is expected directory structure/namimg
- create global structure for each zoom (or just the needed structure for region, eg. CA)
- populate by copying or linking
- from z0-z/y+1/basename_yyy+1_xxx+1.ers to /z/x/y.ers
``` 
z0 = 8
n = "topo30epsg3857z8"
for z in *;
  do cd z;
  for x in *;
    do cd x;
    for y in *;
      ln -s ../../../tiles/$(($z0-$z))/$(($y+1))/${n}_$(($y+1))_$((x+1)) $y.raw
      done;
      ...```
      





