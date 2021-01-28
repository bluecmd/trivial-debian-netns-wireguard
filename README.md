# Usage

Add the following to your `/etc/network/interfaces` file:

```
auto vpn0
iface vpn0 inet manual
  pre-up (ip netns add $IFACE 2>/dev/null && ip netns exec $IFACE ip link set lo up) || true
  pre-up ip netns exec $IFACE ip link del $IFACE 2>/dev/null || true
  pre-up ip link add $IFACE type wireguard && ip link set dev $IFACE netns $IFACE
  pre-up ip netns exec $IFACE wg setconf $IFACE /etc/wg/$IFACE.conf || (ip netns exec $IFACE ip link del $IFACE; false)
  up ip netns exec $IFACE ip link set $IFACE up
  up ip netns exec $IFACE ip addr add 10.71.73.26/32 dev $IFACE
  up ip netns exec $IFACE ip -6 addr add fc00:bbbb:bbbb:bb01::8:4919/128 dev $IFACE
  up ip netns exec $IFACE ip ro add default dev $IFACE
  up ip netns exec $IFACE ip -6 ro add default dev $IFACE
  down ip netns exec $IFACE ip link del $IFACE
```

This script configures both IPv4 and IPv6 for a Wireguard tunnel, reading configuration from `/etc/wg/vpn0.conf`.

To enter the namespace as your user, do something like: `sudo ip netns exec vpn0 sudo -iu "$USER"`
