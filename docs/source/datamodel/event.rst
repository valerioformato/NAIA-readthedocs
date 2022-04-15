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
     - 
   * - `EventSummary <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1EventSummaryData.html>`_
     - evSummary
     - 
   * - `DAQ <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1DAQData.html>`_
     - daq
     - 
   * - `TofBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TofBaseData.html>`_
     - tofBase
     - 
   * - `TofPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TofPlusData.html>`_
     - tofPlus
     - 
   * - `TofBaseStandalone <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TofBaseData.html>`_
     - tofBaseSt
     - 
   * - `TofPlusStandalone <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TofPlusData.html>`_
     - tofPlusSt
     - 
   * - `EcalBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1EcalBaseData.html>`_
     - ecalBase
     - 
   * - `EcalPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1EcalPlus.html>`_
     - ecalPlus
     - 
   * - `TrTrackBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TrTrackBaseData.html>`_
     - trTrackBase
     - 
   * - `TrTrackPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TrTrackPlus.html>`_
     - trTrackPlus
     - 
   * - `TrdKBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TrdKBase.html>`_
     - trdKBase
     - 
   * - `TrdKBaseStandalone <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1TrdKBase.html>`_
     - trdKBaseSt
     - 
   * - `RichBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1RichBaseData.html>`_
     - richBase
     - 
   * - `RichPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1RichPlusData.html>`_
     - richPlus
     - 
   * - `UnbExtHitBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1UnbExtHitBaseData.html>`_
     - extHitBase
     - 
   * - `MCTruthBase <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1MCTruthBaseData.html>`_
     - mcTruthBase
     - 
   * - `MCTruthPlus <https://naia-docs.web.cern.ch/naia-docs/classNAIA_1_1MCTruthPlusData.html>`_
     - mcTruthPlus
     - 

This class acts as an interface to group and access containers with information from the various subdetectors. 
This should be provided by the chain class as a transient view of the event information.

This class implements a data structure where all dataobjects are branches of a single tree. 
However all read/write operations are handled by the single containers.

