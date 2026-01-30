# K3S Continuous Deployment

To obtain this, we'll install [Argo CD](https://argo-cd.readthedocs.io/en/stable/).

## Installation

To install it, let's execute the following:

```bash
sudo kubectl create namespace argocd
sudo kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
sudo kubectl patch deployment argocd-server -n argocd --type json -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--insecure"}]'
```

Now lets make an ingress to open Argo CD from our browser in `~/k3s/argocd/argocd-ingress.yaml` with the following content (Changing the example.com for your domain):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  
spec:
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: http
```

And apply it with:

```bash
sudo kubectl apply -f ~/k3s/argocd/argocd-ingress.yaml
```

Now you can enter from the browser to `argocd.example.com` and it should open.

To configure everything, we need to download Argo CD CLI, downloading the latest Argo CD version from [https://github.com/argoproj/argo-cd/releases/latest](https://github.com/argoproj/argo-cd/releases/latest), in our case the linux-arm.

```bash
wget https://github.com/argoproj/argo-cd/releases/download/v3.2.6/argocd-linux-arm64
chmod +x argocd-linux-arm64
sudo mv argocd-linux-arm64 /usr/bin/argocd
echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> ~/.bashrc
source ~/.bashrc
```

## Configuration

To see the default password of the user admin i execute:

```bash
sudo -E argocd admin initial-password -n argocd
```

Now login to the web on `argocd.example.com` with the user admin and the password you just got.

Inside go to User Info -> Update Password and change the password to a new one.

