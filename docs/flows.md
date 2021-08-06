---
layout: default
title: Flows
nav_order: 2
---

# Flows

A "flow" is a series of stages for moving data between systems. It begins with a trigger and always terminates, typically after transforming data from the trigger and storing or publishing that data. However there may be intermediary stages that hydrate or expand the data by querying other systems. Stages can also fan-out (e.g. map) or combine (e.g. reduce) the results of prior stage executions, allowing for efficient computations. The most formal definition of a "flow" is that of a directed acyclical graph ([DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph)), as flows must be fully traversable, not contain cycles between stages, and fully terminate.

![](../assets/images/Flows.svg)

The behavior of each stage defaults to buffering inputs within a given bounds, retry on errors with backoff, collect metrics on runtime stats, and collect logs from stdout/err.

## Flow runs and triggers
Every flow defines a trigger for starting a "run" of the flow. Flow "runs" comprise of an execution of the flow stages for the initial input from the flow's trigger. Triggers can be manual, but most often will be automated.

Trigger stages are pre-packaged and follow the same behavior as any other stage, as described below. The most generic trigger stage is a webhook trigger, which can be configured with a specific subdomain and path. This will trigger the start of a run on each HTTP request, with input that matches the stage props (see below for more information about stage props). Other triggers can start a flow run based on streaming data, changes to a database, or events generated from third-party systems.

Fow runs must complete, either in success or error, and each flow run is recorded and timed by default.

## Flow stages
Stages are discreet units that encapsulate work or logic that transform data mid-flight during a flow, or extract and combine additional data into a flow in addition to the trigger.

For example, suppose there is the following sequence defined as a flow:
 1. Read events from an internal event stream, such as Kinesis, and deserialize into JSON
 2. Remap each JSON data into a new JSON object, renaming or redacting fields
 3. Fetch additional data from internal services to append to the object
 4. Accumulate JSON objects into batches
 5. Published batched data to a third-party API

[_INSERT FLOW GRAPHIC_]
 
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

### Flow Schema
The complete YAML definition for a Flow must conform to the following schema:

```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "additionalProperties": false,
    "definitions": {
        "stage": {
            "additionalProperties": false,
            "properties": {
                "name": {
                    "description": "Uniquely identify the stage within a flow",
                    "type": "string"
                },
                "action": {
                    "description": "The name of the action this stage will perform. The name could be custom to identify this stage, or correspond to a packaged stage.",
                    "type": "string"
                },
                "props": {
                    "additionalProperties": true,
                    "description": "Props to pass to the action. This can be a json schema, loosely define data types or reference manifest items.",
                    "properties": {},
                    "type": "object"
                },
                "out": {
                    "description": "List of stages to invoke with the result of this stage.",
                    "items": {
                        "type": "string"
                    },
                    "type": "array"
                }
            },
            "required": [
                "action",
                "props",
                "name",
                "out"
            ],
            "type": "object"
        }
    },
    "properties": {
        "name": {
            "description": "Uniquely identify the flow",
            "type": "string"
        },
        "stages": {
            "description": "List of stages for the flow",
            "items": {
                "$ref": "#/definitions/stage"
            },
            "type": "array"
        }
    },
    "required": [
        "name",
        "stages"
    ],
    "type": "object"
}
```