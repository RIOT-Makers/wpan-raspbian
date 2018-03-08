# wpan-raspbian
Tools for a WPAN enabled Raspbian

## Overview

To ease deployment the folder structure in this repo matches root filesystem
layout of Raspbian - just copy files as need to the respective location. They
depend on a Raspbian Jessie running a WPAN enabled Linux Kernel with installed
WPAN tools.

Take a look at our [Wiki] for a guide to create a WPAN enabled Raspbian image.

_Note_: install or running these scripts and services requires _root_ or _sudo_
privileges!

## Helper scripts

The following shell scripts are little helper to create and delete lowpan or
monitor devices. To install the scripts copy them as follows:

```
# cp <path/to/repo/clone>/usr/local/sbin/* /usr/local/sbin/.
# chmod +x /usr/local/sbin/*
```

### Create lowpan device

Create a 6LoWPAN device named `lowpan0` with distinct `CHANNEL` and `PANID`.
This script will also take down an existing `monitor0` device and will update
the _channel_ and _pan id_ of `lowpan0` if it already exists. This script is
used by the systemd lowpan service described below to create a device on startup.
```
# create_lowpan <CHANNEL> <PANID> [<LLADDR>]
```
_Note_: optionally this scripts allows to modify/set the LLADDR of the lowpan
device, as some devices generate a new LLADDR at each boot.

### Delete lowpan device

Delete an existing 6LoWPAN device named `lowpan0`. This script is used
by the systemd lowpan service described below to take down the device.
```
# delete_lowpan
```

### Create monitor device

Create a 6LoWPAN monitoring device named `monitor0` with distinct `CHANNEL`.
This script will also take down an existing `lowpan0` device and will update
the _channel_ of an existing `monitor0`.
```
# create_monitor <CHANNEL>
```

### Delete monitor device

Delete an existing 6LoWPAN monitoring device named `monitor0`.
```
# delete_monitor
```

## Systemd service

For _real_ IoT deployment you typically want to have your 6LoWPAN devices
created and configured during the boot process. Raspbian Jessie uses _Systemd_
to run services, so we want to use that to init a lowpan devices on startup.

### LowPAN setting at startup
The `lowpan.service` file provides a service definition to create `lowpan0`. It
requires the file `/etc/default/lowpan` to specify _channel_ (`CHN`), _panid_
(`PAN`), and (optional) the _link layer address_ (`MAC`). Further it uses the
scripts `create_lowpan` and `delete_lowpan` as described above, so install them
as well.

To install the service, copy the files
```
# cp <path/to/repo/clone>/etc/default/lowpan /etc/default/.
# cp <path/to/repo/clone>/etc/systemd/system/lowpan.service /etc/systemd/system/.
# systemctl enable lowpan.service
```
_Note_: modify _channel_ and _panid_ if required in `/etc/default/lowpan`.

### LowPAN and a route setting at startup

When the Pi should not only span the lowPAN but also make the devices accessible 
from other host in the network the `lowpan_route` service can be used instead of the 
`lowpan service`. Make sure to only have one of them in `/etc/systemd/system/`.

The `lowpan_route.service` file provides a service definition to create `lowpan0`. It
requires the file `/etc/default/lowpan` to specify _channel_ (`CHN`), _panid_
(`PAN`), and (optional) the _link layer address_ (`MAC`). Further it uses the
scripts `create_lowpan` and `delete_lowpan` as described above, so install them
as well.

Additional it uses the skript `interface_add_IP_route` which sets an IP to an 
interface and enables routing of the traffic from the lowPAN to the specified 
interface by adding a route.
It requires the file `/etc/default/lowpan` to specify an _interface_ (`INTERFACE`),
a _IPv6 to add_ (`ADD_IPv6`), and the _route to the lowpan_ (`ROUTE_PREFIX`).
Further it uses the scripts `interface_add_IP_route` and `interface_del_IP_route`,
so install and make them executeable as well.

To install the service, copy the files
```
# cp <path/to/repo/clone>/etc/default/lowpan /etc/default/.
# cp <path/to/repo/clone>/etc/systemd/system/lowpan_route.service /etc/systemd/system/.
# systemctl enable lowpan_route.service
```

## Router Advertisement Daemon (radvd)

The 6LoWPAN standard states that there must be at least one router (6LR) in a
network answering router solicitations (RS) by responding with router
advertisements (RA). Typically, a rather _strong_ machine like the Pi would also
act as the _Authoritative Border Router_  and gateway (6LBR) between LoWPAN and
_real_ IPv6 networks such as the Internet. A simple way to transform the Pi
in a 6LBR is to run `radvd`. More detailed information can be found [here].

A radvd configuration file can be found in `wpan-raspian/etc/radvd.conf` it is 
configured for `lowpan0` and `wlan0` to be used in a setup where the Pi bridges the 
lowpan and the wlan and also advertises the prefixes for both networks.
This setup can be used to span a secondary network in parrallel to the running
network without affecting an existing network setup.
The additional hosts can join the network by adding an IP with the defined prefix.
Then they can ping the Pi, when they add a route to reach the lowpan over teh Pi other 
host can then ping the lowpan devices through the Pi.

For example the Pi advertises

```
lowpan: fe18:a:b:2::/64
wlan:   fe18:a:b:1::/64
```

Other linux host can join the network and ping the lowPAN devices like this.
```
sudo ifconfig wlan0 add fd18:a:b:1::2/64
ping fd18:a:b:2:dedc:20b7:3c21:8b3e
```


[Wiki]: https://github.com/RIOT-Makers/wpan-raspbian/wiki/Create-a-generic-Raspbian-image-with-6LoWPAN-support
[here]: https://github.com/RIOT-Makers/wpan-raspbian/wiki/Setup-native-6LoWPAN-router-using-Raspbian
