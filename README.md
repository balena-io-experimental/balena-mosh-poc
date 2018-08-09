Proof of concept to use mosh to access a container on a resin device

**This is just a quick proof of concept and can be much better polished and automated,
there is also much more room to improve security, like not have to run ssh on the desktop as root.**


In general this works like this:

In the container on the device there will be an ssh server running, which will be made
reachable via ngrok. We will use ssh tunnel forwarding (https://help.ubuntu.com/community/SSH_VPN)
to create a virtual network tunnel between the container and the desktop computer. Through this
tunnel we can open a mosh connection with the mosh-server on the device.
We need the virtual tunnel, because mosh is UDP based, so the client and the server need 
to be able to exchange UDP packets.

How to use this:

Add your ssh key to the authorized keys file of root in the container.

Start your mosh-server on the device:

```
mosh-server
```

To reach the SSH service on the device run ngrok:

```
/usr/apps/ngrok tcp 22 --authtoken <YOUR_AUTH_TOKEN>
```

Then open an ssh connection with tunnel forwarding to the ngrok endpoint:

```
sudo ssh -w0:0 root@<NGROK_HOST> -p<NGROK_PORT>
```

When this connection is open setup the newly created tunnel interface on the device:

```
ip link set tun0 up
ip addr add 10.0.0.100/32 peer 10.0.0.200 dev tun0
```

and the newly created tunnel interface on the desktop:

```
ip link set tun0 up
ip addr add 10.0.0.200/32 peer 10.0.0.100 dev tun0
```


Now you should be able to ping the device through the tunnel:

```
ping 10.0.0.100
PING 10.0.0.100 (10.0.0.100) 56(84) bytes of data.
64 bytes from 10.0.0.100: icmp_seq=1 ttl=64 time=284 ms
...
```

If that works start your mosh session:

```
mosh root@10.0.0.100
```
