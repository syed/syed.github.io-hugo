+++
title = "Creating an LXC container from a tarball"
date = "2015-05-06"
+++

I recently wanted to create LXC containers on multiple hosts using a standard
rootfs. My idea was to install all my apps on one container, `tar` it up and
use it everywhere. As I was working on implementing this, I found that LXC does
not provide an option to create  a container using a tarball as the rootfs.

After a bit of googling I found
[this](https://raw.githubusercontent.com/saltstack/salt/develop/salt/templates/lxc/salt_tarball)
script from [saltstack](http://saltstack.com/) which lets you create a
container from a tarball. Here is the whole process

Assuming the container that you want to set as the templete is called
`base-container`. Start the container and install all the apps that your require. 
shutdown the container after all depependeinces are installed. 

```bash
$ lxc-stop -n base-container
```

By default, the rootfs for this container will be located
at `/var/lib/lxc/base-conainer/rootfs`. Create the tar using the following 
command 

```bash
$ cd /var/lib/lxc/base-container/
$ tar -cvzf template.tar.gz rootfs/
```

To use `templete.tar.gz` as the template for creating new containers, download
the `salt_tarball` script from
[here](https://raw.githubusercontent.com/saltstack/salt/develop/salt/templates/lxc/salt_tarball).
Run the lxc-create with this script as the template

```bash 
$ lxc-create -n newcontainer -t salt_tarball -- --network_link lxcbr0 --imgtar template.tar.gz 
``` 

the `--network_link` is set to the default `lxcbr0` bridge that LXC creates. If
you have a different networking set-up, change `lxcbr0`  to your bridge.
