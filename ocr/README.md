# Dicom OCR Scraper

**important** this model would need to be rebuilt to work with more recent
python and packages. The older version is provided
at [vanessa/dicom-scraper](https://hub.docker.com/r/vanessa/dicom-scraper).
We will need to rebuild the model using python 3 and a later version of scikit-learn
to update the base container.
_______________________


This is currently under development, and builds a Docker image to run text (letter detection) 
on a demo image. You can either detect (just find and report) or clean the data 
(and save cleaned png images). If this is a route we want to go, the 
data can be saved as dicom proper.

## Docker
First, to build the image (or just skip to download and use version built on 
[Docker Hub](https://hub.docker.com/r/vanessa/dicom-scraper)):

```python
$ docker build -t pydicom/dicom-ocr-cleaner .
```

Then to run it, you can first see if it works:

```bash
$ docker run pydicom/dicom-ocr-cleaner --help
```

and you should see usage

```python
$ docker run pydicom/dicom-ocr-cleaner --help
usage: main.py [-h] [--input FOLDER] [--outfolder OUTFOLDER] [--detect]
               [--verbose]

Deid (de-identification) pixel scaping tool.

optional arguments:
  -h, --help            show this help message and exit
  --input FOLDER, -i FOLDER
                        input folder to search for images.
  --outfolder OUTFOLDER, -o OUTFOLDER
                        full path to save output, will use /data folder if not
                        specified
  --detect, -d          Only detect, but don't try to scrub
  --verbose, -v         if set, print more image debugging to screen.
```

We see that you should provide a folder with dicom files to the `--input` argument. 
If you want to see the image files preprocessed (with contenders in red boxes), 
you should also map a `--volume`. If you only want to detect (and not clean) 
you can use `--detect`.

### Detection
Let's cd to some folder with dicom images, and then map it (the `$PWD` to /data) 
in the container. We will specify `--input` to be `/data`, meaning the mapped 
folder with our images. Let's try just detection first

```python
$ cd dicom_folder
$ docker run --volume $PWD:/data pydicom/dicom-ocr-cleaner --input /data --detect
``` 

You'll see overly verbose output (this would be nice to replace with a progress bar) 
followed by the final summary of detection:

```bash
DETECTED: 84
SKIPPED:  27
CLEAN:    1
TOTAL:    112
```

### Clean after Detection
Now we will specify the same command, but without `--detect` so we also perform cleaning (note this is under development).

```
cd dicom_folder
$ docker run --volume $PWD:/data pydicom/dicom-ocr-cleaner --input /data
``` 

You'll see the pixels that are being cleaned, and the output (png files for preview) in the same folder (the deprecation warnings need to be disabled):

```bash
$ docker run --volume $PWD:/data pydicom/dicom-ocr-cleaner --input /data
DEBUG Found 5 contender files in data
DEBUG Checking 5 dicom files for validation.
/opt/anaconda2/lib/python2.7/site-packages/skimage/transform/_warps.py:84: UserWarning: The default mode, 'constant', will be changed to 'reflect' in skimage 0.15.
  warn("The default mode, 'constant', will be changed to 'reflect' in "
/opt/anaconda2/lib/python2.7/site-packages/skimage/feature/_hog.py:119: skimage_deprecation: Default value of `block_norm`==`L1` is deprecated and will be changed to `L2-Hys` in v0.15
  'be changed to `L2-Hys` in v0.15', skimage_deprecation)
Found 5 valid dicom files
Scrubbing (108,0,408,512)
Scrubbing (418,200,431,230)
Scrubbing (418,241,431,245)
Scrubbing (418,255,431,263)
Scrubbing (422,268,431,287)
Scrubbing (422,288,435,296)
Scrubbing (437,202,452,214)
Scrubbing (437,214,450,222)
Scrubbing (437,223,452,236)
Scrubbing (437,239,450,245)
============================================================
Scrubbing (48,255,63,271)
Scrubbing (50,249,63,253)
Scrubbing (54,276,63,295)
Scrubbing (54,296,67,304)
============================================================
Scrubbing (64,0,447,512)
Scrubbing (438,268,447,287)
Scrubbing (438,288,451,296)
Scrubbing (453,202,468,214)
Scrubbing (453,214,466,222)
Scrubbing (453,223,468,236)
Scrubbing (453,239,466,245)
============================================================
Scrubbing (90,184,103,214)
Scrubbing (90,225,103,229)
Scrubbing (90,239,103,247)
Scrubbing (94,252,103,271)
Scrubbing (94,272,107,280)
Scrubbing (109,0,474,512)
Scrubbing (109,198,122,206)
Scrubbing (109,223,122,229)
============================================================
Scrubbing (62,283,69,288)
Scrubbing (73,302,80,307)
Scrubbing (90,330,97,335)
============================================================
```


Complete credit for the base work goes to [@FraPochetti](http://francescopochetti.com/portfoliodata-science-machine-learning/), I just wrapped the functions in a container, added xvfb and other dependencies to (hopefully) reproduce most of the versions that he used, and then added functions to save to file.
