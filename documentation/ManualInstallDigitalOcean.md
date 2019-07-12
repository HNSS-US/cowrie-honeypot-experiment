# Installing cowrie on Digital Ocean Ubuntu 18.04 x64

### Introduction
The following are the instructions for installing cowrie on a [Digital Ocean](https://m.do.co/c/6de21b7fa280) Ubuntu 18.04 x64 droplet. For the honeypot, the minimum virtual machine is all that is required. As of this writing, the cost is $5.00/mo.  The steps are placed in this document to consolidate the steps I take to deploy a honeypot.

Once the Ubuntu 18.04 droplet is created, follow the steps outlined below to install cowrie. Note, the following steps employ ssh for connecting to the droplet. How you connect is a matter of choice. 

###### *Note: If you use the Digital Ocean console to interact with your droplet. The default configuration for the keyboard does not allow the character '|' to be typed. Instead, it returns the '>' character. The work around is to turn on Num Lock, hold alt key down and enter 124.*
#### Setup Digital Ocean account and configure droplet
###### *note*
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
         4. SSH into droplet as root from remote computer
            - $ ssh root@droplet.ip.address
         5. Create password for root
            - $ passwd
         6. Change timezone of server to match your location. (Opptional)
            - $ dpkg-reconfigure tzdata
#### Configure userids to manage cowrie tasks
###### *Note: (It is best to choose a name common or related to a server for fs.pickle)*
         1. Create a user to manage the droplet. (Here still logged in remotely as root)
            - $ adduser develop (user exists on remote computer)
            - $ usermod -aG sudo develop
            Allow user develop to run sudo without password. (Optional)
            - $ cp /etc/sudoers /root/sudoers.org
            - $ visudo
            develop ALL=(ALL) NOPASSWD:ALL
            To avoid creating SSH keys copy root ssh key
            - rsync --archive --chown=develop:develop ~/.ssh /home/develop
#### Create cowrie userid and others
###### *Note: (These userids are common ssh attack ids or related to a server for fs.pickle)*            
         . SSH into droplet from remote computer as user develop
      - $ ssh develop@droplet.ip.address
      6. Create cowrie and releated users.
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
   #### Insalling Cowrie
   ###### *Note: These steps are taken directly from the cowrie [documentation](https://cowrie.readthedocs.io/en/latest/INSTALL.html#step-1-install-dependencies) Note, stilled logged into the droplet as the user develop.*
         1. Install support for Python3 virtual environments and other dependencies
         -  $ sudo apt-get install git python-virtualenv libssl-dev libffi-dev build-essential libpython3-dev python3-minimal authbind
         - $ sudo su - cowrie
         2. Checkout the cowrie code from github
         - $ git clone http://github.com/cowrie/cowrie
         - $ cd cowrie
         3. Setup Python3 Virtual Environment
         - $ virtualenv --python=python3 cowrie-env
         4. Activate the virtual environment and install packages
         - $ source cowrie-env/bin/activate
         - $ pip install --upgrade pip
         - $ pip install --upgrade -r requirements.txt
         Installation of cowrie.cfg is done from HNSS github
         - wget https://raw.githubusercontent.com/HNSS-US/cowrie-honeypot-experiment/master/cowrie.cfg
#### Send Cowrie Output to a MySQL Database
###### *Note:In order to install MySQL 8.0 there are steps not in the cowrie [documentation](https://cowrie.readthedocs.io/en/latest/sql/README.html#how-to-send-cowrie-output-to-a-mysql-database)*
         1. Using curl find the latest version of MySQL
            - $ curl -s  https://repo.mysql.com// | grep deb
         2. Download file (2019-07-12)
            - $ curl -OL curl -OL https://repo.mysql.com//mysql-apt-config_0.8.13-1_all.deb
         3. Install
            - $ sudo dpkg -i mysql-apt-config_0.8.13-1_all.deb
         4. At prompt select 'OK'
         5. Delete Package
            - $ rm mysql-apt-config_0.8.13-1_all.deb
         6. Installation (Now back to cowrie [document](https://cowrie.readthedocs.io/en/latest/sql/README.html#installation)
            - $ sudo apt-get install mysql-server libmysqlclient-dev python-mysqldb
         7. Securing MySQL (Not in cowrie documentation)
            - $ sudo mysql_secure_installation
            - Enter a password for user root:*********
         8. Change to user cowrie
            - $ sudo su - cowrie (Back in cowrie documentation, missing sudo)
         9. Activate the virtual environment
            - $ source cowrie/cowrie-env/bin/activate
         10. Install mysqlclient
            - $ pip install mysqlclient
         
