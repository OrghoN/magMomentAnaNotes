# Training Transformer_ee for Nova 

This guide is for training transformer_ee with data from nova.
It assumes that you already have the `.xz` dataset that you want to train on. If you don't already have that, instructions on generating the dataset can be found on this [github page.](https://github.com/OrghoN/magMomentAnaNotes/blob/main/transformerDataExtractorInstructions.md)
The Example used in this guide will be for the near detector forward horn current dataset  and will assume that the locations of the files generated are in the locations mentioned on the [data extraction guide.](https://github.com/OrghoN/magMomentAnaNotes/blob/main/transformerDataExtractorInstructions.md)
Furthermore, this guide is written assuming you will be working on the [elastic analysis facility (EAF).](https://eafdocs.fnal.gov)
Everyone with a fermilab computing account has access to EAF by default and this can be accessed via the browser as described in the [EAF documentation.](https://eafdocs.fnal.gov)
For the purposes of training, a FIFE server with GPU access is reccomended.
Training can of course be done on any system that has appropriate memory and GPU but if you are running on a different machine than an EAF FiFe server, instructions may need to be modified.

## Quickstart

### ssh vs web access

I am using EAF via ssh whereas you probably will be using it over the web browser.
All commands can be typed into the terminal of the EAF jupyter hub server.
Because I am accessing it via ssh, I need to generate a token for it and as a result, the `$USER` variable is replaced by the token number rather than my username.
This is why I need to change the `$USER` variable to point to my username manually using

```bash
export USER=oneogi
```

You should not need to do this step since the `$USER` variable will be pointing to your username by default.

