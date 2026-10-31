<!-- AIR:tour -->

# SONiC Numbered BGP EVPN VXLAN Demo

This environment demonstrates a common configuration for deploying an extended layer 2 environment (L2 stretch) with [**SONiC**](https://sonicfoundation.dev/). This type of network enables layer 2 domains to communicate over layer 3 fabrics with VXLAN tunnels using the EVPN control plane. 

In this demo, we use the [**SONiC ConfigDB**](https://github.com/sonic-net/SONiC/wiki/Configuration), the [**SONiC CLI**](https://github.com/sonic-net/sonic-utilities/blob/master/doc/Command-Reference.md) and [**FRR**](https://docs.frrouting.org/en/latest/) to set the [**EVPN layer 2 extension**](https://github.com/sonic-net/SONiC/blob/master/doc/vxlan/EVPN/EVPN_VXLAN_HLD.md) configuration on ToR switches and EVPN VXLAN fabric.

When layer 2 domains are divided by layer 3 fabrics, VXLAN encapsulation is implemented to tunnel the layer 2 traffic over the layer 3 underlay network. This technology suits many use cases where traditional VLANs can fall short. EVPN VXLAN is used to stretch old legacy layer 2 apps over layer 3, build scalable and reliable data centers, enhance network security and manageability, and more. 

For this demo, each ToR (leaf) is a VTEP and hosts the VLANs (mapped to VNIs) located on its rack. The same VNI must be set on all VTEPs where the specific subnet is located to achieve an extended layer 2 domain.

This environment **does not allow inter-VLAN connectivity**. To route between different VNIs, **EVPN centralized** or **distributed symmetric routing** must be used.

Check out this [blog](https://developer.nvidia.com/blog/exploring-sonic-on-nvidia-air/) for top-level insight into the lab and why NVIDIA created it.

Check out this [blog](https://developer.nvidia.com/blog/looking-behind-the-curtain-of-evpn-traffic-flows/) for more information on how EVPN traffic flows within a virtualized environment.

## Features and Services

This demo includes the following features and services:

 * [**BGP**](https://github.com/sonic-net/SONiC/blob/master/doc/mgmt/SONiC_Design_Doc_Unified_FRR_Mgmt_Interface.md) underlay fabric using **BGP numbered** interfaces
 * [**VXLAN & EVPN**](https://github.com/sonic-net/SONiC/blob/master/doc/vxlan/EVPN/EVPN_VXLAN_HLD.md) overlay encapsulation data plane, and overlay control plane, respectively

 <!-- AIR:page -->
 
## Demo Topology Information 

### Devices

The sample topology consists of **5** [**SONiC-vs-202305**](https://sonic.software/) switches and **6 Ubuntu 18.04** servers.

| __Leaf (SONiC)__ | __Spine (SONiC)__  | __Server (Ubuntu)__| 
| --------         | ----------         | ---------          | 
|   leaf01         | spine01            | server01           |
| leaf02           | spine02            | server02           |
| leaf03           |                    | server03           |
|                  |                    | server04           | 
|                  |                    | server05           | 
|                  |                    | server06           | 

<!-- AIR:page -->

### IPAM

#### Hosts

| __Hostname__| __Interface__ | __VRF__ | __VLAN__ | __IP Address__    |
| ---------   | ------------- | ------- | -------- | ----------------- |
| **server01**    | eth0          | mgmt    |          | 192.168.200.7/24  |
|             | eth1          | default | 10       | 172.16.10.101/24  |
| **server02**    | eth0          | mgmt    |          | 192.168.200.8/24  |
|             | eth1          | default | 20       | 172.16.20.102/24  |
| **server03**    | eth0          | mgmt    |          | 192.168.200.9/24  |
|             | eth1          | default | 10       | 172.16.10.103/24  |
| **server04**    | eth0          | mgmt    |          | 192.168.200.10/24 |
|             | eth1          | default | 20       | 172.16.20.104/24  |
| **server05**    | eth0          | mgmt    |          | 192.168.200.11/24 |
|             | eth1          | default | 10       | 172.16.10.105/24  |
| **server06**    | eth0          | mgmt    |          | 192.168.200.12/24 |
|             | eth1          | default | 20       | 172.16.20.106/24  |

#### Switches

| __Hostname__| __Interface__ | __VLAN__ | __IP Address__    | 
| ---------   | ------------- | -------  | ----------------  | 
| **leaf01**      | eth0          |          | 192.168.200.4/24  |
|             | Ethernet0     | 10       |                   | 
|             | Ethernet4     | 20       |                   | 
|             | Ethernet8     |          |  172.16.1.1/31    | 
|             | Ethernet12    |          |  172.16.2.1/31    | 
|             | Loopback0     | vtep     |  10.0.0.1/32      |
| **leaf02**      | eth0          |          | 192.168.200.5/24  |
|             | Ethernet0     | 10       |                   | 
|             | Ethernet4     | 20       |                   | 
|             | Ethernet8     |          |  172.16.1.3/31    | 
|             | Ethernet12    |          |  172.16.2.3/31    | 
|             | Loopback0     | vtep     |  10.0.0.2/32      |
| **leaf03**      | eth0          |          | 192.168.200.6/24  |
|             | Ethernet0     | 10       |                   | 
|             | Ethernet4     | 20       |                   | 
|             | Ethernet8     |          |  172.16.1.5/31    | 
|             | Ethernet12    |          |  172.16.2.5/31    | 
|             | Loopback0     | vtep     |  10.0.0.3/32      |
| **spine01**     | eth0          |          | 192.168.200.3/24  |
|             | Ethernet0     |          |  172.16.1.0/31    | 
|             | Ethernet4     |          |  172.16.1.2/31    | 
|             | Ethernet8     |          |  172.16.1.4/31    | 
|             | Loopback0     |          |  10.0.0.101/32    |
| **spine02**     | eth0          |          | 192.168.200.4/24  |
|             | Ethernet0     |          |  172.16.2.0/31    | 
|             | Ethernet4     |          |  172.16.2.2/31    | 
|             | Ethernet8     |          |  172.16.2.4/31    | 
|             | Loopback0     |          |  10.0.0.102/32    |

Servers are configured to access VLAN on the leaf switches.

<!-- AIR:page -->

### Physical Connectivity

__Hostname__| __AIR Port__ |  __Sonic Port__  | __Remote Port__ | __Remote Device__ |
----------- | -------------| ---------------  | --------------- |------------------ |
| **leaf01**    | eth1         |   Ethernet0      | eth1            | server01          |
|           | eth2         |   Ethernet4      | eth1            | server02          |
|           | eth3         |   Ethernet8      | Ethernet0       | spine01           |
|           | eth4         |   Ethernet12     | Ethernet0       | spine02           |
| **leaf02**    | eth1         |   Ethernet0      | eth1            | server03          |
|           | eth2         |   Ethernet4      | eth1            | server04          |
|           | eth3         |   Ethernet8      | Ethernet4       | spine01           |
|           | eth4         |   Ethernet12     | Ethernet4       | spine02           |
| **leaf03**    | eth1         |   Ethernet0      | eth1            | server05          |
|           | eth2         |   Ethernet4      | eth1            | server06          |
|           | eth3         |   Ethernet8      | Ethernet8       | spine01           |
|           | eth4         |   Ethernet12     | Ethernet8       | spine02           |
| **spine01**   | eth1         |   Ethernet0      | Ethernet8       | leaf01            |
|           | eth2         |   Ethernet4      | Ethernet8       | leaf02            |
|           | eth3         |   Ethernet8      | Ethernet8       | leaf03            |
| **spine02**   | eth1         |   Ethernet0      | Ethernet12      | leaf01            |
|           | eth2         |   Ethernet4      | Ethernet12      | leaf02            |
|           | eth3         |   Ethernet8      | Ethernet12      | leaf03            |

<!-- AIR:page -->

## Demo Environment Access

You can access any node's console either via the jump server `oob-mgmt-server` or by opening a device's console directly when viewing the simulation in your Simulation List. 

Use the default access credentials to login the `oob-mgmt-server`:
 - Username: ***ubuntu***
 - Password: ***nvidia***

***Note:*** *Once you first login, you must change the default password.*

You can use the web intergrated console or create an SSH service to access the `oob-mgmt-server` using any SSH client, such as PuTTY.  
For more information and step-by-step instructions, check out NVIDIA Air [Quick Start](https://docs.nvidia.com/networking-ethernet-software/guides/nvidia-air/Quick-Start/) guide.

When accessing other devices via the `oob-mgmt-server`, after first logging in, you can access any device in the topology using its hostname.

To login to the `servers`, use the following credentials:

 - Username: ***ubuntu***
 - Password: ***nvidia*** 

```
ubuntu@oob-mgmt-server:~$ ssh server01
```

To login to the `spines` and `leaves`, use the following credentials:

 - Username: ***admin***
 - Password: ***YourPaSsWoRd*** 

```
ubuntu@oob-mgmt-server:~$ ssh admin@leaf01
```

When logging in from other environment devices, use `ssh admin@<ip-address>`.

<!-- AIR:page -->

## Connectivity and Configuration Validation 

### Server to Server Connectivity

Verify that commmunication is only available between servers within the same VLAN. 

- _VLAN10:_ `server01`, `server03`, `server05` 
- _VLAN20:_ `server02`, `server04`, `server06` 


_**Log into `server01`:**_
```
ubuntu@oob-mgmt-server:~$ ssh server01
ubuntu@server01's password:
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 4.15.0-200-generic x86_64)
...
#########################################################
#      You are successfully logged in to: server01      #
#########################################################
Last login: Thu May 16 07:38:44 2024 from 192.168.200.1
```

_**Ping `server03` and `server05` to validate L2 intra-VLAN connectivity:**_
```
ubuntu@server01:~$ ping 172.16.10.103 -c 3
PING 172.16.10.103 (172.16.10.103) 56(84) bytes of data.
64 bytes from 172.16.10.103: icmp_seq=1 ttl=64 time=2.79 ms
64 bytes from 172.16.10.103: icmp_seq=2 ttl=64 time=1.76 ms
64 bytes from 172.16.10.103: icmp_seq=3 ttl=64 time=1.72 ms

--- 172.16.10.103 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 1.722/2.093/2.796/0.499 ms

ubuntu@server01:~$ ping 172.16.10.105 -c 3
PING 172.16.10.105 (172.16.10.105) 56(84) bytes of data.
64 bytes from 172.16.10.105: icmp_seq=1 ttl=64 time=5.26 ms
64 bytes from 172.16.10.105: icmp_seq=2 ttl=64 time=2.44 ms
64 bytes from 172.16.10.105: icmp_seq=3 ttl=64 time=2.90 ms

--- 172.16.10.105 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 2.442/3.540/5.269/1.237 ms
```

_**Ping `server02`, `server04`, and `server06` to validate that L3 inter-VLAN connectivity is not avaliable:**_
```
ubuntu@server01:~$ for srv in {20.102,20.104,20.106} ; do ping 172.16.${srv} -c 3; done
ING 172.16.20.102 (172.16.20.102) 56(84) bytes of data.

--- 172.16.20.102 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2054ms

PING 172.16.20.104 (172.16.20.104) 56(84) bytes of data.

--- 172.16.20.104 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2027ms

PING 172.16.20.106 (172.16.20.106) 56(84) bytes of data.

--- 172.16.20.106 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2027ms
```
<!-- AIR:page -->

### Switch Configuration 

The following will show the configuration for the lab switches. Output is presented on `leaf01` as a reference. 

The SONiC lab uses the SONiC image built for virtual devices, called sonic-vs. This image is already installed on all the SONiC switches in the lab.

SONiC architecture is containerized. Each module is placed in an independent docker container. Configuration is managed by a redisDB instance referred to as **ConfigDB**. ConfigDB can be manipulated in multiple ways, including editing the `config_db.json` file directly, or using the [**SONiC CLI**](https://github.com/sonic-net/sonic-utilities/blob/master/doc/Command-Reference.md). You can set many settings here. See the [SONiC Configuration Database Manual](https://github.com/sonic-net/sonic-buildimage/blob/master/src/sonic-yang-models/doc/Configuration.md) for more information.

In order to implement EVPN VXLAN in SONiC, **split-unified mode** has been implemented for this lab. This option allows separate switch and routing configuration. Switch configuration is handled by ConfigDB, while routing is configured with [**FRR**](https://docs.frrouting.org/en/latest/evpn.html#frr-configuration) and placed into a single `etc/sonic/frr/frr.conf` file. This option is enabled by inserting `docker_routing_config_mode: split-unified` into `DEVICE_METADATA` in `/etc/sonic/config_db.json.`

_**Login to `leaf01`:**_
```
admin@leaf01:~$
```

_**Show routing configuration:**_
```
admin@leaf01:~$ show runningconfiguration bgp
Building configuration...

Current configuration:
!
frr version 8.5.1
frr defaults datacenter
hostname leaf01
service integrated-vtysh-config
!
router bgp 65101
 bgp router-id 10.0.0.1
 no bgp default ipv4-unicast
 bgp bestpath as-path multipath-relax
 neighbor underlay peer-group
 neighbor underlay remote-as 65199
 neighbor 172.16.1.0 peer-group underlay
 neighbor 172.16.1.0 description spine01
 neighbor 172.16.2.0 peer-group underlay
 neighbor 172.16.2.0 description spine02
 !
 address-family ipv4 unicast
  network 10.0.0.1/32
  neighbor underlay activate
  maximum-paths 64
 exit-address-family
 !
 address-family l2vpn evpn
  neighbor underlay activate
  advertise-all-vni
 exit-address-family
exit
!
route-map RM_SET_SRC permit 10
exit
!
end
```

_**Show ConfigDB configuration (shortened for clarity):**_

You can also show the `config_db.json` directly by running `sudo cat /etc/sonic/config_db.json`.
```
admin@leaf01:~$ show runningconfiguration all
{
    "CONTAINER_FEATURE": {
        ...
    },
    "CRM": {
        ...
    },
    "DEVICE_METADATA": {
        "localhost": {
            "bgp_asn": "65101",
            "default_bgp_status": "up",
            "default_pfcwd_status": "disable",
            "hostname": "leaf01",
            "hwsku": "Mellanox-SN2700",
            "mac": "52:54:00:12:34:56",
            "platform": "x86_64-kvm_x86_64-r0",
            "type": "LeafRouter",
            "docker_routing_config_mode": "split-unified"
        }
    },
    "FEATURE": {
        ...
    },
    "FLEX_COUNTER_TABLE": {
        ...
    },
    "INTERFACE": {
        "Ethernet0": {},
        "Ethernet4": {},
        "Ethernet8": {},
        "Ethernet12": {},
        "Ethernet8|172.16.1.1/31": {},
        "Ethernet12|172.16.2.1/31": {}
    },
    "LOOPBACK_INTERFACE": {
        "Loopback0": {},
        "Loopback0|10.0.0.1/32": {}
    },
    "PORT": {
        ...
    },
    "VLAN": {
        "Vlan10": {
            "members": [
                "Ethernet0"
            ],
            "vlanid": "10"
        },
        "Vlan20": {
            "members": [
                "Ethernet4"
            ],
            "vlanid": "20"
        }
    },
    "VLAN_INTERFACE": {
        "Vlan10": {},
        "Vlan20": {}
    },
    "VLAN_MEMBER": {
        "Vlan10|Ethernet0": {
            "tagging_mode": "untagged"
        },
        "Vlan20|Ethernet4": {
            "tagging_mode": "untagged"
        }
    },
    "VXLAN_EVPN_NVO": {
        "nvo": {
            "source_vtep": "vtep"
        }
    },
    "VXLAN_TUNNEL": {
        "vtep": {
            "src_ip": "10.0.0.1"
        }
    },
    "VXLAN_TUNNEL_MAP": {
        "vtep|map_10_Vlan10": {
            "vlan": "Vlan10",
            "vni": "10"
        },
        "vtep|map_20_Vlan20": {
            "vlan": "Vlan20",
            "vni": "20"
        }
    }
}

```

<!-- AIR:page -->


### Showing Basic SONiC Information

SONiC offers many commands to view various important information.

_**Show the current SONiC version:**_
```
admin@leaf01:~$ show version

SONiC Software Version: SONiC.202305_RC.78-c121fc9fa_Internal_VS
SONiC OS Version: 11
Distribution: Debian 11.8
Kernel: 5.10.0-23-2-amd64
Build commit: c121fc9fa
Build date: Tue Jan 30 20:33:25 UTC 2024
Built by: sw-r2d2-bot@r-build-sonic-ci03-242

Platform: x86_64-kvm_x86_64-r0
HwSKU: Mellanox-SN2700
ASIC: vs
ASIC Count: 1
Serial Number: N/A
Model Number: N/A
Hardware Revision: N/A
Uptime: 07:34:50 up  2:51,  1 user,  load average: 0.03, 0.03, 0.00
Date: Fri 17 May 2024 07:34:50

Docker images:
REPOSITORY                                         TAG                                  IMAGE ID       SIZE
docker-orchagent                                   202305_RC.78-c121fc9fa_Internal_VS   fff0a2270125   330MB
docker-orchagent                                   latest                               fff0a2270125   330MB
docker-sflow                                       202305_RC.78-c121fc9fa_Internal_VS   214bc58792ae   319MB
docker-sflow                                       latest                               214bc58792ae   319MB
docker-teamd                                       202305_RC.78-c121fc9fa_Internal_VS   67f5f4b82ad5   318MB
docker-teamd                                       latest                               67f5f4b82ad5   318MB
docker-fpm-frr                                     202305_RC.78-c121fc9fa_Internal_VS   c5b1f67ca6d6   350MB
docker-fpm-frr                                     latest                               c5b1f67ca6d6   350MB
docker-nat                                         202305_RC.78-c121fc9fa_Internal_VS   ae14be90ffbd   321MB
docker-nat                                         latest                               ae14be90ffbd   321MB
docker-macsec                                      latest                               14b7ecbcab57   320MB
docker-gbsyncd-vs                                  202305_RC.78-c121fc9fa_Internal_VS   6eaed15aa084   311MB
docker-gbsyncd-vs                                  latest                               6eaed15aa084   311MB
docker-syncd-vs                                    202305_RC.78-c121fc9fa_Internal_VS   7babd1056ed4   315MB
docker-syncd-vs                                    latest                               7babd1056ed4   315MB
docker-dhcp-relay                                  latest                               8a1dd90534f8   308MB
docker-eventd                                      202305_RC.78-c121fc9fa_Internal_VS   2c73e112da40   300MB
docker-eventd                                      latest                               2c73e112da40   300MB
docker-snmp                                        202305_RC.78-c121fc9fa_Internal_VS   f9d73e1688e9   339MB
docker-snmp                                        latest                               f9d73e1688e9   339MB
docker-sonic-telemetry                             202305_RC.78-c121fc9fa_Internal_VS   8d541830cc5d   387MB
docker-sonic-telemetry                             latest                               8d541830cc5d   387MB
docker-platform-monitor                            202305_RC.78-c121fc9fa_Internal_VS   f58d269d03c1   422MB
docker-platform-monitor                            latest                               f58d269d03c1   422MB
docker-lldp                                        202305_RC.78-c121fc9fa_Internal_VS   1d2e6d23a5fe   343MB
docker-lldp                                        latest                               1d2e6d23a5fe   343MB
docker-database                                    202305_RC.78-c121fc9fa_Internal_VS   286850963053   300MB
docker-database                                    latest                               286850963053   300MB
docker-router-advertiser                           202305_RC.78-c121fc9fa_Internal_VS   16eeddfe5142   300MB
docker-router-advertiser                           latest                               16eeddfe5142   300MB
docker-mux                                         202305_RC.78-c121fc9fa_Internal_VS   177f966e4961   349MB
docker-mux                                         latest                               177f966e4961   349MB
docker-sonic-mgmt-framework                        202305_RC.78-c121fc9fa_Internal_VS   412b8a0a6f16   416MB
docker-sonic-mgmt-framework                        latest                               412b8a0a6f16   416MB
urm.nvidia.com/sw-nbu-sws-sonic-docker/sonic-wjh   1.6.2-202305                         ea03b424e21e   434MB
urm.nvidia.com/sw-nbu-sws-sonic-docker/doai        1.1.1-202305                         6144c8e186b3   276MB
```

_**Show the LLDP (Link Layer Discovery Protocol) table:**_
```
admin@leaf01:~$ show lldp table
Capability codes: (R) Router, (B) Bridge, (O) Other
LocalPort    RemoteDevice     RemotePortID       Capability    RemotePortDescr
-----------  ---------------  -----------------  ------------  -----------------
Ethernet0    ubuntu           48:b0:2d:74:5d:16  O             eth1
Ethernet4    ubuntu           48:b0:2d:5d:54:2d  O             eth1
Ethernet8    spine01          Ethernet0          BR            Ethernet0
Ethernet12   spine02          Ethernet0          BR            Ethernet0
eth0         oob-mgmt-switch  swp4               BR            swp4
--------------------------------------------------
```

_**Show IP interfaces:**_
```
admin@leaf01:~$ show ip interfaces
Interface    Master    IPv4 address/mask    Admin/Oper    BGP Neighbor    Neighbor IP
-----------  --------  -------------------  ------------  --------------  -------------
Ethernet8              172.16.1.1/31        up/up         N/A             N/A
Ethernet12             172.16.2.1/31        up/up         N/A             N/A
Loopback0              10.0.0.1/32          up/up         N/A             N/A
docker0                240.127.1.1/24       up/down       N/A             N/A
eth0                   192.168.200.4/24     up/up         N/A             N/A
lo                     127.0.0.1/16         up/up         N/A             N/A
```

See the [**SONiC Command Line Interface Guide**](https://github.com/sonic-net/sonic-utilities/blob/master/doc/Command-Reference.md) for an extensive list of show, configuration, and other CLI commands.


<!-- AIR:page -->

### EVPN BGP

The lab is configured with **numbered BGP EVPN** for the control plane.

**Unnumbered BGP** means that BGP neighbors are configured with interfaces, instead of IPv4 addresses. Unnumbered BGP simplifies configuration as you do not need to manage IP addresses, but can be more difficult to debug and track. This lab uses numbered BGP for that purpose.

_**Show BGP session:**_
```
admin@leaf01:~$ show ip bgp summary

IPv4 Unicast Summary:
BGP router identifier 10.0.0.1, local AS number 65101 vrf-id 0
BGP table version 5
RIB entries 9, using 1728 bytes of memory
Peers 2, using 1483952 KiB of memory
Peer groups 1, using 64 bytes of memory


Neighbhor      V     AS    MsgRcvd    MsgSent    TblVer    InQ    OutQ  Up/Down      State/PfxRcd  NeighborName
-----------  ---  -----  ---------  ---------  --------  -----  ------  ---------  --------------  --------------
172.16.1.0     4  65199       5858       5855         0      0       0  04:51:39                3
172.16.2.0     4  65199       5858       5855         0      0       0  04:51:39                3
```

_**Enter FRR:**_
```
admin@leaf01:~$ vtysh

Hello, this is FRRouting (version 8.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

leaf01#
```
_**Show BGP session (FRR):**_
```
leaf01# show ip bgp summary

IPv4 Unicast Summary (VRF default):
BGP router identifier 10.0.0.1, local AS number 65101 vrf-id 0
BGP table version 7
RIB entries 9, using 1728 bytes of memory
Peers 2, using 1449 KiB of memory
Peer groups 1, using 64 bytes of memory

Neighbor            V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
spine01(172.16.1.0) 4      65199     16604     16603        0    0    0 13:48:17            3        5 spine01
spine02(172.16.2.0) 4      65199     16618     16617        0    0    0 13:49:00            3        5 spine02

Total number of neighbors 2
```

_**Show EVPN BGP Session (FRR):**_
```
leaf01# show bgp l2vpn evpn summary
BGP router identifier 10.0.0.1, local AS number 65101 vrf-id 0
BGP table version 0
RIB entries 11, using 2112 bytes of memory
Peers 2, using 1449 KiB of memory
Peer groups 1, using 64 bytes of memory

Neighbor            V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
spine01(172.16.1.0) 4      65199     16616     16615        0    0    0 13:48:53           12       18 spine01
spine02(172.16.2.0) 4      65199     16630     16629        0    0    0 13:49:36           12       18 spine02

Total number of neighbors 2
```


_**Show Underlay Routing (FRR):**_
```
leaf01# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

K>* 0.0.0.0/0 [0/202] via 192.168.200.1, eth0, 03:23:28
C>* 10.0.0.1/32 is directly connected, Loopback0, 03:23:28
B>* 10.0.0.2/32 [20/0] via 172.16.1.0, Ethernet8, weight 1, 03:23:24
  *                    via 172.16.2.0, Ethernet12, weight 1, 03:23:24
B>* 10.0.0.3/32 [20/0] via 172.16.1.0, Ethernet8, weight 1, 03:23:24
  *                    via 172.16.2.0, Ethernet12, weight 1, 03:23:24
B>* 10.0.0.101/32 [20/0] via 172.16.1.0, Ethernet8, weight 1, 03:23:24
B>* 10.0.0.102/32 [20/0] via 172.16.2.0, Ethernet12, weight 1, 03:23:24
C>* 172.16.1.0/31 is directly connected, Ethernet8, 03:23:28
C>* 172.16.2.0/31 is directly connected, Ethernet12, 03:23:28
C>* 192.168.200.0/24 is directly connected, eth0, 03:23:28
```

_**Show EVPN MACs (FRR):**_
```
leaf01# show evpn mac vni all

VNI 10 #MACs (local and remote) 3

Flags: N=sync-neighs, I=local-inactive, P=peer-active, X=peer-proxy
MAC               Type   Flags Intf/Remote ES/VTEP            VLAN  Seq #'s
48:b0:2d:c8:27:92 remote       10.0.0.2                             1/0
48:b0:2d:da:47:ad remote       10.0.0.3                             1/0
48:b0:2d:1a:f6:dc local        Ethernet0                      10    0/0

VNI 20 #MACs (local and remote) 3

Flags: N=sync-neighs, I=local-inactive, P=peer-active, X=peer-proxy
MAC               Type   Flags Intf/Remote ES/VTEP            VLAN  Seq #'s
48:b0:2d:e3:eb:a0 remote       10.0.0.3                             1/0
48:b0:2d:3b:dd:1a remote       10.0.0.2                             1/0
48:b0:2d:ad:60:1f local        Ethernet4                      20    0/0
```


_**Show EVPN VNI Details (FRR):**_
```
leaf01# show evpn vni detail
VNI: 10
 Type: L2
 Tenant VRF: default
 VxLAN interface: vtep-10
 VxLAN ifIndex: 57
 SVI interface: Vlan10
 SVI ifIndex: 55
 Local VTEP IP: 10.0.0.1
 Mcast group: 0.0.0.0
 Remote VTEPs for this VNI:
  10.0.0.3 flood: HER
  10.0.0.2 flood: HER
 Number of MACs (local and remote) known for this VNI: 3
 Number of ARPs (IPv4 and IPv6, local and remote) known for this VNI: 3
 Advertise-gw-macip: No
 Advertise-svi-macip: No

VNI: 20
 Type: L2
 Tenant VRF: default
 VxLAN interface: vtep-20
 VxLAN ifIndex: 58
 SVI interface: Vlan20
 SVI ifIndex: 56
 Local VTEP IP: 10.0.0.1
 Mcast group: 0.0.0.0
 Remote VTEPs for this VNI:
  10.0.0.3 flood: HER
  10.0.0.2 flood: HER
 Number of MACs (local and remote) known for this VNI: 3
 Number of ARPs (IPv4 and IPv6, local and remote) known for this VNI: 3
 Advertise-gw-macip: No
 Advertise-svi-macip: No
```
See the [**SONiC Command Line Interface Guide**](https://github.com/sonic-net/sonic-utilities/blob/master/doc/Command-Reference.md) for an extensive list of show, configuration, and other CLI commands.

<!-- AIR:page -->

### VXLAN

_**Show VLANs:**_
```
admin@leaf01:~$ show vlan brief
+-----------+--------------+-----------+----------------+-------------+-----------------------+
|   VLAN ID | IP Address   | Ports     | Port Tagging   | Proxy ARP   | DHCP Helper Address   |
+===========+==============+===========+================+=============+=======================+
|        10 |              | Ethernet0 | untagged       | disabled    |                       |
+-----------+--------------+-----------+----------------+-------------+-----------------------+
|        20 |              | Ethernet4 | untagged       | disabled    |                       |
+-----------+--------------+-----------+----------------+-------------+-----------------------+
```

_**Show VXLAN Interface:**_
```
admin@leaf01:~$ show vxlan interface
VTEP Information:

        VTEP Name : vtep, SIP  : 10.0.0.1
        NVO Name  : nvo,  VTEP : vtep
        Source interface  : Loopback0
```

_**Show VNI Mapping:**_
```
admin@leaf01:~$ show vxlan vlanvnimap
+--------+-------+
| VLAN   |   VNI |
+========+=======+
| Vlan10 |    10 |
+--------+-------+
| Vlan20 |    20 |
+--------+-------+
Total count : 2
```

_**Show Remote VNIs:**_
```
admin@leaf01:~$ show vxlan remotevni all
+--------+--------------+-------+
| VLAN   | RemoteVTEP   |   VNI |
+========+==============+=======+
| Vlan10 | 10.0.0.2     |    10 |
+--------+--------------+-------+
| Vlan10 | 10.0.0.3     |    10 |
+--------+--------------+-------+
| Vlan20 | 10.0.0.2     |    20 |
+--------+--------------+-------+
| Vlan20 | 10.0.0.3     |    20 |
+--------+--------------+-------+
Total count : 4
```
_**Show Learned MACs:**_
```
admin@leaf01:~$ show vxlan remotemac all
+--------+-------------------+--------------+-------+---------+
| VLAN   | MAC               | RemoteVTEP   |   VNI | Type    |
+========+===================+==============+=======+=========+
| Vlan10 | 48:b0:2d:26:2f:1b | 10.0.0.3     |    10 | dynamic |
+--------+-------------------+--------------+-------+---------+
| Vlan10 | 48:b0:2d:a3:73:aa | 10.0.0.2     |    10 | dynamic |
+--------+-------------------+--------------+-------+---------+
| Vlan20 | 48:b0:2d:38:50:e6 | 10.0.0.3     |    20 | dynamic |
+--------+-------------------+--------------+-------+---------+
| Vlan20 | 48:b0:2d:c7:18:68 | 10.0.0.2     |    20 | dynamic |
+--------+-------------------+--------------+-------+---------+
Total count : 4
```


For more information and commands about EVPN VXLAN, check out the [GitHub documentation](https://github.com/sonic-net/SONiC/blob/master/doc/vxlan/EVPN/EVPN_VXLAN_HLD.md#512-show-commands).

<!-- AIR:page -->


## Automation

### Configuration
 
This lab has already been fully configured using the following Ansible scripts. You do not need to run these for the lab to work.

The following demonstrate how to redeploy the configuration to the environment, should you wish to make changes, try out some new config, or just learn how the lab is configured. You can safely rerun them even on an already configured lab, but beware they will overwrite any existing config.

1. Log into `oob-mgmt-server`
```
ubuntu@oob-mgmt-server:~$ 
```

2. Clone the repository to `oob-mgmt-server`
```
git clone https://gitlab.com/cumulus-consulting/goldenturtle/sonic-demo-evpn-vxlan.git
```
3. Move into project directory
```
cd sonic-demo-evpn-vxlan
```
4. Move into `automation` directory
```
cd automation
```
5. Configure SONiC spines and leaves
```
ansible-playbook -i inventories/ playbooks/configure_sonic.yml
```
6. Configure Ubuntu servers
```
ansible-playbook -i inventories/ playbooks/configure_servers.yml
```

### Reset SONiC Nodes

You can use the `reset_sonic.yml` script to reset the configuration of SONiC nodes. This script removes the `config_db.json` and `bgpd.conf` files and reboots the system. It will not affect local ports (LLDP) or the mgmt ip address.
```
ansible-playbook -i inventories/ playbooks/reset_sonic.yml
```

<!-- AIR:tour -->
?cache=8
