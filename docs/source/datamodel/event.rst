The ``Event`` class
===================

The ``Event`` class is nothing more than a collection of containers

.. list-table:: Event class layout
   :widths: 25 25 50
   :header-rows: 1

   * - Container type
     - Name
     - Description
   * - `Header <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1HeaderData.html>`_
     - header
     - Contains simple information like run number, run tag, event number and UTC time.
   * - `EventSummary <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1EventSummaryData.html>`_
     - evSummary
     - Contains some aggregated variables to roughly describe the event. 
   * - `DAQ <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1DAQData.html>`_
     - daq
     - Contains variables describing the status of the AMS DAQ system for the event.
   * - `TofBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TofBaseData.html>`_
     - tofBase
     - Contains basic Tof variables that are accessed most frequently
   * - `TofPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TofPlusData.html>`_
     - tofPlus
     - Contains additional Tof variables that are accessed less frequently
   * - `TofBaseStandalone <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TofBaseData.html>`_
     - tofBaseSt
     - Contains basic Tof variables that are accessed most frequently (no-tracker reconstruction)
   * - `TofPlusStandalone <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TofPlusData.html>`_
     - tofPlusSt
     - Contains additional Tof variables that are accessed less frequently (no-tracker reconstruction)
   * - `EcalBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1EcalBaseData.html>`_
     - ecalBase
     - Contains basic Ecal variables that are accessed most frequently
   * - `EcalPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1EcalPlus.html>`_
     - ecalPlus
     - Contains additional Ecal variables that are accessed less frequently
   * - `TrTrackBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TrTrackBaseData.html>`_
     - trTrackBase
     - Contains basic Track variables that are accessed most frequently
   * - `TrTrackPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TrTrackPlus.html>`_
     - trTrackPlus
     - Contains additional Track variables that are accessed less frequently
   * - `TrdKBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TrdKBase.html>`_
     - trdKBase
     - Contains basic TRD variables that are accessed most frequently
   * - `TrdKBaseStandalone <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TrdKBase.html>`_
     - trdKBaseSt
     - Contains additional TRD variables that are accessed less frequently
   * - `RichBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1RichBaseData.html>`_
     - richBase
     - Contains basic RICH variables that are accessed most frequently
   * - `RichPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1RichPlusData.html>`_
     - richPlus
     - Contains additional RICH variables that are accessed less frequently
   * - `UnbExtHitBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1UnbExtHitBaseData.html>`_
     - extHitBase
     - Contains basic variables for unbiased external hits
   * - `MCTruthBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1MCTruthBaseData.html>`_
     - mcTruthBase
     - Contains basic MC truth variables that are accessed most frequently
   * - `MCTruthPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1MCTruthPlusData.html>`_
     - mcTruthPlus
     - Contains additional MC truth variables that are accessed less frequently

The `Event` class acts as an interface to group and access containers with information from the various subdetectors. 
This should be provided by the chain class as a transient view of the event information.

.. note::
    Containers are actually made up from two classes. The first one is the one holding all the
    variables, while the second one adds the "read-on-demand" behavior to the container.
    When navigating the `doxygen documentation <https://naia-docs.web.cern.ch/naia-docs/annotated.html>`_ 
    remember to go check the `XXXData` class, where "XXX" is the container Class, and you'll find
    the description for all the container variables.


The event `Category`
^^^^^^^^^^^^^^^^^^^^