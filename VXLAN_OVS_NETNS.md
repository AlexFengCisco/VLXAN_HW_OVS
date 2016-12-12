### Try OVS VXLAN tunnel test with Network Namespaces

####on vagrant box-1
#### -----------------

    sudo ip netns add vm1
    sudo ip link add name veth1 type veth peer name sw1-p1
    sudo ip link set dev veth1 netns vm1
    sudo ip netns exec vm1 ifconfig veth1 10.0.0.1/24 up

    sudo ovs-vsctl add-br sw1
    sudo ovs-vsctl add-port sw1 sw1-p1
    sudo ip link set sw1-p1 up
    sudo ip link set sw1 up


#### on vagrant box-2
#### -----------------

    sudo ip netns add vm2
    sudo ip link add name veth1 type veth peer name sw2-p1
    sudo ip link set dev veth1 netns vm2
    sudo ip netns exec vm2 ifconfig veth1 10.0.0.2/24 up

    sudo ovs-vsctl add-br sw2
    sudo ovs-vsctl add-port sw2 sw2-p1
    sudo ip link set sw2-p1 up
    sudo ip link set sw2 up

#### on vagrant box-1
#### -----------------
    sudo ovs-vsctl add-port sw1 vx1-- set Interface vx1 type=vxlan options:remote_ip=192.168.56.12

#### on vagrant box-2
#### -----------------
    sudo ovs-vsctl add-port sw2 vx1 -- set Interface vx1 type=vxlan options:remote_ip=192.168.56.11


#### Test that now there is connectivity between 'vm1' and 'vm2'
#### ----------------------------------------------------------------
    ubuntu@box-1 ~$ sudo ip netns exec vm1 ping 10.0.0.2
    PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
    64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.644 ms
    64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.436 ms
