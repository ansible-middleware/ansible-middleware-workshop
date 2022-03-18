# 2 - Adding the ansible collections.

This workshop uses the following collections:

* middleware_automation.jcliff: This collection is used to perform configuration of the JBoss EAP instance.  In this workshop we will use jcliff to install postgresql drivers, postgresql datasource, and configure mod_cluster.  For the purpose of this workshop we will use the following version: 0.0.21.  For more information see https://ansible-middleware.github.io/ansible_collections_jcliff/latest/

* middleware_automation.wildfly: This collection is used to perform the installation and configuration of the JBoss EAP instance.  For the purpose of this workshop we will use the following version: 0.0.4.  For more information see https://ansible-middleware.github.io/wildfly/main/


* middleware_automation.redhat_csp_download: This collection is used to perform the installation and configuration of Jboss core services, which we will use for load balancing.  For the purpose of this workshop we will use the following version: 1.2.1. For more information see https://ansible-middleware.github.io/redhat-csp-download/latest/


* community.postgresql: This collection is used to perform the installation and configuration of postgresql.  For more information see https://docs.ansible.com/ansible/latest/collections/community/postgresql/index.html

* community.general: This collection is requred by the moocule testing. For more information see https://docs.ansible.com/ansible/latest/collections/community/general/index.html 

To add these collections to your project, copy and paste the following in the file /molecule/default/requirements.yml

```
---
collections:
  - name: middleware_automation.jcliff
    version: ">=0.0.21"
  - name: middleware_automation.wildfly
    version: "==0.0.4"
  - name: community.general
  - name: community.postgresql
  - name: middleware_automation.redhat_csp_download
    version: ">=1.2.1"
```

Save changes to this file, and run the following command to install the collections: 

`ansible-galaxy collection install -r molecule/default/requirements.yml`

Once this command has completed, you should see the following output:

```
Process install dependency map
Starting collection install process
Installing 'middleware_automation.jcliff:0.0.21' to '/home/xxx/.ansible/collections/ansible_collections/middleware_automation/jcliff'
Installing 'middleware_automation.wildfly:0.0.4' to '/home/xxx/.ansible/collections/ansible_collections/middleware_automation/wildfly'
Installing 'community.general:4.6.0' to '/home/xxx/.ansible/collections/ansible_collections/community/general'
Installing 'community.postgresql:2.1.1' to '/home/xxx/.ansible/collections/ansible_collections/community/postgresql'
Installing 'middleware_automation.redhat_csp_download:1.2.1' to '/home/xxx/.ansible/collections/ansible_collections/middleware_automation/redhat_csp_download'
```

To check the status of the collections, run the following command: 

`tree -L 2 ~/.ansible/collections/ansible_collections/`

Once this command has completed, you should see the following output:
```
/home/xxx/.ansible/collections/ansible_collections/
├── community
│   ├── general
│   └── postgresql
└── middleware_automation
    ├── jcliff
    ├── redhat_csp_download
    └── wildfly

```

The collections are installed, we'll now proceed to the next step, installing a postgresql database.

Next [Step 3](./3-configuring-postgresql.md)