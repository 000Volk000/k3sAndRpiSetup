# K3S Installation

We are going to download and set up [K3S](https://k3s.io/) on our Raspberry Pi and the wildcard certificate we already got and set to auto renew.

Why K3S? Because its a distribution of Kubernetes (K8S) that aims to be lightweight, optimized for ARM (Raspberry Pi's architecture) and without loosing any of the K8S advantages.

## Set Up

To start off we need to enable `cgroups`, to do that we'll need to append `cgroup_memory=1 cgroup_enable=memory` to `/boot/firmware/cmdline.txt`.

Be careful not to put any extra space or symbol, once done that and rebooted we can continue.

Lets download it with the install script with:

```bash
curl -sfL https://get.k3s.io | sh -
```

And that is really it, the script should install and configure everything so we can use our k3s cluster and it starting again when rebooted.