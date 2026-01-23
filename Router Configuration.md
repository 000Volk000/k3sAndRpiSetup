# Router Configuration
## Static IP
As mentioned before we'll start making so the DHCP server (Assigns the ips automatically on our local private network) assigns always the same ip address to our Raspberry Pi.

To do that we need the MAC address of our ethernet port, so we shall ssh to the raspberry pi and execute the following.
```bash
ip a
```
This is what are we looking for (The text highlighted), remember it should be on the eth0 interface (Ethernet 0).
<div align="center">
  <img src="assets/router/staticIp/mac.png" alt="mac" width="650">
</div>
Once get, shutdown the Raspberry Pi and open the admin page of your ethernet router (It should come on the label behind the device).

Every ISP is different, you need to find something similar to `Static DHCP - Local Network`, in my case it is on the Advanced mode on Configuration -> LAN.

Once found, you need to place the MAC we got before and assign an IP address of your preference, remember that it should be on your local network (keeping the same network address and only changing the host address)
<div align="center">
  <img src="assets/router/staticIp/dhcp.png" alt="dhcp" width="600">
</div>
When your router reboots you can start your Raspberry Pi and it should get the IP address you just assigned (Remember that now the ssh command changes to the new ip).