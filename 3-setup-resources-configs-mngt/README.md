# Setup infrasturcture

## Assignment Part 1 - Install and configure Ansible to reach web hosts

- Install `ansible`

- Add your web hosts to `/etc/ansible/hosts` (default ansible inventory). If directory does not exist, create it.

Ex.
```
[webservers]
mai-server-2
mai-server-1
```

- Add default configuration for the web hosts to `/etc/ansible/ansible.cfg`

Ex.
```
[webservers]
inventory = /etc/ansible/hosts
ansible_ssh_private_key_file = ~/.ssh/mai-key.pem
```

- Add web hosts SSH configurations to `~/.ssh/config`

Ex.
```
Host mai-bastion
    User ubuntu
    HostName <public-ip>
    IdentityFile ~/.ssh/mai-key.pem

Host mai-server-1
    User ubuntu
    HostName <private-ip>
    ProxyCommand ssh -q -W %h:%p mai-bastion
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/mai-key.pem

Host mai-server-2
    User ubuntu
    HostName <private-ip>
    ProxyCommand ssh -q -W %h:%p mai-bastion
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/mai-key.pem
```

- Verify that you can reach your web hosts with ansible by pinging them

```
ansible webservers -m ping
```

Output Ex.
```
Output:

mai-server-2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
mai-server-1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## Assignment Part 2 - Ansible

- Remove the web servers on the hosts or disable them. Docker won't be able to use port 80 is the servers are active.

- Setup an Ansible playbook to install neccessary packages for running docker containers.

- Run a web server in a docker container in each host

- Remember to add the health endpoint, similar to the one in lab 2. Your Load balancer will else fail to serve traffic.

**Links:**

- [How to build you inventroy](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) 

- [Use Ansible to Install and Set Up Docker](https://www.digitalocean.com/community/tutorials/how-to-use-ansible-to-install-and-set-up-docker-on-ubuntu-20-04) / [Apache Web Server in a Docker Container](https://www.ansiblepilot.com/articles/deploy-apache-web-server-in-a-docker-container-for-debian-like-systems-ansible-modules-docker_image-and-docker_container/) 

### Example of a final playbook

See file `playbook.yml` for an example of one way of setting up docker containers.

Example of commands to use:
```
# Use `--check` to dry run
ansible-playbook playbook.yml --check

# Use `–diff` to view config difference and list of actions to be taken
ansible-playbook playbook.yml --diff
```

## Assignment Part 3 - Docker

- SSH into your web servers and view your docker containers
```
docker ps -a
```

```
Output:

CONTAINER ID   IMAGE     COMMAND              CREATED         STATUS         PORTS                  NAMES
6c05ae5a8006   httpd     "httpd-foreground"   5 seconds ago   Up 4 seconds   0.0.0.0:8080->80/tcp   webserver1
```

- Check LISTEN ports on the web hosts 
```
lsof -i -P -n | grep LISTEN

Output:

sshd       5695            root    3u  IPv4  37582      0t0  TCP *:22 (LISTEN)
sshd       5695            root    4u  IPv6  37593      0t0  TCP *:22 (LISTEN)
systemd-r  5965 systemd-resolve   13u  IPv4  38845      0t0  TCP 127.0.0.53:53 (LISTEN)
docker-pr 58498            root    4u  IPv4 287005      0t0  TCP *:80 (LISTEN)
```

- Verify in the web browser that you deploy succeeded.

## Challenge!

First:

Create multiple web servers on one host

Second:

Integrate Object Store into you Web Application

Create an Object store, upload two or more pictures to the container and use python to fetch the pictures and present it in you web application. 

