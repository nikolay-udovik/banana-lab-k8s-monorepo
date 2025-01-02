# Overview

## My tools

### Kubernetes

I currently cannot use AWS or GCP, and to avoid the complexity of bootstrapping VMs, I decided to keep things as simple as possible. My choice is KinD.

### Automation

#### Scripts

Since I am using KinD for this lab, my deployment consists of a collection of small bash snippets. In my opinion, the best way to handle such a scenario is by using something like a Makefile. I plan to automate the process using **Taskfiles**, which is a modern alternative to Makefiles.

#### Dependencies

* I use NixOS and Devbox to manage dependencies for my projects. While Devbox is recommended for an efficient setup, it is not mandatory.
   If you choose to proceed without Devbox, ensure that you manually fit the dependencies listed in `devbox.json`.
* As we use Kubernetes-in-Docker(KinD), we don't need anything particularly special. Just install `task`, cli tools like `argocd`, `helm` and `argocd`, and `jq`. See `devbox.json`.

### Reprodusability remark

The entire process should be easily reproducible. I can deploy everything from scratch on my Fedora Linux 41, and I believe you will be able to do it on any OS without modifications. All you need to do is install CLI tools like kubectl, helm, argocd, kind, and task. Additionally, note that KinD will bind to ports 80 and 443 on your PC. Ensure this is allowed by your firewall and does not interfere with other applications running on your PC (if you have any).

### Security

Since I want to showcase how to bootstrap a lab locally, I believe security can be overlooked for this demonstration. I won't focus on securing credentials, TLS, or other security measures.

* My git repositories are public.
* My docker hub repo is public(I will use it for task 2)

### Deployed apps

* Infra apps:

   * Ingress Nginx
   * ArgoCD
   * Keda
   * Meterics Server
   * Kube Prometheus Stack

* Workload apps:

   * Podinfo backend
   * Podinfo frontent
   * Podinfo token validator

#### Routes

After the bootstrap is completed, you will be able to access the following routes:

* Podinfo: [http://localhost/](http://localhost/)
* ArgoCD: [http://localhost/argocd](http://localhost/argocd)
* Prometheus: [http://localhost/prometheus](http://localhost/prometheus)
* Grafana: [http://localhost/grafana](http://localhost/grafana)

#### Passwords

ArgoCD and Grafana:

* User `admin`
* Password `Aa123456`

### Repo Structure

1. `devbox.json` - manages dev environment for this project using nixos and devbox.
2. `cluster/`

   * Contains environment-specific configs for managing the cluster.
   * Example: `local` directory stores scripts to automate tasks like provisioning and managing Kubernetes clusters in local labs.

3. `kubernetes/` - main directory for kubernetes manifests.
   * `kubernetes/apps` - holds config files for environment-specific deployments(e.g., helm values and kustomization yamls)
     *  `infra` - holds infra-related app configs(e.g., argocd, keda ...)
     * `workloads` - holds application-layer related apps (e.g., podinfo-back, podinfo-front)
   * `kubernetes/argocd` - ArgoCD manifests for managing the entire system's applications.
     * `infra-apps.yaml` - applies infra apps.
     * `workload-apps.yaml` - applies workload apps.
     * `all-in-one.yaml` - all apps to manage everything!

# Get Started

## Install

Execute the cold-start task. It will handle the entire installation process—from scratch to up and running. What happens under the hood will be explained below.

```sh
task local:cold-start
```

### What is going to happen?

Cold-start is a wrapper for a few other tasks. To inspect all tasks, visit `cluster/local/Taskfile.yaml` or use `task -a` to see all tasks and their descriptions.

#### 1. Create kind cluster

Deploy a KinD cluster named `bring-local` with ports `80` and `443` exposed.

#### 2. Install ingress-nginx

Deploy ingress-nginx and wait until it becomes available.

#### 3. Install argocd

Install ArgoCD and wait until it becomes available. Once ArgoCD is ready, it will initialize the ArgoCD admin's password.

#### 4. Apply apps

All `kubernetes/argocd/local` apps will be applied and picked up by ArgoCD. From this point on, the deployment process depends on ArgoCD and will be executed asynchronously.

What's inside: Everything, from podinfo to infrastructure apps (including nginx and ArgoCD itself).

After the apps are applied, the task `argocd-apps-all-in-one-wait` will be executed. It will exit once all apps are synced and healthy."

## Podinfo Stress Test

### Start

This task will create `pod/podinfo-stress-tester`. The pod will send a high volume of `/echo` requests to the `svc/podinfo-backend`. It is expected that KEDA will begin scaling up the replicas of `podinfo-backend`.

```sh
task local:podinfo-stress-test-start
```

### Stop

This will remove the `pod/podinfo-stress-tester`, and KEDA will downscale the `podinfo-backend` replicas.

```sh
task local:podinfo-stress-test-stop
```

## Destroy

To destroy the lab, simply remove the kind cluster.

```sh
task local:kind-delete
```
