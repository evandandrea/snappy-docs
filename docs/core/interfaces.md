---
layout: base
title: Interfaces
---

Interfaces allow snaps to communicate or share resources according to the
protocol established by the interface. Each connection has two ends, a "plug" (consumer) and a "slot" (provider).  A
plug and a slot can be connected if they use the same interface name.  The
connection grants necessary permissions for snaps to operate according to the
protocol.

For example, a snap using the camera can declare it requires the `camera` interface. On the other end of the connection, the core snap declares it provides the `camera` interface. When the interface gets connected, the snap gets read access to `/dev/video*`.

<img src="https://assets.ubuntu.com/v1/4d5afbf9-Snapcraft-Interfaces-plugs-and-slots.svg" alt="Interfaces - Plugs and Slots" style="width: 50%;"/>

Slots may support multiple connections to plugs.  For example the core snap
exposes the ``network`` slot and all applications that can talk over the
network connect their plugs there.

The availability of an interface depends on a number of factors and may be
may be provided by the core snap or via snaps providing the slot.  The
available interfaces on a given system can be seen with ``snap interfaces``.

## Supported Interfaces

A complete list of interfaces is provided in the [Interfaces reference](/reference/interfaces "Interfaces reference"). You can also see the list of interfaces available on a system and the snaps using them with `snap interfaces` or use the command to get more specific information, including:

- `snap interfaces <snap>` to find the slots offered and plugs used by the specified snap.
- `snap interfaces <snap>:<slot or plug>` for details of only the specified slot or plug.
- `snap interfaces -i=<interface> [<snap>]` to get a filtered list of  plugs and/or slots.

### Transitional interfaces

Most interfaces are designed for strong application isolation and user control
such that auto-connected interfaces are considered safe and users choose what
applications to trust and to what extent via manually connected interfaces.

Some interfaces are considered transitional to support traditional Linux
desktop environments and these transitional interfaces typically are
auto-connected. Since many of the underlying technologies in these environments
were not designed with strong application isolation in mind, users should only
install applications using these interfaces from trusted sources.  Transitional
interfaces will be deprecated as replacement or modified technologies that
enforce strong application isolation are available.

## Creating an interface

The OS snap exposes a number of interfaces to grant snaps access to system functions. You can extend this access by creating your own interfaces.

The following tutorial will show you how: [Your first interface](http://www.zygoon.pl/2016/08/creating-your-first-snappy-interface.html).

### Requesting an interface

You can also file an interface request by [opening a bug report](https://bugs.launchpad.net/snappy/+bugs?field.tag=snapd-interface) with the `snapd-interface` bug tag.

## Manually connecting interfaces

Interfaces may either be auto-connected by `snapd` on install or manually connected after
install.

To list the available connectable interfaces and connections:

    $ snap interfaces

To make a connection:

    $ snap connect <snap>:<plug interface> <snap>:<slot interface>

To disconnect snaps:

    $ snap disconnect <snap>:<plug interface> <snap>:<slot interface>

Consider a snap ``foo`` that uses ``plugs: [ log-observe ]``. Since
``log-observe`` is not auto-connected, ``foo`` will not have access to the
interface upon install:

    $ sudo snap install foo
    $ snap interfaces
    Slot                 Plug
    :log-observe         -
    -                    foo:log-observe

You may manually connect using ``snap connect``:

    $ sudo snap connect foo:log-observe core:log-observe
    $ snap interfaces
    Slot                 Plug
    :log-observe         foo:log-observe

and disconnect using ``snap disconnect``:

    $ sudo snap disconnect foo:log-observe core:log-observe
    $ snap interfaces # shows they are disconnected
    Slot                 Plug
    :log-observe         -
    -                    foo:log-observe

On the other hand, ``bar`` could use ``plugs: [ network ]`` and since
``network`` is auto-connected, ``bar`` has access to the interface upon
install:

    $ sudo snap install bar
    $ snap interfaces
    Slot                 Plug
    :network             bar:network

You may disconnect an auto-connected interface:

    $ sudo snap disconnect bar:network core:network
    $ snap interfaces
    Slot                 Plug
    :network             -
    -                    bar:network

Whether the slot is provided by the core snap or not doesn't matter in terms of
snap interfaces except that if the slot is provided by a snap, a snap that
implements the slot must be installed for it to be connectable. Eg, the
``bluez`` interface is not provided by the core snap so a snap author
implementing the bluez service might use ``slots: [ bluez ]``. Then after
install, the bluez interface shows up as available:

    $ sudo snap install foo-blue
    $ snap interfaces
    Slot                 Plug
    foo-blue:bluez       -

Now install and connect works like before (eg, ``baz`` uses
``plugs: [ bluez ]``):

    $ sudo snap install baz
    $ snap interfaces
    Slot                 Plug
    foo-blue:bluez       -
    -                    baz:bluez
    $ sudo snap connect baz:bluez foo-blue:bluez
    $ snap interfaces
    Slot                 Plug
    foo-blue:bluez       baz:bluez