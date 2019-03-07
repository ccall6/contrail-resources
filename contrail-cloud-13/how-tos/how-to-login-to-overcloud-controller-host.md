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
