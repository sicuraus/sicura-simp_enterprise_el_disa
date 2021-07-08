# simp_enterprise_el_disa

-------------------------------------------------------------------------------
> Copyright Onyx Point, Inc - All Rights Reserved
-------------------------------------------------------------------------------

SIMP Compliance Engine data for DISA STIG benchmarks.

The following DISA STIG benchmarks are currently supported:

* Red Hat Enterprise Linux 7 V3R2
* Oracle Linux 7 V2R2
* Red Hat Enterprise Linux 8 V1R0-3 (draft)

## Enabling DISA STIG enforcement

In your environment-level `hiera.yaml`, add a layer for `compliance_markup::enforcement`.

```yaml
hierarchy:
  - name: SIMP Compliance Engine
    lookup_key: compliance_markup::enforcement
```

At a Hiera layer that applies to the nodes where you are enforcing the
compliance profile, add a `compliance_markup::enforcement` array. For
example, to enforce `disa:mac-1:classified`, use the following:

```yaml
compliance_markup::enforcement:
  - disa:mac-1:classified
```

The following compliance profiles are available:

* `disa:mac-1:classified`
* `disa:mac-1:public`
* `disa:mac-1:sensitive`
* `disa:mac-2:classified`
* `disa:mac-2:public`
* `disa:mac-2:sensitive`
* `disa:mac-3:classified`
* `disa:mac-3:public`
* `disa:mac-3:sensitive`

## User Required Activities

There are a few items that the user must configure since there is no method for
automatically determining the appropriate values.

### Bootloader Username/Password

DISA STIG requires all systems to have a bootloader username and password set.
To accomplish this, first run `grub2-mkpasswd-pbkdf2` to generate a hashed grub
password. Once that is complete, place the following in the hieradata for the
system(s) being managed:

```yaml
simp_grub::admin: <grub_username>
simp_grub::password: <hashed_password>
```

### Disk Partitioning

The SIMP ISO installation will appropriately configure the disk per the DISA STIG
requirements. Alternatively, the kickstart files provided as templates by the
SIMP project can be used to kickstart a system.

If neither of these options are chosen, the user will need to set the
partitions appropriately themselves.

### Package Updates

DISA STIG requires that all security patches be applied to a system. SIMP makes no
assumptions about the connectivity to a given set of repositories or the
appropriateness of applying any given updates to your system.

That said, nightly updates are enabled via the `simp::yum::schedule` class and
will update all packages to the latest version based on whatever repositories
are present and configured on the system at that time.

### High I/O Activities

Some checks that require scanning the entire filesystem (such as checking for
incorrect permissions) would put a generally unacceptable load on the system
in order to remediate at each run of Puppet. Users are encouraged to scan
their systems regularly to find and address these kinds of issues.

If a suitable lightweight mechanism can be found to address this issue in the
future, it will be added to the SIMP module space.

### Graphical Display

The DISA STIG benchmark indicates that no X11 packages should be installed if you do
not require an actual graphical system display. Unfortunately there is no good
method to determine exactly which packages are part of a required installation
and which are needed by other applications as part of their dependency chain.
As such, users should start with the default SIMP installation (no GUI) and
then use the `simp-gnome` and `simp-gdm` modules to add a GUI to their system
as required.

### Remote Rsyslog Configuration

Given that there is no way to know where and/or if a syslog server may be
present on the network, there are no remote syslog servers configured by
default.

To bring your system into compliance, set the following via Hiera:

```yaml
simp_options::syslog::log_servers:
  - log.server.one
```

To set failover log servers, set the following via Hiera:

```yaml
simp_options::syslog::failover_log_servers:
  - failover.server.one
```
