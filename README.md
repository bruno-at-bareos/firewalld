[![Build status](https://github.com/firewalld/firewalld/workflows/testsuite/badge.svg)](https://github.com/firewalld/firewalld/actions/workflows/testsuite.yml?query=event%3Apush+workflow%3Atestsuite)
[![Integration status](https://github.com/firewalld/firewalld/workflows/integration%20testsuite/badge.svg)](https://github.com/firewalld/firewalld/actions?query=event%3Apush+branch%3Amaster+workflow%3Aintegration-testsuite)
[![Source status](https://github.com/firewalld/firewalld/workflows/source%20code%20and%20build%20checks/badge.svg)](https://github.com/firewalld/firewalld/actions?query=event%3Apush+branch%3Amaster+workflow%3Asource-checks)


FirewallD - A firewall daemon with D-Bus interface providing a dynamic firewall
====================

firewalld provides a dynamically managed firewall with support for network or 
firewall zones to define the trust level of network connections or interfaces. 
It has support for IPv4, IPv6 firewall settings and for ethernet bridges and a 
separation of runtime and permanent configuration options. It also provides an 
interface for services or applications to add ip*tables and ebtables rules 
directly. 


Development
-----------

To check out the source repository, you can use:

```sh
git clone https://github.com/firewalld/firewalld.git
```

This will create a local copy of the repository.

The contributor code of conduct can be found in [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)

Language Translations
---------------------

Firewalld uses GNU gettext for localization support. Translations can be done
using [Fedora's Weblate instance][1]. Translations are periodically merged into
the main firewalld repository.

[1]: https://translate.stg.fedoraproject.org/projects/firewalld/

Dependencies
------------

These are the runtime dependencies:

```
  linux >= 5.3
  python3-dbus
  python3-gobject
  python3-nftables >= 0.9.4
```

**Note**: python2 is _not_ supported.

Optional Dependencies
---------------------

These dependencies may enhance firewalld's functionality, but they are not
required.

```
  ebtables
  ipset
  iptables
  polkit
  python3-capng (libcap-ng-python3)
```

Working With The Source Repository
----------------------------------

In addition to the runtime dependencies some others are needed to build from
source:

```
  desktop-file-utils: /usr/bin/desktop-file-install
  gettext
  intltool
  glib2: /usr/bin/glib-compile-schemas
  glib2-devel: /usr/share/aclocal/gsettings.m4
  systemd-units
```

To be able to create man pages and documentation from docbook files:

```
  docbook-style-xsl
  libxslt
```

Use the usual autoconf/automake incantation to generate makefiles

```sh
./autogen.sh
./configure
```

You can use a specific python interpreter by passing the PYTHON variable. This
is also used by the testsuite.

```sh
./configure PYTHON=/path/to/python3
```

Use

```sh
make
```

to create the documentation and to update the po files.

Use

```sh
make check
```

to run the testsuite. Tests are run inside network namespaces and do not
interfere with the host's running firewalld. They can also be run in parallel
by passing flags to autotest.

```sh
make check TESTSUITEFLAGS="-j4"
```

The testsuite also uses keywords to allow running a subset of tests that
exercise a specific area.
For example:

```sh
make check TESTSUITEFLAGS="-k rich -j4"
```

Output:

```sh
  24: rich rules audit                                ok
  25: rich rules priority                             ok
  26: rich rules bad                                  ok
  53: rich rules audit                                ok
  23: rich rules good                                 ok
  55: rich rules bad                                  ok
  74: remove forward-port after reload                ok
```

You can get a list of tests and keywords

```sh
make -C src/tests check TESTSUITEFLAGS="-l"
```

Or just the keywords

```sh
make -C src/tests check TESTSUITEFLAGS="-l" \
  |awk '/^[[:space:]]*[[:digit:]]+/{getline; print $0}' \
  |tr ' ' '\n' |sort |uniq
```

There are integration tests. Currently this includes NetworkManager. These may
be _destructive_ to the host. Run them in a disposable VM or container.

```sh
make check-integration
```

There is also a check-container target that will run the testsuite inside
various podman/docker containers. This is useful for coverage of multiple
distributions. It also runs tests that may be destructive to the host such as
integration tests.

```sh
make check-container TESTSUITEFLAGS="-j4"
```

OCI Container Image
-------------------

As part of the `dist` build target an OCI container image is generated. This is
distributed alongside the normal release tarball. It can be used to run
firewalld from a container. The containerized firewalld will _not_ integrate
with the host (e.g. podman, libvirt, NetworkManager).

To manually load the container image into your environment:

```sh
podman load -i .../path/to/firewalld-oci-<ver>.tar
```

To fetch the image from quay.io:

```sh
podman pull quay.io/firewalld/firewalld:<ver>
```

where `<ver>` is optional, the latest version will be used if omitted.
                  
To start the daemon/container:

```sh
podman run -d --network host --privileged \
              --name my-firewalld firewalld
```

Firewalld's configuration will live inside the container. Therefore
users may want to occasionally `podman commit` the image.

Using firewalld's CLI should be done via podman exec after the
daemon/container has been started:

```sh
podman exec my-firewalld firewall-cmd ...
```

### Container Integration with Host

The same container image can be used to integrate with the host's running
NetworkManager, podman, libvirt, etc. This requires the host to have a dbus
policy for firewalld.

A dbus policy can be obtained from the firewalld source code tree at location
`config/FirewallD.conf`.

```sh
cp config/FirewallD.conf /usr/share/dbus-1/system.d/FirewallD.conf
```

Once the dbus policy is in place the container could be started as such:

```sh
podman run -d -v /run/dbus/system_bus_socket:/run/dbus/system_bus_socket \
              --network host --privileged \
              --name my-firewalld firewalld \
              firewalld --nofork --nopid
```

The only addition are: volume mount, explicit CMD.

The some approach can be use to store firewalld's configuration files on the
host.

```sh
podman run -d -v /run/dbus/system_bus_socket:/run/dbus/system_bus_socket \
              -v /etc/firewalld:/etc/firewalld \
              --network host --privileged \
              --name my-firewalld firewalld \
              firewalld --nofork --nopid
```

RPM package
-----------

For Fedora and RHEL based distributions, there is a spec file in the source
repo named firewalld.spec. This should be usable for Fedora versions >= 16 and
RHEL >= 7.


Links
-----

Homepage:          http://firewalld.org

Report a bug:      https://github.com/firewalld/firewalld/issues

Git repo browser:  https://github.com/firewalld/firewalld

Git repo:          https://github.com/firewalld/firewalld.git

Documentation:     http://firewalld.org/documentation/


Mailing lists
-------------

For usage:         https://lists.fedorahosted.org/archives/list/firewalld-users@lists.fedorahosted.org/

For development:   https://lists.fedorahosted.org/archives/list/firewalld-devel@lists.fedorahosted.org/


Directory Structure
-------------------

| Directory         | Content                                               |
| ----------------- | ----------------------------------------------------- |
| config/           | Configuration files                                   |
| config/icmptypes/ | Predefined ICMP types                                 |
| config/services/  | Predefined services                                   |
| config/zones/     | Predefined zones                                      |
| config/ipsets/    | Predefined ipsets                                     |
| doc/              | Documentation                                         |
| doc/man/          | Base directory for man pages                          |
| doc/man/man1/     | Man(1) pages                                          |
| doc/man/man5/     | Man(5) pages                                          |
| po/               | Translations                                          |
| shell-completion/ | Base directory for auto completion scripts            |
| src/              | Source tree                                           |
| src/firewall/     | Import tree for the sevice and all applications       |
| src/icons/        | Icons in the sizes: 16, 22, 24, 32, 48 and scalable   |
| src/tests/        | Testsuite                                             |
