Autofs Puppet Module
====================

[![Build Status](https://travis-ci.org/dhollinger/autofs-puppet.svg?branch=master)](https://travis-ci.org/dhollinger/autofs-puppet)
[![Puppet Forge](https://img.shields.io/puppetforge/v/dhollinger/autofs.svg)](https://forge.puppetlabs.com/dhollinger/autofs)

#### DEPRECATION WARNING
Release 1.4.5 will be the final feature release supporting Puppet 3.x.
Puppet 3.x is End of Life on Dec 31st, 2016 and as such version 1.4.x will remain
supported for maintenance until Jun 30th, 2017 at which point support will end.

Releases 2.0.0 and above will be compatible only with Puppet 4.x and later.

#### Table of Contents
1. [Description - - What the module does and why it is useful](#description)
2. [Setup - The basics of getting started with Autofs](#setup)
  * [The module manages the following](#the-module-manages-the-following)
  * [Requirements](#requirements)
  * [Incompatibilities](#incompatibilities)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
4. [Limitations - OS compatibility, etc](#limitations)
5. [Development - Guide for contributing to the module](#development)
6. [Support - When you need help with this module](#support)

Description
-----------
The Autofs module is a Puppet module for managing the configuration of automount
network file system. This is a global module designed to be used by any
organization. This module assumes the use of Hiera to set variables and serve up
configuration files.

Setup
-----
### The Module manages the following:
* Autofs package
* Autofs service
* Autofs Configuration File (/etc/auto.master)
* Autofs Map Files (i.e. /etc/auto.home)

### Requirements

* The [stdlib](https://forge.puppetlabs.com/puppetlabs/stdlib) Puppet Library
* The [concat](https://github.com/puppetlabs/puppetlabs-concat) Puppet Module

### Usage

The module includes a single class:

```puppet
include autofs
```

By default this installs and starts the autofs service with the default master
file. No parameters exist yet, but are in active development to allow for more
granular control.

To manage the service, use the following code in your profile:

```puppet
class { 'autofs':
  service_enable => false,
  service_ensure => 'stopped'
}
```

### Map Files

To setup the Autofs Map Files, there is a defined type that can be used:
```puppet
autofs::mount { 'home':
  mount       => '/home',
  mapfile     => '/etc/auto.home',
  mapcontents => ['* -user,rw,soft,intr,rsize=32768,wsize=32768,tcp,nfsvers=3,noacl server.example.com:/path/to/home/shares'],
  options     => '--timeout=120',
  order       => 01
}
```
This will generate content in both the auto.master file and a new auto.home map
file:

##### auto.master
```
/home /etc/auto.home --timeout=120
```

##### auto.home
```
* -user,rw,soft,intr,rsize=32768,wsize=32768,tcp,nfsvers=3,noacl server.example.com:/path/to/home/shares
```

The defined type requires all parameters, except direct and execute, to build the autofs config.
The direct and execute parameters allow for the creation of indirect mounts, see the Parameters section for more information on the defaults for direct and execute.

In hiera, there's a `autofs::mounts` class you can configure, for example:
```yaml
---
autofs::mounts:
  home:
    mount: '/home'
    mapfile: '/etc/auto.home'
    mapcontents:
      - '* -user,rw,soft,intr,rsize=32768,wsize=32768,tcp,nfsvers=3,noacl server.example.com:/path/to/home/shares'
    options: '--timeout=120'
    order: 01
```

##### Direct Map `/-` arugment

The autofs module also supports the use of the built in autofs `/-` argument used with Direct Maps.

###### Examples:

Define:
``` puppet
autofs::mount { 'foo':
  mount       => '/-',
  mapfile     => '/etc/auto.foo',
  mapcontents => ['/foo -o options /bar'],
  options     => '--timeout=120',
  order       => 01
}
```

Hiera:
``` yaml
---
autofs::mounts:
  foo:
    mount: '/-'
    mapfile: '/etc/auto.foo'
    mapcontents:
      - '/foo -o options /bar'
    options: '--timeout=120'
    order: 01
```

##### Autofs `+dir:` options

The autofs module now supports the use of the `+dir:` option in the auto.master.
This option is 100% functional, but does require some work to simplify it.

###### Usage

Define:
```puppet
autofs::mount { 'home':
  mount       => '/home',
  mapfile     => '/etc/auto.home',
  mapcontents => ['* -user,rw,soft,intr,rsize=32768,wsize=32768,tcp,nfsvers=3,noacl server.example.com:/path/to/home/shares'],
  options     => '--timeout=120',
  order       => 01,
  use_dir     => true
}
```

Hiera:
```yaml
---
autofs::mounts:
  home:
    mount: '/home'
    mapfile: '/etc/auto.home'
    mapcontents:
      - '* -user,rw,soft,intr,rsize=32768,wsize=32768,tcp,nfsvers=3,noacl server.example.com:/path/to/home/shares'
    options: '--timeout=120'
    order: 01
    use_dir: true
```

Reference
----------

#### Parameters
* **mount_name** - This is a logical, descriptive name for what what autofs will be
mounting. This is represented by the "home:" and "tmp:" entries above.
* **mount** - This Mapping describes where autofs will be mounting to. This map
entry is referenced by concat as part of the generation of the /etc/auto.master
file.
* **mapfile** - This Mapping describes the name and path of the autofs map file.
This mapping is used in the auto.master generation, as well as generating the map
file from the auto.map.erb template. This parameter is no longer required.
* **mapcontent** - This Mapping describes the folders that will be mounted, the
mount options, and the path to the remote or local share to be mounted. Used in
mapfile generation.
* **options** - This Mapping describes the auto.master options to use (if any)
when mounting the automounts.
* **order** - This Mapping describes where in the auto.master file the entry will
be placed. Order CANNOT be duplicated.
* **master** - This Parameter sets the path to the auto.master file. Defaults to
`/etc/auto.master`.
* **map_dir** - This Parameter sets the path to the Autofs configuration directory
for map files. Applies only to autofs 5.0.5 or later. Defaults to
`/etc/auto.master.d`.
* **use_dir** - This Parameter tells the module if it is going to use $map_dir.
Defaults to `false`.
* **direct** - Boolean to allow for indirect map. Defaults to true to be backwards compatible.
* **execute** - Boolean to set the map to be executable. Defaults to false to be backward compatible.
* **replace** - Boolean to set the map file to not be replaced. Defaults to true as Puppet File resource does.

Limitations
------------

#### Operating Systems

* Supported
    * Ubuntu 14.04/16.04
    * CentOS/RHEL/Oracle Linux 6.x/7.x
    * SLES 10 SP4/11 SP1/12
* Self Support - should work, support not provided by developer
    * Solaris 10, 11
    * Debian 7, 8
    * Fedora 23, 24
    * OpenSUSE
* Unsupported
    * Windows (Autofs not available)
    * Mac OS X (Autofs Not Available)

Development
-------------

Please see the [CONTRIBUTING.md](CONTRIBUTING.md) file for instructions regarding development environments and testing.

Support
-------

David Hollinger: [david.hollinger@moduletux.com](mailto:david.hollinger@moduletux.com)
