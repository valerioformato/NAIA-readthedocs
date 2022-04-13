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

The ``Event`` class
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



