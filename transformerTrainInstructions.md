# Training Transformer_ee for Nova 

This guide is for training transformer_ee with data from nova.
It assumes that you already have the `.xz` dataset that you want to train on. If you don't already have that, instructions on generating the dataset can be found on this [github page.](https://github.com/OrghoN/magMomentAnaNotes/blob/main/transformerDataExtractorInstructions.md)
The Example used in this guide will be for the near detector forward horn current dataset  and will assume that the locations of the files generated are in the locations mentioned on the [data extraction guide.](https://github.com/OrghoN/magMomentAnaNotes/blob/main/transformerDataExtractorInstructions.md)
Furthermore, this guide is written assuming you will be working on the [elastic analysis facility (EAF).](https://eafdocs.fnal.gov)
Everyone with a fermilab computing account has access to EAF by default and this can be accessed via the browser as described in the [EAF documentation.](https://eafdocs.fnal.gov)
For the purposes of training, a FIFE server with GPU access is reccomended.
Training can of course be done on any system that has appropriate memory and GPU but if you are running on a different machine than an EAF FiFe server, instructions may need to be modified.

## Quickstart

Assuming you are running on eaf fife serer and have the data files in the default locations the following block of code should get things up and running.

```bash
cd /exp/nova/app/users/$USER/transformer
git clone git@github.com:OrghoN/transformer_ee.git
cd transformer_ee
git remote add upstream git@github.com:wswxyq/transformer_ee.git
conda create --name transformer_ee
conda init bash
source ~/.bashrc
conda activate transformer_ee
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
conda install scipy pandas polars numpy matplotlib
cp ../nd_fhc_data.csv.xz ./transformer_ee/data/nova/magMomentAna/nd_fhc_data.csv.xz
python3 train_script_nova.py
```

### ssh vs web access

I am using EAF via ssh whereas you probably will be using it over the web browser.
All commands can be typed into the terminal of the EAF jupyter hub server.
Because I am accessing it via ssh, I need to generate a token for it and as a result, the `$USER` variable is replaced by the token number rather than my username.
This is why I need to change the `$USER` variable to point to my username manually using

```bash
export USER=oneogi
```

You should not need to do this step since the `$USER` variable will be pointing to your username by default.

### Cloning the repository

The Original transformer_ee repository  can be found [here.](https://github.com/wswxyq/transformer_EE)
For the purposes of this guide, I will be using [my fork of transformer_ee.](https://github.com/OrghoN/transformer_ee)

This is because the default example code probided in the original is meant for training over DUNE data, whereas my fork is for training over Nova data.

The first step is to navigate to the directory we will be working in.

```bash
cd /exp/nova/app/users/$USER/transformer
```

Next we clone the repository using

```bash
# ssh path
git clone git@github.com:OrghoN/transformer_ee.git
# https path
git clone https://github.com/OrghoN/transformer_ee
```

The ssh path is recommended due to both security and convenience reasons.
This requires ssh keys to be setup with github and a tutorial on this setup can be found [here.](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
Tutorial on forwarding ssh-keys can be found [here.](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/using-ssh-agent-forwarding)

The next step is to add the upstream url that points to the original software repository

```bash
cd transformer_ee
# ssh path
git remote add upstream git@github.com:wswxyq/transformer_ee.git
# https path
git remote add upstream https://github.com/wswxyq/transformer_EE
```

Adding the upstream url allows for syncing between the fork and the upstream repository.
More instructions about that can be found [here.](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/configuring-a-remote-repository-for-a-fork)

### Configuring Conda

There are more detailed instructions on the conda configuration at the [readme for the repository](https://github.com/OrghoN/transformer_ee) so I will just list the instructions without going into great detail about what they mean.

```bash
conda create --name transformer_ee
conda init bash
source ~/.bashrc
conda activate transformer_ee
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
conda install scipy pandas polars numpy matplotlib
```

The `conda init` instructions are necessary for EAF since conda isn't initialized by default.
Once it has been initialized, the `.bashrc` or equivalent has to be sourced for it to kick in correctly.

The creation of the environment or installation of the packages doesn't need to be done every time, but the environment does need to be activated every time you want to work on the repository.

### Copying data

Once the conda environment has been setup, the training data needs to be copied to the `transformer_ee/data/nova` within the relevant subdirectory based on the config you are using.
for our example, we can copy the ND FHC data using

```bash
cp ../nd_fhc_data.csv.xz ./transformer_ee/data/nova/magMomentAna/nd_fhc_data.csv.xz
```

### Running training script

We are now ready to run the training script.

This can be done with
```bash
python3 train_script_nova.py
```

