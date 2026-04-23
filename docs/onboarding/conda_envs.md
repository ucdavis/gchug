# 1. Miniforge, conda, and mamba

Fortunately, It is easy to download and start using anything you want with mamba/conda, which is a _package manager_. This just means it helps install the software and any required dependencies for your system, without too much extra effort on your end (usually).

To install mamba/conda, use [miniforge3](https://github.com/conda-forge/miniforge)

```sh
# download the right installer for your OS/arch
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"

# Make it executable and install (interactive mode to $HOME/miniforge3)
chmod +x Miniforge3-$(uname)-$(uname -m).sh
./Miniforge3-$(uname)-$(uname -m).sh
```

This will download the miniforge3 installer and unpack it into the `$HOME/miniforge3` directory.

The interactive installation will prompt you to initialize conda with your shell.

Now, if you log out and log back in, you should see "base" in your command line

```sh
(base) username@login2$
```

This means that conda is _active_ and you are in your _base_ environment. You should keep this base environment clean. Don't install new packages here. 

## 1.1 Environments

Imagine that you have a bunch of tools that you use to work on your car and also make repairs at home. As time goes on, it becomes confusing to keep all your tools in the same place, especially as your collection keeps growing. What size allen key does that bolt need? What size screwdriver do I need for that screw? Is it philips or flathead?

What if instead of having a jumbled pile of tools, you could have specific toolboxes that you use to store the tools that are relevant to the specific repair you are making. That way, you'll never confuse your socket wrenches or allen keys.

This is exactly what package managers like conda and mamba allow us to do. We can create 1 environment that contains the software necessary for aligning, and another environment that contains the software needed for making nice figures.

Since most software these days depend on modules (dependencies), it is best to install all the tools you'll think you need for a particular job all at the same time. Mamba/conda will deal with the dependency issues during installation and everything should play together fine. 

```sh
conda create -n minimap2 bioconda::minimap2 bioconda::samtools -y

# activate env
conda activate minimap2

# verify installation
minimap2 --version
# 2.30-r1287

# deactivate env
conda deactivate
```

Here, `conda create -n minimap2` creates a new environment called "minimap2", and `bioconda::minimap2 bioconda::samtools` tells conda to look for the minimap2 and samtools softwares from the bioconda channel (imagine there are 3 stores you can buy a hammer from. You want to specify that you want to buy a hammer from ace hardware instead of target).

## 1.2 A note on `mamba`

In previous versions, `mamba` was a faster way to create packages, as it used a more sophisticated solver. In modern miniforge3, `conda` uses the `libmamba` solver under the hood, so there are very minimal speed differences between the two. Nevertheless, in most instances, `conda` and `mamba` are equivalent and interchangeable. 

LINK TO BEST_PRACTICES FOR CONDA/MAMBA. LIKE CLONING, MAKING YAMLS, ETC.
