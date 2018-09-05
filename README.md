# StopCfg

## Update Cfg Files

1. First create and copy new text files listing root files to EOS.

- Go to SusyAnaTools condor area.

```cd $CMSSW_BASE/src/SusyAnaTools/Tools/condor```

- Run batchList.py (with -l -c and -d, and with path to ntuples).

python batchList.py -lc -d /eos/uscms/store/user/lpcsusyhad/Stop_production/CMSSW8028_2016


2. Second replace file paths in sample text files

- Example

old path: /eos/uscms/store/user/lpcsusyhad/Stop_production/SoftBjet_PhotonNtuples/
new path: /eos/uscms/store/user/lpcsusyhad/Stop_production/CMSSW8028_2016/

replace SoftBjet_PhotonNtuples with CMSSW8028_2016
```sed -i -e 's/SoftBjet_PhotonNtuples/CMSSW8028_2016/g' sampleSets.cfg```

3. Third compile nEvents.C and run nEvts with the output stored in a file.
```
make -j8
./nEvts -w > nEvents_v3.txt 
```

4. Fourth run this script with options.
```python updateSamples.py```

