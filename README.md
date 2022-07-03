# Helm Chart repository for OSM

## ACCESS KNF
bridges:

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
    
routes:


    10.255.0.0/24 dev net1 proto kernel scope link src 10.255.0.1 
    192.168.100.0/24 dev net2 proto kernel scope link src 192.168.100.1 


## CPE KNF

bridges:

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

routes:

    default via 10.100.1.254 dev net1 
    10.20.1.0/24 via 192.168.255.253 dev brint 
    10.100.1.0/24 dev net1 proto kernel scope link src 10.100.1.1 
    192.168.100.0/24 dev net2 proto kernel scope link src 192.168.100.2 
    192.168.255.0/24 dev brint proto kernel scope link src 192.168.255.254 

## WAN KNF

bridges:

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

routes:

    192.168.100.0/24 dev net2 proto kernel scope link src 192.168.100.3 

flows:

    cookie=0x3151b, duration=18.058s, table=0, n_packets=0, n_bytes=0, priority=45002,ip,in_port=axswan,nw_dst=10.20.2.192/26 actions=output:net1
    cookie=0x31512, duration=18.197s, table=0, n_packets=0, n_bytes=0, priority=45001,in_port=cpewan,dl_src=00:00:00:00:00:20 actions=output:axswan
    cookie=0x31511, duration=18.273s, table=0, n_packets=0, n_bytes=0, priority=45000,in_port=axswan,dl_dst=ff:ff:ff:ff:ff:ff actions=FLOOD
    cookie=0x31513, duration=18.127s, table=0, n_packets=0, n_bytes=0, priority=45001,in_port=axswan,dl_dst=00:00:00:00:00:20 actions=output:net1
    cookie=0x1, duration=18.412s, table=0, n_packets=0, n_bytes=0, priority=40000,in_port=cpewan actions=output:net1
    cookie=0x2, duration=18.342s, table=0, n_packets=0, n_bytes=0, priority=40000,in_port=net1 actions=output:axswan


## ACCESS KNF 2

bridges:

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

routes:

    10.255.0.0/24 dev net1 proto kernel scope link src 10.255.0.1 
    192.168.200.0/24 dev net2 proto kernel scope link src 192.168.200.1 

## CPE KNF 2

bridges:

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

routes:

    default via 10.100.2.254 dev net1 
    10.20.2.0/24 via 192.168.255.253 dev brint 
    10.100.2.0/24 dev net1 proto kernel scope link src 10.100.2.1  
    192.168.200.0/24 dev net2 proto kernel scope link src 192.168.200.2 
    192.168.255.0/24 dev brint proto kernel scope link src 192.168.255.254 

## WAN KNF 2
bridges:

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
                options: {dst_port="8741", key=sdwn, remote_ip="192.168.100.2"}
        Port net1
            Interface net1
    ovs_version: "2.13.5"

routes:

    192.168.200.0/24 dev net2 proto kernel scope link src 192.168.200.3 
