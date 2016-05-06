# coreos-consul-haproxy
Run your own cloud cluster: CoreOS+Fleet+Consul+Haproxy

# Architecture
CoreOS provides native support to etcd, fleet and docker.
This project adds a layer for easy service discovery, registry and reaching. This is done using Consul for service registry, registrator for discory and haproxy for reaching the services.

# cloud-config explained
The cloud-config file contains all the configuration to add a new node to your cluster.

## Host basic setting
The beginning of the file sets the hostname and ssh key

## Etcd setting
Declare the configuration of the etcd cluster. I prefer to use DNS discovery as it has a more permanent and controlable aspect than `etcd Discovery` while being more maintenable than `static discovery`.

## Units
This declares all the services to run at startup

### etcd2 and fleet
Not much to say, it just starts them at boot.

### data.mount
This mounts a shared NFS data folder. It allows all the user services to find their persistent data in one location: `/data`.

### settimezone.service
This sets the correct time on the server for my area. If you live in GMT you can skip that.

### consul.service
Starts consul service, it share the network with the host.
It also try to connect to other servers declared on the `server.example.com` DNS.
If it cannot reach it, it will try again.

`-advertise ${COREOS_PRIVATE_IPV4}` is used to tell consul which IP address should be advertise for the local services. If you don't use this parameter, you might end up having consul advertise `localhost` or `127.0.0.1` which doesn't make any sense from another node points of view.

### registrator.service
Automatically register/deregister docker services as they come and go.
Use `-e SERVICE_NAME=` and `-e SERVICE_TAGS=` on your `docker run` to tell haproxy how to serve your services. If no `SERVICE_TAGS` is provided, the service will be available on all domains reaching haproxy.

For example, this:
```
docker run --name webserver -p 80 -e SERVICE_80_NAME=www -e SERVICE_TAGS=example_com httpd
```
Will make the web-server reacheable from querying `www.example.com`. The underscore in `SERVICE_TAGS=example_com` is just a hack done for compatibility reason. You can serve the same service on multiple domain by listing them: `SERVICE_TAGS=example_com,example_net`.

### haproxy.service
Uses consul's data to serve every service as a subdomain on port 80 or 443. It uses a provided `consul.tmpl` to allow for multi-domain support.
If you don't want haproxy to act as an SSL termination just remove or comment out `bind *:443 ssl crt /certs/` ([L16 of consul.tmpl](consul.tmpl#L16))
