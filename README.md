# coreos-consul-haproxy
Run your own cloud cluster: CoreOS+Fleet+Consul+Haproxy

## Architecture
CoreOS provides native support to etcd, fleet and docker.
This project adds a layer for easy service discovery, registry and reaching. This is done using Consul for service registry, registrator for discory and haproxy for reaching the services.

## Cloud-config explained
If you don't understand the content of the cloud-config file, you can find more information in the projet's wiki page [cloud-config explained](https://github.com/auguster/coreos-consul-haproxy/wiki/cloud-config-explained).

## Deploy your service
In order to deploy your services, a [blank.service](blank.service) is provided. You can customize it however you like.
It contains commented out section for Fleet to show you what you can do with it.

### Two unit service
If you have a meta-service that has a web-server and a database, the best practice is to have the web-server act as a "sidekick" to the database.

For example, the database is simply declared as `website_db.service`:
```
[Unit]
Description=Database for the website
Requires=docker.service
After=docker.service
RequiresMountsFor=/data
```
The website (`website_www.service`) however depends upon the database:
```
[Unit]
Description=Website
Requires=docker.service
After=docker.service website_db.service
PartOf=website_db.service
RequiresMountsFor=/data

...

[X-Fleet]
MachineOf=website_db.service
```
In this setting the website will follow the database wherever it goes in the cluster.
