# okunbound

This repository demonstrates a kubernetes installation of the [unbound](http://www.unbound.net) DNSSEC compliant name resolver using docker, kubectl and helm. It's built as a refactor of a project [kunbound](), but done in a minimal effort way that largely maintains unbound's configuration method (hence the "ok" in the name).  The repo contains a dockerfile and helm chart to assist with building the image (if you don't want to just pull it from my hub account) and installing the helm chart into your cluster.

## Requirements

* [docker](https://www.docker.com/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* [helm](https://helm.sh/)

In addition you'll obviously need a running kubernetes cluster. The yaml and scripts in kunbound were tested with kubectl 1.7.5 and helm 2.5.0 running against a cluster with master and nodes at 1.7.5 running in Google Container Engine. There are no GKE dependencies so this should work anywhere the above tools work.

## Repository structure

```
okunbound/
  image/	- contains Dockerfile and necessary resources for building the okunbound image
  charts/	- the root directory of the helm chart
```

## Building the Docker image

A Dockerfile is included to enable easily building and pushing the Docker image if desired over the already built image.  Change the corresponding value in the helm chart to use your image URL if you do so..

### To install the chart

TODO rewrite

## Update kube-dns to set the upstream

TODO update

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
