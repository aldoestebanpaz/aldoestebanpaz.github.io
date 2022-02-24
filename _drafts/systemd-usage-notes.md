# Systemd usage notes

## Unit types

Use the following command for listing the available unit types.

```sh
systemctl --type=help
# my output was:
#   service
#   mount
#   swap
#   socket
#   target
#   device
#   automount
#   timer
#   path
#   slice
#   scope
```

## Unit states

Use the following command for listing the available unit states.

```sh
systemctl --state=help
# my output was:
#   Available unit load states:
#   stub
#   loaded
#   not-found
#   bad-setting
#   error
#   merged
#   masked
#
#   Available unit active states:
#   active
#   reloading
#   inactive
#   failed
#   activating
#   deactivating
#   maintenance
#
#   Available unit file states:
#   enabled
#   enabled-runtime
#   linked
#   linked-runtime
#   alias
#   masked
#   masked-runtime
#   static
#   disabled
#   indirect
#   generated
#   transient
#   bad
#
#   Available automount unit substates:
#   dead
#   waiting
#   running
#   failed
#
#   Available device unit substates:
#   dead
#   tentative
#   plugged
#
#   Available mount unit substates:
#   dead
#   mounting
#   mounting-done
#   mounted
#   remounting
#   unmounting
#   remounting-sigterm
#   remounting-sigkill
#   unmounting-sigterm
#   unmounting-sigkill
#   failed
#   cleaning
#
#   Available path unit substates:
#   dead
#   waiting
#   running
#   failed
#
#   Available scope unit substates:
#   dead
#   running
#   abandoned
#   stop-sigterm
#   stop-sigkill
#   failed
#
#   Available service unit substates:
#   dead
#   condition
#   start-pre
#   start
#   start-post
#   running
#   exited
#   reload
#   stop
#   stop-watchdog
#   stop-sigterm
#   stop-sigkill
#   stop-post
#   final-watchdog
#   final-sigterm
#   final-sigkill
#   failed
#   auto-restart
#   cleaning
#
#   Available slice unit substates:
#   dead
#   active
#
#   Available socket unit substates:
#   dead
#   start-pre
#   start-chown
#   start-post
#   listening
#   running
#   stop-pre
#   stop-pre-sigterm
#   stop-pre-sigkill
#   stop-post
#   final-sigterm
#   final-sigkill
#   failed
#   cleaning
#
#   Available swap unit substates:
#   dead
#   activating
#   activating-done
#   active
#   deactivating
#   deactivating-sigterm
#   deactivating-sigkill
#   failed
#   cleaning
#
#   Available target unit substates:
#   dead
#   active
#
#   Available timer unit substates:
#   dead
#   waiting
#   running
#   elapsed
#   failed
```

## Services

### List al services

To list all loaded services on your system:

```sh
systemctl --type=service
# or
# systemctl list-units --type=service
```

This includes units that are either referenced directly or through a dependency,
units that are pinned by applications programmatically, or units that were active
in the past and have failed.

By default only units which are active, have pending jobs, or have failed are shown.

When listing units with `--all`, it also shows inactive units and units which are following other units:

```sh
systemctl --type=service --all
# or
# systemctl list-units --type=service --all
```

### List services with specific statuses

```
systemctl --type=service --state={STATE}
# or
# systemctl list-units --type=service --state={STATE}
```

{STATE} is a comma-separated list of unit LOAD (unit load), SUB (specific unit type), or ACTIVE (unit active) states. When listing units, show only those in the specified states.

Given that we are filternig services, {STATE} can be one of the following.

Unit LOAD states:
- stub
- loaded
- not-found
- bad-setting
- error
- merged
- masked

Unit ACTIVE states:
- active
- reloading
- inactive
- failed
- activating
- deactivating
- maintenance

Service unit SUBstates:
- dead
- condition
- start-pre
- start
- start-post
- running
- exited
- reload
- stop
- stop-watchdog
- stop-sigterm
- stop-sigkill
- stop-post
- final-watchdog
- final-sigterm
- final-sigkill
- failed
- auto-restart
- cleaning

### Enable a service

```sh
sudo systemctl enable <UNIT_NAME>
# or
sudo systemctl enable <UNIT_PATH>
```

### Start a service

```sh
sudo systemctl start <UNIT_NAME>
# or
sudo systemctl start <UNIT_PATH>
```

### Get service status

```sh
sudo systemctl status <UNIT_NAME>
# or
sudo systemctl status <UNIT_PATH>
```

### Reload service with new configuration

First make systemd aware of the new configuration:

```sh
systemctl daemon-reload
```

Then restart (or reload) your service:

```sh
systemctl restart <UNIT_NAME>
# or
systemctl restart <UNIT_PATH>
```

### View logs

```sh
sudo journalctl -fu <UNIT_NAME>
# or
sudo journalctl -fu <UNIT_PATH>
```

Where:
- `-f`, `--follow` - Show only the most recent journal entries, and continuously print new entries as they are appended to the journal.
- `-u`, `--unit=UNIT|PATTERN` - Show messages for the specified systemd unit UNIT (such as a service unit), or for any of the units matched by PATTERN.

For further filtering, time options such as `--since today`, `--until 1 hour ago`, or a combination of these can reduce the number of entries returned.

```sh
sudo journalctl -fu <UNIT_NAME> --since "2016-10-18" --until "2016-10-18 04:00"
# or
sudo journalctl -fu <UNIT_PATH> --since "2016-10-18" --until "2016-10-18 04:00"
```

## List unit files

To list all units installed in the file system, use the list-unit-files command instead.

