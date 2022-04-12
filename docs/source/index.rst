NAIA (Ntuple for AMS-Italy Analysis)
===================================

This is the official documentation page for the **NAIA** project. The project
focuses on providing a common data format for AMS analysis that can be shared by
multiple groups.

Requirements
============
To use NAIA you'll need:
- A C++ compiler with full c++14 support (tested with gcc >= 7.3.0)
- CMake version >= 3.13
- A ROOT installation compiled with c++14 support (tested with ROOT >= 6.18/04)

If you have access to cvmfs then you can find all the requirements in 
```
/cvmfs/ams.cern.ch/Offline/amsitaly/public/install/x86_64-centos7-gcc9.3/naia
```
and a `setenv` script is already provided with each NAIA version, e.g. for CentOS7:
```
/cvmfs/ams.cern.ch/Offline/amsitaly/public/install/x86_64-centos7-gcc9.3/naia/v0.1.0/setenvs/setenv_gcc6.22_cc7.sh
```

For the ntuple production some additional requirements are needed:
* A gbatch installation compiled with
    * `export NOCXXSTD=1` (gbatch hardcodes `-std=c++11` in the Makefile... This variable prevents that)
    * `export GLIBCXX_USE_CXX11=1` (gbatch hardcodes the old gcc ABI in the Makefile... Most likely someone didn't know what he was doing)
    * Run `CPPFLAGS="-std=c++14" make lib` to build the gbatch library (if you don't want to hack the Makefile and change the C++ standard manually)


Building and installing
=======================

Follow this simple procedure
* Clone this repository
    * (Kerberos) `git clone https://:@gitlab.cern.ch:8443/ams-italy/naia.git`
    * (SSH) `git clone ssh://git@gitlab.cern.ch:7999/ams-italy/naia.git`
    * (HTTPS) `git clone https://gitlab.cern.ch/ams-italy/naia.git`
* Create a build and install directory
    * e.g: `mkdir naia.build naia.install`
* Build the project
    * `cd naia.build` 
    * `cmake ../naia` (for ntuple production add the `-DPRODUCTION_CODE=ON` arg)
    * `make all install`

.. **Lumache** (/lu'make/) is a Python library for cooks and food lovers
.. that creates recipes mixing random ingredients.
.. It pulls data from the `Open Food Facts database <https://world.openfoodfacts.org/>`_
.. and offers a *simple* and *intuitive* API.

.. Check out the :doc:`usage` section for further information, including
.. how to :ref:`installation` the project.

.. note::

   This project is under active development.

Contents
--------

.. toctree::

   usage
   api
