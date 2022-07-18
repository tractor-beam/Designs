# Prototype
## Prototype Goals
- A NAS based heavily on Podman and containers for every service
- A set of small, high performance services built on rust
- A Podman based service mesh that is much lighter to use than kubernetes with less dependencies. No fancy orchestration, just the basics.
- Deploy the entire NAS as a service mesh
- Distributed mesh between multiple mesh instances even across the internet

## Other Thoughts
#### Kubernetes
Out of scope as this creates quite a heavy footprint and is somewhat overkill for 
- Use k0s as base instead of podman
- Use linkerd on top of k0s to support service mesh and discovery
- Use Traefik as ingress

## Deployment
### OS Setup
1. (Optional) Mirrored ZFS boot volume
2. Install Vanilla Alpine Linux  in `sys` mode, instructions are based on v3.15.
3. Enable the edge community repo and install podman v4 from there.
4. Install `apk add iptables ip6tables iptables-openrc ip6tables-openrc podman podman-openrc`.
5. (Optional) Use ZFS
	1.  `apk add zfs zfs-libs zfs-openrc`
	2. Set xattr storage method to allow large inode. (This along with smb stream_xattr) keep hidden files away. `zfs set xattr=sa dnodesize=auto <pool>`
	3. Change podman storage driver
	4. To do
6. Enable `iptables` and `cgroups` services.
7. Edit `/etc/conf.d/iptables` and `/etc/conf.d/ip6tables`  set `SAVE_ON_STOP="no"`.
8. (Optional) Rootless services
	1. To do
9. Enable start-up services
	1. `rc-update add zfs-import boot`  
	2. `rc-update add zfs-mount boot`  
	3. `rc-update add iptables`
	4. `rc-update add ip6tables`
	5. `rc-update add podman`
10. [*] Setup Samba
11. [*] Setup qemu+libvirt

### Container setup
1. Create a new network in podman using `podman network create <name>` . The default network does not enable DNS on containers and we want to be able to leverage this.
2. Edit `/etc/containers/container.conf` and assign the new network name as the `default_network`
3. Create `config` directory in the home folder to host proxy configuration files.
4. Create `traefik.yaml` in the home folder with the following content. This will enable a traefik as a proxy for http and https traffic on 80 and 443. It will also setup the file and docker provider so containers can be proxied automatically via labels as well as adding static configuration in the config directory.
```
################################################################
#
# Configuration sample for Traefik v2.
#
# For Traefik v1: https://github.com/traefik/traefik/blob/v1.7/traefik.sample.toml
#
################################################################

################################################################
# Global configuration
################################################################
global:
  checkNewVersion: true
  sendAnonymousUsage: false

################################################################
# EntryPoints configuration
################################################################
entryPoints:
  web:
    address: :80
  websecure:
    address: :443

################################################################
# Traefik logs configuration
################################################################
log:
  level: ERROR
#  format: json

################################################################
# Access logs configuration
################################################################
accessLog:
#  format: json

################################################################
# API and dashboard configuration
################################################################
api:
  insecure: true
  dashboard: true

################################################################
# Docker configuration backend
################################################################
providers:
  docker:
#    endpoint: tcp://10.10.10.10:2375
#    defaultRule: Host(`{{ normalize .Name }}.docker.localhost`)
    exposedByDefault: false
  file:
    directory: /service-configs
    watch: true

################################################################
# Observability configuration
################################################################
metrics:
  prometheus:
    entryPoint: web
```

4. Launch services as pods with proxy via file provider
5. Launch services as containers and use labels for automatic configuration
