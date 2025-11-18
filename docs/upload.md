# Uploading data to lincbrain.org
**Note:** The steps below require access to https://lincbrain.org, which is a private repository for LINC project investigators.

## Contribute to an existing dataset or create a new dataset
Before you create a new dataset, please look over the existing datasets and make sure that there is no dataset appropriate for adding your new images to. Only create a new dataset as a last resort. If your images come from a brain that is already included in an existing dataset, definitely add to the existing dataset, even if the existing images come from a different part of that brain and/or were imaged with a different modality than yours. A dataset refers to a collection of brains that have been processed and imaged for a similar purpose. It typically contains data from multiple brains or samples, possibly imaged with multiple modalities.

1. Contribute to an existing dataset
    1. Log into the [LINC Data Platform](lincbrain.org).
    1. Browse the existing datasets under the [SHARED DATASETS](https://lincbrain.org/dandiset) tab.
    1. Contact the `Owner` by email requesting that they add you as an `Owner` of the dataset.
1. Create a new dataset
    1. Log into the [LINC Data Platform](lincbrain.org).
    1. Browse the existing datasets under the [SHARED DATASETS](https://lincbrain.org/dandiset) tab. Make sure that you have followed the guidelines above and there is no existing dataset that your new files can be added to.
    1. Click on the `NEW DATASET` button at the top right of the page.
    1. Fill out the title, description, and license. (The license option exists only because lincbrain.org is a clone of DANDI. It has no effect here, as lincbrain.org is a private repository.)
    1. Click on the `REGISTER DATASET` button to create the new dataset.

## Install the dandi command-line interface (CLI)
On your local machine, install the [dandi CLI package](https://pypi.org/project/dandi/) in a python environment:

`pip install dandi`

**Note:** If you are in the MGH LINC team, do not do this step. Instead, ask your labmates how to activate the existing dandi python environment.

## Copy your lincbrain API key
Log into lincbrain.org and click on the button with your initials at the top right of the page. Copy your API key and enter it in the following environment variable on your local machine:

`export DANDI_API_KEY=<EnterYourKeyHere>`

## Download your new (empty) dataset locally

### For a new dataset
You can find the command that you need to run to download a specific dataset by navigating to the dataset landing page on lincbrain.org, clicking on the `DOWNLOAD` drop-down menu that you'll see at the top right corner of that page, and copying the `dandi download ...` command that you see when you click on that menu. 

On your local machine, create a directory that you will use as a staging area for uploading data. Then change into this directory, and run the download command that you copied above. For example:
```
cd /path/to/my/staging/area
dandi download https://lincbrain.org/dandiset/101010/draft
```

The above example will create a directory called `/path/to/my/staging/area/101010` with a file called `dandiset.yaml` in it. Any data files that you want to upload to your new lincbrain.org dataset have to first be saved here, and organized according to the [Brain Imaging Data Structure (BIDS)](https://bids-specification.readthedocs.io/).

### For an existing dataset
For an existing dataset you will probably not want to download the entire dataset from lincbrain.org which can be many terabyes in size.  Using the command below, you can download the dataset with just the `dandiset.yaml` and `dataset_description.json` files.  You will need to replace `101010` with your `dandiset-id`.

```
cd /path/to/my/staging/area
dandi download --preserve-tree dandi://linc/101010@draft/dataset_description.json
```

The above example will create a directory called `/path/to/my/staging/area/101010`. Any data files that you want to upload to the lincbrain.org dataset have to first be saved here, and organized according to the [Brain Imaging Data Structure (BIDS)](https://bids-specification.readthedocs.io/).

For more information on the `--preserve-tree` option please see the [DANDI Docs](https://docs.dandiarchive.org/user-guide-using/accessing-data/downloading/#download-the-dandisetyaml-file-and-a-specific-file-within-the-directory-tree-of-the-dandiset).

## Organize your data
An example from a (fictional) dataset that includes dMRI and histology data from two brains is shown below. This can be used as a guide for organizing your own dataset. A few things to watch out for:
1. If you are creating a new dataset, you have to create the `dataset_description.json` file.
1. If you are adding data from a new subject to an existing dataset, you have to add an entry with the new subject's information to the `participants.tsv` file.
1. If you are adding data from a new sample (part of the brain) to an existing dataset, you have to add an entry with the new sample's information to the `samples.tsv` file.
1. Any data files that you add must go under:
   * `rawdata/` if they are raw (unprocessed) data in NIfTI or Zarr format
   * `sourcedata/` if they are raw (unprocessed) data in other formats
   * `derivatives/` if they are the outputs of pre-processing or other downstream analyses of the raw data 

```
101010/
  dataset_description.json
  participants.tsv
  samples.tsv
  rawdata/
    sub-Ken1/
      dwi/
        sub-Ken1_sample-brain_acq-DSI_dwi.bval
        sub-Ken1_sample-brain_acq-DSI_dwi.bvec
        sub-Ken1_sample-brain_acq-DSI_dwi.json
        sub-Ken1_sample-brain_acq-DSI_dwi.nii.gz
      micr/
        sub-Ken1_sample-slice0001_photo.json
        sub-Ken1_sample-slice0001_photo.tif
        sub-Ken1_sample-slice0001_stain-Nissl_BF.json
        sub-Ken1_sample-slice0001_stain-Nissl_BF.tif
        sub-Ken1_sample-slice0002_photo.json
        sub-Ken1_sample-slice0002_photo.tif
        sub-Ken1_sample-slice0002_stain-LY_DF.json
        sub-Ken1_sample-slice0002_stain-LY_DF.tif
        sub-Ken1_sample-slice0009_photo.json
        sub-Ken1_sample-slice0009_photo.tif
        sub-Ken1_sample-slice0009_stain-Nissl_BF.json
        sub-Ken1_sample-slice0009_stain-Nissl_BF.tif
        sub-Ken1_sample-slice0010_photo.json
        sub-Ken1_sample-slice0010_photo.tif
        sub-Ken1_sample-slice0010_stain-LY_DF.json
        sub-Ken1_sample-slice0010_stain-LY_DF.tif
    sub-Ken2/  
      dwi/
        sub-Ken2_sample-brain_acq-DSI_dwi.bval
        sub-Ken2_sample-brain_acq-DSI_dwi.bvec
        sub-Ken2_sample-brain_acq-DSI_dwi.json
        sub-Ken2_sample-brain_acq-DSI_dwi.nii.gz
        sub-Ken2_sample-brain_acq-MulShellMulTE_dwi.bval
        sub-Ken2_sample-brain_acq-MulShellMulTE_dwi.bvec
        sub-Ken2_sample-brain_acq-MulShellMulTE_dwi.json
        sub-Ken2_sample-brain_acq-MulShellMulTE_dwi.nii.gz
      micr/
        sub-Ken2_sample-slice0001_photo.json
        sub-Ken2_sample-slice0001_photo.tif
        sub-Ken2_sample-slice0001_stain-Nissl_BF.json
        sub-Ken2_sample-slice0001_stain-Nissl_BF.tif
        sub-Ken2_sample-slice0002_photo.json
        sub-Ken2_sample-slice0002_photo.tif
        sub-Ken2_sample-slice0002_stain-LY_DF.json
        sub-Ken2_sample-slice0002_stain-LY_DF.tif
        sub-Ken2_sample-slice0003_photo.json
        sub-Ken2_sample-slice0003_photo.tif
        sub-Ken2_sample-slice0003_stain-FR_DF.json
        sub-Ken2_sample-slice0003_stain-FR_DF.tif
        sub-Ken2_sample-slice0009_photo.json
        sub-Ken2_sample-slice0009_photo.tif
        sub-Ken2_sample-slice0009_stain-Nissl_BF.json
        sub-Ken2_sample-slice0009_stain-Nissl_BF.tif
        sub-Ken2_sample-slice0010_photo.json
        sub-Ken2_sample-slice0010_photo.tif
        sub-Ken2_sample-slice0010_stain-LY_DF.json
        sub-Ken2_sample-slice0010_stain-LY_DF.tif
        sub-Ken2_sample-slice0011_photo.json
        sub-Ken2_sample-slice0011_photo.tif
        sub-Ken2_sample-slice0011_stain-FR_DF.json
        sub-Ken2_sample-slice0011_stain-FR_DF.tif
  derivatives/
    sub-Ken1/
      dwi/
        ...
      scenes/
        ...
      xfms/
        ...
    sub-Ken2/
      dwi/
        ...
      scenes/
        ...
      xfms/
        ...
```

The files and subdirectories in this example dataset are described in detail below.

### Top-level files

#### dataset_description.json
This text file is [described in detail in the BIDS specification](https://bids-specification.readthedocs.io/en/stable/modality-agnostic-files.html#dataset_descriptionjson). A minimal `dataset_description.json` would look like this:

```
{
	"Name": "Seminal post mortem dMRI and histology dataset from the laboratory of Dr. Barbara Millicent Roberts",
	"BIDSVersion": "1.9.0"
}
```

#### participants.tsv
This text file is [described in detail in the BIDS specification](https://bids-specification.readthedocs.io/en/stable/modality-agnostic-files.html#participants-file). For this dataset, the `participants.tsv` might look like this:

```
participant_id age sex diagnosis
sub-Ken1 43 M healthy
sub-Ken2 61 M hypertension
```

#### samples.tsv
This text file is [described in detail in the BIDS specification](https://bids-specification.readthedocs.io/en/stable/modality-agnostic-files.html#samples-file). For this dataset, the `samples.tsv` would look like this:

```
participant_id sample_id sample_type
sub-Ken1 sample-brain tissue
sub-Ken1 sample-slice0001 tissue
sub-Ken1 sample-slice0002 tissue
sub-Ken1 sample-slice0009 tissue
sub-Ken1 sample-slice0010 tissue
sub-Ken2 sample-brain tissue
sub-Ken2 sample-slice0001 tissue
sub-Ken2 sample-slice0002 tissue
sub-Ken2 sample-slice0003 tissue
sub-Ken2 sample-slice0009 tissue
sub-Ken2 sample-slice0010 tissue
sub-Ken2 sample-slice0011 tissue
```

Label the sample as `brain` if the whole brain was imaged. Label it as `lefthemi` or `righthemi` if a whole hemisphere was imaged.

### rawdata/
This directory contains one subdirectory for each brain, which contains one subdirectory for each modality, which in turn contains raw image data files named according to the BIDS specification.

The name of every image or other data file under `rawdata/` must start with a subject and sample label, in that order. Examples: 
* `sub-Ken1_sample-brain_*.nii.gz`: NIfTI images of the whole brain of subject Ken1
* `sub-Ken2_sample-slice0001_*.json`: metadata files associated with images of a section of subject Ken2

#### dwi/
This directory contains dMRI data files [as described in detail in the BIDS specification](https://bids-specification.readthedocs.io/en/stable/modality-specific-files/magnetic-resonance-imaging-data.html#diffusion-imaging-data). 

In this example the data include images (`.nii.gz`), b-value tables (`.bval`), gradient tables (`.bvec`), and metadata (`.json`). The name of every image or other data file under `rawdata/dwi/` must contain the subject, sample, and acquisition label, in that order, followed by the `dwi` suffix. Examples:
* `sub-Ken1_sample-brain_acq-DSI_dwi.nii.gz`: NIfTI volumes of DWIs acquired with a DSI scheme from the whole brain of subject Ken1
* `sub-Ken2_sample-brain_acq-MulShellMulTE_dwi.bval`: b-value table of a scan acquired with a multi-shell, multi-echo scheme, from the whole brain of subject Ken2

#### micr/
This directory contains microscopy data files [as described in detail in the BIDS specification](https://bids-specification.readthedocs.io/en/stable/modality-specific-files/microscopy.html).

In this example the data include images (`.tif`) and metadata (`.json`) from multiple brain sections. For each section there is a blockface photo (`_photo`) and a histological stain (`_stain`). Sections from Ken1 and Ken2 were either processed with a Nissl stain and imaged under brightfield microscopy (`_BF`), or processed for the fluorescent tracer Lucifer Yellow (`LY`) and imaged under darkfield microscopy (`_DF`). Additional sections from Ken2 were processed for the fluorescent tracer Fluoro-Ruby (`FR`) and imaged under darkfield microscopy (`_DF`).

### derivatives/
This directory contains one subdirectory for each brain, which contains one subdirectory for each modality, which in turn contains any data files that were produced by analyzing the raw data of that modality. At the same level as the modality-specific directories, there can also be directories for saving files that are typically produced by co-analyzing images of more than one modality, e.g., `xfms/` for transform files or `scenes/` for scene files.

The name of every image or data file under `derivatives/` must start its subject and sample label, in that order. For any files that have been transformed to a space other than the native space, i.e., the one where the images were acquired, this must be followed by a space label. Do not use a space label for files that are in the native space. The name of the file must end with a description label that includes the methodology followed by the name of the derivative. Examples:
* `sub-Ken1_sample-brain_desc-DTI_FA.nii.gz`: a fractional anisotropy (FA) map derived from a diffusion tensor imaging (DTI) analysis of a whole-brain scan of subject Ken1, in its native (individual DWI) space
* `sub-Ken2_sample-brain_space-CIT168_desc-CSD_tractography.trk`: a tractogram derived from a constrained spherical deconvolution (CSD) analysis of a whole-brain scan of subject Ken2, transformed to CIT168 template space

#### Special case: high-res histology annotation
All annotation files use the following naming scheme:
`<dataset-name> + _desc-[label] + _suffix.ome.zarr`
where `[label]` is replaced by the annotator's initials and `_suffix` indicates the type of segmentations being annotated. Specifically, when annotating discrete segmentations, use `_dseg` as the suffix (see the [BIDS spec on discrete segmentations](https://bids-specification.readthedocs.io/en/stable/derivatives/imaging.html#discrete-segmentations)).

For example, an annotator with an initial JS annotating discrete segments would name the annotation file as `000003_sub-MR243_sample-slice0000slice0004_stain-LY_DF_desc-JS_dseg.ome.zarr`

A `<matches>.tsv` file could be included to map the IDs (integer values) of the discrete segmentations to the custom labels, where `<matches>` is replaced by the name of the annotation file. It contains a lookup table with the following columns (see the [BIDS spec on custom TSV](https://bids-specification.readthedocs.io/en/stable/derivatives/imaging.html#common-image-derived-labels)):
```
index  name
1      Single Fiber
2      Light Bundle
3      Moderate Bundle
...
```

## Upload your data
If you are uploading to a new or existing dataset on lincbrain.org, please ensure that you have the `dandiset.yaml` and `dataset_description.json` files in your local directory (i.e. the `/path/to/my/staging/area/101010` directory in the example above).

Upload the data from your local machine to lincbrain.org:

```
cd /path/to/my/staging/area/101010
dandi upload -i linc
```

Check the output in your terminal for validation errors. If there were no errors, your data files should now appear on lincbrain.org.

## Delete data files or directories
Individual data files can be deleted on lincbrain.org by clicking on the trashcan icon next to each file. Alternatively, directories or files can be deleted from the command line.

The following examples delete a directory named "horses" on lincbrain.org:

```
dandi delete -i linc /path/to/my/staging/area/101010/rawdata/Ken2/horses
```

```
dandi delete -i linc "https://lincbrain.org/dandiset/101010/draft/files?location=rawdata%2Fsub-Ken2%2Fhorses"
```
