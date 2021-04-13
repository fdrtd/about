---
layout: default
title: design goals
permalink: /explanation/design_goals
parent: explanation
nav_order: 3
---


# design decisions and design goals

`fdrtd` adopts a number of design decisions to meet the following design goals:

* **standardized API**
  - **separation of concerns**: business logic and cryptography layer change for different reasons.
    the business logic is a high-level concern and changes only due to domain-specific and task-specific requirements.
    the cryptography layer is a low-level concern and changes only to improve privacy preservation on the network layer.
  - **dependency inversion**: the business logic depends only on the abstraction of the interface, and not its
    particular protocols or realizations. the cryptography layer depends only on the abstraction of the interface,
    and not the particular shape, size, or form of the business logic. neither depend on details of implementation.
  - **compatibility**: implementations are easy to port to different operating systems and legacy environments
  - **interoperability**: users do not need to run the same software or systems and still are able to directly collaborate
* **microservices**
  - **single responsibility**: every microservice is responsible for one particular privacy-preserving computation
    (e.g. secure sum, private set intersection). sacrificing monolithic universality allows every single one of
    them to be highly optimized to their task.
  - **open-closed principle**: extending the functionality is as simple as adding yet another microservice to the bus.
    on the other hand, no microservice has to ever be modified for changes in another. the services may continue
    to run as is across versions, servers, operating systems.
* **client-server architecture**:
  - **performance**: the complete computing overhead is offloaded to the server where it can be handled much more efficiently
  - **ready for the cloud**: simply run the server in an instance in the cloud. there are also particular microservices
    for e.g. databases provided by the major IaaS platforms
  - **ready for IoT**: the remaining clients are so small (kilobytes) that they run on any smartwatch or IoT device
* **security by design**:
  - **keep data away from people**: one may keep the entire data flow server-side resp. in the cloud
  - **one-way flow of data**: there are no endpoints to download raw data to the client, only results
  - **ingress free**: through simple daemons outside of the firewall, peer-to-peer servers may operate egress-only
* **open technology**:
  - **convenient devsecops**: `fdrtd` is not another framework but rather an API standard defining the
    architecture of a middleware designed to make privacy-preserving computing simple to develop,
    convenient to operate and straightforward to secure
  - **freedom of choice**: users of `fdrtd` do not have to choose a particular protocol (e.g. secure multiparty
    computation, homomorphic encryption) or particular implementation, but may switch them on the fly
    without any changes to the business logic
  - **3rd party ecosystem**: server-side cryptography developers may write wrappers for their frameworks
    to make them available to the audience of `fdrtd`   
* **ready for business**:
  - **industry-grade**: we aim for industry-grade best practices in code design and deployment
  - **turn-key**: we aim to provide a turn-key solution that gets you up to speed within a single scrum sprint
  - **functionality**: we aim to provide wrappers for languages and databases as needed by the majority of users
* **free and open source**
  - **transparency**: users of `fdrtd` can have absolute peace of mind that their sensitive data is processed
    in the exact way it should be.
  - **no barriers to entry**: we want every company, institution, and individual to be able to reap the benefits
    of privacy-preserving computation (read more about our motivation at [fdrtd.com](https://fdrtd.com))
