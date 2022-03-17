# 6 - Deploying an application

Now that JBoss EAP is deployed and configured, we can deploy our application.  To do this we'll need to provide a pre-built war file.  For the purpose of this workshop we've already prepared one and made it available in Google drive.  

First thing we'll do is add details of the app to the vars section of the jboss.yml file.  We'll need to provide the following details:

```
    app:
      name: 'addressbook.war'
      url: 'https://drive.google.com/uc?export=download&id=1HOfRNHFxuuQuCEtMFt1jhvlclIpz888D'
```

Next we'll add the following to the tasks section of the jboss.yml file to download the application:

```
    - name: "Ensures webapp {{ app.name }} has been retrieved from {{ app.url }}"
      get_url:
        url: "{{ app.url }}"
        dest: "{{ wildfly_install_workdir }}/{{ app.name }}"
```

Finally we'll add the following to the tasks section of the jboss.yml file to deploy the application:

```
    - name: "Deploy webapp"
      include_role:
        name: jboss_eap
        tasks_from: jboss_cli.yml
      vars:
        jboss_home: "{{ wildfly_home }}"
        query: "'deploy --force {{ wildfly_install_workdir }}/{{ app.name }}'"
        jboss_cli_controller_port: "{{ item }}"
      loop: "{{ instance_management_ports }}"
```

Save changes to jboss.yml and re-run the playbook.

Once the playbook is complete, you should be able to access the application.

# Testing the application

To test the application, open your browser and navigate to the ip address of one of your JBoss EAP nodes.  e.g.

`http://<ip address>:8080/addressbook/`

You should see the following screen:

 ![default apache landing page](../images/addressbook.png)

Next [Step 7](./7-deploying-jbcs.md)