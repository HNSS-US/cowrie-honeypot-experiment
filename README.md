# Welcome to HNSS Honeypot Experiment Project
HNSS Honeypot Experiment uses the Cowrie SSH and Telnet Honeypot to study Brute Force attacks and shell interactions of  attackers. [Cowrie](https://www.cowrie.org) is a medium interaction reasearch honeypot developed by [Michel Oosterhof](http://www.micheloosterhof.com/) based on his work of extending the [Kippo honeypot](http://en.wikipedia.org/wiki/Kippo) which was developed by [deaster](https://github.com/desaster).

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
The work done by Michel Oosterhof and the cowrie group is very impressive. Cowrie gives anyone with basic Linux skills and the ability to follow written instructions the path to create a great tool for analyzing Cyber Attacks.

###### Blogs of interest.
- blog
