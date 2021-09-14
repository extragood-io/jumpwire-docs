---
layout: default
title: Flows
nav_order: 2
has_children: true
---

# Flows

A **Flow** is defined as a series of stages for moving data between systems. It starts with a trigger and always completes, typically after transforming data from the trigger and storing or publishing that data. Intermediary stages can filter data to match a criteria, hydrate the data by querying other systems, or add or rename fields. Stages can also fan-out (e.g. map) or combine (e.g. reduce) the results of prior stage executions, allowing for efficient computations.

The most formal definition of a Flow is as a directed acyclical graph ([DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph)), as Flows must be fully traversable, not contain cycles between stages, and terminate.

![](../../assets/images/Flows.svg)

The behavior of each stage defaults to buffering inputs within a given bounds, retry on errors with backoff, collect metrics on runtime stats, and collect logs from stdout/err.

## Creating a Flow

Flows are created in the JumpWire app by clicking on the "Add Flow" button on the Flows page. Give the Flow a unique name, add tags as necessary, and enter the Flow YAML in the definition text area.

After succesfully saving a Flow, a graphical visualization can be seen under the tab "Flow Visualization", which displays the sequence of stages by their name and action.

![](../../assets/images/flows_page.png)

## Flow runs and triggers

Each execution of a Flow is called a "run", and Flows must declare a stage that triggers the start of a fun. Triggers can be scheduled like a cron job, invoked through a webhook, subscriptions to streams, or change data capture.

The status of each stage's execution can be seen in real time in the JumpWire web app, under the Flow definition. Flow runs must complete, either in success or error, and each flow run is recorded and timed by default.

![](../../assets/images/flow_runs.png)

For a complete list of Flow stages and their capabilities, please see the [flow stages page](stages)

## Flow Schema

The complete YAML definition for a Flow must conform to the following JSON schema:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "additionalProperties": false,
  "definitions": {
    "stage": {
      "additionalProperties": false,
      "properties": {
        "action": {
          "description": "The name of the action this stage will perform. The name could be custom to identify this stage, or correspond to a packaged stage.",
          "type": "string"
        },
        "config": {
          "additionalProperties": true,
          "description": "Configuration to pass to the action.",
          "properties": {},
          "type": "object"
        },
        "name": {
          "description": "Uniquely identify the stage within a flow",
          "type": "string"
        },
        "out": {
          "description": "List of output stages.",
          "items": {
            "type": "string"
          },
          "type": "array"
        }
      },
      "required": ["action", "name"],
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
  "required": ["name", "stages"],
  "type": "object"
}
```
