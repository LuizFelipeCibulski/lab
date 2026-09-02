# Homelab

A ideia principal desse repositorio é compartilhar e centralizar as configurações de alguns projetos que estou desenvolvendo no meu homelab! 

---

## Cluster

| | |
|---|---|
| **Nodes** | `master` (control-plane) + `worker01` |
| **OS** | Ubuntu 24.04.4 LTS, kernel 6.8 |
| **Kubernetes** | v1.36.2, bootstrapped with `kubeadm` |
| **Runtime** | containerd 2.2.1 |
| **Network** | `10.15.100.0/24` |

## Platform stack

| Component |
|---|
| **Cilium** | 
| **MetalLB** | 
| **Traefik** | 
| **Longhorn** | 
| **MinIO** | 
| **CloudNativePG** |
| **Registry** | 
| **kube-prometheus-stack** | 
| **Argo CD** |

## Notes

* Automatizar a criação do cluster utilizando ansible.
* Organizar pasta de projetos
* Minicraft rodando no cluster?
* Knative e KubeVirt
