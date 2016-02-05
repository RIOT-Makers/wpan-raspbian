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
# cp <path/to/repo/clone>/usr/local/sbin/*.sh /usr/local/sbin/.
```

### Create lowpan device

Create a 6LoWPAN device named `lowpan0` with distinct `CHANNEL` and `PANID`.
This script will also take down an existing `monitor0` device and will update
the _channel_ and _pan id_ of `lowpan0` if it already exists. This script is
used by the systemd lowpan service described below to create a device on startup.
```
# create_lowpan <CHANNEL> <PANID>
```

_Note_: this script will also unload the kernel module for UDP header
compression `nhc_udp`, as this is not supported by the RIOT-OS (yet).

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

Delete an existing &LoWPAN monitoring device named `monitor0`.
```
# delete_monitor
```

## Systemd service

For _real_ IoT deployment you typically want to have your 6LoWPAN devices
created and configured during the boot process. Raspbian Jessie uses _Systemd_
to run services, so we want to use that to init a lowpan devices on startup.

The `lowpan.service` file provides a service definition to create `lowpan0`. It
requires the file `/etc/default/lowpan` to specify _channel_ (`CHN`) and
_panid_ (`PAN`); Further it uses the scripts `create_lowpan` and `delete_lowpan`
as described above, so install them as well.

To install the service, copy the files
```
# cp <path/to/repo/clone>/etc/default/lowpan /etc/default/.
# cp <path/to/repo/clone>/etc/systemd/lowpan.service /etc/systemd/system/.
# systemctl enable lowpan.service
```
_Note_: modify _channel_ and _panid_ if required in `/etc/default/lowpan`.

----

[Wiki]: https://github.com/RIOT-Makers/wpan-raspbian/wiki/Create-a-generic-Raspbian-image-with-6LoWPAN-support
