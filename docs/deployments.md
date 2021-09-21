---
layout: default
title: Deployments
nav_order: 2
---

## Deploying JumpWire

The deployment strategy for JumpWire is to self-host the software directly in a customer's cloud. This is referred to as an "on-premise" deployment. By running JumpWire in their own infrastructure, customers can connect JumpWire to internal data systems without additional IT work to expose those systems to a remote host. This also alleviates security concerns with sharing connectivity or data with another third party.

JumpWire supports multiple deployment options for flexibility in self-hosting. The goal is to make JumpWire as easy to maintain as any SaaS product. Most configuration parameters come with defaults. When running multiple nodes of JumpWire, each node can discover and join the rest of the cluster without any intervention necessary. Every node is resilient to crashes and restarts without interrupting the operations of the cluster.

There are two platforms for running JumpWire today, with more being developed - AWS EC2 and container orchestrators. JumpWire publishes pre-built artifacts with all the necessary software for running JumpWire. This includes the execution engine, the JumpWire web portal, and data persistence.

### PostgreSQL

JumpWire can plug into an existing instance of PostgreSQL for application data persistence. This includes definitions for [Flows](/docs/flows), [Manifests](/docs/manifests), and [Secrets](/docs/secrets), as well as the traces of every Flow execution. **Please note** this database is not primarily used to store event data directly.

Using PostgreSQL is optional but recommended, as it provides the most durable persistence for the JumpWire application data. To enable PostgreSQL, all that is necessary is to provide host and connection details. The JumpWire application will use this configuration to automatically start with the PostgreSQL database being used for storage.

### AWS EC2

JumpWire provides a Cloudformation template for deploying into an AWS VPC. This template will set up an auto-scaling group of EC2 instances, running a pre-built AMI containing JumpWire. The following parameters can be provided to configure or override defaults:

| Parameter        | Required/Optional | Description                                                                                       | Default     |
| ---------------- | ----------------- | ------------------------------------------------------------------------------------------------- | ----------- |
| Token            | Required          | Authentication token used to connect to the JumpWire frontend                                     | n/a         |
| KeyName          | Required          | Name of an existing EC2 KeyPair to enable SSH access into the JumpWire instances                  | n/a         |
| VpcId            | Required          | ID of the VPC for the entire stack                                                                | n/a         |
| SubnetIds        | Required          | List of VPC subnets to launch the instances in                                                    | n/a         |
| InstanceType     | Optional          | EC2 instance type for the JumpWire servers                                                        | t2.small    |
| WebhookPort      | Optional          | Port to use for incoming webhooks                                                                 | 4000        |
| PostgresHost     | Optional          | Optional hostname of an optional PostgreSQL instance, used for event and flow definitions storage | n/a         |
| PostgresUser     | Optional          | Optional username for connecting to PostgreSQL                                                    | n/a         |
| PostgresPassword | Optional          | Optional password for connecting to PostgreSQL                                                    | n/a         |
| MaxSize          | Optional          | The maximum number of instanced in the AutoScalingGroup                                           | 2           |
| FrontendHost     | Optional          | Hostname or IP address of the JumpWire frontend                                                   | jumpwire.ai |

The full Cloudformation template will be provided to customers during onboarding.

### Container orchestration

JumpWire builds and makes available Docker container images for running in a container orchestration systems, such as Kubernetes or AWS ECS. More details for this type of deployment can be provided by contacting the JumpWire team.

Coming soon
{: .label .label-yellow }
