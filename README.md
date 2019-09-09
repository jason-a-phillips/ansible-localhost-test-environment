# Build a Local Ansible Test Environment with Ansible and Manged Hosts

Ansible is an automation and deployment tool used for pushing state to target managed host environments. What do we need to put Ansible through its paces? Some managed hosts, of course. You could provision some AWS or Azure infrastructure, but why not just spin them up locally with Docker? 

What's that you're saying? 

> DOCKER?? But you shouldn't SSH into a container!? You're doing Docker wrong!

Right, I get that. But in this case I want to quickly spin up managed hosts on my local machine into which I can SSH and run Ansible playbooks. Docker is suitable here for creating those managed hosts with very little fuss.

So let's get started. My desktop environment is Ubuntu 18.04 with Docker CE and Ansible (installed according to [these instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-apt-ubuntu)). I chose not to install Ansible using pip/pip3 as it does not create the artifacts located in `/etc/ansible`, which we will use.

Create an Ansible user on your control node (your desktop). Add ansible-user to the Docker group.

```bash
sudo useradd -m -s /bin/bash ansible-user
sudo usermod -aG docker ansible-user
sudo passwd ansible-user
```

Become your Ansible user and create an SSH key

```bash
su - ansible-user
ssh-keygen
```

Drop out of impersonating the Ansible user and return to your normal user with sudo privileges.

Copy ansible-user's PUBLIC key into a file called authorized_keys under the `docker/` folder in this project. The copied text will look like this:

```bash
ssh-rsa LOTSOFCHARACTERSOFKEYTEXTHERE ansible-user@mycomputername
```

Modify the Ansible hosts file located at `/etc/ansible/hosts` to create a group in the Ansible inventory for our managed hosts' Docker containers. We can reference them by this group name in the future.

```bash
sudo vi /etc/ansible/hosts
```

Add these lines:

```
[managed-hosts]
172.17.0.2
172.17.0.3
172.17.0.4
172.17.0.5
```

The four Docker containers will have IP addresses which we will need to know from the perspective of your desktop's network stack (the Ansible control node). Docker on Linux normally begins assigning IP addresses in the sequence above. However, if this is not the case, you must inspect the running containers using:

```bash
docker inspect <container ID or name>
```

...to find the assigned IP address that is exposed to the Docker host's networking stack. Modify the `/etc/ansible/hosts` file accordingly so that you can reference our running containers by group name in inventory.

Next we are ready to build the image for the manged host containers. In the `playbooks/` folder, run:

```bash
ansible-playbook build-host.yml
```

This will build the managed hosts' Docker containers. Alternatively, you could build the image with the Docker command in the `docker/` folder:

```bash
docker build -t ansible-host .
```

For the base Docker image, I chose Ubuntu 18.04 because it is easy to install the required ssh server. It will also be easy enough to install any services that we'd like to fiddle with later in our Ansible adventures. In the Dockerfile, I added our authorized key for SSH, provisioned the same ansible-user as was provisioned on our Ansible control node and have also turned off ssh password login.

Next, we are ready to create our array of managed hosts. This playbook not only runs the managed hosts' Docker containers, but also cleans the known_hosts file and adds the keys from our new containers into ansible-user's local known_hosts file. We need to become the ansible-user (1) so that the SSH key that we created in the Ansible control node is used by Ansible against the managed hosts and (2) the managed hosts' keys are added to the ansible-user's known_hosts file. The ssh-keyscan commands take forever but it works. Become the ansible-user and run the ansible-playbook command in the `playbooks/` folder:

```bash
su - ansible-user
ansible-playbook run-hosts.yml
```

Four managed host containers should have been started against which Ansible can run playbooks. You can see them by running:

```bash
docker ps
```

You can ping the managed hosts by running the Ansible ping module against the `managed-hosts` group that we configured in the hosts file inventory. 

```bash
ansible managed-hosts -m ping
```

To stop and remove the managed hosts' containers, in the `playbooks/` folder, run:

```bash
ansible-playbook rm-hosts.yml
```

There you have it! A fairly quick and painless process for provisioning a test environment with multiple Ansible managed hosts right alongside Ansible, all running on the same machine. Spin it up quickly and painlessly without provisioning unnecessary infrastrcture on AWS, Azure, etc. Easy peasy lemon squeezy.
