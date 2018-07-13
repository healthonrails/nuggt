#NeUroGlancer Ground-Truth: nuggt

[![Travis CI Status](https://travis-ci.org/chunglabmit/nuggt.svg?branch=master)](https://travis-ci.org/chunglabmit/nuggt)

This script uses Neuroglancer to collect
cell center ground truth for a 3-d cell
segmentation.

Usage:

Compile Neuroglancer according to the directions
[here](https://github.com/google/neuroglancer/blob/master/README.md)
and [here](https://github.com/google/neuroglancer/blob/master/python/README.md)

Start the static content server:
```bash
> npm run dev-server
```

Install *nuggt* like this:

```
> git clone https://github.com/chunglabmit/nuggt
> cd nuggt
> pip install nuggt
```

Run *nuggt* like this:
```
> nuggt --image <image-file> --output <points-json-file> [--min-distance <d>]
```
where
* *image-file* is the path to a .tif stack file
* *points-json-file* is the path to the points.json file where the
  voxel coordinates selected by the user are stored
* *d* is the minimum allowed distance in voxels between entered points

Commands available in the browser:
* *shift-a* - add a point at the current cursor (easiest to place cursor over
            the x-y, x-z or y-z displays to pick the point in 2-d)
* *shift-s key* - save the points to *points.json*
* *shift-d key* - delete the point nearest to the cursor

##yea-nay

**yea-nay** is a tool for quickly separating a list of points into two classes,
e.g. true positives and false positives. The user is presented with each point
in turn and can either choose "yea" or "nay" (vote yes for "yea" or vote 
no for "nay"). At the end, the list of points
in either class is written out. As-yet undecided points are displayed as
yellow dots, "yea" points are displayed as green dots and "nay" points are
displayed as red dots.

```commandline
> yea-nay --help
```
gives the command-line options for the tool. Inside the browser, the following
commands are available:

* *shift-y* - move the current point into the "yea" class and move to the next
point.
* *shift-n* - move the current point into the "nay" class and move to the next
point.
* *shift-[* - move to the next point, leaving the current point in whatever
class it is in.
* *shift-]* - move to the previous point, leaving the current point in
whatever class it is in.
* *control-q* - finish, writing the points to the "yea" and "nay" files as
designated on the command-line.

## nuggt-align

Align images using Neuroglancer.

**nuggt-align** is a command-line program that starts two neuroglancer
instances, one for a *moving image* and one for a *reference image*. The
purpose of the program is to establish common points of reference in each
image which can be used to warp the *moving image* onto the *reference image*.

Run **nuggt-align** like this:
```bash
> nuggt-align --moving-image <moving-image-file> \
              --reference-image <reference-image-file> \
              --segmentation <segmentation> \
              --points <points-file> \
              [--no-launch] \
              [--ip-address <ip-address>] \
              [--port <port>] \
              [--static-content-source <static-content-url>] \
              [--reference-voxel-size <reference-voxel-size>] \
              [--moving-voxel-size <moving-voxel-size]
```
where:
* **moving-image-file** is a 3D .tiff file containing the volume of the
image to be aligned to the reference image
* **reference-image-file** is a 3D .tiff file containing the volume of
the reference image, in the reference coordinate frame.
* **segmentation** is a 3D .tiff file containing the reference segmentation,
accompanying the reference image.
* **points-file** is a file to contain (or already containing) the
moving image reference points and their corresponding points in the 
reference image. **nuggt-align** initially loads the points from this file
if it exists and, when you save, it overwrites this file with the current
list of points.
* **--no-launch** if you want to start **nuggt-align** without
automatically creating new browser windows with the moving and
reference images.
* **--ip-address** the IP address to bind the webserver to. This
defaults to *localhost* which is appropriate for local use. Other
choices are *0.0.0.0* for all NICs on your machine or the IP address
of your machine on the network.
* **--port** the port number to bind to. The port number that the server
binds to. By default, *nuggt-align* uses any available port.
* **--static-content-source** the URL of the Node server serving the
static content. Default is to use the Neuroglancer default. If you are
running `npm run python-dev-server` on your machine, "http://localhost:8080"
is likely the correct choice
* **--reference-voxel-size** The size of a voxel in the reference image
(in nanometers). The three sizes should be specified, separated by commas,
e.g. "1000.0,1000.0,1000.0".
* **--reference-voxel-size** The size of a voxel in the moving image
(in nanometers). The three sizes should be specified, separated by commas,
e.g. "1000.0,1000.0,1000.0".

**nuggt-align** will print the URLs of the reference viewer and moving
viewer on startup. These can be used to launch browser instances for
editing.

**nuggt-align** works by maintaining a list of point annotations
for each image in the *correspondence-points* annotation layer. It
maintains one pair of points in the *edit* annotation layer. These are
the points currently being edited. The following commands control the
edit process:

* *control-mouse-button0* (typically left mouse button) - place an edit point
* *shift-e* - edit the currently selected point (move currently selected point
from the correspondence-points annotation layer to the edit layer).
* *shift-d* - done with the point currently being edited. This moves the
point in the edit annotation layer in the reference and moving views to the
correspondence-points layer.
* *shift-c* - clear. Clears edit points from the edit annotation layer
* *shift-n* - saves the point being edited and moves to the next point in
the correspondence-points annotation layer.
* *shift-p* - saves the point being edited and moves to the previous point
in the correspondence-points annotation layer.
* *shift-t* - transform. This places or moves the point in the edit annotation
layer in the moving view at the location that corresponds to the point in the
edit annotation layer in the reference view (according to the last warping
transform). You can place a point in the reference view, press *shift-t*, and
then see and edit its location in the moving view to correct the warp.
* *shift-w* - warp the moving layer's image to the reference layer using the
point correspondences.

## SITK-ALIGN

**sitk-align** aligns a moving image to a reference image based on work
done by Jae Cho to adapt SimpleElastix / SimpleITK to mouse brain images. The
application can both produce an aligned image and, more importantly, it can
take a set of reference points, stored in a JSON file and convert these to a
point file for **nuggt-align**.

### Installation

SimpleElastix has a non-standard installation procedure because it is primarily
a C++ application with Python bindings. It should be installed following
the [super build](https://github.com/SuperElastix/SimpleElastix#building-with-the-superbuild)
instructions. One installation method might be to create and activate an
Anaconda environment, pip install Nuggt, then run the superbuild in the
activated environment. Nuggt has been tested with SimpleElastix git hash
[9dfa8cb](https://github.com/SuperElastix/SimpleElastix/tree/9dfa8cb7c99e78b36f64bb6600b084b70960f166).

### Invocation

Invoke **sitk-align** like this:

```commandline
> sitk-align --moving-file <moving-file> --fixed-file <fixed-file> \
             [--fixed-point-file <fixed-point-file>] \
             [--alignment-point-file <alignment-point-file>] \
             [--aligned-file <aligned-file>]\
             [--final-grid-spacing <final-grid-spacing>]
```
where
* *moving-file* is the path to the moving 3-d image. See SimpleITK for accepted
formats *(hint - 3D tif works)*.
* *fixed-file* is the path to the fixed or reference 3-d image.
* *fixed-point-file* is the optional .json file of reference points in the
reference image's space. These will be converted to points in the moving-file's
space. The format is a list of lists with the inner list being the Z, Y and X
coordinates of the point, in that order.
* *alignment-point-file* is the optional .json file to be written. This file
will be in the format used by the *--points-file* argument of **nuggt-align**.
* *aligned-file* is the optional file name for the moving file, warped into the
fixed-point-file's space. The file will not be written if the switch is not
specified.
* *final-grid-spacing* is an optional parameter for the alignment algorithm
It specifies the number of voxels between bspline grid points in the x,
y and z directions - a higher number results in a more rigid alignment
whereas a lower number results in more local adaptability to deformation.
It should be specified as three comma-separated values, e.g. "32,32,32".

## make-alignment-file

The **make-alignment-file** command uses an alignment and a point
file, such as generated by **nuggt**, to make --points file for
nuggt-align.

The command line is

```commandline
make-alignment-file \
    --moving-image <moving-image> \
    --reference-points <reference-points> \
    --output <output-points> \
    [--xyz] \
    [--reference-key] <reference-key> \
    [--moving-key] <moving-key> \
    <transform-parameters-file-0> \
    [<transform-parameters-file-N>]
```

where
* **moving-image** is the path to the 3D tiff file of the moving image
as input into the alignment.
* **reference-points** are points, e.g. as edited by **nuggt** that will
serve as the keypoints of the transformation.
* **output** is the path to the output file for **nuggt-align**
* **--xyz** should be specified if the points in the reference-points
file are in the X, Y, Z rather than Z, Y, X order, e.g. if the reference-points
were created using **nuggt**.
* **reference-key** is the key name for the reference points in the JSON
dictionary. The default is "reference", but "moving" may be used if the 
reference and moving images were switched when creating the initial
transformation.
* **moving-key** is the key name for the moving image in the JSON output
dictionary. The default value is "moving".

The keyword parameters are followed by the positional parameters which are
a list of TransformParameters.txt files, in the order that they were output
by the transformation script, e.g. "TransformParameters.0.txt, 
TransformParameters.1.txt".

## count-points-in-region

**count-points-in-region** creates a .csv file listing brain regions and
counts of the points that are in them. It does this by warping the input points
to the reference segmentation, using an alignment. It finds the segment ID
in the reference segmentation for each point and looks up that segment ID
in the brain regions CSV input file. You can specify a granularity level
for the brain regions file - 7 is the finest grain and 1 is the coarsest.

To run **count-points-in-region**:

```commandline
count-points-in-region \
    --points <points-file> \
    --alignment <alignment-file> \
    --reference-segmentation <reference-segmentation-file> \
    --brain-regions-csv <brain-regions-file> \
    --output <output-file> \
    [--level <level>] \
    [--xyz]
```

where
* **points-file** is the list of points, e.g. as generated by **nuggt**.
* **alignment-file** is the alignment file generated by **nuggt-align* and
possibly rescaled using **rescale-alignment-file**.
* **reference-segmentation-file** is a .tif file containing the reference
segmentation, e.g. from the Allen Brain Atlas
* **brain-regions-file** is the accompanying .csv file to the reference
segmentation, e.g. AllBrainRegions.csv
* **output-file** is the CSV file generated by the program. If the level is 7,
then there are three columns in the file, a segment ID, the name and the
number of cells found in that region. If the level is 1 through 6, then there
are two columns (the segment ID column will be missing). The file has a header
that gives the column names, strings are quoted and the counts are not quoted.
* **level** is the atlas granularity level to be accessed with 7 as the finest
and 1 as the coarsest. At levels 1 through 6, the counts for segments in
a particular region will be accumulated together, at level 7, each segment
will be associated with its single region.
* **--xyz** is a flag that indicates that the points-file has each of its points
ordered as X, Y and Z. This flag should be specified if the points are from
**nuggt**.

## rescale-alignment-file

**rescale-alignment-file** takes an alignment file that was generated using
a downscaled and possibly flipped and rotated moving file and converts the
alignment into one in the original frame of reference. It uses the ratio between
the full-size stack's size and the dimensions of the moving file to scale
the alignment. If you've transposed the axes, you can correct this using
the *--x-index*, *--y-index* and *--z-index* switches. If you've flipped axes,
you can correct this using the *--flip-x*, *--flip-y* and *--flip-z* switches.

If you need to transpose axes, the --x-index, --y-index and --z-index values
refer to the index (0, 1, or 2) of the x, y and z axes in the alignment file
and image.

To run:

```commandline
rescale-alignment-file \
    --input <input-alignment-file> \
    --alignment-image <alignment-image> \
    --stack <image-file-pattern> \
    --output <output-alignment-file> \
    [--x-index <x-index>] \
    [--y-index <y-index>] \
    [--z-index <z-index>] \
    [--flip-x] \
    [--flip-y] \
    [--flip-z]

```