# 9 - Combining the playbooks

The last step of the workshop is to tie everything together.  To do this, we'll create a playbook called demo.yml, and paste the following into it:

```
---
# Postgres playbook
- import_playbook: postgresql.yml

# JBoss Core Services (Apache HTTPD)
- import_playbook: jbcs.yml

# Wildfly Playbook
- import_playbook: jboss.yml

```

This playbook will run the postgresql, jbcs, and jboss playbooks in sequence.  The jbcs playbook will install the load balancer, and the jboss playbook will install the JBoss instances and addressbook application.  The postgresql playbook will install the postgresql database.

Save this file and run the playbook with the following command:

`ansible-playbook -i ./inventory/hosts demo.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>" `

# Conclusion

You can download the completed workshop from here:  https://github.com/deewhyweb/wildfly-cluster-demo/tree/postgresql

The workshop is now complete, we've demonstrated the use of the ansible-middleware collection to deploy JBoss EAP, a J2EE application, and JBoss Core Services as a load balancer.  Persistence is provided by a postresql database.

Next [Step 10](./10-day-two.md)