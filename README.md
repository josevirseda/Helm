# Configuration guide

Example creating vxlan tunnel:

        ovs-vsctl add-port brint axscpe -- set interface axscpe options:remote_ip=192.168.100.2 options:dst_port=8742 options:key=inet
        
## ACCESS KNF

**Bridges:**

    Bridge brint
        Port axscpe
            Interface axscpe
                type: vxlan
                options: {dst_port="8742", key=inet, remote_ip="192.168.100.2"}
        Port vxlan2
            Interface vxlan2
                type: vxlan
                options: {dst_port="8742", key=inet, remote_ip="10.255.0.2"}
        Port brint
            Interface brint
                type: internal
    Bridge brwan
        Port brwan
            Interface brwan
                type: internal
        Port vxlan1
            Interface vxlan1
                type: vxlan
                options: {remote_ip="10.255.0.2"}
        Port axswan
            Interface axswan
                type: vxlan
                options: {remote_ip="192.168.100.3"}
    ovs_version: "2.13.5"
    
**Routes:**


    10.255.0.0/24 dev net1 proto kernel scope link src 10.255.0.1 
    192.168.100.0/24 dev net2 proto kernel scope link src 192.168.100.1 


## CPE KNF

**Necessary packages:**

    apt-get install iptables    
    
**Bridges:**

    Bridge brwan
        Port vxlan1
            Interface vxlan1
                type: vxlan
                options: {remote_ip="10.100.2.1"}
        Port brwan
            Interface brwan
                type: internal
        Port cpewan
            Interface cpewan
                type: vxlan
                options: {dst_port="8741", key=sdwn, remote_ip="192.168.100.3"}
    Bridge brint
        Port axscpe
            Interface axscpe
                type: vxlan
                options: {dst_port="8742", key=inet, remote_ip="192.168.100.1"}
        Port brint
            Interface brint
                type: internal
    ovs_version: "2.13.5"

**Routes:**

    default via 10.100.1.254 dev net1 
    10.20.1.0/24 via 192.168.255.253 dev brint 
    10.100.1.0/24 dev net1 proto kernel scope link src 10.100.1.1 
    192.168.100.0/24 dev net2 proto kernel scope link src 192.168.100.2 
    192.168.255.0/24 dev brint proto kernel scope link src 192.168.255.254 

## WAN KNF

neccessary packages:

    apt-get install ryu-bin
    flowmanager
    
**Bridges:**

    Bridge brwan
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        Port cpewan
            Interface cpewan
                type: vxlan
                options: {dst_port="8741", key=sdwn, remote_ip="192.168.100.2"}
        Port brwan
            Interface brwan
                type: internal
        Port axswan
            Interface axswan
                type: vxlan
                options: {remote_ip="192.168.100.1"}
        Port net1
            Interface net1
    ovs_version: "2.13.5"

**Routes:**

    192.168.100.0/24 dev net2 proto kernel scope link src 192.168.100.3 
controller set-up:

    OVS_DPID="0000000000000001"

    ryu-manager --verbose flowmanager/flowmanager.py ryu.app.ofctl_rest 2>&1 | tee ryu.log &

    service openvswitch-switch start
    ovs-vsctl add-br brwan
    ovs-vsctl set bridge brwan protocols=OpenFlow10,OpenFlow12,OpenFlow13
    ovs-vsctl set-fail-mode brwan secure
    ovs-vsctl set bridge brwan other-config:datapath-id=$OVS_DPID

**flows:**

    cookie=0x3151b, duration=983.405s, table=0, n_packets=53, n_bytes=5194, priority=45002,ip,in_port=axswan,nw_dst=10.20.2.192/26 actions=output:net1
    cookie=0x31512, duration=983.552s, table=0, n_packets=0, n_bytes=0, priority=45001,in_port=net1,dl_src=00:00:00:00:00:20 actions=output:axswan
    cookie=0x31511, duration=983.651s, table=0, n_packets=13, n_bytes=546, priority=45000,in_port=axswan,dl_dst=ff:ff:ff:ff:ff:ff actions=FLOOD
    cookie=0x31513, duration=983.475s, table=0, n_packets=0, n_bytes=0, priority=45001,in_port=axswan,dl_dst=00:00:00:00:00:20 actions=output:net1
    cookie=0x1, duration=983.793s, table=0, n_packets=36, n_bytes=3024, priority=40000,in_port=axswan actions=output:cpewan
    cookie=0x2, duration=983.722s, table=0, n_packets=78, n_bytes=7140, priority=40000,in_port=cpewan actions=output:axswan


## ACCESS KNF 2

**Bridges:**

    Bridge brwan
        Port vxlan1
            Interface vxlan1
                type: vxlan
                options: {remote_ip="10.255.0.2"}
        Port axswan
            Interface axswan
                type: vxlan
                options: {remote_ip="192.168.200.3"}
        Port brwan
            Interface brwan
                type: internal
    Bridge brint
        Port axscpe
            Interface axscpe
                type: vxlan
                options: {dst_port="8742", key=inet, remote_ip="192.168.200.2"}
        Port vxlan2
            Interface vxlan2
                type: vxlan
                options: {dst_port="8742", key=inet, remote_ip="10.255.0.2"}
        Port brint
            Interface brint
                type: internal
    ovs_version: "2.13.5"

**Routes:**

    10.255.0.0/24 dev net1 proto kernel scope link src 10.255.0.1 
    192.168.200.0/24 dev net2 proto kernel scope link src 192.168.200.1 

## CPE KNF 2

**Necessary packages:**

    apt-get install iptables    
    
**Bridges:**

    Bridge brwan
        Port brwan
            Interface brwan
                type: internal
        Port cpewan
            Interface cpewan
                type: vxlan
                options: {dst_port="8741", key=sdwn, remote_ip="192.168.200.3"}
        Port vxlan1
            Interface vxlan1
                type: vxlan
                options: {remote_ip="10.100.1.1"}
    Bridge brint
        Port brint
            Interface brint
                type: internal
        Port axscpe
            Interface axscpe
                type: vxlan
                options: {dst_port="8742", key=inet, remote_ip="192.168.200.1"}

**Routes:**

    default via 10.100.2.254 dev net1 
    10.20.2.0/24 via 192.168.255.253 dev brint 
    10.100.2.0/24 dev net1 proto kernel scope link src 10.100.2.1  
    192.168.200.0/24 dev net2 proto kernel scope link src 192.168.200.2 
    192.168.255.0/24 dev brint proto kernel scope link src 192.168.255.254 

## WAN KNF 2

**Bridges:**

    Bridge brwan
        Controller "tcp:127.0.0.1:6633"
        fail_mode: secure
        Port axswan
            Interface axswan
                type: vxlan
                options: {remote_ip="192.168.200.1"}
        Port brwan
            Interface brwan
                type: internal
        Port cpewan
            Interface cpewan
                type: vxlan
                options: {dst_port="8741", key=sdwn, remote_ip="192.168.200.2"}
        Port net1
            Interface net1
    ovs_version: "2.13.5"
    
**Routes:**

    192.168.200.0/24 dev net2 proto kernel scope link src 192.168.200.3 
