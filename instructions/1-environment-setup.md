# 1 - Prerequisites

You will need an Red Hat network account to be able to download the JBoss EAP installation zip files.  The account you use must not be managed by SSO in order for the Ansible scripts to be able to authenticate.  You can create a new account by following the instructions in the [Red Hat Customer Portal](https://sso.redhat.com/auth/realms/redhat-external/login-actions/registration?client_id=customer-portal&tab_id=RiPOv96eZ74).  You will need to use these credentials when you run the ansible playbooks later in the workshop.

## Set up the environment

This workshop requires the provisioning of the following RHPDS environment.  Once this environment is provisioned your will have the following ... TODO

## Accessing the repository

If you are using the provided RHPDS environment, the code repository should already be cloned and availalbe in the `/tmp/ansible-middleware-workshop` directory.  If you are using a different environment, you will need to clone the repository yourself.

The code for this workshop is hosted on GitHub.  You can clone the repository by running the following command: 

`git clone https://github.com/deewhyweb/wildfly-cluster-demo.git && cd wildfly-cluster-demo && git checkout stage1`

## Check ansible hosts
Login to the Ansible Automation controller via ssh.

Open the file `inventory/hosts`, you should see the ip addresses of your RHEL nodes listed as shown below.

```
# Placeholder Group
[demo]

# jboss Group
[jboss]
192.168.122.64 
192.168.122.224 

# postgres database Group
[pgsql]
192.168.122.20

# jcbs Group
[jbcs]
192.168.122.247

[demo:children]
jboss
pgsql
jbcs

```

## Test ansible hosts
Once we have checked the hosts file we can test the ansible hosts file by running the following command.

`ansible -i inventory/hosts demo -m ping`

You should see the following output:

```
192.168.122.64 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.224 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.247 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.122.20 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

We've checked our environment, now we can continue with the workshop and add the required ansible collections.

Next [Step 2](./2-adding-collections.md)