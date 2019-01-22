

## Introduction

THis is a panorama stitching program written in C++ from scratch (without any vision libraries). It mainly follows the routine
described in the paper [Automatic Panoramic Image Stitching using Invariant Features](http://matthewalunbrown.com/papers/ijcv2007.pdf),


[![Build Status]

### Compile Dependencies:

* gcc >= 4.7 (Or VS2015)
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page)
* [FLANN](http://www.cs.ubc.ca/research/flann/) (already included in the repository, slightly modified)
* [CImg](http://cimg.eu/) (optional. already included in the repository)
* libjpeg (optional if you only work with png files)
* cmake or make

Eigen, CImg and FLANN are header-only, to simplify the compilation on different platforms.
CImg and libjpeg are only used to read and write images, so you can easily get rid of them.

On Ubuntu, install dependencies by: `sudo apt install build-essential sed cmake libjpeg-dev libeigen3-dev`

### Compile:
#### Linux / OSX / WSL (bash on windows)
```
$ make -C src
```
or
```
$ mkdir build && cd build && cmake .. && make
```
The default clang on OSX doesn't contain openmp support.
You may need gcc or different clang. See #16.


### Options:

Three modes are available (set/unset the options in ``config.cfg``):
+ __cylinder__ mode. Give better results if:
	+ You are __only__ turning left (or right) when taking the images (as is usually done), no
		translations or other type of rotations allowed.
	+ Images are taken with the same camera, with a known ``FOCAL_LENGTH`` set in config.
	+ Images are given in the left-to-right order. (I might fix this in the future)

+ __camera estimation mode__. No translation is the only requirement on cameras.
  It can usually work well as long as you don't have too few images.
  But it's slower because it needs to perform pairwise matches.

+ __translation mode__. Simply stitch images together by affine transformation.
  It works when camera performs pure translation and scene points are roughly at the same depth.  It also requires ordered input.

Some options you may care:
+ __FOCAL_LENGTH__: focal length of your camera in [35mm equivalent](https://en.wikipedia.org/wiki/35_mm_equivalent_focal_length). Only useful in cylinder mode.
+ __ORDERED_INPUT__: whether input images are ordered sequentially. has to be `1` in CYLINDER and TRANS mode.
+ __CROP__: whether to crop the final image to avoid irregular white border.

Other parameters are quality-related.
The default values are generally good for images with more than 0.7 megapixels.
If your images are too small and cannot produce satisfactory results,
it might be better to resize your images rather than tune the parameters.

### Run:

```
$ ./image-stitching <file1> <file2> ...
```

The output file is ``out.jpg``.

Before dealing with very large images (4 megapixels or more), it's better to resize them.

In cylinder/translation mode, the input file names need to have the correct order.

## Examples :

Zijing Apartment in Tsinghua University:
![dorm](results/apartment.jpg)
<!--
   -Zijing Playground in Tsinghua University:
   -![planet](https://github.com/ppwwyyxx/panorama/raw/master/results/planet.jpg)
	 -->


## Speed & Memory:
Tested on Intel Core i7-6700HQ, with `ESTIMATE_CAMERA` mode:

+ 11 ordered images of size 600x400: 3.2s.

Memory consumption is known to be huge with default libc allocator.
Simply using a modern allocator (e.g. tcmalloc, jemalloc) can help a lot.
Also, setting `LAZY_READ` to 1 can save memory at the cost of a minor slow down.

Peak memory in bytes (assume each input has the same w & h):

+ Without `LAZY_READ` option: max(finalw \* finalh \* 12, #photos \* w \* h \* 12 + #photos \* #matched\_pairs * 96 + #keypoints * 520)
+ With `LAZY_READ` option: max(finalw \* finalh \* 16, #threads \* w \* h \* 12, #photos \* #matched\_pairs * 96 + #keypoints * 520)

## Algorithms
+ Features: [SIFT](http://en.wikipedia.org/wiki/Scale-invariant_feature_transform)
+ Transformation: use [RANSAC](http://en.wikipedia.org/wiki/RANSAC) to estimate a homography or affine transformation.
+ Optimization: focal estimation, [bundle adjustment](https://en.wikipedia.org/wiki/Bundle_adjustment), and some straightening tricks.

## Quality Guidelines

To get the best stitching quality:
+ While rotating the camera for different shots, try to keep the position of camera lens static.
+ Keep the exposure parameters unchanged.
+ Do not shoot on moving objects.
+ Objects far away will stitch better.
+ The algorithm doesn't work well with wide-angle cameras where images are distorted heavily. Camera
	parameters are needed to undistort the images.


