---
title: "Installing rTREES in a virtual environment using conda"
date: 2023-05-10T11:12:15-06:00
draft: false
tags:
    - Tutorials
---

This tutorial is designed to help you run the TREES model in a "virtual environment," a quarantine chamber for software, so that it doesn't interfere with/get affected by your other R projects.

In general though, it's a good idea to manage all of your software projects using virtual environments. It prevents clutter from building up, and makes it less likely that packages will interfere with each other.
It also makes it easier for you to share your work with others, by giving you a portable/sharable environment that other people can directly use to run your code.

This tutorial uses anaconda to manage your environments, but there are other ways of doing this as well. I'm most familiar with anaconda, and I think it's the most common environment manager out there.
If you get through this tutorial and you hated every second of it, you can check out the `renv` virtual environment manager exlusively for R. I've never used it, but it's worth looking into maybe: https://posit.co/blog/renv-project-environments-for-r/.

If you're curious about how to use anaconda for other purposes, check out my Jupyter tutorial as well: [Working in Python with Jupyter and Anaconda]({{< ref "jupyter-anaconda" >}})

# R, rTREES, and RStudio

This tutorial will show you how to

1. install R in a virtual environment
2. Install the rTREES package on R in that environment
3. Interact with the version/environment using RStudio or the command line.

# Anaconda environments
Anaconda is a program to help manage virtual environments, which are like little quarantine chambers for each of your code projects.

A virtual environment is a collection of software and packages that is segregated from other virtual environments to prevent conflicts between them.

Say that you're working on project A that relies on python version 2.7 (This is probably what your computer came with if you have a mac), but you have another project (B) that uses PyMC (a Bayesian modeling package), which needs python version 3.6. 
You can't update python to version 3.6, because that would break everything in your first project! 
Instead, you put each project into its own virtual environment so that they can't interfere. 
Any time you want to work on project A, you can enter the environment you set up for it, and likewise for project B. 
There's a little more to keep track of now, since you need to know to activate environment A to work on project A, but it easier to troubleshoot problems, make changes to your projects, and share your work.


# Installing and Setting Up Anaconda

## Install Anaconda
You can install anaconda from here: https://www.anaconda.com/. Download the graphical installer and follow the instructions.

This will install both the command line interface and the "anaconda navigator" GUI. I'm more familiar with the command line interface (which is also slightly faster), but the GUI provides (almost) all the same functionality. I won't explain the GUI here, but you should explore it on your own if you want. 
Below, I'll explain how to interact with the command line interface: how to create environments, install R, and install certain packages.

## Start Anaconda
Once anaconda has been installed, open up a terminal and type `conda --version`. 
This hopefully should show that you have the latest version (22.11.1 or 23.1.0 or something? not sure)

You should also see, next to your username, the word "base": mine says `(base) alex@Alexs-MacBook-Pro ~ %`.

This indicates that conda is running, and that you're in the default (base) environment.

If you don't see this, then anaconda isn't running. You can start anaconda by typing `conda` in the terminal.

If any of this gives you an error, then anaconda and its environments might not have been added to your path. I'd recommend asking Alex for help if you encounter this.

### For a technical aside about how anaconda actually does this, scroll to the bottom of the page.

## Create a new environment
We're going to create a conda environment using R version 4.2.3. I'm sure other versions will also work, but that's the version my computer uses, and that version definitely works with rTREES.

create a new virtual environment with the command`conda create --name choose_your_favorite_name r-base=4.2.3`. 
I'm going to call my environment `rTREES_env`: `conda create --name rTREES_env r-base=4.2.3`. 

Note the "r-" prefix when we specified the R version. Conda assumes that everything is a python package. We can tell it to search R packages instead by specifying the "r-" prefix. "r-ggplot2", "r-dplyr", etc. This only applies when using conda, and has no effect when you're in Rstudio, for example.

