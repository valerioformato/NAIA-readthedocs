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
        auto innerCharge = event.trTrackBase->Charge[NAIA::TrTrack::ChargeRecoType::YJ];

Variable types and structure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most variables in AMS analysis are computed for several different variants, which usually refer to different 
possible reconstructions of the same quantity. To mantain the data format as light as possible, and not 
write to disk non-existing data, variables in NAIA are often implemented as associative containers 
(e.g: ``std::map``).

If that is the case, then there is always a ``enum`` describing all the available variants for a given variable.

If you want to make sure that a given variant exists you can use the ``KeyExists`` function

.. code-block:: cpp

  if (NAIA::KeyExists(NAIA::Tof::ChargeType::Upper, event.tofBase->Charge))
    tofCharge = event.tofBase->Charge[NAIA::Tof::ChargeType::Upper];

because it is not guaranteed that, for example, a particular reconstruction succeeded, or that there is a hit on a given layer.

.. note::
  Not all variables are stored in associative containers, when we know that all possible variants of a variable will be present
  we use a ``std::vector`` instead.

In NAIA there are several variable archetype defined, so that it is clear which ``enum`` to use and what kind of variable 
variant is available. The archetypes in the NAIA data model are:

* ``EcalEnergyVariable``: one number for each energy reconstruction type.

  * Uses the ``Ecal::EnergyRecoType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1Ecal.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using EcalEnergyVariable = std::vector< T >
 
* ``EcalLikelihoodVariable``: one number for each likelihood type.

  * Uses the ``Ecal::LikelihoodType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1Ecal.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using	EcalLikelihoodVariable = std::vector< T >
 
* ``EcalBDTVariable``: one number for each BDT type.

  * Uses the ``Ecal::BDTType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1Ecal.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using EcalBDTVariable = std::vector< T >
 
* ``RichBetaVariable``: one number for each RICH beta reconstruction type.

  * Uses the ``Rich::BetaType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1Rich.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using RichBetaVariable = std::map< Rich::BetaType, T >
 
* ``TofChargeVariable``: one number for each kind of Tof charge.

  * Uses the ``Tof::ChargeType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1Tof.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using	TofChargeVariable = std::map< Tof::ChargeType, T >
 
* ``TofBetaVariable``: one number for each Tof beta reconstruction type.

  * Uses the ``Tof::BetaType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1Tof.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using	TofBetaVariable = std::map< Tof::BetaType, T >
 
* ``TofClusterTypeVariable``: one number for each Tof cluster type.

  * Uses the ``Tof::BetaClusterType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1Tof.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using	TofClusterTypeVariable = std::map< Tof::BetaClusterType, T >
 
* ``TrdChargeVariable``: one number for each TRD charge reconstruction type.

  * Uses the ``TrdK::ChargeType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrdK.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrdChargeVariable = std::vector< T >
 
* ``TrdLikelihoodVariable``: one number for each TRD likelihood type.

  * Uses the ``TrdK::LikelihoodType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrdK.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using	TrdLikelihoodVariable = std::vector< T >
 
* ``TrdLikelihoodRVariable``: one number for each TRD likelihood ratio type.

  * Uses the ``TrdK::LikelihoodRType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrdK.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrdLikelihoodRVariable = std::vector< T >
 
* ``TrdOnTrackVariable``: one number for on-track / off-track TRD hits.

  * Uses the ``TrdK::QualType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrdK.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrdOnTrackVariable = std::vector< T >
 
* ``TrackChargeVariable``: one number for each Tracker charge reconstruction type.

  * Uses the ``TrTrack::ChargeRecoType`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using	TrackChargeVariable = std::map< TrTrack::ChargeRecoType, T >
 
* ``TrackFitVariable``: one number for each track fitting type, and for each track span type.

  * Uses the ``TrTrack::Fit`` and ``TrTrack::Span`` `enums <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

       template<class T>
       using TrackFitVariable = std::map< TrTrack::Fit, std::map< TrTrack::Span, T >>
 
* ``TrackFitOnlyVariable``: one number for each Track fit type.

  * Uses the ``TrTrack::Fit`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrackFitOnlyVariable = std::map< TrTrack::Fit, T >
 
* ``TrackFitPosVariable``: one number for each fixed z-position in the Tracker.

  * Uses the ``TrTrack::FitPositionHeight`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrackFitPosVariable = std::map< TrTrack::FitPositionHeight, T >
 
* ``TrackSideVariable``: one number for each Tracker side.

  * Uses the ``TrTrack::Side`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrackSideVariable = std::map< TrTrack::Side, T >
 
* ``TrackDistanceVariable``: one number for each distance-from-the-track type.

  * Uses the ``TrTrack::DistanceFromTrack`` `enum <https://naia-docs.web.cern.ch/naia-docs/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using 	TrackDistanceVariable = std::map< TrTrack::DistanceFromTrack, T >
 
* ``HitChargeVariable``: same as ``TrackChargeVariable``
 
* ``LayerVariable``: one number for each layer (applies to Tracker, Tof, TRD, ...).

  * Uses the layer number ``(0, ..., N-1)`` for access
  * .. code-block:: cpp

      template<class T>
      using LayerVariable = std::map< unsigned int, T >

Please refer to the doxygen documentation for all the details.