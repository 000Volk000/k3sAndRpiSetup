# K3S Set Up

We are going to download and set up [K3S](https://k3s.io/) on our Raspberry Pi and the wildcard certificate we already got and set to auto renew.

Why K3S? Because its a distribution of Kubernetes (K8S) that aims to be lightweight, optimized for ARM (Raspberry Pi's architecture) and without loosing any of the K8S advantages.

## Installation

To start off we need to enable `cgroups`, to do that we'll need to append `cgroup_memory=1 cgroup_enable=memory` to `/boot/firmware/cmdline.txt`.

Be careful not to put any extra space or symbol, once done that and rebooted we can continue.

Lets download it with the install script with:

```bash
curl -sfL https://get.k3s.io | sh -s - server --secrets-encryption
```

And that is really it, the script should install and configure everything so we can use our k3s cluster and it starting again when rebooted because of a systemd service.

## Container registries authentication

As we told before, we want auto deployment, in my case I want to make a github workflow that publish to ghcr.io the container i want to deploy.

We sometimes want to make private registries and to deploy from them we need to authenticate into ghcr.io with our username and a PAT.

To get our PAT we'll need to log in [github](https://github.com/) -> click into your user profile picture -> Settings -> Developer Settings -> Personal Access Tokens -> Tokens (classic) -> Generate new token (Classic) -> Select only the read:packages -> Generate token

<div align="center">
  <img src="assets/k3s/registries/githubPAT.png" alt="githubPAT" width="600">
</div>

So, lets create the `/etc/rancher/k3s/registries.yaml` and fill it with the following lines, where password is the github PAT we just got:

```yaml
configs:
  "ghcr.io":
    auth:
      username: "xxxxxxxxxxxx"
      password: "xxxxxxxxxxxx"
```

And reboot the k3s service:

```bash
sudo systemctl restart k3s
```

## SSL/TLS certificate

Lastly on this section lets make that traefik use our ssl/tls certificate for all the incoming communications, to do that we'll follow some steps.

We'll create a kubernetes tls secret with:

```bash
sudo kubectl create secret tls tls-cert \
  --cert=path/to/cert/file \
  --key=path/to/key/file
  -n kube-system
```

Once done, lets create a file on `~/k3s/traefik/traefik-config.yaml` where we'll configure traefic to work with the created secret.

```bash
mkdir -p ~/k3s/traefik
vim.tiny ~/k3s/traefik/traefik-config.yaml
```

Inside the file, lets put:

```yaml
apiVersion: traefik.io/v1alpha1
kind: TLSStore
metadata:
  name: default
  namespace: kube-system

spec:
  defaultCertificate:
    secretName:  tls-cert
```

Lastly apply it with

```bash
sudo kubectl apply -f ~/k3s/traefik/traefik-config.yaml
```

## Next Step

All the k3s initial installation and configuration is done, the next thing we are going to do is to configure the auto-detect and release of packages when a change is made on github -> [K3S Continuous Deployment](K3S%20Continuous%20Deployment.md)