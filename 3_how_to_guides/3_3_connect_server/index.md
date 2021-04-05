---
layout: default
title: connect to server
permalink: /how_to/connect_to_server
parent: how-to guides
nav_order: 3
---

#### prerequisites

* installed `fdrtd` package [(see here)](../3_1_install_package)
* imported `fdrtd` library [(see here)](../3_2_import_client_library)

#### how-to

to connect to a `fdrtd` server, you provide its URL to 
the constructor of the API interface `fdrtd.Api`:

<details markdown="block">
  <summary> Python </summary>
 
```python
api = fdrtd.Api(url='my.server.com/fdrtd/latest/')
```
</details>

#### result

`api`, an instance of `fdrtd.Api`
