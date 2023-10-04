# 4 - Deploying JBoss EAP

Next thing we're going to do is deploy JBoss EAP.  To do this we'll create a jboss playbook by creating a file jboss.yml in the top level folder.  This playbook is going to use the eap_install and eap_systemd roles from the rehat.eap collection.

Copy the contents of the following file into jboss.yml:

``` yaml
---
- name: "Deploy a JBoss EAP"
  hosts: jboss
  become: true
  vars:
    eap_apply_cp: true
    eap_version: 7.4.0
    eap_offline_install: false
    eap_config_base: 'standalone-full-ha.xml'
  collections:
    - redhat.eap
  roles:
    - eap_install
    - eap_systemd
```

We are setting some variables to configure the install process.

* eap_apply_cp: We are setting this to true to ensure all available JBoss EAP patches are downloaded and applied.
* eap_version: we are setting this to 7.4.0 to determine the starting version of JBoss EAP to deploy.
* eap_offline_install: This is set to "false" which will ensure JBoss EAP and patches are downloaded from Red Hat Network.
* eap_config_base: We are setting this to "standalone-full-ha.xml", we need to use this profile to enable the mod_cluster subsystem.


Note that the Ansible collection for JBoss EAP will also take care of downloading the required assets from the Red Hat Customer Portal (the archive containing the Java app server files). However, one does need to provide the credentials associated with a service account. You can manage service accounts using the hybrid cloud console. Within this portal, on the [service accounts tab](https://console.redhat.com/application-services/service-accounts), you can create a new service account if one does not already exist.

To run the playbook we'll run the following command, you will need your Red Hat network credentials that we created in section 1: 

`ansible-playbook -i ./inventory/hosts jboss.yml -e rhn_username=<client_id> -e rhn_password=<client_secret>"`

Once this command completes, you should see something like:

```

PLAY RECAP ***************************************************************************************************************
192.168.122.224            : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0   
192.168.122.64             : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0   

```

## Testing the installation

Run `curl http://app1.xxxxx.internal:8080/|grep "<h3>"` in the terminal.  You should see the following:

```
curl app1.xxxxx.internal:8080/ | grep "<h3>"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1531  100  1531    0     0   373k      0 --:--:-- --:--:-- --:--:--  498k
      <h3>Your Red Hat JBoss Enterprise Application Platform is running.</h3>
      ```

Now that JBoss EAP is installed, let's move on to the next section, configuring JBoss EAP.

Next [Step 6](./6-configuring-jboss-eap.md)