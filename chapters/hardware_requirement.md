## Hardware Requirement

### Computing Nodes

To build a cluster, actually we need just more than one machines. Your old PC, spare laptop, or Raspberry Pi, would all be great candidates.

Here I used three Raspberry Pi 3, equipped with [Raspbian operating system](https://www.raspberrypi.org/downloads/raspbian/) which is based on Debian.

<br><br>

### Network Components

I tried to use Wi-Fi to build the connections among nodes, which turned out to be an extremely bad idea. The low speed and high latency made the cluster unstable and impossible to work properly. Intead, I use an Ethernet switch plus cat-6 cables.

These would be enough for our project, which is a simple project for learning. But, please note, for those real-world high-performance computing cluster, they use dedicated communication standard like [InfiniBand](https://en.wikipedia.org/wiki/InfiniBand).

<br><br>

### Others

Of course, we also need other devices, like monitor, keyboard, and devices to power other devices, etc. 

