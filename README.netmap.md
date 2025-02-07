Fast packet I/O using netmap
============================

ZMap can be built for sending and receiving packets using netmap(4), for very
high packet rates, especially on 10 GbE and faster links.
See [netmap/README.md](https://github.com/luigirizzo/netmap) for more
information on netmap.

Netmap is available by default on FreeBSD on many architectures, including
amd64 and arm64, and is easy to add to the kernel config on architectures where
it is not built by default.  While netmap has been ported to Linux, ZMap's
netmap mode currently only supports FreeBSD and will require porting to build
and run on Linux.


### Prerequisites

  0. A working ZMap development environment (see [INSTALL.md](INSTALL.md)).
  1. A kernel with netmap support (check for existence of `/dev/netmap`).
  2. For best results, a NIC with a driver that is netmap-aware, such as
     FreeBSD's `ixgbe` or `ixl`.


### Building

To build navigate to the root of the repository and run:

```
$ cmake -DWITH_NETMAP=ON -DENABLE_DEVELOPMENT=OFF .
$ make
```


### Running

Run zmap as you would normally.  For best results, use the `--cores` option to
pick which cores to pin to, pinning to different physical cores.  The number of
send threads is automatically capped to the number of TX rings, and to the
number of available cores after setting aside one core for the receive thread,
but you may still want to override the number of threads with `-T`.

Warning:  Netmap will disconnect the NIC from the host network stack for the
duration of the scan.  If you use an interface that you depend on for e.g. SSH
access, you will lose connectivity until ZMap exits.

```
$ sudo ./src/zmap -p 443 -i ix0 -o output.csv
```


### Considerations

DO NOT TAKE THIS LIGHTLY!

Running ZMap at 10Gbps hits every /16 on the Internet over 200 times a second.
Even if you have a large source IP range to scan from, it's very obvious that
you're scanning. As always, follow scanning best practices, honor blocklist
requests, and signal benign/research intent via domain names and websites on
your scan IPs.

Remember, you're sending a lot of traffic.
