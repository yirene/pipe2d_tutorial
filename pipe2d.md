---
title: PFS 2D pipeline pipe2d tutorial
---

# Installation

For the most up-to-date version of the pipe2d pipeline, installation
needs to be done through script. The script can be cloned from the
pfs_pipe2d repository.

`$ git clone http://github.com/Subaru-PFS/pfs_pipe2d  
$ cd pfs_pipe2d/bin`

Note: currently, pipe2d can only be installed to CentOS-7 system.

## Dependencies

Before installing the pipeline, make sure git-lfs is installed. You can
check it with command *git lfs install*. If it is not installed, you can
install the binaries and set up configurations as follows.

```$ sudo yum install -y epel-release
$ sudo yum install -y git-lfs
$ git lfs install```

You can also install git-lfs without admin permission. Choose a directory you want to install git-lfs in.
::: {.codebox}
$ wget https://github.com/git-lfs/git-lfs/releases/download/v3.4.0/git-lfs-linux-amd64-v3.4.0.tar.gz
$ tar xzf git-lfs-linux-amd64-v3.4.0.tar.gz$cd git-lfs-3.4.0
$ PREFIX=$(dirname $(command -v git)) ./install.sh
:::
Note: \textit{wget} sometimes does not work due to illegal characters. Please type the command instead of copying when error \textit{"No such file or directory"} appears.

The following packages are also required.
'$ install bison blas bzip2 bzip2-devel cmake curl flex fontconfig freetype-devel gawk gcc-c++ gcc-gfortran gettext git glib2-devel java-1.8.0-openjdk libcurl-devel libuuid-devel libXext libXrender libXt-devel make mesa-libGL ncurses-devel openssl-devel patch perl perl-ExtUtils-MakeMaker readline-devel sed tar which zlib-dev'

## Install LSST + PFS

Enter the directory where pfs_pipe2d is in.

'$ mkdir -p pfs 
$ mkdir -p pfs/stack 
$ cd pfs_pipe2d/bin 
$ ./install_pfs.sh -t current ../../pfs/stack 
$ source  /pfs/stack/loadLSST.bash 
$ setup pfs_pipe2d -t current'

Then, we need to install the flux model data.


'$ wget https://hscdata.mtk.nao.ac.jp/hsc_bin_dist/pfs/fluxmodeldata-ambre-20230608.tar.gz
$ tar xzf fluxmodeldata-ambre-20230608.tar.gz -C /PATH/TO/pfs_pipe2d
$ cd /PATH/TO/pfs_pipe2d/fluxmodeldata-ambre-20230608$$./install.py --prefix=/PATH #the package will be installed in /PATH/fluxmodeldata
$ setup -jr /PATH/fluxmodeldata'

## Testing installation
Version of loaded pipeline and packages can be checked by

'$ eups list -s pfs_pipe2d # check pipeline version
$ eups list # check all loaded packages'

Then run the integration test.
\begin{codebox}$mkdir -p /PATH/TO/integrationTest
$cd /PATH/TO/integrationTest
pfs_integration_test.sh -c 4 .
\end{codebox}

\section{Data processing}
Before starting data processing, you must activate the pipeline environment and load the necessary packages.
\begin{codebox}
source /PATH/TO/pfs/stack/loadLSST.bash
setup pfs_pipe2d -t current
setup -jr /PATH/TO/pfs/stack/fluxmodeldata

#If you were running the pipeline on Hilo server, set up the environment as follows.
source /work/stack/loadLSST.bash
setup pfs_pipe2d w.2023.45
setup -jr /work/fluxCal/fluxmodeldata-ambre-20230608-full/
\end{codebox}

Create a data repository and put \textit{\_mapper} in it.
\begin{codebox}
cd /PATH/TO/pfs
mkdir -p dataRepo
echo lsst.obs.pfs.PfsMapper > /PATH/TO/pfs/dataRepo/_mapper
\end{codebox}

