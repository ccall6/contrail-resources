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
