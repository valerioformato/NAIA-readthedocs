# Getting started

## Requirements

To use NAIA you'll need:

* A C++ compiler with full c++17 support (tested with gcc >= 12.1.0)
* CMake version >= 3.13
* A ROOT installation compiled with c++17 support (tested with ROOT >= 6.28/04)

Supported platforms: CentOS7 and RHEL9 derivatives (Alma9, Rocky Linux9, ...)

If you have access to cvmfs then you can find all the requirements in

```text
/cvmfs/ams.cern.ch/Offline/amsitaly/public/install/x86_64-centos7-gcc12.1/naia
/cvmfs/ams.cern.ch/Offline/amsitaly/public/install/x86_64-el9-gcc12.1/naia
```

and a `setenv` script is already provided with each NAIA version, e.g. for CentOS7

```text
/cvmfs/ams.cern.ch/Offline/amsitaly/public/install/x86_64-centos7-gcc12.1/naia/v1.1.0/setenvs/setenv_gcc6.28_cc7.sh
```

```{note}
For the ntuple production some additional requirements are needed:

* A gbatch installation compiled with
  * `export NOCXXSTD=1` (gbatch hardcodes `-std=c++11` in the Makefile... This variable prevents that)
  * `export GLIBCXX_USE_CXX11=1` (gbatch hardcodes the old gcc ABI in the Makefile... Most likely someone didn't know what he was doing)
  * Run `CPPFLAGS="-std=c++17" make lib` to build the gbatch library (if you don't want to hack the Makefile and change the C++ standard manually)
```

## Building and installing

Follow this simple procedure:

* Clone this repository
  * `git clone --recursive https://:@gitlab.cern.ch:8443/ams-italy/naia.git -b v1.1.0` (Kerberos)
  * `git clone --recursive ssh://git@gitlab.cern.ch:7999/ams-italy/naia.git -b v1.1.0` (SSH) 
  * `git clone --recursive https://gitlab.cern.ch/ams-italy/naia.git -b v1.1.0` (HTTPS) 

````{note}
  Starting with NAIA version 1.1.0 all external dependencies are handled as submodules rather than having cmake download them during the configuration step.
  This means that if you have already cloned the NAIA repository prior to version 1.1.0 and want to switch to it you need to initialize its submodules in 
  order to build the project

  ```bash
    git submodule init && git submodule update
  ```
  If you cloned it from scratch with the recursive flag, then all submodules will already be initialized.
````

* Create a build and install directory
  * e.g: `mkdir naia.build naia.install`
* Build the project
  * `cd naia.build` 
  * `cmake ../naia -DCMAKE_INSTALL_PREFIX=${your-install-path-here}` (for ntuple production add the `-DPRODUCTION_CODE=ON` arg)
  * `make all install`


## Using the project

To use the NAIA ntuples your project needs:

* the headers in `naia.install/include`
* the `naia.install/lib/libNAIAUtility.so` library
* the `naia.install/lib/libNAIAContainers.so` library
* the `naia.install/lib/libNAIAChain.so` library

The recommended way of using NAIA in your project is to use CMake and let it do all the heavy lifting for you.
NAIA targets are set up so that required includes and libraries are automatically passed to your targets. 
In your `CMakeLists.txt` you just need:

```cmake
  find_package(NAIA REQUIRED)
  
  set(SOURCES MyProgram.cpp)

  add_executable(MyProgram ${SOURCES})
  target_link_libraries(MyProgram NAIA::NAIAChain)
```

and you should be good to go.

Alternatively you can set up your own makefile and do all the work manually. For this see the examples provided in the 
NAIA repository. 

````{note}
  If you are using a pre-installed NAIA distribution (e.g. from cvmfs) you might have to export the ``ROOT_INCLUDE_PATH`` variable to 
  include the path of the NAIA headers.

  ```bash
    export ROOT_INCLUDE_PATH=${path-to-the-NAIA-install}/include:$ROOT_INCLUDE_PATH
  ```

  This is due to ROOT needing to parse the headers at runtime. ([see for example](https://root-forum.cern.ch/t/problem-with-dictionaries-in-root6/27244/7))
````

## Included facilities

These two libraries are automatically built with the project and included in the installation so that they could be used out-of-the-box

### fmt

See https://github.com/fmtlib/fmt

This is a library for text formatting that implements the [formatting specification introduced in the C++20 standard](https://en.cppreference.com/w/cpp/utility/format), 
the syntax is similar to the [python format() function](https://www.w3schools.com/python/ref_string_format.asp).
It's a header-only library that is always lighter and faster than using iostream ([example](https://github.com/fmtlib/fmt#speed-tests)).

```{note} 
It is incredibly useful and flexible once you get used to the syntax (and it's way better than littering your code with thousands of `<<`)
```

### spdlog

https://github.com/gabime/spdlog

This is a header-only library for asynchronous logging build on top of `fmt` which allows to quickly log messages from a program with different 
levels of depth, customization and filtering.

```{note} 
It can be useful saving you from several `if(DEBUG) std::cout << "debug statement" << std::endl;` :)
```

```{note}
For any question or in case you need help write to valerio.formato@cern.ch 
```