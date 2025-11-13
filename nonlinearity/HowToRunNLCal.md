
# How to Run the Nonlinearity Calibration Module

This document describes how to run the Euclid Nonlinearity calibration module.

## Source code

The nonlinearity calibration module (cal-module) source code is in project
NIR_NonLinearSaturation: https://gitlab.euclid-sgs.uk/PF-NIR/NIR_NonLinearSaturation.

In general, use the ‘develop’ branch for runs.  This is the default under Euclid CVMFS.
When there are modifications in your local copy of the code,
you’ll need to make & install first, and your modifications should be picked by the pipeline runner.

The nonlinearity IAL pipeline scripts are under project NIR_IAL_Pipelines:
https://gitlab.euclid-sgs.uk/PF-NIR/NIR_IAL_Pipelines/-/tree/develop/NIR_IAL_Pipelines/auxdir/NIR_Pipelines/NIR_Nonlinearity_Pipeline?ref_type=heads

To find out which version of the pipeline script to use, look under:
/cvmfs/euclid-dev.in2p3.fr/EDEN-3.1/opt/euclid/NIR_IAL_Pipelines/

Use one of the newer versions.  As of 7/11/2025, use 6.0+.

## Inputs

The inputs to the cal-module include the nonlinearity calibration data
taken during the PV phase and during the flight, the MDB XML file,
the NISP detector slot CSV file, and the NISP LED fluence look up table, also a CSV file.

The calibration datasets are the PV-018, F-008 and F-009,
taken from 11/2023 through current day.  These datasets have been downloaded from the
ESA and are staged on the ENSCI server.

The paths to these datasets are the following:

	/euclid/observations/PV/CALBLOCK_PV-018_Nov2023/data

	/euclid/observations/PV/CALBLOCK-F-008/data
 
	/euclid/observations/PV/CALBLOCK-F-009/data

Inside each of these calibration data directories, there are XML list files
which correspond to the data inside/under that directory.
One will need to assemble the right combinations of these list files before each cal-module run.

## Outputs

Three products are generated for each run of the cal-module: the nonlinearity 
coefficients, covariance matrix and failed mask.

A XML file associated with these 3 products is also generated.  It is located as combineData/outfile.xml.

The cal-module works on both MACC modes, Photo and Spectro.

## Environment

To be able to run the Euclid IAL pipeline runner, one needs to have a **bash** 
login shell on the ENSCI clusters.  One can login to any of the euclidops nodes to run the cal-module.

## Directory structure

IAL has certain directory structure assumptions:
```
  |-- your_current_run_dir
     |--data
     |--input
     |--runs (optional)
```

All the data file in FITS and CSV need to be under ‘data’ directory.
Other XML files should be under ‘input’ directory.  Directory ‘runs’ is 
optional but conveniently used to store files such as ‘run.dat’.

Due to the large volume of calibration data, it is recommended that
symbolic links are created inside the ‘data’ directory rather than copying.
A helper function to make the symbolic links is at the end of this document.

## Execution steps

```
###set the CONFIG environment variable
###assume $IAL_ROOT_DIR=/euclid/staff/your_login_name/euclid-ial/workspace for example
export CONFIG=$IAL_ROOT_DIR/eden3.1_pipelinerunner3.2.2_n_nodes.properties

###set the IAL pipeline runner
export PR=/cvmfs/euclid-dev.in2p3.fr/CentOS7/INFRA/1.1/opt/euclid/ST_PipelineRunner/3.2.8

###start the server in background (indefinitely)
cd $PR/bin
nohup ./python pipeline_runner.py server --config=$CONFIG &

###source /cvmfs/euclid-dev.in2p3.fr/CentOS7/EDEN-3.1/bin/activate if not already done
###make & install your projects (need link the /euclide/staff/your_login_name/Work/Projects to /home/your_login_name/
###Work/Projects as pipeline_runner looks under /home/your_login_name!!)

export PIPELINE=PipScript_NIR_Nonlinearity.py
export CWORKDIR=your_current_run_dir
export RUN_DAT=$IAL_ROOT_DIR/$CWORKDIR/runs/run.dat

###run the pipeline script
./python pipeline_runner.py submit --pipeline=$PIPELINE --data=$RUN_DAT

```

