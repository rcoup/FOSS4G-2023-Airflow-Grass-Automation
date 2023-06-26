Pull the image
 ```bash 
docker pull mundialis/grass-py3-pdal:8.2.1-alpine
```
Create grass directory
 ```bash 
cd /opt
mkdir grass
```
Create intermedate folders
 ```bash 
cd grass
mkdir -p input output scripts
```

Run GRASS from a container in Docker
(important, being in /opt/grass)
 ```bash 
docker run -v ./grassdata:/grassdb -it mundialis/grass-py3-pdal:8.2.1-alpine bash
```

* on macOS with M1/M2, you need to enable Rosetta in _Docker…Settings…Features
  in Development_. Then `docker run` as above adding `--platform=linux/amd64`

cd to volume
 ```bash 
cd /grassdata
```

create mapset

 ```bash 
grass -c EPSG:32634 test
```


Import bands 4 and 8
 ```bash 
r.in.gdal input=T34TDM_20220715T093051_B04_10m.jp2 output=band_4
r.in.gdal input=T34TDM_20220715T093051_B08_10m.jp2 output=band_8
```

set region and create the NDVI
 ```bash 
g.region raster=band_4 -p
i.vi red=band_4 nir=band_8 viname=ndvi output=NDVI --overwrite
```

download geojson from https://overpass-turbo.eu/s/1wn5

import it
```bash 
v.import input=highways.geojson output=highways
```

buffer the highways
```bash 
v.buffer input=highways output=highways_buffer distance=30 --overwrite
```

clip the region
```bash 
g.region vector=highways_buffer -p
```
Create the mask
```bash 
r.mask vector=highways_buffer
```

Export the PNG and quit the mask
```bash 
r.out.png input=NDVI output=/grassdb/output/prueb_ndvi -w --overwrite
r.mask -r
```






UTILS
g.region vector=highways_buffer -p
g.region raster=highways_buffer -p
grass PERMANENT --exec bash /grassdb/scripts/ndvi.sh
