# IAmTheNSA (a project building a central database for research and analysis)
2018 summer project put together that is a grouping of tools, scripts and a central search system that mimics the NSA but in a legal way. 


Will be updating this and posting more as I complete this project. The drive for this project is learning and something to use for getting a job.


#### Goals:

The goal of this is to collect as much information possible that is public knowledge and then categoarize it in database and use a central system to search all the projects collecting information and then nicely display all the information in one clean webpage.

The idea is to have the ability to type in any IP address and see what is running or where that IP is located. Pull websites on IPs and log their web application software for versions. Have the ability to search for a version of an application and see all that hosts that it is running. Easy research for seeing how badly a 0day could rip through the internet or networks. 

## PHASE 1 :

### => SNMP and Script Scanning prep
(( fun fact: i was able to monitor many servers, firewalls, and switches from major datacenters and hosting providers. There was also a massive amount of vulnerable IPMIs, iDRAC, ect... exposed to the internet. Sent pictures to some of my buddies who are CEOs and CTOs in the industry. They found it funny but their systems were actually done right )) 

Idea inspired after setting up several systems to monitor our devices at work. Surely, people who copy and paste from these guides 
SNMP Grabbing. 

The first step was to write a script that could find IPs from with port 161/UDP open and then pipe those IPs from nmap to snampwalk to see if the default community password worked on v2c systems.

If it worked, we could then pipe it to one of the many SNMP monitoring and graphing systems I had running. 

ipinfo.io was my friend but I needed a quick way to pull IPs without doing it by hand. 

```python
import urllib2
import re
import sys
import getopt

def main(argv):
        url = 'https://ipinfo.io/%s' % (argv)
        #print url
        content = urllib2.urlopen(url).read()
        ver_regex = re.compile(r"(?:\d{1,3}\.){3}\d{1,3}(?:/\d\d?)")
        fia = ver_regex.findall(content)
        filename = '%s' % (argv)
        f = open(filename, 'w')
        #print fia
        for x in fia:
            #print x
            f.write(x)
            f.write('\n')

if __name__ == "__main__":
   # make 0 if not doing python file.py AS12345
   main(sys.argv[1])
```

Next we could take the text file and pipe to the following script.

```
#!/bin/sh

  for ipsub in $(cat $1)
  do
          nmap -P0 -v -sU -p 161 -oA snmp_scan $ipsub
          for i in *.gnmap
          do
            for j in $(grep '161/open/' $i | awk '{ print $2 }')
            do
              snmpwalk -v2c -c public $j
                   #&> snmpwalk_${j}_public.txt
              if [ "$?" = "0" ]; then /addhost.php $j; fi
              snmpwalk -v2c -c private $j
                   #&> snmpwalk_${j}_private.txt
              if [ "$?" = "0" ]; then echo "$j accepts SNMP community string private" >> finalscans.txt; fi
            done
          done
  done
  ```

In the end, I was able to monitor 10000's of 1000's of devices. The hard part, spending days reading documents to make the servers stable and not drop drawing all the graphcs. I was finally able to stablize the system and have full statistics without dropping.

![alt text](https://preview.ibb.co/dbxDrJ/Untitled.png "SNMP GRAPHS")

It was also cool to see which datcenters took massive dDoS attacks all the time. You also had the ability to see VLANS and even internal routing and routes. This made it easier to also see how the internal and routing networks were setup. 


This was too manual but it worked but in the end I wanted something that could do this automatically and then recycle. Piped all the ASNs/Subnets to their own table which the system would then use later to pull from. This was a much more reliable and versitile way of doing this. Eventually ended up on this script and style:





### Mass IP Scanning

In order to complete the step of having a list of every IP and a list of services and versions running on that IP, we need something more powerful. 

Tool: 
massscan
