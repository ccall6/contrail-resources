# How to ssh into VM via metadata address
**Tested on CC13.0.2**

## Pre-steps
- The VM must be configured to accept ssh connections over the desired virtual network. For example, if you wanted to SSH into the fxp0 interface of a newly created vSRX VM (starting with the factory configuration), then you would first need to use the console to set the root password, to allow root login via SSH (or configure a user account), and to turn on DHCP on fxp0. Here is an example:
```
set system root-authentication encrypted-password "$5$jTsDiV0W$e2ZaB4KIdfnhfj3Ucz3wMU1cLlWos0NIJYl4PO770VA"
set system services ssh root-login allow
set interfaces fxp0 unit 0 family inet dhcp
```
- You also must know the VM name you want to SSH into as well as the virtual network you want to connect to.
    - For this example, the VM name is `Basic vSRX` and the virtual network is `basic-vsrx-mgmt`.

## On jumphost, connect as contrail user
```bash
[root@r4-ru43 ~]# su contrail
[contrail@r4-ru43 root]$
```

## SSH to undercloud VM
```bash
[contrail@r4-ru43 root]$ ssh undercloud
Last login: Fri Mar  8 14:26:13 2019 from 192.168.122.1
[stack@undercloud ~]$
```

## Source the overcloudrc file
```bash
[stack@undercloud ~]$ . overcloudrc
(overcloud) [stack@undercloud ~]$
```

## Get the VM address on the virtual network
```bash
(overcloud) [stack@undercloud ~]$ openstack server show "Basic vSRX" | grep basic-vsrx-mgmt
| addresses                           | basic-vsrx-mgmt=10.0.0.3                                             |
```

## Get the compute node hostname where the VM is running
```bash
(overcloud) [stack@undercloud ~]$ openstack server show "Basic vSRX" | grep OS-EXT-SRV-ATTR:host
| OS-EXT-SRV-ATTR:host                | overcloudt6n-comp-0.dc150-left.juniper.net                           |
```

## SSH into compute node using heat-admin user
- Extract the hostname portion of the FQDN server name
    - In this example, `overcloudt6n-comp-0`
- Append `.ctlplane` to the hostname and SSH using user heat-admin
```bash
(overcloud) [stack@undercloud ~]$ ssh heat-admin@overcloudt6n-comp-0.ctlplane
Last login: Fri Mar  8 14:40:29 2019 from 192.0.2.1
[heat-admin@overcloudt6n-comp-0 ~]$
```

## Determine the vif for the VM
**Note: This example is being done on a kernel vRouter compute node.**
- Execute vif --list within contrail_vrouter_agent container and grab the lines surrounding the VM's address
```bash
[heat-admin@overcloudt6n-comp-0 ~]$ sudo docker exec contrail_vrouter_agent vif --list | grep -C 1 10.0.0.3
vif0/3      OS: tap161f9de8-8c
            Type:Virtual HWaddr:00:00:5e:00:01:00 IPaddr:10.0.0.3
            Vrf:2 Mcast Vrf:2 Flags:PL3L2DEr QOS:-1 Ref:6
[heat-admin@overcloudt6n-comp-0 ~]$
```

## Assemble the VM's metadata address
- The metadata address will be `169.254.X.Y`
    - X = first number of vif (in this example, `0`)
    - Y = second number of vif (in this example, `3`)
    - In this example, the metadata address is `169.254.0.3`

## SSH to the metadata address
_Using the root user because that is what is configured on the vSRX VM._

```
[heat-admin@overcloudt6n-comp-0 ~]$ ssh root@169.254.0.3
The authenticity of host '169.254.0.3 (169.254.0.3)' can't be established.
ECDSA key fingerprint is SHA256:1btmSWK59La5L3J6P6In+NWPMjvNKy2tvZ3xz1nsa4Y.
ECDSA key fingerprint is MD5:aa:68:15:14:aa:18:fd:cf:f8:53:6a:23:ca:99:da:f8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '169.254.0.3' (ECDSA) to the list of known hosts.
Password:
Last login: Fri Mar  8 22:34:15 2019 from 10.0.0.2
--- JUNOS 15.1X49-D160.2 built 2018-12-11 22:37:01 UTC
root@% cli
root> show chassis hardware
Hardware inventory:
Item             Version  Part number  Serial number     Description
Chassis                                1B0EE54A50A8      VSRX
CB 0            
Routing Engine 0          BUILTIN      BUILTIN           VSRX-S
FPC 0            REV 07   611-049549   RL3714040884      FPC
  PIC 0                   BUILTIN      BUILTIN           VSRX DPDK GE

root>
```
