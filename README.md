# ansible-role-k8s-worker
Ansible role for Kubernetes worker nodes

- [ansible-role-k8s-worker](#ansible-role-k8s-worker)
- [Overview](#overview)
- [Configuration](#configuration)
  * [Versions](#versions)
  * [PKI](#pki)
  * [Kubernetes](#kubernetes)
- [Installation](#installation)

# Overview

This is an ansible role for configuring a worker node within the Kubernetes container orchestration platform. It can be used on it's own, but it is recommended that this role is used in conjunction with the associated [master node role](https://github.com/adammillerio/ansible-role-k8s-master). This role supports being ran against multiple worker nodes at a time.

This playbook performs the following steps:

* Creates the kubernetes configuration directories (dirs.yml)
* Installs the CloudFlare PKI and TLS toolkit (cfssl.yml)
* Generates the required Public Key Infrastructure (pki.yml)
    * Certificate Authority (ca.pem, ca-key.pem)
    * Client certificate for each worker node ($HOSTNAME.pem $HOSTNAME-key.pem)
    * Kubernetes proxy certificate (kube-proxy.pem, kube-proxy-key.pem)
* Installs and configures the Docker Container Runtime Interface (docker.yml)
* Installs and configures the Kubernetes base daemons (k8s-base.yml)
    * Container Networking Interface (CNI) plugins
    * Kubernetes client (kubectl)
    * Kubernetes agent (kubelet)
    * Kubernetes proxy (kube-proxy)

There are a few major configuration choices of note in this cluster configuration:
* Targets Debian 9.0 "stretch"
* Uses systemd units for the `kubelet` and `docker` runtimes
* Uses Kubernetes [Static Pods](https://kubernetes.io/docs/tasks/administer-cluster/static-pod/) for all other configured daemons that are managed via the `kubelet`
* If used with the corresponding master role, uses the [flannel](https://github.com/coreos/flannel) Container Networking Interface to implement the [Kubernetes Networking Model](https://kubernetes.io/docs/concepts/cluster-administration/networking/) which allows for bare-metal deployments

# Configuration
The following variables are utilized within the role. Sensible defaults have been provided for each:

## Versions

| Name  | Default  | Description  |
|---|---|---|
| cfssl_version | 1.2 | Version of the CloudFlare PKI and TLS toolkit to install |
| k8s_version | 1.9.0 | Version of the Kubernetes components to install |
| docker_version | 17.03.2 | Version of the Docker CRI to install |
| docker_edition | ce | Edition of the Docker CRI to install |
| cni_plugins_version | 0.6.0 | Version of the CNI plugins to install |

## PKI
| Name  | Default  | Description  |
|---|---|---|
| pki_expiry | 8760h | Time period before the generated Certificate Authority certificate expires |
| pki_algorithm | rsa | Algorithm used to generate the certificates |
| pki_key_size | 2048 | Size of the generated certificates in bits |
| pki_country | US | Country of the generated certificates |
| pki_locality | Annandale | Locality of the generated certificates |
| pki_state | VA | State of the generated certificates |

## Kubernetes
| Name  | Default  | Description  |
|---|---|---|
| k8s_apiserver_public_host | api.k8s.local | Public hostname of the K8S API Server, to determine where nodes connect to register |
| k8s_apiserver_internal_host | api-internal.k8s.local | Internal hostname of the K8S API Server, used to determine where nodes connect to register |
| k8s_cni_cluster_cidr | 100.64.0.0/10 | CIDR range to be used within the cluster |
| k8s_cni_node_cidr | 100.96.0.0/11 | CIDR range to be assigned to nodes within the cluster |
| k8s_cni_cluster_dns_host | 100.64.0.10 | In-cluster IP address to be used by the Kubernetes DNS addon |
| k8s_cni_cluster_dns_domain | cluster.local | Domain to be used by the Kubernetes DNS addon |

Other variables are defined in `defaults/main.yml` but should only be overriden in special circumstances.

# Installation
Before using this role, ensure the following are available/configured on the host:
* ansible (Tested on 2.4.3.0)
* CloudFlare SSL toolkit (Tested on 1.2)
* User running the playbook has access to the `files` directory, as keys will be generated on the host and placed there

Then, simply add the `k8s-worker` role to a playbook