# 5 - Configuring JBoss EAP
The JBoss EAP instances will need to connect to a shared postgresql instance to persist data.  JBoss EAP will need to be configured with the Postgresql driver and datasource.

To configure JBoss EAP we're going to download and install postgresql jdbc drivers.  The ansible-middleware collection provides a mechanism to do this, using the widlfly_driver role.  We'll add these tasks to the jboss.yml by adding the following to the tasks section after the "Set up for JBoss instance" task:


```
    - name: "Set up for JBoss module"
      include_role:
        name: eap_driver
```

To use these tasks we need to add the required ansible variables to identify and download the jdbc drivers.


```
    eap_driver_module_name: 'org.postgresql'
    eap_driver_version: 9.4.1212
    eap_driver_jar_filename: "postgresql-{{ eap_driver_version }}.jar"
    eap_driver_jar_url: "https://repo.maven.apache.org/maven2/org/postgresql/postgresql/{{ eap_driver_version }}/{{ eap_driver_jar_filename }}"
```

Add the above environment variables to the jboss.yml file to the end of the vars section, and rerun the playbook to deploy the JDBC driver.


## Configuring with yaml

Add the following

    eap_enable_yml_config: True
    eap_yml_configs:
      - eap_configuration.yml.j2

Create a file eap_configuration.yml.j2

```
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

If you're familiar with JBoss EAP configuration using the cli, you'll recognise these configurations. 

<!-- In this instance we're passing the correct management port to JCliff, then using JCliff to add a postresql driver and datasource.  We're also using the loop task to iterate over the management ports, in our case we only have one instance per node, so a single management port. -->

Re-run the playbook to see the changes.

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
            "driver-xa-datasource-class-name" => "org.postgresql.xa.PGXADataSour
ce",
....

```

To return to the bastion node, run the following command three times:

`exit`

This confirms the postgresql driver is installed; now exit the JBoss cli, logout from the node and go back to the ansible directory.

Next [Step 6](./6-deploying-applications.md) will deploy the application.


