
# How to Run the Nonlinearity Apply Module

This document describes how to run the Euclid Nonlinearity apply module.

## Source code

The nonlinearity apply module source code is in project NIR_NonLinearSaturation: 
https://gitlab.euclid-sgs.uk/PF-NIR/NIR_NonLinearSaturation.
The main source code file is Nonlinearity.py.

In general, use the 'develop' branch.  This is the default under Euclid CVMFS.
Nothing special is needed, unless there are modifications in your local copy of the code.
In that case, you'll need to make & install first, and your modification should be
picked by the ERun.

## Command syntax

```
ERun NIR_NonLinearSaturation correctNonlinearity --help

usage: Nonlinearity correction
             ~> ERun NIR_NonLinearSaturation correctNonlinearity   \
                --xmlfile=The_MDB_file                             \
                --calxmlfile=ListOfCalProductsXmlFiles             \
                --config=$workdir/theNIRconfigset.xml              \
                --infile=input_data.fits                           \
                --outfile=output_data.fits                         \
                --workdir=./                                       \
                --logdir=./                                        \
             
       [-h] [--infile INFILE] [--xmlfile XMLFILE] [--calxmlfile CALXMLFILE] [--outfile OUTFILE] [--workdir WORKDIR] [--logdir LOGDIR]
       [--config CONFIG] [--config-file CONFIG_FILE] [--log-file LOG_FILE] [--log-level LOG_LEVEL] [--version]

optional arguments:
  -h, --help            show this help message and exit
  --infile INFILE       Input image frame Fits file
  --xmlfile XMLFILE     The MDB XML file
  --calxmlfile CALXMLFILE
                        JSON file listing the calibration products XML files
  --outfile OUTFILE     Output Fits file
  --workdir WORKDIR     Working directory path
  --logdir LOGDIR       Logging directory path
  --config CONFIG       The NIR configuration set, optional if using the nonlinearity model

Generic Options:
  --config-file CONFIG_FILE
                        Name of a configuration file
  --log-file LOG_FILE   Name of a log file
  --log-level LOG_LEVEL
                        Log level: FATAL, ERROR, WARN, INFO (default), DEBUG
  --version             show program's version number and exit

```

## Inputs

Task correctNonlinearity accepts the following arguments:
 * --infile: input image frame in FITS
 * --xmlfile: the MDB XML file
 * --calxmlfile: JSON file listing the nonlinearity calibration products XML files
 * --outfile: nonlinearity corrected image frame in FITS
 * --workdir: working directory path
 * --logdir: logging directory path
 * --config: the NIR configuration set, optional if using the nonlinearity model

Actual data needed include the following:
 * NISP gain file (data/EUC_NISP_GAIN-Flight_00.01.fits, specified in the MDB XML file)
 * nonlinearity calibration products (specified in calxmlfile):
   * photo coefficients (data/EUC_NIR_NLPHOTO_COEF-IPAC_20241211T190325.078510Z.fits)
   * photo covariance matrix (data/EUC_NIR_NLPHOTO_COVMAT-IPAC_20241211T190339.214700Z.fits)
   * photo failed mask (data/EUC_NIR_NLPHOTO_FAILED-IPAC_20241211T190439.915903Z.fits)

   * spectro coefficients (data/EUC_NIR_NLSPECTRO_COEF-IPAC_20241203T004842.582055Z.fits)
   * spectro covariance matrix (data/EUC_NIR_NLSPECTRO_COVMAT-IPAC_20241203T004900.174221Z.fits)
   * spectro failed mask (data/EUC_NIR_NLSPECTRO_FAILED-IPAC_20241203T004956.643629Z.fits)
 * nonlinearity limits file (data/EUC_NIR_NONLINEARITY_LIMITS.json) if config is specified

## Outputs

A nonlinearity corrected frame is the output, in FITS format.

## Test data

The following data can be used for test purposes.
There are LE1 images, containing 2 dithers for both Photo and Spectro modes.
See: /euclid/ops/data/xliu/F-001_240606_R2

As mentioned in the README file, 4 run steps had been executed:
runInitialize, badPixMasking, saturation, nonLinearity.
This directory contains runs inputs and outputs from these 4 steps.

* infile argument
  For the purpuse just running the nonlinearity apply module, one can start from 
  the saturation corrected results.  They are: 
  * /euclid/ops/data/xliu/F-001_240606_R2/outs/photo_sat.fits for Photo mode
  * /euclid/ops/data/xliu/F-001_240606_R2/outs/spectro_sat.fits for Spectro mode

* xmlfile argument
  input/EUC_MDB_MISSIONCONFIGURATION_DEV_2024-03-04-2.xml

* calxmlfile argument
  input/nonlinearcalibfile.json

* outfile argument
  your_dir/nl.fits

* workdir argument
  /euclid/ops/data/xliu/F-001_240606_R2

* logdir argument
  /euclid/ops/data/xliu/F-001_240606_R2/

* config argument
  This argument is optional if the nolinearity model is used.  It means that the calmin & calmax are
  from the nonlinearity coeficients array.

  Otherwise, it is:
  input/DpdNirConfigurationSet-NIR_PROCFIELD_CONFIG_GAIA_V5.0.1_20250214-0.xml
  Note that, this config file specifies the JSON file which contains upper and/or lower limits.
  In this case, it's data/EUC_NIR_NONLINEARITY_LIMITS.json.


