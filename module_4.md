# Linux Server Setup - Basic Configurations

After installing a Linux operating system, several initial configuration steps need to be completed before using the system as a server. These are setup tasks that every Linux administrator performs, regardless of whether the server runs on physical hardware, a virtual machine, or in a cloud environment.  

## Web server - Building Blocks
When you deploy a web server to the public internet, several essential components must be in place and correctly configured. Together, these building blocks ensure that your server is reachable, secure, and able to serve content reliably.  

1. A (cloud) server with basic configuration  
2. Domain name and DNS records that point to server's public IP address  
3. TLS certificate for HTTPS  

Different types of cloud service providers offer Linux hosts as a service. Which cloud provider to select depends on the use case. Here are some aspects to be considered:  

| Feature   | Smaller Cloud Provider |  Hyperscaler |  
| -------- | ---------------------- | -------------|  
| Target audience  | startups, developers   | enterprises |  
| Ease of use        | simple UI, beginner friendly   | steeper learning curve, enterprise grade tools|  
|Pricing| transparent, low cost, flat monthly fee   | pay-as-you-go, saving plans    |  
| Services  | basic IaaS  | hundreds of IaaS/PaaS/SaaS services |  
| Use cases  | web apps, blogs, dev environments | scalable enterprise applications, global services|  


## 1. Update the System Software

After installing Debian (or any Linux distribution), the first thing to do is update the system to ensure that the latest security patches and bug fixes are installed. To install the latest updates:  
- ```sudo apt-get update``` updates the local package index (for example ```/etc/apt/sources.list```) by downloading the latest list of available packages and versions from the package repositories. However, it does not install anything yet.  
- ```sudo apt-get upgrade``` installs the newest versions of all installed packages based on the updated package index.

## 2. Create a non‑root user with sudo

Depending on your hosting provider, you may receive a virtual machine with root access, or you may initally get a non‑root user with sudo privileges. If you do have root access, the first recommended step is to create a separate administrative user. Working directly as root must be avoided because root has unrestricted power and even small mistakes can cause severe or permanent system damage. To create a new administrative user  

1) Create a new user:  ```sudo adduser linuxuser```  
2) Add the new user to the sudo group (grant administrative privileges): ```sudo usermod -aG sudo linuxuser```  
3) Allow non-root linuxuser to read system logs without using sudo: ```sudo usermod -aG adm linuxuser```  


Notice: The list of groups the user is in can be listed with command ```groups```  
```
linuxuser@test043:/etc/ssh$ groups  
linuxuser adm dialout cdrom floppy sudo audio dip video plugdev  
```

Notice2: The user must log out and log back in for the new group memberships to take effect.  





  
## 3. SSH Server 

Cloud servers always come with ssh enabled because ssh is the primary (and often the only) way to access them remotely. A self-installed Debian system does not include ssh server by default, therefore OpenSSH Server must be installed first to enable remote access:  
- Installation: ```sudo apt-get install openssh-server```  
- Status check: ```sudo systemctl status ssh```

Do not confuse the ssh client with the ssh server: The ssh client (the program you use to connect to other machines using ```ssh user@host```) is always installed on Debian by default:   
- SSH client → used to connect to other systems  
- SSH server → allows others to connect to your system

Whether your ssh server came pre‑installed by a cloud provider or you installed it yourself, some basic security hardening should be applied. Security Hardening means making a computer system more secure by reducing its weaknesses, limiting access, and configuring it to resist attacks. For ssh-server, hardening is done by adjusting settings in the ssh server configuration file:  ```/etc/ssh/sshd_config```  
 
Just like Apache, ssh server runs as a systemd deamon, which means that any changes you make to the configuration file require restarting the service: ```sudo systemctl restart ssh```    

A Daemon is a program that works quietly in the background and waits for something to do. It starts at boot, does not need a user logged in, and performs ongoing tasks for the system or applications. Daemons are controlled by Linux system and service manager, ```systemd``` using tools like ```systemctl```:  

```  
systemctl start <service>
systemctl restart <service>
systemctl stop <service>
systemctl status <service>
```   
### __Some essential SSH Security Settings:__   
In ```/etc/ssh/sshd_config``` file:  
- Disable password logins and force users to log in using ssh keys only: ```PasswordAuthentication no```   
- Disable root login over ssh and force admins to log in as a normal user: ```PermitRootLogin no```  

