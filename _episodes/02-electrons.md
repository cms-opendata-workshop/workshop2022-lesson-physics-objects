---
title: "Electrons"
teaching: 15
exercises: 35
questions:
- "How are electrons treated in CMS OpenData?"
objectives:
- "Learn electron member functions for common track-based quantities"
- "Bookmark informational web pages for electrons"
- "Learn member functions for identification and isolation of electrons"
- "Learn member functions for electron detector-related quantities"
keypoints:
- "Quantities such as impact parameters and charge have common member functions."
- "Physics objects in CMS are reconstructed from detector signals and are never 100% certain!"
- "Identification and isolation algorithms are important for reducing fake objects."
- "Member functions for these algorithms are documented on public TWiki pages."
---

Electrons are reconstructed in the electromagnetic calorimeter in CMS, so they share many common properties and functions.
Electrons (and also photons) are standard tools that help us to measure better and understand the properties of known particles. For example, one way to find a Higgs Boson is by looking for signs of two photons in the debris of high energy collisions. Because electrons and photons are crucial in so many different scenarios, the physicists in the CMS Collaboration make sure to do their best to reconstruct and identify these objects.

In POET we will study the `ElecronAnalyzer.cc`

## ElectronAnalyzer.cc file

The first thing that you will see is a set of includes. In particular we have a set of headers for electrons:

~~~
//class to extract electron information
#include "DataFormats/PatCandidates/interface/Electron.h"
#include "DataFormats/EgammaCandidates/interface/GsfElectron.h"
#include "DataFormats/VertexReco/interface/VertexFwd.h"
#include "DataFormats/VertexReco/interface/Vertex.h"
~~~
{: .language-cpp}


## Electron 4-vector and track information

In the loop over the electron collection in `ElectronAnalyzer.cc`, we access elements of the four-vector as shown in the last episode: 
~~~
for (const pat::Electron &el : *electrons){
    ...
    electron_e.push_back(el.energy());
    electron_pt.push_back(el.pt());
    ...
}
~~~
{: .language-cpp}

Most charged physics objects are also connected to tracks from the CMS tracking detectors. The charge of the object can be queried directly:
~~~
electron_ch.push_back(el.charge());
~~~
{: .language-cpp}

Information from tracks provides other kinematic quantities that are common to multiple types of objects.
Often, the most pertinent information about an object to access from its
associated track is its **impact parameter** with respect to the primary interaction vertex.
Since muons can also be tracked through the muon detectors, we first check if the track is
well-defined, and then access impact parameters in the xy-plane (`dxy` or `d0`) and along
the beam axis (`dz`), as well as their respective uncertainties. 

~~~
math::XYZPoint pv(vertices->begin()->position());
...

electron_dxy.push_back(el.gsfTrack()->dxy(pv));
electron_dz.push_back(el.gsfTrack()->dz(pv));
electron_dxyError.push_back(el.gsfTrack()->d0Error());
electron_dzError.push_back(el.gsfTrack()->dzError());
~~~
{: .language-cpp}

>Note: in the case of Photons, since they are neutral objects, do not have a direct track link (though displaced track segments may appear from electrons or positrons produced by the photon as it transits the detector material). While the `charge()` method exists for all objects, it is not used in photon analyses. 
{: .testimonial}

## Detector information for identification

The most signicant difference between a list of certain particles from a Monte Carlo generator and a list
of the corresponding physics objects from CMS is likely the inherent uncertainty in the reconstruction.
Selection of "a muon" or "an electron" for analysis requires algorithms designed to separate "real"
objects from "fakes". These are called **identification** algorithms.

Other algorithms are designed to measure the amount of energy deposited near the object, to determine
if it was likely produced near the primary interaction (typically little nearby energy), or from the
decay of a longer-lived particle (typically a lot of nearby energy). These are called **isolation**
algorithms. Many types of isolation algorithms exist to deal with unique physics cases!

Both types of algorithms function using **working points** that are described on a spectrum from
**"loose"** to **"tight"**. Working points that are "looser" tend to have a high efficiency for accepting
real objects, but perhaps a poor rejection rate for "fake" objects. Working points that are
"tighter" tend to have lower efficiencies for accepting real objects, but much better rejection
rates for "fake" objects. The choice of working point is highly analysis dependent! Some analyses
value efficiency over background rejection, and some analyses are the opposite.

The "standard" identification and isolation algorithm results can be accessed from the physics
object classes.

## Multivariate Electron Identification

In the MVA approach, one forms a single discriminator variable that is computed based on multiple parameters of the electron object and provides the best separation between the signal and backgrounds. One can then cut on discriminator value or use the distribution of the values for a shape based statistical analysis.

There are two basic types of MVAs that are usually provided by EGM:

 * **the triggering MVA**: the discriminator is trained on the electrons that pass typical electron trigger requirements
 * **the non-triggering MVA**: the discriminator is trained on all electrons regardless of the trigger
 
For our purposes, we will use the non-triggering MVA. Note the tags that are used `...wp90` and `...wp80`. As mentioned above, the difference lies on the working point (`wp`).
~~~
      electron_ismvaLoose.push_back(el.electronID("mvaEleID-Spring15-25ns-nonTrig-V1-wp90"));
      electron_ismvaTight.push_back(el.electronID("mvaEleID-Spring15-25ns-nonTrig-V1-wp80"));
~~~
{: .language-cpp}

 * Electrons: [Multivariate Electron Identification for Run2
](https://twiki.cern.ch/twiki/bin/viewauth/CMS/MultivariateElectronIdentificationRun2Archive#Non_triggering_electron_MVA_deta)


## Cut Based Electron ID

Most `pat::<object>` classes contain member functions that return detector-related information. In the
case of electrons, we see this information used as identification criteria:

~~~
      electron_veto.push_back(el.electronID("cutBasedElectronID-Spring15-25ns-V1-standalone-veto"));//
      electron_isLoose.push_back(el.electronID("cutBasedElectronID-Spring15-25ns-V1-standalone-loose"));
      electron_isMedium.push_back(el.electronID("cutBasedElectronID-Spring15-25ns-V1-standalone-medium"));
      electron_isTight.push_back(el.electronID("cutBasedElectronID-Spring15-25ns-V1-standalone-tight"));
~~~
{: .language-cpp}

Let's break down these criteria:
 * `cutBasedElectronID...veto` is a tag that rejects electrons coming from photon conversions in the tracker, which should instead be reconstructed as part of the photon.
 * The impact parameters `dxy` and `dz` should also be small for good quality electrons produced in the initial collision.

**Isolation** is computed in similar ways for all physics objects: search for particles in a cone around the object of interest and sum up their energies, subtracting off the energy deposited by pileup particles. This sum divided by the object of interest's transverse momentum is called **relative isolation** and is the most common way to determine whether an object was produced "promptly" in or following the proton-proton collision (ex: electrons from a Z boson decay, or photons from a Higgs boson decay). Relative isolation values will tend to be large for particles that emerged from weak decays of hadrons within jets, or other similar "nonprompt" processes. For electrons, isolation is computed as:

~~~
...
electron_iso.push_back(el.ecalPFClusterIso());
...
~~~
{: .language-cpp}

>Note: current POET implementations of identification working points are appropriate for 2015 data analysis.
{: .testimonial}

{% include links.md %}
