# Final Image header
Montage has a command that will create the final image header information given the exposure images' header information. To generate this header one uses the command **mMakeHdr images.tbl template.hdr**. 

* template.hdr is the name of the generated header.

# Reprojection of Images

To create the mosaic one will first need to reproject the raw images. The command **mImgtbl rawdir images-rawdir.tb** will create a table with header infomation about the exposures.

* rawdir is the directory containing the exposures
* images_rawdir.tb is the file name for the created table

To run the reprojection the command **mProjExec -p rawdir images-rawdir.tbl template.hdr projdir stats.tbl** is used.

* template.hdr is a file containing the header information for the final mosaic image. An example is shown below.
* projdir is the directory where the reprojeted images will be stored.
* stats.tbl is a created file with status information for each image.

Example header information:

SIMPLE =          T / conforms to FITS standard  
BITPIX =           -64 /array data type   
NAXIS =            2 / number of array dimensions   
NAXIS1 =          2797  
NAXIS2 =          2797  
EXTEND =         T  
CTYPE1 =  'RA---TAN'  
CTYPE2 =  'DEC--TAN'  
CRVAL1 =      53.1153485  
CRPIX1 =       1024.0   
CRVAL2 =      -27.8015545   
CRPIX2 =      1024.0   
CD1_1 =        8.6466189E-06   
CD1_2 =       1.227034E-07   
CD2_1 =       -1.219408E-07  
CD2_2 =       8.7006938E-06   

END

The projected image information is given in a table by running the command **mImgtbl projdir images.tbl**.

* images.tbl is the name of the file containing this information.

# Uncorrected Mosaic

A mosaic is now created without background matching. The command used is **mAdd -p projdir images.tbl template.hdr final/example_uncorrected.fits**.

* final is the directory in which final mosaic images are stored
* example_uncorrected.fits is the filename of the uncorrected mosaic
* -p is the path to the directory with the reprojected images.

# Background Matching

Background matching is done to create a "corrected" mosaic.

First the overlaps of the reprojected images are found using **mOverlaps images.tbl diffs.tbl**.

* diff.tbl is a table describing the overlaps

The command **mDiffExec -p projdir diffs.tbl template.hdr diffdir** will subtract each pair of overlapping images and creates a set of difference images.

* diffdir is the directory where the difference images are stored

A plane fitting is now done on each difference image using **mFitExec diffs.tbl fits.tbl diffdir**.

* fits.tbl is a table with coefficients found from the fitting.

Backgorund removal is now done on the reprojected images. First a table is created with corrections to be applied to each image with command **mBgModel images.tbl fits.tbl corrections.tbl**.

* corrections.tbl is the file containing these corrections

The corrections are then applied with **mBgExec -p projdir images.tbl corrections.tbl corrdir**.

* corrdir is the directory containing corrected exposure images.

# Final Mosaic

Finally a mosaic is produced using **mAdd -p corrdir images.tbl template.hdr final/mosaic.fits**.

* mosaic.fits is the final mosaic image

# Create a Weight Map

Montage can generate a fits image of the number of times that an output pixel overlaps the input images, that is a weight map. The command used to generate this is **mAdd -p corrdir -a count images.tbl F090W_8exp.hdr final/full_F090W_8exp.fits**. 
Note that it is the same **mAdd** command as above expect the **-a** option is used. 

* count gives the number of times a pixel overlaps.