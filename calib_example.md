```
source /work/stack/loadLSST.bash
git lfs install
setup pfs_pipe2d w.2023.50
setup obs_pfs w.2023.50
setup -jr /work/fluxCal/fluxmodeldata-ambre-20230608-full/
setup -jr /work/USERNAME/drp_pfs_data
setup display_matplotlib -t current
setup -jr /work/wtg/drp_qa


mkdir -p /work/drp/rerun/USERNAME/test_calib
mkdir -p /home/USERNAME/log_calib

CALIB=/work/drp/rerun/USERNAME/test_calib_05mar24
RERUN=USERNAME/test_calib/calib_test
LOG=/home/USERNAME/log_calib

makePfsDefects --mko
ingestCuratedCalibs.py /work/drp --calib=$CALIB $DRP_PFS_DATA_DIR/curated/pfs/defects 2>&1 | tee -a $LOG/ingest.log

ingestPfsCalibs.py /work/drp --calib=$CALIB --validity=1800 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/CALIB/IPC/*.fits 2>&1 | tee -a $LOG/ingest.log


constructPfsBias.py /work/drp --calib=$CALIB  --rerun=$RERUN --batch-type=smp --cores 16 --doraise --id visit=103173..103197 2>&1 | tee -a $LOG/BIAS-DARK-FLAT.log
ingestPfsCalibs.py /work/drp --calib=$CALIB --validity=1800 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/BIAS/pfsBias-*.fits 2>&1 | tee -a $LOG/ingest.log

constructPfsDark.py /work/drp --calib=$CALIB --rerun=$RERUN --batch-type=smp --cores 16 --doraise --id visit=103198..103222 2>&1 | tee -a $LOG/BIAS-DARK-FLAT.log
ingestPfsCalibs.py /work/drp --calib=$CALIB --validity=1800 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/rerun/$RERUN/DARK/pfsDark-2023-12-16-103198-*.fits 2>&1 | tee -a $LOG/ingest.log

ingestPfsCalibs.py /work/drp --calib=$CALIB --validity=18000 --longlog=1 --config clobber=True --mode=copy --doraise -- /work/drp/CALIB/FLAT/pfsFlat*.fits 2>&1 | tee -a $LOG/ingest.log
ingestPfsCalibs.py /work/drp --calib $CALIB $DRP_PFS_DATA_DIR/detectorMap/detectorMap-sim-*.fits --mode=copy --validity 100000 -c clobber=True 2>&1 | tee -a $LOG/ingest.log
```
## database manipulation ##
```
sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE bias SET validEnd="2023-12-08T00:00:00" WHERE visit0=96834 and spectrograph in (1, 3)'
sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE bias SET validEnd="2023-12-08T00:00:00" WHERE visit0=102122 and spectrograph=2'
sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE bias SET validStart="2023-12-08T00:00:01" WHERE visit0=103173 and spectrograph in (1, 2, 3)'

sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE dark SET validEnd="2023-12-08T00:00:00" WHERE validEnd="2023-12-16T07:28:50.806166"'
sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE dark SET validStart="2023-12-08T00:00:01" WHERE validStart="2023-12-16T07:28:50.806167"'

sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE fiberProfiles SET validEnd="2023-07-19T00:00:00" WHERE visit0=91014;'
sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE fiberProfiles SET validEnd="2023-07-19T00:00:00" WHERE visit0=91427;'
sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE fiberProfiles SET validEnd="2023-07-19T00:00:00" WHERE visit0=91942;'
sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE fiberProfiles SET validStart="2023-07-19T00:00:01" WHERE visit0=97774;'
sqlite3 $CALIB/calibRegistry.sqlite3 'UPDATE fiberProfiles SET validStart="2023-07-19T00:00:01" WHERE visit0=97779;'
```

# Construct bootstrap detectorMap

