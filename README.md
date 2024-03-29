# Welcome to the HNSS Honeypot Experiment Project
HNSS Honeypot Experiment uses the Cowrie SSH and Telnet Honeypot to study Brute Force attacks and shell interactions of  attackers. [Cowrie](https://www.cowrie.org) is a medium interaction reasearch honeypot developed by [Michel Oosterhof](http://www.micheloosterhof.com/) based on his work of extending the [Kippo honeypot](http://en.wikipedia.org/wiki/Kippo) which was developed by [deaster](https://github.com/desaster).
The instructions here do not vary greatly from the other tutorials on the internet. However, there are two exceptions. The order of the creation of username(s) and the configuration of iptables is done before installing the cowrie enviorment. Also, and perhaps most importantly is the handling of the cowrie.cfg file. All tutorials I reveiwed gave the step of *cp cowrie.cfg.dist cowrie.cfg*. Then editing cowrie.cfg to suit your needs. As Mr. Oosterhof states in [documentation](https://cowrie.readthedocs.io/en/latest/INSTALL.html#step-5-install-configuration-file), *'Both files are read on startup, where entries from cowrie.cfg take precedence.'* Therefore, it is best to create a cowrie.cfg with only the changes you want. 

## Overview of the HNSS Honeypot Experiment
This experiment will make use of the standard honeypot analysis of veiwing the frequency of userid and password combinations, the IP address and country of origin of attacks and so forth. The question that will be  tested is whether the geographical location of the server has any affect on the origins of the attacking country and the other data points.

To start, I am using using the VPS [Digital Ocean](https://m.do.co/c/6de21b7fa280) to create "droplets" on various datacenters. These centers are located in:
- New York City, United States
- Amsterdam, the Netherlands
- San Francisco, United States
- Singapore
- London, United Kingdom
- Frankfurt, Germany
- Toronto, Canada
- Bangalore, India

## Sharing Information
The cowrie.cfg file allows for sharing information with Cyber Security research organizations. In this experiment informnation is shared with [SANS DShield](https://isc.sans.edu/ssh.html), [Virus Total](https://www.virustotal.com/), and [csirtg](https://csirtg.io).

## Conclusion
The work done by Michel Oosterhof and the cowrie group is very impressive. Cowrie gives anyone with basic Linux skills and the ability to follow written instructions the means to create a great tool for analyzing Cyber Attacks.

#### Blogs of interest.
- Bløgg.no
  - [Control Code Usernames in Telnet Honeypot](https://bløgg.no/2017/12/control-code-usernames-in-telnet-honeypot/)
