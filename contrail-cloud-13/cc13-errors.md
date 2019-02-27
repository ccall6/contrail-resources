# Installation error: "AnsibleUndefinedVariable" error
**Seen with Contrail Cloud 13.0.1**
**Seen with openstack-deploy.sh**

## Error message
```
failed: [192.168.122.58] (item=post_deployment.yaml) => {"changed": false, "failed": true, "item": "post_deployment.yaml", "msg": "AnsibleUndefinedVariable: 'None' has no attribute 'mysql'"}
```

## Resolution
The problem was due to incorrect YAML syntax in one of my config files. There is no syntax checking in the scripts themselves. If you have invalid YAML (in my case, an invalid mapping statement caused by comments/deleted lines), then you'll likely get a weird YAML error like this. To resolve, validate the syntax of the YAML file.

_Note: If it's complaining about 'ansible_default_ipv4', then that's something different. See below._

---

# Installation error: Repository error
**Seen with Contrail Cloud 13.0.1**

## Error message
```
"stdout": "Error: rhel-ha-for-rhel-7-server-rpms is not a valid repository ID. Use --list option to see valid repositories.\nRepository 'rhel-7-server-rpms' is enabled for this system.\nRepository 'rhel-7-server-extras-rpms' is enabled for this system.\nRepository 'rhel-7-server-rh-common-rpms' is enabled for this system.",
"stdout_lines": [
    "Error: rhel-ha-for-rhel-7-server-rpms is not a valid repository ID. Use --list option to see valid repositories.",
    "Repository 'rhel-7-server-rpms' is enabled for this system.",
    "Repository 'rhel-7-server-extras-rpms' is enabled for this system.",
    "Repository 'rhel-7-server-rh-common-rpms' is enabled for this system."
```

## Resolution
Registration error problem caused by manual changes to the repositories. Redoing the initial setup script fixed it.

---

# Installation error: control-hosts-deploy.sh script can't ping nodes
**Seen with Contrail Cloud 13.0.1**
**Seen with control-hosts-deploy.sh script**

## Error message
```
TASK [wait for node ssh (pre os-net-config)] ***********************************
Friday 26 October 2018  21:42:24 -0700 (0:00:00.067)       0:37:47.331 ********
fatal: [192.0.2.5]: FAILED! => {"changed": false, "elapsed": 603, "failed": true, "msg": "timed out waiting for ping module test success: SSH Error: data could not be sent to remote host \"192.0.2.5\". Make sure this host can be reached over ssh"}
```

## Resolution
Problem was caused by issue on jumphost itself due to using it manually as PXE boot server. I wiped it with fresh install of RHEL 7.5 and reinstalled and the error went away.

---

# Installation error: PreNetworkConfig CREATE_FAILED
**Seen with Contrail Cloud 13.0.1**

## Error Message
```
TASK [overcloud : debug] *******************************************************
task path: /var/lib/contrail_cloud/ansible/playbooks/roles/overcloud/tasks/status.yml:24
Friday 02 November 2018  13:16:57 -0700 (0:00:06.629)       4:13:04.010 *******
ok: [192.168.122.7] => {
    "msg": [
        "overcloud.Compute.0.PreNetworkConfig:",
        "  resource_type: OS::TripleO::Compute::PreNetworkConfig",
        "  physical_resource_id: 48d373a8-4f0f-4bbe-8dfa-4ba708441353",
        "  status: CREATE_FAILED",
        "  status_reason: |",
        "    resources.PreNetworkConfig: Stack CREATE cancelled"
    ]
}
```

## Resolution
The wrong interface name was in the overcloud_nics.yml file. Fixed that but still saw this error. The core problem turned out to be that the compute server was stuck at an EFI prompt because it was not set to book as HD as 2nd boot. Changed the boot order to be 1st PXE Boot and 2nd HD Boot and this error went away.

---

# Installation error: Heat StructureDeployment CREATE_FAILED because something isn't pingable
**Seen with Contrail Cloud 13.0.1**

