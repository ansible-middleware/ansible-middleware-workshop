# 6 - Deploying an application

Now that JBoss EAP is deployed and configured, we can deploy our application.  To do this we're going to provide a pre-built war file.  For the purpose of this workshop we've already prepared a war file and made it available in S3.  This war file will use the postgresql datasource to persist addressbook records.

First thing we'll do is add details of the app to the vars section of the jboss.yml file.  We'll need to provide the following details:

``` yaml
    app:
      name: 'addressbook.war'
      url: 'https://gpte-public.s3.amazonaws.com/addressbook.war'
```

Next we'll add the following to the tasks section of the jboss.yml file to download the application:

``` yaml
    - name: "Ensures webapp {{ app.name }} has been retrieved from {{ app.url }}"
      get_url:
        url: "{{ app.url }}"
        dest: "/opt/jboss_eap/jboss-eap-7.4/{{ app.name }}"
```

Finally we'll add the following to the end of the tasks section of the jboss.yml file to deploy the application:

``` yaml
    - name: "Deploy webapp"
      include_role:
        name: eap_utils
        tasks_from: jboss_cli.yml
      vars:
        jboss_cli_query: "'deploy --force /opt/jboss_eap/jboss-eap-7.4/{{ app.name }}'"
```

Save changes to jboss.yml and re-run the playbook.

Once the playbook is complete, you should be able to access the application.

# Testing the application

To test the install, open url https://xxxxx/addressbook (replace xxxxx with the frontend url listed in your email) 


Now that the application is deployed, let's move on to the next section, testing.

Next [Step 8](./8-testing.md)