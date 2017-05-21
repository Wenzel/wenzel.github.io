---
layout: post
title: Using libvirt API to get a guest's IP address
---

Working on Nitro and Virtual Machine Instrumentation can be tedius, because 
there is a lot of components that are involved. Also i needed to build a test 
suite to make sure there was no regression and that everything worked as 
expected.

For a reminder, here is what you need to work on Nitro:

- Modified [QEMU](https://github.com/KVM-VMI/qemu) with memory access patch
- Modified [KVM](https://github.com/KVM-VMI/kvm) with an API to query nitro events
- Python [Nitro](https://github.com/KVM-VMI/nitro) library
- [Libvmi](https://github.com/KVM-VMI/libvmi) API to read/write the memory access offered by QEMU 

The test procedure for Nitro was quite simple:

1. boot a virual machine
2. wait for IP
3. monitor the syscalls with Nitro in a separate thread
4. run the test (powershell script or an .exe)
5. wait for the test to terminate
6. collect the events, analysis and shutdown

In this post i wanted to explore a better solution that i found to get the 
Guest's IP address.

# Quick'n dirty

My original solution was just run `ip neigh` to get the `ARP` table and parse the output, for example:

{% highlight shell lineos %}
192.168.1.1 dev wlp5s0 lladdr 40:65:a3:0b:13:d3 REACHABLE
192.168.122.199 dev virbr0 lladdr 52:54:00:87:3d:60 STALE
{% endhighlight %}

Then, you just have to get the Mac address, which can be obtained by parsing the XML 
description of the domain:

{% highlight python lineos %}
# find MAC address
import re
import subprocess
import xml.etree.ElementTree as tree

dom_elem = tree.fromstring(domain.XMLDesc())
mac_addr = dom_elem.find("./devices/interface[@type='network']/mac").get('address')

while True:
    output = subprocess.check_output(["ip", "neigh"])
    for line in output.splitlines():
        m = re.match('(.*) dev [^ ]+ lladdr {} STALE'.format(mac_addr), line.decode('utf-8'))
        if m:
            ip_addr = m.group(1)
            return ip_addr
    time.sleep(1)

{% endhighlight %}

Why is this a bad design ?

1. a new process is created at each second
2. we are parsing the standard output of a process, which is raw text, not a standard format like `JSON`
3. the regexp contains some hardcoded parts (here if the state if different than `STALE`, the regexp fails)

# A better solution

While i was exploring the libvirt API searching for something else, i found that you can directly get the `DHCP Leases`
of a given network (`virNetwork` object).

in `IPython`
{% highlight python lineos %}
net.DHCPLeases()
[{'clientid': None,
  'expirytime': 1495387467,
  'hostname': 'developer-PC',
  'iaid': None,
  'iface': 'virbr0',
  'ipaddr': '192.168.122.199',
  'mac': '52:54:00:87:3d:60',
  'prefix': 24,
  'type': 0}]
{% endhighlight %}

And the new solution
{% highlight python lineos %}
import libvirt
import time
import xml.etree.ElementTree as tree

con = libvirt.open('qemu:///system')
domain = con.lookupByName('ubuntu')

def get_mac(domain):
    dom_elem = tree.fromstring(domain.XMLDesc())
    mac_addr = dom_elem.find("./devices/interface[@type='network']/mac").get('address')
    return mac_addr

def wait_for_ip(domain, network_name='default'):
    mac = get_mac(domain)
    while True:
        net = con.networkLookupByName(network_name)
        leases = net.DHCPLeases()
        for l in leases:
            if l['mac'] == mac:
                return l['ipaddr']
        time.sleep(1)

ipaddr = wait_for_ip(domain)
print(ipaddr)


{% endhighlight %}


This is definitely better because we dropped the `subprocess` and the `re` dependencies, 
and we rely on the `JSON` output of a `libvirt API` !
