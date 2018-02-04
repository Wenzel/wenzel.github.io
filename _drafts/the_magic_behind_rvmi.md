---
layout: post
title: The Magic behind RVMI
---

Last year, when FireEye released `RVMI`, i was really excited and immediately installed the project on my computer to
try it.

However, i never really digged into the internals of how this was made possible.  You
neither ? Good, because in this article, i will explore `RVMI`'s architecture and highlight the details and features that
are hidding behing it.

But first, what does a hypervisor-level debugger really brings us ?

# Why a Hypervisor-level Debugger ?

Debugging any process in a virtual machine from the hypervisor itself is a very powerful paradigm, for multiple reasons:

## 1. Painless debugging tools

Hypervisor-level debuggers are very handy because you can debug any virtual machine as long as your hypervisor has been
modified to have VMI capabilities.

You don't have to install debugging tools, which removes the pain of downloading and installing a debugger in each
virtual machine you have. (for Windows 7 for example you have to install a lot of updates/.NET framework before you can
actually install `WinDBG`)

## 2. Stealth

As there is no debugger, you are agentless, and therefore completely invisible, leaving (almost) no traces on the
virtual machine.

On the contrary of a a traditional debugger like GDB, which has to become the parent of the process you are actually
debugging, you are not modifying the execution environment of your debuggee.

That's exactly what you want, especially when you are dealing with **malwares**.


## 3. Kernel/User mode debugger

With a hypervisor-level debugger, you can intercept **any** process running inside the VM. Kernel included.

This means you have the **same tool** to debug usermode processes and the kernel, and considering the diversity of
debuggers available on Windows, each having it's own interface/scripting capabilities, building your knowledge and
skills on top of one tool is definitely a plus.

## 4. Cross-platform debugger

Another big advantage is that the debugger totally abstracted from the operating system it's trying to debugging.

It only relies on the `x86` architecture as well as the virtual machine's hardware.

This means that you can debug processes running on `Linux/Windows/MacOS`/whatever operating system running on your
hypervisor's architecture.

## 5. Cloud provider ?

Let's take the cloud point of view now.

You are running hundreds of VMs, and someone lost his `SSH` credentials to one of them.

No login means you cannot install a debugger (among other things...).

We could imagine your cloud provider support team debugging virtual machines in realtime, and even offering you a VMI
based interface as a rescue system !

# RVMI's architecture

`RVMI` is built on top of `QEMU/KVM` and `Rekall`.

In short, `QEMU/KVM` have been modified to add Virtual Machine Introspection capabilities, and `Rekall` has been
customized make to interact with `QEMU`.

Let's look at the modifications


## QEMU

kvm.h
    nouvelles ioctl
kvm-all.c
    kvm_update_mtf
sysemu.h
    vm_start_silent
kvm.h
    kvmi_vmi_event
kvm_vmi.h

savevm.c
    qmp_save/loadvm

monitor.c
    vmi_initialize/deinitliaz
qapi-schema.json
    new qemu commands
kvm.c
    new EXIT_REASON
vl.c
    vm_start_silent
vmi.c

## Rekall

ipython_support.py
    vmi_ipython init
qmp_as.py
    
callbacks.py

common.py
    core class for VMI plugins
event.py
    rvmi event handling
iface.py
    QMP interface
qmp.py
vmi.py
