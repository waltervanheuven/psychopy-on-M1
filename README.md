# PsychoPy v2021.2.3 on M1 Mac (Apple Silicon)

Current version of [PsychoPy](https://www.psychopy.org) runs using Rosetta. Below are the instructions how to install an 
Apple Silicon version of PsychoPy.

## First install latest python for M1 Macs using [brew](https://brew.sh)

Install latest python3 version on M1 Mac (tested with M1 MacBook Pro).

```sh
brew install python3
```

Python 3.9.7 (3 September 2021).

## QT5

Install QT5 using brew. Requires venv with system-site-packages to work in Python.

```sh
brew install pyqt5
```

Create virtual environment with system-site-packages.

```sh
python3 -m venv --system-site-packages ~/venv/psychopy
```

activate virtual environment

```sh
source ~/venv/psychopy/bin/activate
````

## numpy

```sh
pip install numpy
```

## scipy

```sh
export OPENBLAS=$(brew --prefix openblas)
export CFLAGS="-falign-functions=8 ${CFLAGS}"
pip install Cython pybind11 pythran
pip install --no-use-pep517 scipy
```

## pygame

```sh
brew install mercurial
brew install sdl sdl\_image sdl\_mixer sdl\_ttf portmidi
brew install smpeg
brew install sdl2
```

```sh
pip install pygame
```

## tables

```sh
brew install hdf5 c-blosc lzo bzip2
export HDF5_DIR=/opt/homebrew/opt/hdf5
export BLOSC_DIR=/opt/homebrew/opt/c-blosc
export LZO_DIR=/opt/homebrew/opt/lzo
export BZIP2_DIR=/opt/homebrew/opt/bzip2

pip install tables
```

## pyo

```sh
brew install liblo libsndfile portaudio portmidi
git clone https://github.com/belangeo/pyo.git
cd pyo
python setup.py install --use-coreaudio --use-double
```

Error:

```txt
src/engine/ad_coreaudio.c:132:21: error: implicit declaration of function 'AudioGetCurrentHostTime' is invalid in C99 [-Werror,-Wimplicit-function-declaration]
```

Hack to get pyo to compile. Change line 132 in ad_coreaudio.c to

```txt
    now.mHostTime = 0 ; // AudioGetCurrentHostTime();
```

## PsychToolbox

Download [psychtoolbox from pypi] (https://pypi.org/project/psychtoolbox/3.0.17.8/).
Download `psychtoolbox-3.0.17.8.zip`

Download: [ptb-wheels source] (https://github.com/aforren1/ptb-wheels)

From folder `ptb-wheels`:
Copy file `libportaudio_osx_64.a` to `PsychSourceGL/Cohorts/PortAudio/`
Copy file `libHID_Utilities64.a` to `PsychSourceGL/Cohorts/HID_Utilities_64Bit/build/Release/`

In folder `psychtoolbox-3.0.17.8` type:

```sh
python setup.py install

# check if psychtoolbox is successfully installed
pip list
```

## Final steps

```sh
export CXXFLAGS=-I/opt/homebrew/include
pip install wxPython

# needed for mp4
pip install ffmpeg

# install now psychopy
pip install psychopy
```

## Issues

- mp4 movies are not working
