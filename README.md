# Usefull udev rules

commands:
```
$ udevadm monitor # basic events
$ udevadm monitor --property # "verbose" version
```

## Device renaming

Look up the MAC addressses for your devices:
```
$ ifconfig
```

/etc/udev/rules.d/10-network-interface-names.rules
```
# Rename network interfaces

SUBSYSTEM=="net", ATTR{address}=="00:11:22:33:44:55", NAME="wifi"
SUBSYSTEM=="net", ATTR{address}=="11:22:33:44:55:66", NAME="wired"
```


## MAC spoofing

/etc/udev/rules.d/70-mac-spoof.rules
```
# Set random ending to wifis MAC address
ACTION=="add", SUBSYSTEM=="net", DRIVERS=="iwlwifi", RUN+="/usr/local/sbin/mac-spoof.sh %k"
```

/usr/local/sbin/mac-spoof.sh
```
#!/bin/bash
# Spoof the ending of MAC address

readonly interface="$1"

if [[ -n $interface ]]; then
  ifconfig "$interface" down
  /usr/bin/macchanger -e "$interface"
  ifconfig "$interface" up
fi
```

## Monitor set up

Set up monitor when connected. Customize to your needs.

/etc/udev/rules.d/99-monitor-connect.rules:
```
# Set up VGA monitor when connected.

ACTION=="change", SUBSYSTEM=="drm", RUN+="/usr/local/sbin/monitor-connect.sh"
```

/usr/local/sbin/monitor-connected.sh:
```
#!/bin/bash
# Automatically set up VGA monitor on plug.

export DISPLAY=:0

readonly VGA_connected=$(xrandr | grep "VGA1 connected")

if [[ $VGA_connected ]]; then
  xrandr --output VGA1 --left-of LVDS1 --auto
else
  xrandr --output VGA1 --auto
fi
```
