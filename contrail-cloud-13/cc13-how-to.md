# How to login to overcloud controller VM
**Tested on CC13.0.2 pre-release 341**

## On jumphost, connect as contrail user
```bash
[root@r4-ru43 ~]# su contrail
[contrail@r4-ru43 root]$
```

## SSH to undercloud VM
```bash
[contrail@r4-ru43 root]$ ssh undercloud
Last login: Tue Nov 20 06:17:00 2018 from 192.168.122.1
[stack@undercloud ~]$
```

## Source the stackrc file
```bash
[stack@undercloud ~]$ . stackrc
(undercloud) [stack@undercloud ~]$
```

## Display the list of servers
```bash
(undercloud) [stack@undercloud ~]$ openstack server list
+--------------------------------------+---------------------+--------+---------------------+----------------+-----------------------------+
| ID                                   | Name                | Status | Networks            | Image          | Flavor                      |
+--------------------------------------+---------------------+--------+---------------------+----------------+-----------------------------+
| 651355a5-c6d9-4e28-8481-60aea6e20d3a | overcloudwfx-ca-0   | ACTIVE | ctlplane=192.0.2.59 | overcloud-full | contrail-analytics          |
| 5580b412-b692-4c57-8f78-cda1b6a94783 | overcloudwfx-cc-0   | ACTIVE | ctlplane=192.0.2.60 | overcloud-full | contrail-controller         |
| 7f30d39d-047b-49f9-924f-f85a89d8e89a | overcloudwfx-comp-0 | ACTIVE | ctlplane=192.0.2.58 | overcloud-full | compute                     |
| a0fdb86f-6455-4a1c-b6ef-64ed65ddf6a4 | overcloudwfx-ctrl-0 | ACTIVE | ctlplane=192.0.2.56 | overcloud-full | control                     |
| 8ce18019-f71f-4cd2-a91d-b3568c3aa1fc | overcloudwfx-cadb-0 | ACTIVE | ctlplane=192.0.2.55 | overcloud-full | contrail-analytics-database |
+--------------------------------------+---------------------+--------+---------------------+----------------+-----------------------------+
```

## SSH to control server address as heat-admin user
```bash
(undercloud) [stack@undercloud ~]$ ssh heat-admin@192.0.2.56
Warning: Permanently added '192.0.2.56' (ECDSA) to the list of known hosts.
Last login: Tue Nov 20 06:23:08 2018 from 192.0.2.1
[heat-admin@overcloudwfx-ctrl-0 ~]$
```
---

# How to login to overcloud controller host
**Tested on CC13.0.2**

## On jumphost, connect as contrail user
```bash
[root@r4-ru43 ~]# su contrail
[contrail@r4-ru43 root]$
```

## SSH to undercloud VM
```bash
[contrail@r4-ru43 root]$ ssh undercloud
Last login: Tue Nov 20 06:17:00 2018 from 192.168.122.1
[stack@undercloud ~]$
```

## Grep for the controller host name (from control-host-nodes.yml config file) in /etc/hosts
```bash
[stack@undercloud ~]$ grep r4u37-control /etc/hosts
192.0.2.5 r4u37-control.dc150-left.juniper.net r4u37-control
```

## Return to the jumphost but remain as contrail user
```bash
[stack@undercloud ~]$ exit
logout
Connection to 192.168.122.252 closed.
[contrail@r4-ru43 root]$
```

## SSH to control server address (as contrail user)
```bash
[contrail@r4-ru43 root]$ ssh 192.0.2.5
Warning: Permanently added '192.0.2.5' (ECDSA) to the list of known hosts.
Last login: Tue Feb 26 09:08:40 2019 from gateway
[contrail@r4u37-control ~]$
```
---

# How to increase the MTU
**Tested on CC13.0.2 pre-release 341**

## site.yml: Set desired MTU for all overcloud networks
Example:
```yaml
overcloud:
  network:
    tenant:
      mtu: 9100
```

## control-host-nodes.yml: Set desired MTU for all bridges and interfaces that will carry overcloud network traffic (i.e. the bond bridge and the bond physical interface members)
Example:
```yaml
control_host_nodes_network_config:
  - type: ovs_bridge
    name: br-bond0
    use_dhcp: false
    mtu: 9100
    members:
       -
         type: ovs_bond
         name: bond0
         ovs_options: "bond_mode=balance-slb lacp=active other_config:lacp-time=fast other_config:bond-detect-mode=miimon other_config:bond-miimon-interval=100"
         use_dhcp: false
         members:
           -
             type: interface
             mtu: 9100
             name: p3p1
           -
             type: interface
             mtu: 9100
             name: em1
```

## overcloud-nics.yml: Set desired MTU on physical interfaces that carry overcloud networks
_contrail_network_config and controller_network_config - Set on eth interfaces_
Example:
```yaml
- type: interface
  name: eth1
  use_dhcp: false
  mtu: 9100
```

_compute_network_config - Set on bond interfaces_
Example:
```yaml
- type: linux_bond
  name: bond0
  use_dhcp: false
  mtu: 9100
  bonding_options: "mode=802.3ad xmit_hash_policy=layer3+4 lacp_rate=fast updelay=1000 miimon=100"
  members:
  - type: interface
    name: p3p1
    mtu: 9100
    primary: true
  - type: interface
    name: em1
    mtu: 9100
```

_compute_dpdk_network_config - Set on vhost bond and normal bond interfaces_
Example:
```yaml
- type: contrail_vrouter_dpdk
  name: vhost0
  vlan_id:
    get_param: TenantNetworkVlanID
  driver: "{{ overcloud['contrail']['vrouter']['dpdk']['driver'] }}"
  bond_policy: layer2+3
  cpu_list: 0x14
  bond_mode: 4
  mtu: 9100
  members:
  - type: interface
    name: p3p1
    mtu: 9100
  - type: interface
    name: em1
    mtu: 9100
```

