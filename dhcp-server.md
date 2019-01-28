# Set up a DHCP Server for your LAN
## Overview
In this guide, we’ll take the first steps toward creating a Pi-based router by setting up a DHCP server to serve addresses and related configuration to client computers on the LAN.


1. Determine an address range for your local network
2. Configure a static address for your Pi
3. Install and configure DHCP provide addresses to local clients

> __Before you Begin:__ Complete initial setup of Raspbian and `systemd-networkd` and ensure that you can connect directly to the Pi via a point-to-point Ethernet link and that the Pi can connect to the Internet via wireless. 

## Details

**Determine a Local Address Range:** Select a base network from _RFC 1918_ private address allocations. Extend the network prefix (add more bits) to create a subnet that contains 64 addresses (including the network id and subnet broadcast). You will use these addresses in your initial static configuration and again within DHCP.

> __Note:__ Try to keep your subnet range distinct from your home network or the UW network. You will run into issues if you have the same address range on your Pi as is used on the wireless LAN that you use to access the Internet.

**Configure Static Addresses for your Raspberry Pi:** Pick an address from the network you defined to use as the static IP address for your Pi. Most administrators will place gateways on the first or last address of the subnet. Do not deviate from this convention without good reason.

Create a file named `20-eth0.network` in  `/etc/systemd/network/` (you'll need to use `sudo`) and define a static configuration that includes your chosen IP address and CIDR range. This path should already contain a default configuration for ethernet interfaces, e.g., `44-eth.network`. We are overriding this configuration for `eth0` by naming the file in such a way that it will appear earlier in sorted order and by ensuring that the configuration will match `eth0` exactly.

Your file should look something like:

```
[Match]
# We only want to match the eth0 interface
Name=eth0

# Provide full address config in abbreviated notation
[Network]
Address=172.16.4.1/28

# Enable link local addresses for IPv6 (FE80::)
LinkLocalAddressing=ipv6
```

> **Important:** You don't need to (and definitely shouldn't) add a gateway or name server on this interface. The Pi will get it’s DNS from the wireless interface, which received it’s DNS settings from DHCP. 

We'd usually need to set up a static address on our local device, but this shouldn’t be necessary since SSH will typically use our auto-configured IPv6 addresses to talk to the Pi. 

In order to test these new settings,call `sudo systemctl restart systemd-networkd.service` or reboot your Pi. Once you log back in, verify that `networkd` is running by calling `systemctl status systemd-networkd` and use the `ip addr` command to check that `eth0` is online with the new address.

> __Note:__ Raspbian tutorials will often instruct you to configure static addresses in the `/etc/dhcpcd.conf` file. While this is a reasonable solution for some applications, it is not compatible with the ISC DHCP Server. This tutorial assumes that `dhcpcd` has already been disabled.

### Install and Configure DHCP
For this part of the exercise, we'll use the ISC DHCP Server. The ISC server is available in apt repositories, so it can be installed using:

```bash
sudo apt update
sudo apt install isc-dhcp-server
```

Edit `/etc/default/isc-dhcp-server` to specify the interfaces (`eth0` only) on which to run the DHCP server. Comment out any options pertaining to DHCP for IPv6.

Using the resources provided or other tutorials, edit `/etc/dhcp/dhcpd.conf` and configure the DHCP server to distribute addresses within your predefined address range. 

> Note: The stock configuration provides plenty of example subnet declarations. If you are using the this file, you should comment out the lines at the top of the file that specify global settings for nameservers and domain (we won’t use them right now).

* You'll need to determine the network ID and subnet mask (in dotted decimal form) to add a network block into the dhcpd.conf file. 
* Configure a contiguous `range` of IP addresses for the DHCP pool out of the address range you selected earlier.
* Exclude your Pi's static IP address from the DHCP pool (we don't want to give it away).
* You should not specify any additional options within the DHCP block at this point in time. 

### Test DHCP Server
Use `systemctl` to restart the DHCP service as shown here:

`sudo systemctl restart isc-dhcp-server.service`

If you see errors such as the one shown here, you’ll need to continue troubleshooting the DHCP server (see next section).

```
pi@titan:~ $ sudo systemctl restart isc-dhcp-server.service
Job for isc-dhcp-server.service failed because the control process exited with error code.
See "systemctl status isc-dhcp-server.service" and "journalctl -xe" for details.
```

Check your local machine to confirm that an IP address in the specified network range was provided.

Try running `ping` from both directions (laptop -> pi and pi -> laptop) to confirm that the IP address and related settings are configured correctly.


### Troubleshooting ISC DHCP Server

Raspbian provides us with several commands to troubleshoot and find errors related to services. Check status and recent log output using `systemctl status isc-dhcp-server.service` or `journalctl -xe`  .

Search the system logs for relevant errors `journalctl -u isc-dhcp-server`

As you debug, keep the following points in mind:

* A misplaced space or bracket may cause DHCP to fail, so pay close attention to syntax.
* Your Pi will keep it's static address, so be sure that you excluded the address from the DHCP pool.
* Double check that you’ve configured the server defaults with the correct interface names and commented out IPv6 related settings.

### Resources
[Debian Network Configuration Documentation](https://wiki.debian.org/SystemdNetworkd) - Configuring network settings with `networkd`

[Raspberry Pi DHCP Server Tutorial](http://www.noveldevices.co.uk/rp-dhcp-server) - Configuring `isc-dhcp-server` (ignore the details about `/etc/network/interfaces` since we're using `networkd`)

