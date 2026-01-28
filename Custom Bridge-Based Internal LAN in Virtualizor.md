This note documents how I created an internal LAN network in Virtualizor using **two network adapters** for a pfSense firewall, and the **issue encountered during LAN setup**.
## Creating the Bridge
Although Virtualizor creates its own Network Bridge, you will need to create the additional Bridge to use it.  
Create the bridge with whatever name you want. The following is a bridge created for internal networks with the name intbr0.

> Note : You will need **bridge-utils** package installed on the server

### Almalinux 8.x/9.x   

Configuring an internal bridge on an AlmaLinux server is similar to setting up a normal bridge.  
The commands to create an internal bridge are the same as those used for a normal bridge, viifbr0.**  
Note: The IP address range 192.168.1.x is being used as an example. so i used 10.10.x.x

```
nmcli connection add type bridge con-name intbr0 ifname intbr0

nmcli connection modify intbr0 ipv4.addresses '**10.10.1.2/24**' ipv4.dns '**10.10.1.3**'  ipv4.method manual

nmcli connection up intbr0
```

>Secondary Adapter NIC

If you have a secondary NIC for interserver internal network, you can set the following  
Assuming you have eth1 as the secondary NIC :

![[attachments/Pasted image 20260128161258.png]]


```
nmcli connection add type bridge con-name intbr0 ifname intbr0

nmcli connection modify intbr0 ipv4.addresses '**10.10.1.2/24**' ipv4.dns '**10.10.1.3**'  ipv4.method manual  

nmcli connection modify eno1 master intbr0

nmcli connection modify intbr0 connection.autoconnect-slaves 1

nmcli connection up intbr0

nmcli connection up eno1
```

![[attachments/Pasted image 20260128161225.png]]
![[attachments/Pasted image 20260128161155.png]]
adding the ip internal ip pool.


## Resolution

- Verified the correct adapter was mapped to **LAN** inside pfSense.
    
- Ensured **NAT and firewall rules** allowed LAN → WAN traffic.
    
- Confirmed internal VMs were attached to the same Virtualizor LAN bridge and receiving IPs from pfSense.

# Outcome

- Internal LAN successfully isolated and routable.
    
- Internal machines can communicate with each other and access external networks **only through pfSense**, enabling realistic lab and CTF-style environments.