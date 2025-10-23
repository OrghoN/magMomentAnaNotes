# Steps on running magnetic moment analysis

This document outlines the steps to run the magnetic moment analysis.
This assumes that you are working in the novagpvm computers and have already been added to the novaExperiment team on github..

This document does not explain the analysis.

## Quickstart

The following commands typed into the terminal should get things up and running, assuming you are okay with the defaults, the following code block should be run once ssh'd into the novagpvm computers.

```bash
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
spack load nova-grid-utils
setup_fnal_security
sl7-nova
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
mkdir /exp/nova/app/users/$USER
cd /exp/nova/app/users/$USER
git lfs install --skip-repo
newrel -t development magneticMoment
cd magneticMoment
srt_setup -a 
addpkg_git -h -t NuMagMomentAna
novasoft_build -t
cp -r /exp/nova/app/users/oneogi/magmomentana_files/* .
srt_setup -a 
novasoft_build -t
mkdir plotsNormalized
source cafe_run.sh
```

assuming everything works, there should be some plots created in 
`/exp/nova/app/users/$USER/magneticMoment/plotsNormalized`

In order to run over the entire dataset over the grid, after the tests above have been succesfully conducted, the following steps can be taken.

```bash
testrel_tarball . ../magneticMoment
cd ../
mkdir -p /pnfs/nova/scratch/users/$USER/spectra_event_selection
chmod g+w /pnfs/nova/scratch/users/$USER/spectra_event_selection
cd /pnfs/nova/scratch/users/$USER
cp /exp/nova/app/users/$USER/magneticMoment/NuMagMomentAna/NuMMAnalysis/EventSelection/make_spectra_event_selection.C .
cp /exp/nova/app/users/$USER/magneticMoment/NuMagMomentAna/NuMMAnalysis/master_header.h .
cp -r /exp/nova/app/users/oneogi/make_spectra_headers .
cp -r /exp/nova/app/users/oneogi/prediction_headers .
cd /exp/nova/app/users/$USER
exit
```

To actually submit the job, the following code can be run.
It is split into multiple lines for visibility but the entire block should be copied at once unlike the last block where each command is to be run one at a time.

 ```bash
submit_cafana.py -n 250 --print_jobsub \
--rel development -o /pnfs/nova/scratch/users/$USER/spectra_event_selection \
--user_tarball ./magneticMoment.tar.bz2 \
-i /pnfs/nova/scratch/users/$USER/make_spectra_headers/constants.h -i /pnfs/nova/scratch/users/$USER/make_spectra_headers/NuMMCuts.h -i /pnfs/nova/scratch/users/$USER/make_spectra_headers/NuoneCuts.h -i /pnfs/nova/scratch/users/$USER/make_spectra_headers/NuoneHistAxis.h -i /pnfs/nova/scratch/users/$USER/make_spectra_headers/NuoneVars.h -i /pnfs/nova/scratch/users/$USER/make_spectra_headers/NuoneWeights.h \
-i /pnfs/nova/scratch/users/$USER/prediction_headers/LDMSysts.h -i /pnfs/nova/scratch/users/$USER/prediction_headers/NDPredictionSingleElectron.h -i /pnfs/nova/scratch/users/$USER/prediction_headers/OscCalcSingleElectron.h \
-i /pnfs/nova/scratch/users/$USER/master_header.h \
-ss --ndid --numubarccinc \
--lib NuMagMomentAnaCuts --lib NuMagMomentAnaVars --lib NuMagMomentAnaPrediction --lib NuMagMomentAnaSysts --lib NuMagMomentAnaOscCalc \
/pnfs/nova/scratch/users/$USER/make_spectra_event_selection.C
```

The more detailed guide that follows explains what each of the above lines do.

### Ssh into novagpvm

To access novagpvm's  one can ssh into any of them but I'm using 02 as an example.

```bash
kinit -f <kerberos principal>@fnal.gov
```

Where  <kerberos principal> is to be replaced by your kerberos username.
This will get the kerberos tickets.

```bash
ssh -k <kerberos principal>@novagpvm02.fnal.gov
```

Where  <kerberos principal> is to be replaced by your kerberos username.
Now you should be in the novagpvm system.

The following commands are to be run in novagpvm computers after sshing in unless otherwise stated.

### Setup novasoft
First step is to setup novasoft. 
This instruction needs to be run every time upon login.
It can also be setup to run automatically upon login by putting it in the `.bashrc`, `.zshrc` or something of the like.

```bash
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
```

When novasoft was designed, it was supposed to run on scientific linux 7.
Now, all the novagpvm's are running alma 9 so we will use a sl7 container to run the code.

To have access to the sl7 container, we need to load nova grid utils  with 

```bash
spack load nova-grid-utils
```

With the utilities loaded, the container can be started by running

```bash
sl7-nova
```

Once in the container, novasoft has to be setup again using

```bash
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
```

### Setup Working Directory

The conventional place for the working directory is `/exp/nova/app/users/$USER`.
If your working directory is not already created, you can create it using 

```bash
mkdir /exp/nova/app/users/$USER
```

You can then `cd` into it using 

```bash
cd /exp/nova/app/users/$USER
```

If you're using emacs, the tramp ssh path for the working directory is

```bash
/ssh:<kerberos principal>@novagpvm02.fnal.gov:/exp/nova/app/users/<kerberos principal>
```

Where <kerberos principal> is to be replaced by your username for novagpvms.
The emacs path command above should be run in your local emacs instance and not on the nova gpvm machine.

### Setup git lfs

Git lfs needs to be setup.
This can ve done with 

```bash
git lfs install --skip-repo
```

This initializes Git LFS in your global Git configuration, enabling LFS features across all your repositories while not affecting the configuration of your current repository.

The above command needs to be only run once, no need to run it upon starting every session.

### Setting up new release

The next step is to set up a new test release for development purposes

```bash
newrel -t development magneticMoment
```

With the new release set up, we can cd into it using 

```bash
cd magneticMoment
```

Then we can run setup using

```bash
srt_setup -a 
```

### Adding NuMagMomentAna

We then need to add the analysis code from github by adding the package NuMagMomentAna using

```bash 
addpkg_git -h NuMagMomentAna
```

This sets up all the linking correctly and  can be built using

```bash 
novasoft_build -t
```

The files that are added from github are out of date, more up to date files may be found at `/exp/nova/app/users/oneogi/magmomentana_files/` and copied by

```bash
cp -r /exp/nova/app/users/oneogi/magmomentana_files/* .
```

With the more updated files copied in, we need to build it again using

```bash 
novasoft_build -t
```

The build needs to be repeated every time there is a change in the code.

### Running the analysis code 

There are two files that run out of the box, located in `/exp/nova/app/users/$USER/magneticMoment/NuMagMomentAna/NuMMAnalysis/EventSelection` if all the instructions thus far have been followed including using suggested file paths. 

- *make_spectra_event_selection.C* : This produces a root file with histograms of all the signal/background information
- *plot_spectra_event_selection.C* : puts the histogram info from each cut together on a plot

Before these files can be run, a directory has to be created to store the plots and this can be done using

```bash
mkdir plotsNormalized
```

Once the directory is created, the files can be run using 

```bash
source cafe_run.sh
```

### Getting job logs

```bash
jobsub_fetchlog --jobid=87799120.0@jobsub02.fnal.gov --unzipdir=./jobLogs
```

```bash
jobsub_q --user=oneogi
```
