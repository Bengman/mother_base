
# Secure networking

Network should always be considered hostile. This is especially true for laptop users who need to connect to
the Internet via public WiFi hotspots, e.g. at the airport or in the hotel. Today networking software is very
complex - it comprises the network card drivers (e.g. WiFi drivers), the various protocol stacks (e.g. the
802.11 very complex stack), the TCP/IP stack, the firewalling software, and often applications such as DHCP
clients, etc. Most of this software runs with kernel or system privileges on typical OS like Windows or Linux,
which means that the attacker who exploited a bug e.g. in the WiFi driver or stack can compromise the whole
system, including all user data and applications.

Itʼs an obvious attack surface and Qubes OS architecture aims at eliminating it, by moving all the world-
facing networking code into a dedicated network domain. Another reason to move the networking code out of
Dom0 is to prevent potential attacks originating from other VMs and targeting potential bugs in Xen network
backends, or Dom0 minimal TCP/IP stack.

