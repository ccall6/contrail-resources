# How to login to overcloud compute node
**Tested on CC13.0.2**

## On jumphost, connect as contrail user
```bash
[root@r4-ru43 ~]# su contrail
[contrail@r4-ru43 root]$
```

## SSH to undercloud VM
```bash
[contrail@r4-ru43 root]$ ssh undercloud
Last login: Wed Mar  6 13:00:32 2019 from 192.168.122.1
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
| 695175e2-6af9-465a-8734-9e924fd27a70 | overcloudt6n-ca-0   | ACTIVE | ctlplane=192.0.2.53 | overcloud-full | contrail-analytics          |
| 6aef4878-0158-4839-bce3-d1cfe541b2ee | overcloudt6n-cadb-0 | ACTIVE | ctlplane=192.0.2.52 | overcloud-full | contrail-analytics-database |
| 7c137de9-a850-4bd3-9590-744bdd04d6fd | overcloudt6n-ctrl-0 | ACTIVE | ctlplane=192.0.2.61 | overcloud-full | control                     |
| 4e11738b-bdb4-4b6a-a960-4dfafe9c1cb3 | overcloudt6n-comp-0 | ACTIVE | ctlplane=192.0.2.51 | overcloud-full | compute                     |
| cd391dd0-e165-4a04-b310-04305e9c5c4c | overcloudt6n-cc-0   | ACTIVE | ctlplane=192.0.2.57 | overcloud-full | contrail-controller         |
+--------------------------------------+---------------------+--------+---------------------+----------------+-----------------------------+
```

## SSH to compute server address as heat-admin user
```bash
(undercloud) [stack@undercloud ~]$ ssh heat-admin@192.0.2.51
Warning: Permanently added '192.0.2.51' (ECDSA) to the list of known hosts.
Last login: Wed Mar  6 12:39:11 2019 from 192.0.2.1
[heat-admin@overcloudt6n-comp-0 ~]$
```
