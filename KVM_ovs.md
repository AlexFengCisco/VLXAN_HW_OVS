
    #virsh net-destroy default
    # apt-get purge ebtables 

    # apt-get install openvswitch-switch

    # ovs-vsctl add-br ovsbr0
    # virsh net-edit ovsbr0
         <network>
         <name>ovsbr0</name>
<forward mode='bridge'/>
<bridge name='ovsbr0'/>
<virtualport type='openvswitch'/>
</network>
# virsh net-autostart ovsbr0

virsh # net-info ovsbr0
Name:           ovsbr0
UUID:           a19267df-19e0-4888-b121-d1fc9399f744
Active:         yes
Persistent:     yes
Autostart:      yes
Bridge:         ovsbr0

virsh # net-list 
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 ovsbr0               active     yes           yes

ohara@ubu-ovs:~$ cat /etc/network/interfaces
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet manual 

auto ovsbr0
iface ovsbr0 inet static
address 192.168.74.129
netmask 255.255.255.0
gateway 192.168.74.2
dns-nameservers 192.168.74.2

ohara@ubu-ovs:~$ sudo ovs-vsctl show
c3c9a3e4-4d63-4d42-b354-8ef244b7d4fb
    Bridge "ovsbr0"
        Port "ovsbr0"
            Interface "ovsbr0"
                type: internal
        Port "eth0"
            Interface "eth0"
    ovs_version: "2.0.2"

ohara@ubu-ovs:~$ cat StartOSv.sh 
sudo qemu-system-x86_64 -vnc :1 -gdb tcp::1234,server,nowait -m 1G -smp 2 \
  -chardev stdio,mux=on,id=stdio -mon chardev=stdio,mode=readline,default \
  -device isa-serial,chardev=stdio -drive file=osv-v0.13.qemu.qcow2,if=virtio,cache=unsafe \
  -netdev tap,id=hn0,script=ovs-ifup,downscript=ovs-ifdown,vhost=on \
  -device virtio-net-pci,netdev=hn0,id=nic1,mac=52:54:00:12:34:56 \
  -device virtio-rng-pci -enable-kvm -cpu host,+x2apic

ohara@ubu-ovs:~$ cat StartOSv-2.sh 
sudo qemu-system-x86_64 -vnc :2 -gdb tcp::1235,server,nowait -m 1G -smp 2 \
  -chardev stdio,mux=on,id=stdio -mon chardev=stdio,mode=readline,default \
  -device isa-serial,chardev=stdio -drive file=osv-v0.13.qemu.qcow2,if=virtio,cache=unsafe \
  -netdev tap,id=hn0,script=ovs-ifup,downscript=ovs-ifdown,vhost=on \
  -device virtio-net-pci,netdev=hn0,id=nic1,mac=52:54:00:12:34:57 \
  -device virtio-rng-pci -enable-kvm -cpu host,+x2apic

ohara@ubu-ovs:~$ cat ovs-ifup 
#!/bin/sh
switch='ovsbr0'
/sbin/ifconfig $1 0.0.0.0 up
ovs-vsctl add-port ${switch} $1

ohara@ubu-ovs:~$ cat ovs-ifdown
#!/bin/sh
switch='ovsbr0'
/sbin/ifconfig $1 0.0.0.0 down
ovs-vsctl del-port ${switch} $1

root@ubu-ovs:/home/ohara# ovs-vsctl show                                                                                                   
c3c9a3e4-4d63-4d42-b354-8ef244b7d4fb
    Bridge "ovsbr0"
        Port "ovsbr0"
            Interface "ovsbr0"
                type: internal
        Port "tap0"
            Interface "tap0"
        Port "eth0"
            Interface "eth0"
        Port "tap1"
            Interface "tap1"
    ovs_version: "2.0.2"

root@ubu-ovs:/home/ohara# ovs-ofctl show ovsbr0                                                                                            
OFPT_FEATURES_REPLY (xid=0x2): dpid:0000000c29094cca
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: OUTPUT SET_VLAN_VID SET_VLAN_PCP STRIP_VLAN SET_DL_SRC SET_DL_DST SET_NW_SRC SET_NW_DST SET_NW_TOS SET_TP_SRC SET_TP_DST ENQUEUE
 1(eth0): addr:00:0c:29:09:4c:ca
     config:     0
     state:      0
     current:    1GB-FD COPPER AUTO_NEG
     advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
     supported:  10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
     speed: 1000 Mbps now, 1000 Mbps max
 6(tap0): addr:3a:53:d6:da:61:7d
     config:     0
     state:      0
     current:    10MB-FD COPPER
     speed: 10 Mbps now, 0 Mbps max
 7(tap1): addr:86:26:d3:30:6e:a3
     config:     0
     state:      0
     current:    10MB-FD COPPER
     speed: 10 Mbps now, 0 Mbps max
 LOCAL(ovsbr0): addr:72:72:b0:8d:76:8f
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0