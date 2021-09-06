# PsychoPy v2021.2.3 on M1 Mac (Apple Silicon)

The current version of [PsychoPy](https://www.psychopy.org) uses Rosetta 2 to run on M1 Macs. Below are the instructions to install an Apple Silicon native version of PsychoPy on an M1 Mac using `pip install psychopy`.

## First install latest Python for M1 Macs

Install the latest native Python3 version for M1 Macs using [brew](https://brew.sh).

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

Activate the virtual environment.

```sh
source ~/venv/psychopy/bin/activate
```

## Install numpy

```sh
pip install numpy
```

## Install scipy

```sh
export OPENBLAS=$(brew --prefix openblas)
export CFLAGS="-falign-functions=8 ${CFLAGS}"
pip install Cython pybind11 pythran
pip install --no-use-pep517 scipy
```

## Install pygame

```sh
brew install mercurial
brew install sdl sdl\_image sdl\_mixer sdl\_ttf portmidi
brew install smpeg
brew install sdl2
```

```sh
pip install pygame
```

## Install tables

```sh
brew install hdf5 c-blosc lzo bzip2
export HDF5_DIR=/opt/homebrew/opt/hdf5
export BLOSC_DIR=/opt/homebrew/opt/c-blosc
export LZO_DIR=/opt/homebrew/opt/lzo
export BZIP2_DIR=/opt/homebrew/opt/bzip2

pip install tables
```

## Install pyo

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

Add the following line to the file `include/ad_coreaudio.h` after line 24 to fix the issue:

```txt
#include <CoreAudio/HostTime.h>
```

## Install Psychtoolbox

- Download [psychtoolbox from pypi] (https://pypi.org/project/psychtoolbox/3.0.17.8/). Download `psychtoolbox-3.0.17.8.zip`
- Download: [ptb-wheels source] (https://github.com/aforren1/ptb-wheels)

From folder `ptb-wheels`:

- Copy file `libportaudio_osx_64.a` to `PsychSourceGL/Cohorts/PortAudio/`
- Copy file `libHID_Utilities64.a` to `PsychSourceGL/Cohorts/HID_Utilities_64Bit/build/Release/`

In folder `psychtoolbox-3.0.17.8` type:

```sh
python setup.py install

# check if psychtoolbox is successfully installed
pip list
```

## Install wxPython

```sh
export CXXFLAGS=-I/opt/homebrew/include
pip install wxPython
```

## Install ffmpeg

```sh
## needed for mp4
brew install ffmpeg
```

## Install SpeechRecognition

```sh
pip install SpeechRecognition
```

## Install Psychopy

```sh
pip install psychopy

# start psychopy
psychopy
```

## Issues

- Audio issues. Switching to PTB breaks PsychoPy.
- Psychtoolbox issues:
    Symbol not found: _AllocateHIDObjectFromIOHIDDeviceRef
- mp4 movies are not working.

If PsychoPy fails to start up after you have changed the preferences, remove the file `userPrefs.cfg`

```sh
rm ~/.psychopy3/userPrefs.cfg
```
