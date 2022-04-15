Examples
========

NAIA ships with a few examples to help you getting started

All examples show the steps needed to compile/run a simple executable that loops over a ``NAIAChain``

CMake
^^^^^

.. warning::

    **This is the recommended way to go**

To compile just run
- ``mkdir build``
- ``cd build``
- ``cmake .. -DNAIA_DIR=/path/to/your/naia/install/cmake``
- ``make``

If you take a look at the included ``CMakeLists.txt`` you'll notice that the only two lines needed to link against the NAIA libraries are:

.. code-block:: cmake

    find_package(NAIA REQUIRED)
    # ...
    target_link_libraries(main PUBLIC NAIA::NAIAChain)

this is because NAIA internally defines everything that is needed in terms of targets. 
The ``NAIA::NAIAChain`` target internally knows all the include paths, preprocessor macros, library paths, libraries 
that it needs so that CMake can propagate these requirements to all targets linking against ``NAIA::NAIAChain``.

.. note::

    **This also means that if in the future these requirements will change you don't have to adapt the build of your project.**

Makefile
^^^^^^^^

To compile you need to update the ``NAIA_DIR`` variable inside the ``Makefile`` and then you can just call ``make``. 
Remember to add include paths/libraries if needed or if something changes in the NAIA project.

ROOT macros
^^^^^^^^^^^

Before running the examples as a ROOT macros you need to either load the ``load.C`` macro beforehand

.. code-block:: 

    root load.C main.cpp

or add the content of ``load.C`` to your ``.rootlogon.C``.

.. note::

    Also in this case you have to update the value of ``naia_dir`` inside of ``load.C`` and keep track of changes 
    in the upstream NAIA project. 

RDataFrame
^^^^^^^^^^

.. note::

    **NB: this mode is not particularly tested, and usage of containers is slightly different**

    However, it is extremely cool

This example shows how to plot one histogram on NAIA events applying some simple selections, using the ``RDataFrame`` approach. 
There are a few caveats when using this approach:

* You don't use ``NAIAChain``, instead you have to create the ``RDataFrame`` object reading the original tree from file, 
or creating a traditional ``TChain``. How this ties with the ``RTIInfo`` and ``FileInfo`` trees is to be investigated.
* You have to work with the "Data" container classes, without the "read-on-demand" part. ``RDataFrame`` is supposed to take 
care of the rest by itself.
* You have to use the correct branch name in all the operations, which should be the same as the corresponding container 
"Data" class.

Simple macro
^^^^^^^^^^^^

.. note::

    **NB: this mode is not particularly tested, and generally discouraged**

This example shows how to loop on a ``NAIAChain`` from a root macro. It is identical to the simple CMake and Makefile examples.