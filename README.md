# Hostname failover with No-IP

This document describes the steps to configure EXPRESSCLUSTER to use [No-IP](https://noip.com) for host name failover.  
This method can be used with EXPRESSCLUSTER X for Linux, and can be used as an alternative to EXPRESSCLUSTER's DDNS resource if all cluster nodes can access the Internet.

## Steps

Prepare following
1. A cluster and a failover group.
2. No-IP account (ID/PASSWORD) and hostname (FQDN) for the cluster.

#### Adding EXEC resource

This steps configure a group resource which sends updates to DDNS on starting the failover group.

1. Open EXPRESSCLUSTER WebUI > Change to [Config mode]
2. Click [Add resource] on the right of the failover group.
3. Select [EXEC resource] as [Type] > Input [exec-ddns] as [Name] > [Next]
4. Uncheck [Follow the default dependency] > [Next]
5. Select [No operation 8activate next resource) and [No operation (deactivate next resource) as [Final Action] > [Next]
6. Select [start.sh] > [Edit] > edit as folloing

    ```bash
    #!/bin/sh

    # When the failover-group is active on Site#1 (Site#2), this  script updates
    # Dynamic DNS to have $NOIP_HOSTNAME resolves to $IP_ADDRESS1  ($IP_ADDRESS2).
    # Parameters should be edited depend on the system  configuration.

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
    echo $$ > /var/run/exec-ddns
    while [ 1 ]; do
    	curl "http://$NOIP_USERNAME:$NOIP_PASSWORD@dynupdate.no-ip.com/nic/update?hostname=$NOIP_HOSTNAME&myip=$IP_ADDR"
    	sleep $((60*60*24))
    done
    ```

   Replace
   - [USERNAME] and [PASSWORD] by the No-IP account.
   - [example.ddns.net] by the provided hostname by No-IP.
   - [10.0.0.1] and [10.1.0.1] by the IP address associated with the hostname at each site.

   Select [stop.sh] > [Edit] > edit as folloing > [OK] > [Finish]

    ```bash
    #!/bin/sh

    kill -HUP `cat /var/run/exec-ddns.pid`
    ```

7. Click [Apply the Configuration File]

----

<div align="right">2020.05.19 Miyamoto Kazuyuki &lt;kazuyuki@nec.com&gt;</div>
<div align="right">2020.06.05 Miyamoto Kazuyuki &lt;kazuyuki@nec.com&gt;</div>
