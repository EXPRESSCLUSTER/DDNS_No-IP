# Hostname failover with No-IP

This document describes the steps to configure EXPRESSCLUSTER to use [No-IP](https://noip.com) for host name failover.  
This method can be used with EXPRESSCLUSTER X for Linux, and can be used as an alternative to EXPRESSCLUSTER's DDNS resource if all cluster nodes can access the Internet.

## Steps

0. Prepare a cluster and a failover group.
1. Open EXPRESSCLUSTER WebUI > Change to [Config mode]

#### Adding EXEC resource

This steps configure a group resource which sends updates to DDNS on starting the failover group.

2. Click [Add resource] on the right of the failover group.
3. Select [EXEC resource] as [Type] > Input [exec-ddns] as [Name] > [Next]
4. Uncheck [Follow the default dependency] > [Next]
5. [Next]
6. Select [start.sh] > [Edit] > edit as folloing > [OK] > [Finis]
   ```sh
   #! /bin/sh

   # When the failover-group is active on Site#1 (Site#2), this script updates
   # Dynamic DNS to have $NOIP_HOSTNAME resolves to $IP_ADDRESS1 ($IP_ADDRESS2).
   # Parameters should be edited depend on the system configuration.

   # Parameters
   #-----------
   IP_ADDRESS1='10.0.0.1'
   IP_ADDRESS2='10.1.0.1'
   NOIP_USERNAME='USERNAME'
   NOIP_PASSWORD='PASSWORD'
   NOIP_HOSTNAME='example.ddns.net'
   #-----------
   
   if [ "$CLP_SERVER" = "HOME" ]; then
       IP_ADDRESS=$IP_ADDRESS1
   else
       IP_ADDRESS=$IP_ADDRESS2
   fi
   curl 'http://$NOIP_USERNAME:$NOIP_PASSWORD@dynupdate.no-ip.com/nic/update?hostname=$NOIP_HOSTNAME&myip=$IP_ADDR'
   exit 0
   ```
   Replace
   - [USERNAME] and [PASSWORD] by the No-IP account.
   - [example.ddns.net] by the provided hostname by No-IP.
   - [10.0.0.1] and [10.1.0.1] by the IP address associated with the hostname at each site.

#### Adding Custom monitor resource

This steps configure a monitor resource which sends updates to DDNS every 24 hours.

7. Click [Add monitor resource] onn the right of the [Monitors]
8. Select [Custom monitor] as [Type] > Input [genw-ddns] as [Name] > [Next]
9. Input [86400] sec as [Interval] > Select [Active] as [Monitor Timing] > Click [Brouse] > select [exec-ddns] > [OK] > [Next]
10. Click [Edit] > Edit as following > Input [/opt/nec/clusterpro/log/genw-ddns.log] as [Log Output Path] > Check [Rotate Log] > [Next]

   ```sh
   #!/bin/bash

   # Parameters
   #-----------
   # When the failover-group is active on alpha (bravo), this script updates
   # Dynamic DNS to have $NOIP_HOSTNAME resolves to $IP_ADDRESS1 ($IP_ADDRESS2).

   HOSTNAME1='alpha';
   IP_ADDRESS1='10.0.0.1'

   HOSTNAME2='bravo';
   IP_ADDRESS2='10.1.0.1'

   NOIP_USERNAME='USERNAME'
   NOIP_PASSWORD='PASSWORD'
   NOIP_HOSTNAME='example.ddns.net'
   #-----------

   tmp=`hostname`
   if [ tmp = $HOSTNAME1 ]; then
       IP_ADDR=$IP_ADDRESS1
   else
       IP_ADDR=$IP_ADDRESS2
   fi
   curl 'http://$NOIP_USERNAME:$NOIP_PASSWORD@dynupdate.no-ip.com/nic/update?hostname=$NOIP_HOSTNAME&myip=$IP_ADDR'

   ```
   Replace
   - [alpha] and [bravo] by the hostname of the servers in site#1 and site#2.
   - [USERNAME] and [PASSWORD] by the No-IP account.
   - [example.ddns.net] by the provided hostname by No-IP.
   - [10.0.0.1] and [10.1.0.1] by the IP address associated with the hostname at each site.

11. Click [Browse] > Select [exec-ddns] > [OK] > [Finish]
12. Click [Apply the Configuration File]

----

<div align="right">2020.05.19 Miyamoto Kazuyuki &lt;kazuyuki@nec.com&gt;</div>
