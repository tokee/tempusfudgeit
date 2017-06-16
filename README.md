# Tempus Fudge It
Still images from video by keeping pixel coordinates in space but changing time

Original idea by Yves Maurer from Luxembourg.

## Status
Not a single line of code yet, just a place holder.


## Sample case
Record a (short) movie of a moving subject in resolution 1920x1080 pixels.
Pick the first line of pixels from the first frame, the second line of pixels from
the second frame and so forth, until 1080 lines has been collected. Build a still
image from the lines, from top to bottom.


## Spatial flexibility
Instead of taking the pixels line-by-line, they can be taken from anywhere on the
canvas. A time-map could be represented  as an 8- or 16-bit greyscale image, where
each pixel value (greyscale intensity) maps to a specific point in time.


## Temporal flexibility
Each time-map value could map directly to the frames ahead in time, with the value
1 being 1 frame ahead, 2 being 2 frames ahead etc. A more flexible mapping would be
to define the range of values (256 or 65536, depending on 8- or 16-bit resolution)
to represent a user-defined time-interval.


## Tech ideas
Performance is likely to be a challenge for fast processing. Storing 1 minute of
uncompressed pixels from a 25fps HD-movie takes about 1920x1080x3x25x60 = 7GB, so
it seems feasible to start by doing that. Thus there is no de-compression involved
after the initial generation: The program could simple seek directly to the needed
position.

Given a time-map in the same resolution as the movie, Tempus Fudge It could start
by creating a tiny bitmap of length 256 or 65536, each bit signifying if there are
any entries in the time-map with the matching value: When generating the image, it
could consider only the time-values that were present in the time-map.

For each represented time-value, the time-map would be scanned and the entries
matching the value would in turn trigger a fetch of the matching pixel stored in
the large uncompressed cache.

The advantage of this approach is that the many-times-iterated time-map could be
held in memory, while access to the uncompressed image cache could be sequential
with skip-aheads and work fine on storage.

Having speedy rendering of the end-image would make it possible to build an interface
where the starting frame (or time) for processing could be adjusted on the fly. The
time-mapping-factor could likewise be adjusted interactively.


## Proof of concept
Sample CC0 (Public Domain) video, download the one in 1920x1080 resolution:
https://pixabay.com/en/videos/rotating-sphere-motion-rotate-8848/

Dump frames as uncompressed RGV TIFF
```shell
mkdir frames ; ffmpeg -i Rotating\ -\ 8848.mp4 -pix_fmt rgb24 -compression_algo raw frames/frame_%04d.tif
```

Crop the relevant line from each frame using ImageMagic, use montage to create a full image from the slices
```shell
mkdir -p slices ; for Y in {0000..1079}; do echo -n "$Y " ; convert frames/frame_$(printf '%04d' $((10#$Y/5+1)) ).tif -crop 1920x1+0+${Y} +repage slices/h_${Y}.tif ; done
montage slices/h_*.tif -tile 1x -geometry 1920x1+0+0 poc_h.png

mkdir -p slices ; for Y in {0000..1919}; do echo -n "$Y " ; convert frames/frame_$(printf '%04d' $((10#$Y/8+1)) ).tif -crop 1x1080+${Y}+0 +repage slices/v_${Y}.tif ; done
montage slices/v_*.tif -tile x1 -geometry 1x1080+0+0 poc_v.png
```

View `poc_h.png` & `poc_v.png` after processing.


Is another video is used, adjust the parameters to the scripts accordingly. The division factors `5` and `8` are selected because the sample video only has 250 frames in total and the end image is 1920x1080 pixels (approximately 8*250 x 5x250).

## Loose notes
Determine starting offset for raw RGB data in uncompressed RGB TIFF:
```shell
tiffinfo -s frames/frame_0001.tif | grep -A 1 "[0-9]\+ Strips" | tail -n 1 | sed 's/ *0: \[ * \([0-9]\+\), * [0-9]*\]/\1/'
```
(likely to be 8, with the header being stored after the RGB data, at the end of the file)

