# Step 3: Setup an IP Range for MetalLB

MetalLB is a load-balancer implementation for bare metal Kubernetes clusters, using standard routing protocols. Follow these steps to correctly set up an IP range for MetalLB:

## Install sipcalc

Open a terminal window and execute the following command to install `sipcalc`, an IP subnet calculator:

```bash
sudo apt-get install -y sipcalc
```

Identify Your Network Interface and IP Address
Use the ip a s to list all network interfaces and their IP addresses. Look for the interface connected to your network:

```bash
ip a s
```

You will see output like this:
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eno2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 94:40:c9:f3:20:e8 brd ff:ff:ff:ff:ff:ff
    altname enp100s0f0
    inet 10.239.200.41/16 brd 10.239.255.255 scope global eno2
       valid_lft forever preferred_lft forever
    inet6 fe80::9640:c9ff:fef3:20e8/64 scope link
       valid_lft forever preferred_lft forever
3: eno3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 94:40:c9:f3:20:e9 brd ff:ff:ff:ff:ff:ff
    altname enp100s0f1
4: eno4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 94:40:c9:f3:20:ea brd ff:ff:ff:ff:ff:ff
    altname enp100s0f2
5: eno1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 94:40:c9:f3:20:e7 brd ff:ff:ff:ff:ff:ff
    altname enp2s0
6: eno5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 94:40:c9:f3:20:eb brd ff:ff:ff:ff:ff:ff
    altname enp100s0f3
7: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:0e:ca:8c:07 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

What we want to add to the metallb range is the ip address block that is externally facing, or accessible on a private network:
`inet 10.239.200.41/16`
# Calculate the IP Range Using sipcalc

Run sipcalc with your network interface's IP address and subnet mask as arguments:

```bash
sipcalc 10.239.200.41/16
```

Configure MetalLB with the Desired IP Range
Enable MetalLB on MicroK8s by specifying the IP range:
(here we are specifying 249 ip addresses to be configurable by metallb)

```bash
microk8s enable metallb:10.239.255.1-10.239.255.250
```

By following these steps, you've successfully set up an IP range for MetalLB in your MicroK8s cluster.