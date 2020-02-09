# How to set up a firewall using FirewallD on CentOS 8

last updated December 1, 2019 



[![img](https://www.cyberciti.biz/media/new/category/old/centos_logo.png)](https://www.cyberciti.biz/faq/category/centos/)

Iam a new CentOS Enterprise Linux 8 sysadmin. How do I set up a firewall using FirwallD on CentOS 8?*Introduction*– A Linux firewall used to protect your workstation or server from unwanted traffic. You can set up rules to either block traffic or allow through. CentOS 8 comes with a dynamic, customizable host-based firewall with a D-Bus interface. You can add or delete or update firewall rules without restarting the firewall daemon or service. The firewall-cmd act as a frontend for the nftables. In CentOS 8 nftables replaces iptables as the default Linux network packet filtering framework. This page shows**how to set up a firewall for your CentOS 8 and manage with the help of firewall-cmd**administrative tool.







## Basic concepts of FirewallD

firewalld simplifies the concepts of network traffic management. You have two main ideas as follows when it comes to firewalld on CentOS 8.

### 1. zones

Firewalld zones are nothing but predefined sets of rules. You can see all zones by running the following ls command:
`$ ls -l /usr/lib/firewalld/zones/`
Use the [cat command](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/) to view drop zone:
`$ cat /usr/lib/firewalld/zones/public.xml`
![How to list all firewalld zones on CentOS 8](https://www.cyberciti.biz/media/new/faq/2019/09/How-to-list-all-firewalld-zones-on-CentOS-8.png)

### Understanding predefined zones

1. **block** – All incoming network connections rejected. Only network connections initiated from within the system are possible.
2. **dmz** – Classic demilitarized zone (DMZ) zone that provided limited access to your LAN and only allows selected incoming ports.
3. **drop** – All incoming network connections dropped, and only outgoing network connections allowed.
4. **external** – Useful for router type of connections. You need LAN and WAN interfaces too for masquerading (NAT) to work correctly.
5. **home** – Useful for home computers such as laptops and desktops within your LAN where you trust other computers. Allows only selected TCP/IP ports.
6. **internal** – For use on internal networks when you mostly trust the other servers or computers on the LAN.
7. **public** – You do not trust any other computers and servers on the network. You only allow the required ports and services. For cloud servers or server hosted at your place always use public zone.
8. **trusted** – All network connections are accepted. I do not recommend this zone for dedicated servers or VMs connected to WAN.
9. **work** – For use at your workplace where you trust your coworkers and other servers.

Run the following command to see all zones on CentOS 8:
`$ firewall-cmd --get-zones`

### How to find out your default zone

One can assign network interface and source to a zone. One of these zones set as the default zone. To get your default zone run:
`$ firewall-cmd --get-default-zone`
To see your network interface names run either [ip command](https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/) or nmcli command:
`$ ip link show$ nmcli device status`
When new interface connection added (such as eth0 or ens3) to NetworkManager, they are attached to the default zone. Verify it by running the following command:
`$ firewall-cmd --get-active-zones`
![firewall-cmd --get-active-zones](https://www.cyberciti.biz/media/new/faq/2019/09/firewall-cmd-get-active-zones.png)

### 2. services

A service is nothing but a list of local ports, protocols, source ports, destinations, and firewall helper modules. Some examples:

- Port – 443 or 25 or 110
- Service – SSH, HTTP
- Protocols – ICMP

### How to see firewall rules or services associated with the public zone

Run:
`$ sudo firewall-cmd --list-all`
OR
`$ sudo firewall-cmd --list-all --zone=public`
![How to find out your defaul firewalld zones and rules](https://www.cyberciti.biz/media/new/faq/2019/09/How-to-find-out-your-defaul-firewalld-zones-and-rules.png)
The above commands indicate that my default zone is public and I am allowing incoming SSH connections (port 22), dhcpv6-client, and [cockpit service port on CentOS 8/RHEL 8](https://www.cyberciti.biz/faq/install-activate-cockpit-the-web-console-on-rhel-8/). All other traffic dropped by default. If I configure Apache or Nginx on CentOS 8, I need to open port 80/443 using firewall-cmd. Say you do not want unnecessary services such as cockpit or dhcpv6-client, you can drop them by modifying rules. For example, remove services dhcpv6-client and cockpit:
`$ sudo firewall-cmd --remove-service=cockpit --permanent$ sudo firewall-cmd --remove-service=dhcpv6-client --permanent$ sudo firewall-cmd --reload$ sudo firewall-cmd --list-services`
![Remove services dhcpv6-client and cockpit on CentOS 8](https://www.cyberciti.biz/media/new/faq/2019/09/Remove-services-dhcpv6-client-and-cockpit-on-CentOS-8.png)

### How to see which services are allowed in the current zone

`$ sudo firewall-cmd --list-services`
OR
`$ sudo firewall-cmd --list-services --zone=public$ sudo firewall-cmd --list-services --zone=home`
OR use [bash for loop](https://www.cyberciti.biz/faq/bash-for-loop/) as follows:

```
`## or just use 'sudo firewall-cmd --list-all-zones' ## for z in $(firewall-cmd --get-zones) do      echo "Services allowed in $z zone: $(sudo firewall-cmd --list-services --zone=$z)" done`
```

![Firewalld see which services are allowed in the current zone](https://www.cyberciti.biz/media/new/faq/2019/09/Firewalld-see-which-services-are-allowed-in-the-current-zone.png)

## How to start, stop, restart firewalld service on an CentOS 8

By now you know about firewalld zones, services, and how to view the defaults. It is time to activate and configure our firewall on CentOS 8 Linux box.

### Start and enable firewalld

```
$ sudo systemctl start firewalld$ sudo systemctl enable firewalld
```

### Stop and disable firewalld

```
$ sudo systemctl stop firewalld$ sudo systemctl disable firewalld
```

### Check the firewalld status

```
$ sudo firewall-cmd --state
```

### Command to reload a firewalld configuration when you make change to rules

```
$ sudo firewall-cmd --reload
```

### Get the status of the firewalld service

`$ sudo systemctl status firewalld`
![Installing and Managing FirewallD on CentOS 8](https://www.cyberciti.biz/media/new/faq/2019/09/Installing-and-Managing-FirewallD-on-CentOS-8.png)

## Understanding runtime and permanent firewall rule sets

Runtime firewalld configuration changes are temporary. When you reboot the CetnOS 8 server, they are gone. For example, the following will temporarily open TCP port 80/443 (https) for the Nginx/Apache web server:
`$ sudo firewall-cmd --zone=public --add-service=http$ sudo firewall-cmd --zone=public --add-service=https`
Above rule is not retained when you reboot the Linux box or upon restarting firewalld services itself.

### How to add the rule to the permanent set and reload firewalld

Let us add rule (HTTPS/443 and HTTP/80) permanently and reload firewalld:
`$ sudo firewall-cmd --zone=public --add-service=http --permanent$ sudo firewall-cmd --zone=public --add-service=https --permanent$ sudo firewall-cmd --reload`
Verify it:
`$ sudo firewall-cmd --list-services$ sudo firewall-cmd --list-services --permanent`

![Firewalld runtime vs permanent rule set examples](https://www.cyberciti.biz/media/new/faq/2019/09/Firewalld-runtime-vs-permanent-rule-set-examples.png)Firewalld runtime vs permanent rule set examples



## How to find of list of services supported by firewalld

The syntax is as follows on your system:
`$ sudo firewall-cmd --get-services$ sudo firewall-cmd --get-services | grep mysql$ ls -l /usr/lib/firewalld/services/$ cat /usr/lib/firewalld/services/ssh.xml`

![Firewalld get a list of the available services](https://www.cyberciti.biz/media/new/faq/2019/09/Firewalld-get-a-list-of-the-available-services.png)Firewalld get a list of the available services to add or delete from rule sets



## Firewalld rule sets examples

Let us see some common examples of firewalld for your default zone.

### How to add a service to your zone

Add dns service (TCP/UDP port 53):
`sudo firewall-cmd --zone=public --add-service=dns --permanent`

### How to remove (delete) service from your zone

Delete vnc server service (TCP port range 5900-5903):
`sudo firewall-cmd --zone=public --remove-service=vnc-server --permanent`

### How to allow/open TCP/UDP port/protocol

Open TCP port # 9009:
`sudo firewall-cmd --zone=public --add-port=9009/tcp --permanent`
To view added ports, run:
`$ sudo firewall-cmd --zone=internal --list-ports`

### How to deny/block TCP/UDP port/protocol

Open TCP port # 23:
`sudo firewall-cmd --zone=public --remove-port=23/tcp --permanent`

### How to write port forwarding firewalld rule

Forward TCP port 443 to 8080 on the same server:
`$ sudo firewall-cmd --zone=public --add-forward-port=port=80:proto=tcp:toport=8080 --permanent`
To delete above port forwarding, run
`$ sudo firewall-cmd --zone=public --remove-forward-port=port=80:proto=tcp:toport=8080`
Turn on masquerading if you need to forward traffic (port 443) to lxd server/container hosted at 192.168.2.42 port 443:
`$ sudo firewall-cmd --zone=public --add-masquerade$ sudo firewall-cmd --zone=public --add-forward-port=port=443:proto=tcp:toport=443:toaddr=192.168.2.42 --permanent`
To delete above masquerading rules, run:
`$ sudo firewall-cmd --zone=public --remove-masquerade$ firewall-cmd --zone=public --remove-forward-port=port=443:proto=tcp:toport=443:toaddr=192.168.2.42 --permanent`
As usual use the following to list rules:
`$ firewall-cmd --zone=public --list-all --permanent`

### Rich rule example

Say you want to allow access to SSH port 22 only from 10.8.0.8 IP address, run:
`sudo firewall-cmd --permanent --zone=public --add-rich-rule 'rule family="ipv4" source address="10.8.0.8" port port=22 protocol=tcp accept'`
To verify new rules, run:
`$ sudo firewall-cmd --list-rich-rules --permanent`
In this following example allow 192.168.1.0/24 sub/net to access tcp port 11211:

```
`sudo firewall-cmd --permanent --zone=public --add-rich-rule=' rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="11211" accept'`
```

Again verify it:
`$ sudo firewall-cmd --list-rich-rules --permanent`
Sample outputs:

```
rule family="ipv4" source address="10.8.0.8" port port="22" protocol="tcp" accept
rule family="ipv4" source address="192.168.1.0/24" port port="11211" protocol="tcp" accept
```

You can delete rich rules as follows:
`$ sudo firewall-cmd --remove-rich-rule 'rule family="ipv4" source address="10.8.0.8" port port=22 protocol=tcp accept' --permanent$ sudo firewall-cmd --remove-rich-rule 'rule family="ipv4" source address="192.168.1.0/24" port port="11211" protocol="tcp" accept' --permanent`

## Conclusion

You learned the basic concept of firewalld and some common examples for CentOS 8 server. For more info see the official firewalld documentation [here](https://firewalld.org/documentation/).

This entry is   

1. [RHEL 8 FirewallD](https://www.cyberciti.biz/faq/configure-set-up-a-firewall-using-firewalld-on-rhel-8/)
2. CentOS 8 FirewallD
3. [OpenSUSE 15.1 FirewallD](https://www.cyberciti.biz/faq/set-up-a-firewall-using-firewalld-on-opensuse-linux/)





SHARE ON [Facebook](https://www.facebook.com/sharer/sharer.php?u=https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/) [Twitter](https://twitter.com/intent/tweet?text=How+to+set+up+a+firewall+using+FirewallD+on+CentOS+8&url=https://www.cyberciti.biz/faq/how-to-set-up-a-firewall-using-firewalld-on-centos-8/&via=nixcraft)