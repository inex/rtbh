#### Mikrotik RTBH Client

We have observed configuration issues with setting `192.0.2.1/32` as a blackholed next-hop IP address within a Mikrotik configuration.

Generally it's expected that once an IP is announced from the RTBH server to a Mikrotik router, the router will blackhole packets destined for the victims IP address (where `192.0.2.1` would be the next hop that has been preconfigured as a blackhole).

While the announcement is accepted, it is not considered active due to `192.0.2.1` being unreachable. This is potentially solvable using scope/target-scope attributes which we will lab up when we have a chance.

**Example:**

Victims IP: `198.51.100.22`

```
dst-address=198.51.100.22/32 gateway=192.0.2.1 gateway-status=192.0.2.1 unreachable
    distance=200 scope=40 target-scope=30 bgp-local-pref=5000
    bgp-origin=igp bgp-communities=xxx:6666 received-from=rtbh01
```

To work around this, there is a need to add a inbound filter to accept the RTBH bgp-community and set the IP as blackholed locally on a Mikrotik router.

  - Assuming no filters are yet in place for a RTBH BGP session, create the following filter:

```
/routing filter
add chain=rtbh01-in action=accept bgp-communities=yourasn:6666 comment="RTBH" set-type=blackhole
add chain=rtbh01-in action=discard
```

 - Add above filter chain (rtbh01-in) to RTBH peer

```
/routing bgp peer
set X in-filter=rtbh01-in
```

In order to announce RTBH IP addresses upstream, add the appropriate version of following to an existing routing filter for an upstream provider. This is an example with Hibernia networks (`ebgp-5580-out`):

```
add action=accept bgp-communities=yourasn:9999 chain=ebgp-5580-out set-bgp-communities=5580:666
```

The result of these configurations will be to blackhole packets destined for a RTBH'd IP address both locally on a Mikrotik router and on ingress to your upstream provider's networks'.
