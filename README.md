# StopCfg

## Update Cfg Files

Here is an example for updating sampleSets.cfg with new weights.
For this example, we are changing from the SoftBjet_PhotonNtuples samples to the CMSSW8028_2016 samples.
- old path: /eos/uscms/store/user/lpcsusyhad/Stop_production/SoftBjet_PhotonNtuples/
- new path: /eos/uscms/store/user/lpcsusyhad/Stop_production/CMSSW8028_2016/

1. First create and copy new text files listing root files to EOS.

- Go to SusyAnaTools/Tools/condor area.

```cd $CMSSW_BASE/src/SusyAnaTools/Tools/condor```

- Run batchList.py (with -l for list, -c for copy to eos, and -d for the path to ntuples).

```python batchList.py -lc -d /eos/uscms/store/user/lpcsusyhad/Stop_production/CMSSW8028_2016```


2. Second replace file paths in sample text files.

- Go to SusyAnaTools/Tools area.

```cd $CMSSW_BASE/src/SusyAnaTools/Tools```

- replace SoftBjet_PhotonNtuples with CMSSW8028_2016

```sed -i -e 's/SoftBjet_PhotonNtuples/CMSSW8028_2016/g' sampleSets.cfg```

3. Third compile SusyAnaTools and run nEvts with the output stored in a file.

- Go to SusyAnaTools/Tools area.

```cd $CMSSW_BASE/src/SusyAnaTools/Tools```

- Compile.

```make -j8```

- The script nEvts will take a long time to run (hours). You should use screen. Here are some useful screen commands.
  - To enter screen, use ```screen```.
  - To exit screen, use ```exit```.
  - To attach screen, use ```screen -r```.
  - To detach screen, use ```CRTL-A D```.
  - To list screens, use ```screen -ls```.

- When in screen, you are entering a new shell and a new environment. You will need to run some commands to obtain the desired environment. Bash users can begin with `source ~/.bash_profile`.

```
screen
source ~/.bash_profile
source /cvmfs/cms.cern.ch/cmsset_default.sh
cmsenv
source $CMSSW_BASE/src/TopTagger/TopTagger/test/taggerSetup.sh
./nEvts -ws 1> nEvents.txt 2> nEvents_errors.log
```

When we run nEvts, we are redirecting stdout (1) to nEvents.txt and stderr (2) to nEvents_errors.log.

4. Fourth run updateSamples.py with options (-e for output of nEvts, s for original cfg file, and -o for new cfg file).
```
python updateSamples.py -e nEvents.txt -s sampleSets.cfg -o sampleSets_v2.cfg > update.log
```

We redirect the output to update.log. Check update.log to see that each sample in sampleSets.cfg found exactly one match in nEvents.txt. The updateSamples.py script will produce sampleSets_v1.cfg (a copy of original sampleSets.cfg) and sampleSets_v2.cfg (the updated version of sampleSets.cfg).