# b1
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=102562 arm=b spectrograph=1 --arcId visit=102558 arm=b spectrograph=1 -c isr.doFlat=True profiles.findThreshold=500 spatialOffset=18.0 spectralOffset=-10.0 findLines.threshold=50 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=5 -C configs/sm1.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapb1.log
```
# r1
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=102561 arm=r spectrograph=1 --arcId visit=102559 arm=r spectrograph=1 -c isr.doFlat=True profiles.findThreshold=500 spatialOffset=-16.5 spectralOffset=-5.5 findLines.threshold=100 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=5 -C configs/sm1.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapr1.log
```
# n1 
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=102561 arm=n spectrograph=1 --arcId visit=102560 arm=n spectrograph=1 -c isr.doFlatNir=True profiles.findThreshold=300 spatialOffset=8.0 spectralOffset=20.0 findLines.threshold=20 readLineList.minIntensity=20 spatialOrder=2 spectralOrder=2 rejThreshold=5 -C configs/sm1.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapn1.log
```
# m1
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=102564 arm=m spectrograph=1 --arcId visit=102563 arm=m spectrograph=1 -c isr.doFlat=True profiles.findThreshold=500 spatialOffset=6.0 spectralOffset=-20.5 findLines.threshold=20 readLineList.minIntensity=20 spatialOrder=2 spectralOrder=2 rejThreshold=5 -C configs/sm1.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapm1.log
```
# b2
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103155 arm=b spectrograph=2 --arcId visit=103152 arm=b spectrograph=2 -c isr.doFlat=True profiles.findThreshold=100 spatialOffset=-7.5 spectralOffset=13.0 findLines.threshold=50 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=5 -C configs/sm2.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapb2.log
```
# r2
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103155 arm=r spectrograph=2 --arcId visit=103153 arm=r spectrograph=2 -c isr.doFlat=True profiles.findThreshold=100 spatialOffset=9.0 spectralOffset=2.5 findLines.threshold=100 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=5 -C configs/sm2.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapr2.log
```
# n2
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103155 arm=n spectrograph=2 --arcId visit=103154 arm=n spectrograph=2 -c isr.doFlatNir=False profiles.findThreshold=100 spatialOffset=2.0 spectralOffset=9.4 findLines.threshold=80 readLineList.minIntensity=20 spatialOrder=2 spectralOrder=2 rejThreshold=2 -C configs/sm2.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapn2.log
```
# m2
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103157 arm=m spectrograph=2 --arcId visit=103156 arm=m spectrograph=2 -c isr.doFlat=True profiles.findThreshold=100 spatialOffset=3.5 spectralOffset=-4.5 findLines.threshold=20 readLineList.minIntensity=20 spatialOrder=2 spectralOrder=2 rejThreshold=5 -C configs/sm2.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapm2.log
```
# b3
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103142 arm=b spectrograph=3 --arcId visit=103138 arm=b spectrograph=3 -c isr.doFlat=True profiles.findThreshold=500 spatialOffset=2 spectralOffset=-10 findLines.threshold=10 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=3 -C configs/sm3.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapb3.log
```
# r3
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103141 arm=r spectrograph=3 --arcId visit=103139 arm=r spectrograph=3 -c isr.doFlat=True profiles.findThreshold=500 spatialOffset=-3.7 spectralOffset=8.8 findLines.threshold=70 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=3 -C configs/sm3.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapr3.log
```
# n3
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103141 arm=n spectrograph=3 --arcId visit=103140 arm=n spectrograph=3 -c isr.doFlat=False isr.doFlatNir=True profiles.findThreshold=300 spatialOffset=6.5 spectralOffset=-3.5 findLines.threshold=20 readLineList.minIntensity=20 spatialOrder=2 spectralOrder=2 rejThreshold=2.5 -C configs/sm3.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapn3.log
```
# m3
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103144 arm=m spectrograph=3 --arcId visit=103145 arm=m spectrograph=3 -c isr.doFlat=True isr.ampOffset.detection.minPixels=3 profiles.findThreshold=500 spatialOffset=-19.0 spectralOffset=0.0 findLines.threshold=20 readLineList.minIntensity=20 spatialOrder=2 spectralOrder=2 rejThreshold=3.8 -C configs/sm3.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapm3.log
```
# b4
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103031 arm=b spectrograph=4 --arcId visit=103032 arm=b spectrograph=4 -c isr.doFlat=True profiles.findThreshold=500 spatialOffset=-6.5 spectralOffset=-13.5 findLines.threshold=40 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=4 -C configs/sm4.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapb4.log
```

# r4
```
bootstrapDetectorMap.py /work/drp --calib=$CALIB --rerun=$RERUN --flatId visit=103034 arm=r spectrograph=4 --arcId visit=103036 arm=r spectrograph=4 -c isr.doFlat=True profiles.findThreshold=500 spatialOffset=9.0 spectralOffset=-11.0 findLines.threshold=200 readLineList.minIntensity=50 spatialOrder=2 spectralOrder=2 rejThreshold=4 -C configs/sm4.config --no-versions --clobber-config 2>&1 | tee -a $LOG/bootstrapr4.log
```
# remove simulated detector map
```
sqlite3 $CALIB/calibRegistry.sqlite3 'delete from detectormap where visit0=0'
rm $CALIB/DETECTORMAP/pfsDetectorMap-000000-*
```
# ingest bootstrap detectormap
```
ingestPfsCalibs.py /work/drp --output=$CALIB --validity=1800 --doraise --mode=copy --config clobber=True -- /work/drp/rerun/$RERUN/DETECTORMAP/pfsDetectorMap-*.fits 2>&1 | tee -a $LOG/ingest.log
```
# remove detectormap in output directory
```
rm -r /work/drp/rerun/$RERUN/DETECTORMAP
```
# Construct pfsFiberProfiles
## brn-band ##

# b4
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=b spectrograph=4 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-b4.py --cores 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/fiberprofileb4.log 
# r4
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=r spectrograph=4 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-r4.py --cores 16 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/fiberprofiler4.log
# b2
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=b spectrograph=2 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-b2.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofileb2.log
# r2
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=r spectrograph=2 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-r2.py --cores 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/fiberprofiler2.log
# n2
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=n spectrograph=2 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlatNir=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-n2.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofilen2.log
# b1
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=b spectrograph=1 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-b1.py --cores 16 --clobber-config --longlog 1 2>&1 | tee -a $LOG/fiberprofileb1.log
# r1 
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=r spectrograph=1 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-r1.py --cores 16 --clobber-config --longlog 1 2>&1 | tee -a $LOG/fiberprofiler1.log
# n1
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=n spectrograph=1 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlatNir=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-n1.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofilen1.log
# b3
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=b spectrograph=3 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-b3.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofileb3.log
# r3
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=r spectrograph=3 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-r3.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofiler3.log
# n3
constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104144..104148 arm=n spectrograph=3 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlatNir=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-n3.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofilen3.log

## m-band ##

constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104176..104180 arm=m spectrograph=1 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-m1.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofilem1.log

constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104176..104180 arm=m spectrograph=2 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=2000 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-m2.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofilem2.log

constructFiberProfiles.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104176..104180 arm=m spectrograph=3 -c profiles.profileRadius=3 profiles.profileOversample=3 profiles.profileSwath=500 profiles.profileRejThresh=5 isr.doFlat=True doAdjustDetectorMap=True adjustDetectorMap.doSlitOffsets=True -C configs/profilesConfig-m3.py --cores 16 --clobber-config --longlog 1 --no-versions 2>&1 | tee -a $LOG/fiberprofilem3.log

ingestPfsCalibs.py /work/drp --output=$CALIB --validity=1800 --doraise --mode=copy --config clobber=True -- /work/drp/rerun/$RERUN/FIBERPROFILES/pfsFiberProfiles-*.fits 2>&1 | tee -a $LOG/ingest.log
rm -r /work/drp/rerun/$RERUN/FIBERPROFILES

# Construct detectorMap

# construct only detMaps of brn
# spectrograph1
reduceArc.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104149..104173 spectrograph=1  -c reduceExposure.isr.doFlat=True reduceExposure.isr.doFlatNir=True fitDetectorMap.doSlitOffsets=True -j 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/detectormap-brn1.log
# spectrograph2
reduceArc.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104149..104173 spectrograph=2 -c reduceExposure.isr.doFlat=True reduceExposure.isr.doFlatNir=True fitDetectorMap.doSlitOffsets=True -j 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/detectormap-brn2.log
# spectrograph3
reduceArc.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104149..104173 spectrograph=3 -c reduceExposure.isr.doFlat=True reduceExposure.isr.doFlatNir=True fitDetectorMap.doSlitOffsets=True -j 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/detectormap-brn3.log
# spectrograph4
reduceArc.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104149..104173 spectrograph=4 arm='b^r' -c reduceExposure.isr.doFlat=True fitDetectorMap.doSlitOffsets=True -j 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/detectormap-br4.log

# construct only detMaps of m
reduceArc.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104181..104203 spectrograph=1 -c reduceExposure.isr.doFlat=True fitDetectorMap.doSlitOffsets=True -j 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/detectormap-m1.log
reduceArc.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104181..104203 spectrograph=2 -c reduceExposure.isr.doFlat=True fitDetectorMap.doSlitOffsets=True -j 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/detectormap-m2.log
reduceArc.py /work/drp --calib=$CALIB --rerun=$RERUN --id visit=104181..104203 spectrograph=3 -c reduceExposure.isr.doFlat=True fitDetectorMap.doSlitOffsets=True -j 16 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/detectormap-m3.log

#remove bootstrap detectormap
sqlite3 $CALIB/calibRegistry.sqlite3 'delete from detectormap'
rm $CALIB/DETECTORMAP/pfsDetectorMap-*

ingestPfsCalibs.py /work/drp --output=$CALIB --validity=1800 --doraise --mode=copy --config clobber=True -- /work/drp/rerun/$RERUN/DETECTORMAP/pfsDetectorMap-*.fits 2>&1 | tee -a $LOG/ingest.log

# reduce exposures
mkdir -p $CALIB/test_rerun
mkdir -p $LOG/extractionQA
mkdir -p $LOG/DetectorMapQA

RERUN=USERNAME/test_calib_05mar24/test_rerun

# SM1(brn)
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104160 spectrograph=1 arm=b --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104149 spectrograph=1 arm=r --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104149 spectrograph=1 arm=n --config isr.doFlatNir=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=1 arm=b --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=1 arm=r --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=1 arm=n --config isr.doFlatNir=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log


# SM1(m)
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104181 spectrograph=1 arm=m --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104176 spectrograph=1 arm=m --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

# SM3(brn)
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104160 spectrograph=3 arm=b --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104149 spectrograph=3 arm=r --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104149 spectrograph=3 arm=n --config isr.doFlatNir=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=3 arm=b --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=3 arm=r --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=3 arm=n --config isr.doFlatNir=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

# SM3(m)
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104181 spectrograph=3 arm=m --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104176 spectrograph=3 arm=m --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

# SM2(brn)
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104160 spectrograph=2 arm=b --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104149 spectrograph=2 arm=r --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104149 spectrograph=2 arm=n --config isr.doFlatNir=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=2 arm=b --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=2 arm=r --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=2 arm=n --config isr.doFlatNir=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

# SM2(m)
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104181 spectrograph=2 arm=m --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104176 spectrograph=2 arm=m --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

# SM4(br)
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104149 spectrograph=4 arm='b^r' --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log
reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104160 spectrograph=4 arm=b --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log

reduceExposure.py /work/drp --calib=$CALIB --rerun=$RERUN --longlog=1 -j 16 --longlog 1 --id visit=104144 spectrograph=4 arm='b^r' --config isr.doFlat=True adjustDetectorMap.doSlitOffsets=False --clobber-config --no-versions 2>&1 | tee -a $LOG/extractionQA/reduceExposure.log




# extraction QA
setup -jr /work/wtg/obs_pfs

# SM1 (brn)
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104149 arm='r^n' spectrograph=1 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn1.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104160 arm=b spectrograph=1 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn1.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104144 arm='b^r^n' spectrograph=1 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn1-quartz.log

# SM3 (brn)
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104149 arm='r^n' spectrograph=3 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn3.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104160 arm=b spectrograph=3 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn3.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104144 arm='b^r^n' spectrograph=3 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn3-quartz.log

# SM2 (brn)
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104149 arm='r^n' spectrograph=2 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn2.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104160 arm=b spectrograph=2 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn2.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104144 arm='b^r^n' spectrograph=2 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-brn2-quartz.log

# SM4 (br)
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104149 arm=r spectrograph=4 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-br4.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104160 arm=b spectrograph=4 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-br4.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104144 arm='b^r' spectrograph=4 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-br4-quartz.log

# arm m
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104181 arm=m spectrograph=1 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-m1.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104181 arm=m spectrograph=2 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-m2.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104181 arm=m spectrograph=3 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-m3.log

extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104176 arm=m spectrograph=1 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-m1-quartz.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104176 arm=m spectrograph=2 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-m2-quartz.log
extractionQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104176 arm=m spectrograph=3 --clobber-config --no-versions --longlog 1 2>&1 | tee -a $LOG/extractionQA/extractionQA-m3-quartz.log


# DetectorMap QA
# SM1 (brn)
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104149 arm='r^n' spectrograph=1 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn1.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104160 arm=b spectrograph=1 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn1.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104144 arm='b^r^n' spectrograph=1 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn1-quartz.log
# SM3 (brn)
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104149 arm='r^n' spectrograph=3 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn3.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104160 arm=b spectrograph=3 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn3.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104144 arm='b^r^n' spectrograph=3 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn3-quartz.log
# SM2 (brn)
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104149 arm='r^n' spectrograph=2 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn2.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104160 arm=b spectrograph=2 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn2.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104144 arm='b^r^n' spectrograph=2 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-brn2-quartz.log
# SM4 (br)
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104149 arm=r spectrograph=4 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-br4.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104160 arm=b spectrograph=4 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-br4.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104144 arm='b^r' spectrograph=4 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-br4-quartz.log
# arm m
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104181 arm=m spectrograph=1 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-m1.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104181 arm=m spectrograph=2 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-m2.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104181 arm=m spectrograph=3 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-m3.log

detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104176 arm=m spectrograph=1 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-m1-quartz.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104176 arm=m spectrograph=2 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-m2-quartz.log
detectorMapQa.py /work/drp --calib $CALIB --rerun $RERUN --id visit=104176 arm=m spectrograph=3 --no-versions --clobber-config --longlog 1 2>&1 | tee -a $LOG/DetectorMapQA/DMQA-m3-quartz.log

