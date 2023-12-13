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

    for (NAIA::Event& event : chain){
      // your analysis here :)
    }

If you're uncomfortable with `range-based for loops <https://en.cppreference.com/w/cpp/language/range-for>`_ you can still do it the old fashioned way

.. code-block:: cpp

    unsigned long long nEntries = chain.GetEntries());

    for (unsigned long long iEv = 0; iEv < nEntries; iEv++) {
      Event &event = chain.GetEvent(iEv);
  
      // your analysis here :)
    }

NAIA root-files contain two more ``TTree`` with additional data for the analysis. 

The ``RTIInfo`` tree
^^^^^^^^^^^^^^^^^^^^

The data about the ISS position, its orientation, and physical quantities connected to them, as well as some time-averaged data about the run 
itself are usually retrieved in AMS analysis from the RTI (Real Time Information) database. This database stores data with a time granularity 
of one second, and it can be accessed using the gbatch library.

Since we try to get rid of any dependency on gbatch during the analysis the entire RTI database is converted to a ``TTree`` that is stored 
alongside the main event ``TTree`` in the NAIA root-files. This tree has only one branch, which contains objects of the ``RTIInfo`` 
`class <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/classNAIA_1_1RTIInfo.html>`_, one for each second of the current run.

When looping over the events you can get the ``RTIInfo`` object for the current event by calling

.. code-block:: cpp

  NAIA::RTIInfo &rti_info = chain.GetEventRTIInfo();

In some cases you might not want to loop over all the events, but still perform analysis on the RTI data standalone. In such cases you can
directly retrieve the RTI tree from the NAIA file and loop over each second.

.. code-block:: cpp

  TChain* rti_chain = chain.GetRTITree();
  NAIA::RTIInfo* rti_info = new NAIA::RTIInfo();
  rti_chain->SetBranchAddress("RTIInfo", &rti_info);

  for(unsigned long long isec=0; isec < rti_chain->GetEntries(); ++isec){
    rti_chain->GetEntry(isec);
    
    // your analysis here :)
  }

The ``FileInfo`` tree
^^^^^^^^^^^^^^^^^^^^^

In a similar fashion we also store some useful information about the original AMSRoot file that from which the current NAIA file was derived.
This information is stored in the FileInfo ``TTree``, which usually has only a single entry for each NAIA root-file. Having this data in a 
``TTree`` allows us to chain multiple NAIA root-files and still be able to retrieve the FileInfo data for the current run we're processing.

This tree has one branch, which contains objects of the ``FileInfo`` `class <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/classNAIA_1_1FileInfo.html>`_ 
and, if the NAIA root-file is a Montecarlo file, an additional branch containing objects of the 
``MCFileInfo`` `class <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/classNAIA_1_1MCFileInfo.html>`_.

When looping over the events you can get both objects for the current event by calling

.. code-block:: cpp

  NAIA::FileInfo &file_info = chain.GetEventFileInfo();
  NAIA::MCFileInfo &mcfile_info = chain.GetEventMCFileInfo();

Also in this case you can directly retrieve the FileInfo tree from the NAIA file and loop over each entry.

.. code-block:: cpp

  TChain* file_chain = chain.GetFileInfoTree();
  NAIA::FileInfo* file_info = new NAIA::FileInfo();
  NAIA::MCFileInfo* mcfile_info = new NAIA::MCFileInfo();

  file_chain->SetBranchAddress("FileInfo", &file_info);
  if(chain.IsMC()){
    file_chain->SetBranchAddress("MCFileInfo", &mcfile_info);
  }

  for(unsigned long long i=0; i < file_chain->GetEntries(); ++i){
    file_chain->GetEntry(i);

    // do stuff with file_info

    if(chain.IsMC()){
      // do stuff with mcfile_info
    }
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
        //                                  ^^
        //                            this is very important :)

Variable types and structure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most variables in AMS analysis are computed for several different variants, which usually refer to different 
possible reconstructions of the same quantity. To mantain the data format as light as possible, and not 
write to disk non-existing data, variables in NAIA are often implemented as associative containers 
(e.g: ``std::map``).

If that is the case, then there is always a ``enum`` describing all the available variants for a given variable.

If you want to make sure that a given variant exists you can use the ``ContainsKeys`` `function <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/group__contvar.html#gadbb95738c905854cc9e90e40f4789072>`_.
This function takes a container and one or more keys and will check recursively that those keys exist in the container structure.

.. code-block:: cpp

  if (NAIA::ContainsKeys(event.tofBase->Charge, NAIA::Tof::ChargeType::Upper))
    tof_charge = event.tofBase->Charge[NAIA::Tof::ChargeType::Upper];

because it is not guaranteed that, for example, a particular reconstruction succeeded, or that there is a hit on a given layer.

.. note:: 
  
  The ``KeyExists`` function is completely replaced by ``ContainsKeys``. It is still available for backward-compatibility but it is now deprecated
  and will be removed in a future release. A warning message will be printed (at most 10 times), advising to switch to ``ContainsKeys``.

As an example, what before would have been achieved with

.. code-block:: cpp

  if (KeyExists(layer, LayerCharge) && KeyExists(NAIA::Track::ChargeRecoType::YJ, LayerCharge.at(layer)) &&
    KeyExists(TrTrack::Side::X, LayerCharge.at(layer).at(NAIA::Track::ChargeRecoType::YJ)))

is now done by

.. code-block:: cpp

  if (ContainsKeys(LayerCharge, layer, NAIA::Track::ChargeRecoType::YJ, TrTrack::Side::X))


.. note::

  Not all variables are stored in associative containers, when we know that all possible variants of a variable will be present
  we use a ``std::vector`` instead.

In NAIA there are several variable archetype defined, so that it is clear which ``enum`` to use and what kind of variable 
variant is available. The archetypes in the NAIA data model are:
 
* ``LayerVariable``: one number for each layer (applies to Tracker, Tof, TRD, ...).

  * Uses the layer number ``(0, ..., N-1)`` for access
  * .. code-block:: cpp

      template<class T>
      using LayerVariable = std::map< unsigned int, T >
  * Example:

    .. code-block:: cpp

      unsigned int layer = 4; // layer 5
      if (NAIA::ContainsKeys(event.trTrackPlus->TrackFeetDistance, layer))
        track_distance_to_feet_l5 = event.trTrackPlus->TrackFeetDistance[layer];

* ``EcalEnergyVariable``: one number for each energy reconstruction type.

  * Uses the ``Ecal::EnergyRecoType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1Ecal.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using EcalEnergyVariable = std::vector< T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.ecalBase->Energy, NAIA::Ecal::EnergyType::EnergyD))
        ecal_energy_D = event.ecalBase->Energy[NAIA::Ecal::EnergyType::EnergyD];

