# How to execute vrouter commands on compute node
**Tested on CC13.0.2**

## Description
There are a number of CLI commands that can be used to query the Contrail vRouter. You can see documentation here:
https://www.juniper.net/documentation/en_US/contrail5.0/topics/task/configuration/vrouter-cli-utilities-vnc.html

Before Contrail was containerized, you could execute these from the CLI of the compute node, but now that Contrail has been containerized, the workflow is a little different and will be demonstrated in this how-to.

_Note: This is for a kernel compute node. A DPDK compute node is a little different._

_Note: This how-to assumes you are already connected to the compute node via SSH. For instructions to do that, see this other how-to:_ https://github.com/ccall6/contrail-resources/blob/master/contrail-cloud-13/how-tos/how-to-login-to-overcloud-compute-node.md

## Executing single command from host CLI
The Contrail vRouter agent is now part of its own container: contrail_vrouter_agent. If you wish, you can execute individual vRouter commands from the host CLI via the `docker exec` command:

```bash
[heat-admin@overcloudt6n-comp-0 ~]$ sudo docker exec contrail_vrouter_agent vif --get 0/0
Vrouter Interface Table

Flags: P=Policy, X=Cross Connect, S=Service Chain, Mr=Receive Mirror
       Mt=Transmit Mirror, Tc=Transmit Checksum Offload, L3=Layer 3, L2=Layer 2
       D=DHCP, Vp=Vhost Physical, Pr=Promiscuous, Vnt=Native Vlan Tagged
       Mnp=No MAC Proxy, Dpdk=DPDK PMD Interface, Rfl=Receive Filtering Offload, Mon=Interface is Monitored
       Uuf=Unknown Unicast Flood, Vof=VLAN insert/strip offload, Df=Drop New Flows, L=MAC Learning Enabled
       Proxy=MAC Requests Proxied Always, Er=Etree Root, Mn=Mirror without Vlan Tag, Ig=Igmp Trap Enabled

vif0/0      OS: vlan312 (Speed 20000, Duplex 1)
            Type:Physical HWaddr:3c:fd:fe:ab:16:70 IPaddr:0.0.0.0
            Vrf:0 Mcast Vrf:65535 Flags:L3L2VpEr QOS:-1 Ref:4
            RX packets:1438769  bytes:97948509 errors:0
            TX packets:1746202  bytes:109982853 errors:0
            Drops:0
```

_Note: Root access is required, hence the `sudo`_

## Execute commands from within vRouter agent container
Alternatively, you can create a shell within the vRouter agent container, allowing you to work within it:

```bash
[heat-admin@overcloudt6n-comp-0 ~]$ sudo docker exec -it contrail_vrouter_agent bash
(vrouter-agent)[root@overcloudt6n-comp-0 /]$
(vrouter-agent)[root@overcloudt6n-comp-0 /]$ vif --get 0/1
Vrouter Interface Table

Flags: P=Policy, X=Cross Connect, S=Service Chain, Mr=Receive Mirror
       Mt=Transmit Mirror, Tc=Transmit Checksum Offload, L3=Layer 3, L2=Layer 2
       D=DHCP, Vp=Vhost Physical, Pr=Promiscuous, Vnt=Native Vlan Tagged
       Mnp=No MAC Proxy, Dpdk=DPDK PMD Interface, Rfl=Receive Filtering Offload, Mon=Interface is Monitored
       Uuf=Unknown Unicast Flood, Vof=VLAN insert/strip offload, Df=Drop New Flows, L=MAC Learning Enabled
       Proxy=MAC Requests Proxied Always, Er=Etree Root, Mn=Mirror without Vlan Tag, Ig=Igmp Trap Enabled

vif0/0      OS: vlan312 (Speed 20000, Duplex 1)
            Type:Physical HWaddr:3c:fd:fe:ab:16:70 IPaddr:0.0.0.0
            Vrf:0 Mcast Vrf:65535 Flags:L3L2VpEr QOS:-1 Ref:4
            RX packets:1438871  bytes:97955265 errors:0
            TX packets:1746326  bytes:109990515 errors:0
            Drops:0

(vrouter-agent)[root@overcloudt6n-comp-0 /]$
(vrouter-agent)[root@overcloudt6n-comp-0 /]$ flow -l
Flow table(size 80609280, entries 629760)

Entries: Created 1678 Added 1677 Deleted 1805 Changed 1819Processed 1678 Used Overflow entries 0
(Created Flows/CPU: 68 44 80 54 66 58 78 65 47 38 109 71 99 88 60 97 89 71 76 78 15 27 39 15 4 1 1 2 17 3 18 3 1 3 0 25 7 0 28 33)(oflows 0)

Action:F=Forward, D=Drop N=NAT(S=SNAT, D=DNAT, Ps=SPAT, Pd=DPAT, L=Link Local Port)
 Other:K(nh)=Key_Nexthop, S(nh)=RPF_Nexthop
 Flags:E=Evicted, Ec=Evict Candidate, N=New Flow, M=Modified Dm=Delete Marked
TCP(r=reverse):S=SYN, F=FIN, R=RST, C=HalfClose, E=Established, D=Dead

    Index                Source:Port/Destination:Port                      Proto(V)
-----------------------------------------------------------------------------------
(vrouter-agent)[root@overcloudt6n-comp-0 /]$
(vrouter-agent)[root@overcloudt6n-comp-0 /]$
```

### Exit
To leave the container, simply use the `exit` command:

```bash
(vrouter-agent)[root@overcloudt6n-comp-0 /]$ exit
exit
[heat-admin@overcloudt6n-comp-0 ~]$
```