__Notice:__ Be careful that you do not lock yourself out when configuring a remote server. For example, before disabling password logins ensure that ssh keys are configured properly. And before disabling root login ensure that a non-root user is created and has access to the server.  
__Notice2:__ Be aware that some cloud environments override or bypass parts of the SSH daemon configuration through cloud-init or provider-specific mechanisms.  

## 4. Configuring ssh key authentication  
```ssh-keygen``` creates a public + private ssh key pair for the user. Key pair is used for passwordless and secure ssh login: private key signs a challenge from the server and the server uses public key to verify the signature.  

How to generate an ssh key pair on your __local machine__: 
```ssh-keygen``` asks three questions:  
- Enter file in which to save the key (/home/username/.ssh/id_rsa):  
- Enter passphrase (empty for no passphrase):   
- Enter same passphrase again (empty for no passphrase):
- Notice: if ssh-client is not installed you need to do that first: ```sudo apt-get install openssh-client```   
	
In test enviroments the default values (enter) can be used. In this case both keys are stored in ```~/.ssh``` directory:  
	- ~/.ssh/id_rsa        ← private key  
	- ~/.ssh/id_rsa.pub    ← public key	  
Without a passphrase ssh login works without typing any password. A passphrase protects private key with an extra encryption layer. It is asked every time the private key is used.  
 
After the key-pair is created the public key is copied to the __remote server__: 
- option 1: ```ssh-copy-id linuxuser@your_server_ip```  
- option 2: copy manually the contents of ```~/.ssh/id_rsa.pub``` file to the server’s ```~/.ssh/authorized_keys``` file.

Remember that your private key must never leave the machine where it was created. (The only common exception is when a cloud provider generates the key pair for you. In that case, the private key is displayed once and must be saved securely, because it cannot be retrieved again.)

Finally, test ssh login with keys: ```ssh linuxuser@<your_server_ip>``` and only after that disable password-based authentication.  

## 5. Install and Configure Firewall (ufw)
Linux servers should be protected by firewall. Uncomplicated firewall (ufw) is a firewall tool for linux, that controls which network connections are allowed into and out of your server. The basic principle is to block all incoming traffic except the traffic you explicitly allow.  

- ```sudo apt-get install ufw```
- ```sudo ufw allow ssh``` or ```sudo ufw allow 22/tcp``` (do not lock yourself out - open ssh port before you enable ufw)  
- ```sudo ufw enable```  
- ```sudo ufw status verbose```  

If the server runs a website:  
- ```sudo ufw allow http``` or ```sudo ufw allow 80/tcp```  
- ```sudo ufw allow https``` or ```sudo ufw allow 443/tcp```  

Usually, if Linux server is running in the cloud, the server is protected by two layers or firewall:  
- Cloud security groups (or network policies) that run outside the server at the cloud provider's boundary. They control which traffic can reach the server at all, and are configured in cloud provider's web console, not inside linux.
- ufw, that controls traffic inside Linux server. It controls traffic that has already passed through the cloud firewall (security group).

[Internet] → [CLOUD SECURITY GROUP] → [UFW on SERVER] → Linux services  

## 6. Automatic Security Updates
Unattended Upgrades automatically installs important security patches for installed packages without requiring any manual action from the administrator.

Automatic security updates are recommended when security is more important than absolute stability, especially for servers that are exposed to the Internet or systems where administrators may forget to apply updates regularly. However, automatic updates are not always appropriate, for example, on systems that require strict stability, predictable maintenance windows, or very high uptime where updates must be tested before deployment.  

To enable automatic security updates, run:  
- ```sudo apt-get install unattended-upgrades```  
- ```sudo dpkg-reconfigure unattended-upgrades```

The first command installs the tool and the second one activates automatic security patching.  

## 7. Networking

Connectivity issues are common. Linux offers tools needed to diagnose and resolve network problems effectively.  
__ip addr__ command can be used to check your network interfaces and IP addresses. 

