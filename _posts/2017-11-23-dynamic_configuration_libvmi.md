---
layout: post
title: Dynamic configuration of LibVMI using Rekall
---

# LibVMI

I have been using [LibVMI](https://github.com/libvmi/libvmi) since a year now, and it has been really helpful when
working on [KVM-VMI](https://github.com/KVM-VMI/kvm-vmi) and [Nitro](https://github.com/KVM-VMI/nitro/tree/master). It
provides a nice abstraction on top of Xen and KVM and deals with the low-level details of accessing content from the
virtual/physical memory (page tables), intercepting hardware events, or reading the VCPU registers.

But there is a catch: Before you are able to introspect a virtual machine, LibVMI needs to know the offsets of some
important structures like `EPROCESS` on Windows and `task_struct` on Linux.

Let's review LibVMI configuration methods.

# Configuration

LibVMI proposes 3 methods to configure a domain for instrospection:

{% highlight C %}
typedef enum vmi_config {

    VMI_CONFIG_GLOBAL_FILE_ENTRY, /**< config in file provided */

    VMI_CONFIG_STRING,            /**< config string provided */

    VMI_CONFIG_GHASHTABLE,        /**< config GHashTable provided */
} vmi_config_t;
{% endhighlight %}

Once your VMI application is configured, you can start introspecting your domain.
For example, look at `libvmi/examples/process-list winxp` to enumerate the processes running on the virtual machine !

## From a file

The first and the simplest way to configure LibVMI is to use a configuration file.
LibVMI will try to find it in the following locations:
- current direction
- `$HOME/etc/libvmi.conf`
- `etc/libvmi.conf`

In this file you have to specify the exact name of the domain you want to introspect and a few offsets.

For Windows XP:

{% highlight C %}
winxp {
    win_pdbase  = 0x18;
    win_pid     = 0x84;
    win_tasks   = 0x88;
    win_pname = 0x174;
}
{% endhighlight %}

{% highlight C %}
int main(int argc, char *argv[])
{
    vmi_instance_t vmi;
    char *name = "winxp";
    vmi_init_complete(&vmi, name, VMI_INIT_DOMAINNAME, NULL,
                        VMI_CONFIG_GLOBAL_FILE_ENTRY, NULL, NULL));
}
{% endhighlight %}


## From a string

The second method allows you to set the configuration from a string, in your application.

{% highlight C %}
int main(int argc, char *argv[])
{
    vmi_instance_t vmi;
    char *name = "winxp";
    char *config = "{ostype = "Windows"; win_pdbase=0x18; win_pid=0x84; win_tasks=0x88; win_pname=0x174;}"
    vmi_init_complete(&vmi, name, VMI_INIT_DOMAINNAME, NULL,
                        VMI_CONFIG_STRING, (void*)config, NULL));
}
{% endhighlight %}

## From a GHashTable

The last and the most flexible is to provide the offsets via a GLib `GHashTable`:

{% highlight C %}
int main(int argc, char *argv[])
{
    vmi_instance_t vmi;
    char *name = "winxp";

    int winpdbase = 0x18;
    int winpid = 0x84;
    int wintasks = 0x88;
    int winpname = 0x174;
    GHashTable *config = g_hash_table_new(g_str_hash, g_str_equal);
    g_hash_table_insert(config, "ostype", "Windows");
    g_hash_table_insert(config, "win_pdbase", &winpdbase);
    g_hash_table_insert(config, "win_pid", &winpid);
    g_hash_table_insert(config, "win_tasks", &wintasks);
    g_hash_table_insert(config, "win_pname", &winpname);

    vmi_init_complete(&vmi, name, VMI_INIT_DOMAINNAME, NULL,
                        VMI_CONFIG_GHASHTABLE, (void*)config, NULL));
}
{% endhighlight %}

## From a rekall profile

There is a alternative method to configure LibVMI, which makes use of Rekall profiles.

A profile is a JSON file containing all the information Rekall will need to inspect the memory of the virtual machine
and run it's plugins like `pslist` or `dlllist`.

The procedure to create this profile is documented on the [README](https://github.com/libvmi/libvmi), so i will not
detail it here.

{% highlight C %}
int main(int argc, char *argv[])
{
    vmi_instance_t vmi;
    char *name = "winxp";
    char *rekall_profile = "/home/wenzel/.rekall/profiles/winxp.json";

    GHashTable *config = g_hash_table_new(g_str_hash, g_str_equal);
    g_hash_table_insert(config, "ostype", "Windows");
    g_hash_table_insert(config, "rekall_profile", profile);

    vmi_init_complete(&vmi, name, VMI_INIT_DOMAINNAME, NULL,
                        VMI_CONFIG_GHASHTABLE, (void*)config, NULL));
}
{% endhighlight %}

# Dynamic configuration with Rekall

But all of this is very tedius at the end.

To introspect you domain, you have to configure it in advance, get the offsets and put in a config
file/string/ghashtable or generate a rekall profile.

