# Networking

# Network Namespaces
Once we have our containers created, it is isolated between the host systems's network, process, which means the process running inside the container is not visible to pysical host.

We can create as such in below examples.

## Physical Host
```
ps aux
route
arp
ip netns
```

## Virtual/Containers
```
ip netns exec <namespace> ps aux
ip netns exec <namespace> route
ip netns exec <namespace> arp
ip netns exec <namespace> ip netns
```

### Operations - Network Namespace
```
ip netns add red
ip netns add blue
ip netns exec red ip link
ip netns exec blue ip link
ip link add veth-red type veth peer name veth-blue
ip link set veth-red netns red
ip link set veth-blue netns blue
ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n blue addr add 192.168.15.2/24 dev veth-blue
ip -n red link set veth-red up
ip -n blue link set veth-blue up
ip netns exec red ping 192.168.15.2
```

# Cluster Networking


# Pod Networking


# CNI in Kubernetes


# CoreDNS


# Ingress
