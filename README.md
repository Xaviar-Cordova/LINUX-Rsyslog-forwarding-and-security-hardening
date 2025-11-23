# LINUX-Ryslog-forwarding-and-security-hardening

Forwarding Cisco's equipments events and backups to Rsyslog server
![2024-10-07 15_58_38-RHEL Project Demo - GNS3](https://github.com/user-attachments/assets/6295cdff-010b-4a67-b100-80523355f28d)

In this example, we emphasize security hardening for RHEL-based servers by creating a scenario where a Linux server acts as the central destination for network equipment to store their running configurations and forward their event logs. We use GNS3 as the lab platform, with a Windows AD server, a switch, and a router to demonstrate proof of concept, and CentOS Stream 9 as the Linux operating system.

We'll use nmtui to set a static IP address of 10.0.1.250, rename the host to LXSR-01, and as our Active directory domain is xyz.com will point DNS settings to it. After the AD trust is established, we update visudo to define admin rights for an AD security group called sudoers. We also create a regular AD group called linuxuser to provide non-privileged access to the server. To log security-related events such as logins, we install and configure auditd. Services like these should start automatically at boot, so to enable that we use systemctl enable --now so they persist across reboots. We also create a local account named localadmin and add it to the wheel group, then reserve the root account for emergency use only.

The link is a Live 5-minute video [demonstration](https://github.com/user-attachments/assets/bdf604b2-da3b-4386-85f0-6ac569808f42)

![Screenshot 2024-11-25 155427](https://github.com/user-attachments/assets/d01b3810-b4b8-46de-99f6-7f5796337db1)

After modifying the rsyslog.conf file to listen on UDP port 514, we'll whitelist the port in firewalld. Because this is not a public-facing server and would usually reside on a dedicated VLAN in production, we set the default firewall zone to internal. Although UDP 514 is the default syslog port, we also verified the SELinux port context with semanage port to ensure the service is correctly associated. We then confirm connectivity using ss -lunp and tcpdump -i xyz udp port 514.
![2024-10-07 16_04_19-QEMU (LXSR-0-1) - TightVNC Viewer](https://github.com/user-attachments/assets/35d206ab-1b63-465c-bf24-aaa5c8e16c0d)

Remote access to this server will be useful as it is not always feasible or even secure to have physical access to it considering by default rd.break can allow access if no grub password is set (as this is a virtual platform, this portion can't be carried out). So Secure shell will prove vital to access it, but it too can benefit from enhanced security. we'll navigate to the sshd.conf file and modify idle time to close in 5 minutes, set the port to 2024, set maxauthtries to 3 to get logged quicker, and allowgroup to be sudoers and linuxusers.To acknowledge as password authentication is still enabled, ssh-keygen is considered best practice however we'll need to find a solution that integrate with AD to allow users keys to attach to their profiles then to the computer themselves so passwords will be in used in this demonstration. we'll also need to whitelist the new port and modify the security context for 2024 to be associated to SSH.

![Fire-wall](https://github.com/user-attachments/assets/d1818e5a-6ee5-4481-99d6-52e1cc953d54)


For the files we intends to be stored here for network configurations, We'll be creating a public directory that hosts each branch's Topology layout and equipment running-configs. this may be one of the few areas were execution right may be needed, but we do not wish for anyone to just have it. To ensure everyone can at least see directory files but can't execute we'll set umask o=rX. Anticipating future sub-directories will be created, Setfacl will be used to set the default rights to be assigned based on usergroups. One in particular that hosting the configurations, we'll remove any rights for linuxuser and only sudoers will have access.

![ACL](https://github.com/user-attachments/assets/6fd07eb4-74df-4ae2-8265-229908398b70)


To finalize our project on security and hardening, we'll sync our NTP to AD and disable unnecessary services. As mentioned the primary purpose of this server is to be a rsyslog server hosting network information, services like NFS or SAMBA should be disabled and blocked in our firewall, with semanage booleans checked if any additional features were turned on to accommodate them. It is also a good practice to modify over /etc/security/limit.conf to define the parameter in case DDOS does target it in later attacks.

![NTP](https://github.com/user-attachments/assets/facdb6e1-9f70-4ad0-8ca4-4abfd93255ae)
