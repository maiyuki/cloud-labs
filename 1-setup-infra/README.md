# Setup infrasturcture

## Assignment Part 1 - Networking

Manual setup infrastructure which includes networking, web servers, load balancing, and other necessary components.

For every configuration you do in Openstack there is a description that will explain the configuration. Read them to get a better understanding.

### Create a Key pair

Create a key pair: `<name-key>`

Your key will automatically be downloaded. You will need the key later.

### Setup Network and Subnet

Setup a Network with a Subnet with the following configurations:

- Network
    - name: `<name>_network`
- Subnet
    - name: `<name>_subnet`
    - range: `10.0.x.0/24`
    - dns: `8.8.8.8` and `8.8.4.4`

### Create a router

Create a router and add the ID to the external network (it should already exist).

### Create an Interface 

Create an Interface on the router and point it to your Subnet.

### Security Groups

Create a Security Group with Rules for your Bastion host.

- Name: `<name>-bastion-sg`
- Create rule: Allow SSH from the Internet and a description

### Create a Bastion host

A Bastion host / Proxy server and allows the client machines to connect to the remote server.

- Instance
    - Name: `<name>-bastion-<az>` // ex. mai-bastion-sto1
    - AZ: Choose any zone
    - Boot Source: `Image`
    - New Volume: `No`
    - Choose an Image (You will need to find what the default username is for the image you choose.)
    - Flavor: `v1-micro-1`
    - Network: The Network you created in the earlier steps
    - Security Groups: The Security Group you created in the earlier steps
    - Key Pair: The key pair you created in the earlier steps
    - User Data: 
        ```
        #cloud-config
        final_message: "The system is finally up, after $UPTIME seconds"
        ```
    - Metadata: Add a Custom one. Ex: "Image used" : "ubuntu-20.04-server-latest"


### Floating IP

To be able to reach the Bastion host, we need a Floating IP.

- Allocate a Floating IP from a given floating IP pool
- Associate the IP with your Subnet's port

### View the Network Topology

You should be able to see your created resources in the topology. Have a look around and explain "to a rubber duck" how traffic flows to your instance.

### View Instance Log

Head to Instance and View Log on your instance. You will see the message from User Data.

### SSH into Bastion host

- Change permissions on your key on your local host to rw for user
- `ssh <username>@<floating-ip> -i <key>`

Pro tip!
- Add you key to SSH authentication agent `ssh-add <key>`
- `ssh <username>@<floating-ip>`

## Assignment Part 2 -  Web server and Load balancing

## Create a Server Group

- Name: `webservers-group`
- Policy: `soft-anti-affinity`

### Security Groups

Create a Security Group with Rules for your Web Servers.

- Name: `<name>-web-server-sg`
- Create rule: Allow SSH from the Bastion Security Group and a description
- Create rule: Allow HTTP from the Internet and a description

### Create a Web Server

Create a new Instance and make sure you can SSH to it only from Bastion host.

- Instance:
    - Name: `<name>-server-<az>-<number>` // ex. mai-server-sto1-1
    - AZ: Choose any zone
    - Boot Source: `Image`
    - New Volume: `No`
    - Choose an Image (You will need to find what the default username is for the image you choose.)
    - Flavor: `v1-micro-1`
    - Network: The Network you created in the earlier steps
    - Security Groups: The Security Group you created for the web server
    - Key Pair: The key pair you created in the earlier steps
    - User Data: 
        ```
        #cloud-config
        package_update: true
        package_upgrade: true
        packages:
        - python3-minimal
        - apache2
        - jq
        runcmd:
        - curl -s http://169.254.169.254/latest/meta-data/hostname >/var/www/html/index.html
        - echo >>/var/www/html/index.html 
        - curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone >>/var/www/html/index.html
        - echo >>/var/www/html/index.html 
        - echo >>/var/www/html/index.html
        - echo "OK" >/var/www/html/health.html
        final_message: "ELX - The system is finally up, after $UPTIME seconds"
        ```
    - Server Groups: The one you created eariler
    - Schedular Hints: ex. `group = <server group's ID>`
    - Metadata: Add a Custom one. Ex: "group" : "web"

Repeat this step for as many instances as you need.

### Create a Floating IP

Create a Floating IP.

### Create a Load balancer

Create a Load balancer to load the load between the web servers.

- Load balancer:
    - Load balancer Details:
        - Subnet: `<name>_subnet`
    - Listener Details:
        - Protocol: `HTTP`
        - Port: `80`
        - Tick `X-Forwarded-For`
    - Pool Details:
        - X-Forwarded-For: `ROUND_ROBIN`
    - Pool members:
        - Allocated Members: 
            - Add all the web servers you created in the previous step with port `80`
        - Type: `HTTP`
    - Monitor Details:
        - URL Path: `/health.html` (we have added a health.html file in the web server)

Leave the rest of the values as defaults.

### Associate Floating IP to Load balancer

Associate Floating IP to Load balancer.

### Add Security group to Load balancer

Go to Ports in Subnet and add th.e same security group for the servers to the Load balancer (Octavia)

### Verify the setup

- SSH to the Bastion host
- SSH to the Web servers
- Copy and past Floating IP for the Load balancer in the web browser and refresh - you should see the content switching
- Visit the health endpoint

## Challenge!

- Add to your SSH configuration to jump directly to the Server via Bastion instead of first SSH into Bastion and from Bastion SSH to Server.
