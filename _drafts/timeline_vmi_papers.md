---
layout: post
title: Timeline of VMI related papers
---

The first thing to do when you want to know what's the _state of the Art_ regarding a specific research field,
is to read the related scientific litterature and figure out what problems they are trying to solve next.

It goes the same for _Nitro_ and _Virtual Machine Introspection_, and i would like to share with you my own
research into VMI related papers which led me to rebuild a timeline of the publications and their key findings.

# 2003
## A Virtual Machine Introspection Based Architecture for Intrusion Detection

This paper is the first one which really uses and defines the term of _Virtual Machine Introspection_. And also 
note that one of the author is the co-founder of _VMware_.

They start by describing the dilemma of _IDS_ deployment:

Either you deploy a _NIDS_, which can only make policies based on the network traffic, is hard to compromise, but is easy to evade.


Or you deploy a _HIDS_, which offers a far better detection rate since it has more sensors, but is also the first
target of any malware trying to compromise the system.

The idea will be to build an "_out-of-guest_" IDS called Livewire.

# 2006
## Antfarm: Tracking Processes in a Virtual Machine Environment

This [paper](research.cs.wisc.edu/adsl/Publications/antfarm-usenix06.ps) describes techniques to track operating system
processes from the VMM.

Process creation is easy since it relies on detecting a new `CR3`

Detecting a context switch is also trivial, because it is the same as detecting that `CR3` has changed.
And writing to `CR3` triggers a `VM_EXIT`, therefore the VMM is notified.

However, detecting process exit is a bit more complicated as there is not a specific event that you can monitor
