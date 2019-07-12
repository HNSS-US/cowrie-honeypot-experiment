# Installing cowrie on Digital Ocean Ubuntu 18.04 x64

### Introduction
The following are the instructions for installing cowrie on a [Digital Ocean](https://m.do.co/c/6de21b7fa280) Ubuntu 18.04 x64 droplet. For the honeypot, the minimum virtual machine is all that is required. As of this writing, the cost is $5.00/mo.  The steps are placed in this document to consolidate the steps I take to deploy a honeypot.

Once the Ubuntu 18.04 droplet is created, follow the steps outlined below to install cowrie. Note, the following steps employ ssh for connecting to the droplet. How you connect is a matter of choice. 

###### *Note: If you use the Digital Ocean console to interact with your droplet. The default configuration for the keyboard does not allow the character '|' to be typed. Instead, it returns the '>' character. The work around is to turn on Num Lock, hold alt key down and enter 124.*
#### Setup Digital Ocean account and configure droplet
   1. [Create Digital Ocean Account](https://m.do.co/c/6de21b7fa280)
   2. Recomended to create a *New Project*. For me, it is Cowrie-Honeypot-Experiment.
   3. Select desired project and [Create Digital Ocean Droplet](https://www.digitalocean.com/docs/droplets/how-to/create/)
      1. For image choose Ubuntu 18.04 x64
      2. For plan choose $5/mo
      3. Choose a datacenter region of your choice.
      4. Select additional options 'IPv6'.
      5. Authentication should be SSH keys
      6. Select username(s) of SSH keys
      7. Enter a hostname.
      8. Add tag if desied (Example: cowrie-honeypot-experiment)
      9. SSH into droplet as root from remote computer
         - $ ssh root@droplet.ip.address
      10. Create password for root
         - $ passwd
      11. Change timezone of server to match your location. (Opptional)
         - $ dpkg-reconfigure tzdata
   #### Configure Droplet user to manage server and for usernames related to cowrie.
   ###### *Note: (It is best to choose a name common or related to a server. The reason is explained later when discussing the cowrie.cfg file)*
      1. Create a user to manage the droplet. (Here still logged in remotely as root)
         1. Create User
            - $ adduser develop (user exists on remote computer)
         2. Add to the sudo group
            - $ usermod -aG sudo develop
         3. Allow user develop to run sudo without password. (Optional)
           - Make back up copy of sudoers file
              - $ cp /etc/sudoers /root/sudoers.org
           - Edit file
              - $ visudo
              - To bottom of file add:
                 develop ALL=(ALL) NOPASSWD:ALL
         4. To avoid creating SSH keys copy root ssh key
            - rsync --archive --chown=develop:develop ~/.ssh /home/develop
         4. SSH into droplet from remote computer as user develop
               - $ ssh develop@droplet.ip.address
         5. Create cowrie and releated users.
               - $ sudo adduser --disabled-password cowrie
               - $ sudo adduser --disabled-password tomcat
               - $ sudo adduser --disabled-password oracle
               - $ sudo adduser --disabled-password administrator
               - $ sudo adduser --disabled-password pi
   #### Configure droplet to have cowrie listening on port(22).
   Hacker Target has a great diagram of what is being configured with the port change in sshd_config and the iptable rules.
   ![logo](https://hackertarget.com/wp-content/uploads/2018/03/cowrie-honeypot-layout.png "cowrie ssh diagram")
   
   In the following steps, the server administration (SSH) differs from the diagram by changing 22222 to 22666.
   ###### *Note: Using IP Tables is one of three possible [methods](https://cowrie.readthedocs.io/en/latest/INSTALL.html) to have cowrie listen on Port 22.*
      1. Change default port in sshd_config
         1. sudo cp /etc/ssh/sshd_config /root/sshd_config.org
         2. sudo vi /etc/ssh/sshd_config
         3. At the top of file change #Port 22 to your value example:
            - Port 22666
         4. Check (Optional, I am always at least checking before and after changes.)
            - $ grep Port /etc/ssh/sshd_config
            - $ sudo systemctl status ssh (Curious, seeing invalid attempts from user 'pi'...)
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
              Save the configuration information.
            - $ sudo sh -c "iptables-save > /etc/iptables.rules"
              Make iptables persistent
            - $ sudo apt-get update
            - $ sudo apt-get install iptables-persistent
              Test changes
            - $ sudo shutdown -r now
              From remote computer
            - $ ssh develop@droplet.ip.address (should fail on Port 22)
            - $ ssh develop@droplet.ip.address -p 22666 (should succeed)
              Update if needed
            - sudo apt-get update
            - sudo apt-get upgrade
              Confirm the changes made.
            - $ sudo netstat -nalp | grep -i ssh (should show ssh on port 22666)
            - $ sudo iptables -t nat -L (should show redirect rules)
   #### Insalling Cowrie.
   ###### *Note: This is taken directly from the cowrie [documentation](https://cowrie.readthedocs.io/en/latest/INSTALL.html)*
        
            
            
         
