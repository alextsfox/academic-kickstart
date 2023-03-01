---
title: "Working in Python with Jupyter and Anaconda"
date: 2023-03-01T11:12:15-06:00
draft: false
tags:
    - Tutorials
---
Image source: https://xkcd.com/1987/

I wrote this tutorial to help people in my lab learn how to manage their python and R environments using anaconda. I've gotten some positive feedback on it, so hopefully you'll find it helpful too.


# Jupyter, Python, and Anaconda on a mac

Jupyter is a development environment for creating markdown notebooks: scripts containing code interspersed with formatted markdown text. This document, for example was written as an R-Markdown notebook. Notebooks are great ways of sharing simple workflows and demonstrations, or for just keeping track of your own work. There are of course other ways to user python: directly through the terminal, using a traditional IDE like Spyder or PyCharm, or by writing python scripts in a text editor and executing them from the command line. 

Jupyter itself is a  collection of different development environments: 

* jupyter-notebook provides a very lightweight interface. It provides the ability to create individual notebooks. 
* jupyter-lab isa little more complex: it provides more ways of writing and managing individual notebooks. It's very similar to a jupyter-notebook, but with more bells and whistles.
* jupyter-hub is meant for large collaborations: it lets multiple users interact with one jupyter notebooks, and incorporates many more bells and whistles like Docker Containers to allow for code and dependencies to be standardized across many people's computers. 

You interact with jupyter-lab through your web browser. I'm not sure why this is, but I think it makes it so that they don't have to develop their own app interface, and can just write everything in html or something. It doesn't mean that jupyter-lab connects to a website, just that it uses your web browser application as its GUI.

Jupyter-lab and Jupyter-notebooks are akin to working with an Rmarkdown notebook.

In this document I'll be talking mostly about jupyter-lab, but everything applies to jupyter-notebook as well.

Jupyter-lab can be used on its own with your base version of python, but I really recommend using an environment manager as well. Python is famously messy with how it's installed, so it's good to get ahead of the curve and just use an environment manager like anaconda.

# Anaconda environments
Anaconda helps to manage virtual environments which are like little quarantine chambers for each of your code projects

A virtual environment is a collection of software and packages that is segregated from other virtual environments to prevent conflicts between them.

Say that you're working on project A that relies on python version 2.7 (This is probably what your computer came with if you have a mac. If you have a mac, open a terminal and type `python --version` to check), but you have another project (B) that uses PyMC (a Bayesian modeling package), which needs python version 3.6. You can't update python to version 3.6, because that would break everything in your first project! Instead, you put each project into its own virtual environment so that they can't interfere. Any time you want to work on project A, you can enter the environment you set up for it, and likewise for project B. There's nothing stopping you from working on project A in the wrong environment, but it probably won't work when you try to run your code.

# Installing and Setting Up Anaconda

## Install Anaconda
You can install anaconda from here: https://www.anaconda.com/. Download the graphical installer and follow the instructions.

This will install both the command line interface and the "anaconda navigator" GUI. The GUI can be nice (for example, it comes pre-installed with jupyter usually), but it can be very slow. Below, I'll explain how to interact with the command line interface: how to create environments, install packages, and install jupyter

## Start Anaconda
Once anaconda has been installed, open up a terminal and type `conda --version`. This hopefully should show that you have the latest version (22.11.1 or 23.1.0 or something? not sure)

Maybe you will also see next to your username, the word "base": mine says `(base) alex@Alexs-MacBook-Pro ~ %`. This indicates that you are currently inside the default anaconda environment. 

## Create a new environment
create a new virtual environment with `conda create --name choose_your_favorite_name python=choose_your_favorite_python_version`. I'm going to call my environment `test_env` and use python version 3.9: `conda create --name test_env python=3.9`. 

