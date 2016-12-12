## Basic OVS setup

    #ovs-vsctl add-br mybridge
    #ifconfig mybridge up
    #ovs-vsctl add-port mybridge  eth0
    #ovs-vsctl show
    #ifconfig eth0 0
    #dhclient mybridge
    #ip tuntap add mode tap vport1
    #ip tuntap add mode tap vport2
    #ifconfig vport 1 up
    #ifconfig vport 2 up
    #ovs-vsctl add-port mybridge vport1
    #ovs-vsctl add-port mybridge vport2
    #ovs-appctl fdb/show mybridge
    #ovs-ofctl show mybridge
    #ovs-ofctl dump-flows mybridge
    #ovs-vsctl list Bridge

    #ip link
    #ip link [nic xx] up

## qemu setup

    #kvm-ok   //check cpuinfo if it support virtual machine
    #apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils

    virt-install   or  virt-manager

####create vm
    # virt-install \
     -n myRHELVM1 \
     --description "Test VM with RHEL 6" \
     --os-type=Linux \
     --os-variant=rhel6 \
     --ram=2048 \
     --vcpus=2 \
     --disk path=/var/lib/libvirt/images/myRHELVM1.img,bus=virtio,size=10 \
     --graphics none \
     --cdrom /var/rhel-server-6.5-x86_64-dvd.iso \
     --network bridge:br0

    ＃virsh list —all
    #virsh dominfo vmxxx
    #virt-top

    #virsh console vmxxx
    #virsh shutdown vmxxx
    #virsh restart vmxxx
    #virsh start vmxxx

####delete a vm and vol
    ＃virsh destroy virt1.example.com
    ＃virsh undefine virt1.example.com
    ＃virsh vol-delete --pool vg0 virt1.example.com.img
    ＃virsh vol-list —pool default

####virsh console to vm tips

    #sudo cp /etc/init/tty1.conf /etc/init/ttyS0.conf
    #sudo vi ttyS0.conf and change the line:

    exec /sbin/getty -8 115200 ttyS0 xterm

    #sudo vi /etc/default/grub
     GRUB_CMDLINE_LINUX_DEFAULT=“serial=tty0 console=ttyS0"

    #sudo update-grub

    

####also vi /etc/ovs-ifup/down to automatic add port to OVS bridge
    ＃sudo vi /etc/ovs-ifup
    #!/bin/sh
    switch=“ovsbridge"
    /sbin/ifconfig $1 0.0.0.0 up
    ovs-vsctl add-port ${switch} $1
    
    #sudo vi /etc/ovs-ifdown
    #!/bin/sh
    switch=“ovsbridge"
    /sbin/ifconfig $1 0.0.0.0 down
    ovs-vsctl del-port ${switch} $1
    
    
####create vm via kvm
    #sudo kvm -m 2048 -net nic,macaddr=00:00:00:00:cc:10 -net tap,script=/etc/ovs-ifup,downscript=/etc/ovs-ifdown cirros-0.3.2.img

####convert img to qcow2
    ＃qemu-img convert -O xxxxx.img xxxx.qcow2



