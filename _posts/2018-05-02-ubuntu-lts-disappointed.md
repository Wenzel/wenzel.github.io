---
layout: post
title: My disappointment with Ubuntu 16.04 LTS
---

Last year, I came into a situation where I had to work daily with Linux on my company laptop.

The thing is that Windows 10 is already installed and configured by the IT department.

Installing it in a VM was a possibility, but not really convenient.

So i decided to install it as my main operating system.

I immediately choose `Ubuntu`, for a couple of reasons:

- It is already used and adopted by companies around the world
- Good hardware support in general, some Dell XPS laptops are shipped with Ubuntu (my laptop is a Dell)
- The community is always friendly and happy to help you
- If i need to install extra software, there is a good chance that they only exist for `Ubuntu`

For more stability, i pick the latest `LTS`: `Ubuntu 16.04`.

And so far I am quite **disappointed**.

# 1 - The WiFi

I start with the first, and probably the most annying bug I had (and still have):
When you suspend the laptop, and then later on resume it, the `iwlwifi` kernel module that handles
the Wifi network card is **fucked up**.

It just cannot recover and get the device in a ready state.

![iwlwifi_bug]({{ "/public/images/ubuntu_bug/iwlwifi.png" | absolute_url }})

So what are you supposed to do ?

Well, remove the kernel module, and load it again. It fixes the bug.

That's how the following ended up in my bash history:

~~~
$ sudo modprobe -r iwlwifi && sudo modprobe iwlwifi
~~~

Problem solved. But think again.
Every time should suspend your laptop to move into a meeting in the company, you have to type
this command, and probably your sudo password.

Try to picture how annoying it must be to do this 5 times per day.

And Why ? Because we are in 2018, and somehow, I still have WiFi issues on Linux.

Yay.

# 2 - NetworkManager

We continue with the second one, `NetworkManager`, which was sometimes completely absent when Ubuntu finishes its booting sequence
and arrived on the Desktop.

I simply had no network, because `NetworkManager` was `inactive` !

It had never been started in the first place.

And this continued to happen every once and a while.

So again, the fix

~~~
$ sudo systemctl restart NetworkManager
~~~

But why do I have to do this ??

# 3 - Firefox and the Window manager

The third one: when i had to start `Firefox` for the first time after booting, I was filled with fear.

I was afraid that it might crash `Ubuntu`.

Sometimes, a bug in the `i915` kernel module was freezing the window manager when `Firefox` was trying to maximize its main window.

And you are good for a reboot, and all that it implies (`LVM` password, account password, restarting `NetworkManager` if dead ...)

# 4 - Crashes

And now the best, i had crashes, crashes everywhere !

My desktop was filled with those little bug report windows !!

A few samples of my selection:

## nm-connection-editor

![nm-connection-editor]({{ "/public/images/ubuntu_bug/nm-connection-editor.png" | absolute_url }})

## ibus

![ibus]({{ "/public/images/ubuntu_bug/ibus.png" | absolute_url }})

## gedit

![gedit]({{ "/public/images/ubuntu_bug/gedit.png" | absolute_url }})

## unity-settings-daemon

![unity-setting-daemon]({{ "/public/images/ubuntu_bug/unit-settings-daemon.png" | absolute_url }})

And to this day, everytime `Ubuntu` boots and I login, I am greeted by a bug report:

![after-login]({{ "/public/images/ubuntu_bug/after_login.png" | absolute_url }})

# Conclusion

So my question is: 

Is this the best that `Ubuntu` has to offer me ? Is this the most stable, battle tested software that i can get ?

Or my hopes and expectations are maybe too high.

What do you think ?
