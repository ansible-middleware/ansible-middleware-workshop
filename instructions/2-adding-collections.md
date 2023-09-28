# 2 - Adding the ansible collections.

This workshop uses the following collections:

* [middleware_automation.jcliff](https://ansible-middleware.github.io/ansible_collections_jcliff/latest/): This collection is used to perform configuration of the JBoss EAP instance.  In this workshop we will use jcliff to install postgresql drivers, postgresql datasource, and configure mod_cluster.  For the purpose of this workshop we will use the following version: 0.0.21.  
* [middleware_automation.wildfly](https://ansible-middleware.github.io/wildfly/latest/): This collection is used to perform the installation and configuration of the JBoss EAP instance.  For the purpose of this workshop we will use the following version: 0.0.4. 


* [middleware_automation.redhat_csp_download](https://ansible-middleware.github.io/redhat-csp-download/latest/): This collection is used to perform the installation and configuration of Jboss core services, which we will use for load balancing.  For the purpose of this workshop we will use the following version: 1.2.1. 


* [community.postgresql](https://docs.ansible.com/ansible/latest/collections/community/postgresql/index.html): This collection is used to perform the installation and configuration of postgresql.  

* [community.general](https://docs.ansible.com/ansible/latest/collections/community/general/index.html ): This collection is requred by the moocule testing. 

First we need to configure ansible to use the Red Hat Automation hub

Create a file called ansible.cfg and paste the following  

```
#ansible.cfg:
[defaults]
host_key_checking = False
retry_files_enabled = False
nocows = 1

[inventory]
# fail more helpfully when the inventory file does not parse (Ansible 2.4+)
unparsed_is_failed=true

[galaxy]
server_list = automation_hub, galaxy
[galaxy_server.galaxy]
url=https://galaxy.ansible.com/
[galaxy_server.automation_hub]
url=https://cloud.redhat.com/api/automation-hub/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=<paste-your-token-here>
```

Replace "paste-your-token-here" with a token retrieved from console.redhat.com by following these instructions:

* Navigate to https://cloud.redhat.com/ansible/automation-hub/token/.
* Click Load Token.
* Click copy icon to copy the API token to the clipboard.

To add these collections to your project, copy and paste the following in the file collections/requirements.yml

```
---
collections:
  - name: redhat.eap
    version: "==1.4.3"
  - name: community.general
    version: "==7.4.0"
  - name: community.postgresql
    version: "==3.2.0"
  - name: redhat.redhat_csp_download
```

Save changes to this file, and run the following command to install the collections: 

`ansible-galaxy collection install -r ./collections/requirements.yml`

Once this command has completed, you should see the following output:

```
Process install dependency map
Starting collection install process
Installing 'redhat.eap:1.4.3' to '/home/devops/.ansible/collections/ansible_collections/redhat/eap'
Installing 'community.general:7.4.0' to '/home/devops/.ansible/collections/ansible_collections/community/general'
Installing 'community.postgresql:3.2.0' to '/home/devops/.ansible/collections/ansible_collections/community/postgresql'
Installing 'ansible.posix:1.5.4' to '/home/devops/.ansible/collections/ansible_collections/ansible/posix'
Installing 'redhat.runtimes_common:1.1.3' to '/home/devops/.ansible/collections/ansible_collections/redhat/runtimes_common'
```

To check the status of the collections, run the following command: 

`tree -L 2 ~/.ansible/collections/ansible_collections/`

Once this command has completed, you should see the following output:
```
/home/devops/.ansible/collections/ansible_collections/
├── ansible
│   └── posix
├── community
│   ├── general
│   └── postgresql
└── redhat
    ├── eap
    └── runtimes_common

```

The collections are installed, we'll now proceed to the next step, installing a postgresql database.

Next [Step 3](./3-configuring-postgresql.md)