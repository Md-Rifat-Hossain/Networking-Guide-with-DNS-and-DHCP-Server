# Configuring Static Routes and the Default Gateway
  Routing is a mechanism that allows a system to find the network path to another system. Routing is often handled by devices on the network dedicated to routing (although any device can be configured to perform routing).
  Therefore, it is often not necessary to configure static routes on Red Hat Enterprise Linux servers or clients. Exceptions include traffic that must pass through an encrypted VPN tunnel or traffic that should take a specific route for reasons of cost or security.

  When a host's interface is configured by DHCP, an address of a gateway that leads to an upstream network or the Internet is usually assigned. This gateway is usually referred to as the default gateway as it is the gateway to use if no better route is known to the system (and present in the routing table).
  Network administrators often use the first or last host IP address in the network as the gateway address; for example, **192.168.10.1 or 192.168.10.254**. Not to be confused by the address which represents the network itself; in this example, 192.168.10.0, or the subnet's broadcast address; 
  in this example **192.168.10.255**. The default gateway is traditionally a network router. The default gateway is for any and all traffic which is not destined for the local network and for which no preferred route is specified in the routing table.

  
## 1. Configuring Static Routes Using nmcli
  To configure static routes using the nmcli tool, use one of the following:
  - the nmcli command line
    - To configure a static route for an existing Ethernet connection using the command line:
      
      ```
      nmcli connection modify enp1s0 +ipv4.routes "192.168.122.0/24 10.10.10.1"
      ```
      ✉️ This will direct traffic for the 192.168.122.0/24 subnet to the gateway at 10.10.10.1.
      
 
  - the nmcli interactive editor
      
    - To configure a static route for an Ethernet connection using the interactive editor:
      ```
        nmcli con edit ens3
        ===| nmcli interactive connection editor |===

        Editing existing '802-3-ethernet' connection: 'ens3'

        Type 'help' or '?' for available commands.
        Type 'describe [<setting>.<prop>]' for detailed property description.

        You may edit the following settings: connection, 802-3-ethernet (ethernet), 802-1x, dcb, ipv4, ipv6, tc, proxy
        nmcli> set ipv4.routes 192.168.122.0/24 10.10.10.1
        nmcli> save persistent
        Connection 'ens3' (23f8b65a-8f3d-41a0-a525-e3bc93be83b8) successfully updated.
        nmcli> quit
      ```
     

## 2. Configuring Static Routes with ip commands
As a system administrator, you can configure static routes using the ip route command.

To display the IP routing table, use the ip route command. For example:

  ```
    ip route
    default via 192.168.122.1 dev ens9  proto static  metric 1024
    192.168.122.0/24 dev ens9  proto kernel  scope link  src 192.168.122.107
    192.168.122.0/24 dev enp1s0  proto kernel  scope link  src 192.168.122.126
  ```
The ip route commands take the following form:

  ```
  ip route [ add | del | change | append | replace ] destination-address
  ```
where 192.0.2.1 is the IP address of the host in dotted decimal notation, 10.0.0.1 is the next hop address and interface is the exit interface leading to the next hop.

- To add a static route to a network, in other words to an IP address representing a range of IP addresses:

  ```
    ip route add 192.0.2.0/24 via 10.0.0.1 [dev interface]
  ```
   where 192.0.2.0 is the IP address of the destination network in dotted decimal notation and /24 is the network prefix. The network prefix is the number of enabled bits in the subnet mask.
   This format of network address slash network prefix length is sometimes referred to as classless inter-domain routing (CIDR) notation.
- To remove the assigned static route:
  ```
   ip route del 192.0.2.1
  ```

