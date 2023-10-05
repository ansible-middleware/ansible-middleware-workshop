# 4 - Deploying JBoss Core Services

Before we install JBoss EAP and our application, we need to make sure we can connect to our instances via the frontend gateway.  Our JBoss servers are do not have a public IP or DNS so we'll need to install a load balancer to provide this service. JBoss Core Services (JBCS) will provide this functionality using Apache HTTP server.

To install JBCS we need to ensure the mailcap package is installed otherwise /etc/mime.types will not be found.  We're going to create a separate playbook called pre-reqs to do this. Create a file pre-reqs.yml

``` yaml
---
- name: Playbook for installing prerequisites
  hosts: jbcs
  tasks:
    - name: "Install packages"
      ansible.builtin.include_role:
        name: fastpackages
      vars:
        packages_list:
          - mailcap

```

And run this with:

`ansible-playbook -i ./inventory/hosts pre-reqs.yml `

# Install valid ssl cert

We need to ensure ssl is enabled on our frontend gateway, so we're going to create an SSL cert before we install JBCS.  To do this we'll use Let's Encrypt via certbot.

Create a file called ssl.yml and past the following:


``` yaml
---
- name: Request Let's Encrypt Static Certificates
  hosts: jbcs
  become: true
  gather_facts: False
  tasks:

  - name: Install on node RHEL 8
    block:
      - name: Enable EPEL
        dnf:
          name: "https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm"
          state: present
          disable_gpg_check: true

      - name: install nginx/certbot
        yum:
          name:
            - certbot
            - python3-certbot-apache
          state: present


  - name: Generate certbot
    command: >-
      certbot certonly --standalone
      -m dummy@redhat.com
      --agree-tos
      -d {{ jbcs_external_domain_name }} -n


  - name: Remove the Apache package
    ansible.builtin.yum:
      name: httpd
      state: absent

  - name: Copy letsecrypt key
    copy:
      src: "/etc/letsencrypt/live/{{ jbcs_external_domain_name }}/cert.pem"
      dest: /etc/pki/tls/certs/{{ jbcs_external_domain_name }}.crt
      remote_src: yes
      mode: 0644

  - name: Copy letsecrypt certificate
    copy:
      src: "/etc/letsencrypt/live/{{ jbcs_external_domain_name }}/privkey.pem"
      dest: /etc/pki/tls/private/{{ jbcs_external_domain_name }}.key
      remote_src: yes

      mode: 0644

```

This playbook will install the certbot package and use this to create a certificate and key for the domain name of the frontend gateway.  When this playbook is run we need to provide the frontend domain name with the variable jbcs_external_domain_name.  This domain name can be found in the instructions email. NOTE: Do not add the "https://" part, just the hostname

Run this playbook with the following command e.g. ansible-playbook -i ./inventory/hosts ssl.yml  --extra-vars "jbcs_external_domain_name=frontend-domain-name"


`ansible-playbook -i ./inventory/hosts ssl.yml  --extra-vars "jbcs_external_domain_name=<your frontend domain name>"`

We'll now create a playbook to install JBCS.  Create a file called jbcs.yml in the top level folder.  Copy the following snippet to the top of the file:

``` yaml
---
- name: Playbook for loadbalancer Hosts
  hosts: jbcs
  become: true
  vars:
    jbcs_ssl_enable: true
    jbcs_zip_path: /home/devops/workshop
    jbcs_bind_address: 0.0.0.0
    jbcs_offline_install: false
    jbcs_mod_cluster_enable: true
  collections:
    - redhat.jbcs
  roles:
    - jbcs
  pre_tasks:
    - name: Set httpd_can_network_connect flag on and keep it persistent across reboots
      ansible.posix.seboolean:
        name: httpd_can_network_connect
        state: yes
        persistent: yes
    - name: Set nid_enabled flag on and keep it persistent across reboots
      ansible.posix.seboolean:
        name: nis_enabled
        state: yes
        persistent: yes
    - name: Add reverse proxied port to selinux
      ansible.builtin.command: semanage port -m -t http_port_t -p tcp 8080
      changed_when: False
    - name: Add reverse proxied port to selinux
      ansible.builtin.command: semanage port -m -t http_port_t -p tcp 8009
      changed_when: False
  tasks:
    - name: install firewalld
      become: true
      action: yum name=firewalld state=installed
    - name: Start firewalld
      become: true
      service: name=firewalld state=started enabled=yes

    - name: configure firewall for JBCS ports
      become: yes
      firewalld:
        port: "{{ port }}"
        permanent: true
        state: enabled
        immediate: yes
      loop:
        - "80/tcp"
        - "6666/tcp"
        - "443/tcp"  
      loop_control:
        loop_var: port
```

Note that the Ansible collection for JBCS will also take care of downloading the required assets from the Red Hat Customer Portal (the archive containing the Java app server files). However, one does need to provide the credentials associated with a service account. You can manage service accounts using the hybrid cloud console. Within this portal, on the [service accounts tab](https://console.redhat.com/application-services/service-accounts), you can create a new service account if one does not already exist.

Save this file, and test the playbook by running the following command:

`ansible-playbook -i ./inventory/hosts jbcs.yml  --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password> jbcs_external_domain_name=<your frontend domain>"` 

Replace your frontend domain with the url of the frontend listed in the instructions email.

# Testing the JBCS installation

To test the JBoss Core Services are installed correctly, use your browser and navigate to the external hostname of your JBoss Core Server.

e.g. `https://frontend1.xxxxx.domainname.com`



You should see the default apache landing page.

![default apache landing page](../images/apache.png)


Next [Step 5](./5-deploying-jboss-eap.md)