* ``EcalLikelihoodVariable``: one number for each likelihood type.

  * Uses the ``Ecal::LikelihoodType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1Ecal.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using EcalLikelihoodVariable = std::vector< T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.ecalPlus->Likelihood, NAIA::Ecal::Likelihood::Integral))
        ecal_likelihood = event.ecalPlus->Likelihood[NAIA::Ecal::Likelihood::Integral];
 
* ``EcalBDTVariable``: one number for each BDT type.

  * Uses the ``Ecal::BDTType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1Ecal.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using EcalBDTVariable = std::vector< T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.ecalBase->BDT, NAIA::Ecal::BDTType::v7std))
        bdt = event.ecalBase->BDT[NAIA::Ecal::BDTType::v7std];
 
* ``RichBetaVariable``: one number for each RICH beta reconstruction type.

  * Uses the ``Rich::BetaType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1Rich.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using RichBetaVariable = std::map< Rich::BetaType, T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.richBase->GetBeta(), NAIA::Rich::BetaType::CIEMAT))
        rich_beta = event.richBase->GetBeta()[NAIA::Rich::BetaType::CIEMAT];
 
* ``TofChargeVariable``: one number for each kind of Tof charge.

  * Uses the ``Tof::ChargeType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1Tof.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TofChargeVariable = std::map< Tof::ChargeType, T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.tofBase->Charge, NAIA::Tof::ChargeType::Upper))
        tof_charge = event.tofBase->Charge[NAIA::Tof::ChargeType::Upper];

* ``TofBetaVariable``: one number for each Tof beta reconstruction type.

  * Uses the ``Tof::BetaType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1Tof.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TofBetaVariable = std::map< Tof::BetaType, T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.tofBase->Beta, NAIA::Tof::BetaType::BetaH))
        tof_beta = event.tofBase->Beta[NAIA::Tof::BetaType::BetaH];
 
* ``TofClusterTypeVariable``: one number for each Tof cluster type.

  * Uses the ``Tof::BetaClusterType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1Tof.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TofClusterTypeVariable = std::map< Tof::BetaClusterType, T >
  * Example:

    .. code-block:: cpp

      unsigned int layer = 0;
      if (NAIA::ContainsKeys(event.tofPlus->Nclusters, layer, NAIA::Tof::BetaClusterType::OnTime))
        ontime_clusters = event.tofPlus->NClusters[layer][NAIA::Tof::BetaClusterType::OnTime];
 
* ``TrdChargeVariable``: one number for each TRD charge reconstruction type.

  * Uses the ``TrdK::ChargeType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrdK.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrdChargeVariable = std::vector< T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.trdKBase->Charge, NAIA::TrdK::ChargeType::Total))
        trd_charge = event.trdKBase->Charge[NAIA::TrdK::ChargeType::Total];
 
* ``TrdLikelihoodVariable``: one number for each TRD likelihood type.

  * Uses the ``TrdK::LikelihoodType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrdK.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrdLikelihoodVariable = std::vector< T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.trdKBase->Likelihood, NAIA::TrdK::LikelihoodType::Electron))
        trd_like_e = event.trdKBase->Likelihood[NAIA::TrdK::LikelihoodType::Electron];
 
