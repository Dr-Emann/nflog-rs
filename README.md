# nflog-bindings-rust

[![Build Status](https://travis-ci.org/chifflier/nflog-rs.svg?branch=master)](https://travis-ci.org/chifflier/nflog-rs)

## Overview

nflog-rust is a wrapper library for
[libnetfilter-log](http://www.netfilter.org/projects/libnetfilter_log/). The goal is to provide a library to gain access to packets queued by the kernel packet filter.

It is important to note that these bindings will not follow blindly libnetfilter_log API. For ex., some higher-level wrappers will be provided for the open/bind/create mechanism (using one function call instead of three).

Since libraries to decode ip packets are already available, the bindings
will use them.

To use the library, a program must
- open a queue
- bind to a network family (`AF_PACKET` for IPv4)
- provide a callback function, which will be automatically called when a packet is received.
- create the queue, providing the queue number (which must match the `--nflog-group` from the iptables rules, see below
- run a loop, waiting for events. The program should also provide a clean way to exit the loop (for ex on `SIGINT`)

## Documentation

Documentation is created using the `cargo doc` command.

## Example

See [test_nflog](src/bin/nfprint.rs) for a minimal example.

## IPtables

You must add rules in netfilter to send packets to the userspace queue.
The number of the queue (--nflog-group option in netfilter) must match the
number provided to create_queue().

Example of iptables rules:

    iptables -A OUTPUT --destination 1.2.3.4 -j NFLOG --nflog-group 0

Of course, you should be more restrictive, depending on your needs.

## Privileges

nflog-rust does not require root privileges, but needs to open a netlink socket and send/receive packets to the kernel.

You have several options:
- Use the CAP_NET_ADMIN capability in order to allow your application to receive from and to send packets to kernel-space:
```setcap 'cap_net_admin=+ep' /path/to/program```
- Run your program as `root` and drop privileges

## License

This library is licensed under the GNU General Public License version 2.0, or (at your option) any later version.
