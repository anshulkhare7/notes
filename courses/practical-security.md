# Practical Security: Simple Practices for Defending Your Systems

## Patching

Patching is the ongoing practice of looking at what software you have in place, researching what vulnerabilities have been discovered in that software, upgrading the vulnerable software to secure versions, and testing to make sure that the new versions work.

**Why Patch?**

* Even if your developers write the perfect code, it's hard to figure out what vulnerabilities exist in third-party software, when these vulnerabilities will become public, or when patches will become available. 
* Once a vulnerability is made public, security researchers and criminals alike start writing tools to scan for vulnerable computers. 
* Scanning tools allow attackers to find your vulnerable public systems even if they’d never had any reason to attack you before. 
* Even if the technical details of a vulnerability are not made public and only a patch is made available, motivated attackers can look at what’s changed to try to find likely attack vectors. 
* Continuing to work on old version of libraries is dangerious, even they function well, because the day a vulnerability is found, you'd be required to move to latest fixed version which can break our functionality. Which means we won't be unable to upgrade to the fixed version, even though we know there is a vulnerability in the version we’re using.

**Good Patching Practice**

* A good patching practice is one where one should be able to discover what 3rd-party libraries are being used, upgrade them quickly, test the upgrade. Above all, since these upgrades are often required to be done on short notice and frequently, automating the whole process is essential to avoid errorprone manual upgrades which themselves can become source of new vulnerabilities.
* 
