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

/etc/init.d/firewall disable
/etc/init.d/firewall stop
uci set network.lan=interface
uci set network.lan.ifname='eth4'
uci set network.lan.type=bridge
uci set network.lan.proto=static
uci set network.lan.ipaddr=192.168.4.1
uci set network.lan.netmask=255.255.255.0
uci set network.wan=interface
uci set network.wan.ifname=eth5
uci set network.wan.proto=static
uci set network.wan.ipaddr=172.16.10.10
uci set network.wan.netmask=255.255.255.0
uci set network.wan.macaddr=00:03:7F:BA:DB:20
uci commit
/etc/init.d/network restart

ip link add vxlan0 type vxlan id 10 dev eth5 dstport 4789 remote 172.16.10.20 local 172.16.10.10
brctl addif br-lan vxlan0
ip link set vxlan0 up


**DUT2**

/etc/init.d/firewall disable
/etc/init.d/firewall stop
uci set network.lan=interface
uci set network.lan.ifname='eth4'
uci set network.lan.type=bridge
uci set network.lan.proto=static
uci set network.lan.ipaddr=192.168.4.2
uci set network.lan.netmask=255.255.255.0
uci set network.wan=interface
uci set network.wan.ifname=eth5
uci set network.wan.proto=static
uci set network.wan.ipaddr=172.16.10.20
uci set network.wan.netmask=255.255.0.0
uci set network.wan.macaddr=00:03:7F:BA:DB:40
uci commit
/etc/init.d/network restart

ip link add vxlan0 type vxlan id 10 dev eth5 dstport 4789 remote 172.16.10.10 local 172.16.10.20
brctl addif br-lan vxlan0
ip link set vxlan0 up

**NOTE: Disable firewall**

**Debug commands:**

ifconfig
brctl show
bridge fdb show
echo 0 > /proc/sys/kernel/printk
echo 8 > /proc/sys/kernel/printk
dmesg -n8
