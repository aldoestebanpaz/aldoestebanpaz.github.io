# Nmap cheatsheet

## Host discovery

The following two commands allows to find active IPs in a network.

## List scan

Simply lists each host of the network(s) specified, without sending any packets to the target hosts. By default, Nmap still does reverse-DNS resolution on the hosts to learn their names.

Since the idea is to simply print a list of target hosts, options for higher level functionality such as port scanning, OS detection, or ping scanning cannot be combined with this.

Example: `sudo nmap -sn 192.168.100.0/24`.

## No port scan - AKA. Ping scan

The `sudo nmap -sn {NETWORK}` command (in previous releases of Nmap, -sn was known as -sP) tells Nmap not to do a port scan after host discovery, and only print out the available hosts that responded to the scan.

Example: `sudo nmap -sP 192.168.100.0/24`.
