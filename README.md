# Data-Science-for-Remote-Sensing
---
A non-exhaustive set of useful tools I have been using while working in data science for remote sensing.

I am learning a lot about Data Science, Remote Sensing, and everything in between. A lot of the times, the documentations I found weren't dummy proof (at least not enough for me), so I started making my own documentations that helped me gain a lot of time. Hopefully this might help others in their projects, and getting more efficient at using some of these tools.

- Most of the wording uses the term _tif_ to refer to rasters, and _polygons_ to refer to vectors. I simply find it easier, since most rasters are in .tif, and most vectors are polygons in .geojson or (god forbid) in .shp; at least most of what I have encountered.
- Most of this will have a focus on working in Python.
- I have mostly worked on satellite imagery, which will also be reflected here.
(Mostly Sentinel images)

## Table of contents
- [GDAL terminal commands](#gdal-terminal-commands)
  - [Convert tif to 8bit (Byte)](#convert-tif-to-8bit)
  - [Convert tif to RGB & 8 bit](#convert-to-rgb-and-8bit)
  - [Get info about a tif](#get-info-about-a-tif)
  - [Warp a tif to specific cellsize and extent](#warp-a-tif-to-a-specific-cellsize-and-extent)
  - [Normalize tif](#normalize-tif)
  - [Normalize tif with empty data](#normalize-tif-with-empty-data)
  - [Change no data value to ```nan``` (or anything else)](#change-no-data-value-to-nan)
  - [Clip a tif to polygons](#clip-a-tif-to-polygons)
  - [Polygonize a tif to polygons](#polygonize-a-tif-to-polygons)

## [GDAL terminal commands](#gdal-terminal-commands)

A set of GDAL command line operations that are particularly useful when handling satellite images. I find GDAL to be particularly tricky to use, and the documentation to be quite overwhelming. There are only a few bits of codes I _really_ use, which can be found in this repo.

The GDAL documentation can be found [here](https://gdal.org/programs/index.html#raster-programs) for more details.

Stupid yet sometimes forgotten little reminder: you need to ```cd``` into the directory where your tifs are for these to work.

### [Convert tif to 8bit (Byte)](#convert-tif-to-8bit)
Converts a tif into 8 bits. This is particularly useful for feeding the images into other libraries that only support 8 bits, or reducing the 'complexity' of an image (when creating GLCMs, feeding images into a neural network, etc. this _can_ be useful).

For most Sentinel images, note that the min_value is roughly 0, and max_value 0.5
```
gdal_translate -ot Byte -co "COMPRESS=DEFLATE" -scale <min_value> <max_value> <input_file.tif> <output_8bit_file.tif>
```

### [Convert tif to RGB & 8 bit](#convert-to-rgb-and-8bit)
Satellite images can be very heavy, having multiple bands, and a high pixel range.
For visual validation, being able to load a lighter version of the image is pretty helpful (especially, if like me, you're pretty low on RAM). This converts a tif, only keeping the RGB bands and converting the pixel range to 8 bits.

- Note that for Sentinel 2, R, G & B are bands 1, 2 & 3. This can change for each sensor,
so modify accordingly
```
gdal_translate -b 1 -b 2 -b 3 -ot Byte -scale <input_file.tif> <output_RGB_8bit_file.tif>
```

### [Get info about a tif](#get-info-about-a-tif)
Very handy little command, simply gives a lot of information about a given tif,
directly in the command line, without having to open it.
```
gdalinfo <file.tif>
```

### [Warp a tif to specific cellsize and extent](#warp-a-tif-to-a-specific-cellsize-and-extent)
All images don't have the same cellsize, and even if they do, their cell grid don't all necessarily overlap perfectly. This can be a huge issue when trying to compare multiple images of the same area.

(Note that [EPSG 4326](https://epsg.io/4326) uses degrees as a unit. Use [EPSG 3857](https://epsg.io/3857) for meters).

(For Sentinel, in most cases: in degrees, X_resolution = Y_resolution = 0.00009)
```
gdalwarp -t_srs "EPSG:4326" -tr <X_resolution> <Y_resolution> -co "COMPRESS=DEFLATE" -crop_to_cutline -cutline <polygon_to_cut_to.geojson> <input.tif> <output.tif>
```

### [Normalize tif](#normalize-tif)
Normalizing the values of a tif between 0 and 1.

(Big warning if trying to run this in subprocess.run() in Python: Don't put the ```""``` in the ```--calc``` operation. It won't work.)
```
gdal_calc.py -A <input.tif> --outfile=<Normlized_output.tif> --calc="(A-A.min())/(A.max()-A.min())" --NoDataValue=-9999
```

### [Normalize tif with empty data](#normalize-tif-with-empty-data)
Normalizing the values of a tif containing empty data (no data), between 0 and 1.

First, it is necessary to [change the no data values by ```nan```](#change-no-data-value-to-nan) if it isn't already the case.

(Big warning if trying to run this in subprocess.run() in Python: Don't put the ```""``` in the ```--calc``` operation. It won't work.)
```
gdal_calc.py -A <input.tif> --outfile=<Normlized_output.tift> --calc="(A-A.nanmin())/(A.nanmax()-A.nanmin())"
```

### [Change no data value to ```nan``` (or anything else)](#change-no-data-value-to-nan)
Changing the no data values in a tif containing any to ```nan```.

Note that you can of course change it to anything else.

```
gdalwarp -dstnodata nan <input.tif> <output.tif>
```

### [Clip a tif to polygons](#clip-a-tif-to-polygons)
Clipping a tif to a polygon, or a multi-polygon.

If you get an error message beginning with: ```Self-intersection at or near point ...```, then adding ```--config GDALWARP_IGNORE_BAD_CUTLINE YES``` just after ```gdalwarp``` allows you to perform the clipping.
```
gdalwarp -cutline <polygons_to_clip_to.geojson> -crop_to_cutline <input.tif> <output.tif>
```

### [Polygonize a tif to polygons](#polygonize-a-tif-to-polygons)
Polygonizing a tif to polygons. I find these mostly useful for binary tifs, or tifs with a very limited amount of values (after a classification for example).
```
gdal_polygonize.py <input.tif> <output.geojson>
```
