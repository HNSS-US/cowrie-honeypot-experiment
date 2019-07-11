# Installing cowrie on Digital Ocean Ubuntu 18.04 x64

### Introduction
The following are the instructions for installing cowrie on a [Digital Ocean](https://m.do.co/c/6de21b7fa280) Ubuntu 18.04 x64 droplet. For the honeypot, the minimum virtual machine is all that is required. As of this writing, this cost $5.00/mo. These steps are copied and pasted from the cowrie [documentation](https://cowrie.readthedocs.io/en/latest/). The steps are placed in this document for ease of use.

Once the Ubuntu 18.04 droplet is created, follow the steps outlined below to install cowrie. Note, the following steps employ ssh for connecting to the droplet. How you connect is a matter of choice. The steps 


###### *Note: If you use the Digital Ocean console to interact with your droplet. The default configuration for the keyboard does not allow the character '|' to be typed. Instead, it returns the '>' character. The work around is to turn on Num Lock, hold alt key down and enter 124.*
#### Setup Digital Ocean account and configure droplet
   1. [Create Digital Ocean Account](https://m.do.co/c/6de21b7fa280)
   2. Recomended to create a new project. For me, it is Cowrie-Honeypot-Experiment.
   2. If you do not have ssh keys, follow the steps in [How to Create SSH Keys with OpenSSH](https://www.digitalocean.com/docs/droplets/how-to/add-ssh-keys/create-with-openssh/)
   3. Add ssh keys to your Digital Ocean account Using the [Control Panel](https://www.digitalocean.com/docs/droplets/how-to/add-ssh-keys/to-account/).
   4. Select desired project and [Create Digital Ocean Droplet](https://www.digitalocean.com/docs/droplets/how-to/create/)
      1. For image choose Ubuntu 18.04 x64
      2. For plan choose $5/mo
      3. Choose a datacenter region of your choice.
      4. Select additional options 'IPv6'.
      5. Authentication should be SSH keys
      6. Choose a hostname. For me, I use the nomenclature of **hcluster**.*city*.*country*. For example, hcluster-fra.de, for a droplet created in Frankfurt Germany.
      7. SSH to droplet as root
         - $ ssh root@droplet.ip.address
      8. Create password for root
         - $ passwd
      9. Change timezone of server to match your location. (Opptional)
         - $ dpkg-reconfigure tzdata
   #### Configure Droplet user to manage server and for usernames related to cowrie.
   ###### *Note: (It is best to choose a name common or related to a server. The reason is explained later when discussing the cowrie.cfg file)*
      1. Create a user to manage the droplet. (Here still logged in as root)
         1. Create User
            - $ adduser develop
         2. Add to the sudo group
            - $ usermod -aG sudo develop
         3. Allow user develop to run sudo without password. (Optional)
           - Make back up copy of sudoers file
              - $ cp /etc/sudoers /root/sudoers.bak   
           - Edit file
              - $ visudo
              - To bottom of file add:
                 develop ALL=(ALL) NOPASSWD:ALL
         4. Log in as user develop
               - $ su - develop
         5. Create cowrie and releated users. (No need to fill out user info.)
               - $ sudo adduser --disabled-password cowrie
               - $ sudo adduser --disabled-password tomcat
               - $ sudo adduser --disabled-password oracle
               - $ sudo adduser --disabled-password administrator
               - $ sudo adduser --disabled-password pi
   #### Configure droplet to have cowrie listening on port(22).
   Hacker Target has a great diagram of what is being configured.
   ![logo](https://hackertarget.com/wp-content/uploads/2018/03/cowrie-honeypot-layout.png "cowrie ssh diagram")
   
   In the following steps, the server administration (SSH) is differs from the diagram by changing 22222 to 22666.
   ###### *Note: Using IP Tables one of three possible [methods](https://cowrie.readthedocs.io/en/latest/INSTALL.html) to have cowrie listen on Port 22.*
      1. Change default port in sshd_config
         1. sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bck
         2. sudo vi /etc/ssh/sshd_config
         3. At the top of file change #Port 22 to your value example:
            - Port 22666
         4. Check (Optional, I am always at least checking before and after changes.)
            - $ grep Port /etc/ssh/sshd_config
            - $ sudo systemctl status ssh (Seeing invalid attempts from user 'pi'.)
            - $ sudo netstat -nalp | grep -i ssh (Confirm ssh on Port 22)
            - $ sudo systemctl restart ssh
            - $ sudo netstat -nalp | grep -i ssh (Confirm ssh on Port 22666)
            - $ sudo systemctl status ssh 
         5. iptable rules
            - $ sudo iptables -t nat -L (Current state)
              Add rules REROUTING to where cowrie will be running on 2222 and 2223
            - $ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
            - $ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
            - $ sudo iptables -t nat -L (Confirm changed state)
            - 
            
         
