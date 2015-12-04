---
title: vmfex
authors: apuimedo, dyasny
wiki_title: VDSM-Hooks/vmfex
wiki_revision_count: 6
wiki_last_updated: 2014-05-05
wiki_conversion_fallback: true
wiki_warnings: conversion-fallback
---

# vmfex

**NOTE: There is a new hook with better integration to vNic profile management** [UCS Integration](http://www.ovirt.org/Features/UCS_Integration)****

This hook enables the use of Cisco VM Fabric Extender (VM-FEX) in oVirt.

The hook receives a VM's virtual NIC's MAC adress as it is defined in the engine and a FEX Port Profile name, and reattaches that NIC to the FEX Port instead of the logical network it is assigned to in the Engine. This way the NIC MACs can be controlled in the Engine, but the NIC is actually running on FEX.

Since the only unique parameter passed from the engine to the VM is the MAC, a dictionary-like mapping of MAC to port profile will be used Sample:

     vmfex={'00:11:22:33:44:55:66':'Profile1',
           '00:11:22:33:44:55:67':'Profile2',...} (one line)

Will add 2 virtual nics attached to profile1 and profile2 using the vnic MAC addresses specified, replacing the actual NICs assigned to the VM.

A VM NIC with a MAC that is not mentioned in this dictionary will not be altered, and will remain attached to the bridge/logical network defined for it in the engine.

Libvirt internals (what the hook will do): Replace the existing interface xml:

with the following interface xml:

Note how <mac></mac> is preserved

The hook also defines a dynamic pool of VM-FEX dynamic NICs on every host. Libvirt internals: dynamic network with libvirt (define a NIC pool, so libvirt can assign VMs to NICs dynamically):

          <network>
            &lt;name&gt;direct&#45;pool&lt;/name&gt;
            &lt;forward mode=&quot;passthrough&quot;&gt;
              &amp;lt;interface dev=&amp;quot;eth3&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
              &amp;lt;interface dev=&amp;quot;eth4&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
              &amp;lt;interface dev=&amp;quot;eth5&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
              &amp;lt;interface dev=&amp;quot;eth6&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
              &amp;lt;interface dev=&amp;quot;eth7&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
              &amp;lt;interface dev=&amp;quot;eth8&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
              &amp;lt;interface dev=&amp;quot;eth9&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
              &amp;lt;interface dev=&amp;quot;eth10&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
              &amp;lt;interface dev=&amp;quot;eth11&amp;quot;&amp;gt;&amp;lt;/interface&amp;gt;
            &lt;/forward&gt;
          </network>

Using libvirt, the network is defined like this (the hook uses libvirt python API for the same purpose, and does this automatically):

       virsh net-define /tmp/direct-pool.xml
       virsh net-start direct-pool
       virsh net-autostart direct-pool

(where /tmp/direct-pool.xml contains the xml above)

Note that these interface definitions are completely static - you never need to modify them due to migration, or starting up/shutting down the guest.

Download link: <http://ovirt.org/releases/nightly/rpm/EL/6/hooks/vdsm-hook-vmfex-4.10.0-0.395.gite6d01e8.el7.noarch.rpm>