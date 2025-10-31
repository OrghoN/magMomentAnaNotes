# Transformer_ee Data Extraction for Nova

Transformer_ee is a machine learning model that is used to reconstruct neutrino events.
It can be used for many different experiments and provides a generic framework  for training based on the data you care about.
More information on how to do this as well as the code can be found on the [github page for trasformer_ee](https://github.com/wswxyq/transformer_EE/tree/main).

Because it can accept data from various experiments, the data has to be extracted and prepared for the transformer_ee.
This document provides step by step instructions on how to do this with Nova data.
The example provided in this document is for  Mini Production 6.1 OPAL sample; specifically for near detector with forward horn current but the general principles can be adapted to other nova data by adjusting the macros shown and the  cuts applied.
This assumes that you are working in the novagpvm computers and have already been added to the novaExperiment team on github..

## Quickstart

The quickstart part of this guide assumes that you have setup novasoft before.
If you have never setup novasoft before, I recommend looking at this [beginner wiki page.](https://cdcvs.fnal.gov/redmine/projects/novaart/wiki/Documentation_FOR_BEGINNERS)
It has helpful resources, gives a overview of how to work with novasoft and contains the one time setup instructions that need to be run before following the instructions in this section of the document.

The following commands typed into the terminal should get things up and running, assuming you are okay with the defaults, the following code block should be run once ssh'd into the novagpvm computers.
This will get things setup to be able to submit extraction of the near detector forward horn current dataset.

```bash
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
spack load nova-grid-utils
setup_fnal_security
cd /exp/nova/app/users/$USER
mkdir transformer
cd transformer
sl7-nova
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
newrel -t development transformerEE_data_extract
cd transformerEE_data_extract
git checkout feature/wus_transformerEE_data_extract
cp -r /exp/nova/app/users/oneogi/transformeree_data_script/ .
srt_setup -a
novasoft_build -t
```

The ND-FHC macro can then be run interactively over 5 files for testing by using

```bash
cafe -bq -l 5 ./transformeree_data_script/mprod6.1_OPAL/mprod6_exporter_transformer_ee_nd_fhc_nonswap.C
```

However, to run over the entire dataset, it is recommended to run it over the grid. The following code block shows the setup instructions for that.

```bash
testrel_tarball . ../transformerEE_data_extract
cd ../
cp /exp/nova/app/users/oneogi/transformeree_data_script/mprod6.1_OPAL/mprod6_exporter_transformer_ee_nd_fhc_nonswap.C /pnfs/nova/scratch/users/$USER/mprod6_exporter_transformer_ee_nd_fhc_nonswap.C
mkdir -p /pnfs/nova/scratch/users/$USER/transformer/nd_fhc_data
chmod g+w /pnfs/nova/scratch/users/$USER/transformer/nd_fhc_data
exit
```

To actually submit the job, the following code can be run.
It is split into multiple lines for visibility but the entire block should be copied at once unlike the last few blocks where each command is to be run one at a time.

 ```bash
submit_cafana.py -n 250 --print_jobsub \
--rel development -o /pnfs/nova/scratch/users/$USER/transformer/nd_fhc_data \
--user_tarball ./transformerEE_data_extract.tar.bz2 \
/pnfs/nova/scratch/users/$USER/mprod6_exporter_transformer_ee_nd_fhc_nonswap.C
```

Once the jobs finish, you can merge the output csv files by the following commands

```bash
cd /exp/nova/app/users/$USER/transformer
cp -r /pnfs/nova/scratch/users/$USER/transformer/nd_fhc_data .
cp /exp/nova/app/users/oneogi/transformeree_data_script/mprod6.1_OPAL/merge_csv.sh .
chmod +x merge_csv.sh
./merge_csv.sh nd_fhc_data.csv.xz nd_fhc_data/dataset_*.csv
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
The following  instruction needs to be run every time upon login.
It can also be setup to run automatically upon login by putting it in the `.bashrc`, `.zshrc` or something of the like.

```bash
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
```

### Setting up grid access

We will eventually use the grid and therefore need to load the utility functions for the grid which can be done with 

```bash
spack load nova-grid-utils
```

With the grid utilities loaded, a token needs to be setup based on  your kerberos ticket for authentication to the grid.
use `klist` to check if you have a valid ticket and if not, run `kinit -f <kerberos principal>@fnal.gov` as described before.
If a valid kerberos ticket exists, a token can be generated with

```bash
setup_fnal_security
```

This token does expire, so if you are having authentication issues when submitting to the grid or using jobsub in general, it is a good idea to run the above command again to refresh the token.

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

### Accessing sl7 nova container

When novasoft was designed, it was supposed to run on scientific linux.
Now, all the novagpvm's are running alma 9 so we will use a sl7 container to run the code.
the container can be started by running

```bash
sl7-nova
```

Once in the container, novasoft has to be setup again using

```bash
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
```

### Setting up new release

I would strongly recommend creating a new directory for organizing the code discussed in this document and then `cd` into it.
This will keep all the code in a subdirectory instead of pollutin your main working directory.
This can be done using the following commands assuming you want to call this directory transformer but the actual name doesn't matter, feel free to call it something else.

```bash
mkdir transformer
cd transformer
```

In the future, the code for extraction will be merged into the main branch  but for now we still need to set up a custom test release.


```bash
newrel -t development transformerEE_data_extract
```

With the new release set up, we can cd into it using 

```bash
cd transformerEE_data_extract
```

### Switching to the correct branch

The main branch of transformer_ee does not have the code required for extraction so we have to switch to the `feature/wus_transformerEE_data_extract` branch. This can be done using

```bash
git checkout feature/wus_transformerEE_data_extract
```

### Building the test release

First, we can run setup using 

```bash
srt_setup -a 
```

Now, we are ready to build our test release.
This can be done using

```bash 
novasoft_build -t
```

Every time the code is changed, it needs to be rebuilt using the above command.

### Tarring up the test release

In order to use our custom test release on the grid,   we need to turn our release into a tarball.

the general syntax for doing this is as follows

```bash
tar czf <name_of_archive_file>.tar.gz <name_of_directory_to_tar>

```

For our example, you can run

```bash
cd ../
tar czf transformerEE_data_extract.tar.gz ./transformerEE_data_extract
```

### Location of Macros

A full suite of macros for extracting data for both the near and far detector  can be found at

```bash
/pnfs/nova/persistent/users/wus/transformeree_data_script/mprod6.1_OPAL
```

For this example, we will use the macro for the near detector and forward horn current.
This macro should be copied to the local directory using

```bash
cp /pnfs/nova/persistent/users/wus/transformeree_data_script/mprod6.1_OPAL/mprod6_exporter_transformer_ee_nd_fhc_nonswap.C .
```

### Making an output directory

The output from the grid has to be dumped somewhere and so we need to create a directory to hold our outputs. For this example, this can be done with

```bash
mkdir nd_fhc_data
```

we have to make sure the output directory is group writeable. This can be done using chmod as follows

```bash
chmod g+w ./nd_fhc_data/
```

### Submitting to the grid

Submitting to the grid has to be done outside the singularity container we have been working in so far.
We can usually exit the container with `Ctrl-d` or by typing the following command

```bash
exit
```

If the security token is expired, it can be refreshed with 

```bash
setup_fnal_security
```

The general syntax  for running these macros over the grid is

```bash
submit_cafana.py -n <NUMBER_OF_JOBS> --print_jobsub \
--rel development -o <OUT_DIR> \
--user_tarball <PATH_TO_TARBALL> \
<MACRO_PATH>
```

Where

- `<NUMBER_OF_JOBS>` is to  be replaced by a integer that specifies how many jobs to run
- `<OUT_DIR>` is to be replaced by location of output directory
- `<PATH_TO_TARBALL>` is to be replaced by path to tarball of test release
- `<MACRO_PATH>` is to be replaced by the path to the macro you wish to run

For the purposes of our example, this can be accomplished by

```bash
submit_cafana.py -n 250 --print_jobsub \
--rel development -o nd_fhc_data \
--user_tarball transformerEE_data_extract.tar.gz \
mprod6_exporter_transformer_ee_nd_fhc_nonswap.C
```

### Getting job logs

```bash
jobsub_fetchlog --jobid=87925287.0@jobsub02.fnal.gov --unzipdir=./jobLogs
```

```bash
jobsub_q --user=oneogi
```




