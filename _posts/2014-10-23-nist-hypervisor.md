---
layout:     post
title:      NIST Hypervisor Security Recommendations
date:       2014-10-23 14:00:00
categories:
  - hypervisor
  - nist

---

NIST recently posted a draft version of their [hypervisor security recommendations](http://csrc.nist.gov/publications/drafts/800-125a/sp800-125a_draft.pdf). Surprisingly they are pretty straight forward and common sense.

<!-- more -->

### My Interpretation

In general the recommendations are fairly easy to understand and even not too hard to implement.

The things that are easy:

1. Use a bare metal hypervisor
2. Hardware assisted virtualization (essentially VT-x)
3. Secure management console with restricted access
4. Standardized logging
5. Separate library of images
6. Setting memory ratio
7. Limit CPU ratios
7. Patch your servers
8. Number of vCPUS less than actual cores
8. Hypervisor firewall
9. Redundant network connections
10. Automated deployment of hypervisor configuration (how could you even do virtualization on any sort of scale without this?)
12. Central authentication system
13. Access control solution

Some of the more difficult recommendations to implement for the average organization might be:

1. TPM and measured launch environments
2. Multi-factor authentication
3. Disallowing drivers
4. Guaranteed vCPU and memory
5. Security monitoring with some sort of VM introspection API

Those are all pretty doable, but would take some work I expect in most organizations.

### NIST Security Recommendations

These are the actual 22 recommendations (a couple are covered by the same points) which I pulled from the NIST document.

1. **HY-SR-1** Bare Metal (Type 1) Hypervisor – Reduced Attack surface because of no Host O/S
2. **HY-SR-2** Hypervisor Platform with hardware assisted virtualization (both Instruction Set and Memory)
4. **HY-SR-3** Hypervisor Platform that provides a Measured Launch Environment (MLE) and a standards-based TPM and an attestation process
5. **HY-SR-4** Hypervisor Management Console with a smaller code and disk footprint and smaller number of exposed interfaces
6. **HY-SR-5** (a) Disallowing non-certified drivers and (b) Running QEMU (device emulation code) process on unprivileged VMs (instead of in a Management or Privileged VM) if architecture permits
7. **HY-SR-6** Limiting the Ratio of Combined Virtual RAM allocated to VMs to Total Physical RAM of the Virtualized Host
8. **HY-SR-7** Ability to specify RAM allocations with Guaranteed, Upper Limit and Priority Values
9. **HY-SR-8** The number of virtual CPUs allocated to a VM should be strictly less than the total number of cores in the hypervisor host
10. **HY-SR-9** It should be possible to specify a lower and upper limit for CPU clock cycles for each VM and also a priority value for CPU access
11. **HY-SR-10** VM Image library should not be hosted on the hypervisor host and should have digitally signed VM images
12. **HY-SR-11 & HY-SR-12** Security Monitoring on VM Activities shouldcover: (a) Malicious processes running inside VMs and (b) Malicious traffic going in and out of VM. These monitoring functions should be based on tools residing outside any monitored VMs and should be based on VM Introspection API
13. **HY-SR-13** Access Control solution with granular permission assignment for VM Management – Permission at VM level, VM Group level, VM Group level with specific exceptions Limiting the number of user accounts with direct access to hypervisor host
14. **HY-SR-14** Limiting the number of user accounts with direct access to hypervisor host
14. **HY-SR-15** Integrating user accounts on the hypervisor host to an enterprise directory infrastructure to: (a) Enable Robust Multi-factor Authentication Protocols and (b) Maintain Integrity of User Account Maintenance
15. **HY-SR-16** Access to Hypervisor Management Console should be: (a) denied to root account and (b) restricted to a limited administrative accounts
16. **HY-SR-17** Use Hypervisor features that enable: (a) definition of a complete set of configuration settings (Gold Configuration) for a hypervisor deployment (b) automate application of those configuration settings to a new or existing hypervisor installation and (c) check compliance of existing hypervisor installation against those configuration settings, if available, in order to minimize manual configuration errors that may increase the security risk.
17. **HY-SR-18** A good hypervisor patch management practice (supported by a homegrown or an established COTS product) should be in placeto keep the hypervisor current with all relevant patches.
18. **HY-SR-19** The built-in firewall for the hypervisor should only be configured for allowing traffic needed for enabled services in the hypervisor, such as Management and specialized security agents and third-party applications.
19. **HY-SR-20** It would be preferable to have a hypervisor logging feature that generates logs in a standardized format to help leverage the use of tools with good analytical capabilities as well as the feature to transmit log records in real time over a secure channel to an external server for fault tolerance.
20. **HY-SR-21** The management interface of the hypervisor should: (a) be placed in a dedicated virtual network segment and (b) access to that interface should only be allowed from designated subnets in the enterprise network
21. **HY-SR-22** Redundant communication channels from a VM to the enterprise (external) network by configuring virtual network paths from a given VM to multiple physical Network interface adapters of the virtualized host.
