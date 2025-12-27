# Introduction

I have a Raspberry Pi lying around which is currently not being used anymore, so I figured I could as well use it to experiment a little bit with Kubernetes.
Therefore I've decided to install k3s on it.
In this document, you can find the steps I took to install k3s on the Raspberry Pi.

## Preparing the Raspberry PI

I have prepared a SD card with the Raspberry Pi Imager.  All steps for this can be found on [Tom's hardware](https://www.tomshardware.com/how-to/set-up-raspberry-pi).

Once an OS has been installed on the Raspberry Pi, you should be able to connect to it via ssh:

```
ssh <username>@<ip>
```

To verify what OS the Pi is running, execute this command:

```bash
cat /etc/os-release
```

## Additional configuration

### Assign a static IP

I've decided to give the Raspberry Pi a static IP.  That will make it easier to ssh to it.

To do this, add these lines to the `/etc/dhcpcd.conf` file:

```
interface eth0
static ip_address=<ip>/24
static routers=<your-router-ip>
static domain_name_servers=<your-router-ip> 8.8.8.8
```

or do this via the command-line text tool (recommended):
```
sudo nmtui
```

After you've done that, do not forget to add the IP you've choosen to the DHCP address reservation list in your router. 

The changes only have effect after rebooting the Raspberry Pi.

## Installing K3s

Some prerequisites for installing k3s on a Raspberry Pi can be found [here](https://docs.k3s.io/advanced#raspberry-pi).
If you modify the `/boot/cmdline.txt` file, reboot before installing k3s.

Quickstart guide for installing k3s can be found [here](https://docs.k3s.io/quick-start)

## Install k3s

Execute the command below; this will execute the k3s installation script that is available. 

```
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
```

Customizations via commandline arguments are possible (see documentation).
Note that it is not recommended to use `docker` as the container runtime in k3s, as this can give issues, especially on ARM64 installations.

When a warning is given during installation whic says something like this:
>  Failed to find memory cgroup, you may need to add "cgroup_memory=1 cgroup_enable=memory" to your linux cmdline (/boot/cmdline.txt on a Raspberry Pi)

The installation will probably fail.  These 2 flags must be added to the `cmdline.txt` file.  Afterwards, reboot the raspberry pi.

Once installation has finished, execute this command to verify if k3s is running

```
kubectl get node
```

When k3s is not running, verify the logs:
```
journalctl -u k3s.service
```

## Install pi-hole

See the `deployment.yaml` in the ´./pihole´ folder in this repo.  This deployment installs pihole on a single-node k3s (runs on 1 raspberry pi)

The deployment has a volume-mount to `/srv/pihole`.  This folder must exist before executing the deployment.
Create the folder and assign the correct owner:

```
sudo mkdir /srv/pihole
chown -R 1000:1000 /srv/pihole
```

Since other services might also be running in the k3s cluster that have claimed port 80 already, I have specified a `NodePort` which exposes pihole-web on port `30080`. (See the `deployment.yaml`).

> Before installing the pihole deployment, make sure that the hostname is correctly set in the `nodeAffinity` property of the `persistent volume` definition.  (default value is `raspberrypi`)

Install it by executing the following command:

```
kubectl apply -f deployment.yaml -n pihole
```

Change the password of the pihole webadmin tool by logging in into the container and set the password:

```
kubectl get pods -n pihole
kubectl exec -n pihole -it <podname> -- bash
```

Now, you have a prompt in the container.  Execute this command:

```
pihole setpassword
```

Navigate to

```
http://<raspberry-ip>:30080/admin
```