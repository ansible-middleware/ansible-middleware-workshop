# 3 - Installing the postgresql role

Our workshop uses a postgresql database.  We will use the postgresql role to install and configure the postgresql database.  

First thing we need to do is create a "roles" folder.  To do this, run the following command from the root of the workshop folder:

`mkdir roles`

We have included the boiler plate postgresql role files in an archive in the workshop folder.  We will extract this archive to the roles folder.  To do this, run the following command:

`unzip workshop/postgresql.zip -d roles`

You should now see a postgresql folder in the roles folder.  We shouldn't need to make any changes to these files.  Before we move on, let's take a look at what these files are doing.

There are two main files we'll look at:

* defaults/main.yml
* tasks/main.yml

## Reviewing the postgresql role

### defaults/main.yml

The defaults/main.yml file is the main file that contains the default variables that are used by the postgresql role.  

Included in these variables are the following:

* pgdata location override option, using environment variable overrides_pgsql_pgdata
* packages_list, which is a list of packages to install
* db variable which contains a list of database users e.g. db.name, db.user, db.password

### tasks/main.yml

The tasks/main.yml file contains the tasks that are used to perform the postgresql role.  The tasks are as follows:

* Install the postgresql packages.  This task uses the fastpackages role to install the packages required by postgresql.  The list of packages are defined in the packages_list variable.

* Init PostgresQL Db. This task initializes the postgresql database.  The database is initialized using the initdb command.  The database is initialized using the following command: /usr/bin/postgresql-setup initdb

* Ensures postgresql.service.d/ dir exists.  This task ensures that the postgresql.service.d/ dir exists.  

* Installs an update to the stop.conf file as a workaround to a known issue referenced in https://access.redhat.com/solutions/5605181

* Configures postgres to accept connections from the app servers.  pg_hba.conf is configured with the ip addresses from the other app servers

* Updates postgresql.conf to accept remote connections

* Checks the Postgresql service is running

* Creates the postgresql databases.  This task creates the databases defined in the db variable as an array.

* Created users for each database.  The users are defined in the db variable as an array.

* Install and start the firewalld service.

* Inserts a rule to allow the app servers access to the postgresql service on port 5432


## Installing the fastpackages role

The postgresql role requires the fastpackes role to install the packages required by postgresql.  This role is included in the workshop folder.  We will extract this archive to the roles folder.  To do this, run the following command: 

`unzip workshop/fastpackages.zip -d roles`

## Create the postgresql playbook

To use the postgresql role, we need to create a playbook.  We will create a playbook called `postgresql.yml`.  This playbook will contain the tasks that are required to install the postgresql role.  The playbook will be as follows:

```
---
- name: Playbook for Postgresql Setup
  hosts: pgsql
  become: true
  tasks:
    - name: pgsql Role
      include_role:
        name: postgresql
```

## Run the postgresql playbook

To test the postgresql playbook, we will run the following command:

`ansible-playbook -i ./inventory/hosts postgresql.yml`

Once this command has completed, you should be able to connect to and test the postgresql service.  To do this, connect to the postgresql server using ssh.  Open inventory/hosts and find the ip address listed under the pgsql group.  Connect to this host with the command (replace xxxxx with the guid of your workshop, you can find this from the previous ansible command)

`ssh appdb1.xxxxx.internal`

Once connected to the postgresql server, you should be able to connect to the postgresql service using the following command: 

`psql -U wildfly` 

Once connected to postgresql, you should see the following output:

```
wildfly=>
```

At this prompt you should be able to list the created databases.  To do this, run the following command:

`SELECT datname FROM pg_database;`

Once this command has completed, you should see the following output:

```
  datname  
-----------
 postgres
 template1
 template0
 keycloak
 wildfly
(5 rows)
```

This shows that postgresql is installed and configured correctly with the correct databases created.

To quit the postgresql client, enter the following command:

`\q`

To return to the bastion host enter

`exit`

Our postgresql database is now installed and configured, we can now move on to the next step, installing JBoss EAP.

Next [Step 4](./4-deploying-jboss-eap.md)