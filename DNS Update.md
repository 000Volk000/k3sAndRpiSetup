# DNS Update

For this section I will use the [jameskimmel script](https://github.com/jameskimmel/deSEC_DynDNS) for getting the desired behavior. 
## Installation

To start we need to install some utilities to make the script work as intended, executing:

```bash
sudo apt install dnsutils
```

Then, we'll execute the following commands to install the script:

```bash
curl -o deSEC_DynDNS https://raw.githubusercontent.com/jameskimmel/deSEC_DynDNS/refs/heads/main/deSEC_DynDNS.sh
chmod +x deSEC_DynDNS
sudo mv deSEC_DynDNS /usr/bin
```

Now we need to modify the script `/usr/bin/deSEC_DynDNS` to add the requirements for it to work.

On the line `DOMAIN_NAME='xxxxxxxxxx'` change the x to your domain without the wildcard (example.com) and on `TOKEN='xxxxxxxxxxxxxxxxx'` put the same token we got on the previous step on deSEC.

And thats it, if you execute this on the Raspberry Pi it should start the comprobation and update.

```bash
deSEC_DynDNS
```

But that only updates the non widcard domain, to the changed ip, to make the wildcard change too, we'll duplicate the script changing the `DOMAIN_NAME` line to the wildcard name (* .example.com) 

```bash
sudo cp /usr/bin/deSEC_DynDNS /usr/bin/deSEC_DynDNS2
```

And now change the `DOMAIN_NAME` on /usr/bin/deSEC_DynDNS2

## Automate Updates

Once got both the scripts working we'll create a systemd timer and systemd service to update the DNS automatically.

To do that, we need to create the `/etc/systemd/system/dynDNS.timer` and `/etc/systemd/system/dynDNS.service`, with the following content each one:

- /etc/systemd/system/dynDNS.timer

```bash
[Unit]
Description=Update DNS timer

[Timer]
Persistent=true
# Use a randomly chosen time:
OnCalendar=*-*-* 4:35
# add extra delay, here up to 1 hour:
RandomizedDelaySec=1h

[Install]
WantedBy=timers.target
```

-  /etc/systemd/system/dynDNS.service

```bash
[Unit]
Description=Update DNS service

[Service]
Type=oneshot
ExecStart=/usr/bin/deSEC_DynDNS
ExecStart=/usr/bin/deSEC_DynDNS2
User=example
```

Once created both the files, you should enable and start the timer with:

```bash
sudo systemctl enable dynDNS.timer
sudo systemctl start dynDNS.timer
```

## Next Step
We already did everything with the domain, dns, certificates and that, now lets start with kubernetes -> [K3S](K3S.md)