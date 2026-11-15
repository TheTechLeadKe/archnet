# archnet

A TCP/IP stack I'm writing from scratch in [c4](https://github.com/adadynamics), on top of a
Linux TAP device. The same code is meant to run on a bare kernel later — the only
platform-specific part is the handful of bytes that actually touch the wire.

No libc sockets, no kernel networking. Ethernet, ARP, IPv4, ICMP and TCP all get built
here by hand. It isn't trying to be fast or complete. The point is to actually understand
what the kernel does for you when you call `connect()` — you don't really know a protocol
until you've had to answer an ARP request yourself.

## Where it's at

Working:
- TAP device wrapper — open, configure, read/write raw Ethernet frames
- NetDevice / Interface split (L2 device vs L3 address binding)

Next:
- ARP — reply to who-has, keep a small cache
- IPv4 + ICMP echo, so `ping` answers back
- UDP, then TCP

So today it brings up `tap0`, echoes frames, and not much else. Early days.

## Design

Two layers at the bottom:

- `NetDevice` owns the wire — a MAC address and two methods, `receive` and `transmit`.
  On Linux that's the TAP fd; on bare metal it's a NIC driver. Everything above only ever
  calls those two methods, and that's what keeps the stack portable.
- `Interface` binds an IP and netmask to a device. Addresses are a layer-3 thing and don't
  belong on the hardware, so they live here, not on the device.

c4 has no traits yet, so `NetDevice` is a plain struct standing in as the hardware black
box rather than an interface a driver implements. When