* ``TrdLikelihoodRVariable``: one number for each TRD likelihood ratio type.

  * Uses the ``TrdK::LikelihoodRType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrdK.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrdLikelihoodRVariable = std::vector< T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.trdKBase->LikelihoodRatio, NAIA::TrdK::LikelihoodRType::ep))
        trd_likeratio_ep = event.trdKBase->LikelihoodRatio[NAIA::TrdK::LikelihoodRType::ep];
 
* ``TrdOnTrackVariable``: one number for on-track / off-track TRD hits.

  * Uses the ``TrdK::QualType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrdK.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrdOnTrackVariable = std::vector< T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.trdKBase->NHits, NAIA::TrdK::QualType::OffTrack))
        offtrack_hits = event.trdKBase->NHits[NAIA::TrdK::QualType::OffTrack];
 
* ``TrackChargeVariable``: one number for each Tracker charge reconstruction type.

  * Uses the ``TrTrack::ChargeRecoType`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrackChargeVariable = std::map< TrTrack::ChargeRecoType, T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.trTrackBase->InnerCharge, NAIA::TrTrack::ChargeRecoType::YJ))
        trtrack_charge_inner = event.trtrackBase->InnerCharge[NAIA::TrTrack::ChargeRecoType::YJ];
 
* ``TrackFitVariable``: one number for each track fitting type, and for each track span type.

  * Uses the ``TrTrack::Fit`` and ``TrTrack::Span`` `enums <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

       template<class T>
       using TrackFitVariable = std::map< TrTrack::Fit, std::map< TrTrack::Span, T >>
  * .. note::

      For this kind of variable you can use ``TrTrackBase::FitIDEsists(TrTrack::Fit fit, TrTrack::Span span)`` to check if a given fit+span combination exists

  * Example:

    .. code-block:: cpp

      if (event.trTrackBase->FitIDExists(NAIA::TrTrack::Fit::Kalman, NAIA::TrTrack::Span::InnerL1))
        trtrack_rigidity_innerL1 = event.trtrackBase->Rigidity[NAIA::TrTrack::Fit::Kalman][NAIA::TrTrack::Span::InnerL1];
 
* ``TrackFitOnlyVariable``: one number for each Track fit type.

  * Uses the ``TrTrack::Fit`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrackFitOnlyVariable = std::map< TrTrack::Fit, T >
  * Example:

    .. code-block:: cpp

      unsigned int layer = 1; // exclude layer 2
      if (NAIA::ContainsKeys(event.trTrackPlus->PartialRigidity, layer, NAIA::TrTrack::Fit::Choutko))
        ontime_clusters = event.trTrackPlus->PartialRigidity[layer][NAIA::TrTrack::Fit::Choutko];
 
* ``TrackSideVariable``: one number for each Tracker side.

  * Uses the ``TrTrack::Side`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrackSideVariable = std::map< TrTrack::Side, T >
  * Example:

    .. code-block:: cpp

      if (NAIA::ContainsKeys(event.trTrackBase->TrTrackHitPos, layer, NAIA::TrTrack::Side::X))
        ontime_clusters = event.trTrackBase->TrTrackHitPos[layer][NAIA::TrTrack::Side::X];
 
* ``TrackFitPosVariable``: one number for each fixed z-position in the Tracker.

  * Uses the ``TrTrack::FitPositionHeight`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrackFitPosVariable = std::map< TrTrack::FitPositionHeight, T >
  * Example:

    .. code-block:: cpp

      auto fit = NAIA::TrTrack::Fit::Kalman;
      auto span = NAIA::TrTrack::Span::InnerL1;

      if (NAIA::ContainsKeys(event.trtrackBase->TrTrackFitPos, NAIA::FitPositionHeight::TofLayer0)){
        if (event.trTrackBase->FitIDExists(fit, span)){
          trtrack_position_at_upper_tof_x = event.trtrackBase->TrTrackFitPos[NAIA::FitPositionHeight::TofLayer0][fit][span][NAIA::TrTrack::Side::X];
        }
      }

* ``TrackDistanceVariable``: one number for each distance-from-the-track type.

  * Uses the ``TrTrack::DistanceFromTrack`` `enum <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/namespaceNAIA_1_1TrTrack.html>`_ for access
  * .. code-block:: cpp

      template<class T>
      using TrackDistanceVariable = std::map< TrTrack::DistanceFromTrack, T >
  * Example:

    .. code-block:: cpp

      unsigned int layer = 1; // layer 2
      if (NAIA::ContainsKeys(event.trTrackPlus->NClusters, layer, NAIA::TrTrack::DistanceFromTrack::Onecm, TrTrack::Side::X))
        track_clusters_within_onecm_x = event.trTrackPlus->NClusters[layer][NAIA::TrTrack::DistanceFromTrack::Onecm][TrTrack::Side::X]; 

* ``HitChargeVariable``: same as ``TrackChargeVariable``

Please refer to the `doxygen documentation <https://naia-docs.web.cern.ch/naia-docs/v1.0.2/annotated.html>`_ for all the details.
