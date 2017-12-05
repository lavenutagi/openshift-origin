# Setup OpenShift Origin on RHEL 7 / CentOS 7

### Reference guide:

* [Local Cluster Management]

### Preconditions:

* RHEL 7 or CentOS 7 on bare metal or VM
* Docker CE installed
* Check that ``sysctl net.ipv4.ip_forward`` is set to ``1``

## Step 1: Download OpenShift Origin

* Download ``openshift-origin-client-tools-VERSION-linux-64bit.tar.gz`` from https://github.com/openshift/origin/releases
* Extract OC and move it to ``/opt/rh`` (add to ``PATH``)

## Step 2: Configure the Docker daemon

> Configure the Docker daemon with an insecure registry parameter of ``172.30.0.0/16``

Edit the ``/etc/docker/daemon.json`` file and add the following:

```json
{
   "insecure-registries": [
      "172.30.0.0/16"
   ]
}
```

Reload ``systemd`` and restart the Docker daemon:

```sh
$ sudo systemctl daemon-reload

$ sudo systemctl restart docker
```

## Step 3: Ensure that your firewall allows containers access to the OpenShift endpoints

Determine the Docker bridge network container subnet:

```sh
$ docker network inspect -f "{{range .IPAM.Config }}{{ .Subnet }}{{end}}" bridge
```

> You should get a subnet like: ``172.17.0.0/16``

Create a new firewalld zone for the subnet and grant it access to the API and DNS ports:

```sh
$ firewall-cmd --permanent --new-zone dockerc

$ firewall-cmd --permanent --zone dockerc --add-source 172.17.0.0/16

$ firewall-cmd --permanent --zone dockerc --add-port 8443/tcp

$ firewall-cmd --permanent --zone dockerc --add-port 53/udp

$ firewall-cmd --permanent --zone dockerc --add-port 8053/udp

$ firewall-cmd --reload
```

## Step 4: Start OpenShift Cluster

```sh
$ oc cluster up
```

[Local Cluster Management]: <https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md>