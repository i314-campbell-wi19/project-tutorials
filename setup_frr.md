# Install FRRouting (FRR) on Raspbian Stretch

## Review Project Homepage
`https://frrouting.org`

## Download appropriate package
`wget https://github.com/FRRouting/frr/releases/download/frr-6.0.2/frr_6.0.2-0.deb9u1_armhf.deb`

## Run installer 
`sudo dpkg -i frr_4.0-1.debian9.1_armhf.deb` 

## Resolve dependency-related errors
`sudo apt install -f`

## Grant permissions
`sudo usermod -aG frrvty pi`

## Enable required daemons
Edit `/etc/frr/daemons` and set `bgpd=yes`. Repeat this process for any other routing daemons you will be configuring (if applicable).

> In previous versions, we also needed to enable the Zebra daemon (handles basic network and routing configuration), but this is no longer necessary per the comments in the file.

## Restart FRRouting
`sudo systemctl restart frr`

## Launch VTY shell (starts in enable mode)
`vtysh`

## VTY Shell Basics
### Enter Configuration Mode
`configure terminal`

### Save Configuration (from enable mode)
`write memory` or `copy running-config startup-config`

### Show the Running Config (from enable mode)
`show running-config` or `write terminal`

### Exit the Current Mode
`quit`

### Shortcuts
Command abbreviations are accepted in VTY shell as long as the abbreviation is unambiguous. The shell will do its best to determine your intent and complete the command.

- `configure terminal` can be abbreviated `conf t`
- `quit` can be abbreviated `q`
- `write memory` can be abbreviated `w m` 
