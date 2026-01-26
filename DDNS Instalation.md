# DDNS Instalation
A DDNS or Dynamic DNS is a DNS but with the difference that its value can change over time to be the public ip address allways, even behind NAT.

We'll do 2 main things on this section:
1. Set up the SSL/TLS certificate so we can make use of HTTPS ([SSL-TLS Certificate](SSL-TLS%20Certificate.md))
2. Make our Raspberry Pi change the ip address of the domain when detects that our true public ip has changed ([DNS Update](DNS%20Update.md))