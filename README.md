Chromium-OzoneGBM
=================

My Personnel repo to maintain all needed dependencies to run Chromium Browser with Ozone-GBM backend on Linux Desktop.

#Clone, Configure & build Mesa

You might need to install any dependencies for Mesa specific to your distribution. These instructions assume you have done this already.

Setup environment for local install:
 ```
  $export WLD=$HOME/install 
  $export LD_LIBRARY_PATH=$WLD/lib
  $export PKG_CONFIG_PATH=$WLD/lib/pkgconfig/:$WLD/share/pkgconfig/
  $export ACLOCAL_PATH=$WLD/share/aclocal
  $export ACLOCAL="aclocal -I $ACLOCAL_PATH"
  ```
## Clone & Build DRM
 ```
  $ git clone git://anongit.freedesktop.org/git/mesa/drm  
  $ cd drm
  $ ./autogen.sh --prefix=$WLD
  $ make -j4 && make install
  ```

## Clone &Build Mesa

 ```
  $ git clone git://anongit.freedesktop.org/mesa/mesa
  $ cd mesa
  Apply Mesa patches 0001 -003* found in the repo.
  $ ./autogen.sh --prefix=$WLD --enable-gles2 --disable-gallium-egl \
  --with-egl-platforms=drm --enable-gbm --disable-dri3 --enable-shared-glapi \
  --with-gallium-drivers= --enable-texture-float --with-dri-drivers=i965 \
  --disable-llvm-shared-libs --disable-glu --disable-glut --without-demos \
  --disable-option-checking --disable-glx
  $ make -j4 && make install
  ```
TODO: Add steps to build libva and vaapi.

## Clone, Configure & build Chromium
HW: At Least 20GB of hard disk space is recommended. I have a dual channel 8GB Ram (16GB in Total). If you have like 8GB ram in total, I recommend updating it to 16GB, unless you want to spend lot of time linking things (if you go for static_library builds mentioned below).

We need Depot tools before doing anything.

### Depot Tools
Chromium uses a package of scripts called depot_tools to manage checkouts and code reviews.

Cloning Depot Tools:
 ```
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
  ```
Add Depot Tools to your path:
 ```
$ export PATH=`pwd`/depot_tools:"$PATH"
  ```
If you don't want to set this manually every time you open a new shell, add this to .bashrc or whatever is your shell equivalent.

### First time Chromium Checkout
 ```
$ fetch --nohooks chromium
or
$ fetch --nohooks --no-history chromium # get a shallow checkout (saves disk space and fetch time at the cost of no git history)
Now, we should have a new directory src.
$ cd src
$ git checkout master
  ```

### Dependencies:
 ```
$ build/install-build-deps.sh
  ```
  
I am not sure if it works for all Linux distributions. If it fails, You can look at the dependencies list here https://src.chromium.org/svn/trunk/src/build/install-build-deps.sh 
and install your distribution equivalent.

### Update Chromium code to latest master TOT:
 ```
$ gclient sync
  ```
### Configure Chromium

Gyp: The cross-platform build configuration system is called gyp, and on Linux it generates ninja build files. Running gyp is analogous to the ./configure step seen in most other software.

Ninja: The actual build itself uses ninja. A prebuilt binary is in depot_tools and should already be in your path if you followed the steps above.

Firstly you need to export the GYP_DEFINES(which will be later used by Ninja while compiling):

 ```
$ export GYP_DEFINES="component=static_library use_ash=1 use_aura=1 chromeos=1 use_ozone=1 ozone_platform=gbm remove_webcore_debug_symbols=1"
  ```
### Update Project files
 ```
$ build/gyp_chromium
  ```
You need to re-run this every time GYP_DEFINES are changed or any gyp files have changed..

### Build Chromium
 ```
$ ninja -C out/Debug chrome -> Debug build.
$ ninja -C out/Release chrome -> Release build.
  ```
### Launch Chromium
You might need to export EGL_PLATFORM=drm. I needed this as Mesa used to default to X11 otherwise, causing crashes during initialization.
Set LD_LIBRARY_PATH to point to your local install. i.e. LD_LIRARY_PATH=$WLD/lib
 ```
$ out/Debug chrome --disable-sandbox --ozone-platform=gbm --ozone-use-surfaceless
  ```
