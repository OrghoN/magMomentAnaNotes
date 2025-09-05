# Transformer_ee Data Extraction for Nova

Transformer_ee is a machine learning model that is used to reconstruct neutrino events.
It can be used for many different experiments and provides a generic framework  for training based on the data you care about.
More information on how to do this as well as the code can be found on the [github page for trasformer_ee](https://github.com/wswxyq/transformer_EE/tree/main).

Because it can accept data from various experiments, the data has to be extracted and prepared for the transformer_ee.
This document provides step by step instructions on how to do this with Nova data.
The example provided in this document is for  Mini Production 6.1 OPAL sample; specifically for near detector with forward horn current but the general principles can be adapted to other nova data by adjusting the macros shown and the  cuts applied.
This assumes that you are working in the novagpvm computers and have already been added to the novaExperiment team on github..

## Quickstart

The following commands typed into the terminal should get things up and running, assuming you are okay with the defaults, the following code block should be run once ssh'd into the novagpvm computers.
This will get things setup to be able to submit extraction of the near detector forward horn current dataset.

```bash
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
spack load nova-grid-utils
cd /exp/nova/app/users/$USER
mkdir transformer7
cd transformer7
sl7-nova
source /cvmfs/nova.opensciencegrid.org/novasoft/slf7/novasoft/setup/setup_nova.sh
newrel -t development transformerEE_data_extract
cd transformerEE_data_extract
git checkout feature/wus_transformerEE_data_extract
srt_setup -a
novasoft_build -t
cd ../
tar czf transformerEE_data_extract.tar.gz ./transformerEE_data_extract
cp /pnfs/nova/persistent/users/wus/transformeree_data_script/mprod6.1_OPAL/mprod6_exporter_transformer_ee_nd_fhc_nonswap.C .
mkdir -p /pnfs/nova/scratch/users/oneogi/transformer7/nd_fhc_data
chmod g+w /pnfs/nova/scratch/users/oneogi/transformer7/nd_fhc_data
exit
setup_fnal_security
```

To actually submit the job, the following code can be run.
It is split into multiple lines for visibility but the entire block should be copied at once unlike the last block where each command is to be run one at a time.

 ```bash
submit_cafana.py -n 250 --print_jobsub \
--rel development -o /pnfs/nova/scratch/users/oneogi/transformer7/nd_fhc_data \
--user_tarball ./transformerEE_data_extract.tar.gz \
./mprod6_exporter_transformer_ee_nd_fhc_nonswap.C
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
jobsub_fetchlog --jobid=79571052.0@jobsub01.fnal.gov --unzipdir=./jobLogs
```

```bash
jobsub_q --user=oneogi
```




