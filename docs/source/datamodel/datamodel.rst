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

Variable types and structure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most variables in AMS analysis are computed for several different variants, which usually refer to different 
possible reconstructions of the same quantity. To mantain the data format as light as possible, and not 
write to disk non-existing data, variables in NAIA are often implemented as associative containers 
(e.g: ``std::map``).

If that is the case, then there is always a ``enum`` describing all the available variants for a given variable.

If you want to make sure that a given variant exists you can use the ``KeyExists`` function

.. code-block:: cpp

  if (KeyExists(Tof::ChargeType::Upper, event.tofBase->Charge))
    tofCharge = event.tofBase->Charge[Tof::ChargeType::Upper];

because it is not guaranteed that, for example, a particular reconstruction succeeded, or that there is a hit on a given layer.

.. note::
  Not all variables are stored in associative containers, when we know that all possible variants of a variable will be present
  we use a ``std::vector`` instead.

In NAIA there are several variable archetype defined, so that it is clear which ``enum`` to use and what kind of variable 
variant is available. The archetypes in the NAIA data model are:

* ``EcalEnergyVariable``: one number for each energy reconstruction type.

  * Uses the ``Ecal::EnergyRecoType`` enum for access
  * .. code-block:: cpp

      template<class T>
      using EcalEnergyVariable = std::vector< T >
 
* ``EcalLikelihoodVariable``: one number for each likelihood type.

  * Uses the ``Ecal::LikelihoodType`` enum for access
  * .. code-block:: cpp

      template<class T>
      using	EcalLikelihoodVariable = std::vector< T >
 
* ``EcalBDTVariable``: one number for each BDT type.

  * Uses the ``Ecal::BDTType`` enum for access
  * .. code-block:: cpp

      template<class T>
      using EcalBDTVariable = std::vector< T >
 
* ``RichBetaVariable``: one number for each RICH beta reconstruction type.

  * Uses the ``Rich::BetaType`` enum for access
  * .. code-block:: cpp

      template<class T >
      using RichBetaVariable = std::map< Rich::BetaType, T >
 
.. template<class T >
.. using 	TofChargeVariable = std::map< Tof::ChargeType, T >
 
.. template<class T >
.. using 	TofBetaVariable = std::map< Tof::BetaType, T >
 
.. template<class T >
.. using 	TofClusterTypeVariable = std::map< Tof::BetaClusterType, T >
 
.. template<class T >
.. using 	TrdChargeVariable = std::vector< T >
 
.. template<class T >
.. using 	TrdLikelihoodVariable = std::vector< T >
 
.. template<class T >
.. using 	TrdLikelihoodRVariable = std::vector< T >
 
.. template<class T >
.. using 	TrdOnTrackVariable = std::vector< T >
 
.. template<class T >
.. using 	TrackChargeVariable = std::map< TrTrack::ChargeRecoType, T >
 
.. template<class T >
.. using 	TrackFitVariable = std::map< TrTrack::Fit, std::map< TrTrack::Span, T >>
 
.. template<class T >
.. using 	TrackFitOnlyVariable = std::map< TrTrack::Fit, T >
 
.. template<class T >
.. using 	TrackFitPosVariable = std::map< TrTrack::FitPositionHeight, T >
 
.. template<class T >
.. using 	TrackSideVariable = std::map< TrTrack::Side, T >
 
.. template<class T >
.. using 	TrackDistanceVariable = std::map< TrTrack::DistanceFromTrack, T >
 
.. template<class T >
.. using 	HitChargeVariable = std::map< TrTrack::ChargeRecoType, T >
 
.. template<class T >
.. using 	LayerVariable = std::map< unsigned int, T >

Please refer to the doxygen documentation for all the details.