## Error Message
```
overcloud.ComputeAllNodesValidationDeployment.0:",
        "  resource_type: OS::Heat::StructuredDeployment",
        "  physical_resource_id: cb32958b-d634-40f5-ab8f-c9b6fc8175aa",
        "  status: CREATE_FAILED",
        "  status_reason: |",
        "    Error: resources[0]: Deployment to server failed: deploy_status_code : Deployment exited with non-zero status code: 1",
        "  deploy_stdout: |",
        "    Trying to ping 172.16.0.105 for local network 172.16.0.0/24.",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    Ping to 172.16.0.105 failed. Retrying...",
        "    FAILURE",
        "  deploy_stderr: |",
        "    172.16.0.105 is not pingable. Local Network: 172.16.0.0/24"
```

## Resolution
This was an infrastructure problem. The script wants to ping the default gateway for the Tenant and External network. I had to configure these correctly on the MX, the underlay switch, and with the site.yml and overcloud_nics.yml files.

---

# Installation error: NTP Sync error
**Seen with Contrail Cloud 13.0.1**

## Error Message
```
TASK [Ensure system is NTP time synced] ****************************************",
       "    fatal: [localhost]: FAILED! => {\"changed\": true, \"cmd\": [\"ntpdate\", \"-u\", \"45.33.48.4\"], \"delta\": \"0:00:08.104502\", \"end\": \"2018-11-09 10:57:02.699800\", \"failed\": true, \"msg\": \"non-zero return code\", \"rc\": 1, \"start\": \"2018-11-09 10:56:54.595298\", \"stderr\": \" 9 Nov 10:57:02 ntpdate[10068]: no server suitable for synchronization found\", \"stderr_lines\": [\" 9 Nov 10:57:02 ntpdate[10068]: no server suitable for synchronization found\"], \"stdout\": \"\", \"stdout_lines\": []}",
       "    \tto retry, use: --limit @/var/lib/heat-config/heat-config-ansible/11a6d9ef-cdb6-49d6-a816-41c245b17dc3_playbook.retry",
       "    ",
       "    PLAY RECAP *********************************************************************",
       "    localhost                  : ok=30   changed=21   unreachable=0    failed=1   ",
       "    ",
```

## Resolution
The path to the Internet via the External network was faulty. There was a default route on the MX SDN Gateway pointing out fxp0. I got rid of that and the problem went away.

---

# Installation error: Undefined variable complaint about ansible_default_ipv4
**Seen with Contrail Cloud 13.0.2 pre-release 341**
**Seen with install_contrail_cloud_manager.sh script**

## Error Message
```
fatal: [192.168.122.225]: FAILED! => {
    "failed": true,
    "msg": "The task includes an option with an undefined variable. The error was: 'dict object' has no attribute 'ansible_default_ipv4'\n\nThe error appears to have been in '/var/lib/contrail_cloud/ansible/playbooks/install_contrail_cloud_manager.yml': line 111, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n      when: jumphost_port.rc != 0\n    - name: Update host file on undercloud\n      ^ here\n\nexception type: <class 'ansible.errors.AnsibleUndefinedVariable'>\nexception: 'dict object' has no attribute 'ansible_default_ipv4'"
}
```

## Resolution
There was an extra file in the /var/lib/contrail_cloud/ansible/playbooks/inventory directory: undercloud.rpmnew. This file contained an IP address for a non-existent host. When the script tried to contact that host, the above error was generated. The fix is to delete the undercloud.rpmnew file from that directory. Note that this file appears to be left in that directory when the install_contrail_cloud_manager.sh script ends unsuccessfully, so don't be shocked if you have to delete it more than one time.

---

# Installation error: Containers not updating as part of upgrade
**Seen with Contrail Cloud 13.0.2 pre-release 341**
**Seen with install_contrail_cloud_manager.sh**

## Error
As part of the upgrade, the new containers should be downloaded by the undercloud.

## Resolution
There was a problem in the satellite settings, but we had also run the subscription-manager register command on the jumphost rather than the undercloud VM. It needs to be run on the undercloud VM. You can check the repolist to see if the new containers have correctly been registered or not.

---

# Horizon error: Unable to retrieve usage information (etc)
**Seen with Contrail Cloud 13.0.2 pre-release 341**

## Error
### In Horizon GUI
Unable to retrieve usage information
Unable to retrieve limit information
Unable to retrieve network quota information
Unable to retrieve volume limit information