## Redeploy the overcloud
If you are modifying MTU on an existing deployment, then you will likely need to remove the nodes and readd them for the MTU to take effect. In my lab, I ran -c on openstack-deploy.sh, control-vms-deploy.sh, and control-hosts-deploy.sh, and then redeployed in the reverse order. That might have been overkill but is how I did the change in my lab.

## Notes
- I tried deploying with just openstack-deploy.sh, but it didn't affect the MTUs. Need to redeploy the overcloud I assume.
---

# How to view interface info from control hosts
**Tested on CC13.0.2 pre-release 341**

## On jumphost, connect as contrail user
```bash
[root@r4-ru43 ~]# su contrail
[contrail@r4-ru43 root]$
```

## SSH to undercloud VM
```bash
[contrail@r4-ru43 root]$ ssh undercloud
Last login: Tue Nov 20 06:17:00 2018 from 192.168.122.1
[stack@undercloud ~]$
```

## Source the stackrc file
```bash
[stack@undercloud ~]$ . stackrc
(undercloud) [stack@undercloud ~]$
```

## Display the BMS list
```bash
(undercloud) [stack@undercloud ~]$ openstack baremetal node list
+--------------------------------------+---------------------------------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name                                  | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+---------------------------------------+--------------------------------------+-------------+--------------------+-------------+
| 310adac6-e279-4393-99e7-e357bef705f9 | r4u39-compute                         | db042212-1fc3-4175-adce-7e5fdaaad1da | power on    | active             | False       |
| 887828bb-b567-4e70-a755-70f9180287d0 | r4u42-control                         | None                                 | power on    | active             | False       |
| 2dda9b04-048e-4126-af5f-04adb91cc0f2 | control-192_0_2_5                     | b2800a6f-28c5-40f2-b74a-e5ca94847edc | power on    | active             | False       |
| cf99e06d-e242-4e1f-a12e-eea9b1d52315 | contrail-controller-192_0_2_5         | 8a085ee6-ac1b-4b11-96ee-f5e1bb13b0e3 | power on    | active             | False       |
| 599b79ad-c7e2-407e-94eb-91ec9c41c9cf | contrail-analytics-database-192_0_2_5 | 38a6996d-6bdd-4442-902c-a4a377921d68 | power on    | active             | False       |
| bc054f23-4256-43d5-b31d-fff6397f69c6 | contrail-analytics-192_0_2_5          | 3d25ff88-4e16-45c2-9566-a7824db9fcce | power on    | active             | False       |
| b93e4e72-8381-4ffa-a75a-77728795975a | appformix-controller-192_0_2_5        | None                                 | power off   | available          | False       |
+--------------------------------------+---------------------------------------+--------------------------------------+-------------+--------------------+-------------+
```

## Grab interface list from control host
```bash
(undercloud) [stack@undercloud ~]$ openstack baremetal introspection interface list r4u42-control
+-----------+-------------------+----------------------------+-------------------+----------------+
| Interface | MAC Address       | Switch Port VLAN IDs       | Switch Chassis ID | Switch Port ID |
+-----------+-------------------+----------------------------+-------------------+----------------+
| eno1      | 0c:c4:7a:ac:ff:c4 | [300]                      | 80:ac:ac:5b:10:80 | 591            |
| eno2      | 0c:c4:7a:ac:ff:c5 | [0]                        | 78:fe:3d:31:fc:80 | 577            |
| ens2f0    | 3c:fd:fe:9f:62:20 | [1008, 312, 313, 314, 319] | 80:ac:ac:5b:10:80 | 532            |
| ens2f1    | 3c:fd:fe:9f:62:21 | [1008, 312, 313, 314, 319] | 80:ac:ac:5b:10:80 | 530            |
+-----------+-------------------+----------------------------+-------------------+----------------+
```

## Show specific interface on control host
```bash
(undercloud) [stack@undercloud ~]$ openstack baremetal introspection interface show r4u42-control ens2f0
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                                | Value                                                                                                                                                                                  |
+--------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| interface                            | ens2f0                                                                                                                                                                                 |
| mac                                  | 3c:fd:fe:9f:62:20                                                                                                                                                                      |
| node_ident                           | r4u42-control                                                                                                                                                                          |
| switch_capabilities_enabled          | [u'Bridge', u'Router']                                                                                                                                                                 |
| switch_capabilities_support          | [u'Bridge', u'Router']                                                                                                                                                                 |
| switch_chassis_id                    | 80:ac:ac:5b:10:80                            
```
(Output was truncated)
---

# How to check DPDK interface status
**Tested on CC13.0.2 pre-release 341**

_Steps ommitted, connect to jumphost, then undercloud, then DPDK node, then DPDK agent container_

## Display DPDK status
```bash
(vrouter-agent-dpdk)[root@overcloudbnw-compdpdk-0 /]$ /opt/contrail/bin/dpdk_nic_bind.py --status

Network devices using DPDK-compatible driver
============================================
0000:02:00.0 'Ethernet Controller X710 for 10GbE SFP+' drv=vfio-pci unused=i40e

Network devices using kernel driver
===================================
0000:01:00.0 'I350 Gigabit Network Connection' if=eno1 drv=igb unused=vfio-pci *Active*
0000:01:00.1 'I350 Gigabit Network Connection' if=eno2 drv=igb unused=vfio-pci
0000:02:00.1 'Ethernet Controller X710 for 10GbE SFP+' if=ens2f1 drv=i40e unused=vfio-pci

Other network devices
=====================
<none>
---
```
