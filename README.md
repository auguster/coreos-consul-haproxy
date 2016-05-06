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

## cloud-config-server support
The [ejs/](ejs) folder contains the template file for the [cloud-config-server](https://github.com/auguster/cloud-config-server). The entrypoint is the file [cfch.ejs](ejs/cfch.ejs).

### Usage
Launch cloud-config-server:
```
docker run --rm -it --name cloud -v $PWD/ejs:/data:ro -p 8080:8080 auguster/cloud-config-server
```
Then query the cloud-config file:
```
http://localhost:8080/cfch
```

### Parameters
List of available parameters
 - `host` sets the hostname of the instance (default to `host`)
 - `domain` sets the domain of the instance, used for etcd DNS discovery (default to empty)
 - `dataserver` sets the NFS server's address (data[.domain])
 - `timezone` sets the timezone for the instance (default to `Europe/Paris`)
 - `consul` sets the URL of a consul server (default to domain). Overwritten by `bootstrap`.
 - `bootstrap` puts the instance in bootstrap mode. Choose the number of expected cluster member for bootstrap. Etcd might be broken while in bootstrap mode (not fully tested).  