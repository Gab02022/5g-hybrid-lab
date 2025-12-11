# 5g-edge-lab 📡

**GitOps-driven 5G Standalone (SA) Core & RAN on Edge Hardware.**

This repository contains the Infrastructure-as-Code (IaC) to deploy a fully functional 5G network on highly constrained legacy edge hardware. Orchestrated with **K3s** and **ArgoCD**, utilizing **Open5GS** for the Core and **UERANSIM** for Radio Access Network simulation.

## 🎯 Project Objective (Thesis)
To demonstrate the technical viability of deploying next-generation telecom microservices (5G SA) on edge computing infrastructure with severe resource constraints (Legacy CPU without AVX support and <4GB RAM), managed via GitOps pipelines.

## 🏗️ Architecture & Hardware

The cluster is deployed on a Single-Node architecture optimized for "bare-metal" performance.

### Edge Node Specifications
| Component | Specification | Constraints / Notes |
| :--- | :--- | :--- |
| **Device Type** | Legacy Edge Gateway | Industrial form factor / Thin Client. |
| **CPU** | **AMD G-T56N** @ 1.65GHz | Bobcat Architecture (Dual Core). <br>⚠️ **Non-AVX Instruction Set:** Required a custom implementation of MongoDB 4.4 as v5.0+ is incompatible with the CPU instruction set. |
| **RAM** | 4GB DDR3 (3.3GB Usable) | Aggressive memory optimization (4G/EPC components disabled). |
| **OS** | Ubuntu 20.04 LTS | Kernel 5.4+. |
| **Orchestrator** | K3s (Lightweight K8s) | CNI: Flannel. |

---

## 🚀 Quick Start Guide

### 1. Prerequisites
* A running Kubernetes cluster (K3s recommended for edge).
* `kubectl` configured.
* [Optional] `argocd` CLI.

### 2. Clone Repository
* If the GitOps controller is not yet installed:
```bash
git clone [https://github.com/Gab02022/5g-edge-lab.git](https://github.com/Gab02022/5g-edge-lab.git)
cd 5g-edge-lab
```
### 3. Install ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
```
### 4. Deploy 5G Network (The GitOps Way)
*Instead of manual Helm installs, we let ArgoCD sync the desired state from this repository.*
*Via Web UI*
1. Create New App.
2. Repo URL: https://github.com/Gab02022/5g-edge-lab.git
3. Path: charts/open5gs
4. Value File: values.yaml
### 5. 🧪 Validation & Testing
*1. Connectivity Check (Ping)*
Verify that the User Equipment (UE) has internet access through the GTP tunnel (Interface uesimtun0).
```bash
# Get the UE pod name
UE_POD=$(kubectl get pod -n ueransim -l app.kubernetes.io/name=ueransim-ues -o jsonpath="{.items[0].metadata.name}")

# Execute Ping from within the UE
kubectl exec -it -n open5gs $UE_POD -- ping -I uesimtun0 8.8.8.8
```
*2. Throughput Test*
Speed test performed using iperf3 or direct download inside the container:
```bash
kubectl exec -it -n open5gs $UE_POD -- curl --interface uesimtun0 -o /dev/null [http://speedtest.tele2.net/100MB.zip](http://speedtest.tele2.net/100MB.zip)
```

### 📸 Evidence & Screenshots
*1. ArgoCD Dashboard (Network State)*
Showing "Healthy" and "Synced" status for Core and RAN microservices.

*2. Traffic Analysis (Wireshark)*
Packet capture demonstrating GTP Encapsulation. ICMP/TCP traffic is observed traveling inside the GTP-U tunnel (Port 2152).

*3. Terminal (Ping Success)*
Confirmation of latency and PDU Session establishment.

### 📜 License
This project is part of academic research. Based on open-source charts by Gradiant.
