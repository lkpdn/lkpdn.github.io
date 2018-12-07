---
layout: single
title: _About QEMU Rocker device implementation_
date: 2018-12-07 21:08:23 +0900
categories: linux
---
QEMU rocker device was introduced by Scott Feldman, see the [paper](http://wiki.netfilter.org/pablo/netdev0.1/papers/Rocker-switchdev-prototyping-vehicle.pdf) for Netdev 0.1 ([https://www.netdevconf.org/0.1/sessions/2.html](https://www.netdevconf.org/0.1/sessions/2.html)), Quote:
> Rocker is an emulated network switch platform created to accelerate development of an in­kernel network switch driver model. Rocker has two parts: an Qemu emulation of a 62­port switch chip with PCI host interface and a Linux device driver. The goal is to emulate capabilities of contemporary network switch ASICs used in data­center/enterprise so the community can develop the switch device driver interfaces in the kernel.

Rocker device kernel module is a [switchdev](https://www.kernel.org/doc/Documentation/networking/switchdev.txt) driver, and the "World" inside of it implements some characteristic features with corresponding "World" implementation inside of QEMU userland backend device emulation, assuming the possibly forthcoming switch device ASIC. Note: OF-DPA (OpenFlow Data Plane Abstraction) World is the only choice in the upstream at present.

As a PCI device backend, QEMU userland prepares things in the same way as it does for all the other kinds. Ring structures are for (i). a command ring into which the kernel module enqueues commands, (ii). a event ring into which the QEMU userland enqueues events to notify and (iii). TX/RX rings.

[![0007-qemu-rocker-ring-seen-from-both-sides.png](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0007-qemu-rocker-ring-seen-from-both-sides.png){:class="img-responsive"}](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0007-qemu-rocker-ring-seen-from-both-sides.png)

and for each ring, QEMU userland allocates MSI-X vectors. How those are set up is:

[![0008-qemu-rocker-msix-setup-overview.png](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0008-qemu-rocker-msix-setup-overview.png){:class="img-responsive"}](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0008-qemu-rocker-msix-setup-overview.png)

As mentioned above, currently OF-DPA World is the only choice. So let's see what the switchdev notifications are like in the OF-DPA World's context. Note that the "operation" in the next figure is what I call it for clarity, not a formal way to call it.

[![0009-qemu-rocker-switchdev-notifications.png](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0009-qemu-rocker-switchdev-notifications.png){:class="img-responsive"}](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0009-qemu-rocker-switchdev-notifications.png)

How the admin operation on FDB are propagated to switchdev and how the FDB the device has learned are notified upwards are now clear, isn't it?
Lastly I offer you the whole picture of how the OF-DPA processing sometimes bypasses the upward delivery, sometimes not, with the entire callgraph.

[![0010-qemu-rocker-ofdpa-processing-overview.png](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0010-qemu-rocker-ofdpa-processing-overview.png){:class="img-responsive"}](https://raw.githubusercontent.com/lkpdn/images/master/imgs/0010-qemu-rocker-ofdpa-processing-overview.png)
