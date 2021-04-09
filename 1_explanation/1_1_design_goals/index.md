---
layout: default
title: design goals
permalink: /explanation/design_goals
parent: explanation
nav_order: 1
---


# background: advantages of privacy-preserving computation

privacy-preserving computation technologies such as 
[secure multiparty computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation) or 
[homomorphic encryption](https://en.wikipedia.org/wiki/Homomorphic_encryption) 
are interesting and active topics in cryptography research.

through secure protocols, they enable a number of parties to jointly compute a function
without revealing their private inputs to each other or a third party.

the usually conflicting goals of collaboration and data protection are thus reconciled
by avoiding data sharing in the first place.

further reading:
* [Maximize collaboration through secure data sharing](https://www.accenture.com/us-en/insights/digital/maximize-collaboration-secure-data-sharing): a report by Accenture
* [UN Handbook on Privacy-Preserving Computation Techniques](http://publications.officialstatistics.org/handbooks/privacy-preserving-techniques-handbook/UN%20Handbook%20for%20Privacy-Preserving%20Techniques.pdf): a report by the Privacy Preserving Techniques Task Team of the United Nations
* [MPC alliance](https://www.mpcalliance.org/): an industry alliance of companies interested in secure multiparty computation


# listen to your clients

while elegant in theory, it has proven quite difficult to provide practical implementations and demonstrate use cases.
proposing the technology to potential users often creates a lot of interest at first but little traction in following
up. while CEOs and CTOs are typically in favour, CIOs, CSAs and developers tend to have reservations. they may claim
the technology ...

* was complex and intransparent
* was built on monolithic code
* required a lot of computing power
* required very specific runtimes
* required exotic developer skills
* was cumbersome to develop and operate
* did not provide a standardized interface
* did not integrate well into legacy ecosystems
* was targeted at academic crypto experts

talking to decision makers and stakeholders quickly reveales that quite the opposite is desired:

* they want to solve specific problems and did not care about universality
* their problems are typically quite simple in algorithmic terms
* they do not have any crypto experts nor particularly trained developers
* their data protection officers are sceptical about novel methods
* they need a precise standard that could be referenced in legal documents
* they need a prototyping approach that was up to speed with their CI/CD
* they want a framework that was as simple as possible to operate and service
* they want to collaborate with others who had very different systems
* they want to choose their own language and development environments
* they need the new methods to be integrated step-by-step into legacy code

thus it quickly became clear that what was needed was *not another framework*
but rather some *middleware* that would help to develop for existing
frameworks and help to deploy and maintain solutions in a more convenient way.

further reading:
* [fdrtd.com](https://fdrtd.com): non-technical information for decision makers and stakeholders


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
  - **transparency**:
  - **no barriers to entry**: we want every company, institution, and individual to be able to reap the benefits
    of privacy-preserving computation (read more about our motivation at [fdrtd.com](https://fdrtd.com))

further reading:
* [architecture](/docs/explanation/architecture)
* [feature set](/docs/explanation/feature_set)
* [typical setups](/docs/explanation/typical_setups)
