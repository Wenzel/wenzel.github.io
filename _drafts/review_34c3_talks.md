---
layout: post
title: A review of 34c3 talks
---

This is my _personal_ review of the 34c3 talks.

- I haven't watched all of them, and i don't intend to do so
- the selection is just based on my **personal** taste and opinion on what interests me at the moment.

Enjoy !

# Favorites

## Gamified Control

[Link](https://media.ccc.de/v/34c3-8874-gamified_control#t=2031)

I'm starting by this talk since it's a **must watch** in my opinion.

China has implemented their new Social Credit System (`SCS`) throught the use of big data and surveillance.
This system will rate the online and offline behavior of the citizen to assess them and give them a score.
Depending how well you _perform_, you will get bonuses or penalities. More important, the score is determined by
you friend's score.

China has **gamified** being an obedient citizen, using **social pressure**.

Notes:
- SCS will be mandatory for every citizen in 2020
- the BAT (Baidu, Alibaba, Tencent) are collaborating with the governement
- what pushes your score up
    * retweeting the governement propaganda (economy, etc)
    * buying local products
    * having obedient friends
- whats pushes your score down
    * posting stuff about the tiananmen massacre...
    * buying foreign product (anime from japan, etc..)
    * having non-obedient friends...
- bonuses:
    * faster administration paperwork and procedures (travel papers)
- penalties:
    * lower internet
    * filtered access to the jobmarket
- **anybody can check your score**
- and which friends are dragging your score down
- it's the most powerful oppression tool, since it doesn't rely on violence or
  fear, but on social pressure, which is maintained by the individual
  themselves
- in preparation since 2005, the Chinese gov understood from the beginning the potential of social media and big data
- borader spectrum than China: a tendency to answer social problems with tehcnological solutions

## Forensic Architecture

[link](https://media.ccc.de/v/34c3-9276-forensic_architecture)

An independant research agency investigating new methods in forensics to undertake human rights abuse.

They reconstructed a 3D scene from pictures, and videos from multiples angles to determine exactly when and where a
missile hit the floor, which building were affected, etc

Really impressive !

## Spy vs. Spy: A Modern Study Of Microphone Bugs Operation And Detection

[Link](https://media.ccc.de/v/34c3-8735-spy_vs_spy_a_modern_study_of_microphone_bugs_operation_and_detection)

What's the status of security research to detect hidden spy microphones ?

The researchers present a state-of-the-art study of microphone bugs, as well as
a tool called `Salamandra` to detect and locate hidden microphones !

Notes:
- _The Thing_: a microphone bug, *batteryless*, was installed in a US ambassy
- You can plant a bug in an appartment and listen to the audio just fine from
  300m.

## Bringing Linux back to server boot ROMs with NERF and Heads

[Link](https://media.ccc.de/v/34c3-9056-bringing_linux_back_to_server_boot_roms_with_nerf_and_heads)

Replacing proprietary vendor boot firmware by an open source Linux runtime.

Notes:
- servers are using UEFI, we can replace this with `LinuxBoot`
- during `DXE` (Direct Execution Environement) phases of UEFI, lot of drivers are loaded
    - full network stack
    - displaying the vendor logo
    - legacy device drivers (floppy)
    - same of them have contained buffer overflows...
- UEFI has a network device driver
- GRUB has a network device driver
- Linux ....
- same for USB
- all these components are vulnerable
- reduced the boot time from **8min -> 20sec** !!
- [linuxboot.org](https://www.linuxboot.org/)

## How to drift with any car

[link](https://media.ccc.de/v/34c3-8758-how_to_drift_with_any_car)

They describe how the various electronic components that control your car are
communicating with each over, and how you can listen to the BUS and send your
own messages.

The funny part was when they plugged this with a XBox controller, and played a
video game using the car's wheel as a controler input... :)

Notes:
- various parts of your cars are controller by ECU
- ABS, Transmission, Engine, etc...
- ECU: `Electronic Control Unit`
- more than 70 in a modern car
- they talk to each other through a CAN bus
- you can talk to the CAN bus with a special diagnostic mode known as `OBD-II`

## Briar

[link](https://media.ccc.de/v/34c3-8937-briar)

Briar is a new P2P, resilient messaging system.

And it works even without internet (Wifi, Bluetooth, etc...)

I loved the talk, very good speaker and presentation.

# Surveillance

## Uncovering British spies’ web of sockpuppet social media personas 

[link](https://media.ccc.de/v/34c3-9233-uncovering_british_spies_web_of_sockpuppet_social_media_personas)

The JTRIG (GCHQ) had to create sockpuppet accounts and fake content on social media for their missions.

How easy it is to unmask them ? :)

## Policing in the age of data exploitation

[link](https://media.ccc.de/v/34c3-8940-policing_in_the_age_of_data_exploitation)

How the police and law enforcement are collecting data and exploiting them for their
investigations

Notes:
- Using the data generated by smart cities
- predictive policing
    * if you map where crimes happens, you can predict
    * PredPol is an example
    * Hawkes process (earthquakes, same events will happen shortly after)
    * they even use **moon phases** in their crime prediction !
    * if a crime happens, every person in the network will be arrested, preventively
    * Cellebrite overs a toolkit
- law does not ensure protection
- the police can extract all the data they want
- IOT is the new crime scene
- the police extract every possible information on all the devices they can find on a crime scene, so your data might be
  in their database !

# Tech

## Inside Intel Management Engine

[link](https://media.ccc.de/v/34c3-8762-inside_intel_management_engine)

The Intel ME vulnerability to run unsigned code

I have to go throught the presentation again, because i didn't catch everything on my first watch ...

Notes:
- Intel ME is poorly documented
- but the root of trust for
    * PAVP
    * PTT
    * TPM
    * Boot Guard
- full access to many devices
- hardware capabilities to intercept user activity
- independant 32 bit processor, x86
- custom MINIX
- contains a Java Runtime Machine
- operates when main CPU is powered down (M3 mode)
- Intel JTAG (Joint Test Actions Group)
- mechanism for debugging electronic chips
- Since Intel Skylake

## Are all BSDs created equally?

[link](https://media.ccc.de/v/34c3-8968-are_all_bsds_created_equally)

A survey of BSD vulnerablities.

Destroying the myth, that because it's BSD, it's more secure and there no vulnerabilities compared to Linux

Notes:
- whats the kernel attack surface ?
    * syscalls
    * TCP/IP stack
    * ioctls, drivers
    * filesystems (FUSE)
- syscalls should be tested
- assomption is false, bugs in syscall implementation occur regularly
- OpenBSD was the winner
    * no loadable modules
    * no compat code
    * removed the entire bluetooth stack
    * less syscalls (200+)
    * cut support for older architectures
    * code quality

# AI

## Deep-learning blindspots

[link](https://media.ccc.de/v/34c3-8860-deep_learning_blindspots)

The state of the art research of adversarial learning and how to fool them.

Researchers have found **blindspots**.

Example: a turtle recognized as ... a **rifle**.

# Arts

## Humans as software extensions

[link](https://media.ccc.de/v/34c3-9077-humans_as_software_extensions)

Notes:
- humans today can be employed for a 10 mins work (design, video, etc..)
- work will become fragmented again and again
- post work world ?

# Misc

## Organisational Structures for Sustainable Free Software Development 

[link](https://media.ccc.de/v/34c3-9087-organisational_structures_for_sustainable_free_software_development)

How to manage your free software project ? 

What organizational structures exists to sustain the development ?

What about the funding ?

Notes:
- heartbleed, 7 April 2014
- OpenSSL can't find the funds to sustain the project
- [The art of community building](https://www.amazon.com/Art-Community-Building-New-Participation/dp/1449312063)
- the essence of community
    * belonging
    * share belief
    * creating opportunity to contribute
- Funding is dangerous !
- you need a legal entity
- the funds cannot go through one individual
- you need explicitely documented decision process, to deal with conflicts
