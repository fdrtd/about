---
layout: default
title: architecture
permalink: /explanation/architecture
parent: explanation
nav_order: 2
---

# client-server architecture

there is a strict separation between **clients** and **servers**.

clients run the **business logic**. the business logic typically imports the `fdrtd` library,
which is nothing but a lightweight client-side **API wrapper**.

servers run an **API controller** which puts any requests on a **bus** to be served by **microservices**.
the microservices also use the bus to communicate with each other.

![figure 1](/docs/assets/img/client_server.png "figure 1 // client-server architecture")


# local, on premises, and in the cloud

the server is really only a collection of microservices and need not be a physical machine.
in fact, the server and the client may be installed on the same **local machine**.
this is the simplest setting for small-scale development, working with test data,
and for some applications such as citizen science.

in production, however, the server will most likely be situated in a secure data center **on premises**.
security are more easily managed here as servers, databases, and all data flow in between remain in custody.
in this setting, the **remote client** does not get to see any input data, it only receives the results.
furthermore, the server also provides all of the processing power, so the remote client may be very lean.

similarly, the server may be run in a managed **cloud instance**, or even fully **serverless**.
in this setting, the server's microservices will connect to other cloud based resources such as
managed databases. from the view of the client, there is no difference to a server on premises.
this setting lends itself to large-scale production system and highly scalable IoT applications.

of course, any combination of clients and servers can be freely mixed and matched.
the API interface guarantees full compatibility and interoperability.

![figure 2](/docs/assets/img/local_onpremises_cloud.png "figure 2 // local, on premises, or in the cloud")


# one-way flow of data

clients may **push** data to the server (see [how to upload data to the server](/docs/how_to/upload_data)).
alternatively, the client may tell the server to **pull** data from other sources such as a database
(see [how to upload data to the server](/docs/how_to/upload_data)). in this way, data remains and is
processed in the data center, which is preffered both for security and performance.

data that has been made available on the server may be accessed in the clear by other microservices.
however, data may never be downloaded by the client. it is forbidden by the API. instead, the client
may download results of computations only.

![figure 3](/docs/assets/img/one_way_flow_of_data.png "figure 3 // one-way flow of data")


# distributed computing

every party in distributed computing run their own server, databases, and other data sources.
in this way, all of their data always remains in their own custody. as for joint computation,
there are microservices to do just that.

these microservices implement **secure protocols** such as secure multiparty computation or
homomorphic encryption. to do so, the communicate peer-to-peer with their sisters and brethren
on the other servers.

# flow of control

clients have full control over what data is processed and what joint computations are done.
however, unlike servers, not every party need to run their own. there may be situations
where one party (e.g. a researcher) will access multiple servers (of their collaborators)
to perform a joint computation. without the need of their colleagues to be at the keyboard
at the same time. this is also the most probably setting in IoT scenarios where coordination
across thousands of devices is much easier with one central client sending requests to all.

