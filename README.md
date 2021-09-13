# PsychoPy v2021.2.3 on M1 Mac (Apple Silicon)

The current version of [PsychoPy](https://www.psychopy.org) runs on M1 Macs through Rosetta 2. Below are instructions to install a native Apple Silicon (arm64) version of PsychoPy on an M1 Mac using `pip install psychopy`.

## 1. Install native Python for M1 Macs

Use [brew](https://brew.sh) to install latest version of Python for M1 Macs.

```sh
brew install python3
```

Python 3.9.7 (3 September 2021).

## 2. Install QT5

Install QT5 using brew. Requires venv with system-site-packages to work in Python.

```sh
brew install pyqt5
```

## 3. Create virtual environment with system-site-packages

```sh
python3 -m venv --system-site-packages ~/venv/psychopy
```

Activate the virtual environment.

```sh
source ~/venv/psychopy/bin/activate

# update pip setuptools wheel
``` sh
pip install --upgrade pip setuptools wheel
```

## 4. Install numpy

```sh
pip install numpy
```

## 5. Install scipy

```sh
brew install openblas
```

```sh
export OPENBLAS=$(brew --prefix openblas)
export CFLAGS="-falign-functions=8 ${CFLAGS}"
pip install Cython pybind11 pythran
pip install --no-use-pep517 scipy
```

## 6. Install pygame

```sh
brew install pkg-config
#brew install sdl sdl\_image sdl\_mixer sdl\_net sdl\_ttf smpeg
brew install sdl2 sdl2_image sdl2_mixer sdl2_net sdl2_ttf smpeg
brew install portmidi
```

```sh
pip install pygame
```

## 7. Install tables

```sh
brew install hdf5 c-blosc lzo bzip2
export HDF5_DIR=$(brew --prefix hdf5)
export BLOSC_DIR=$(brew --prefix c-blosc)
export LZO_DIR=$(brew --prefix lzo)
export BZIP2_DIR=$(brew --prefix bzip2)

pip install tables
```

Note: `opt` folder of homebrew for Apple Silicon is located in `/opt/homebrew/`. On Intel macs it is located in `/usr/local/`.
To find correct path use brew: `$(brew --prefix ....)`.

## 8. Install pyo

```sh
pip install pyo
```

Installation succesful. However, error in PsychoPy when using `pyo`!

```txt
Pyo error: Pyo built without Coreaudio support
```

Solution is to build `pyo` from source.

Uninstall `pyo`.

```sh
pip uninstall pyo
```

Install `pyo` library from source.

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

## 9. Install Psychtoolbox

- Download [psychtoolbox from pypi](https://pypi.org/project/psychtoolbox/3.0.17.8/). Download `psychtoolbox-3.0.17.8.zip`
- Download: [ptb-wheels source](https://github.com/aforren1/ptb-wheels)

From folder `ptb-wheels-master`:

- Copy file `ptb-wheels-master/psychtoolbox-3/PsychSourceGL/Cohorts/PortAudio/libportaudio_osx_64.a` to `psychtoolbox-3.0.17.8/PsychSourceGL/Cohorts/PortAudio/`
- Copy file `ptb-wheels-master/psychtoolbox-3/PsychSourceGL/HID_Utilities_64Bit/build/Release/libHID_Utilities64.a` to `psychtoolbox-3.0.17.8/PsychSourceGL/Cohorts/HID_Utilities_64Bit/build/Release/`

In folder `psychtoolbox-3.0.17.8` type:

```sh
python setup.py install

# check if psychtoolbox is successfully installed
pip list
```

However, importing `psychtoolbox` in Python fails at the moment due to ImportError (`from .PsychHID import PsychHID` fails).

## 10. Install wxPython

```sh
export CFLAGS=-I/$(brew --prefix)/include
pip install wxPython
```

## 11. Install ffmpeg

```sh
## needed for mp4 videos
brew install ffmpeg
```

## 12. Install other libraries (not in requirements.txt but needed for some demos)

```sh
pip install SpeechRecognition pyfilesec
```

## 13. Install Psychopy

Installing PsychoPy and remaining dependencies.

```sh
pip install psychopy

# start psychopy
psychopy
```

PsychoPy starts up and some things work (see below for issues). One of the issues is that `core.wait()` is not working properly. To fix this download the [PsychoPy source](https://github.com/psychopy/psychopy).

Next move to source folder and uninstall psychopy.

```sh
# move to source folder
cd psychopy-release

# uninstall psychopy
pip uninstall psychopy
```

Issue is the definition of `getTime()` in `clock.py`, which doesn't work properly on Apple Silicon.
Because `psychtoolbox` import fails, `getTime()` is defined based on the `darwin` specific code in `clock.py`.

Output `clocksAndTimers.py` demo:

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

`_timebase.numer` and `_timebase.denom` are 1 on Intel but on Apple Silicon these values are not 1. `_timebase.numer / _timebase.denom` indicates the scaling factor. More info [here](https://eclecticlight.co/2020/11/27/inside-m1-macs-time-and-logs/).

The follow change fixes `getTime()` (change lines 91-94):

```python
# scaling factor
_scaling_factor = _timebase.numer / _timebase.denom

def getTime():
    return (_mach_absolute_time() * _scaling_factor) / 1.0e9
```

Also change lines 56-60 to:

```python
import platform
if platform.processor() == 'arm' and sys.platform == 'darwin':
    havePTB = False
else:
    try:
        import psychtoolbox
        havePTB = True
    except ImportError:
        havePTB = False
```

The above code prevents that PTB (`psychtoolbox`) is used for timing on arm processors.

After making the above changes in `clock.py`, install psychopy again.

```sh
python setup.py install
```

The `Benchmark wizard` now works (`Report: All values seem reasonable (no warnings, but there might still be room for improvement)`) as well as `timeByFramesEx.py`.

Correct output `clocksAndTimers.py` demo:

```txt
down       up          clock
3.0000   -3.0000   0.0000
2.8000   -2.8000   0.2000
2.5999   -2.5999   0.4001
2.3999   -2.3999   0.6001
2.1999   -2.1999   0.8001
1.9999   -1.9999   1.0001
1.7999   -1.7999   1.2001
1.5998   -1.5998   1.4002
1.3998   -1.3998   1.6002
1.1998   -1.1998   1.8002
0.9998   -0.9998   2.0002
0.7998   -0.7998   2.2002
0.5998   -0.5998   2.4003
0.3997   -0.3997   2.6003
0.1997   -0.1997   2.8003

down          clock
2.9997   3.0003
2.7997   3.2003
2.5997   3.4003
2.3997   3.6003
2.1997   3.8003
1.9997   4.0003
1.7997   4.2003
1.5997   4.4003
1.3997   4.6003
1.1997   4.8003
0.9997   5.0003
0.7997   5.2003
0.5997   5.4003
0.3997   5.6003
0.1997   5.8003
The last run should have been precise sub-millisecond
```

## Other issues with fixes

- Audio issues

    Default audio library is `sounddevice`. Switching audio library in PsychoPy Preferences to `PTB` (Psychtoolbox) breaks PsychoPy. Switching to `pyo` works!

- Audio capture not working

    The 'psychtoolbox' library cannot be loaded but is required for audio capture.

- Psychtoolbox is not imported

  Import Error: dlopen(/Users/waltervh/venv/psychopy/lib/python3.9/site-packages/psychtoolbox-3.0.17.11-py3.9-macosx-11-arm64.egg/psychtoolbox/PsychHID.cpython-39-darwin.so, 2). `Symbol not found: _AllocateHIDObjectFromIOHIDDeviceRef`.

- `System info...` in Help menu fails

  Error is due to a bug in psutil: [psutil.cpu_freq() broken on Apple M1](https://github.com/giampaolo/psutil/issues/1892).

- Drivers for Apple Silicon Macs

  - USB driver for Cedrus response box/Stimtracker (ARM) [download D2XX direct driver](https://ftdichip.com/drivers/d2xx-drivers/)

If PsychoPy fails to start up after you have changed the preferences, remove the file `userPrefs.cfg`

```sh
rm ~/.psychopy3/userPrefs.cfg
```

## Status

After fixing `getTime()` and installing `pyo` from source (with fix), playing movies and audio works! Tested several PsychoPy experiments, and so far things seem to work.
