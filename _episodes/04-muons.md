---
title: "Muons"
teaching: 10
exercises: 30
questions:
- "How are muons reconstructed in CMS?"
- "How are muons treated in CMS OpenData?"
- "How do I add variables to the muon analyzer?"
objectives:
- "Understand how muons are reconstructed in CMS"
- "Learn member functions for muon track-based quantities"
- "Learn member functions for identification and isolation of muons"
- "Learn how to add new variables to the muon analyzer"
keypoints:
- "Track access may differ, but track-related member functions are common across objects."
- "Physics objects in CMS are reconstructed from detector signals and are never 100% certain!"
- "Muons typically use pre-configured identification and isolation variable member functions."
- "Sometime it is necessary to be able to understand the code to add new variables"
---

> ## Prerequisites
>
> * We will be still running on the `CMSSW` Docker container.  If you closed it for some reason, just fire it back up.
> * During the last episode we made modifications to a few files.  If you fell behind, worry not, you can get the modifications with the files below:
>   * Download [this](https://raw.githubusercontent.com/cms-opendata-analyses/PhysObjectExtractorTool/odws2022-poetlesson/PhysObjectExtractor/trunk/poet_cfg.py_4)
>   file and save it as `python/poet_cfg.py`
>   * Download [this](https://raw.githubusercontent.com/cms-opendata-analyses/PhysObjectExtractorTool/odws2022-poetlesson/PhysObjectExtractor/trunk/ElectronAnalyzer.cc_1) file and save it as `src/ElectronAnalyzer.cc`
>   * Download [this](https://raw.githubusercontent.com/cms-opendata-analyses/PhysObjectExtractorTool/odws2022-poetlesson/PhysObjectExtractor/trunk/BuildFile.xml_1) file and save it as `BuildFile.xml`
>
{: .prereq}

## Overview of muon reconstruction

Muons are the *M* in CMS (Compact Muon Solenoid).  This is in part because they are reconstructed basically using all the CMS sub-detectors.  As it nicely summarized [here](https://cms.cern/detector/detecting-muons):

*[A muon] is measured by fitting a curve to the hits registered in the four muon stations, which are located outside of the magnet coil, interleaved with iron "return yoke" plates. The particle path is measured by tracking its position through the multiple active layers of each station; for improved precision, this information is combined with the CMS silicon tracker measurements. Measuring the trajectory provides a measurement of particle momentum. Indeed, the strong magnetic field generated by the CMS solenoid bends the particle's trajectory, with a bending radius that depends on its momentum: the more straight the track, the higher the momentum.*

![](https://cms.cern/sites/default/files/inline-images/MuStations.gif)

Depending essentially on the kind of sub-detectors were used to reconstruct muons, they are usually classified accroding to the summary image below.

![](https://twiki.cern.ch/twiki/pub/CMSPublic/WorkBookMuonAnalysis/muonreco.png)


Muons have many features in common with electrons, but their own unique identification algorithms. We will be studying the `MuonAnalyzer.cc`, which is part of the POET collection of EDAnalyzers.

<!-- This link is not available publicly yet
CMS TWiki references:
 * Muons: [SWGuide Muon ID](https://twiki.cern.ch/twiki/bin/viewauth/CMS/SWGuideMuonIdRun2#Baseline_muon_selections_for_Run)
-->

## The `MuonAnalyzer.cc` EDAnalyzer

If you open the `src/MuonAnalyzer.cc` file, you will notice the headers used to extract information for Muons.  As for the electron, you may have noticed that several variables depend also on (or are calculated with respect to) the primary vertex.

~~~
#include "DataFormats/PatCandidates/interface/Muon.h"
#include "DataFormats/VertexReco/interface/VertexFwd.h"
#include "DataFormats/VertexReco/interface/Vertex.h"
~~~

Now, remember the EDAnalyzer class you learned about in the pre-exercises and how to access data inside the EDAnalyzer. The *analyze* function of an EDAnalyzer is performed once per event. Muons can be accessed like this:

~~~
void
MuonAnalyzer::analyze(const edm::Event &iEvent, const edm::EventSetup &iSetup)
{

  using namespace edm;
  using namespace std;

  Handle<pat::MuonCollection> muons;
  iEvent.getByToken(muonToken_, muons);

  Handle<reco::VertexCollection> vertices;
  iEvent.getByToken(vtxToken_, vertices);
~~~
{: .language-cpp}

The result of the *getToken* method is an object called “muons”, which is a collection of all the muon objects. Collection classes are generally constructed as vectors. We can quickly create a loop to access individual muons:

~~~
for (const pat::Muon &mu : *muons){
  {

    muon_e.push_back(mu.energy());
    muon_pt.push_back(mu.pt());
    muon_eta.push_back(mu.eta());
    muon_phi.push_back(mu.phi());

    muon_px.push_back(mu.px());
    muon_py.push_back(mu.py());
    muon_pz.push_back(mu.pz());

}
~~~
{: .language-cpp}

Many of the most important kinematic quantities defining a physics object are accessed in a common way across all the objects. They have associated energy-momentum vectors, typically constructed using transverse momentum, pseudorapdity, azimuthal angle, and mass or energy. In our case of study, the `MuonAnalyzer.cc`, the muon four-vector elements are accessed as shown above. 


### Muon identification and isolation

To get information associated to the track of the muon, we have to include specific variables. Those variables are added inside the private member function, as vector variables. The primary purpose is to record several values.
~~~
std::vector<float> muon_dxy;
std::vector<float> muon_dz;
std::vector<float> muon_dxyError;
std::vector<float> muon_dzError;
~~~
{: .language-cpp}

The process for accesing the track includes a loop which is placed in the analyzer function `MuonAnalyzer::MuonAnalyzer`, as the previous example.
~~~
for (const pat::Muon &mu : *muons)
    { muon_dxy.push_back(mu.muonBestTrack()->dxy(PV.position()));
      muon_dz.push_back(mu.muonBestTrack()->dz(PV.position()));
      muon_dxyError.push_back(mu.muonBestTrack()->d0Error());
      muon_dzError.push_back(mu.muonBestTrack()->dzError());
}
~~~
{: .language-cpp}
For initializing the variables, one has to clear them at the beginning of the analyzer function:
~~~
muon_dxy.clear();
muon_dz.clear();
muon_dxyError.clear();
muon_dzError.clear();
~~~
{: .language-cpp}

The CMS Muon object group has created member functions for the identification algorithms that
store pass/fail decisions about the quality of each muon. As shown below, the algorithm depends
on which vertex is considered as the primary interaction vertex.

Hard processes produce large angles between the final state partons. The final object of interest will be separated from
the other objects in the event or be *isolated*. For instance, an isolated muon might be produced in the decay of a W boson.
In contrast, a non-isolated muon can come from a weak decay inside a jet.

Muon isolation is calculated from a combination of factors: energy from charged hadrons, energy from
neutral hadrons, and energy from photons, all in a cone of radius $$ R = \sqrt{\eta^2 + \phi^2} < 0.3$$ or $$<0.4$$ around
the muon. Many algorithms also feature a *correction factor* that subtracts average energy expected
from pileup contributions to this cone -- we'll explore this later as an exercise. Decisions are made by comparing this energy sum to the
transverse momentum of the muon.

~~~
auto iso04 = mu.pfIsolationR04();
muon_pfreliso04all.push_back((iso04.sumChargedHadronPt + iso04.sumNeutralHadronEt + iso04.sumPhotonEt)/mu.pt());
~~~
{: .language-cpp}

### Modifications to align with our target analysis

For muons, we will also need the `sip3d` variable we included already for the electrons.  This is the next task:

> ## Add the sip3d variable to muons
>
> Now that you know the answer for electrons, it should be rather straight-forward to implent something similar for the muon analyzer.  Go ahead and make that change very quickly.  Don't forget to recompile and test.  Make sure the `myoutput.root` histogram contains this variable for muons as well as for electrons.
> 
> > ## Solution
> >
> > You can download [this](https://raw.githubusercontent.com/cms-opendata-analyses/PhysObjectExtractorTool/odws2022-poetlesson/PhysObjectExtractor/trunk/MuonAnalyzer.cc_1) file and save it as `src/MuonAnalyzer.cc`.  Do not forget to recompile and run.
> {: .solution}
{: .challenge}

Now, there is another variable that was used for our target analysis but doesn't exist in your `src/MuonAnalyzer.cc`.  

Muon isolation is based on the sum of the $$p_{T}$$ of the charged and neutral hadrons as well as photons 
reconstructed in a cone R=0.4 around the muon momentum. The sum of the $$p_{T}$$ of the charged hadrons associated to vertices other
than the primary vertex, is used to correct for pileup contamination in the total flux of neutrals found in the muon isolation cone. 
A factor of $$\beta = 0.5$$ is used to scale this contribution as: 

$$
I_{\mu} = \frac{1}{p_{T}}\cdot \displaystyle\Sigma_{R<0.4}\left[p_{T}^{ch}+{\rm max}(p_{T}^{\gamma}+p_{T}^{nh}-0.5\cdot p_{T}^{pu\ ch},0)\right]
$$


> ## Add the `pfreliso04DBCorr` variable
>
> As you may notice, the so-called $$\beta$$ factor in the expression above was never included in our `MuonAnalyzer.cc`.  Your task consists of computing and adding this new relative isolation variable (we will call it `pfreliso04DBCorr`) to our analyzer.  Resist the urgency to look at the solution (it is truly very simple.)
>
> These are some hints:
> * We already have most of that variable implemented under other (very similar) reliso variable.
> * Similar to, for instance, the method `.sumPhotonEt` of the `iso04` object, there must be another method which relates to the so-called pile-up contamination that we need.  You might need to do some detective work in order to find it.
> * The *max* function in the expression above can be thought of just the C++ `std::max` function.
>
> > ## Solution
> >
> > You can download [this](https://raw.githubusercontent.com/cms-opendata-analyses/PhysObjectExtractorTool/odws2022-poetlesson/PhysObjectExtractor/trunk/MuonAnalyzer.cc_2) file and save it as `src/MuonAnalyzer.cc`.  Do not forget to recompile and run.
> {: .solution}
{: .challenge}

{% include links.md %}
