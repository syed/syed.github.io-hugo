+++
title = "Linux Kernel Development using KVM"
date = "2016-07-14"
+++

I am back to doing some kernel hacking. I am building my kernel as I write this post. 
Every time I try to build a kernel, I have to setup the whole environment again which 
requires a bunch of google searches and skimming a bunch of tutorials. I am writing this 
post to consolidate all of that so that the next time I pick this up, I don't have to search
everywhere.

## Getting the source

From kenrel.org or from `apt-get` both work.

{% highlight bash %}

$ apt-get source linux-image-$(uname  -r)

{% endhighlight %}


## Building the kernel

{% highlight bash %}

 $ tar -xvf linux_3.13.0.orig.tar.gz 
 $ cd linux-3.13.0/
 $ make bzImage
 $ make_modules

{% endhighlight %}

This should give you a kernel image in `./arch/x86_64/boot/bzImage`. Now we need to create
an initrd which we will boot into. For this I use busybox. Make sure you have the statically
linked version of the `busybox` binary and place it in the root of the kernel source. 


## Building the initrd

Save the following script in the root of the kernel source with the filename `create_initrd.sh`.
This will build the scafollding of our initrd

{% highlight bash %}

mkdir -p  initramfs/bin
mkdir -p  initramfs/etc

touch  initramfs/init
touch  initramfs/etc/fstab

cp busybox initramfs/bin
cd initramfs/bin
ln -s busybox sh

{% endhighlight %}

The resultant directory structure will be:

{% highlight bash %}

initramfs
├── bin
│   ├── busybox
│   └── sh -> busybox
├── etc
│   └── fstab
├── init

{% endhighlight %}

Edit the `fstab` file and add the following lines

{% highlight bash %}

proc /proc proc rw,noexec,nosuid,nodev 0 0
sysfs /sys sysfs rw,noexec,nosuid,nodev 0 0

{% endhighlight %}

Edit the `init` file and add the following lines

{% highlight bash %}
#!/bin/sh

# Mount things needed by this script
mount -t proc proc /proc
mount -t sysfs sysfs /sys

# Create all the symlinks to /bin/busybox
busybox --install -s

# Create device nodes
mknod /dev/null c 1 3
mknod /dev/tty c 5 0
mdev -s

{% endhighlight %}

create a file `build_initrd.sh` which will be run everytime there is an update
to initrd. This may be useful if you add your program to initrd and want to 
run the kernel with the new program. This goes in the root of your kernel source

{% highlight bash %}

cd initramfs
find . | cpio -H newc -o > ../initramfs.cpio
cd ..
cat initramfs.cpio | gzip > initramfs.igz

{% endhighlight %}


## Running the kernel with KVM

Finally we are ready with our kernel and initrd. Create the file `run_kernel.sh` 
which will run the kenrnel in your terminal. 

{% highlight bash %}

kvm -kernel arch/x86_64/boot/bzImage \
    -initrd initramfs.gz \
    -chardev stdio,id=stdio,mux=on \
    -device virtio-serial-pci \
    -device virtconsole,chardev=stdio \
    -mon chardev=stdio \
    -display none \
    -append 'console=hvc0'\
    -s

{% endhighlight %}


## Debugging using remote `gdb`

The above command uses KVM to run the kernel and displays the output in the
current terminal. It also starts a gdb server for debugging which is listening
on port `tcp:1234`. You can use a remote `gdb` to  debug.


{% highlight bash %}

 $ gdb vmlinux
 (gdb) set architecture  i386:x86-64
 (gdb) target remote :1234

{% endhighlight %}

That's it. Once the kernel is up and running, you can attach the debugger and
debug. If you want to add any modules, throw them in the initramfs directory,
update the `init` script to load them when the kernel boots up, rebuild the
initramfs and you are set.


### References

[1] http://jootamam.net/howto-initramfs-image.htm <br/>
[2] https://blog.nelhage.com/2013/12/lightweight-linux-kernel-development-with-kvm/


