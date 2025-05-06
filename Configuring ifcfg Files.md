# Configuring IP Networking with ifcfg Files
As a system administrator, you can configure a network interface manually, editing the **ifcfg**  files.

Interface configuration (ifcfg) files control the software interfaces for individual network devices. As the system boots, it uses these files to determine what interfaces to bring up and how to configure them. 
These files are usually named **ifcfg-name**, where the suffix name refers to the name of the device that the configuration file controls. By convention, the ifcfg file's suffix is the same as the string given by the DEVICE directive in the configuration file itself.

### Configuring an Interface with Static Network Settings Using ifcfg Files
For example, to configure an interface with static network settings using **ifcfg** files, for an interface with the name enp1s0, create a file with the name **ifcfg-enp1s0** in the **/etc/sysconfig/network-scripts/** directory, that contains:

- For IPv4 configuration
  ```
  DEVICE=enp1s0         # The physical network interface
  BOOTPROTO=none        # No DHCP; use static IP
  ONBOOT=yes            # Bring up the interface at boot
  PREFIX=24             # Subnet mask = 255.255.255.0
  IPADDR=10.0.1.27      # Static IP address

  ```
  Equivalent nmcli Command

  ```
  nmcli con add type ethernet ifname enp1s0 con-name static-enp1s0 ip4 10.0.1.27/24 ipv4.method manual
  ```
  
- For IPv6 configuration
  ```
  DEVICE=enp1s0
  BOOTPROTO=none
  ONBOOT=yes
  IPV6INIT=yes
  IPV6ADDR=2001:db8::2/48
  ```
📖 You do not need to specify the network or broadcast address as this is calculated automatically by **ipcalc**.

### Configuring an Interface with Dynamic Network Settings Using ifcfg Files

To configure an interface named em1 with dynamic network settings using ifcfg files:

1. Create a file with the name ifcfg-em1 in the /etc/sysconfig/network-scripts/ directory, that contains:
   ```
   DEVICE=em1
   BOOTPROTO=dhcp
   ONBOOT=yes

    ```
2. To configure an interface to send a different host name to the DHCP server, add the following line to the ifcfg file:
   ```
   DHCP_HOSTNAME=hostname
   ```
   
    To configure an interface to send a different fully qualified domain name (FQDN) to the DHCP server, add the following line to the ifcfg file:
  
    ```
      DHCP_FQDN=fully.qualified.domain.name
    ```

3. To configure an interface to use particular DNS servers, add the following lines to the ifcfg file:
    ```
    PEERDNS=no
    DNS1=ip-address
    DNS2=ip-address
    ```
  where ip-address is the address of a DNS server. This will cause the network service to update /etc/resolv.conf with the specified DNS servers specified. Only one DNS server address is necessary, the other is optional.

4. To configure static routes in the ifcfg file, see [“Configuring Static Routes in ifcfg files”.](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/networking_guide/sec-Configuring_Static_Routes_in_ifcfg_files)

    By default, NetworkManager calls the DHCP client, dhclient, when a profile has been set to obtain addresses automatically by setting BOOTPROTO to dhcp in an interface configuration file. If DHCP is required, an instance of dhclient
    is started for every Internet protocol, IPv4 and IPv6, on an interface. If NetworkManager is not running, or is not managing an interface, then the legacy network service will call instances of dhclient as required.
5. To apply the configuration:
  - Reload the updated connection files:
    ```
  	   nmcli connection reload
    ```
  - Re-activate the connection:
    ```
  	  nmcli connection up connection_name
    ```

## Managing System-wide and Private Connection Profiles with ifcfg Files

The permissions correspond to the USERS directive in the ifcfg files. If the USERS directive is not present, the network profile will be available to all users. 
As an example, the following command in an ifcfg file will make the connection available only to the users listed:
  ```
    USERS="joe bob alice"
  ```
  Also, you can set the USERCTL directive to manage the device:
  
-  If you set yes, non-root users are allowed to control this device.
-  If you set no, non-root users are not allowed to control this device.

