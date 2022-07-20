---
title: "Muons"
teaching: 20
exercises: 0
questions:
- "How are muons treated in CMS OpenData?"
objectives:
- "Learn member functions for muon track-based quantities"
- "Bookmark informational web pages for different objects"
- "Learn member functions for identification and isolation of muons"
keypoints:
- "Track access may differ, but track-related member functions are common across objects."
- "Physics objects in CMS are reconstructed from detector signals and are never 100% certain!"
- "Muons typically use pre-configured identification and isolation variable member functions."
- "Member functions for these algorithms are documented on public TWiki pages."
---
Muons have many features in common with electrons, but their own unique identification algorithms. We will be studying `MuonAnalyzer.cc`.


CMS TWiki references:
 * Muons: [SWGuide Muon ID](https://twiki.cern.ch/twiki/bin/viewauth/CMS/SWGuideMuonIdRun2#Baseline_muon_selections_for_Run)

Inside the code files found in `PhysObjectExtractor/src/`, the definitions of different classes are included. Continuing with muons, we include the following in `PhysObjectExtractor/src/MuonAnalyzer.cc`. The included statements mentioned below correspond to muons.
~~~
#include "DataFormats/PatCandidates/interface/Muon.h"
#include "DataFormats/VertexReco/interface/VertexFwd.h"
#include "DataFormats/VertexReco/interface/Vertex.h"
~~~

Now, remember the EDAnalyzer class you learned about in the pre-exercises and how to access data inside the EDAnalyzer. The “analyze” function of an EDAnalyzer is performed once per event. Muons can be accessed like this:
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

The result of the getByToken command is a variable called “muons” which is a collection of all the muon objects. Collection classes are generally constructed as std::vectors. We can quickly create a loop to access individual muons:
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

Many of the most important kinematic quantities defining a physics object are accessed in a common way across all the objects. They have associated energy-momentum vectors, typically constructed using transverse momentum, pseudorapdity, azimuthal angle, and mass or energy. In our case of study, `MuonAnalyzer.cc`, the muon four-vector elements are accessed as shown above. 

## Muon identification and isolation
To manipulate the data of the track, we have to include three additional variables. Those variables are added inside the private member function, as vector variables. Using c++ language a vector variable is a dynamic array. The primary purpose is to record several values.
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
For initializing the variables, it has to be a declared at the beginning of the analyzer function as the following:
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
the other objects in the event or be "isolated". For instance, an isolated muon might be produced in the decay of a W boson.
In contrast, a non-isolated muon can come from a weak decay inside a jet.

Muon isolation is calculated from a combination of factors: energy from charged hadrons, energy from
neutral hadrons, and energy from photons, all in a cone of radius dR < 0.3 or 0.4 around
the muon. Many algorithms also feature a "correction factor" that subtracts average energy expected
from pileup contributions to this cone -- we'll explore this in the hands-on exercise. Decisions are made by comparing this energy sum to the
transverse momentum of the muon.

~~~
auto iso04 = mu.pfIsolationR04();
muon_pfreliso04all.push_back((iso04.sumChargedHadronPt + iso04.sumNeutralHadronEt + iso04.sumPhotonEt)/mu.pt());
~~~
{: .language-cpp}

{% include links.md %}