Any changes that you make to the routing table using ip route do not persist across system reboots. To permanently configure static routes, you can configure them by creating a route-interface file in the 
***/etc/sysconfig/network-scripts/*** directory for the interface. For example, static routes for the enp1s0 interface would be stored in the /etc/sysconfig/network-scripts/route-enp1s0 file. 
Any changes that you make to a route-interface file do not take effect until you restart either the network service or the interface. The route-interface file has two formats:


## 3. Configuring Static Routes in ifcfg files
  Static routes set using ip commands at the command prompt will be lost if the system is shutdown or restarted. To configure static routes to be persistent after a system restart, 
  they must be placed in per-interface configuration files in the /etc/sysconfig/network-scripts/ directory. The file name should be of the format route-interface. There are two types of commands to use in the configuration files:

  - Static Routes Using the IP Command Arguments Format
    If required in a per-interface configuration file, for example /etc/sysconfig/network-scripts/route-enp1s0, define a route to a default gateway on the first line. This is only required if the gateway is not set through DHCP and is not set globally in the /etc/sysconfig/network file:
      ```
      default via 192.168.1.1 dev interface
      ```
    where **192.168.1.1** is the IP address of the default gateway. The interface is the interface that is connected to, or can reach, the default gateway. The dev option can be omitted, it is optional. Note that this setting takes precedence over a setting in the **/etc/sysconfig/network** file.
  If a route to a remote network is required, a static route can be specified as follows. Each line is parsed as an individual route:
    ```
    10.10.10.0/24 via 192.168.1.1 [dev interface]
    ```
    where 10.10.10.0/24 is the network address and prefix length of the remote or destination network. The address 192.168.1.1 is the IP address leading to the remote network. It is preferably the next hop address but the address of the exit interface will work.
    The “next hop” means the remote end of a link, for example a gateway or router. The dev option can be used to specify the exit interface interface but it is not required. Add as many static routes as required.



  The following is an example of a route-interface file using the ip command arguments format. The default gateway is 192.168.0.1, interface enp1s0 and a leased line or WAN connection is available at 192.168.0.10. 
  The two static routes are for reaching the 10.10.10.0/24 network and the 172.16.1.10/32 host:
    
    ```
    default via 192.168.0.1 dev enp1s0
    10.10.10.0/24 via 192.168.0.10 dev enp1s0
    172.16.1.10/32 via 192.168.0.10 dev enp1s0
    ```
    
  In the above example, packets going to the local 192.168.0.0/24 network will be directed out the interface attached to that network. Packets going to the 10.10.10.0/24 network and 172.16.1.10/32 host will be directed to 192.168.0.10. Packets to unknown, remote,
  networks will use the default gateway therefore static routes should only be configured for remote networks or hosts if the default route is not suitable. Remote in this context means any networks or hosts that are not directly attached to the system.

  The ip route format can be used to specify a source address. For example:
    
    ```
    10.10.10.0/24 via 192.168.0.10 src 192.168.0.2
    ```
    
  ### Static Routes Using the Network/Netmask Directives Format

  You can also use the network/netmask directives format for route-interface files. The following is a template for the network/netmask format, with instructions following afterwards:
    
    ```
    ADDRESS0=10.10.10.0  #ADDRESS0=10.10.10.0 is the network address of the remote network or host to be reached.
    NETMASK0=255.255.255.0  #NETMASK0=255.255.255.0 is the netmask for the network address defined with ADDRESS0=10.10.10.0.
    GATEWAY0=192.168.1.1  #GATEWAY0=192.168.1.1 is the default gateway, or an IP address that can be used to reach ADDRESS0=10.10.10.0
    ```
    
he following is an example of a route-interface file using the network/netmask directives format. The default gateway is 192.168.0.1 
but a leased line or WAN connection is available at 192.168.0.10. The two static routes are for reaching the 10.10.10.0/24 and 172.16.1.0/24 networks:

  ```

  ADDRESS0=10.10.10.0
  NETMASK0=255.255.255.0
  GATEWAY0=192.168.0.10
  ADDRESS1=172.16.1.10
  NETMASK1=255.255.255.0
  GATEWAY1=192.168.0.10
  ```
✉️ Subsequent static routes must be numbered sequentially, and must not skip any values. For example, ADDRESS0, ADDRESS1, ADDRESS2, and so on.
