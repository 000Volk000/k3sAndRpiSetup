# K3S and Raspberry Pi 5 setup
## Introduction
This is a document of my process setting up from scratch k3s and all the needs to have a working kubernetes station running on a Raspberry Pi with automatized updates for your apps directly from a git push.

>[!IMPORTANT]
>This is my first kubernetes project.<br>Some things done might not be the best industry practice. 
## Goal
The final distribution of this kubernetes setup will be a:
- K3S for the kubernetes backend.
- Raspberry Pi 5 as the master (and only) node.<br>(Connected directly to a generic ISP router via ethernet connection with a DHCP reserved IP address)
- Wildcard domain being handled by [deSEC](https://desec.io) as the DNS registry.
- Automatic, no-charge, SSL/TLS certificate renewal.
- Proxy handling the domain.
- Automatic deployment based on git.
## Materials Used
This are the exact materials i made this setup work, they can vary in your case:
- 1 Raspberry Pi 5 (8 GB Ram)
- 1 Raspberry Pi Active Cooler
- 1 Pimoroni M.2 Base for RPI5
- 1 Raspberry Pi Power Supply
- 1 Geeekpi 4 Layers Acrylic Case
- 1 Ethernet Cable
- 1 Kingston microSD (CANVAS Select Plus)
- 1 Team Group NVMe SSD (MP33 PRO)
## Index
- [Set Up](Set%20Up.md)
	- [Raspberry Pi Hardware](Raspberry%20Pi%20Hardware.md)
	- [Raspberry Pi Software](Raspberry%20Pi%20Software.md)
	- [Router Configuration](Router%20Configuration.md)
	- [DNS Configuration](DNS%20Configuration.md)
- [DDNS Instalation](DDNS%20Instalation.md)
	- [SSL-TLS Certificate](SSL-TLS%20Certificate.md)
	- [DNS Update](DNS%20Update.md)
- [K3S](K3S.md)
	- [K3S Set Up](K3S%20Set%20Up.md)
	- [K3S Continuous Deployment](K3S%20Continuous%20Deployment.md)
- [App Deployment](App%20Deployment.md)
	- [Github Workflow](Github%20Workflow.md)
	- [Argo Workflow](Argo%20Workflow.md)
## License
Created under the MIT License. See [LICENSE](https://github.com/000Volk000/k3sAndRpiSetup/blob/main/LICENSE) for more information.

Created by [Darío Martínez Kostyuk](https://linktree.volkhost.es/) - 2026
