# kubernetes-on-rpi
Kubernetes on Raspberry Pi


A simple setup guide for installing an N-host Kubernetes cluster on Raspberry Pi. For my cluster, I am using 1x Raspberry Pi 4 (4Gb) as Master and 2x Raspberry Pi 4 (1Gb) & 2x Raspberry Pi 3 B+ as nodes. I have NFS exports on my Synology NAS for shared storage, but this isn't essential - you can use the local SD card storage of your Pi if you want to.

All configurations are carried out in Ansible except for the initial setup which is done using cloud-init as part of the HypriotOS base install.

Section 1 - Bare Installation:

1. Set up your RPis - get them power and networking. For the Pi4, wired vs wireless doesn't really matter but the wired NIC on the Pi3 has shared throughput with the USB bus, whereas the wlan NIC does not. There are arguments for both.

2. Install HypriotOS from here: https://blog.hypriot.com/downloads/ - there are other OSes out there but I'm giving this one a go as it has been built specifically for containerisation. I successfully built a Docker Swarm cluster using Raspbian so you may be able to adapt this to suit that OS if you prefer.

3. Flash the OS onto your card. I used Etcher to flash the MicroSD cards, but use whatever suits you.

4. Once flashed, open up user-data and edit the hostname, time zone and region settings. This file is used by cloud-init and is in YAML format which means spaces, tabs and line breaks are particularly important. Try not to break the syntax - I recommend you run it through a linter first.

5. On first boot, your Pis will go through the boot sequence (and update, if you told it to in the user-data file) which might take a few minutes. Once done, fish the IPs out of your DNS/DHCP server and you should be able to SSH to each with the default credentials (see the Hypriot FAQs)

Once you have confirmed that SSH access is working, it is time to move on to Section 2...

Section 2 - Prepare the OS / Install all deps:

1. Ansible Inventory

Now that we have a working OS installed on the Raspberry Pis, it is time to switch to Ansible for automating all of this stuff. The first thing to do is build a hosts file. This file should be in 'inventories/' somewhere, in my case I have separated production and development out (which is good practice) so hy hosts file is in 'inventores/prod/hosts'. Check out my example file which should give you an idea how to lay your file out. As with all YAML, spaces matter so please take care when creating your own.

2. Ansible Roles

There are many ways to do things in Ansible but I find breaking things down into distinct roles to be the best approach. I have created a role called rpi-base-install which will do all of the basic tasks common to all of my rpis regardless of if they are running as part of my Kubernetes cluster or not. This way, I can pick up the role and re-use it later on without having to change too much.

The base installations carried out by rpi-base-install are straightforward. The tasks make sure the apt cache is up to date, installs the correct version of Docker and fixes some little issues I had with HypriotOS's baked-in Docker install. I also install NFS which will allow me some flexibility with my storage choices later on.

I also have chosen to use Ansible to automate the naming of the hosts and the setup of /etc/hosts on each of the Pis.


Section 3 - Install Kubernetes

This is broken out into two parts - one role for the Kubernetes Master and another for the regular Nodes.

How Kubernetes works is beyond the scope of this, but basically these roles install the components required to get Kubernetes up and running, installs Flannel as the CNI (networking plugin) and then joins the worker nodes to the cluster.

Once you've got this far you need to decide for yourself how you'd like to continue - you'll need to set up an Ingress Controller (eg Traefik) and get some persistent storage working.
