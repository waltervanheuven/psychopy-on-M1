# PsychoPy v2021.2.3 on M1 Mac (Apple Silicon)

The current version of [PsychoPy](https://www.psychopy.org) runs on M1 Macs through Rosetta 2. Below are instructions to install an Apple Silicon native version of PsychoPy on an M1 Mac using `pip install psychopy`.

## Install Python for M1 Macs

Use [brew](https://brew.sh) to install latest version of Python for M1 Macs.

```sh
brew install python3
```

Python 3.9.7 (3 September 2021).

## Install QT5

Install QT5 using brew. Requires venv with system-site-packages to work in Python.

```sh
brew install pyqt5
```

## Create virtual environment with system-site-packages

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
brew install openblas
```

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

After fixing the missing header file install pyo again.

```sh
python setup.py install --use-coreaudio --use-double
```

## Install Psychtoolbox

- Download [psychtoolbox from pypi] (<https://pypi.org/project/psychtoolbox/3.0.17.8/>). Download `psychtoolbox-3.0.17.8.zip`
- Download: [ptb-wheels source] (<https://github.com/aforren1/ptb-wheels>)

From folder `ptb-wheels-master`:

- Copy file `ptb-wheels-master/psychtoolbox-3/PsychSourceGL/Cohorts/PortAudio/libportaudio_osx_64.a` to `psychtoolbox-3.0.17.8/PsychSourceGL/Cohorts/PortAudio/`
- Copy file `ptb-wheels-master/psychtoolbox-3/PsychSourceGL/HID_Utilities_64Bit/build/Release/libHID_Utilities64.a` to `psychtoolbox-3.0.17.8/PsychSourceGL/Cohorts/HID_Utilities_64Bit/build/Release/`

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

## Install other libraries (not in requirements.txt but needed for some demos)

```sh
pip install SpeechRecognition pyfilesec
```

## Install Psychopy

Installing PsychoPy and other dependencies.

```sh
pip install psychopy

# start psychopy
psychopy
```

PsychoPy starts up and some things work (see below for issues). One of the issues is that `core.wait()` is not working. To fix this download the [PsychoPy source](https://github.com/psychopy/psychopy).

```sh
# move to source folder
cd psychopy-release

# uninstall psychopy
pip uninstall psychopy
```

There are issues with pyglet in the function `core.wait()`. A hack that works is to set the hogCPUperoid to zero.

In the file `build/lib/psychopy/clock.py` add a line to function `wait` (function starts at line 291):

```python
    from . import core

    hogCPUperiod = 0
```

Install psychopy

```sh
python setup.py install
```

## Issues

- Audio issues. Default audio library is `sounddevice`. Switching audio library in PsychoPy Preferences to `PTB` (Psychtoolbox) breaks PsychoPy. Switching to `pyo` is the solution. Audio of mp4 movies are working.

- Psychtoolbox issues:
    `Symbol not found: _AllocateHIDObjectFromIOHIDDeviceRef`

    Are issues with PTB due to PortAudio and libHID_Utilities static libraries which require a rebuild for Apple Silicon?

- Playing mp4 movies is not working.

- `clocksAndTimers.py` not working as expected.

    Output:

    ```txt
    down       up          clock
    3.0000   -3.0000   0.0000
    2.9999   -2.9999   0.0001
    2.9998   -2.9998   0.0002
    2.9996   -2.9996   0.0004
    2.9995   -2.9995   0.0005
    2.9994   -2.9994   0.0006
    2.9993   -2.9993   0.0007
    2.9992   -2.9992   0.0008
    2.9991   -2.9991   0.0009
    2.9989   -2.9989   0.0011
    2.9988   -2.9988   0.0012
    ```

    Should be: (output PsychoPy running through Rosetta)

    ```txt
    down       up          clock
    3.0000   -3.0000   0.0000
    2.7999   -2.7999   0.2001
    2.5998   -2.5998   0.4002
    2.3998   -2.3998   0.6002
    2.1998   -2.1997   0.8003
    1.9997   -1.9997   1.0003
    1.7996   -1.7996   1.2004
    1.5996   -1.5996   1.4004
    1.3995   -1.3995   1.6005
    1.1995   -1.1995   1.8005
    0.9994   -0.9994   2.0006
    0.7994   -0.7994   2.2006
    0.5993   -0.5993   2.4007
    0.3993   -0.3993   2.6007
    0.1993   -0.1993   2.8007
    ```

    Issue is definition of `getTime()` in `clock.py` for macOS. Requires rewrite based on this [code](https://github.com/aforren1/toon/tree/master/toon/util).

If PsychoPy fails to start up after you have changed the preferences, remove the file `userPrefs.cfg`

```sh
rm ~/.psychopy3/userPrefs.cfg
```
