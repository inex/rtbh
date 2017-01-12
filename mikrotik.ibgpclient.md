#### Mikrotik RTBH Client

We have observed configuration issues with setting `192.0.2.1/32` as a blackholed next-hop IP address within a mikrotik configuration.

Generally it's expected that once an IP is announced from the RTBH server to a mikrotik router, the result will send the offending IP address to `192.0.2.1` that has been preconfigured as a blackhole.
While the announcment is accepted, it is not considered active due to `192.0.2.1` being unreachable.

**Example:**

```
dst-address=198.51.100.22/32 gateway=192.0.2.1 gateway-status=192.0.2.1 unreachable distance=200 scope=40 target-scope=30 bgp-local-pref=5000
    bgp-origin=igp bgp-communities=xxx:6666,xxx:9999 received-from=rtbh01
```

To work around this, there is a need to add a filter to capture the rtbh bgp-community and set the IP as blackholed locally on a Mikrotik router.

```
/routing filter
add action=accept bgp-communities=yourasn:6666 chain=rtbh01-in comment="RTBH Server" set-type=blackhole
add action=discard chain=rtbh01-in
```


