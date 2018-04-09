---
layout: post
title: Live Memory Forensics with Rekall and LibVMI
---

For the past 2 months, i worked with Michael Cohen, to provide an integration
of `LibVMI` into the `Rekall` framework.

The result is a new address space, allowing you to run Rekall's modules
directly on the physical memory of the guest, whithout having to install any
software to take a memory dump.

This post will explain how to setup and use this new VMI address space.

# Setup

## 1 - Rekall from git

First, you will need `Rekall` from git master, as the pull request have been
merged recently, and there have been no new releases of `Rekall` yet.

    virtualenv -p python3 venv
    source venv/bin/activate
    (venv) pip install --upgrade setuptools pip wheel
    (venv) git clone https://github.com/google/rekall.git
    (venv) pip install --editable rekall/rekall-lib
    (venv) pip install --editable rekall/rekall-core
    (venv) pip install --editable rekall/rekall-agent
    (venv) pip install --editable rekall

## 2 - LibVMI C library

This will download and install the [`LibVMI` C
library](https://github.com/libvmi/libvmi) in `/usr/local/lib`

    git clone https://github.com/libvmi/libvmi
    cd libvmi
    ./autogen.sh
    ./configure
    make -j4
    make install
    sudo make install

## 3 - LibVMI Python bindings

The next step is to install the [Python
bindings](https://github.com/libvmi/python) to `LibVMI`.

Take the same `virtualenv` as for `Rekall`.

    git clone https://github.com/libvmi/python python-libvmi
    cd python-libvmi
    (venv) ./setup.py build
    (venv) ./setup.py install

# Usage

I will assume you are running a `Windows 7` VM under either `Xen` or `KVM`.

Note: To access the memory of a KVM guest with `LibVMI`, you need to patch `QEMU`.

The `VMI` address space can be initialized by a specific `URL`, passed as a `filename`:

    vmi://hypervisor/domain

Therefore, for our case, it will be

    vmi://kvm/windows_7

You can call the `pslist` module like this:

    $ (venv) rekall -f vmi://kvm/windows_7 pslist
    --libvirt version 1003001
    --qmp: virsh -c qemu:///system qemu-monitor-command windows_7 '{"execute": "pmemaccess", "arguments": {"path": "/tmp/vmiEXqyY1"}}'
    --kvm: using custom patch for fast memory access
    --qmp: virsh -c qemu:///system qemu-monitor-command windows_7 '{"execute": "human-monitor-command", "arguments": {"command-line": "info mtree"}}'
    --qmp: virsh -c qemu:///system qemu-monitor-command windows_7 '{"execute": "human-monitor-command", "arguments": {"command-line": "info mtree"}}'
      _EPROCESS            name          pid   ppid  thread_count handle_count session_id wow64    process_create_time       process_exit_time    
    -------------- -------------------- ----- ------ ------------ ------------ ---------- ------ ------------------------ ------------------------
    0xfa8001167040 System                   4      0           79          455          - False  2018-04-10 09:24:06Z     -                       
    0xfa8002216310 smss.exe               252      4            2           29          - False  2018-04-10 09:24:06Z     -                       
    0xfa8002be3720 svchost.exe            272    468           24          435          0 False  2018-04-10 09:24:08Z     -                       
    0xfa8002880950 csrss.exe              328    312            9          322          0 False  2018-04-10 09:24:07Z     -                       
    0xfa80020f0060 wininit.exe            376    312            3           75          0 False  2018-04-10 09:24:07Z     -                       
    0xfa800286f9e0 csrss.exe              384    368            7          149          1 False  2018-04-10 09:24:07Z     -                       
    0xfa8002954060 winlogon.exe           412    368            4          109          1 False  2018-04-10 09:24:07Z     -                       
    0xfa800298c660 services.exe           468    376            9          184          0 False  2018-04-10 09:24:07Z     -                       
    0xfa8002993a60 lsass.exe              480    376            7          436          0 False  2018-04-10 09:24:07Z     -                       
    0xfa8002a573e0 lsm.exe                488    376           10          133          0 False  2018-04-10 09:24:07Z     -                       
    0xfa8002acd2b0 svchost.exe            584    468           12          351          0 False  2018-04-10 09:24:08Z     -                       
    0xfa8002afa060 svchost.exe            644    468            6          216          0 False  2018-04-10 09:24:08Z     -                       
    0xfa8002b199e0 svchost.exe            696    468           23          408          0 False  2018-04-10 09:24:08Z     -                       
    0xfa8002b7d410 svchost.exe            820    468           22          452          0 False  2018-04-10 09:24:08Z     -                       
    0xfa8002b7ab30 svchost.exe            860    468           34          724          0 False  2018-04-10 09:24:08Z     -                       
    0xfa8002bc0b30 svchost.exe            960    468           20          312          0 False  2018-04-10 09:24:08Z     -                       
    0xfa8002c597c0 dwm.exe               1048    820            5           70          1 False  2018-04-10 09:24:09Z     -                       
    0xfa8002c5db30 explorer.exe          1064   1040           28          587          1 False  2018-04-10 09:24:09Z     -                       
    0xfa8002c98190 spoolsv.exe           1128    468            5           74          0 False  2018-04-10 09:24:09Z     -                       
    0xfa8002cb6060 taskhost.exe          1156    468           10          146          1 False  2018-04-10 09:24:09Z     -                       
    0xfa8002cbbb30 svchost.exe           1196    468           22          313          0 False  2018-04-10 09:24:09Z     -                       
    0xfa8002cd6350 svchost.exe           1284    468            5           89          0 False  2018-04-10 09:24:11Z     -                       
    0xfa8002d689e0 svchost.exe           1336    468           10          157          0 False  2018-04-10 09:24:09Z     -                       
    0xfa800316bb30 audiodg.exe           1404    696            5          119          0 False  2018-04-10 09:24:18Z     -                       
    0xfa8002dc6b30 sppsvc.exe            1476    468            5          149          0 False  2018-04-10 09:24:10Z     -                       
    0xfa8002e15b30 wlms.exe              1532    468            4           43          0 False  2018-04-10 09:24:10Z     -                       
    0xfa800280eb30 WmiPrvSE.exe          2924    584            8          118          0 False  2018-04-10 09:25:14Z     -     

The first lines are debugging output for the `LibVMI` `KVM` driver that i
enabled in my own setup.

For the rest, you can see the output of the `pslist` plugin =)

It's your turn now, try your favorite `Rekall` plugin, and enjoy the pleasure
of live memory forensics without memory dump !

Note1: You may need to `export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib"`.

Note2: a more general syntax is available, you can skip the hypervisor:

    vmi:///windows_7

`KVM` will be automatically detected !