Conda will then spit out a few lines where it tries to figure out how to create a new environment without creating any package conflicts (it'll say `solving environment` for a minute or two). 

It'll then spit out some new lines saying what it will update, under `# Package Plan #`. Take a look at you want, but there's nothing super interesting here right now. It just says the R version to install (4.2.3), then the R packages (pip, python, readline, tk, etc) it will install by default, and then where it will download those packages from. 

Type `y` for "yes" and hit enter to proceed. It will do more thinking while they install, then spit out a couple of lines of instructions, telling you how to enter (Activate) or leave (Deactivate) the environment

For me, this took about 5 minutes. Here's what my terminal looks like now:

```
(base) alex@Alexs-MacBook-Pro ~ % conda create --name rTREES_env r-base=4.2.3            
Collecting package metadata (current_repodata.json): done
Solving environment: done


==> WARNING: A newer version of conda exists. <==
  current version: 22.11.1
  latest version: 23.3.1

Please update conda by running

    $ conda update -n base -c defaults conda

Or to minimize the number of packages updated during conda update use

     conda install conda=23.3.1



## Package Plan ##

  environment location: /Users/alex/opt/anaconda3/envs/rTREES_env

  added / updated specs:
    - r-base


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    _r-mutex-1.0.1             |      anacondar_1           3 KB  conda-forge
    bwidget-1.9.14             |       h694c41f_1         119 KB  conda-forge
    ca-certificates-2023.5.7   |       h8857fd0_0         145 KB  conda-forge
    # .... I removed lots of packages here to save you from scrolling forever.
    openssl-3.1.0              |       h8a1eda9_3         2.2 MB  conda-forge
    pango-1.50.14              |       hbce5e75_1         409 KB  conda-forge
    r-base-4.3.0               |       h7a6543b_0        24.2 MB  conda-forge
    tktable-2.10               |       h49f0cf7_3          79 KB  conda-forge
    ------------------------------------------------------------
                                           Total:       125.4 MB

The following NEW packages will be INSTALLED:

  _r-mutex           conda-forge/noarch::_r-mutex-1.0.1-anacondar_1 
  bwidget            conda-forge/osx-64::bwidget-1.9.14-h694c41f_1 
  bzip2              conda-forge/osx-64::bzip2-1.0.8-h0d85af4_4 
  c-ares             conda-forge/osx-64::c-ares-1.18.1-h0d85af4_0 
  ca-certificates    conda-forge/osx-64::ca-certificates-2023.5.7-h8857fd0_0 
  cairo              conda-forge/osx-64::cairo-1.16.0-h297c08e_1015 
  # .... I removed lots of packages here to save you from scrolling forever.
  r-base             conda-forge/osx-64::r-base-4.3.0-h7a6543b_0 
  readline           conda-forge/osx-64::readline-8.2-h9e318b2_1 
  sigtool            conda-forge/osx-64::sigtool-0.1.3-h88f4db0_0 
  tapi               conda-forge/osx-64::tapi-1100.0.11-h9ce4665_0 
  tk                 conda-forge/osx-64::tk-8.6.12-h5dbffcc_0 
  tktable            conda-forge/osx-64::tktable-2.10-h49f0cf7_3 
  xz                 conda-forge/osx-64::xz-5.2.6-h775f41a_0 
  zlib               conda-forge/osx-64::zlib-1.2.13-hfd90126_4 
  zstd               conda-forge/osx-64::zstd-1.5.2-hbc0c0cd_6 


Proceed ([y]/n)? y


Downloading and Extracting Packages
                                                                                          
Preparing transaction: done                                                               
Verifying transaction: done                                                               
Executing transaction: done                                                               
#                                                                                         
# To activate this environment, use                                                       
#                                                                                         
#     $ conda activate rTREES_env                                                              
#                                                                                         
# To deactivate an active environment, use                                                
#                                                                                         
#     $ conda deactivate                                                                  
                                                                                          
(base) alex@Alexs-MacBook-Pro ~ %
```

## Enter or Exit your new conda environment

type `conda env list` to see the names of all current environments that conda knows about, and where they are installed. Mine are located in `/Users/alex/opt/anaconda3` directory:

```
(base) alex@Alexs-MacBook-Pro ~ % conda env list
# conda environments:
#
base                  *  /Users/alex/opt/anaconda3
rTREES_env               /Users/alex/opt/anaconda3/envs/rTREES_env
```

The `*` indicates the environment that's active right now. If you read the technical aside, then I can tell you that the active environment is the one that shows up in your PATH, and that activating a different environment will change your PATH variable to reflect the active environment. 

You can delete the environment using `conda env remove --name rTREES_env`, but don't do that (unless you think you screwed something up).

Activate the environment: `conda activate rTREES_env`. This leaves (deactivates) the current environment. If conda environments are like the rooms in a house, you just left the entryway and walked into the kitchen.

Your terminal line should now look like this: `(rTREES_env) alex@Alexs-MacBook-Pro ~ %`

You can leave the environment by deactivating it: `conda deactivate` (don't specify the env name). This will take you back to `(base)`.

## Verify you created the environment properly

You can see all the packages available in your environment using `conda list`

Look through the package list to make sure that `r-base 4.2.3` is there somewhere.

Now check which version of R actually runs from the command line by running the commands `R --version` and `which R`. 
If `R --version` gives you a version other than 4.2.3, or if the output of `which R` doesn't end with `/opt/anaconda3/envs/rTREES_env`, then ask Alex for help.


## Install rTREES from source
Right now, the alpha version of rTREES is only available to be compiled from the sourcecode. 

To install R packages this way, you need the devtools package. Install this using the `conda install` command: `conda install r-devtools`

Now go to https://buffalo.app.box.com/s/ref6bq1ww4ony2gr0h4poddphydoe3e5 and download the tarball (`.tar.gz` file) containing the rTREES source code. This includes both the rTREES package and the TREES program.

Once you've done this, use devtools to install it. This should be done inside of R. Run the following command:

```bash
(rTREES_env) alex@Alexs-MacBook-Pro ~ % R -e "devtools::install_local('<path/to/rTREES.tar.gz>')"
```

where `'<path/to/rTREES.tar.gz>'` is replaced by the path to the downloaded file. For me, this is `'/Users/alex/Downloads/rTREES_2023.04.26.tar.gz'`

This will generate a TON of output, and also some compile warnings. This took 5-10 minutes on my computer. It should end when it says `* DONE`.

Check that it installed properly: type the command `R -e ".libPaths()"` into your terminal. This will show you where R installs packages to. We need to do this because `devtools` might install packages to a different location than anaconda might. Make sure that this command shows something in the `opt/anaconda3/envs/rTREES_env/lib/R/library` directory:

If installing devtools or compiling rTREES failed, I might be able to help, but there's a good chance your problem is beyond my understanding.

## Confirm rTREES works

1. open R: enter the command `R` in the terminal
```bash
(rTREES_env) alex@Alexs-MacBook-Pro ~ % R

R version 4.2.3 (2023-03-15) -- "Shortstop Beagle"
Copyright (C) 2023 The R Foundation for Statistical Computing
Platform: x86_64-apple-darwin13.4.0 (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

  Natural language support but running in an English locale

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.
```

2. Load rTREES:

```R
> library(rTREES)
Loading required package: data.table
data.table 1.14.8 using 2 threads (see ?getDTthreads).  Latest news: r-datatable.com
Loading required package: dplyr

Attaching package: ‘dplyr’

The following objects are masked from ‘package:data.table’:

    between, first, last

The following objects are masked from ‘package:stats’:

    filter, lag

The following objects are masked from ‘package:base’:

    intersect, setdiff, setequal, union

Loading required package: tidyr
Loading required package: doParallel
Loading required package: foreach
Loading required package: iterators
Loading required package: parallel
Loading required package: tictoc

Attaching package: ‘tictoc’

The following object is masked from ‘package:data.table’:

    shift

Loading required package: ggplot2
Loading required package: ggthemes
```

3. Run the example dataset in TREES (you can find this in the ?rTREES help menu)

```R
example_inputs<-Load_TREES_files(
         driver=system.file("FP_ex","FP_driver.txt",package="rTREES"),
         parameters=system.file("FP_ex","FP_parameter.p",package="rTREES"),
         param_mod=system.file("FP_ex","FP_param_mod",package="rTREES")
)
example_run<<-rTREES(
         env_driver_in  =example_inputs$put_driver,
         base_parameters = example_inputs$put_parameters,
         root_shoot_parameters = example_inputs$put_param_mod
)
rTREES_cleaned_ex<<-Clean_rTREES_output(
         rTREES_output = example_run,
         driver = example_inputs$put_driver
)
```
The first command (`Load_TREES_files`) will generate some example inputs (drivers, parameters, and plant geometry).
The second command (`rTREES`) will run TREES using those inputs, and should generate a lot of output showing you that the model is running.
The third command (`Clean_rTREES_output`) will generate a dataframe of some simple outputs.

## Install some more R Packages

Most of your needs will be taken care of with the essentials package, which includes things like ggplot2 and tidyverse.

```
conda install r-essentials
```

Additional packages can be installed using `conda install r-packagename`. 
You can specify the version of the package using `conda install r-packagename=X.Y.Z` where X.Y.Z is the version number. 
You can look up available versions using `conda search r-packagename` 

You can also do this within R (running in the proper virtual environment) using the `install.packages` command.

## Install RStudio in your conda environment

Run the command `conda install rstudio`. By now you should know that this will install RStudio in your environment directory, `~/opt/anaconda3/envs/rTREES_env`, separately from the version of RStudio that you (probably) use right now. 
This means that you will now have multiple versions of RStudio installed on your computer. 
Make sure to open the right one. 
If you have the RStudio icon in your dock on your desktop, that icon will open a separate version of RStudio not associated with conda, using a different R interpreter than the one conda uses (and probably different from the version in `/usr/local/bin` as well...see the technical aside below)
It will not know where your rTREES is installed. 
Instead, open rstudio from the terminal to get a version associated with the environment you're in:

```bash
(rTREES_env) alex@Alexs-MacBook-Pro ~ % rstudio
```

If rstudio asks you to update it, ignore the prompt and instead update it from the command line:

```bash
(rTREES_env) alex@Alexs-MacBook-Pro ~ % conda update rstudio
```

You can run all the same checks to make sure that RStudio is using the correct interpreter and libraries:

* Check that calling `.libPaths()` points you to the anaconda environment
* Check that rTREES works

You can exit rstudio by going to the command line and hitting "ctrl+C". This will exit the program.

# Recap 

Here's what we did

1. We installed anaconda from the website
2. We created a virtual environment in conda: `conda create --name rTREES_env r-base=4.2.3`
3. We entered that environment: `conda activate rTREES_env`
4. We installed devtools: `conda install r-devtools`
5. We downloaded the rTREES sourcecode and compiled it using devtools: `R -e "devtools::install_local('<path/to/rTREES.tar.gz>')"`
6. We tested rTREES
7. We installed additional packages.
8. We installed RStudio in our rTREES environment using  `conda install rstudio`, then ran it using the `rstudio` command.

Any time you want to set up another conda environment, follow steps 2.2-2.5. Of course, use a different environment name, select the version of R that you want, and install whatever R packages you need there.

# Now what?
Here's how you can actually use this environment:

1. Enter your conda environment: `conda activate rTREES_env`
2. Start jupyter: `rstudio`
3. Close rstudio when you're done, by hitting "ctrl+c" in the terminal.

That's all!

# Uh oh...I messed up

Here's what to do if you think you made a mistake, or forgot something:

I forgot what my environment is called: `conda env list`

I installed the wrong environment, and wish to delete it: `conda env remove --name unwantedEnvironmentName`

I don't know what packages I installed in my environment: `conda list` in terminal or `installed.packages()` in R. rTREES might not show up using `conda list`, since it was installed manually.

I installed the wrong package: `conda remove unwantedPackage` in terminal or `remove.packages(...)` in R.

My version of R (`R --version`) is wrong, or is located somewhere weird, or none of my packages will load even though they show up in my environment: ask Alex for help.


# Technical Aside: How is (base) different from my default shell outside of conda?
When you run a program, for example when you run R from the command line, your computer has to know where to find that program. 
There's this variable called `PATH` that your computer uses to locate programs. If you type `echo $PATH` into your terminal, you might see something like this:

```bash
(base) alex@Alexs-MacBook-Pro ~ % echo $PATH
/Users/alex/opt/anaconda3/bin:/Users/alex/opt/anaconda3/condabin:/opt/local/bin:/opt/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/go/bin:/opt/X11/bin:/Library/Apple/usr/bin:/Library/Frameworks/Mono.framework/Versions/Current/Commands:/Users/alex/Code Projects/eddypro-engine-master/bin/mac
```

This is just a list of directories. Your computer goes through this list, item by item, until it finds what it's looking for. 
If I type `R` into my command line to run R, my computer will first look in `/usr/local/hdf5/bin`. If it doesn't find R there, it will move on to `/Applications/Snowpack/bin`, and so on.

If I type `which R` into my command line, my computer will tell me where it found it:

```bash
(base) alex@Alexs-MacBook-Pro ~ % which R
/usr/local/bin/R
```

If you look in the `PATH` variable above, you can see that `/usr/local/bin` in the 5th entry on that list. 
If R were *also* installed in any of the preceeding directories, like maybe `/Users/alex/opt/anaconda3/bin`, then when we typed `which R` into the console, we would instead get

```bash
(base) alex@Alexs-MacBook-Pro ~ % which R
/Users/alex/opt/anaconda3/bin/R
```

Note that anaconda inserted its *own* directory before anything else in `PATH`. 
That way, when your computer looks for a program, anaconda forces it to look in the anaconda directory before giving up and looking somewhere else.

That `/Users/alex/opt/anaconda3` directory is the "base" anaconda directory, and any programs we install using anaconda while in the base environment will go to `/Users/alex/opt/anaconda3/bin`. 
To ask anaconda where it thinks it should look for packages, type `conda env list`:

```bash
(base) alex@Alexs-MacBook-Pro ~ % conda env list
# conda environments:
#
base                  *  /Users/alex/opt/anaconda3
```

Note that this is the same directory that shows up first in your PATH.

When we create new environments, those go in subdirectories of the base directory. 
Say we make a new virtual environment called "rTREES_env" and activate it:

```bash
(base) alex@Alex-MacBook-Pro ~ % conda activate rTREES_env
(rTREES_env) alex@Alexs-MacBook-Pro ~ % conda env list
# conda environments:
#
base                     /Users/alex/opt/anaconda3
rTREES_env            *  /Users/alex/opt/anaconda3/envs/rTREES_env
```
Then anaconda would create a directory called `/Users/alex/opt/anaconda3/envs/rTREES_env`, and any programs installed from that environment would go into the directory `/Users/alex/opt/anaconda3/envs/rTREES_env/bin`.
When we enter that new environment, anaconda will change the PATH variable to direct everything first to that environment, instead of to the base environment:

```bash
(rTREES_env) alex@Alexs-MacBook-Pro ~ % echo $PATH
/Users/alex/opt/anaconda3/envs/rTREES_env/bin:/Users/alex/opt/anaconda3/condabin:/opt/local/bin:/opt/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/go/bin:/opt/X11/bin:/Library/Apple/usr/bin:/Library/Frameworks/Mono.framework/Versions/Current/Commands:/Users/alex/Code Projects/eddypro-engine-master/bin/mac
```

If a version of R is installed in the `/Users/alex/opt/anaconda3/envs/rTREES_env/bin` directory, then that version is what will run when we ask the terminal to open rTREES. 
If we wanted to open the version outside of anaconda, the version in `/usr/local/bin`, then we would have to do that directly, by running the command `/usr/local/bin/R`. This circumvents the PATH variable and directly opens the version of R in that directory.
