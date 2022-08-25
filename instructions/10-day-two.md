# 10 - Day two operations

There are many day 2 operations involved with the administration of JBoss EAP instances such as upgrades, patching, module deployment, configuration changes, fine tuning, and app deployment. The ansible-middleware collection can help perform some of these activities using built in tasks.  

In this workshop we're going to use ansible-middleware to download a patch for JBoss EAP and apply this patch to our installed servers.

The first thing we're going to do is check the current version of JBoss EAP deployed on our servers. To do this we will ssh into one of our jboss nodes (e.g. ssh app2.xxxxx.internal) and then run the following commands.

```
sudo su jboss
cd /opt/jboss-eap-7.4/bin
./jboss-cli.sh --connect
version
```

You should see a response similar to:

```
JBoss Admin Command-line Interface
JBOSS_HOME: /opt/jboss-eap-7.4
Release: 15.0.2.Final-redhat-00001
Product: JBoss EAP 7.4.0.GA
JAVA_HOME: null
java.version: 1.8.0_322
java.vm.vendor: Red Hat, Inc.
java.vm.version: 25.322-b06
os.name: Linux
os.version: 4.18.0-372.9.1.el8.x86_64
```

You'll see from here the Product is "JBoss EAP 7.4.0.GA" and the Release is "15.0.2.Final-redhat-00001".  We'll run these commands again once we've patched the system.

Enter "exit" three times to return to the bastion host.

To start the patching process, we're going to create a playbook to download the latest patch to our bastion host.  

Create a file called patch-download.yml and paste the following:

```
---
- name: "Download JBoss EAP patch"
  hosts: localhost
  become: yes
  collections:
   - middleware_automation.redhat_csp_download
  vars:
   zipfile_dest: "{{ lookup('env', 'PWD') }}/jboss-eap-{{ patch_version }}-patch.zip"
   rhn_download_url: "https://access.redhat.com/jbossnetwork/restricted/softwareDownload.html?softwareId={{ patch_id }}"
  tasks:
  - name: "Download zipfile from RHN: {{ rhn_download_url }}"
    middleware_automation.redhat_csp_download.redhat_csp_download:
     url: "{{ rhn_download_url }}"
     dest: "{{ zipfile_dest }}"
     username: "{{ rhn_username }}"
     password: "{{ rhn_password }}"

```

To run this playbook we will need to provide two additional ansible variables, patch_version and patch_id.  To download patch "7.4.5", we will use patch_id "104511".  You can find the patch_id from the patch download url at https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=appplatform&downloadType=patches&version=7.4. You will need your Red Hat network id to access this site.

As we did before, replace the rhn login and password with your own credentials and run the playbook.

`ansible-playbook -i ./inventory/hosts patch-download.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password> patch_version=7.4.5 patch_id=104511"`

Once this completes you should see the patch downloaded to the local folder; jboss-eap-7.4.5-patch.zip

Now that the patch file is downloaded we will create another playbook to apply the patch.  Create a new file patch-apply.yml and paste the following.

```
---
- name: "JBoss EAP patch"
  hosts: jboss
  become: yes
  vars:
   wildfly_install_workdir: '/opt/'
   install_name: "{{ override_install_name | default('jboss') }}"
   jboss_version: "7.4"
   wildfly_version: "{{ jboss_version }}"
   wildfly_user: "{{ install_name }}"
   wildfly_group: "{{ wildfly_user }}"
   wildfly_home: "{{ wildfly_install_workdir }}/{{ install_name }}-eap-{{ wildfly_version }}"
   wildfly_basedir_prefix: "/opt/{{ install_name }}"
   wildfly_config_name: "{{ install_name }}"
   wildfly_port_range_offset: 0
   wildfly_instance_name: "{{ inventory_hostname }}"
   wildfly_jboss_eap_apply_cp: True
   wildfly_jboss_eap_enable: True
   wildfly_offline_install: True
   instance_management_ports:
     - 9990
   rhn_cp_id: "{{ patch_id }}"
   rhn_cp_v: "{{ patch_version }}"
  collections:
   - middleware_automation.wildfly
  roles:
   - wildfly_install
```

Run this playbook with the following command 

```
`ansible-playbook -i ./inventory/hosts patch-apply.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>" patch_version=7.4.5 patch_id=104511"`
```

Once this completes, check the version of your JBoss EAP instances. To do this we will ssh into one of our jboss nodes (e.g. ssh app2.xxxxx.internal) and then run the following commands.

```
sudo su jboss
cd /opt/jboss-eap-7.4/bin
./jboss-cli.sh --connect
version
```

You should see a response similar to:
```
JBoss Admin Command-line Interface
JBOSS_HOME: /opt/jboss-eap-7.4
Release: 15.0.13.Final-redhat-00001
Product: JBoss EAP 7.4.5.GA
JAVA_HOME: null
java.version: 1.8.0_322
java.vm.vendor: Red Hat, Inc.
java.vm.version: 25.322-b06
os.name: Linux
os.version: 4.18.0-372.9.1.el8.x86_64
```

Note that the Product is now "JBoss EAP 7.4.5.GA" and Release is "15.0.13.Final-redhat-00001"

We can now be satisfied that the cummulative patch has been successfully deployed to the JBoss EAP servers.  

If we run this playbook a second time, no changes are made which demonstates Ansible's idempotenty.  

This is a very simple example of performing day 2 operations with Ansible, there are many other ways Ansible can be leveraged for these tasks.  For example, we could configure Ansible to take a backup of the current deployment prior to upgrade, and then in the case of failure, restore the system to it's previous working state.

Next [Step 11](./11-conclusion.md)

