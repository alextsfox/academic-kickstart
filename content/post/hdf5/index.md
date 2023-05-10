---
title: "Pulling my hair out over HDF files"
date: 2023-04-25T11:50:15-06:00
draft: false
tags:
    - Tutorials
---

# Installing HDF libraries (on macOS 11.3)

HDF files are crucial for interacting with remotely sensed data or any atmospheric-type data, really. NASA, MODIS, and many other satellites distribute data in this format, and NEON does too.
In my opinion, once you get them working, they're a nice way of storing really complex, multi-demensional data in a hierarchical format: each datapoint can be associated with some set of coordinates and metadata, making it easy to plot and analyze, and to keep track of your workflows as you do so.
Some people prefer relational databases as a structure for this type of data given how uncomplicated they are, but HDF files have a leg up on relational databases on computational capability.
Regardless of whether you think they're a good format, they'll be here to stay for a while.

Unfortunately, HDF files are a NIGHTMARE to deal with if you don't know what you're doing (like me). There's lots of different little sub-formats that each require their own plugins and libraries to read properly.
In this post, I'll detail my process going through this, and getting to the point where I can successfully open (most) HDF files out there.

Normally, I can work with HDF5 files just fine. Scipy seems to be compatible with some basic HDF file types, specifically netCDF4 files. netCDF4 files use the HDF5 format at their backend, but for some reason this doesn't carry over to other forms of files that use HDF5 as their backend.
To solve this, I think I need to install the HDF5 backend by itself from the HDF5 group.

So, here's how I did that:

## Installing HDF5 libraries
First, install some dependencies. Go to https://portal.hdfgroup.org/display/support/Downloads. 

At the bottom, click the links for ZLIB and AEC to download those dependencies. And SZIP too, which will be useful later.

If any of the `make` or `make check` or `./config` commands fail, open the `README` or `INSTALL` files of these libraries to get more detailed info. Some have great, in-depth troubleshooting tips.

### ZLIB
Go to http://www.zlib.net/ and download the latest `.tar.gz` file. Unpack it. Then:

```shell
cd zlib-X.Y.Z # X.Y.Z is your version number
./configure; make test
make install # may need to sudo
```

### AEC
https://gitlab.dkrz.de/k202009/libaec/-/releases

make sure to download the RELEASE, not the source code.

From the libaec directory:
```shell
cd libaec-X.Y.Z
mkdir build
cd build
../configure
make check install
```

### SZIP
https://portal.hdfgroup.org/display/HDF5/Szip+Compression+in+HDF+Products

```shell
cd szip-X.Y.Z
./configure # automatically installs in /usr/local/szip
make
make check
make install # may need to sudo
```

### Install HDF5

Download the latest `.tar.gz` from https://portal.hdfgroup.org/display/support/Downloads

I downloaded the latest (1.14) `.tar.gz` file
3. I unzipped it. You can do this by either double clicking it on a mac, or running `gunzip < hdf5-X.Y.Z.tar.gz | tar xf -`, where X.Y.Z give the version number
4. I followed the directions in the `release_docs/INSTALL` file, section 2:
    ```
    cd hdf5-X.Y.Z
    ./configure --prefix=/usr/local/hdf5 # takes a couple minutes
    make # takes 5-10 minutes
    make check # takes 10-15 minutes
    make install # may need to sudo
    make check-install # may need to sudo
    ```

    Most HDF5 libraries like `h5py` seem to look for plugins in the `/usr/local/hdf5` directory. 
    You can supposedly change this with environment variables, but this is a sloppy workaround. 
    Best to just use the `/usr/local/hdf5` directory by default when you install the libraries.

Now that these are all installed, you might want to add the HDF5 tools to your path. If you're on a newer mac (macOS 11 is what I'm on), go into the `~/.zshrc` file and add the following line:
```shell
# HDF5
export PATH="/usr/local/hdf5/bin":$PATH
```

Now you can call any of the hdf5 tools on your file from anywhere

### HDF-EOS
HDF-EOS is used by NASA, I guess. 

Install it from here: http://hdfeos.org/software/library.php#HDF-EOS5

Once you have the `.tar.gz` file, unzip it and enter the following commands. Make sure to get the HDF-EOS5 version.
```shell
cd hdf-eos5-X.Y
.\configure CC=/usr/local/hdf5/bin/h5cc --with-szlib=no
make
make check
make install # may need to sudo
```

You might get lots of warnings when running `make`, but if `make check` runs okay, then perhaps don't worry?






## Installing HDF4 libraries

Next, I needed to have a way of reading HDF4 files. To read HDF4 files, you will need to first install compression filters for...deep breath...
* SZIP
* JPEG 
* ZLIB
* AEC

Down at the bottom of https://portal.hdfgroup.org/display/support/Download+HDF4, there's a list of downloads for these four libraries. They're all `.tar.gz` files. You might have to navigate for a sec at a couple of the links to get the files you want.

If any of the `make` or `make check` or `./config` commands fail, open the `README` or `INSTALL` files of these libraries to get more detailed info. Some have great, in-depth troubleshooting tips.
### ZLIB
```
cd zlib-X.Y.Z
./configure; make test
make install # may need to sudo
```

### AEC
Not sure if this one is necessary

```
cd aec-X.Y.Z
./configure
make 
make check 
make install
```

For me, `make check` failed on the `sampledata.sh` and `szcomp.sh` tests. This is because (1) I don't have wget installed, and (2) the sample data isn't available at the URL it tries to access.
I didn't try to remedy this. Instead, I'm just going to cross my fingers and hope for the best.



### JPEG

```
cd jpeg-XY
./configure --prefix=/usr/local/jpeg
make
make test
make install # may need to sudo
```





### HDF4
If you have macOS 10.3 or higher (you probably do), you need to use the `--enable-hdf4-xdr` flag. Otherwise, omit it.

```
cd hdf-4.X.X
./configure --with-zlib=/usr/local/zlib --with-jpeg=/usr/local/jpeg --with-szlib=/usr/local/szlib --enable-hdf4-xdr --prefix=/usr/local/hdf4
make >& make.out # doesn't generate stdout, takes a few minutes
make check >& check.out # doesn't generate stdout, should be fast
make install # may need to sudo
make install-examples # may need to sudo
make installcheck # you'll get lots of warnings
```

### HDF5-EOS
This is for nasa stuff: https://wiki.earthdata.nasa.gov/display/DAS/Toolkit+Downloads

```


```

## Utilities

### h5utils
Lots of nice utilities here: https://github.com/NanoComp/h5utils

These utilities let you convert HDF files and check out their formats and metadata. If you can't open HDF files, this should help you figure out why, and maybe solve those problems.

Download the tarball and extract it. Then

```
cd h5utils-X.Y.Z


```


