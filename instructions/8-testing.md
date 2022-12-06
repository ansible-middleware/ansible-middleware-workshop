# 8 - Testing

To test our deployment we'll add a validate section to our jboss.yml.  This step will attempt to connect to the JBoss EAP instance and application.  

First thing we'll do is add a post_tasks section at the end of jboss.yml.

```
  post_tasks:
    - include_tasks: validate.yml
      loop: "{{ instance_http_ports }}"
```

This task requires a file validate.yml, so let's create this file in the same level as jboss.yml. Paste the following into validate.yml:

```
---
- assert:
    that:
      - item is defined

- wait_for:
    port: "{{ item }}"

- name: "Checks that Wildfly server is running and accessible on port {{ item }}"
  get_url:
    url: "http://localhost:{{ item }}/"
    dest: '/dev/null'
  changed_when: False

- include_tasks: app-check.yml
```

This file will validate the JBoss EAP instance is available on the correct port (the instance_http_port).

To check the addressbook application is running, we'll add the following to a file called app-check.yml.

```
---
- assert:
    that:
      - item is defined
    quiet: true

- set_fact:
    result_file: "/tmp/app-check-{{ item }}.txt"

- get_url:
    url: "http://localhost:{{ item }}/addressbook/"
    dest: "{{ result_file }}"
  changed_when: False

- slurp:
    src: "{{ result_file }}"
  register: appcheck_res

- debug:
    msg: "{{ appcheck_res['content'] | b64decode }}"
```

Save changes to jboss.yml and re-run the jboss playbook.

`ansible-playbook -i ./inventory/hosts jboss.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>"`

The Ansible playbook should output the http response from the addressbook application, the script will then complete successfully.

Now that we have our test in place, all that's left is to combine these playbooks into a single playbook.

Next [Step 9](./9-combining.md)