Download images, calibration and pfsConfig from the Hilo server to the empty data repository created in the last step.\\
Ingest calibration and images first.
\begin{codebox}
# Ingest calibrations
# Note: it is suggested to put downloaded calibration files in directory /PATH/TO/pfs/dataRepo/CALIB/calib_test
ingestPfsCalibs.py /PATH/TO/pfs/dataRepo --rerun=CALIB --validity=1800 --longlog=1 --config clobber=True --mode=copy --doraise -- /PATH/TO/pfs/dataRepo/CALIB/calib_test/BIAS/*.fits
ingestPfsCalibs.py /PATH/TO/pfs/dataRepo --rerun=CALIB --validity=1800 --longlog=1 --config clobber=True --mode=copy --doraise -- /PATH/TO/pfs/dataRepo/CALIB/calib_test/DARK/*.fits
ingestPfsCalibs.py /PATH/TO/pfs/dataRepo --rerun=CALIB --validity=1800 --longlog=1 --config clobber=True --mode=copy --doraise -- /PATH/TO/pfs/dataRepo/CALIB/calib_test/FLAT/*.fits
ingestPfsCalibs.py /PATH/TO/pfs/dataRepo --rerun=CALIB --validity=1800 --longlog=1 --mode=copy --doraise -- /PATH/TO/pfs/dataRepo/CALIB/calib_test/FIBERPROFILES/*.fits
ingestPfsCalibs.py /PATH/TO/pfs/dataRepo --rerun=CALIB --validity=1800 --longlog=1 --mode=copy --doraise --config clobber=True -- /PATH/TO/pfs/dataRepo/CALIB/calib_test/DETECTORMAP/*.fits

# Ingest images
# Note: raw data file name is in the format "PF\%1sA\%06d\%1d\%1d.fits" (site, visit, spectrograph, armNum)
#armNum
#1: Blue
#2: Red
#3: IR
#4: Medium resolution red
# Note: raw data can be ingested only once
ingestPfsImages.py /PATH/TO/pfs/dataRepo --calib=/PATH/TO/pfs/dataRepo/CALIB --longlog=1 --mode=copy --doraise --pfsConfigDir=/PATH/TO/pfs/dataRepo/pfsConfig -- /PATH/TO/pfs/dataRepo/data/PFSA*.fits

# Note: /PATH/TO/pfs/dataRepo should be the same throughout the data processing
# Note: --calib=/PATH/TO/pfs/dataRepo/CALIB should contain registry.sqlite3 file
\end{codebox}
This step creates ingested calibrations in /PATH/TO/pfs/dataRepo/CALIB, and ingested images in /PATH/TO/pfs/dataRepo/OBSERVATION-DATE. It also put mask design used in ingestion (pfsConfig file) in /PATH/TO/pfs/dataRepo/pfsConfig/OBSERVATION-DATE.

Next, you need to set the defects.
\begin{codebox}
makePfsDefects --mko #--mko is for observation data taken in Mauna Kea. For integration test, it should be --lam
ingestCuratedCalibs.py /PATH/TO/pfs/dataRepo --calib /PATH/To/pfs/dataRepo/CALIB /PATH/TO/pfs/stack/stack/miniconda3-py38_4.9.2-3.0.0/Linux64/drp_pfs_data/w.2023.42/curated/pfs/defects
\end{codebox}

If you were processing PFS data on the Hilo server, the data and calibration are already ingested, and environments are already set, so you can skip these two steps.

If you were running pipe2d on Hilo server for the first time, you need to set the permission.
\begin{codebox}$ echo \*:\*:registry_gen2:pfs:pfs_hilo_opdb \> .pgpass
$chmod 0600 ~/.pgpass
\end{codebox}

You can now extract spectra using reduceExposure.py.
\begin{codebox}
# VISIT-ID is a 6 digit number
# All output files will be in directory /PATH/TO/pfs/dataRepo/OUTPUT$
reduceExposure.py /PATH/TO/pfs/dataRepo
--calib=/PATH/TO/pfs/dataRepo/CALIB -j=8 --rerun=OUTPUT --longlog=1
--doraise --id visit=VISIT-ID

\# If you were running the pipeline on Hilo server, use the following
arguments \# All output files will be in directory
/work/drp/rerun/USERNAME/test_processing
$reduceExposure.py /work/drp --calib=/work/drp/CALIB --rerun=USERNAME/test_processing -j=8 --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log

# For visitd without dithered flat, error "No locations for get: datasetType:flat" will rise. In this case, add argument --config isr.doFlat=False --clobber-config
# Set -j="$NCORES\" for parallel processing
:::

This step creates the following folders, config, postIsrCcd, apCorr,
pfsArm, arcLines, calExp, DETECTORMAP. *postIsrCcd* and *calExp* are
calibrated images. *calExp* add an extension of PSF model. *apCorr*
contains 2D spectral calibration. *pfsArm* contains reduced and
wavelength-calibrated spectra for each arm (flux calibration is not done
yet). *arcLines* contains information of spectral line measurements.

In the next step, you can merge spectra taken by different arms.

::: {.codebox}
$mergeArms.py /PATH/TO/pfs/dataRepo --calib=/PATH/TO/pfs/dataRepo/CALIB -j=8 --clobber-config --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID

# For Hilo server$ mergeArms.py /work/drp --calib=/work/drp/CALIB
--clobber-config --rerun=USERNAME/test_processing -j=8 --longlog=1
--doraise --id visit=VISIT-ID 2\>&1 \| tee -a test_processing.log
:::

This step output pfsMerged (merged spectra for each fiber).

Next step fits model spectra to observed fluxes, in preparation for flux
calibration.

::: {.codebox}
$fitPfsFluxReference.py /PATH/TO/pfs/dataRepo --calib=/PATH/TO/pfs/dataRepo/CALIB --clobber-config -j=8 --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID

# For Hilo server$ fitPfsFluxReference.py /work/drp
--calib=/work/drp/CALIB --clobber-config
--rerun=USERNAME/test_processing --longlog=1 --clobber-config --doraise
--id visit=VISIT-ID 2\>&1 \| tee -a test_processing.log
:::

This step outputs pfsFluxReference.

Then you can do flux calibration.

::: {.codebox}
$fitFluxCal.py /PATH/TO/pfs/dataRepo --calib=/PATH/TO/pfs/dataRepo/CALIB -j=8 --clobber-config --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID

# For Hilo server$ fitFluxCal.py /work/drp --calib=/work/drp/CALIB
--clobber-config --rerun=USERNAME/test_processing -j=8 --longlog=1
--doraise --id visit=VISIT-ID 2\>&1 \| tee -a test_processing.log
:::

It outputs fluxCal and pfsSingle (flux calibrated spectra from single
observation).

The final step is to coadd all spectra of repeat observations.

::: {.codebox}
$coaddSpectra.py /PATH/TO/pfs/dataRepo --calib=/PATH/TO/pfs/dataRepo/CALIB -j=8 --clobber-config --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID

# For Hilo server$ coaddSpectra.py /work/drp --calib=/work/drp/CALIB
--clobber-config --rerun=USERNAME/test_processing -j=8 --longlog=1
--doraise --id visit=VISIT-ID 2\>&1 \| tee -a test_processing.log
:::

The output is pfsObject.
