# coreos-consul-haproxy
Run your own cloud cluster: CoreOS+Fleet+Consul+Haproxy

# Architecture
CoreOS provides native support to etcd, fleet and docker.
This project adds a layer for easy service discovery, registry and reaching. This is done using Consul for service registry, registrator for discory and haproxy for reaching the services.

# Cloud-config explained
If you don't understand the content of the cloud-config file, you can find more information in the projet's wiki page [cloud-config explained](https://github.com/auguster/coreos-consul-haproxy/wiki/cloud-config).
