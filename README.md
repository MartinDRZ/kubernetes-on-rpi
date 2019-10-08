# kubernetes-on-rpi
Kubernetes on Raspberry Pi


A simple setup guide for installing an N-host Kubernetes cluster on Raspberry Pi. For my cluster, I am using 1x Raspberry Pi 4 (4Gb) as Master and 2x Raspberry Pi 4 (1Gb) & 2x Raspberry Pi 3 B+ as nodes.

All configurations are carried out in Ansible except for the initial setup which is done using cloud-init as part of the HypriotOS base install.

Section 1 - Bare Installation:

1. Set up your RPis - get them power and networking. For the Pi4, wired vs wireless doesn't really matter but the wired NIC on the Pi3 has shared throughput with the USB bus, whereas the wlan NIC does not. There are arguments for both.

2. Install HypriotOS from here: https://blog.hypriot.com/downloads/ - there are other OSes out there but I'm giving this one a go as it has been built specifically for containerisation. I successfully built a Docker Swarm cluster using Raspbian so you may be able to adapt this to suit that OS if you prefer.

3. Flash the OS onto your card. I used Etcher to flash the MicroSD cards, but use whatever suits you.

4. Once flashed, open up user-data and edit the hostname, time zone and region settings. This file is in YAML format which means spaces, tabs and line breaks are particularly important. Try not to break the syntax - I recommend you run it through a linter first.

5. On first boot, your Pis will go through the boot sequence (and update, if you told it to in the user-data file) which might take a few minutes. Once done, fish the IPs out of your DNS/DHCP server and you should be able to SSH to each with the default credentials (see the Hypriot FAQs)

Once you have confirmed that SSH access is working, it is time to move on to Section 2...

Section 2 - Prepare the OS / Install all deps:

