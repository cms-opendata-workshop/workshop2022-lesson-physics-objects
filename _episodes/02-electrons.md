---
title: "Electrons"
teaching: 0
exercises: 0
questions:
- "How are electrons and photons treated in CMS OpenData?"
objectives:
- "Learn electron member functions for common track-based quantities"
- "Bookmark informational web pages for electrons and photons"
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
    electron_e.push_back(itElec->energy());
    electron_pt.push_back(itElec->pt());
    ...
}
~~~
{: .language-cpp}

{% include links.md %}
