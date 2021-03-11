# Practical Security: Simple Practices for Defending Your Systems

## Patching

**Why Patch?**

* Even if your developers We don’t know what vulnerabilities exist in third-party software, when these vulnerabilities will become public, or when patches will become available. * * Once a vulnerability is made public, security researchers and criminals alike start writing tools to scan for vulnerable computers. 
* Scanning tools allow attackers to find your vulnerable public systems even if they’d never had any reason to attack you before. 
* Even if the technical details of a vulnerability are not made public and only a patch is made available, motivated attackers can look at what’s changed to try to find likely attack vectors. 

**Good Patching Practice**

* A good patching practice is one where one should be able to discover what 3rd-party libraries are being used, upgrade them quickly, test the upgrade. Above all, since these upgrades are often required to be done on short notice and frequently, automating the whole process is essential to avoid errorprone manual upgrades which themselves can become source of new vulnerabilities.
* 
