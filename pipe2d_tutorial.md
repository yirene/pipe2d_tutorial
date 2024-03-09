# Installation

For the most up-to-date version of the `pipe2d` pipeline, installation needs to be done through script. The script can be cloned from the `pfs_pipe2d` repository.

```
$ git clone http://github.com/Subaru-PFS/pfs_pipe2d  
$ cd pfs_pipe2d/bin
```
Note: currently, `pipe2d` can only be installed to CentOS-7 system.

## Dependencies

Before installing the pipeline, make sure `git-lfs` is installed. You can check it with command `git lfs install`. If it is not installed, you can install the binaries and set up configurations as follows.

```
$ sudo yum install -y epel-release
$ sudo yum install -y git-lfs
$ git lfs install
```

You can also install git-lfs without admin permission. Choose a directory you want to install git-lfs in.
```
$ wget https://github.com/git-lfs/git-lfs/releases/download/v3.4.0/git-lfs-linux-amd64-v3.4.0.tar.gz
$ tar xzf git-lfs-linux-amd64-v3.4.0.tar.gz
$ cd git-lfs-3.4.0
$ PREFIX=$(dirname $(command -v git)) ./install.sh
```

Note: `wget` sometimes does not work due to illegal characters. Please type the command instead of copying when error `No such file or directory` appears.

The following packages are also required.
```
$ install bison blas bzip2 bzip2-devel cmake curl flex fontconfig freetype-devel gawk gcc-c++ gcc-gfortran gettext git glib2-devel java-1.8.0-openjdk libcurl-devel libuuid-devel libXext libXrender libXt-devel make mesa-libGL ncurses-devel openssl-devel patch perl perl-ExtUtils-MakeMaker readline-devel sed tar which zlib-dev
```

## Install LSST + PFS

Enter the directory where `pfs_pipe2d` is in.
```
$ mkdir -p pfs/stack 
$ cd pfs_pipe2d/bin 
$ ./install_pfs.sh -t current ../../pfs/stack 
$ source  ../../pfs/stack/loadLSST.bash 
$ setup pfs_pipe2d -t current
```
`setup` command is used to load packages. You can use `-t` to select the package with a specific tag or specify the version at the end, e.g. `setup pfs_pipe2d w.2023.46`. When not specified, the default setting will load package with *current* tag. If you would like to remove a package from the environment, you can use `unsetup` command.


Then, we need to install the flux model data.
```
$ wget https://hscdata.mtk.nao.ac.jp/hsc_bin_dist/pfs/fluxmodeldata-ambre-20230608.tar.gz
$ tar xzf fluxmodeldata-ambre-20230608.tar.gz -C /PATH/TO/pfs_pipe2d
$ cd /PATH/TO/pfs_pipe2d/fluxmodeldata-ambre-20230608
$ ./install.py --prefix=/PATH/TO/pfs/stack
$ setup -jr /PATH/TO/pfs/stack/fluxmodeldata
```
The package will be installed in the path given with the flag `--prefix`. It is suggested to install `fluxmodeldata` under `/PATH/TO/pfs/stack`.
Here the flag `-j` means *just* this package, and the flag `-r PRODUCTDIR` gives the path to the package you would like to load.

## Testing installation
You can check the pipeline version by `eups list -s pfs_pipe2d` and check all loaded packages by `eups list`

Then you can test if the pipeline is working properly by running the integration test.
```
$ mkdir -p /PATH/TO/integrationTest
$ cd /PATH/TO/integrationTest
$ pfs_integration_test.sh -c 4 .
```

# Getting started of pipe2d pipeline
## Activate pipeline and load packages
There are some steps we need to do every time running pipe2d. 
First is to activate the pipeline. The pipe2d pipeline is activated as below.
```
$ source /PATH/TO/pfs/stack/loadLSST.bash
$ setup pfs_pipe2d -t current
```
If you were running the pipeline on the pfsa server, set up the environment as follows.
```
$ source /work/stack/loadLSST.bash
$ setup pfs_pipe2d -t current
```
Then we need to load some packages that are useful in building calibration and processing data.
```
$ setup -jr /PATH/TO/pfs/stack/fluxmodeldata
$ setup display_matplotlib -t current
```
On pfsa server,
```
$ setup -jr /work/fluxCal/fluxmodeldata-ambre-20230608-full/
$ setup display_matplotlib -t current
```
If you work on local data, you need to put all data into one data repository and put *_mapper* in it the first time you run the pipeline.
```
$ mkdir -p /PATH/TO/pfs/drp
$ echo lsst.obs.pfs.PfsMapper > /PATH/TO/pfs/drp/_mapper
```
If you were working on PFS data on the pfsa server, you need to set access to the postgreSQL repository when you run the pipeline for the first time. The data repository on pfsa server is `/work/drp`.
```
$ echo *:*:registry_gen2:pfs:pfs_hilo_opdb > .pgpass
$ chmod 0600 ~/.pgpass
```

