---
layout: default
title: create network
permalink: /how_to/create_network
parent: how-to guides
nav_order: 4
---

#### prerequisites

* `api`, an instance of `fdrtd.Api` [(see here)](../3_3_connect_to_the_server)

#### how-to

<details markdown="block">
  <summary> Python </summary>
 
```python
network_id = api.create_network(network_definition=
  {
    'nodes': [<url1>, <url2>, ..., <urlN>],
    'myself': <0 ... N-1>,
    'central': <url>,
    'sync': 'https://fdrtd.alice.com'
  })
```
</details>

#### result

`api`, an instance of `fdrtd.Api`

#### explanation

first, we need to define the network, then upload the definition to the server.

the network definition lists all peer-to-peer nodes as an array of urls.
these are the urls of the servers forming the network.

the nework definition may optionally contain an additional central node
(e.g. for homomorphic encryption protocols with central processing)

the nework definition may optionally contain an additional sync node
for synchronisation services between the nodes in the network.

*SECURITY WARNING* the sync node may be identical to any of the peer-to-peer nodes
or a separate node, but it must never be the central node, or else the central
node might learn homomorphic encryption nonces and keys.

for example:
