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

After you've done that, do not forget to add the IP you've choosen to the DHCP address reservation list in your router. 

The changes only have effect after rebooting the Raspberry Pi.

## Installing K3s

Some prerequisites for installing k3s on a Raspberry Pi can be found [here](https://docs.k3s.io/advanced#raspberry-pi).
If you modify the `/boot/cmdline.txt` file, reboot before installing k3s.

Quickstart guide for installing k3s can be found [here](https://docs.k3s.io/quick-start)

## Install k3s

Execute this command:

```
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
```

Once installation has finished, execute this command to verify if k3s is running

```
kubectl get node
```