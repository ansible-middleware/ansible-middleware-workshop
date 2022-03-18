# 5 - Configuring JBoss EAP
The JBoss EAP instances will need to connect to a shared postgresql instance to persist data.  JBoss EAP will need to be configured with the Postgresql driver and datasource.

To configure JBoss EAP we're going to download and install postgresql jdbc drivers.  The ansible-middleware collection provides a mechanism to do this, using the jdbc_driver.yml tasks.  We'll add these tasks to the jboss.yml by adding the following to the tasks section after the "configure firewall for Wildfly ports" task:


```
    - name: "Set up for JBoss module"
      include_role:
        name: wildfly_driver
        tasks_from: jdbc_driver.yml

```

To use these tasks we need to add the required environment variables to identify and download the jdbc drivers.


```
    jdbc_driver_module_dir: "{{ wildfly_home }}/modules/org/postgresql/main"
    jdbc_driver_version: 9.4.1212
    jdbc_driver_jar_filename: "postgresql-{{ jdbc_driver_version }}.jar"
    jdbc_driver_jar_url: "https://repo.maven.apache.org/maven2/org/postgresql/postgresql/{{ jdbc_driver_version }}/{{ jdbc_driver_jar_filename }}"
    jdbc_driver_jar_installation_path: "{{ jdbc_driver_module_dir }}/{{ jdbc_driver_jar_filename }}"
    jdbc_driver_module_name: 'org.postgresql'
```

Add the above environment variables to the jboss.yml file to the end of the vars section

## Configuring with Jcliff

Next we'll add in the jcliff collection and configure JBoss EAP to use the Postgresql database.

1 - Add the jcliff collection to the jboss.yml file

Under the collections section add the following:

`    - middleware_automation.jcliff`

2 - Add the jcliff tasks

In the tasks section add the following under the "Set up for JBoss module" task:

```
    - name: JCliff
      include_role:
        name: jcliff

    - name: "Fine tuning configuration"
      jcliff:
        wfly_home: "{{ wildfly_home }}"
        timeout: 7000
        management_port: "{{ item }}"
        components:
        - drivers:
          - driver_name: postgresql
            driver_module_name: org.postgresql
            driver_class_name: org.postgresql.Driver
            driver_xa_datasource_class_name: org.postgresql.xa.PGXADataSource
        - datasources:
            - name: "{{ instance_name }}db"
              use_java_context: 'true'
              jndi_name: "java:jboss/datasources/{{ postgres.jndi_ds }}"
              connection_url: "jdbc:postgresql://{{ (groups['pgsql'][0] if groups['pgsql'] | length > 0 else 'localhost') }}:5432/{{ postgres.db_name }}"
              driver_name: postgresql
              user_name: "{{ postgres.user.name }}"
              password: "{{ postgres.user.password }}"

      loop: "{{ instance_management_ports }}"

```

If you're familiar with JBoss EAP configuration using the cli, you'll recognise these configurations.  JCliff is being used to add a postgresql driver and datasource.  

In this instance we're passing the correct management port to JCliff, then using JCliff to add a postresql driver and datasource.  We're also using the loop task to iterate over the management ports, in our case we only have one instance per node, so a single management port.

Re-run the playbook to see the changes.

## Testing the JBoss EAP Configuration

To test the configuration we just performed with JCliff we'll first of all ssh into one of the JBoss EAP instances.  To do this, run the following:

`ssh xxx@<ip-address-of-node>`

Once you're logged in, change user to the jboss user with the following command:

`sudo su jboss`

Then run the following commands to use the JBoss cli:

`cd /opt/jboss-eap-7.4/bin`

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

This confirms the postgresql driver is installed.

Next [Step 6](./6-deploying-applications.md) will deploy the application.


