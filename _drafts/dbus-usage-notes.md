- [D-Bus usage notes](#d-bus-usage-notes)
  - [GUI tools](#gui-tools)
  - [Monitoring messages with dbus-monitor](#monitoring-messages-with-dbus-monitor)
  - [The busctl command](#the-busctl-command)
  - [The gdbus command](#the-gdbus-command)
  - [List services through CLI](#list-services-through-cli)
  - [Invoke a method](#invoke-a-method)
  - [Find the PID of the owner behind a service name](#find-the-pid-of-the-owner-behind-a-service-name)
  - [Find the PID of the owner behind a connection name](#find-the-pid-of-the-owner-behind-a-connection-name)
  - [Extract XML specification (the API in XML format)](#extract-xml-specification-the-api-in-xml-format)


# D-Bus usage notes

## GUI tools

- `qdbusviewer` - it allows you to send D-bus messages as well.
- `d-feet`- D-Bus object browser, viewer and debugger.

## Monitoring messages with dbus-monitor

The `dbus-monitor` command is used to monitor messages going through a D-Bus message bus.

Examples:

```sh
# monitor everything in session dbus
dbus-monitor

# Example of filtering method calls to org.lxqt.SingleApplication.activateWindow on service org.lxqt.lxqt-runner
dbus-monitor "type='method_call',destination='org.lxqt.lxqt-runner',path='/',interface='org.lxqt.SingleApplication',member='activateWindow'"
# Example of filtering signals to org.gnome.TypingMonitor
dbus-monitor "type='signal',sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```

## The busctl command

`busctl` may be used to introspect and monitor the D-Bus bus.

```sh
# get all the services and PIDs behind them
busctl --user list

# get all the objects
busctl --user tree <service.name>

# get the API
busctl --user introspect <service.name> </path/to/object>

# invoke a method
busctl --user call <service.name> </path/to/object> <interface> <method>
```

## The gdbus command

`gdbus` is another tool for working with D-Bus objects.

```sh
# monitor an object
gdbus monitor --session --dest <service.name> --object-path </path/to/object>
# example
#   gdbus monitor --session --dest  "org.lxqt.lxqt-runner" --object-path "/"

# get the API
gdbus introspect --session --dest <service.name> --object-path </path/to/object> --xml --recurse
# example
#   gdbus introspect --session --dest  "org.lxqt.lxqt-runner" --object-path "/" --xml --recurse

# invoke a method
gdbus call --session --dest <service.name> --object-path </path/to/object> --method <interface.method> ARG1 ARG2...
# example
#   gdbus call --session --dest  "org.lxqt.lxqt-runner" --object-path "/" --method org.lxqt.SingleApplication.activateWindow

# emit a signal
gdbus emit --session --dest <service.name> --object-path </path/to/object> --signal <interface.signal> ARG1 ARG2...
```

## List services through CLI

```sh
# using busctl:
busctl --user call org.freedesktop.DBus /org/freedesktop/DBus org.freedesktop.DBus ListNames

# using dbus-send:
dbus-send --print-reply --dest=org.freedesktop.DBus /org/freedesktop/DBus org.freedesktop.DBus.ListNames

# using qdbus:
qdbus org.freedesktop.DBus /org/freedesktop/DBus org.freedesktop.DBus.ListNames

# using gdbus
gdbus call --session --dest org.freedesktop.DBus --object-path /org/freedesktop/DBus --method org.freedesktop.DBus.ListNames
```

## Invoke a method

```sh
# Using busctl
busctl --user call <service.name> </path/to/object> <interface> <method>

# Using dbus-send
dbus-send --print-reply --dest=<service.name> </path/to/object> <interface.method>

# Using qdbus
qdbus <service.name> </path/to/object> <interface.method>

# Using gdbus
gdbus call --session --dest <service.name> --object-path </path/to/object> --method <interface.method>
```

## Find the PID of the owner behind a service name

The method `org.freedesktop.DBus.GetConnectionUnixProcessID` (signature `UINT32 GetConnectionUnixProcessID (in STRING bus_name)`) returns the Unix process ID of the process connected to the server. If unable to determine it (for instance, because the process is not on the same machine as the bus daemon), an error is returned.

The following examples shows the PID of the server behind the `org.freedesktop.Notifications` service.

```sh
# Using busctl
# command signature: busctl call SERVICE OBJECT INTERFACE METHOD [SIGNATURE [ARGUMENT...]]
busctl --user call \
  org.freedesktop.DBus \
  /org/freedesktop/DBus \
  org.freedesktop.DBus GetConnectionUnixProcessID s "org.freedesktop.Notifications"

busctl --user call \
  org.freedesktop.DBus \
  /org/freedesktop/DBus \
  org.freedesktop.DBus GetConnectionCredentials s "org.freedesktop.Notifications"

# Using dbus-send
# command signature: dbus-send --print-reply <service> <path> <interface.method> <args>
dbus-send --print-reply \
  --dest=org.freedesktop.DBus \
  / \
  org.freedesktop.DBus.GetConnectionUnixProcessID string:org.freedesktop.Notifications

dbus-send --print-reply \
  --dest=org.freedesktop.DBus \
  / \
  org.freedesktop.DBus.GetConnectionCredentials string:org.freedesktop.Notifications

# Using qdbus
qdbus \
  org.freedesktop.DBus \
  /org/freedesktop/DBus \
  org.freedesktop.DBus.GetConnectionUnixProcessID org.freedesktop.Notifications

qdbus \
  org.freedesktop.DBus \
  /org/freedesktop/DBus \
  org.freedesktop.DBus.GetConnectionCredentials org.freedesktop.Notifications

# Using gdbus
gdbus call --session \
  --dest org.freedesktop.DBus \
  --object-path / \
  --method org.freedesktop.DBus.GetConnectionUnixProcessID org.freedesktop.Notifications

gdbus call --session \
  --dest org.freedesktop.DBus \
  --object-path / \
  --method org.freedesktop.DBus.GetConnectionCredentials org.freedesktop.Notifications
```

## Find the PID of the owner behind a connection name

The following examples shows the PID of the server behind the `:1.14` connection.

```sh
# Using busctl
# command signature: busctl call SERVICE OBJECT INTERFACE METHOD [SIGNATURE [ARGUMENT...]]
busctl --user call \
  org.freedesktop.DBus \
  /org/freedesktop/DBus \
  org.freedesktop.DBus GetConnectionUnixProcessID s ":1.14"

# Using dbus-send
# command signature: dbus-send --print-reply <service> <path> <interface.method> <args>
dbus-send --print-reply \
  --dest=org.freedesktop.DBus \
  / \
  org.freedesktop.DBus.GetConnectionUnixProcessID "string::1.14"

# Using qdbus
qdbus \
  org.freedesktop.DBus \
  /org/freedesktop/DBus \
  org.freedesktop.DBus.GetConnectionUnixProcessID ":1.14"

# Using gdbus
gdbus call --session \
  --dest org.freedesktop.DBus \
  --object-path / \
  --method org.freedesktop.DBus.GetConnectionUnixProcessID ":1.14"
```

## Extract XML specification (the API in XML format)

Example using busctl:

```sh
# format
#   busctl --user call <service.name> </path/to/object> <interface> <method>
busctl --user call org.lxqt.lxqt-runner / org.freedesktop.DBus.Introspectable Introspect
```

Example using dbus-send:

```sh
# format:
#   dbus-send --print-reply --dest=<service.name> </path/to/object> <interface.method>
dbus-send --print-reply --dest=org.lxqt.lxqt-runner / org.freedesktop.DBus.Introspectable.Introspect
```

Example using qdbus:

```sh
# format:
#   qdbus <service.name> </path/to/object> <interface.method>
qdbus org.lxqt.lxqt-runner / org.freedesktop.DBus.Introspectable.Introspect
```

Example using gdbus

```sh
# format
#   gdbus call --session --dest <service.name> --object-path </path/to/object> --method <interface.method>
gdbus call --session --dest org.lxqt.lxqt-runner --object-path / --method org.freedesktop.DBus.Introspectable.Introspect
```