Conda will then spit out a few lines where it tries to figure out how to create a new environment without creating any package conflicts (it'll say `solving environment` for a minute or two). 

It'll then spit out some new lines saying what it will update, under `# Package Plan #`. Take a look at you want, but there's nothing super interesting here right now. It just says the python version to install (3.9), then the python packages (pip, python, readline, tk, etc) it will install by default, next to where it will download those packages from.

Type `y` for "yes" and hit enter to proceed. It will do more thinking while they install, then spit out a couple of lines of instructions, telling you how to enter (Activate) or leave (Deactivate) the environment

Here's what my terminal looks like now:

```
(base) alex@Alexs-MacBook-Pro ~ % conda create --name test_env python=3.9
Collecting package metadata (current_repodata.json): done
Solving environment: done


==> WARNING: A newer version of conda exists. <==
  current version: 22.11.1
  latest version: 23.1.0

Please update conda by running

    $ conda update -n base -c defaults conda

Or to minimize the number of packages updated during conda update use

     conda install conda=23.1.0



## Package Plan ##

  environment location: /Users/alex/opt/anaconda3/envs/test_env

  added / updated specs:
    - python=3.9


The following NEW packages will be INSTALLED:

  bzip2              conda-forge/osx-64::bzip2-1.0.8-h0d85af4_4 
  ca-certificates    conda-forge/osx-64::ca-certificates-2022.12.7-h033912b_0 
  libffi             conda-forge/osx-64::libffi-3.4.2-h0d85af4_5 
  libsqlite          conda-forge/osx-64::libsqlite-3.40.0-ha978bb4_0 
  libzlib            conda-forge/osx-64::libzlib-1.2.13-hfd90126_4 
  ncurses            conda-forge/osx-64::ncurses-6.3-h96cf925_1 
  openssl            conda-forge/osx-64::openssl-3.0.7-hfd90126_2 
  pip                conda-forge/noarch::pip-23.0-pyhd8ed1ab_0 
  python             conda-forge/osx-64::python-3.9.15-h709bd14_0_cpython 
  readline           conda-forge/osx-64::readline-8.1.2-h3899abd_0 
  setuptools         conda-forge/noarch::setuptools-66.1.1-pyhd8ed1ab_0 
  tk                 conda-forge/osx-64::tk-8.6.12-h5dbffcc_0 
  tzdata             conda-forge/noarch::tzdata-2022g-h191b570_0 
  wheel              conda-forge/noarch::wheel-0.38.4-pyhd8ed1ab_0 
  xz                 conda-forge/osx-64::xz-5.2.6-h775f41a_0 


Proceed ([y]/n)? y


Downloading and Extracting Packages

Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate test_env
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

## Enter or Exit your new conda environment

type `conda env list` to see the names of all current environments that conda knows about, and where they are installed. Mine are located in `/Users/alex/opt/anaconda3`:

```
(base) alex@Alexs-MacBook-Pro ~ % conda env list
# conda environments:
#
base                  *  /Users/alex/opt/anaconda3
test_env                 /Users/alex/opt/anaconda3/envs/test_env
```

The `*` indicates the environment you're in right now.

You can delete the environment using `conda env remove --name test_env`, but let's not do that.

Enter the environment by activating it: `conda activate test_env`. This leaves (deactivates) the current environment.

Your terminal line should now look like this: `(test_env) alex@Alexs-MacBook-Pro ~ %`

Leave the environment by deactivating it: `conda deactivate` (don't specify the env name). This will take you back to `(base)`.

## Modify installed packages

You can see all the packages available in your environment using `conda list`:

```
(test_env) alex@Alexs-MacBook-Pro ~ % conda list
# packages in environment at /Users/waldinian/opt/anaconda3/envs/test_env:
#
# Name                    Version                   Build  Channel
bzip2                     1.0.8                h0d85af4_4    conda-forge
ca-certificates           2022.12.7            h033912b_0    conda-forge
libffi                    3.4.2                h0d85af4_5    conda-forge
libsqlite                 3.40.0               ha978bb4_0    conda-forge
libzlib                   1.2.13               hfd90126_4    conda-forge
ncurses                   6.3                  h96cf925_1    conda-forge
openssl                   3.0.7                hfd90126_2    conda-forge
pip                       23.0               pyhd8ed1ab_0    conda-forge
python                    3.9.15          h709bd14_0_cpython    conda-forge
readline                  8.1.2                h3899abd_0    conda-forge
setuptools                66.1.1             pyhd8ed1ab_0    conda-forge
tk                        8.6.12               h5dbffcc_0    conda-forge
tzdata                    2022g                h191b570_0    conda-forge
wheel                     0.38.4             pyhd8ed1ab_0    conda-forge
xz                        5.2.6                h775f41a_0    conda-forge
```

Make sure that the python version is 3.9 somewhere in the middle.

Next make sure that this is the same version of python that will actually run. When you run python, your terminal has to know where to find python 3.9 associated with this environment, which should be installed wherever your environment is installed. Check this by typing `which python` and `python --version` into the terminal:

```
(test_env) alex@Alexs-MacBook-Pro ~ % which python
/Users/alex/opt/anaconda3/envs/test_env/bin/python
(test_env) alex@Alexs-MacBook-Pro ~ % python --version
Python 3.9.15
```

if it says something significantly different from the above (such as `/usr/bin/python` or any version other than 3.9), please ask Alex for help.

## Install some python packages

I'd like to use NumPy, SciPy, Pandas, and Matplotlib on this project. Install these with `pip`, which is a smart package to help download and install python packages:

```
pip install numpy scipy pandas matplotlib
```

Some packages cannot (or should not) be installed with pip, like PyMC. In general, it's always a good idea to look on a package website for specific installation instructions. For most packages though, you can safely install them with pip. Instead of using pip, you could also have used conda to install things, using `conda install yourpackagenamehere`. This is generally  much slower, but it will check for conflicts before installing, which pip will not always do, so it's less likely to break things on very complicated environments with hundreds of packages.

If you now run `conda list` again, the list of packages should be much much longer (since the packages we installed have lots of dependencies), but you should be able to scroll through and see some familiar names.

# Install Jupyter, and connect it to your environment

## Install jupyter in test_env
To install jupyter-lab: `pip install jupyterlab`

To install jupyter-notebook instead (also): `pip install notebook`

Great! We can run jupyter now right? Well, you can, but you won't have any access to `test_env`. Now you need to .....

## Tell jupyter where to find the iPykernel associated with test_env...yeesh. I promise we're almost done.

Install the ipython kernel on this version of python. You can make display-name whatever you want, but it's a very good idea to make it identical to the environment name.

`python -m ipykernel install --user --name=test_env --display-name test_env`

Check to make sure that your kernel was installed, and that jupyter now know where it is with `jupyter kernelspec list`

```
(test_env) alex@Alexs-MacBook-Pro ~ % jupyter kernelspec list
Available kernels:
  %s    %s python3     /Users/alex/opt/anaconda3/envs/test_env/share/jupyter/kernels/python3
  %s    %s test_env    /Users/alex/Library/Jupyter/kernels/test_env
```

# Run Jupyter

Yay! Now you can run jupyter

## Open jupyter
`jupyter-lab` or `jupyter-notebook` will do this. Running this command will fill up your terminal with what is seemingly gibberish, and should open up a new tab in your web browser at the web address `http://localhost:8888`.

When you create a new file, you select the environment you want to use.

If you still want to use the terminal, create a new terminal window with `cmd+tab` or `cmd+n`.

## Use Jupyter

Create a new notebook file

* Jupyter-notebook: click "new" and select the kernel name you want
* Jupyter-lab: click the blue "+" and select the kernel name you want in the window that pops up

You can change the kernel on an active notebook easily in jupyter-lab by clicking on the kernel name in the top right and selecting the new one that you want.

For everything else about using jupyter, I suggest going to youtube.

## Close your browser

If you close your browser, jupyter will still be running. Reconnect by going to `http://localhost:8888`.

## Quit jupyter

Quit jupyter entirely by typing `ctrl+c` in your terminal, and type `y` for yes.

```
[W 2023-01-31 15:25:34.105 LabApp] Could not determine jupyterlab build status without nodejs
[I 2023-01-31 15:27:35.654 ServerApp] Saving file at /Untitled5.ipynb
^C[I 2023-01-31 16:59:06.613 ServerApp] interrupted                      <----- Here is where I types "ctrl+c" to quit
Serving notebooks from local directory: /Users/waldinian
2 active kernels
Jupyter Server 1.23.4 is running at:
http://localhost:8888/lab?token=db46e92faeaa54ef90a94c25f44d0bdee983768b2a8146de
 or http://127.0.0.1:8888/lab?token=db46e92faeaa54ef90a94c25f44d0bdee983768b2a8146de
Shutdown this Jupyter server (y/[n])? y                                <-------- Here is where I typed "y" to agree
[C 2023-01-31 16:59:08.509 ServerApp] Shutdown confirmed
[I 2023-01-31 16:59:08.523 ServerApp] Shutting down 4 extensions
[I 2023-01-31 16:59:08.525 ServerApp] Shutting down 2 kernels
[I 2023-01-31 16:59:08.539 ServerApp] Kernel shutdown: 0a9912f1-a540-466d-86a9-2b42d3d2325f
[I 2023-01-31 16:59:08.548 ServerApp] Kernel shutdown: ddccb903-6b98-44b1-906e-3cc2dc8bdc79
[I 2023-01-31 16:59:09.294 ServerApp] Shutting down 0 terminals
(base) alex@Alexs-MacBook-Pro ~ % 
```

You can also just close the terminal window.

# Recap 

Here's what we did

1. We installed anaconda from the website
2. We created a virtual environment in conda: `conda create --name test_env python=3.9`
3. We entered that environment: `conda activate test_env`
4. We installed some packages there: `pip install numpy scipy pandas matplotlib`
5. We installed jupyter: `pip install jupyter-lab` or `pip install notebook`
6. We installed installed the iPykernel in our environment to let jupyter talk to it: `python -m ipykernel install --user --name=test_env --display-name test_env`
7. We started jupyter-lab (or a jupyter-notebook): `jupyter-lab` or `jupyter-notebook`
8. We quit jupyter with `ctrl+c`.

Any time you want to set up another conda environment, follow steps 2-6. Of course, use a different environment name, select the version of python that you want, and install whatever python packages you need.

# Now what?
Here's how you can start jupyter again, now that everything is already set up

1. Enter your conda environment: `conda activate test_env`
2. Start jupyter: `jupyter-lab`

That's all!

# Uh oh...I messed up

I forgot what my environment is called: `conda env list`

I installed the wrong environment, and wish to delete it: `conda env remove --name unwantedEnvironmentName`

I don't know what my kernel was called: `jupyter kernelspec list`

I installed the wrong kernel, or I want to remove a vestigial kernel from an environment that I deleted: `jupyter kernelspec uninstall unwantedKernelName` or `jupyter kernelspec remove unwantedKernelName` 

I don't know what python packages I installed in my environment: `conda list` or `pip list`

I installed the wrong python package: `pip uninstall unwantedPackage` or `conda remove unwantedPackage`

I closed my browser and want to start jupyter again: connect to `http://localhost:8888` in your web browser. If you had multiple sessions open, you may have to use `8889` or something instead.

My version of python (`python --version`) is wrong, or is located somewhere weird, or none of my packages will load (`ModuleNotFoundError` in python) even though they show up in my environment: ask Alex for help.

