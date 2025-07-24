# okunbound

This repository demonstrates a kubernetes installation of the [unbound](http://www.unbound.net) DNSSEC compliant name resolver using docker, kubectl and helm. It's built as a minor refactor of [kunbound](https://github.com/Markbnj/kunbound), but done in a minimal effort way that largely maintains unbound's configuration method (hence the "ok" in the name).  The repo contains a dockerfile and helm chart to assist with building the image (if you don't want to just pull it from my hub account) and installing the helm chart into your cluster.

## Why the fork?

While the original repo likely still works, I wanted to reduce it's build and configuration complexity.  In particular, moving away from `make` and having to map all of the configuration options from helm values into unbound, especially since `unbound.conf` does not follow the same YAML rules helm does (i.e. it has repeated properties).  I wanted this to be a project I setup once, and hopefully not mess with for a long time (outside of building updated docker images).

## Requirements

* [docker](https://www.docker.com/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* [helm](https://helm.sh/)
* Working Kubernetes cluster

## Repository structure

```
okunbound/
  image/	- contains Dockerfile and necessary resources for building the okunbound image
  charts/	- the root directory of the helm chart
```

## Building the Docker image

A Dockerfile is included to enable easily building and pushing the Docker image if desired over the already built image.  Change the corresponding `image.repository` value in the helm chart to use your image URL if you do so.

### To install the chart

Modify the `values.yaml` as needed for your use case, then deploy using your preferred method for Helm charts (in my case, [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)). The `unboundConfTpl` property is read as a template file in its `ConfigMap` in case you need to lookup other services or IPs to assign DNS addresses.

## Update kube-dns to set the upstream

**Note:** The section below here has been left  unchanged from kunbound, but I would **highly** recommend changing your router's DNS settings instead if you're using unbound for a homelab setup, and you have access to those settings.  You'll need to in order to use unbound for WAN routing anyway, and I'm not certain how current or stable the changes below are (I think the instructions were before CoreDNS, but I haven't looked in depth).  Search for your specific router's admin settings to point to unbound for DNS.  

If you do want to explore changing this and happen to run [Talos](https://talos.dev/) Kubernetes nodes, [this GitHub issue thread](https://github.com/siderolabs/talos/discussions/10012) is probably a good starting point to make those changes part of your Talos YAML files.

---

To get kube-dns to forward to a specific upstream for a private DNS zone we can edit its configmap in the kube-system namespace:

```
apiVersion: v1
data:
 stubDomains: |
 {“DNS_ZONE”: [“RESOLVER_IP”]}
kind: ConfigMap
metadata:
 labels:
 addonmanager.kubernetes.io/mode: EnsureExists
 name: kube-dns
 namespace: kube-system
```

Set the DNS_ZONE to the domain you want forwarded to unbound, and set RESOLVER_IP to the cluster IP address of the kunbound service that was created when the chart was installed. To find this address run `kubectl get svc | grep kunbound`. In order to update the configmap first run `kubectl get configmap kubedns -n kube-system -oyaml` and save the output to a file. Make the edits shown above to add the stubDomains section if it isn't there, and then use `kubectl apply -f file_path` to update the configmap in the cluster.
