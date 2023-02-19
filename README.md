<div align="center">

<img src="https://camo.githubusercontent.com/5b298bf6b0596795602bd771c5bddbb963e83e0f/68747470733a2f2f692e696d6775722e636f6d2f7031527a586a512e706e67" align="center" width="144px" height="144px"/>

### My home operations repository!

_... managed with Flux, Renovate and GitHub Actions_

</div>

---

## 📖 Overview

This is a mono repository for my home infrastructure and Kubernetes cluster. I try to adhere to Infrastructure as Code (IaC) and GitOps practices using the tools like [Ansible](https://www.ansible.com/), [Terraform](https://www.terraform.io/), [Kubernetes](https://kubernetes.io/), [Flux](https://github.com/fluxcd/flux2), [Renovate](https://github.com/renovatebot/renovate) and [GitHub Actions](https://github.com/features/actions).

---

## Kubernetes

This all started from a template over at [onedr0p/flux-cluster-template](https://github.com/onedr0p/flux-cluster-template). Big thanks to all the folks who put in the work to build that template!

### Installation

My cluster is [k3s](https://k3s.io/) provisioned overtop bare-metal Ubuntu Server using the [Ansible](https://www.ansible.com/) galaxy role [ansible-role-k3s](https://github.com/PyratLabs/ansible-role-k3s). This is currently a hyper-converged cluster, workloads and NFS file storage are sharing the same available resources on my nodes.

🔸 _[Click here](./ansible/) to see my Ansible playbooks and roles._

### Core Components

- [cert-manager](https://cert-manager.io/docs/): Creates SSL certificates for services in my Kubernetes cluster.
- [external-secrets](https://github.com/external-secrets/external-secrets/): Managed Kubernetes secrets using [1Password Connect](https://github.com/1Password/connect).
- [ingress-nginx](https://github.com/kubernetes/ingress-nginx/): Ingress controller to expose HTTP traffic to pods over DNS.
- [sops](https://toolkit.fluxcd.io/guides/mozilla-sops/): Managed secrets for Kubernetes, Ansible and Terraform which are commited to Git.
- [cloudflare-operator](https://github.com/adyanth/cloudflare-operator): Creates and manages Cloudflare Tunnels and DNS records for service resources.

### GitOps

[Flux](https://github.com/fluxcd/flux2) watches my [kubernetes](./kubernetes/) folder (see Directories below) and makes the changes to my cluster based on the YAML manifests.

The way Flux works for me here is it will recursively search the [kubernetes/apps](./kubernetes/apps) folder until it finds the most top level `kustomization.yaml` per directory and then apply all the resources listed in it. That aforementioned `kustomization.yaml` will generally only have a namespace resource and one or many Flux kustomizations. Those Flux kustomizations will generally have a `HelmRelease` or other resources related to the application underneath it which will be applied.

[Renovate](https://github.com/renovatebot/renovate) watches my **entire** repository (except for the archive) looking for dependency updates, when they are found a PR is automatically created. When some PRs are merged [Flux](https://github.com/fluxcd/flux2) applies the changes to my cluster, triggered via a GitHub webhook.

### Directories

This Git repository contains the following directories under [kubernetes](./kubernetes/).

```sh
📁 kubernetes      # Kubernetes cluster defined as code
├─📁 bootstrap     # Flux installation
├─📁 flux          # Main Flux configuration of repository
└─📁 apps          # Apps deployed into my cluster grouped by namespace (see below)
```


### Cluster layout

Below is a a high level look at the layout of how my directory structure with Flux works. In this brief example you are able to see that `pihole` will not be able to run until `nfs-subdir-external-provisioner`, `external-secrets-stores` and  `ingress-nginx` are running. It also shows that the `ClusterSecretStore` custom resource depends on the `external-secrets` Helm chart. This is needed because `external-secrets` installs the `ClusterSecretStore` custom resource definition in the Helm chart.

```python
# Key: <kind> :: <metadata.name>
GitRepository :: home-ops-kubernetes
    Kustomization :: cluster
        Kustomization :: cluster-apps
            Kustomization :: cluster-apps-pihole
                DependsOn:
                    Kustomization :: cluster-apps-nfs-subdir-external-provisioner
                    Kustomization :: cluster-apps-external-secrets-stores
                    Kustomization :: cluster-apps-ingress-nginx
                HelmRelease :: pihole
            Kustomization :: cluster-apps-nfs-subdir-external-provisioner
                HelmRelease :: nfs-subdir-external-provisioner
            Kustomization :: cluster-apps-external-secrets-stores
                DependsOn:
                    Kustomization :: cluster-apps-external-secrets
                ClusterSecretStore :: onepassword-connect
            Kustomization ::  cluster-apps-external-secrets
                HelmRelease :: externalSecrets
            Kustomization ::  cluster-apps-ingress-nginx
                HelmRelease :: ingress-nginx
```

### Networking

Networking is hard. Still figuring it out. I have an Eero Pro 6E mesh system with two nodes. Each kubernetes node is attached to a desktop switch, which is in turn attached to the 2.5 Gpbs port of an Eero node.


[Flannel](https://github.com/flannel-io/flannel) is being used as the CNI. Load balancers are provided by [metallb](https://metallb.universe.tf/). Ingresses are provided by [ingress-nginx](https://kubernetes.github.io/ingress-nginx/), but are only used for internal traffic -- [cloudflare-operator](https://github.com/adyanth/cloudflare-operator) routes traffic directly to the service.

---

## ☁️ Cloud Dependencies

While most of my infrastructure and workloads are selfhosted I do rely upon the cloud for certain key parts of my setup. This saves me from having to worry about two things. (1) Dealing with chicken/egg scenarios and (2) services I critically need whether my cluster is online or not.

| Service                                     | Use                                                            | Cost                               |
| ------------------------------------------- | -------------------------------------------------------------- | ---------------------------------- |
| [1Password](https://1password.com/)         | Secrets with [External Secrets](https://external-secrets.io/)  | ~$65/yr                            |
| [Cloudflare](https://www.cloudflare.com/)   | Domain, DNS and proxy management                               | ~$0/yr                             |
| [GitHub](https://github.com/)               | Hosting this repository and continuous integration/deployments | Free                               |
| [Newshosting](https://www.newshosting.com/) | Usenet access                                                  | ~$150/yr                           |
| [PrivadoVPN](https://privadovpn.com/)       | VPN                                                            | Free (incl. with Newshosting)      |
| [NZBGeek](https://nzbgeek.info)             | Usenet indexer                                                 | ~$12/yr                            |
| [Pushover](https://pushover.net/)           | Kubernetes Alerts and application notifications                | Free ($5 one-time for iOS license) |
|                                             |                                                                | Total: ~$19/mo                     |

---

## 🌐 DNS

### Ingress from the Outside World

Over WAN, I use [Cloudflare](https://www.cloudflare.com/) tunnels, so that I do not need to expose my local cluster directly.
[cloudflare-operator](https://github.com/adyanth/cloudflare-operator) is used to create the tunnels and Cloudflare DNS entries for services I wish to expose to the public.

[Cloudflare](https://www.cloudflare.com/) works as a proxy to hide my homes WAN IP and also as a firewall. When not on my home network, all the traffic coming into my cluster is via a Cloudflare tunnel.

### Internal DNS

[Pi-hole](https://pi-hole.net/) is deployed on the cluster, with a dedicated IP address from the MetalLB-managed range, listening on `:53`. All DNS queries for _**my**_ domains are forwarded to [k8s_gateway](https://github.com/ori-edge/k8s_gateway) that is running in my cluster -- Pi-hole has the k8s_gateway IP listed as the upstream server. With this setup `k8s_gateway` has direct access to my clusters ingresses and services and serves DNS for them in my internal network.

### Ad Blocking

Pi-hole also provides ad-blocking.

---

## 🔧 Hardware

| Device                             | Count | OS Disk Size | Data Disk Size       | Ram  | Operating System | Purpose                                 |
| ---------------------------------- | ----- | ------------ | -------------------- | ---- | ---------------- | --------------------------------------- |
| Libre Computer Board ROC-RK3328-CC | 3     | 32GB SSD     | 1TB HDD (nfs)        | 4GB  | Ubuntu           | Kubernetes Masters/Workers, NFS Servers |
| Hyper-V                            | 1     | 128GB NVMe   | -                    | 4GB  | Ubuntu           | Kubernetes Workers                      |
| eero Pro 6e                        | 2     | -            | -                    | -    | -                | Mesh Router                             |

---

### 🤖 Renovatebot

[Renovatebot](https://www.mend.io/free-developer-tools/renovate/) will scan my repository and offer PRs when it finds dependencies out of date. Common dependencies it will discover and update are Flux, Ansible Galaxy Roles, Terraform Providers, Kubernetes Helm Charts, Kubernetes Container Images, Pre-commit hooks updates, and more!

The base Renovate configuration provided in my repository can be view at [.github/renovate.json5](https://github.com/onedr0p/flux-cluster-template/blob/main/.github/renovate.json5).

## 🤝 Thanks

Big shout out to all the authors and contributors to the projects that I are using in this repository.

[@onedr0p](https://github.com/onedr0p) created [this template](https://github.com/onedr0p/flux-cluster-template) on which I based my cluster off of!