What if you are given an entire cloud to monitor ?
What if all of the virtual machines are different (Windows XP/7/8/10) and they don't have the same patch level ? 
(remember that Service packs changes the offsets of `EPROCESS` structures...

So what can we do ?
Go one by one, setup the configuration and introspect them ?

What if there was a way to dynamically configure LibVMI, on the fly ?

Well, there is: just take a memory dump with `libvirt`, extract the offsets with `Rekall` and provide this to LibVMI !


I have little example of this concept, and it uses the Python wrapper for LibVMI that i'm writing.
I will assume you already have LibVMI installed on your system.

## Installing Rekall from git

We need the **latest** `Rekall` from github:
{% highlight Bash %}
$ virtualenv -p python3 venv
$ source venv/bin/activate
$ pip install --upgrade setuptools pip wheel
$ git clone https://github.com/google/rekall
$ pip install --editable rekall/rekall-lib
$ pip install --editable rekall/rekall-core
$ pip install --editable rekall/rekall-agent
$ pip install --editable rekall
{% endhighlight %}

## Installing the LibVMI python wrapper

Now you can clone the python wrapper for LibVMI, and install it:
{% highlight Bash %}
$ source venv/bin/activate
$ git clone https://github.com/KVM-VMI/libvmi -b pyvmi_cffi
$ cd libvmi/tools/pyvmi
$ python setup.py build
$ python setup.py install
{% endhighlight %}

Perfect !

Now you have the LibVMI Python wrapper installed !

## Downloading the Python module

The python module below contains the code that will take the memory dump and extract the configuration:

You can grab it from this [Github Gist](https://gist.github.com/Wenzel/57112cebcd0b6ad0622e5ab506290c2c).

`auto_config.py`
{% highlight Python %}
import os
import sys
import stat
import logging
import libvirt
from tempfile import TemporaryDirectory, NamedTemporaryFile
from rekall import plugins, session

def extract_config(ram_dump):
    home = os.getenv('HOME')
    local_cache_path = os.path.join(home, '.rekall_cache')
    try:
        os.makedirs(local_cache_path)
    except OSError: # already exists
        pass
    logging.info('Analyzing RAM dump with Rekall')
    s = session.Session(
            filename=ram_dump,
            autodetect=["rsds"],
            logger=logging.getLogger(),
            autodetect_build_local='none',
            format='data',
            profile_path=[
                local_cache_path,
                "http://profiles.rekall-forensic.com"
            ])

    pdbase = s.profile.get_obj_offset('_KPROCESS', 'DirectoryTableBase')
    tasks = s.profile.get_obj_offset('_EPROCESS', 'ActiveProcessLinks')
    name = s.profile.get_obj_offset('_EPROCESS', 'ImageFileName')
    pid = s.profile.get_obj_offset('_EPROCESS', 'UniqueProcessId')

    config = {
        "ostype": "Windows",
        "win_pdbase": pdbase,
        "win_tasks": tasks,
        "win_pid": pid,
        "win_name": name,
    }

    return config


def get_windows_config(domain):
    # take memory dump
    # we need to put the ram dump in our own directory
    # because otherwise it will be created in /tmp
    # and later owned by root
    with TemporaryDirectory() as tmp_dir:
        with NamedTemporaryFile(dir=tmp_dir) as ram_dump:
            # chmod to be r/w by everyone
            os.chmod(ram_dump.name,
                     stat.S_IRUSR | stat.S_IWUSR |
                     stat.S_IRGRP | stat.S_IWGRP |
                     stat.S_IROTH | stat.S_IWOTH)
            # take a ram dump
            logging.info('Dumping physical memory to %s', ram_dump.name)
            flags = libvirt.VIR_DUMP_MEMORY_ONLY
            dumpformat = libvirt.VIR_DOMAIN_CORE_DUMP_FORMAT_RAW
            domain.coreDumpWithFormat(ram_dump.name, dumpformat, flags)
            config = extract_config(ram_dump.name)
            return config
{% endhighlight %}

## Putting everything together

The last thing we need is to build our VMI application and use our auto detection module:

For the test our application will just call the LibVMI API `translate_ksym2v` to get the virtual address of a kernel
symbol.

`test_libvmi.py` ([Github Gist](https://gist.github.com/Wenzel/49c3951f02c747e549bef27a75f897b6))
{% highlight Python %}
#!/usr/bin/env python3

import sys
import libvirt
import logging
from auto_config import get_windows_config
from libvmi import Libvmi, VMIOS, VMIConfig

if __name__ == '__main__':
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    domain_name = sys.argv[1]
    con = libvirt.open("qemu:///system") # change this to your needs
    domain = con.lookupByName(domain_name)
    config = get_windows_config(domain)


    with Libvmi(domain_name, mode=VMIConfig.DICT, config=config) as vmi:
        vaddr = vmi.translate_ksym2v('PsActiveProcessHead')
        logging.info('PsActiveProcessHead %s', hex(vaddr))

{% endhighlight %}

And let's run it against a Windows 7 domain !

{% highlight Bash %}
./test_libvmi.py Windows_7
INFO:root:Dumping physical memory to /tmp/tmpib9hyg4r/tmp0ntme2bm
INFO:root:Analyzing RAM dump with Rekall
INFO:root:Autodetected physical address space Elf64CoreDump
INFO:root:Loaded profile pe from Local Cache - (in 0.006314992904663086 sec)
INFO:root:Loaded profile nt/undocumented from Local Cache - (in 0.173691987991333 sec)
INFO:root:Loaded profile nt/GUID/F8E2A8B5C9B74BF4A6E4A48F180099942 from Local Cache - (in 0.7517926692962646 sec)
INFO:root:Loaded profile nt/eprocess_index from Local Cache - (in 0.010558605194091797 sec)
INFO:root:Detected ntkrnlmp.pdb with GUID F8E2A8B5C9B74BF4A6E4A48F180099942
xencall: error: Could not obtain handle on privileged command interface: No such file or directory
VMI_ERROR: Failed to open libxc interface.
VMI_WARNING: Invalid offset "win_name" given for Windows target
LibVMI Suggestion: set win_ntoskrnl=0x2602000 in libvmi.conf for faster startup.
LibVMI Suggestion: set win_kdbg=0x1e9070 in libvmi.conf for faster startup.
LibVMI Suggestion: set win_kdvb=0xfffff800027eb070 in libvmi.conf for faster startup.
INFO:root:PsActiveProcessHead 0xfffff80002821b30
{% endhighlight %}
