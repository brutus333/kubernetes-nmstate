---
title: "Reporting"
tag: "user-guide"
---

The operator periodically reports state of node network interfaces to the API
server. These reports are available through `NodeNetworkState` objects that are
created per each node.

## List all handled nodes

List `NodeNetworkStates` from all nodes:

```shell
kubectl get nodenetworkstates
```

```
NAME     AGE
node01   22s
node02   22s
```

You can also use short name `nns` to reach the same effect:

> **Note:** there is a bug in kubectl where `get` on short names fails first time it is used.

```shell
kubectl get nns
```

```
NAME     AGE
node01   32s
node02   32s
```

## Read state of a specific node

By using `-o yaml` you obtain the full network state of the given node:

```shell
kubectl get nns node01 -o yaml
```

```yaml
apiVersion: nmstate.io/v1beta1
kind: NodeNetworkState
metadata:
  creationTimestamp: "2020-01-31T12:13:15Z"
  generation: 1
  name: node01
  ownerReferences:
  - apiVersion: v1
    kind: Node
    name: node01
    uid: 5292f6a0-de2d-425c-8c66-ab95fec461e1
  resourceVersion: "946"
  selfLink: /apis/nmstate.io/v1beta1/nodenetworkstates/node01
  uid: aada52e6-f7fa-4bc8-b580-27c2b70f4466
status:
  currentState:
    dns-resolver:
      config:
        search: []
        server: []
      running:
        search: []
        server:
        - 192.168.66.2
    interfaces:
    - bridge:
        options:
          group-forward-mask: 0
          mac-ageing-time: 300
          multicast-snooping: true
          stp:
            enabled: false
            forward-delay: 15
            hello-time: 2
            max-age: 20
            priority: 32768
        port: []
      ipv4:
        address:
        - ip: 10.244.0.1
          prefix-length: 24
        dhcp: false
        enabled: true
      ipv6:
        address:
        - ip: fe80::fc40:9cff:fefa:221a
          prefix-length: 64
        autoconf: false
        dhcp: false
        enabled: true
      mac-address: FE:40:9C:FA:22:1A
      mtu: 1450
      name: cni0
      state: up
      type: linux-bridge
    - ipv4:
        address:
        - ip: 192.168.66.101
          prefix-length: 24
        auto-dns: true
        auto-gateway: true
        auto-routes: true
        dhcp: true
        enabled: true
      ipv6:
        address:
        - ip: fe80::5055:ff:fed1:5501
          prefix-length: 64
        autoconf: false
        dhcp: false
        enabled: true
      mac-address: 52:55:00:D1:55:01
      mtu: 1500
      name: eth0
      state: up
      type: ethernet
    # output truncated
    route-rules:
      config: []
    routes:
      config: []
      running:
      - destination: 0.0.0.0/0
        metric: 0
        next-hop-address: 192.168.66.2
        next-hop-interface: eth0
        table-id: 254
      - destination: 192.168.66.0/24
        metric: 100
        next-hop-address: ""
        next-hop-interface: eth0
        table-id: 254
      - destination: ff00::/8
        metric: 256
        next-hop-address: ""
        next-hop-interface: eth0
        table-id: 255
      # output truncated
  lastSuccessfulUpdateTime: "2020-01-31T12:14:00Z"
```

As you can see, the object is cluster-wide (i.e. does not belong to a
namespace). Its `name` reflects the name of the Node it represents.

The main part of the object is located in `spec.currentState`. It contains the
DNS configuration, list of interfaces observed on the host and their
configuration, and routes.

<!-- TODO: Link API introduction once it is added to docs -->

Last attribute of the object is `lastSuccessfulUpdateTime`. It keeps a timestamp
recording the last successful update of the report. Since the report is updated
periodically and won't get updated while the node is not reachable (e.g. during
reconfiguration of networking), this value can be used to evaluate whether the
observed state is fresh enough.

## Configure refresh interval

The reported state is updated every 5 seconds.

## Node Network State interfaces filtering

All unmanaged `veth` interfaces are omitted from the report in order to not
clutter the output with all Pod connections.

## Continue reading

The following tutorial will guide you through the configuration of node
networking: [Configuration]({{ "user-guide/102-configuration.html" | relative_url }} )
