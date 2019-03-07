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
