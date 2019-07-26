# StopCfg

## Checkout Version of Cfg Files

See instructions [here](https://github.com/susy2015/SusyAnaTools#check-out-stop-config-files).

## Update Cfg Files

If you have not already, first follow the instructions [here](https://github.com/susy2015/SusyAnaTools#instructions) to download and setup the TopTagger, TopTaggerTools and SusyAnaTools repositories. 

This will allow you to run the nEvts.C and updateSamples.py scripts.

Here is an example for updating sampleSets.cfg with new weights.
For this example, we are changing from the SoftBjet_PhotonNtuples samples to the CMSSW8028_2016 samples.
- old path: /eos/uscms/store/user/lpcsusyhad/Stop_production/SoftBjet_PhotonNtuples/
- new path: /eos/uscms/store/user/lpcsusyhad/Stop_production/CMSSW8028_2016/

### 1. Create and copy new text files listing root files to EOS.

- Go to SusyAnaTools/Tools/condor area.

```cd $CMSSW_BASE/src/SusyAnaTools/Tools/condor```

- Run batchList.py.

Please use the path beginning with "/store/user/lpcsusyhad" and not the path beginning with "/eos/uscms/store/user/lpcsusyhad".

First run batchList.py using -d (path to ntuples), -m (regular expression to math root file names) and -l (create text files in current directory).

```
python batchList.py -d /store/user/lpcsusyhad/Stop_production/CMSSW8028_2016/ -m ".*\.root" -l
```

Make sure that the text files were created in your current directory. Check that the text files contain the expected root files with the correct path. The file paths should not contain "/eos/uscms". If you do not see any root files listed in the text files, you may be using the wrong pattern. Use `-m ".*\.root"` to match all root files.

If the text files were produced and contain the correct root files and paths, the you can run the command again with the -c option to copy the files to eos.

```
python batchList.py -d /store/user/lpcsusyhad/Stop_production/CMSSW8028_2016/ -m ".*\.root" -lc
```

If the text files are sucessfully copied to eos, you can remove them in your current directory. Note that when running with default options, you will not be able to copy files to eos if they already exist. Use --force to overwrite files when doing xrdcp (use with caution).


### 2. Replace file paths in sample text files.

- Get sample files and copy them to SusyAnaTools (if you don't already have them).

```
cd $CMSSW_BASE/src
git clone git@github.com:susy2015/StopCfg.git
```

- Copy the config file to the SusyAnaTools/Tools area.

```
cp StopCfg/sampleSets.cfg $CMSSW_BASE/src/SusyAnaTools/Tools
cd $CMSSW_BASE/src/SusyAnaTools/Tools
```

- Replace the old path name with the new path name.

```sed -i -e 's|SoftBjet_PhotonNtuples|CMSSW8028_2016|g' sampleSets.cfg```

Note that we used "|" in the sed command instead of "/". This is useful when you have to replace a string that contains "/" such as "/oldpath/olddir".

```sed -i -e 's|/oldpath/olddir|/newpath/newdir|g' myfile.cfg```

Otherwise, if you use "/" instead of "|" and there are "/" in the pattern you are matching, you have to escape "/" with "\\" by using "\\/".

```sed -i -e 's/\/oldpath\/olddir/\/newpath\/newdir/g' myfile.cfg```

### 3. Calculate Number of Events (nEvents)

- Go to SusyAnaTools/Tools area.

```cd $CMSSW_BASE/src/SusyAnaTools/Tools```

- Compile.

```make -j8```

- Go to condor directory.

```cd condor```

- Run nEvtsCondorSubmit.py. Use -s to provide the name of the config file (e.g. sampleSets_PostProcessed_2016.cfg). You may use pre or post processed config files.

```python nEvtsCondorSubmit.py -s sampleSets_PostProcessed_2016.cfg```

This will make a submission directory with the date and time (e.g. submission_2019-04-29_11-59-34). Check your condor jobs with condor_q.

```condor_q```

Once all your condor jobs are done, you can process the output using processCondorOutput.sh.

```./processCondorOutput.sh -d submission_2019-04-29_11-59-34```

Now the submission directory (submission_2019-04-29_11-59-34) should contain an output directory (contains one text file per sample) and nEvents.txt (all samples together).

```
cmslpc26.fnal.gov condor # (NanoAOD) ls -lhtr submission_2019-04-29_11-59-34
total 352K
-rw-r--r-- 1 caleb us_cms 146K Apr 29 11:59 CMSSW_9_4_4.tar.gz
-rw-r--r-- 1 caleb us_cms  82K Apr 29 11:59 TT.tar.gz
-rw-r--r-- 1 caleb us_cms  27K Apr 29 11:59 condor_submit.txt
-rwxr-xr-x 1 caleb us_cms    0 Apr 29 12:00 docker_stderror
drwxr-xr-x 2 caleb us_cms 8.0K Apr 29 12:10 output
-rw-r--r-- 1 caleb us_cms  26K Apr 29 12:10 nEvents.txt
drwxr-xr-x 2 caleb us_cms  16K Apr 30 19:34 logs
```

You can pass nEvents.txt to updateSamples.py along with an input config file to producde an updated config file (see next section).

<details> <summary> Using screen to run nEvts.C (very slow, no longer used) </summary>

- The script nEvts will take a long time to run (hours). You should use screen. Here are some useful screen commands.
  - To enter screen, use ```screen```.
  - To create a named session, use ```screen -S session_name```.
  - To exit screen, use ```exit```.
  - To scroll in screen, hold down ```SHIFT``` and then scroll with your mouse or trackpad.
  - To list screens, use ```screen -ls```.
  - To attach screen, use ```screen -r```.
  - To attach a named session, use ```screen -r session_name```.
  - To detach screen, use ```CRTL-A D```.

- When in screen, you are entering a new shell and a new environment. You will need to run some commands to obtain the desired environment. Bash users can begin with `source ~/.bash_profile`.

For bash shell:
```
screen
source ~/.bash_profile
source /cvmfs/cms.cern.ch/cmsset_default.sh
cmsenv
source $CMSSW_BASE/src/TopTagger/TopTagger/test/taggerSetup.sh
```

For tcsh shell:
```
screen
source ~/.tcshrc
source /cvmfs/cms.cern.ch/cmsset_default.csh
cmsenv
source $CMSSW_BASE/src/TopTagger/TopTagger/test/taggerSetup.csh
```

Now run nEvts. The options are sSC. To remember this, think of SSC = Superconducting Super Collider (Desertron). Here -s is for "skip data samples" (not required) -S is for the name of the sample sets file (required), and -C is for the name of the sample collections file (required). 

We are redirecting stdout (1) to nEvents.txt and stderr (2) to nEvents_errors.log.


For bash shell:
```
time ./nEvts -s -S sampleSets_PostProcessed_2016.cfg -C sampleCollections_2016.cfg 1> nEvents.txt 2> nEvents_errors.log
```

For tcsh shell:
```
(time ./nEvts -s -S sampleSets_PostProcessed_2016.cfg -C sampleCollections_2016.cfg > nEvents.txt) >& nEvents_errors.log
```

You can also run over a specific sample set (such as DYJetsToLL_HT_100to200_2016) or sample collection (such as DYJetsToLL) by putting the sample name as the last argument.

For bash shell:
```
time ./nEvts -s -S sampleSets_PostProcessed_2016.cfg -C sampleCollections_2016.cfg DYJetsToLL 1> nEvents.txt 2> nEvents_errors.log
```

For tcsh shell:
```
(time ./nEvts -s -S sampleSets_PostProcessed_2016.cfg -C sampleCollections_2016.cfg DYJetsToLL > nEvents.txt) >& nEvents_errors.log
```

 </details>

### 4. Run updateSamples.py with options 

The script updateSamples.py will create a new config file with the n_events that you just calculated. It takes the old sampleSets.cfg config file and the nEvents.txt file as input. It outputs a new sampleSets_v1.cfg config file with the new weights. The options are -e for the text file output of nEvts, -i for input file (original cfg file), and -o for output file (new cfg file).
```
cd $CMSSW_BASE/src/SusyAnaTools/Tools/condor
time python updateSamples.py -i ../sampleSets.cfg -o ../sampleSets_v1.cfg -e nEvents.txt > update.log
```

To run over a specific sample set or collection, use the -d option.
```
cd $CMSSW_BASE/src/SusyAnaTools/Tools/condor
time python updateSamples.py -i ../sampleSets.cfg -o ../sampleSets_v1.cfg -e nEvents.txt -d DYJetsToLL > update.log
```

We redirected the output to update.log. Check update.log to see that each sample in sampleSets.cfg found exactly one match in nEvents.txt.

The script updateSamples.py can also directly calculate the number of events using nEvts.py and create the new config file. This mode is used by not providing the "-e nEvents.txt" option.
```
cd $CMSSW_BASE/src/SusyAnaTools/Tools/condor
time python updateSamples.py -i ../sampleSets.cfg -o ../sampleSets_v1.cfg > update.log
```

To run over a specific sample set or collection, use the -d option.
```
cd $CMSSW_BASE/src/SusyAnaTools/Tools/condor
time python updateSamples.py -i ../sampleSets.cfg -o ../sampleSets_v1.cfg -d DYJetsToLL > update.log
```

The file ../sampleSets_v1.cfg is the updated config file with the new weights.


### 5. Make a new release.
- Download the repo if you have not already.
```
git clone git@github.com:susy2015/StopCfg.git
cd StopCfg
```
- Make a new branch based on the master branch.
```
git checkout -b <new_branch_name>
```
- Copy your updated version of the sample config files here.
- Add and commit your changes.
```
git commit -am "explanation of changes"
```
- Create a token on GitHub by going to Settings, Developer Settings, and finally Personal access tokens. Check the public_repo box. Make sure to copy and store the token that you create somewhere safe. It is only displayed once.
- Source a script that sets the GITHUB_TOKEN to the token that you created and saved. 
```
source ~/.ssh/tokens
```
This script should do the following.
```
#!/bin/bash
export GITHUB_TOKEN=<git_token_created_on_github>
```

- Add github-release and makeStopRelease.sh to your path. You make softlink to my versions.
```
mkdir ~/bin
ln -s /uscms/home/caleb/bin/github-release ~/bin/github-release
ln -s /uscms/home/caleb/bin/makeStopRelease.sh ~/bin/makeStopRelease.sh
export PATH="$PATH:~/bin"
```

- Use the makeStopRelease.sh script to publish a new release.
```
makeStopRelease.sh -h

Usage:
    makeStopRelease.sh -t RELEASE_TAG -b BRANCH [-d SUPP_FILE_DIR] [-m MESSAGE]

Options:
    -t RELEASE_TAG :       This is the github release tag which will be created (Required)
    -b BRANCH :            Git branch to base release off. This branch must exist. It will be pushed to github. (Required)
    -d SUPP_FILE_DIR :     The folder where the supplemental training file can be found (default supplementaryFiles)
    -m MESSAGE :           Commit message for tag (Default empty)

Description:
    This script automatically makes a release of the Stop search config files
    Run this script from the StopCfg directory
    Source a script that does "export GITHUB_TOKEN=<git_token_created_on_github>" before running this script.
```
Here is an example. The branch you use for the release must exist. Make sure that the tag and branch names are different.
Git does not like when branch and tag names are the same because then the names are ambiguous. The tag and release will be created by the script.
```
makeStopRelease.sh -b CMSSW8028_2016 -t CMSSW8028_2016_v1.0.0 -m "CMSSW8028_2016 config files. Weights are updated. The supplementaryFiles are included."
```


