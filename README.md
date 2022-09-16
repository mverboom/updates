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

To check a system for updates

## Upgrading systems
