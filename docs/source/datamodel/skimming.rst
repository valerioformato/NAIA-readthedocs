Skimming
========

Ntuple skimming is very easy to do in NAIA. You can setup a new file for your skimmed ntuples by calling
``NAIAChain::CreateSkimTree`` and specifying the name of the output root file 
(`see doxygen <https://naia-docs.web.cern.ch/naia-docs/v1.0.1/classNAIA_1_1NAIAChain.html#aeca79ddd1a0f42ede2f82642087d91ed>`_). 

``CreateSkimTree`` will return a ``SkimTreeHandle`` `object <https://naia-docs.web.cern.ch/naia-docs/v1.0.1/classNAIA_1_1SkimTreeHandle.html>`_. 
As the name implies the ``SkimTreeHandle`` is a handle to the new tree, you can call ``Fill`` on it to save a given event
in the new tree.

When you're done processing you can call ``Write`` on the ``SkimTreeHandle`` to write the resulting ``TTree`` on the
output root file.

.. note::
    If you don't want to write out some containers (in case you know you won't need them and want to save some space)
    you can pass a semicolon-separated list of containers to exclude as the second argument of ``CreateSkimHandle``. 