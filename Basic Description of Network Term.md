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

