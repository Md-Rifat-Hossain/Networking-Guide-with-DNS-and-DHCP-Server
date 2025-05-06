# Configuring IP Networking with ip Commands

As a system administrator, you can configure a network interface using the ip command, but but changes are not persistent across reboots; when you reboot, you will lose any changes.
The commands for the ip utility, sometimes referred to as iproute2 after the upstream package name, are documented in the man ip(8) page. The package name in Red Hat Enterprise Linux 7 is iproute.
If necessary, you can check that the ip utility is installed by checking its version number as follows:
  ```
    ip -V
    ip utility, iproute2-ss130716
  ```
> Output tells you that your system is using the ip utility from the iproute2 package, specifically the snapshot from July 16, 2013 (ss130716).

The ip commands can be used to add and remove addresses and routes to interfaces in parallel with NetworkManager, which will preserve them and recognize them in nmcli, nmtui, control-center, and the D-Bus API.

- To bring an interface down:
    ```
      ip link set ifname down
    ```
  ✉️ Note that the *ip* utility replaces the **ifconfig** utility because the *net-tools* package (which provides ifconfig) does not support InfiniBand addresses.
  
For information about available OBJECTs, use the ip help command. For example: **ip link help** and **ip addr help**.

✉️ Examples of using the command line and configuration files for each task are included after nmtui and nmcli examples but before explaining the use of one of the graphical user interfaces to NetworkManager, namely, control-center and nm-connection-editor.

The ip utility can be used to assign IP addresses to an interface with the following form:
  ```
    ip addr [ add | del ] address dev ifname
  ```

### Assigning a Static Address Using ip Commands
  
  To assign an IP address to an interface:
  
    
      ip address add 10.0.0.3/24 dev enp1s0
      You can view the address assignment of a specific device:
      ip addr show dev enp1s0
      2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether f0:de:f1:7b:6e:5f brd ff:ff:ff:ff:ff:ff
        inet 10.0.0.3/24 brd 10.0.0.255 scope global global enp1s0
        valid_lft 58682sec preferred_lft 58682sec
        inet6 fe80::f2de:f1ff:fe7b:6e5f/64 scope link
        valid_lft forever preferred_lft forever
### Configuring Multiple Addresses Using ip Commands
  As the ip utility supports assigning multiple addresses to the same interface it is no longer necessary to use the alias interface method of binding multiple addresses to the same interface. The ip command to assign an address can be repeated multiple times in order 
  to assign multiple address. For example:

  ```
      ip address add 192.168.2.223/24 dev enp1s0
     ip address add 192.168.4.223/24 dev enp1s0
     ip addr
      3: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
      link/ether 52:54:00:fb:77:9e brd ff:ff:ff:ff:ff:ff
      inet 192.168.2.223/24 scope global enp1s0
      inet 192.168.4.223/24 scope global enp1s0
  ```
