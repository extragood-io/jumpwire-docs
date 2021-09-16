---
layout: default
title: Welcome
nav_order: 1
permalink: /
---

## Welcome to JumpWire

JumpWire is an integration platform that connects internal data to vendor APIs. The best use cases for JumpWire are those where data stored in private databases must be exchanged with third parties. Since JumpWire is designed to be easily self-hosted, the data passing through JumpWire never leaves local networks, and the internal systems JumpWire connects do not need to open connections to the internet. This stands in contrast of hosted integration platforms, which exposes data and connections to an intermediate SaaS vendor.

JumpWire also enforces data handling in a consistent manner across connections to third-party APIs. This is especially useful for scenarios where sensitive data, such as PII, is being shared but some fields, for example social security numbers, must be redacted before sharing.

![](assets/images/jumpwire-graph.svg)

JumpWire creates a graph of logcial steps between data sources and destinations. Each step can manipulate the data passing through it, and steps combine to create powerful transformations to the data.

The execution of graph steps are distributed across a cluster of servers, allowing JumpWire to scale linearly. Each step, called a "stage", can recover from failures without affecting the performance of the overall system.

In this documentation, we present the major concepts in JumpWire along with instructions for building execution graphs, called "flows", and configuring self-hosted deployments.

### Terminology

It can be used to create connections between various systems for the purpose of transferring data between them. JumpWire calls these connections "flows", and they represent a series of steps for extracting, transforming and loading data between systems. Examples of systems to connect include SaaS or vendor APIs, internal databases or data warehouses, streaming data including change data capture, or data from internal tools or services.

JumpWire provides benefits out-of-the-box for these types of workloads. All ETL steps are resilient to failure and pre-configured for reties and backoffs. This is useful for scenarios where an API or system may be unavailable for a brief period but then recovers. JumpWire also simplifies ETL programming by abstracting common protocols such as HTTPS and DBMS, in some cases eliminating the need for any custom coding beyond field matching.

By using JumpWire, teams gain increased visibility into the characteristics of connected systems such as data types being exchanged as well as rate, volume and availability. This is helpful for monitoring integration health wrt SLAs, and auditing governance of data handling hygene.

To get started with JumpWire, become familiar with the following JumpWire concepts:

- [Flows](flows)
- [Manifests](manifests)
- [Secrets](secrets)

You can reach out to the developers of JumpWire by email -- ryan _at_ jumpwire.ai || william _at_ jumpwire.ai
