# Setup pfSense with TM Unifi [Malaysia ISP]

Since I learn from my mistake why I was unable to connect my WAN connection to TM Unifi ISP because of 1 tick settings. Here I want to share by default to setup pfSense with TM Unifi.

## Pre-requisite

1. PfSense installed in any hardware. If you do not installed yet, you can check out this [documentation](https://docs.netgate.com/pfsense/en/latest/install/install-walkthrough.html) from Netgate.

2. Already go through pfSense wizard initial setup.

## Configure WAN using PPPoE

Since mostly consumers using TM Unifi will be provided with PPPoE credentials to use connect to Internet. This is configure mostly by their technical staff when we're first installed home fiber at out home.

Now this is are mostly current architecture for home network.

![Network architecture for home](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vgvktong6ab6twzfobdf.png)

As you can see above diagram, we want to use pfSense as our main router and firewall and another TM Unifi Router we can use as Access Point.

![Network architecture using pfSense](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s47lgca3v8x2no48xged.png)

**Step 1**:
So the first thing you need to do, go to pfSense WebGUI and login.

**Step 2**:
Go to Interfaces -> Assignments -> VLANs Tab

 Add new VLAN Tag 500,

> Note: TM Unifi using VLAN 500 for connectivity to Internet and for HyppTV is VLAN 600.

```bash
Parent Interface: <your_WAN_interface>
VLAN tag: 500
VLAN Priority: Blank
Description: TM UNIFI
```
and then click Save.

**Step 3**: 
Go to Interface Assignments Tab, on 'WAN' interface, edit Network port

```bash
VLAN 500 on <parent interface> (TM UNIFI)
# <parent_interface> that you configure on VLAN section.
```
and then click Save.

**Step 4**:
Click on 'WAN' Interface

```bash
# General Configuration

Enable: Yes # Enable interface
Description: WAN_TMUNIFI
IPv4 Configuration Type: PPPoE
IPv6 Configuration Type: None
MTU: 1480
MSS: Blank

# PPPoE Configuration

## contact TM Support Center for these details
Username: # Your PPPoE username
Password: # Your PPPoE Password
Service name: Blank
Host-Uniq; Blank
Dial on demand: Yes # Enable Dial-On-Demand mode
Idle timeout: 0
Periodic reset: Disabled

## Yeah the settings I miss is Dial on demand. We need to enable that for PPPoE connection to work.

# Reserved Networks

Block private networks and loopback addresses: Yes
Block bogon networks: Yes
```

and click Save and Apply Changes.

**Step 5**:
Verify your PPPoE connections. Go to Status -> Interfaces

> Note: If you see your static or dynamic public IP address is correct and status and PPPoE is up then you're good. If you're not, ensure VLAN and PPPoE credentials is configure properly.

Now your PfSense is connected into Internet. Hooray.

## Configure LAN using VLAN

Since, we're now using default TM Unifi Router as Access Points, let's create VLAN for our home network. For this example, we're using VLAN 30.


**Step 1**:
Go to Interfaces -> Assignments -> VLANs Tab

 Add new VLAN Tag 30,

```bash
Parent Interface: <your_LAN_interface>
VLAN tag: 30
VLAN Priority: Blank
Description: Home Network.
```
and then click Save.

**Step 2**:
Go back to Interface Assignments Tab, click `+ Add` new Network Port into Interface Assignments.
Choose our VLAN 30 that we've created before. Then click Save.

**Step 3**:
Go to 'OPT1' or any OPT(ID) that has VLAN 30, configure the interface

```bash
# General Configuration

Enable: Yes # Enable interface
Description: VLAN30
IPv4 Configuration Type: Static IPv4
IPv6 Configuration Type: None
MAC Address: Default
MTU: Blank
MSS: Blank
Speed and Duplex: Default

# Static IPv4 Configuration

IPv4 Address: 192.168.30.1/24
IPv4 Upstream gateway: None
# Note: None - since its on LAN it will use what's on WAN interface gateway.

# Reserved Networks:

Block private networks and loopback addresses: No
Block bogon networks: No
```

and click Save and Apply Changes.

Good, now you have setup VLAN 30 for your home network and let's create DHCP Server for VLAN 30 for Access Points to distribute automatically IP addresses.

## Setup DHCP Server for VLAN 30

An automatic distribution and assignment of IP addresses, default gateways, and other network characteristics to client devices is performed by a DHCP server, a type of network server.

**Step 1**:
Go to Services -> DHCP Server. Then go to VLAN30 DHCP Server

**Step 2**:
Under VLAN30 DHCP Server settings, configure

```bash
# General Options

Enable: Yes # Enable DHCP server on VLAN30 interface
BOOTP: No
Deny unknow clients: Allow all clients
Ignored denied clients: No
Ignore client identifiers: No
Subnet: 192.168.30.0
Subnet mask: 255.255.255.0
Available Range: 192.168.30.1 - 192.168.30.254
Range: 192.168.30.11 - 192.168.30.254
# Note: Reserved first 10 IP address in the subnet for backup purposes. This IP addresses can be used for static IP for our Access Points or Management Devices.

# Additional Pools
# - Leave it is as default

# Servers

WINS servers: Default
DNS servers: 
8.8.8.8
1.1.1.1

# OMAPI
# - Leave it is as default

# Other Options

Gateway: 192.168.30.1 # IP VLAN 30 on LAN interface
Domain name: Blank
Domain search list: Blank
Domain lease time: Blank
Maximum lease time: Blank
Failover peer IP: Blank
Static ARP: No
Time format change: No
Statistics graphs: No
Ping check: No

# - Leave it is as default for:
Dynamic DNS
MAC address control
NTP
TFTP
LDAP
Network Booting
Additional BOOTP/DHCP Options
```
Click Save.

**Step 3**:
Set Firewall Rule for VLAN 30 to able to connect to Internet.

Go to Firewall -> Rules and choose VLAN 30. Click 'Up Arrow Add' and edit firewall rule

```bash
Action: Pass
Disabled: No
Interface VLAN30
Address Family: IPv4
Protocol: Any

Source: Any
Destination: Any

Description: Allow INTERNET access
```

> Note: Rules based on line-by-line configuration. By default, at the end of line, it will be "Deny any any all the rules". This basically block everything.

**Step 4**:
Setup our Access Points with complete SSID and password. This depends on which router model you're using. Go to their model documentation how to set or change your router to access points mode.

> Note: Ensure that the Access Points is set Static IP Address. Recommended to use Security WPA/WPA2-Personal

**Step 5**:
Test your Internet connection. Connect to your Wi-Fi and ensure you able to ping all of those.

```
ping 192.168.30.1 # Your PfSense Router
ping 1.1.1.1
ping 8.8.8.8
ping google.com
ping cloudflare.com
```

## Conclusion

Congrats, now you've setup basic home network using PfSense. I would love to recommend to check out this Youtube Guy [Lawrence Systems](https://www.youtube.com/channel/UCHkYOD-3fZbuGhwsADBd9ZQ) for more in depth PfSense configuration and settings.
