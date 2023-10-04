# 5 - Configuring JBoss EAP

Now that JBoss EAP is installed, we now need to configure it for the following:

* The JBoss EAP instances will need to connect to a shared postgresql instance to persist data.  JBoss EAP will need to be configured with the Postgresql driver and datasource. 
* The JBoss EAP instances will need to use mod_cluster to register themselves with the Apache HTTPS server installed by JBCS.  This will enable the HTTP server to load balance connections to the JBoss EAP instances.

To configure JBoss EAP to connect to the PostgreSQL database we're going to download and install PostgreSQL jdbc drivers.  The ansible-middleware collection provides a mechanism to do this, using the eap_driver role.  We'll add these tasks to the jboss.yml by adding the following to the end of jboss.yml:


``` yaml
  tasks:
    - name: "Set up for JBoss module"
      include_role:
        name: eap_driver
```

To use these tasks we need to add the required ansible variables to identify and download the jdbc drivers.


``` yaml
    eap_driver_module_name: 'org.postgresql'
    eap_driver_version: 9.4.1212
    eap_driver_jar_filename: "postgresql-{{ eap_driver_version }}.jar"
    eap_driver_jar_url: "https://repo.maven.apache.org/maven2/org/postgresql/postgresql/{{ eap_driver_version }}/{{ eap_driver_jar_filename }}"
```

Add the above variables to the jboss.yml file to the end of the vars section.


## Configuring with yaml

We also need to create a postgresql datasource and configure the connection to the PostgreSQL database.  We're going to use yaml to do this.

Add the following variables to the jboss.yml file.

``` yaml
    eap_enable_yml_config: True
    eap_yml_configs:
      - eap_configuration.yml.j2
```

Create a new file called eap_configuration.yml.j2 and copy the following snippet to this file:

``` yaml
wildfly-configuration:
  subsystem:
    datasources:
      jdbc-driver:
        postgresql:
          driver-name: postgresql
          driver-xa-datasource-class-name: org.postgresql.xa.PGXADataSource
          driver-module-name: org.postgresql
          driver-class-name: org.postgresql.Driver
      data-source:
        PostgreSQLDS:
          enabled: true
          exception-sorter-class-name: org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLExceptionSorter
          jndi-name: java:jboss/datasources/{{ postgres.jndi_ds }}
          jta: true
          max-pool-size: 20
          min-pool-size: 0
          connection-url: "jdbc:postgresql://{{ (groups['pgsql'][0] if groups['pgsql'] | length > 0 else 'localhost') }}:5432/{{ postgres.db_name }}"
          driver-name: postgresql
          user-name: "{{ postgres.user.name }}"
          password: "{{ postgres.user.password }}"
          validate-on-match: true
          background-validation: false
          background-validation-millis: 10000
          flush-strategy: FailingConnectionOnly
          statistics-enable: false
          stale-connection-checker-class-name: org.jboss.jca.adapters.jdbc.extensions.novendor.NullStaleConnectionChecker
          valid-connection-checker-class-name: org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLValidConnectionChecker
          transaction-isolation: TRANSACTION_READ_COMMITTED

```

Save changes to jboss.yml and re-run the jboss playbook.

`ansible-playbook -i ./inventory/hosts jboss.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>"`

If you're familiar with JBoss EAP configuration using the cli, you'll recognise these configurations. 


## Testing the JBoss EAP Configuration

To test the configuration we just performed we'll first of all ssh into one of the JBoss EAP instances.  To do this, run the following:

`ssh xxx@<ip-address-of-node>`

Once you're logged in, change user to the jboss user with the following command:

`sudo su eap`

Then run the following commands to use the JBoss cli:

`cd /opt/jboss_eap/jboss-eap-7.4/bin`

`./jboss-cli.sh --connect`

`/subsystem=datasources:installed-drivers-list`

The output should show the postgresql driver.  Something like

```
{
    "outcome" => "success",
    "result" => [
        {
            "driver-name" => "postgresql",
            "deployment-name" => undefined,
            "driver-module-name" => "org.postgresql",
            "module-slot" => "main",
            "driver-datasource-class-name" => "",
            "driver-xa-datasource-class-name" => "org.postgresql.xa.PGXADataSource",
....

```

To return to the bastion node, run the following command three times:

`exit`

This confirms the postgresql driver is installed; now exit the JBoss cli, logout from the node and go back to the ansible directory.

## Integrating with JBoss Core Services

To be able to access our JBoss EAP clusters from the outside world we need to integrate with the HTTP server installed by JBCS.  To do this we need to update the eap_configuration.yml.j2 file to add mod_cluster configuration.

Add the following to the bottom of the file:


``` yaml
    modcluster:
      proxy:
        default:
          advertise-socket: modcluster
          proxies:
            - proxy1
          listener: ajp
  socket-binding-group:
    standard-sockets:
      remote-destination-outbound-socket-binding:
        proxy1:
          host: {{ groups['jbcs'][0] }}
          port: 6666

```

 These configurations add an entry to the socket-binding-group called proxy1 which points to the HTTP server installed on the frontend server.  Mod_cluster is then configured to use this connection as the default proxy.


The eap_configuration.yml.j2 file should now look like:

``` yaml
wildfly-configuration:
  subsystem:
    datasources:
      jdbc-driver:
        postgresql:
          driver-name: postgresql
          driver-xa-datasource-class-name: org.postgresql.xa.PGXADataSource
          driver-module-name: org.postgresql
          driver-class-name: org.postgresql.Driver
      data-source:
        PostgreSQLDS:
          enabled: true
          exception-sorter-class-name: org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLExceptionSorter
          jndi-name: java:jboss/datasources/{{ postgres.jndi_ds }}
          jta: true
          max-pool-size: 20
          min-pool-size: 0
          connection-url: "jdbc:postgresql://{{ (groups['pgsql'][0] if groups['pgsql'] | length > 0 else 'localhost') }}:5432/{{ postgres.db_name }}"
          driver-name: postgresql
          user-name: "{{ postgres.user.name }}"
          password: "{{ postgres.user.password }}"
          validate-on-match: true
          background-validation: false
          background-validation-millis: 10000
          flush-strategy: FailingConnectionOnly
          statistics-enable: false
          stale-connection-checker-class-name: org.jboss.jca.adapters.jdbc.extensions.novendor.NullStaleConnectionChecker
          valid-connection-checker-class-name: org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLValidConnectionChecker
          transaction-isolation: TRANSACTION_READ_COMMITTED
    modcluster:
      proxy:
        default:
          advertise-socket: modcluster
          proxies:
            - proxy1
          listener: ajp
  socket-binding-group:
    standard-sockets:
      remote-destination-outbound-socket-binding:
        proxy1:
          host: {{ groups['jbcs'][0] }}
          port: 6666
```

Re-run the playbook to configure the JBCS integration.

## Testing the JBCS integration

We should be able to access the JBoss EAP landing pages from the frontend url.  From your browser navigate to the frontend url listed in your workshop email.

Next [Step 7](./7-deploying-applications.md) will deploy the application.


