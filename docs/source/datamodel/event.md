# The `Event` class

The `Event` class is nothing more than a collection of containers, which try to group variables together 
according to specific criteria. 

Although in NAIA there is no event pre-selection of any kind, there are still plenty of choices that were made
when deciding how to get or compute all the variables in the datamodel. For example, to get a rigidity value 
you first need to decide from which reconstructed track, and similar arguments apply to ToF, RICH, ECAL, and so on...

The main containers are derived from what gbatch thinks is the best association between all the subdetectors
signals in the event. The result of these associations is called "particle" in gbatch terminology, and the
first one (called "particle 0", for obvious reasons) is the the most likely to represent the main particle that
came through AMS.

Some additional containers are either alternative reconstruction of the subdetector signals used by the 
particle 0, or they represent different objects altogether. See the following table for details about
each container. 

| Container type | Name             | Description |
|----------------|------------------|-------------|
| [Header](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1HeaderData.html) | header | Contains simple information like run number, run tag, event number, event mask and UTC time. Lightweight and meant to be used as a tool to quickly identify or select an event. |
| [EventSummary](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1EventSummaryData.html) | evSummary | Contains some aggregated variables to roughly describe the event. |
| [DAQ](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1DAQData.html) | daq | Contains variables describing the status of the AMS DAQ system for the event. |
| [TofBase](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TofBaseData.html) | tofBase | Contains basic Tof variables that are accessed most frequently. Constructed from the particle 0 Tof objects. |
| [TofPlus](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TofPlusData.html) | tofPlus | Contains additional Tof variables that are accessed less frequently. Constructed from the particle 0 Tof objects. |
| [TofBaseStandalone](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TofBaseData.html) | tofBaseSt | Contains basic Tof variables that are accessed most frequently. Constructed from the particle 0 Tof objects, but this reconstruction avoids using any information from the Tracker track. Meant to be used for the Track reconstruction efficiency evaluation. |
| [TofPlusStandalone](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TofPlusData.html) | tofPlusSt | Contains additional Tof variables that are accessed less frequently. Constructed from the particle 0 Tof objects, but this reconstruction avoids using any information from the Tracker track. Meant to be used for the Track reconstruction efficiency evaluation. |
| [EcalBase](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1EcalBaseData.html) | ecalBase | Contains basic Ecal variables that are accessed most frequently. Constructed from the particle 0 ECAL shower. |
| [EcalPlus](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1EcalPlus.html) | ecalPlus | Contains additional Ecal variables that are accessed less frequently. Constructed from the particle 0 ECAL shower. |
| [TrTrackBase](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TrTrackBaseData.html) | trTrackBase | Contains basic Track variables that are accessed most frequently. Constructed from the particle 0 Tracker track. |
| [TrTrackPlus](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TrTrackPlus.html) | trTrackPlus | Contains additional Track variables that are accessed less frequently. Constructed from the particle 0 Tracker track. |
| [SecondTrTrackBase](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TrTrackBaseData.html) | secondTrTrackBase | Contains basic Track variables (a subset of all the variables contained in TrTrackBase) for the "second track". This track is selected by looking for the highest rigidity track (besides the primary one) with at least 5 hits. |
| [TrTrackBaseStandalone](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TrTrackBaseData.html) | trTrackBaseSt | Contains basic Track variables reconstructed without using any Tof information. Constructed from the particle 0 track, but this reconstruction avoids using any information from the Tof. Meant to be used for the Tof reconstruction efficiency evaluation. |
| [TrdKBase](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TrdKBase.html) | trdKBase | Contains basic TRD variables that are accessed most frequently. Constructed using the TrdK method in gbatch, using the Tracker track to select TRD hits. |
| [TrdKBaseStandalone](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1TrdKBase.html) | trdKBaseSt | Contains basic TRD variables that are accessed most frequently. Constructed using the TrdK method in gbatch, using the Tof extrapolation to select TRD hits. |
| [RichBase](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1RichBaseData.html) | richBase | Contains basic RICH variables that are accessed most frequently. Constructed using the particle 0 RICH ring. |
| [RichPlus](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1RichPlusData.html) | richPlus | Contains additional RICH variables that are accessed less frequently. Constructed using the particle 0 RICH ring. |
| [UnbExtHitBase](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1UnbExtHitBaseData.html) | extHitBase | Contains basic variables for unbiased external hits. Constructed using Tof and TRD standalone information. If the event charge estimated by the Tof is greater than 1, then the highest charge hits are selected on both L1 and L9. Otherwise, charge 1 hits closest to the standalone TRD (or Tof, if TRD is not available) extrapolation on L1/L9 are selected. |
| [MCTruthBase](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1MCTruthBaseData.html) | mcTruthBase | Contains basic MC truth variables that are accessed most frequently. |
| [MCTruthPlus](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1MCTruthPlusData.html) | mcTruthPlus | Contains additional MC truth variables that are accessed less frequently. |

The `Event` class acts as an interface to group and access containers with information from the various subdetectors. 
This should be provided by the chain class as a transient view of the event information.

```{note}
Containers are actually made up from two classes. The first one is the one holding all the
variables, while the second one adds the "read-on-demand" behavior to the container.

When navigating the [doxygen documentation](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/annotated.html) 
remember to go check the `XXXData` class, where "XXX" is the container Class, and you'll find
the description for all the container variables.
```

## The event `Category`

To avoid going through every event every single time you can perform a fast event filtering by looking at the event 
[`Category` mask](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/group__contvar.html#ga3961a0a915ed84b69299200e51bd6242) 
in the `Header` container.

````{note}
To check if an event belongs in a given set of categories you can use the [`Header::CheckMask` method](https://naia-docs.web.cern.ch/naia-docs/v1.1.0/classNAIA_1_1HeaderData.html#a2b7f7c8fac62c48b1b71d71e88125989). 

Categories can be combined into a single mask, to check many of them at once

```cpp        
// this mask will check for charge=1 according to both tracker and tof
NAIA::Category cat = NAIA::Category::Charge1_Trk | NAIA::Category::Charge1_Tof;

for (NAIA::Event &event : chain) {
  // check charge with TOF and Tracker
  if (!event.header->CheckMask(cat))
    continue;
```
    
`CheckMask` will check that *all* categories are present in the event. If you want to perform the check in **or** rather
than **and** you can use the `MathAnyBit` free function

```cpp
// this mask will check for charge=1 according to both tracker or tof
NAIA::Category cat = NAIA::Category::Charge1_Trk | NAIA::Category::Charge1_Tof;

for (NAIA::Event &event : chain) {
  // check charge with TOF or Tracker
  if (!NAIA::MatchAnyBit(event.header->Mask(), cat))
    continue;
```

(n.b: the `CheckMask` method uses the `MatchAllBits` free function)
````