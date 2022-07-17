---
title: "Electrons"
teaching: 0
exercises: 0
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

Electrons and photons are both reconstructed in the electromagnetic calorimeter in CMS, so they share many common properties and functions.
In POET we will study the `ElecronAnalyzer.cc`

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

Photons, as neutral objects, do not have a direct track link (though displaced track segments may appear from electrons or positrons produced by the photon as it transits the detector material). While the `charge()` method exists for all objects, it is not used in photon analyses. 

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
"loose" to "tight". Working points that are "looser" tend to have a high efficiency for accepting
real objects, but perhaps a poor rejection rate for "fake" objects. Working points that are
"tighter" tend to have lower efficiencies for accepting real objects, but much better rejection
rates for "fake" objects. The choice of working point is highly analysis dependent! Some analyses
value efficiency over background rejection, and some analyses are the opposite.

The "standard" identification and isolation algorithm results can be accessed from the physics
object classes.

 * Electrons: [Multivariate Electron Identification for Run2
](https://twiki.cern.ch/twiki/bin/viewauth/CMS/MultivariateElectronIdentificationRun2Archive#Non_triggering_electron_MVA_deta)

>Note: current POET implementations of identification working points are appropriate for 2015 data analysis.
{: .testimonial}


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
 * `cutBasedElectronID...veto` indicate how the electron's trajectory varies between the track and the ECAL cluster,
with smaller variations preferred for the "tightest" quality levels.
 * The impact parameters `dxy` and `dz` should also be small for good quality electrons produced in the initial collision.
 * Missing hits are gaps in the trajectory through the inner tracker (shouldn't be any!)

**Isolation** is computed in similar ways for all physics objects: search for particles in a cone around the object of interest and sum up their energies, subtracting off the energy deposited by pileup particles. This sum divided by the object of interest's transverse momentum is called **relative isolation** and is the most common way to determine whether an object was produced "promptly" in or following the proton-proton collision (ex: electrons from a Z boson decay, or photons from a Higgs boson decay). Relative isolation values will tend to be large for particles that emerged from weak decays of hadrons within jets, or other similar "nonprompt" processes. For electrons, isolation is computed as:

~~~
...
electron_veto.push_back(el.electronID("cutBasedElectronID-Spring15-25ns-V1-standalone-veto"));
...
~~~
{: .language-cpp}

* The conversion veto is an algorithm that rejects electrons coming from photon conversions in the tracker, which should instead be reconstructed as part of the photon.

{% include links.md %}
