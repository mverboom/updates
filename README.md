## Updates

A helper script to aid in checkin systems for updates and installing them. Can
create reports and publish them on a wiki.

This script has (basic) support for:
* Debian and related systems
* RedHat and related systems

The script was mostly tested on Debian, so quality of support for other distributions
may vary.

### Background

This script performs two functions
* Check if there are packages available with updates
* Install packages with updates

For both actions, it is possible to generate a brief report that can be published
on a Wiki.

All connectivity to the systems that need to be checked or upgraded goes through
ssh. The script does not do any specific setup for the ssh connection, this should
be validated beforehand.

The script tries to recognise proxmox systems. If there are any running containers
on a system, the defined action will also be executed for all containers.

This should allow for some streamlining in keeping environments up to date.

The script relies on three shell libraries:
* libfrmt
* libwiki
* libupdates

These need to be available in order for the script to work.

### Caching

The script heavily relies on caching to increase performance. This means that
not all actions always have the intended effect. There are options to force and
ignore the cache even if it is not expired. This might be required when running
actions back to back.

### Configuration

A configuration file can be created with .ini style formatting. There are several
sections that configure the script.

#### [main]

In section main, the following configuration keywords are available.

```wikiprofile```

Name of the profile to use when publishing information on a wiki. The profile points
to the profile configuration as defined for libwiki.

```checkreport```

The value of this option is the name of the page to use when generating a report for
available updates on hosts and publishing it on a wiki.


```upgradereport```

The value of this option is the name of the page to use when generating a report for
upgrade results for hosts and publishing it on a wiki.

```allhostslist```

This is a space seperated list of hosts. When not specifying hosts on the commandline
this list will be used as a definition of all hosts.

```allhostscmd```

As an alternative to the statically defined allhostslist the value of this key is executed
and the result is used as the value for the all hosts definition.

```upgradeblacklist```

This is a space seperated list of hosts that should not be upgraded. This list is compared
against host names and container names.

```upgradeblacklistcmd```

As an alternative to the statically defined upgradeblacklist the value of this key is
executed and the result is used as the value for the upgrade blacklist.

```parallelhost```

When parallelism is enabled, the amount of hosts that should be run in parallel. This is
seperate from the amount of parallelism for containers on the specific host. If this is
not specified and parallelism is enabled, the value defaults to 4.

```parallelct```

When parallelism is enabled, the amount of containers on a host that should be run in
parallel. If this is not specified and parallelism is enabled, the value defaults to 2.

#### [hostsectionnames]

Within this part of the configuration, the sections can be defined that should be
used when generating reports.

There are no set keywords that should be used in this section. The keywords form
the identifier for a section. The keywords are placed in alphabetical order in the
report. The value assigned to each keyword is used as the name of the section.

Each keyword used in this section should also be defined in either the
```hostlists``` or ```hostcmds``` sections. For more info, see below.

For example:
```
hardware=Hardware systems
cts=LXC containers
```

#### [hostlists]

This section allows for the keywords used in the ```hostsectionnames``` to be
associated with a specific list of hostnames.

For example:
```
hardware=px1.example.com px2.example.com
cts=lxc1.example.com lxc2.example.com lxc3.example.com
```

#### [hostcmds]

This section allows for the keywords used in the ```hostsectionnames``` to be
associated with a command that generates a list of hostnames to be used for the
setion.

For example:

```
hardware=cdist inventory list -H -a -t online
cts={ cat /etc/cts-vm1 /etc/cts-vm2 | sort -u }
```

## Checking for updates

It is possible to either check all known systems (based on the configured value)
or some specific ones.
The most simple action is:

```updates check```

Or for a specific host:

```updates check host1.example.com```

This will check for updates on all systems. However, if the system has been checked
before and the output was cached and isn't expired yet, the check will not be
performed.
To force a check, even if there is cache output, use the force option:

```updates check -f```

If any of the systems are Proxmox systems, any running containers on these systems
will also be checked for updates. It is possible to disable the behaviour with:

```updates check -e all```

Instead of all, it is also possible to define one or more (seperated by comma's)
container id's that should be excluded. These container id's will be used for any
of the hosts that will be checked. So if there are multiple Proxmox systems with
container id 100, then all of those containers will be excluded.

In order to speed up the process it is possible to use parallelism. This will be
used on 2 levels:
* Number of parallel hosts (defaults to 4)
* Number of parallel containers (defaults to 2)

The number of parallel containers is a limit for each host. So running two Proxmox
servers in parallel will run 2 containers in parallel on each host.

```updates check -p```

## Reporting updates

Based on the information stored in the cache, a report can be generated and
published on a wiki.

```updates check-report```

If there are systems with packages that are marked on-hold, these can be
excluded in order to make the report more focussed on any updates that can
be installed:

```updates check-report -H```

## Upgrading systems

To upgrade systems, it is again possible to do this for all hosts based on the
configuration or for a specific host.

```updates upgrade```

When upgrading a system, a check is first done if there are any packages that
need to be upgraded. This is check on the cached information done by the last
update check. In order to force an update of this list, the force option should
be used:

```updates upgrade -f```

To prevent having to download all packages during an upgrade window, it is possible
to pre download all packages that have upgrades available, but not yet install them.

```updates upgrade -d```

All the same include and exclude options that are available for the check command
are also available for de upgrade command. To upgrade a Proxmox server, but not
all the containers:

```updates upgrade -e allproxmox.example.com```

To update all the containers on a Proxmox server, but not the proxmox server
itself:

```updates upgrade -E proxmox.example.com proxmox.example.com```

Just like with the check command, the upgrade command has the same parallelism
options.

## Reporting upgrades

Based on the information stored in the cache, a report can be generated and
published on a wiki.

```updates upgrade-report```

Any systems or containers that have had issue's during an upgrade will not be
marked with (ok) after their name.

Currently there are two markings that are used for problems during an upgrade:

* FAIL

This indicates the command to run the upgrade exited with a non 0 exit status.
For more details concering the problem, please check the output included in the
details for the specific system.

* CONFIG CHANGE

This indicates the command to run the upgraded exited ok, but one or more pacakges
requested information on what to do with a specific configuration file. Per default
the action is to not change any configuration file. This usually is the safest
option, but could in some cases give undesired results.
