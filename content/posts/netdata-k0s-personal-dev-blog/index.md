---
title: "Deploying Netdata on My k0s Cluster"
date: 2026-06-14T10:00:00+07:00
draft: false
tags:
  - netdata
  - kubernetes
  - k0s
  - helm
  - traefik
  - cloudflare
categories:
  - devops
author: "Ty Van"
summary: "How I deployed a private Netdata dashboard on my personal k0s cluster using Helm, static local storage, Traefik, and Cloudflare Access."
ShowToc: true
TocOpen: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowCodeCopyButtons: true
---

Date: 2026-06-14  
Status: deployed  
Scope: personal dev notes, sanitized

## Why I Wanted Netdata

I run a small k0s Kubernetes cluster for my own services. The cluster was working, but I wanted better visibility into what was happening inside it.

Before this, I could check pods, services, and logs with `kubectl`, but I did not have a simple dashboard for CPU, memory, containers, Kubernetes state, and alerts.

Netdata looked like a good fit because it is lightweight, easy to install with Helm, and gives useful dashboards without much manual setup.

## My Cluster Setup

The cluster is a small single-node k0s setup.

The main pieces were already in place:

- k0s running Kubernetes
- Traefik as the ingress controller
- MetalLB for exposing Traefik
- Cloudflare Tunnel running in the cluster
- Cloudflare Access for private web apps

The important limitation was storage.

My cluster did not have a dynamic Kubernetes storage provisioner, so applications that needed persistent volumes could not just create PVCs and expect them to bind automatically.

Netdata stores metrics and alarm state, so I wanted persistence. That meant I needed static local PersistentVolumes.

## Netdata Design

The Netdata Helm chart deployed three main parts:

- `netdata-parent`: stores metrics and serves the dashboard
- `netdata-child`: runs on the node and collects system/container metrics
- `netdata-k8s-state`: collects Kubernetes object state

I kept the Netdata service internal:

```yaml
service:
  type: ClusterIP
  port: 19999
```

I also disabled the chart's built-in ingress:

```yaml
ingress:
  enabled: false

httpRoute:
  enabled: false
```

I wanted to control exposure myself through Traefik and Cloudflare Access.

## Static Local Storage

Because there was no dynamic storage class, I created local host directories on the node and used static PVs.

Example paths:

```text
/var/lib/netdata-k8s-parent/db
/var/lib/netdata-k8s-parent/lib
/var/lib/netdata-k8s-state/lib
```

Then I used a custom storage class name:

```yaml
storageclass: netdata-local
```

The parent database used a larger volume, and alarm/runtime data used a smaller one.

In my Helm values, the important parts looked like this:

```yaml
parent:
  database:
    persistence: true
    storageclass: netdata-local
    volumesize: 5Gi
  alarms:
    persistence: true
    storageclass: netdata-local
    volumesize: 1Gi

k8sState:
  persistence:
    enabled: true
    storageclass: netdata-local
    volumesize: 1Gi
```

One thing I learned: when static PVs have similar sizes, it is better to make the binding deterministic. Otherwise Kubernetes can bind a PVC to a PV that technically fits but was meant for something else.

## Installing with Helm

The install command was simple:

```bash
helm upgrade --install netdata netdata/netdata \
  --namespace netdata \
  --create-namespace \
  -f netdata-values.yaml \
  --kubeconfig <K0S_ADMIN_KUBECONFIG>
```

After that, I checked the pods and storage:

```bash
k0s kubectl --kubeconfig <K0S_ADMIN_KUBECONFIG> -n netdata get pods
k0s kubectl --kubeconfig <K0S_ADMIN_KUBECONFIG> -n netdata get svc,pvc
k0s kubectl --kubeconfig <K0S_ADMIN_KUBECONFIG> get pv
```

The result I wanted:

```text
Netdata pods:    Running
Netdata service: ClusterIP
Netdata PVCs:    Bound
```

## Exposing Netdata Safely

I did not want Netdata open to the public internet.

The final access flow was:

```text
Browser
  -> Cloudflare Access
  -> Cloudflare Tunnel
  -> Traefik
  -> Netdata service
```

The public hostname was protected by Cloudflare Access, so only trusted users could log in.

On the Kubernetes side, I created a Traefik `IngressRoute` for Netdata and attached an IP allowlist middleware.

The route looked like this, with sensitive values replaced:

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: netdata-cloudflare-tunnel-only
  namespace: netdata
spec:
  ipAllowList:
    sourceRange:
      - <POD_CIDR>
---
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: netdata-cloudflare
  namespace: netdata
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`<NETDATA_HOSTNAME>`)
      kind: Rule
      middlewares:
        - name: netdata-cloudflare-tunnel-only
      services:
        - name: netdata
          port: 19999
```

This means the Traefik route only accepts traffic from the cluster pod network.

For my setup, this worked because Cloudflare Tunnel was running inside the cluster. Normal public requests to Traefik could not directly reach the Netdata route.

## Why I Used Traefik as the Tunnel Origin

At first, I wanted Cloudflare Tunnel to connect directly to the Kubernetes service:

```text
http://netdata.netdata.svc.cluster.local:19999
```

That would have been cleaner.

But my existing `cloudflared` deployment was using external DNS resolvers, so it could not reliably resolve Kubernetes service names.

Instead of changing too many things at once, I used the existing Traefik LoadBalancer as the tunnel origin:

```text
Cloudflare Tunnel -> http://<PUBLIC_IP> -> Traefik -> Netdata
```

Then I added the Traefik allowlist as a second guardrail.

Later, I may move the tunnel origin directly to the internal Kubernetes service after fixing DNS for the `cloudflared` pod.

## Cloudflare Access

In Cloudflare Zero Trust, I created a self-hosted application:

```text
Application type: Self-hosted
Hostname: <NETDATA_HOSTNAME>
Policy: trusted admins only
```

I did not create a public bypass rule.

The tunnel public hostname pointed to the Traefik origin:

```text
Public hostname: <NETDATA_HOSTNAME>
Service type: HTTP
Service URL: http://<PUBLIC_IP>
```

The user-facing URL still uses HTTPS through Cloudflare:

```text
https://<NETDATA_HOSTNAME>
```

## Verification

I checked the Kubernetes resources:

```bash
k0s kubectl --kubeconfig <K0S_ADMIN_KUBECONFIG> -n netdata get pods,svc,pvc
k0s kubectl --kubeconfig <K0S_ADMIN_KUBECONFIG> -n netdata get ingressroute,middleware
```

I expected:

```text
Pods:          Running
Service:       ClusterIP
PVCs:          Bound
IngressRoute:  Present
Middleware:    Present
```

I also tested the dashboard through the protected hostname:

```text
https://<NETDATA_HOSTNAME>
```

Cloudflare Access asked me to authenticate first. After login, the Netdata dashboard loaded correctly.

## What I Learned

The biggest issue was not Netdata itself. The biggest issue was fitting Netdata into my existing cluster shape.

Main notes:

- `ClusterIP` is a good default for internal dashboards.
- Static PVs are necessary when there is no dynamic storage provisioner.
- Cloudflare Access is a simple way to protect private dashboards.
- Cloudflare Tunnel DNS behavior matters if the tunnel runs inside Kubernetes.
- If using Traefik as an origin, add an allowlist so the route is not openly reachable.
- Never paste real tunnel tokens, kubeconfigs, IPs, or hostnames into public notes.

The final setup gives me a private Netdata dashboard for the k0s cluster without exposing Netdata directly to the public internet.
