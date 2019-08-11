# guide_eap_proxy
Using the files from https://github.com/jaysoffian/eap_proxy these are the instructions to get a fully functional bypass using a EdgeRouter Lite and AT&amp;T router.  

Searching multiple places I could only find snippets and partial code.  After piecing these together I got a working config.  

Kudos to https://www.reddit.com/user/SScorpio/ that provided most if not all of the script from https://www.reddit.com/r/uverse/comments/9ii4t5/eli5_how_to_use_eaproxy_and_att_uverse/e9fbtoa?utm_source=share&utm_medium=web2x

I've used this only with a EdgeRouter Lite (ERL), but probably would work with most EdgeOS routers.

This guide configures the ERL with the following for the ports:

	eth0 = WAN (the connection from the ONT)
	eth1 = AT&T Router
	eth2 = LAN

## 1. Setup

Install [WinSCP](https://winscp.net/eng/index.php) - To transfer files

Install [Putty](https://www.putty.org/) - To configure the router

From a factory reset ERL

Plug in a computer to eth0

Set IP of computer to 192.168.1.100, 255.255.255.0, 192.168.1.1 [How to: Change IP address in Windows](https://www.thewindowsclub.com/change-ip-address-windows-10)

Login to the ERL EdgeOS by going to https://192.168.1.1 in your browser

## 2. Backup original configuration

Backup config of the ERL (System > Back Up Config) - Old firmware

Check www.ubnt.com/download to see if there is an updated firmware, download (if desired)

Upgrade firmware of device (if desired) (System > Upgrade System Image)

Backup config of the ERL (System > Back Up Config) - New firmware

## 3. Transfer EAP_Proxy files

Downloaded eap-proxy files from: https://github.com/jaysoffian/eap_proxy

Connect to ERL using WinSCP (IP address for the ERL is 192.168.1.1)

Copy using binary mode eap_proxy.sh to /config/scripts/post-config.d/

Copy using binary mode eap_proxy.py to /config/scripts/

Edit the eap_proxy.sh file that as just uploaded in WinSCP (Go to //root/config/scripts/post-config.d > Right-click on file > Edit)

![Editing eap_proxy.sh](https://i.imgur.com/u1NZa1D.png)

Change the options as such (by default IF_ROUTER is set to eth2):

	IF_WAN=eth0
	IF_ROUTER=eth1

Connect to ERL with Putty

Change permissions of files for execution using the following commands

	sudo chmod +x /config/scripts/post-config.d/eap_proxy.sh
	sudo chmod +x /config/scripts/eap_proxy.py
	
## 4. Configure the router

Configure router using the below, this will setup the firewall, all interfaces and give you working IPv6

I typically run sections this in batches then type in "save;commit"

In Putty, type **"configure"** then Enter to get into the configure prompt

Then, copy & paste the following:

	set firewall all-ping enable
	set firewall broadcast-ping disable
	set firewall ipv6-name IPv6_WAN_IN default-action drop
	set firewall ipv6-name IPv6_WAN_IN description 'IPv6 packets from the Internet to LAN'
	set firewall ipv6-name IPv6_WAN_IN rule 1 action accept
	set firewall ipv6-name IPv6_WAN_IN rule 1 description 'Allow established sessions'
	set firewall ipv6-name IPv6_WAN_IN rule 1 state established enable
	set firewall ipv6-name IPv6_WAN_IN rule 1 state invalid disable
	set firewall ipv6-name IPv6_WAN_IN rule 1 state new disable
	set firewall ipv6-name IPv6_WAN_IN rule 1 state related enable
	set firewall ipv6-name IPv6_WAN_IN rule 2 action drop
	set firewall ipv6-name IPv6_WAN_IN rule 2 state established disable
	set firewall ipv6-name IPv6_WAN_IN rule 2 state invalid enable
	set firewall ipv6-name IPv6_WAN_IN rule 2 state new disable
	set firewall ipv6-name IPv6_WAN_IN rule 2 state related disable
	set firewall ipv6-name IPv6_WAN_IN rule 5 action accept
	set firewall ipv6-name IPv6_WAN_IN rule 5 description 'Allow ICMPv6'
	set firewall ipv6-name IPv6_WAN_IN rule 5 log disable
	set firewall ipv6-name IPv6_WAN_IN rule 5 protocol icmpv6
	save;commit

	set firewall ipv6-name IPv6_WAN_LOCAL default-action drop
	set firewall ipv6-name IPv6_WAN_LOCAL description 'IPv6 packets from the Internet to the router'
	set firewall ipv6-name IPv6_WAN_LOCAL rule 10 action accept
	set firewall ipv6-name IPv6_WAN_LOCAL rule 10 description 'Allow established sessions'
	set firewall ipv6-name IPv6_WAN_LOCAL rule 10 log disable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 10 state established enable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 10 state invalid disable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 10 state new disable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 10 state related enable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 20 action drop
	set firewall ipv6-name IPv6_WAN_LOCAL rule 20 description 'Drop invalid state'
	set firewall ipv6-name IPv6_WAN_LOCAL rule 20 log disable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 20 state established disable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 20 state invalid enable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 20 state new disable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 20 state related disable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 50 action accept
	set firewall ipv6-name IPv6_WAN_LOCAL rule 50 description 'Allow ICMPv6'
	set firewall ipv6-name IPv6_WAN_LOCAL rule 50 log disable
	set firewall ipv6-name IPv6_WAN_LOCAL rule 50 protocol icmpv6
	set firewall ipv6-name IPv6_WAN_LOCAL rule 60 action accept
	set firewall ipv6-name IPv6_WAN_LOCAL rule 60 description 'Allow DHCPv6'
	set firewall ipv6-name IPv6_WAN_LOCAL rule 60 destination port 546
	set firewall ipv6-name IPv6_WAN_LOCAL rule 60 protocol udp
	set firewall ipv6-name IPv6_WAN_LOCAL rule 60 source port 547
	save;commit

	set firewall ipv6-receive-redirects disable
	set firewall ipv6-src-route disable
	set firewall ip-src-route disable
	set firewall log-martians enable
	save;commit

	set firewall name WAN_IN default-action drop
	set firewall name WAN_IN description 'WAN to internal'
	set firewall name WAN_IN rule 10 action accept
	set firewall name WAN_IN rule 10 description 'Allow established/related'
	set firewall name WAN_IN rule 10 state established enable
	set firewall name WAN_IN rule 10 state related enable
	set firewall name WAN_IN rule 20 action drop
	set firewall name WAN_IN rule 20 description 'Drop invalid state'
	set firewall name WAN_IN rule 20 state invalid enable
	save;commit

	set firewall name WAN_LOCAL default-action drop
	set firewall name WAN_LOCAL description 'WAN to router'
	set firewall name WAN_LOCAL rule 10 action accept
	set firewall name WAN_LOCAL rule 10 description 'Allow established/related'
	set firewall name WAN_LOCAL rule 10 state established enable
	set firewall name WAN_LOCAL rule 10 state related enable
	set firewall name WAN_LOCAL rule 50 action drop
	set firewall name WAN_LOCAL rule 50 description 'Drop invalid state'
	set firewall name WAN_LOCAL rule 50 state invalid enable
	save;commit

	set firewall receive-redirects disable
	set firewall send-redirects enable
	set firewall source-validation disable
	set firewall syn-cookies enable
	save;commit

	set interfaces ethernet eth0 description WAN
	set interfaces ethernet eth0 duplex auto
	set interfaces ethernet eth0 firewall in ipv6-name IPv6_WAN_IN
	set interfaces ethernet eth0 firewall in name WAN_IN
	set interfaces ethernet eth0 firewall local ipv6-name IPv6_WAN_LOCAL
	set interfaces ethernet eth0 firewall local name WAN_LOCAL
	set interfaces ethernet eth0 speed auto
	save;commit

	set interfaces ethernet eth0 vif 0 address dhcp
	set interfaces ethernet eth0 vif 0 description 'WAN VLAN 0'
	set interfaces ethernet eth0 vif 0 dhcp-options default-route update
	set interfaces ethernet eth0 vif 0 dhcp-options default-route-distance 210
	set interfaces ethernet eth0 vif 0 dhcp-options name-server update
	set interfaces ethernet eth0 vif 0 dhcpv6-pd pd 0 interface switch0 host-address '::1'
	set interfaces ethernet eth0 vif 0 dhcpv6-pd pd 0 interface switch0 prefix-id 1
	set interfaces ethernet eth0 vif 0 dhcpv6-pd pd 0 interface switch0 service slaac
	set interfaces ethernet eth0 vif 0 dhcpv6-pd pd 0 prefix-length /60
	set interfaces ethernet eth0 vif 0 dhcpv6-pd rapid-commit enable
	set interfaces ethernet eth0 vif 0 firewall in ipv6-name IPv6_WAN_IN
	set interfaces ethernet eth0 vif 0 firewall in name WAN_IN
	set interfaces ethernet eth0 vif 0 firewall local ipv6-name IPv6_WAN_LOCAL
	set interfaces ethernet eth0 vif 0 firewall local name WAN_LOCAL
	set interfaces ethernet eth0 vif 0 mac 'aa:bb:cc:dd:ee:ff'
	save;commit

	set interfaces ethernet eth1 description 'AT&T Router'
	set interfaces ethernet eth1 duplex auto
	set interfaces ethernet eth1 speed auto
	save;commit

	set interfaces ethernet eth2 address 192.168.2.254/24
	set interfaces ethernet eth2 description LAN
	set interfaces ethernet eth2 ipv6 dup-addr-detect-transmits 1
	set interfaces ethernet eth2 ipv6 router-advert cur-hop-limit 64
	set interfaces ethernet eth2 ipv6 router-advert link-mtu 0
	set interfaces ethernet eth2 ipv6 router-advert managed-flag false
	set interfaces ethernet eth2 ipv6 router-advert max-interval 600
	set interfaces ethernet eth2 ipv6 router-advert other-config-flag false
	set interfaces ethernet eth2 ipv6 router-advert prefix '::/64' autonomous-flag true
	set interfaces ethernet eth2 ipv6 router-advert prefix '::/64' on-link-flag true
	set interfaces ethernet eth2 ipv6 router-advert prefix '::/64' valid-lifetime 2592000
	set interfaces ethernet eth2 ipv6 router-advert reachable-time 0
	set interfaces ethernet eth2 ipv6 router-advert retrans-timer 0
	set interfaces ethernet eth2 ipv6 router-advert send-advert true
	set interfaces ethernet eth2 mtu 1500
	set interfaces ethernet eth0 speed auto
	set interfaces loopback lo
	save;commit

	set port-forward auto-firewall enable
	set port-forward hairpin-nat enable
	set port-forward lan-interface eth2
	set port-forward wan-interface eth0.0
	save;commit

	set service dhcp-server disabled false
	set service dhcp-server hostfile-update enable
	set service dhcp-server shared-network-name LAN2 authoritative enable
	set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 default-router 192.168.2.254
	set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 dns-server 192.168.2.254
	set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 lease 86400
	set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 start 192.168.2.5 stop 192.168.2.200
	set service dhcp-server static-arp disable
	set service dhcp-server use-dnsmasq disable
	save;commit

	set service dns forwarding cache-size 150
	set service dns forwarding listen-on eth2
	set service dns forwarding name-server 68.94.157.1
	set service dns forwarding name-server 68.94.156.1
	set service dns forwarding name-server 12.127.17.71
	set service dns forwarding name-server 12.127.16.67
	save;commit

	set service gui http-port 80
	set service gui https-port 443
	set service gui older-ciphers enable
	save;commit

	set service nat rule 5010 description 'masquerade for WAN'
	set service nat rule 5010 outbound-interface eth0.0
	set service nat rule 5010 type masquerade
	set service nat rule 6000 description 'MASQ all networks NTP to WAN'
	set service nat rule 6000 log disable
	set service nat rule 6000 outbound-interface eth2
	set service nat rule 6000 outside-address port 1024-65535
	set service nat rule 6000 protocol udp
	set service nat rule 6000 source port 123
	set service nat rule 6000 type masquerade
	set system offload ipv4 vlan enable
	save;commit

	set service ssh port 22
	set service ssh protocol-version v2
	set service unms disable
	save;commit

	set service upnp
	set service upnp2 acl rule 9000 action deny
	set service upnp2 acl rule 9000 description 'Deny All'
	set service upnp2 acl rule 9000 external-port 0-65535
	set service upnp2 acl rule 9000 local-port 0-65535
	set service upnp2 acl rule 9000 subnet 0.0.0.0/0
	set service upnp2 listen-on eth2
	set service upnp2 nat-pmp enable
	set service upnp2 secure-mode enable
	set service upnp2 wan eth0.0
	save;commit
	
Once everything returns and committed then "exit" from the configure mode

## 5. Hooking everything up

Enter the following in putty to monitor the data:

	sudo /usr/bin/python /config/scripts/eap_proxy.py eth0 eth1 --restart-dhcp --ignore-when-wan-up --ignore-logoff --ping-gateway

Plug in cable in from ONT to eth0

Plug in cable in from AT&T router to eth1 

Plug in cable out to LAN from eth2

In you should see EAPOL start with a whole bunch of communication

Log into the EdgeOS (https://192.168.2.254/) to see everything setup with:

	WAN (eth0) as 192.168.1.1
	WAN VLAN 0 (eth0.0) with WAN IP
	AT&T Router (eth1) has no IP
	LAN (eth2) as 192.168.2.254

Example of what it should look like:

![Configured ERL with Bypass](https://i.imgur.com/LrEg9WW.png)

Good idea to backup your ERL configuration if everything is in working order.
