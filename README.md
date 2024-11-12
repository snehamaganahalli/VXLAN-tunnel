# VXLAN-tunnel
VXLAN tunnel

How to create a VXLAN tunnel?

**Setup Diagrram:**

                      DUT1 eth5(WAN: 172.16.10.10) ===========================================eth5(WAN: 172.16.10.20) DUT2
                      | eth4 (br-lan)                                                                        eth4(br-lan)|
                      |                                                                                                  |
                      PC1                                                                                              PC2

**Setup Desciption:**
eth5 is the WAN interface. eth4 is the LAN interface connected to the PC1 and PC2. It is added to the bridge.

**Configuration:**

**DUT1**

/etc/init.d/firewall disable\
/etc/init.d/firewall stop\
uci set network.lan=interface\
uci set network.lan.ifname='eth4'\
uci set network.lan.type=bridge\
uci set network.lan.proto=static\
uci set network.lan.ipaddr=192.168.4.1\
uci set network.lan.netmask=255.255.255.0\
uci set network.wan=interface\
uci set network.wan.ifname=eth5\
uci set network.wan.proto=static\
uci set network.wan.ipaddr=172.16.10.10\
uci set network.wan.netmask=255.255.255.0\
uci set network.wan.macaddr=00:03:7F:BA:DB:20\
uci commit\
/etc/init.d/network restart

ip link add vxlan0 type vxlan id 10 dev eth5 dstport 4789 remote 172.16.10.20 local 172.16.10.10\
brctl addif br-lan vxlan0\
ip link set vxlan0 up


**DUT2**

/etc/init.d/firewall disable\
/etc/init.d/firewall stop\
uci set network.lan=interface\
uci set network.lan.ifname='eth4'\
uci set network.lan.type=bridge\
uci set network.lan.proto=static\
uci set network.lan.ipaddr=192.168.4.2\
uci set network.lan.netmask=255.255.255.0\
uci set network.wan=interface\
uci set network.wan.ifname=eth5\
uci set network.wan.proto=static\
uci set network.wan.ipaddr=172.16.10.20\
uci set network.wan.netmask=255.255.0.0\
uci set network.wan.macaddr=00:03:7F:BA:DB:40\
uci commit\
/etc/init.d/network restart

ip link add vxlan0 type vxlan id 10 dev eth5 dstport 4789 remote 172.16.10.10 local 172.16.10.20\
brctl addif br-lan vxlan0\
ip link set vxlan0 up

**NOTE: Disable firewall**

**Debug commands:**

ifconfig\
brctl show\
bridge fdb show\
echo 0 > /proc/sys/kernel/printk\
echo 8 > /proc/sys/kernel/printk\
dmesg -n8

============================================
**LOGS:**

**DUT1**

bridge fdb show\
00:00:07:aa:d7:2f dev vxlan0 master br-lan (same as last)\
00:03:7f:46:67:eb dev vxlan0 master br-lan (same as second last)\
2a:50:ef:7c:ce:95 dev vxlan0 master br-lan permanent => its is vxlan0 macaddress of the same DUT. It will be added as soon as you add vxlan0 to bridge.\
00:00:00:00:00:00 dev vxlan0 dst 172.16.10.20 via eth5 self permanent => It is the 0 mac address. It is added as the 1st MAC address. It is the default route.\
00:03:7f:46:67:eb dev vxlan0 dst 172.16.10.20 self => (br-lan and eth4 (LAN) mac of DUT2). They both have same mac address.\
00:00:07:aa:d7:2f dev vxlan0 dst 172.16.10.20 self => It is the MAC address of the client connected to the other end (client connected to PC2)

**DUT2:**

similar to DUT1

==========================

VXLAN tunnel uses a destination port 4789.

Difference between VXLAN and VLAN

VLAN
12 bit identifier, Therefore 2^12 = 4096 subnets.

VXLAN
24 bit identifier (Double the VLAN) => 2^24 = 1,67,77,216 subnets => 16 million or 1.6 crore subnets

VLANs operate with a 12-bit network identifier. This means that you can only create 4096 administrative domains within your network using VLANs. On the other hand, VXLANs operate with a 24-bit network identifier. With this, you can theoretically create as many as 16 million administrative domains.