The above steps will start your pipeline using all available nodes/resources on the ENSCI cluster.
One can monitor the run using slurm command such as squeue.

IAL actually has a graphical monitor interface which is omitted here for simplicity reason. 

## Configuration file examples

### IAL config file (eden3.1_pipelinerunner3.2.2_n_nodes.properties)
```
pipelinerunner.rootDirectory=/euclid/staff/xliu/euclid-ial
workspace.rootPath=/euclid/staff/xliu/euclid-ial/workspace/
drm.submissionHost.protocol=SSH
drm.submissionHost.host=euclidslurm.ipac.caltech.edu
drm.submissionHost.username=your_login_name
drm.submissionHost.password=your_password_to_euclidslurm
#drm.submissionHost.useSSHKey=True                # [bool] True if SSH key should be used instead of password
#drm.submissionHost.SSHKeyPath=$HOME/.ssh/id_rsa.bastet      # [Path] Path to rsa key file.
drm.defaultQueue.scheduler=SLURM
drm.defaultQueue.destination=regular
pipelinerunner.webserver.useSSL=False
pipelinerunner.pilots.genericLight.CPUcores=15              #no of cores per node
pipelinerunner.pilots.genericLight.rssInMB=50000            #50G, total RAM requested
pipelinerunner.pilots.genericLight.walltimeInMin=10000
pipelinerunner.pilots.genericLight.maxInstances=8           #no of pilots to be use, a pilot can be a full node or part of a node
pipelinerunner.pilots.genericLight.maxQueued=6
pipelinerunner.profiling.updateIntervalInSec=15
pipelinerunner.messaging.requestTimeoutInSec=40
# [int]   Interval in seconds which the Pilot sends updates and asks for new jobs.
pipelinerunner.pilots.updateIntervalInSec=20
# [int]   Interval in seconds which the scheduling for new pilots gets triggered
pipelinerunner.pilots.schedulingIntervalInSec=30
# [int]   Max retries to talk to the PipelineRunner. Should be gracefull to survivie a restart.
pipelinerunner.pilots.maxConnectionRetries=40
# [int]   Max time in seconds before a pilot shutsdown due to idle state.
pipelinerunner.pilots.maxIdleTimeInSec=600
```

### run.data
```
workdir= your_current_run_dir
logdir=log
xmllistfile=input/xmllist_spectro_PV202311.json
mdbxmlfile=input/DpdMdbDataBase-EUC_MDB_MISSIONCONFIGURATION_SURVEY_2024-06-12-1-0.xml
pkgRepository=/cvmfs/euclid-dev.in2p3.fr/EDEN-3.1/opt/euclid/NIR_IAL_Pipelines/6.0/InstallArea/x86_64-conda_ry9-gcc11-o2g/auxdir/NIR_Pipelines
pipelineDir=/cvmfs/euclid-dev.in2p3.fr/EDEN-3.1/opt/euclid/NIR_IAL_Pipelines/6.0/InstallArea/x86_64-conda_ry9-gcc11-o2g/auxdir/NIR_Pipelines/NIR_Nonlinearity_Pipeline
edenVersion=eden-3.1-dev
```

Note: 

•	workdir is relative to $IAL_ROOT_DIR 
•	xmllistfile is the list of data for this run, in JSON format
•	use the latest MDB file
•	the version of the pipeline should match the one you plan to use
 
### Helper scripts (link_manyfiles.sh)
```
#!/bin/bash
## Make soft links of calibration data to a local directory
## Syntax:
##cd your_current_run_dir/data
##  ./link_manyfiles.sh  cal_data_dir   EUC_LE1_NISP  fits.gz

#data directory
dir=$1
#specific prefix for the data
fitsprefix=$2
#FITS file extension in either fits or fits.gz
fitsext=$3

files=`ls -1 $dir/${fitsprefix}*.${fitsext}` 
for f in $files
do
   target=$(basename $f)
   ln -s $f $target
done
```


