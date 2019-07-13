# Installing cowrie on Digital Ocean Ubuntu 18.04 x64

### Introduction
The following are the instructions for installing cowrie on a [Digital Ocean](https://m.do.co/c/6de21b7fa280) Ubuntu 18.04 x64 droplet. For the honeypot, the minimum virtual machine is all that is required. As of this writing, the cost is $5.00/mo.  The steps are placed in this document to consolidate the steps I take to deploy a honeypot.

Once the Ubuntu 18.04 droplet is created, follow the steps outlined below to install cowrie. Note, the following steps employ ssh for connecting to the droplet. How you connect is a matter of choice. 

> *Note: If you use the Digital Ocean console to interact with your droplet. The default configuration for the keyboard does not allow the character '|' to be typed. Instead, it returns the '>' character. The work around is to turn on Num Lock, hold alt key down and enter 124.*
#### Setup Digital Ocean account and configure droplet
         1. Create [Digital Ocean Account](https://m.do.co/c/6de21b7fa280)
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
            1. $ ssh root@droplet.ip.address
         5. Create password for root
            1. $ passwd
         6. Change timezone of server to match your location. (Opptional)
            1. $ dpkg-reconfigure tzdata
#### Configure userid to manage cowrie tasks
> *Note: (It is best to choose a name common or related to a server for fs.pickle)*
         1. Create a user to manage the droplet. (Here still logged in remotely as root)
            1. $ adduser develop (user exists on remote computer)
            2. $ usermod -aG sudo develop
         2. Allow user develop to run sudo without password. (Optional)
            1. $ cp /etc/sudoers /root/sudoers.org
            2. $ visudo
            (Add to bottom of the file)
            develop ALL=(ALL) NOPASSWD:ALL
         3. To avoid creating SSH keys for develop copy root's ssh key
            1. rsync --archive --chown=develop:develop ~/.ssh /home/develop
#### Create cowrie userid and others
###### *Note: (These userids are common ssh attack ids or related to a server for fs.pickle)*            
         1. SSH into droplet from remote computer as user develop
            1. $ ssh develop@droplet.ip.address
         2. Create cowrie and releated users.
            1. $ sudo adduser --disabled-password cowrie
            2. $ sudo adduser --disabled-password tomcat
            3. $ sudo adduser --disabled-password oracle
            4. $ sudo adduser --disabled-password administrator
            5. $ sudo adduser --disabled-password pi
#### Configure droplet to have cowrie listening on port(22).
[Hacker Target](https://hackertarget.com) has a great diagram of what is being configured with the port change in sshd_config and the iptable rules.
   ![logo](https://hackertarget.com/wp-content/uploads/2018/03/cowrie-honeypot-layout.png "cowrie ssh diagram")
   
In the following steps, the server administration (SSH) differs from the diagram by changing 22222 to 22666.
###### *Note: Using IP Tables is one of three possible [methods](https://cowrie.readthedocs.io/en/latest/INSTALL.html) to have cowrie listen on Port 22.*
         1. Change default port in sshd_config
            1. sudo cp /etc/ssh/sshd_config /root/sshd_config.org
            2. sudo vi /etc/ssh/sshd_config
            3. At the top of file change #Port 22 to your value example:
            Port 22666
         2. Check (Optional, I am always at least checking before and after changes.)
            1. $ grep Port /etc/ssh/sshd_config
            2. $ sudo systemctl status ssh (Curious, seeing invalid attempts from user 'pi'...)
            3. $ sudo netstat -nalp | grep -i ssh (Confirm ssh on Port 22)
            4. $ sudo systemctl restart ssh
            5. $ sudo netstat -nalp | grep -i ssh (Confirm ssh on Port 22666)
            6. $ sudo systemctl status ssh
#### Configure iptable rules.
         1. iptable rules
            1. $ sudo iptables -t nat -L (Current state)
              Add rules REROUTING to where cowrie will be running on 2222 and 2223
            2. $ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
            3. $ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
            4. $ sudo iptables -t nat -L (Confirm changed state)
              Save the configuration information.
            5. $ sudo sh -c "iptables-save > /etc/iptables.rules"
         2. Make iptables persistent
            1. $ sudo apt-get update
            2. $ sudo apt-get install iptables-persistent
         3. Test changes
            1. $ sudo shutdown -r now
              From remote computer
            2. $ ssh develop@droplet.ip.address (should fail on Port 22)
            3. $ ssh develop@droplet.ip.address -p 22666 (should succeed)
              Update if needed
            4. sudo apt-get update
            5. sudo apt-get upgrade
         4. Confirm the changes made.
            1. $ sudo netstat -nalp | grep -i ssh (should show ssh on port 22666)
            2. $ sudo iptables -t nat -L (should show redirect rules)
#### Insalling Cowrie
###### *Note: These steps are taken from the cowrie [documentation](https://cowrie.readthedocs.io/en/latest/INSTALL.html#step-1-install-dependencies) plus additional steps to install MySQL 8. Note, stilled logged into the droplet as the user develop.*
         1. Install support for Python3 virtual environments and other dependencies
            1.  $ sudo apt-get install git python-virtualenv libssl-dev libffi-dev build-essential libpython3-dev python3-minimal authbind
            2. $ sudo su - cowrie
         2. Checkout the cowrie code from github
            1. $ git clone http://github.com/cowrie/cowrie
            2. $ cd cowrie
         3. Setup Python3 Virtual Environment
            1. $ virtualenv --python=python3 cowrie-env
         4. Activate the virtual environment and install packages
            1. $ source cowrie-env/bin/activate
            2. $ pip install --upgrade pip
            3. $ pip install --upgrade -r requirements.txt
         5. Installation of cowrie.cfg is done from HNSS github
            1. wget https://raw.githubusercontent.com/HNSS-US/cowrie-honeypot-experiment/master/cowrie.cfg
#### Send Cowrie Output to a MySQL Database
###### *Note:In order to install MySQL 8.0 there are steps not in the cowrie [documentation](https://cowrie.readthedocs.io/en/latest/sql/README.html#how-to-send-cowrie-output-to-a-mysql-database)*
         1. Using web browser find the latest version of MySQL
            1. $ https://repo.mysql.com//
         2. Download file (2019-07-12)
            1. $ curl -OL curl -OL https://repo.mysql.com//mysql-apt-config_0.8.13-1_all.deb
         3. Install
            1. $ sudo dpkg -i mysql-apt-config_0.8.13-1_all.deb
         4. At prompt select 'OK'
         5. Delete Package
            -1. $ rm mysql-apt-config_0.8.13-1_all.deb
         6. Installation (Now back to cowrie [document](https://cowrie.readthedocs.io/en/latest/sql/README.html#installation)
            1. $ sudo apt-get install mysql-server libmysqlclient-dev python-mysqldb
         7. Securing MySQL (Not in cowrie documentation)
            1. $ sudo mysql_secure_installation
            2. Enter a password for user root:*********
         8. Change to user cowrie
            1. $ sudo su - cowrie (Back in cowrie documentation, missing sudo)
         9. Activate the virtual environment
            1. $ source cowrie/cowrie-env/bin/activate
         10. Install mysqlclient
            1. $ pip install mysqlclient
### MySQL Configuration
         1. ssh into droplet as user develop
         2. Create an empty database named ‘cowrie'
            1. $ sudo mysql -u root
            2. mysql>  CREATE DATABASE cowrie;
            3. mysql> GRANT ALL ON cowrie.* TO 'cowrie'@'localhost' IDENTIFIED BY 'PASSWORD HERE';
            4. mysql> FLUSH PRIVILEGES;
            5. mysql> exit
            Create cowrie database schema
            6. $ cd /home/cowrie/cowrie/docs/sql
            7. $ mysql -u cowrie -p
            8. mysql> USE cowrie;
            9. mysql> source mysql.sql;
            10. mysql> exit
### Configure cowrie.cfg
            1. cd /home/cowrie/cowrie/etc
            2. sudo 
            