## Basic information of pipe2d commands
`pipe2d` commands are used in the format `command input [option]` (input is the path to the input data repository). The data repository should stay the same throughout the calibration construction and data processing.

All `pipe2d` pipeline commands share the same set of arguments. `--rerun OUTPUT` sets OUTPUT to `/rerun/OUTPUT` relative to the input data repository path. When data repository is `/PATH/TO/pfs/drp`, the OUTPUT will be in `/PATH/TO/pfs/drp/rerun/OUTPUT`. `--config NAME=VALUE` sets config parameters, and the keywords are slightly different for each command. When there are many config parameters to set, `--configfile FILENAME` can read the config file. `--show config` can list all config keywords and their default values. `--clobber-config` can overwrite existing config files. `--validity=VALIDITY` sets the calibration validity period (in days). `--longlog 1` enables more verbose logging. `--doraise` can raise an exception on error. For parallel execution, `--cores CORES` can set the number of cores you would like to use for data processing.

## Set defects
We need to set defects before making calibration files and data processing.
```
$ makePfsDefects --mko 
$ ingestCuratedCalibs.py "$PFS_PATH"/drp --calib $PFS_PATH/drp/CALIB $DRP_PFS_DATA_DIR/curated/pfs/defects 2>&1 | tee -a test_processing.log
```
Here, `--mko` is for observation data taken in Mauna Kea, and the option is `--lam` for the integration test.
Note that `$DRP_PFS_DATA_DIR` should be a writable directory. On pfsa server, you can copy `/work/stack_INFRA-312/stack/miniconda3-py38_4.9.2-3.0.0/Linux64/drp_pfs_data/VERSION` (`VERSION` corresponds to pipeline version) to your own directory, e.g., `/home/USERNAME` or `/work/USERNAME`, then load it using `setup -jr /PATH/TO/drp_pfs_data`.

# Constructing calibrations (unfinished)
*Calibs* are calibration products where the behaviour of the instrument is modelled. Calibs are created in the order `BIAS`, `DARK`, `FLAT`, `FIBERPROFILES`, `DETECTORMAP`(wavelength solution). On pfsa server, `\work\drp\CALIB` already contains calibration products, which you can ingest and use. It is not necessary to make your own calibrations every time, but when there are major changes on the telescope, it is better to make new CALIBs.

Note: This section only discusses constructing calibs on the pfsa server. Commands should be similar if you work locally, but please take care of paths of data repository and calibs.

## Preparation
You can make an empty path as calib repository and another one to save output during calib construction. It is suggested to make a separate path to store log of each set of calibs.
```
$ mkdir -p /work/drp/PATH/TO/CALIB
$ mkdir -p /work/drp/rerun/USERNAME/PATH/TO/OUTPUT
$ mkdir -p /home/USERNAME/PATH/TO/LOG
$ CALIB=/work/drp/PATH/TO/CALIB
$ RERUN=USERNAME/PATH/TO/OUTPUT
$ LOG=/home/USERNAME/PATH/TO/LOG
```
You need to ingest some simulated detector maps, defects and IPC files to the calib repository first.
```
$ ingestPfsCalibs.py /work/drp --calib $CALIB $DRP_PFS_DATA_DIR/detectorMap/detectorMap-sim-*.fits --mode=copy --validity 1000 2>&1 | tee -a $LOG/ingest.log
$ ingestCuratedCalibs.py /work/drp --calib $CALIB $DRP_PFS_DATA_DIR/curated/pfs/defects 2>&1 | tee -a $LOG/ingest.log
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/CALIB/IPC/*.fits 2>&1 | tee -a $LOG/ingest.log
```

