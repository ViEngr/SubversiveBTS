Persistent IPtables on Raspberry Pi (Raspbian)

This article is not about building proper iptable rules but on how to make iptable configurations to load on every reboot.

I have been trying to find a consistent and easy solution to implement iptables on Raspberry Pi (Raspbian-wheezy), the way Debian and Raspbian works does not provide a way to load iptables on every boot, it needs to be added manually as a script to load on start-up. There are ways to make Raspbian work without the instructions below, although the following - I think - are very simple and elegant.

There are some steps to take before we begin, I am going to assume you have a Rasbian Pi up and running with the following commands executed:

# rpi-update 

# apt-get dist-upgrade 

# apt-get update
Step 1: Install iptables-persistent package with apt-get command.

# apt-get install iptables-persistent
On the menu, select Yes on the rule.v4 file. The second choice is about rule.v6 and IPv6 support, choose based on your needs.

Step 2: After the installation is done, go to:

[Replace vim with your favourite editor]

# vim /etc/iptables/rules.v4
Now you can see the existing iptables configuration, in my case since no rules are setup yet, it is completely empty:

# Generated by iptables-save v1.4.14 on Fri Dec 26 20:13:04 2014 

*filter 

:INPUT ACCEPT [5897:7430402] 

:FORWARD ACCEPT [0:0] 

:OUTPUT ACCEPT [1767:169364] 

COMMIT 

# Completed on Fri Dec 26 20:13:04 2014
Now you can start building your iptables on this file, one per line, just before the COMMIT command. Once you are done, save the file.

I would suggest to add at least the following rule, in order to validate our concept.

-A INPUT -p icmp -m icmp –icmp-type 8 -j REJECT
The above rule will filter inbound ICMP type 8 traffic and will respond with a
Destination port unreachable message and will take effect after you have rebooted the Pi.

Step 3: Feel free to do a ping to the device, it should respond normally. Now reboot the device.

# reboot
After the device is back on, do a ping request again. This time you should get the “Destination port unreachable” message. The iptables have loaded successfully, congratulations. Now, issue:

# iptables -L 

Chain INPUT (policy ACCEPT) 

target prot opt source destination 

REJECT icmp – anywhere anywhere icmp echo-request reject-with icmp-port-unreachable 

Chain FORWARD (policy ACCEPT) 

target prot opt source destination 

Chain OUTPUT (policy ACCEPT) 

target prot opt source destination
Our rule is on the third line. Now feel free to add the rest of the rules per Step 2.

Extra tip: In case you prefer adding the rules straight to iptables and not to the file, the following command may be useful:

# /etc/init.d/iptables-persistent save
This command takes the current configuration of your iptables and saves it to the rules.v4 file.

--------

Allow IP Forwarding for GPRS data
By default, IP Forwarding is disabled on any modern Linux distro (file /proc/sys/net/ipv4/ip_forward contains a 0)

To tell your kernel that IP forwarding is allowed on your system, change the 0 (false) to 1 (true) by typing:

echo 1 > /proc/sys/net/ipv4/ip_forward

To preserve these changes after reboot edit the line that says net.ipv4.ip_forward = 0 to net.ipv4.ip_forward = 1 from /etc/sysctl.conf file.
To see if forwarding is enabled or not, you can query the sysctl kernel value net.ipv4.ip_forward
> sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1


Add iptables rules
Install iptables
urpmi iptables
Start iptables service
systemctl start iptables.service
Tell iptables to forward packets from your internal network

iptables -A POSTROUTING -t nat -s 192.168.99.0/24 ! -d 192.168.99.0/24 -j MASQUERADE

Preserve iptables rules after reboot
iptables-save > /etc/sysconfig/iptables
