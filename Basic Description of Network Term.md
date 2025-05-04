# Static IP addressing
When a device is assigned a static IP address, the address does not change over time unless changed manually. It is recommended to use static IP addressing if you want:
- To ensure network address consistency for servers such as DNS, and authentication servers.
* To use out-of-band management devices that work independently of other network infrastructure.

All the configuration tools listed in [“Selecting Network Configuration Methods”](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/networking_guide/ch-Configuring_IP_Networking#sec-Selecting_Network_Configuration_Methods) allow assigning static IP addresses manually. 
The nmcli tool is also suitable, [“Adding and Configuring a Static Ethernet Connection with nmcli”](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/networking_guide/sec-Configuring_IP_Networking_with_nmcli#sec-Adding_and_Configuring_a_Static_Ethernet_Connection_with_nmcli) 

# Dynamic IP addressing
When a device is assigned a dynamic IP address, the address changes over time. For this reason, it is recommended for devices that connect to the network occasionally because IP address might be changed after rebooting the machine.
Dynamic IP addresses are more flexible, easier to set up and administer. The dynamic host control protocol (DHCP) is a traditional method of dynamically assigning network configurations to hosts. See [“Why Use DHCP?”](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/networking_guide/ch-DHCP_Servers#sec-dhcp-why) for more information. 

> [!NOTE]
> By default, NetworkManager calls the DHCP client, dhclient..

# Configuring the DHCP Client Behavior
A Dynamic Host Configuration Protocol (DHCP) client requests the dynamic IP address and corresponding configuration information from a DHCP server each time a client connects to the network.
> Note that **NetworkManager** calls the DHCP client, **dhclient** by default.

### Requesting an IP Address
When a DHCP connection is started, a dhcp client requests an IP address from a DHCP server. The time that a dhcp client waits for this request to be completed is 60 seconds by default. 
You can configure the ipv4.dhcp-timeout property using the nmcli tool or the IPV4_DHCP_TIMEOUT option in the /etc/sysconfig/network-scripts/ifcfg-ifname file. For example, using nmcli:


``` 
~]# nmcli connection modify enp1s0 ipv4.dhcp-timeout 10
```

If an address cannot be obtained during this interval, the IPv4 configuration fails. The whole connection may fail, too, and this depends on the ipv4.may-fail property:

* If ipv4.may-fail is set to yes (default), the state of the connection depends on IPv6 configuration:
  
   **1.** If the IPv6 configuration is enabled and successful, the connection is activated, but the IPv4 configuration can never be retried again.
  
   **2.** If the IPv6 configuration is disabled or does not get configured, the connection fails.
* If ipv4.may-fail is set to no the connection is deactivated. In this case:
  
  **1.** If the autoconnect property of the connection is enabled, NetworkManager retries to activate the connection as many times as set in the autoconnect-retries property. The default is 4.
  
  **2.** If the connection still cannot acquire the dhcp address, auto-activation fails.
> Note that after 5 minutes, the auto-connection process starts again and the dhcp client retries to acquire an address from the dhcp server.

### Requesting a Lease Renewal
When a dhcp address is acquired and the IP address lease cannot be renewed, the dhcp client is restarted for three times every 2 minutes to try to get a lease from the dhcp server. Each time, it is configured by setting the ipv4.dhcp-timeout property in seconds (default is 60) to get the lease. If you get a reply during your attempts, the process stops and you get your lease renewed.
After three attempts failed:

- If ipv4.may-fail is set to yes (default) and IPv6 is successfully configured, the connection is activated and the dhcp client is restarted again every 2 minutes.
- If ipv4.may-fail is set to no, the connection is deactivated. In this case, if the connection has the autoconnect property enabled, the connection is activated from scratch.

###  Making DHCPv4 Persistent
To make DHCPv4 persistent both at startup and during the lease renewal processes, set the ipv4.dhcp-timeout property either to the maximum for a 32-bit integer (MAXINT32), which is 2147483647, or to the infinity value:
```
~]$ nmcli connection modify enps1s0 ipv4.dhcp-timeout infinity
```
As a result, NetworkManager never stops trying to get or renew a lease from a DHCP server until it is successful.
To ensure a DHCP persistent behavior only during the lease renewal process, you can manually add a static IP to the IPADDR property in the /etc/sysconfig/network-scripts/ifcfg-enp1s0 configuration file or by using nmcli:
```
~]$ nmcli connection modify enp1s0 ipv4.address 192.168.122.88/24
```
When an IP address lease expires, the static IP preserves the IP state as configured or partially configured (you can have an IP address, but you are not connected to the Internet), making sure that the dhcp client is restarted every 2 minutes.


## Configuring netconsole for disk logging fails

The netconsole kernel module enables to log kernel messages to another computer over the network.
To be able to use netconsole, you need to have an rsyslog server that is properly configured on your network.

- Procedure 1. Configuring an rsyslog server for netconsole

1. Configure the rsyslogd daemon to listen on the 514/udp port and receive messages from the network by uncommenting the following lines in the MODULES section of the /etc/rsyslog.conf file:

```
     $ModLoad imudp
     $UDPServerRun 514
```
2. Restart the rsyslogd service for the changes to take effect:

```
    ]# systemctl restart rsyslog

```

3. Verify that rsyslogd is listening on the 514/udp port:


```
   ]# netstat -l | grep syslog
  udp        0      0 0.0.0.0:syslog          0.0.0.0:*
  udp6       0      0 [::]:syslog             [::]:*

```
> [!Note]
> The 0.0.0.0:syslog and [::]:syslog values in the netstat -l output mean that rsyslogd is listening on default netconsole port defined in the /etc/services file:

```
  ]$ cat /etc/services | grep syslog
  syslog          514/udp
  syslog-conn     601/tcp                 # Reliable Syslog Service
  syslog-conn     601/udp                 # Reliable Syslog Service
  syslog-tls      6514/tcp                # Syslog over TLS
  syslog-tls      6514/udp                # Syslog over TLS
  syslog-tls      6514/dccp               # Syslog over TLS
```

**Netconsole** is configured using the **/etc/sysconfig/netconsole** file, which is a part of the initscripts package. This package is installed by default and it also provides the netconsole service.

If you want to configure a sending machine, follow this procedure:
- Procedure 2. Configuring a Sending Machine

1. Set the value of the *SYSLOGADDR* variable in the **/etc/sysconfig/netconsole** file to match the IP address of the syslogd server. For example:
```
  SYSLOGADDR=192.168.0.1

```
2. Restart the netconsole service for the changes to take effect:
```
  ]# systemctl restart netconsole.service

```
3. Enable netconsole.service to run after rebooting the system:
```
  ]# systemctl enable netconsole.service

```
4. View the netconsole messages from the client in the /var/log/messages file (default) or in the file specified in rsyslog.conf.

```
 ]# cat /var/log/messages

```

>Note
>By default, rsyslogd and netconsole.service use port 514. To use a different port, change the following line in /etc/rsyslog.conf to the required port number:
>On the sending machine, uncomment and edit the following line in the /etc/sysconfig/netconsole file:
>SYSLOGPORT=514
```
  $UDPServerRun <PORT>
```

## Using Network Kernel Tunables with sysctl

The sysctl -a command in Linux displays all current kernel parameters and their values. These parameters control the behavior of the Linux kernel and can affect things like networking, memory management, process limits, and logging.

To change network settings, use the sysctl commands. For permanent changes that persist across system restarts, add lines to the **/etc/sysctl.conf** file.

To display a list of all available sysctl parameters, enter as root:
```
sysctl -a | grep net.ipv4

```

For more details on network kernel tunables using sysctl, see the ["Using PTP with Multiple Interfaces section in the System Administrator's Guide."](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ptp_using_ptp4l#sec-Using_PTP_with_Multiple_Interfaces)

For more information on network kernel tunables, see the ["Network Interface Tunables section in the Kernel Administration Guide."](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html-single/kernel_administration_guide/index#network_interface_tunables)

## Managing Data Using the ncat utility

##### Brief Selection of ncat Use Cases

-  Enabling Communication between a Client and a Server
  1. Set a client machine to listen for connections on TCP port 8080:
```
   ncat -l 8080
```
2. On a server machine, specify the IP address of the client and use the same port number:
```
  ncat 10.0.11.60 8080
```
You can send messages on either side of the connection and they appear on both local and remote machines.

4. Press Ctrl+D to close the TCP connection.

***Use with UDP (instead of TCP):***
To use UDP instead of TCP (the default for ncat), use the -u option:
```
ncat -u -l 1234
```

```
ncat -u <hostname_or_IP> 1234

```
- Sending Files

***Instead of printing information on the screen, as mentioned in the previous example, you can send all information to a file. For example, to send a file over TCP port 8080 from a client to a server:***
1. On a client machine, to listen a specific port transferring a file to the server machine:
 ```
  ncat -l 8080 > outputfile
```
2. On a server machine, specify the IP address of the client, the port and the file which is to be transferred:
 ```
   ncat -l 10.0.11.60 8080 < inputfile
```
After the file is transferred, the connection closes automatically.

 Creating an HTTP proxy server

***To create an HTTP proxy server on localhost port 8080:***
```
  ncat -l --proxy-type http localhost 8080
```

***Port Scanning***
To view which ports are open, use the –z option and specify a range of ports to scan:

```
  ncat -z 10.0.11.60 80-90
  Connection to 192.168.0.1 80 port [tcp/http] succeeded!
```

***Setting up Secure Client-Server Communication Using SSL***

Set up SSL on a server:
```
  ncat -e /bin/bash -k -l 8080 --ssl
```
On a client machine:
```
  ncat --ssl 10.0.11.60 8080

```

***Connect to a remote service (client mode):***

To connect to a remote server:

```
ncat <hostname_or_IP> 1234
```

Replace <hostname_or_IP> with the actual target server's address.

1234: The port to connect to.

Once connected, you can send text or data between the client and server.


***Secure Connections with SSL/TLS***

You can also use ncat for secure communication over SSL:

Listening for SSL/TLS connections:
```
ncat -l 1234 --ssl
```

Connecting with SSL/TLS:
```
ncat <hostname_or_IP> 1234 --ssl
```


