# Network Namespace with bridge network
We have to create two network namespace and connect them through a bridge so that they can communicate with each other

## Diagram
<img width="547" alt="bridgeNetwork" src="https://user-images.githubusercontent.com/37947169/210134623-662f6387-4f94-4b39-ab70-faf22a6a62a3.png">


## Create two network namespace
```
sudo ip netns add red
sudo ip netns add green
```
To list the namespace
```
sudo ip netns list
```

## Prepare a bridge network
We will connect two network namespace with this bridge
```
sudo ip link add br0 type bridge
sudo ip link set dev br0 up
sudo ip addr add 192.168.0.1/16 dev br0
```

## Create Virtual Ethernet Cable (veth)
For conncecting network namespaces with the bridge
```
sudo ip link add rveth type veth peer name rbveth
sudo ip link add gveth type veth peer name gbveth
```

## Connect one side of eth to namespaces
Here oneside of virtual ethernet is being connected to namespaces and other side will be connected with the bridge as shown in diagram 
```
sudo ip link set rveth netns red
sudo ip link set gveth netns green
```

## Turn on the connector
```
sudo ip netns exec red ip link set dev lo up
sudo ip netns exec red ip link set dev rveth up
sudo ip netns exec green ip link set dev lo up
sudo ip netns exec green ip link set gveth up
```

## Create two port in Bridge network
```
sudo ip link set dev rbveth master br0
sudo ip link set dev gbveth master br0
```

## Turn on the bridge connector
```
sudo ip link set dev rbveth up
sudo ip link set dev gbveth up
```

## Add Ip to Interfaces of Namespaces
```
sudo ip netns exec red bash
ip addr add 192.168.0.2/24 dev rveth

sudo ip netns exec green bash
ip addr add 192.168.0.3/24 dev gveth
```

## Add the routing information via bridge
We need to add bridge ip address to the route table of namespace
```
sudo ip netns exec red ip route add default via 192.168.0.1 dev rveth
sudo ip netns exec green ip route add default via 192.168.0.1 dev gveth
```

Now we can ping green namespace from red using green namespace ip address and vice-versa

```
sudo ip netns exec red ping 192.168.0.3
sudo ip netns exec green ping 192.168.0.2
```