```
linuxuser@linuxuser:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:85:ef:27 brd ff:ff:ff:ff:ff:ff
    altname enx08002785ef27
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 83038sec preferred_lft 83038sec
    inet6 fd17:625c:f037:2:ebf4:2e2:531e:28df/64 scope global dynamic noprefixroute 
       valid_lft 86291sec preferred_lft 14291sec
    inet6 fe80::b56e:cffc:b57d:50cd/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
Notice: Cloud platforms such as Azure do not expose the VM’s public IP inside the operating system. Running ip addr will only show the private IP because the public IP is applied at the Azure host/NIC level using NAT. To see the public IP from within the VM, use an external lookup: ```curl ifconfig.me```  



__ip route__ shows the routing table:  

```
linuxuser@linuxuser:~$ ip route
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
```


__ping__  
A successful ping means that packets can travel from your device to the target and back to your device. It proves that the basic network path is working.
```
linuxuser@linuxuser:~$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=1.10 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=1.56 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=0.945 ms
```

## 8. Install Basic Tools  

Install tools and packages you need. Some common ones are:  
```sudo apt-get install curl wget htop btop net-tools gzip git```  

## 9. Create backups of configuration files  

Before modifying any configuration file, always create a backup copy, for example:  

```sudo cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf.orig```  

This ensures you have the original, working version safely stored. If something goes wrong with your changes, you can easily restore the configuration by reverting to this backup.  

## 10. edituser 
If two users need to add and/or edit files in the same directory, for example another user (edituser) helps linuxuser to maintain the web-content in ```/home/linuxuser/public-sites/```, the directory and file permissions must be shared so that both users can read and write files, new files inherit the correct group, and file ownership does not become inconsistent. To achieve this Linux uses a __shared group__ and __setgid bit__.  

0) Create new user: ```sudo adduser edituser```   
1) Create shared group: ```sudo groupadd webteam```
2) Add both users to the group: ```sudo usermod -aG webteam linuxuser``` and ```sudo usermod -aG webteam edituser```  (Notice: Both users must log out and log back in before the new group membership becomes active.)
3) Assign group ownership to webteam:  
   ```sudo chown linuxuser:webteam /home/linuxuser```  (this is required only because ```public-sites/``` is inside ```/home/linuxuser```)  
   ```sudo chown -R linuxuser:webteam /home/linuxuser/public-sites```  
5) Enable the setgid bit on the directory. It ensures that all new files and directories created inside ```public-sites/``` automatically inherit the group ```webteam```, regardless of which user created them: ```sudo chmod g+s /home/linuxuser/public-sites```.  
After this directory permissions will look like: ```drwxrwsr-x``` This prevents ownership problems when either user adds new files.
6) Give the owner (linuxuser) and group (webteam) read/write access to this directory, and allow others to read (e.g. www-data):  
   ```sudo chmod -R u=rwx,g=rwx,o=rx /home/linuxuser/public-sites```  
   		__Notice:__ ```chmod -R``` must be used carefully, because it changes permissions on everything inside a directory and can break permissions or expose data if the 			contents aren’t meant to share the same access.  
8) Allow the group to enter the parent directory: ```sudo chmod g+rx /home/linuxuser```  (otherwise edituser will get permission denied error on accessing public-sites/)


```

                 ┌───────────────────────────┐
                    Linux Server (Debian)           
                 └───────────────────────────┘
                             │
                             │
                    Shared Group: webteam
                 ┌───────────────────────────┐
                    Group members:                    
                     - linuxuser              
                     - edituser             
                 └───────────────────────────┘
                             │
                             ▼
        ┌─────────────────────────────────────────┐
            /home/linuxuser/public-sites/  (shared folder)  
                                                           
            Permissions:  drwxrwsr-x                
                          │ │ │  │                        
                          │ │ │  └─ others: r-x           
                          │ │ └──── group: rwx            
                          │ └────── owner: rwx            
                          └──────── setgid bit (s)         
                                                           
            Effect:                                        
            - Both users can read/write/edit files         
            - New files inherit group = webteam            
            - No broken ownership when users add files     
            - Apache (www-data) can still read files       
        └──────────────────────────────────────────┘
                             │
                             ▼
                 ┌───────────────────────────┐
                    File creation example     
                 └───────────────────────────┘

 linuxuser creates file A:                           edituser creates file B:
 ───────────────────                            ────────────────────
   owner: linuxuser                                     owner: edituser
   group: webteam     (inherited from directory)        group: webteam
   perms: rw-rw-r--                                     perms: rw-rw-r--

  BOTH files can be edited by BOTH users because group = webteam
```
(AI assisted chart)  