## Calibration products
The information of raw calibration images can be found on [wiki page](https://sumire.pbworks.com). You can find the data type, arm, observation date and other information useful in selecting data to process in the following steps. It is suggested to use visit ID to specify the raw calib images you want to process. If necessary, you can use `arm=ARM` to produce calibs of specific arms (e.g., `arm=b^r`. Check [datamodel](https://github.com/Subaru-PFS/datamodel/blob/a5eb4c04878dd58b2dfeb1eeda9a15fee8e7e717/datamodel.txt) for keywords).

You need to start with `BIAS`.
```
$ constructPfsBias.py /work/drp --calib $CALIB --rerun $RERUN --cores 16 --id visit=VISIT-ID 2>&1 | tee -a $LOG/bias.log
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/BIAS/*.fits 2>&1 | tee -a $LOG/ingest.log
```
This step will put `BIAS` products under `$CALIB/BIAS`.
Then `DARK`.
```
$ constructPfsDark.py /work/drp --calib $CALIB --rerun $RERUN --cores 16 --id visit=VISIT-ID 2>&1 | tee -a $LOG/dark.log
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/DARK/*.fits 2>&1 | tee -a $LOG/ingest.log
```
This step will put `DARK` products under `$CALIB/DARK`.
Next is `FLAT`. Note that a dithered flat is needed here.
```
$ constructFiberFlat.py /work/drp --calib $CALIB --rerun $RERUN --cores 16 --id visit=VISIT-ID arm=ARM 2>&1 | tee -a $LOG/flat.log
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/FLAT/*.fits 2>&1 | tee -a $LOG/ingest.log
```
This will put `FLAT` products under `$CALIB/FLAT`.
Afterwards, you can construct `FIBERPROFILES`.
```
$ constructFiberProfiles.py /work/drp --calib $CALIB --rerun $RERUN --cores 16 --id visit=VISIT-ID arm=ARM slitOffset=0.0 2>&1 | tee -a $LOG/fiberprofiles.log
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/FIBERPROFILES/*.fits 2>&1 | tee -a $LOG/ingest.log
```
This will put `FIBERPROFILES` products under `$CALIB/FIBERPROFILES`.
And last one is `DETECTORMAP`(wavelength solution).
```
$ reduceArc.py /work/drp --calib $CALIB --rerun $RERUN -j 16 --id visit=VISIT-ID arm=ARM 2>&1 | tee -a $LOG/detectormap.log
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/DETECTORMAP/*.fits 2>&1 | tee -a $LOG/ingest.log
```
This will put `DETECTORMAP` products under `$CALIB/DETECTORMAP`.

## Build calibs from scratch
### Dark, bias and flat
To build calibs from scratch, we need to make dark, bias and flat first. It is the same as in section *"Calibration products"*. 
```
$ constructPfsBias.py /work/drp --calib $CALIB --rerun $RERUN --batch-type=smp --cores 16 --doraise --id visit=VISIT-ID 2>&1 | tee -a $LOG/bias.log
$ constructPfsDark.py /work/drp --calib $CALIB --rerun $RERUN --batch-type=smp --cores 16 --doraise --id visit=VISIT-ID 2>&1 | tee -a $LOG/dark.log
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/BIAS/*.fits 2>&1 | tee -a $LOG/ingest.log
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/DARK/*.fits 2>&1 | tee -a $LOG/ingest.log
```
Note that at the current stage, we use fake `flat`, which can be ingested from `/work/drp/CALIB`. 
```
$ ingestPfsCalibs.py /work/drp --calib $CALIB --validity=1000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/CALIB/FLAT/*.fits 2>&1 | tee -a test_calib.log
```
Also note that `/work/drp/CALIB/FLAT/pfsFlat-2023-04-25-091223-n2.fits` is not usable at the moment (2024.02.09). In case there is no `flat` available, use config keyword `isr.doFlat=False` or `reduceExposure.isr.doFlat=False` for optical calibs, and `isr.doFlatNir=False` or `reduceExposure.isr.doFlatNir=False` for n2 calibs.

### Bootstrap detectormap
 `bootstrapDetectorMap.py` can transform the simulated `DETECTORMAP` to approach actual `DETECTORMAP` used in observation. It is recommended to run bootstrap for only one arm and one spectrograph at one time. 
Before running `bootstrapDetectorMap.py`, we need to ingest simulated `DETECTORMAP`.
```
$ ingestPfsCalibs.py /work/drp --calib /work/drp/rerun/USERNAME/CALIB $DRP_PFS_DATA_DIR/detectorMap/detectorMap-sim-*.fits --mode=copy --validity 100000 -c clobber=True 2>&1 | tee -a test_calib.log
```

Here is an example of bootstrap process.
```
$ bootstrapDetectorMap.py /work/drp --calib $CALIB --rerun $RERUN --flatId visit=VISIT-ID arm=ARM spectrograph=SPECTROGRAPH --arcId visit=VISIT-ID arm=ARM spectrograph=SPECTROGRAPH fiberStatus=["GOOD"] profiles.mask=['CR', 'BAD', 'SAT', 'NO_DATA', 'IPC'] profiles.findThreshold=500 spatialOffset=18.0 spectralOffset=-10.0 findLines.threshold=50 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=5 --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrap-[ARM][SPECTROGRAPH].log

```
In the flat is required to be a quartz at a slit dither of 0. Please check the observation information in advance to see which arm and spectrograph each visit corresponds to.
Configs `profiles.findThreshold`, `profiles.mask`, and `fiberStatus` can be used to constrain usable fibers.
After selecting the proper number of fibers, we need to set the configs for fitting. The first two keywords we need to set are `spatialOffset` and `spectralOffset`. To find the approximate values of `spatialOffset` and `spectralOffset`, we can overlap the simulated `DETECTORMAP` on top of the observed image, then count the pixels between simulated emission lines and observed emission lines. 
Once the offsets are fixed, we can adjust `findLines.threshold`, `readLineList.minIntensity`, and `rejThreshold` to increase the lines used in fitting and improve the fitting quality. The `readLineList.minIntensity` changes the number of lines in line list this command used to fit, for each arm, this value should be the same. `findLines.threshold` is the threshold for observed line used in the fitting. `rejThreshold` is used to remove outliers, but it is not suggested to change this value a lot. 
The more lines per fiber used in the fitting, the more reliable this fitting can be. Ideally, we want to have 10 lines/fiber. Another quick check we can do for bootstrap quality is the to check the rms in the log. A good fitting should have rms <0.1 in both axes for optical and rms <0.2 in both axes for near-infrared.

We need to remove ingested simulated detectormap in calib repository first. 
```
$ sqlite3 $CALIB/calibRegistry.sqlite3 'delete from detectormap where visit0=0'
$ rm $CALIB/DETECTORMAP/pfsDetectorMap-000000-*
```
Then we can ingest bootstrap detectormaps.
```
$ ingestPfsCalibs.py /work/drp --output=$CALIB --validity=1800 --doraise --mode=copy --config clobber=True -- /work/drp/rerun/$RERUN/DETECTORMAP/pfsDetectorMap-*.fits 2>&1 | tee -a $LOG/ingest.log
$ rm -r /work/drp/rerun/$RERUN/DETECTORMAP
```


### Construct fiber profiles
In this step, we make rough fiber profiles based on bootstrap detectormap. We need to know that the quality of bootstrap detectormaps are not good enough as a proper detectormap. So we can use `constructFiberProfiles.py`, which is not sensitive to detectormap to make fiber profiles in this step. Still we recommend to run for each arm and spectrograph separately.
```
$ constructFiberProfiles.py /work/drp --calib $CALIB --rerun $RERUN --id visit=VISIT-ID arm=ARM spectrograph=SPECTROGRAPH -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True --cores 16 --clobber-config 2>&1 | tee -a $LOG/fiberprofiles-[ARM][SPECTROGRAPH].log 
```
Configs `isr.doFlat=True`, `doAdjustDetectorMap=True`, and `adjustDetectorMap.doSlitOffsets=True` are necessary. For near-infrared arm, use config `isr.doFlatNir=True`.
Then we can ingest the fiberprofiles into calib repository.
```
$ ingestPfsCalibs.py /work/drp --output=$CALIB --validity=1800 --doraise --mode=copy --config clobber=True -- /work/drp/rerun/$RERUN/FIBERPROFILES/pfsFiberProfiles-*.fits 2>&1 | tee -a $LOG/ingest.log
$ rm -r /work/drp/rerun/$RERUN/FIBERPROFILES
```

### Construct detectormap
The next step we can use the fiber profiles to make detectormaps.
```
$ reduceArc.py /work/drp --calib $CALIB --rerun $RERUN --id visit=VISIT-ID spectrograph=SPECTROGRAPH  -c reduceExposure.isr.doFlat=True reduceExposure.isr.doFlatNir=True fitDetectorMap.doSlitOffsets=True -j 16 --clobber-config --no-versions 2>&1 | tee -a $LOG/detectormap-[ARM][SPECTROGRAPH].log
```
In this step, it is necessary to set configs `reduceExposure.isr.doFlat=True`, `reduceExposure.isr.doFlatNir=True` and `fitDetectorMap.doSlitOffsets=True`. 
Then we need to remove the bootstrap detectormaps in calib repository and ingest the detectormaps we make in this step.
```
$ sqlite3 $CALIB/calibRegistry.sqlite3 'delete from detectormap'
$ rm $CALIB/DETECTORMAP/pfsDetectorMap-*
$ ingestPfsCalibs.py /work/drp --output $CALIB --validity=1800 --doraise --mode=copy --config clobber=True -- /work/drp/rerun/$RERUN/DETECTORMAP/pfsDetectorMap-*.fits 2>&1 | tee -a $LOG/ingest.log
```

### Construct final fiber profiles
The previous fiber profiles were made from bootstrap detectormaps, which were not accurate. Now with appropriate detectormaps, we can make proper fiber profiles.
```
$ reduceProfiles.py /work/drp --calib $CALIB --rerun $RERUN --id visit=VISIT-ID arm=ARM spectrograph=SPECTROGRAPH --normId visit=VISIT-ID arm=ARM spectrograph=SPECTROGRAPH -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 reduceExposure.isr.doFlat=True reduceExposure.doAdjustDetectorMap=True reduceExposure.adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-b4.py --clobber-config 2>&1 | tee -a $LOG/final-fiberprofiles-[ARM][SPECTROGRAPH].log
```

And we can remove the previous inaccurate fiberprofiles and ingest the ones we build in this step.
```
$ sqlite3 $CALIB/calibRegistry.sqlite3 'delete from fiberprofiles'
$ rm $CALIB/FIBERPROFILES/pfsFiberProfiles-*
$ ingestPfsCalibs.py /work/drp --output=$CALIB --validity=1800 --doraise --mode=copy --config clobber=True -- /work/drp/rerun/$RERUN/FIBERPROFILES/pfsFiberProfiles-*-b4.fits 2>&1 | tee -a $LOG/ingest.log
```

## Check the quality of calibs
There are two QA tasks, extractionQA and detectorMapQA we can run to check the quality of calibs we made. To run these two tasks, we need to 

# Data processing
The data processing procedure follows the following flowchart.
![Alt text](flowchart.png)

When you work on local data, you first need to download them from pfsa server. You can download science images, calibrations and pfsConfig files from the pfsa server to the empty data repository.

The data processing starts with ingesting calibrations.  
Note: it is suggested to put downloaded calibration files in the directory `/PATH/TO/pfs/drp/CALIB`.  It is also suggested to execute all commands at the same location and write down a log to file `test_processing.log` by `2>&1 | tee -a test_processing.log`. 
```
$ PFS_PATH=/PATH/TO/pfs/
$ ingestPfsCalibs.py $PFS_PATH/drp --rerun=CALIB --validity=30 --longlog=1 --config clobber=True --mode=copy --doraise -- "$PFS_PATH"/drp/CALIB/BIAS/*.fits 2>&1 | tee -a test_processing.log
$ ingestPfsCalibs.py $PFS_PATH/drp --rerun=CALIB --validity=30 --longlog=1 --config clobber=True --mode=copy --doraise -- "$PFS_PATH"/drp/CALIB/DARK/*.fits 2>&1 | tee -a test_processing.log
$ ingestPfsCalibs.py $PFS_PATH/drp --rerun=CALIB --validity=30 --longlog=1 --mode=copy --doraise -- "$PFS_PATH"/drp/CALIB/FLAT/*.fits 2>&1 | tee -a test_processing.log
$ ingestPfsCalibs.py $PFS_PATH/drp --rerun=CALIB --validity=30 --longlog=1 --mode=copy --doraise -- "$PFS_PATH"/drp/CALIB/FIBERPROFILES/*.fits 2>&1 | tee -a test_processing.log
$ ingestPfsCalibs.py $PFS_PATH/drp --rerun=CALIB --validity=30 --longlog=1 --mode=copy --doraise --config clobber=True -- "$PFS_PATH"/drp/CALIB/DETECTORMAP/*.fits 2>&1 | tee -a test_processing.log
```

The next step is to ingest raw science images.  
Note: raw image file name follows the format `PF%1sA%06d%1d%1d.fits (site, visit, spectrograph, armNum)` (Check [datamodel](https://github.com/Subaru-PFS/datamodel/blob/a5eb4c04878dd58b2dfeb1eeda9a15fee8e7e717/datamodel.txt) for more information.)

Note: in each data repository, raw images can be ingested only once.
```
$ ingestPfsImages.py "$PFS_PATH"/drp --calib="$PFS_PATH"/drp/rerun/CALIB --longlog=1 --mode=copy --doraise --pfsConfigDir="$PFS_PATH"/drp/pfsConfig -- "$PFS_PATH"/drp/data/PFSA*.fits 2>&1 | tee -a test_processing.log
```
Note: `"$PFS_PATH"/drp` should be the same throughout the data processing. `--calib="$PFS_PATH"/drp/rerun/CALIB` should contain `registry.sqlite3` file.

This step creates ingested calibrations in `"$PFS_PATH"/drp/rerun/CALIB`, and ingested images in `"$PFS_PATH"/drp/OBSERVATION-DATE`. It also puts the mask design used in ingestion (pfsConfig file) in `"$PFS_PATH"/drp/pfsConfig/OBSERVATION-DATE`.

You can now extract spectra using `reduceExposure.py`.

```
$ reduceExposure.py "$PFS_PATH"/drp --calib="$PFS_PATH"/drp/rerun/CALIB -j=8 --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```
VISIT-ID in the command is a 6-digit number. When you want to reduce multiple visits, you can use .. to give a range and ^ to join different visits. All output files will be in `"$PFS_PATH"/drp/rerun/OUTPUT`.
Set `-j="$NCORES"` for parallel processing.

If you were running the pipeline on the pfsa server, use the following arguments. 
```
$ reduceExposure.py /work/drp --calib=/work/drp/CALIB --rerun=USERNAME/OUTPUT -j=8 --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```
All output files will be in `/work/drp/rerun/USERNAME/OUTPUT`.

There are some visits without corresponding dithered flats. `reduceExposure.py` will report the error "No locations for get: datasetType:flat". In this case, add argument `--config isr.doFlat=False --clobber-config`

This step creates the following folders, `config`, `postIsrCcd`, `apCorr`, `pfsArm`, `arcLines`, `calExp`, `DETECTORMAP`. `postIsrCcd` and `calExp` are calibrated images. `calExp` has an extra extension of PSF model. `apCorr` contains 2D spectral calibration. `pfsArm` contains reduced and wavelength-calibrated spectra for each arm (flux calibration has not been done yet). `arcLines` contains information on spectral line measurements.

In the next step, you can merge spectra taken by different arms.

```
$ mergeArms.py "$PFS_PATH"/drp --calib="$PFS_PATH"/drp/rerun/CALIB -j=8 --clobber-config --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```
On pfsa server
```
$ mergeArms.py /work/drp --calib=/work/drp/CALIB --clobber-config --rerun=USERNAME/test_processing -j=8 --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```

This step outputs `pfsMerged` (merged spectra for each visit).

The next step is to fit stellar spectra from the AMBRE stellar library to each of the observed FLUXSTD to derive a flux calibration vector.

```
$ fitPfsFluxReference.py "$PFS_PATH"/drp --calib="$PFS_PATH"/drp/rerun/CALIB --clobber-config -j=8 --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```
On pfsa server
```
$ fitPfsFluxReference.py /work/drp --calib=/work/drp/CALIB --clobber-config --rerun=USERNAME/test_processing --longlog=1 --clobber-config --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```

This step outputs `pfsFluxReference`. It is a collection of the best-fit model templates for all of FLUXSTDs.

Then you can do flux calibration.

```
$ fitFluxCal.py "$PFS_PATH"/drp --calib="$PFS_PATH"/drp/rerun/CALIB -j=8 --clobber-config --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```
On pfsa server
```
$ fitFluxCal.py /work/drp --calib=/work/drp/CALIB --clobber-config --rerun=USERNAME/test_processing -j=8 --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```

In this step, the flux calibration vectors are merged to generate a master flux calibration vector for each visit and it is applied to all of the science fibers in the same visit. It outputs `fluxCal` (the master flux calibration vector) and `pfsSingle` (flux-calibrated spectra from a single observation).

The final step is to combine all spectra of repeat observations.

```
$ coaddSpectra.py "$PFS_PATH"/drp --calib="$PFS_PATH"/drp/rerun/CALIB -j=8 --clobber-config --rerun=OUTPUT --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```
On pfsa server
```
$ coaddSpectra.py /work/drp --calib=/work/drp/CALIB --clobber-config --rerun=USERNAME/test_processing -j=8 --longlog=1 --doraise --id visit=VISIT-ID 2>&1 | tee -a test_processing.log
```

The output is `pfsObject` (stacked spectrum for each object).
