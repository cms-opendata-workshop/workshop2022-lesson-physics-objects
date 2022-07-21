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

The result of the getToken method is an object called “muons” which is a collection of all the muon objects. Collection classes are generally constructed as vectors. We can quickly create a loop to access individual muons:
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
For initializing the variables, it has to be declared at the beginning of the analyzer function as the following:
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

>## Hands-on: adding muon IDs 
>
>Using the documentation on the TWiki page and the `MuonAnalyzer.cc`:
> * 1. Write the expression that get the access to the Loose, Medium, Tight, Soft and HightPt muons.
> * 2. Using c++ language, write the expression of the vector variable declaration to implement as if we would like to record the dxz error. 
> * 3. What is implemented to get the 3D impact parameter?
> * 4. In the muon isolation code, we have this line:
>~~~
>muon_pfreliso04all.push_back((iso04.sumChargedHadronPt + iso04.sumNeutralHadronEt + iso04.sumPhotonEt)/mu.pt());
>~~~
>{: .language-cpp}
> What do you think is doing?
> * 5. Add the pass/fail information about the HighPt Tracker Muon identification working point.
>
>> ## Solution:
>> 1. We need to look into the declaration of the variables mentioned:
>>~~~
>> std::vector<int> muon_isLoose;
>> std::vector<int> muon_isMedium;
>> std::vector<int> muon_isTight;
>> std::vector<int> muon_isSoft;
>> std::vector<int> muon_isHighPt;
>>~~~
>>{: .language-cpp}
>> 2. If we would like to implement the record of the dxz error, we would need to declare:
>>~~~
>> std::vector<float> muon_dxz;
>>~~~
>>{: .language-cpp}
>> 3. First, we need to look into the declaration of the parameter:
>>~~~
>> std::vector<float> muon_ip3d;
>> std::vector<float> muon_sip3d;
>>~~~
>>{: .language-cpp}
>> the branches:
>>~~~
>> mtree->Branch("muon_ip3d",&muon_ip3d);
>> mtree->GetBranch("muon_ip3d")->SetTitle("muon impact parameter in 3d");
>> mtree->Branch("muon_sip3d",&muon_sip3d);
>> mtree->GetBranch("muon_sip3d")->SetTitle("muon significance on impact parameter in 3d");
>>~~~
>>>>{: .language-cpp}
>> vector clearing:
>>~~~
>> muon_ip3d.clear();
>> muon_sip3d.clear();
>>~~~
>>>>{: .language-cpp}
>> and finally, vector filling: 
>>~~~
>> muon_ip3d.push_back(ip3dpv.second.value());
>> muon_sip3d.push_back(ip3dpv.second.significance());
>>~~~
>>>>{: .language-cpp}
>> 4. 
>> 5. As we did in the exercises shown above, to add new variables we need to check four code locations: declarations, branches, vector clearing, and vector filling. 
>> You might add Hight Pt Tracker ID beneath the existing Soft and HightPt IDs in each section:
>>~~~
>>std::vector<float> muon_isHighPtTracker;
>>~~~
>>{: .language-cpp}
>>~~~
>>mtree->Branch("muon_isHighPtTracker",&muon_isHighPtTracker);
>>mtree->GetBranch("muon_isHighPtTracker")->SetTitle("muon tagged high pt tracker");
>>~~~
>>{: .language-cpp}
>>~~~
>>muon_isHighPtTracker.clear();
>>~~~
>>{: .language-cpp}
>>~~~
>>muon_isHighPtTracker.push_back(mu.isHighPtTrackerMuon(PV));
>>~~~
>>{: .language-cpp}
>{: .solution}
{: .challenge}

{% include links.md %}
