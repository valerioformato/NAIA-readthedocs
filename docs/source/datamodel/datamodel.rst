The NAIA data model
===================

The NAIA data model is vaguely inspired by gbatch. The first thing needed to access data is to create a ``NAIAChain`` object

.. code-block:: cpp

  // ...  
  #include "Chain/NAIAChain.h"
  
  int main(int argc, char const *argv[]) {
    NAIA::NAIAChain chain;
    chain.Add("somefile.root");
    chain.SetupBranches();
  }

the ``chain.SetupBranches()`` is mandatory (with some work it could be made automatic with the instantiation of a chain, 
but this might come in a future release) and takes care of setting up the whole "read-on-demand" mechanism.

Looping
-------------------

The chain contains all the events in the added runs, looping over events is particularly easy:

.. code-block:: cpp

    for(Event& event : chain){
      // your analysis here :)
    }

If you're uncomfortable with range-based for loops you can still do it the old fashioned way

.. code-block:: cpp

    unsigned long long nEntries = chain.GetEntries());

    for (unsigned long long iEv = 0; iEv < nEntries; iEv++) {
      Event &event = chain.GetEvent(iEv);
  
      // your analysis here :)
    }

Containers
----------

The main structure for holding data in the NAIA data model is the *Container*. Each container is associated to 
a single branch in the main ``TTree`` and allows for reading the corresponding branch data only when first 
accessed.

This means that if you never use a particular container in your analysis, you'll never read the corresponding
data from file

.. note::
    i.e.: ``TBranch::GetEntry`` will never be called unless actually needed

.. warning::
    In order for this to work in NAIA we overload the ``->`` operator to hide this "read-on-demand" behavior. It is
    required that you always use ``->`` to access the data members and methods of a container.

    Example:

    .. code-block:: cpp
        // Get the inner tracker charge from the "trTrackBase" container
        auto innerCharge = event.trTrackBase->Charge[TrTrack::ChargeRecoType::YJ];
