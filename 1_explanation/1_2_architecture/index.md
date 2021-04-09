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

![figure 1](/assets/img/clientserver.png "figure 1 // client-server architecture")

# flow of control and data

