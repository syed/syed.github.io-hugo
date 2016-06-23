+++
title = "Limiting I/O bandwidth on docker"
date = "2015-06-09"
+++

When you create a docker container, by default, I/O on it is unlimited.  For my
project I needed to create docker containers with limited I/O bandwidth but
docker does not provide I/O limits out of the box like cpu (`--cpuset` and `--cpu-shares`) or memory (`--memory`) limits. However, the
kernel provides `blkio` cgroup subsytem which can be used to limit I/O on a block
device. 

I created this script which limits I/O bandwidth on containers. 

<script src="https://gist.github.com/syed/d44a781e248769f0580a.js"></script>

To use the script, create a container and record the container ID. Pass the ID 
to the script along with the read and write bandwidth (measured in bytes per sec). 

```bash
$ CONTAINER_ID=$(docker run -d ubuntu sleep 4000)
$ python docker_io_limit.py $CONTAINER_ID --read 1048576 --write 1048576

```

The above command will give the docker container a read/write bandwidth of 1MB/s. 
You can verify this by using dd 

```bash
$ docker  exec -it $CONTAINER_ID /bin/bash
root@179768a19696:/# dd if=/dev/zero of=testfile count=10240 oflag=direct
10240+0 records in
10240+0 records out
5242880 bytes (5.2 MB) copied, 6.87569 s, 763 kB/s
root@179768a19696:/# dd of=/dev/null if=testfile count=10240 iflag=direct
10240+0 records in
10240+0 records out
5242880 bytes (5.2 MB) copied, 5.2248 s, 1.0 MB/s
root@179768a19696:/# 
```

As you can see we get about `763 kB/s` write and `1.0 MB/s` read speed. 