### In /var/log/container/horizon/horizon.log on OpenStack controller VM
```
2018-11-20 06:22:31,957 85 ERROR openstack_auth.user Unable to retrieve project list.
Traceback (most recent call last):
  File "/usr/lib/python2.7/site-packages/openstack_auth/user.py", line 350, in authorized_tenants
    is_federated=self.is_federated)
  File "/usr/lib/python2.7/site-packages/openstack_auth/utils.py", line 372, in get_project_list
    projects = client.projects.list(user=kwargs.get('user_id'))
  File "/usr/lib/python2.7/site-packages/keystoneclient/v3/projects.py", line 138, in list
    **kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneclient/base.py", line 75, in func
    return f(*args, **new_kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneclient/base.py", line 397, in list
    self.collection_key)
  File "/usr/lib/python2.7/site-packages/keystoneclient/base.py", line 125, in _list
    resp, body = self.client.get(url, **kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneauth1/adapter.py", line 304, in get
    return self.request(url, 'GET', **kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneauth1/adapter.py", line 463, in request
    resp = super(LegacyJsonAdapter, self).request(*args, **kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneauth1/adapter.py", line 189, in request
    return self.session.request(url, method, **kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneauth1/session.py", line 698, in request
    resp = send(**kwargs)
  File "/usr/lib/python2.7/site-packages/keystoneauth1/session.py", line 760, in _send_request
    raise exceptions.SSLError(msg)
SSLError: SSL exception connecting to https://10.1.151.50:13000/v3/users/7357b4336ea74680b7b549bc75010226/projects?: hostname '10.1.151.50' doesn't match either of '10.87.4.188', '10.87.4.188'
```

## Resolution
I hadn't set the overcloud/tls/common_name value in site.yml and it was still default. I needed to set it to the VIP of my External network (10.1.151.50). To fix, I had to edit the site.yml file, delete all the certs from the /var/lib/contrail_cloud/certs directory, run clean with openstack-deploy.sh and then reploy with openstack-deploy.sh.

## Notes
- Tearing down and rebuilding the overcloud only didn't work.
- Rerunning the control-vms-deploy script by itself didn't work.
- I tried cleaning up both openstack-deploy.sh and control-vms-deploy.sh and it didn't work. It looks like the certs are only generated when they aren't present.
- I tried deleting the cert directory and rerunning openstack-deploy, but it wasn't enough.

---

# Instance creation error from GUI: Build of instance ... did not finish ...
**Seen with Contrail Cloud 13.0.2 pre-release 341**

## Error
_From /var/log/containers/nova/nova-compute.log on compute node_
 BuildAbortException: Build of instance 13f21f06-fa8d-40ab-9434-418fa7312696 aborted: Volume 92f5a23e-46f2-4e2d-896e-bf8e35a767d4 did not finish being created even after we waited 0 seconds or 1 attempts. And its status is error.

## Resolution
The GUI has the option set to create a new volume by default, whereas the CLI does not. I had no storage nodes so no place for a volume to be installed. I unchecked that and it went away.

## Reference
[https://ask.openstack.org/en/question/102998/block-device-mapping-is-invalid/]

---

# Installation error: "overcloud.Compute.0.NetworkDeployment" CREATE_FAILED
**Seen with Contrail Cloud 13.0.2**
**Seen with openstack-deploy.sh**

## Error message
```
"overcloud.Compute.0.NetworkDeployment:",
    "  resource_type: OS::TripleO::SoftwareDeployment",
    "  physical_resource_id: 9dc2237e-98c3-4efe-a2fd-35651ff9aebe",
    "  status: CREATE_FAILED",
    "  status_reason: |",
    "    Error: resources.NetworkDeployment: Deployment to server failed: deploy_status_code : Deployment exited with non-zero status code: 1"
```
Also seen:
```
         "overcloud.ContrailController.0.NetworkDeployment:",
         "  resource_type: OS::TripleO::SoftwareDeployment",
         "  physical_resource_id: de9c8eec-eb9e-4d85-b5b5-a66e70b4124a",
         "  status: CREATE_FAILED",
         "  status_reason: |",
         "    Error: resources.NetworkDeployment: Deployment to server failed: deploy_status_code : Deployment exited with non-zero status code: 1"
```

## Resolution
There were old "failed" servers that were still connected to the OpenStack VLANs on the switch. After removing the server connections to the VLANs, the error message went away. My theory is that one or more of those servers wasn't as dead as thought and was ARPing to its old address, causing contention with new servers.
---
