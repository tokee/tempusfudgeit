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
