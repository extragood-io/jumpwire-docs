---
layout: default
title: Flow Stages
parent: Flows
nav_order: 1
---

# Flow stages

Stages are discreet units that encapsulate work or logic that transform data mid-flight during a flow, or extract and combine additional data into a flow in addition to the trigger.

For example, suppose there is the following sequence defined as a flow:

1.  Read events from an internal event stream, such as Kinesis, and deserialize into JSON
2.  Remap each JSON data into a new JSON object, renaming or redacting fields
3.  Fetch additional data from internal services to append to the object
4.  Accumulate JSON objects into batches
5.  Published batched data to a third-party API

[_ALTERNATIVE EXAMPLE_]

For example, suppose there is a flow that begins when a new user requests information from a marketing site, and ends with that user's information in a data lake and a CRM for follow up by analysts and sales. The flow would be triggered by an form submission from a website, as a result of a new event created in the marketing engine, Hubspot. The flow takes the company name and email address from the form input, splits it into a domain name, validates the domain name against a database of existing customers, pulls an monthly activity report for the existing customers (if any), and uploads the result into the CRM, Salesforce and the internal data lake. The results of the flow will be used by the sales team for outreach and analysts/PMs for refining marketing strategy.

## Flow Definition

All flows can be defined by YAML that corresponds to the following schema. At the top level, a flow is designated by a name and list of stages:

```yaml
name: Publish to Acme API
stages:
  - trigger stage...
  - transform stage...
  - fetch stage...
  - publish stage...
```

Each stage of a flow is configured with a "props" object that corresponds to the shape of the data the stage will receive as input. The props object can reference a manifest entry, such as a row schema from a database. It may also reference a JSON schema of an expected input object. This is helpful for raising warnings during flow runs that data patterns don't match what is expected.

Stage definition YAML follows the following structure:

```yaml
name: Unique name for this stage within a Flow
props:
  key1: "val 1"
  key2:
    - "val 2a"
    - "val 2b"
  key3:
    child: "val 4"
action: Remap Fields
out:
  - Name of a subsequent stage
  - Name of another subsequent stage
```
