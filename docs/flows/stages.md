---
layout: default
title: Flow Stages
parent: Flows
nav_order: 1
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Flow stages

{: .no_toc }

Flows are divided into individual steps called "stages". Stages are discreet units that encapsulate work or logic to handle data passing through an execution run of a Flow. They execute independently and take input from an external source or from a previous stage's output. JumpWire ships with many built-in stages for common operations, or a stage can run custom JavaScript code.

Stages can be wired together, with one stage receiving the output of another stage, allowing for composition of complex data handling or transformation logic. A stage will start execution as soon as there is input available to that stage.

Every Flow will contain a stage that acts as a "trigger", meaning it receives an external input that starts an execution of a Flow run. A stage that contains no list of `out` stages will stop the execution of the run (or branch of the run).

## Stage YAML definition

Each stage is defined by a `name`, `action`, `config `and `out`. The `name` and `action` fields are required. A `config` block may be required, depending on the logic of the stage. An `out` list is necessary if there are subsequent stages that should execute on the result of the given stage.

### Name

The `name` for a stage is a human-friendly label for the given stage. They must be unique within a flow, contain words, and can be referenced by other stages' `out` parameter.

### Action

The `action` for a stage is a reserved string that corresponds to the kind of logic this stage will perform on the incoming data, and what kind of output this stage will have. You can find a list of all supported stages below, and their corresponding `action` string.

### Config

The `config` block for a stage contains fields specific to the stage action needed for the stage to run. It can contain static values or reference [secrets](secrets). Please refer to the list of stages below for their required configuration.

### Out

The `out` list is a list of names of stages that should receive as their input the output of this stage. This allows stages to be wired together to move data through multiple transformations. There can be multiple stage names listed under `out`, and each of those stages will receive the same output. If no `out` list is present in the stage, this will stop the execution of the run (or branch of the run) once the stage is done. This would be an instance of a "terminal" stage.

### Sample YAML

In the example below, the stage "Decode JSON" decodes binary data into a JSON string sends that JSON string to two subsequent stages, which both receive the JSON as input. The "Decode JSON" stage receives input from a previous stage "Some trigger", that starts the Flow run and gets binary data from an external source.

```yaml
name: Handle some JSON like it's hot
stages:
  - name: Some trigger
    ...
    out: Decode JSON

  - name: Decode JSON
    action: json_decode
    out:
    - Subsequent stage name 1
    - Subsequent stage name 2

  - name: Subsequent stage name 1
    ...

  - name: Subsequent stage name 2
    ...
```

## Stage behavior

Stages are built to be very resilient to intermittent errors that might occur during stage execution. Further, errors are isolated within stages, meaning an error during the execution of one stage will not proliferate to other stages, or even to other executions of the same stage happening simultaneously.

Every stage has default behavior for handling failures or errors with retries and backoff. For a given input, each stage will retry up to 5,000 times with exponential backoff. Retry behavior can be customized for specific stages within the `config` block.

## Stage visualization

A visualization of how stages are connected through their `out` listings is available on the Flows page. Just navigate to a given Flow and click on the "Flow Visualization" tab.

![](../../assets/images/flow_viz.png)

The result and timing of each stage is available for every Flow run, by clicking on the result of a run in the table on the Flows page, underneath the Flow definition.

![](../../assets/images/flow_run_stages.png)

## Built-in Triggers

### Kinesis

A trigger stage that tails a given Kinesis data stream and trigger a Flow run for each record read from the stream. It will tail all shards of the stream.

The `action` for this stage is `kinesis`.

The `config` accepts the following values

```yaml
- name: Read from kinesis
  action: kinesis
  config:
    # REQUIRED, name of the Kinesis stream to read from
    stream: some_kinesis_string_name
    # REQUIRED, which portion of the stream to start tailing from at startup
    cold_start: trim_horizon | latest
  out:
    - ...
```

### Kinesis fan-out consumer

A trigger stage that creates a Kinesis fan-out consumer to trigger a Flow run for each record pushed from the stream. It subscribes to all shards of the stream. The consumer that gets created is called "jumpwire".

The `action` for this stage is `kinesis_consumer`.

The `config` accepts the following values

```yaml
- name: Read from kinesis
  action: kinesis
  config:
    # REQUIRED, name of the Kinesis stream to read from
    stream: some_kinesis_string_name
    # REQUIRED, which portion of the stream to start tailing from at startup
    cold_start: trim_horizon | latest
  out:
    - ...
```

### UDP

This trigger can be invoked by receiving a UDP packet to start a Flow run with the contents of the UDP request.

The `action` for this stage is `udp`.

The `config` block accepts the following values

```yaml
- name: Receive UDP data
  action: udp
  config:
    # REQUIRED, port number for receiving UDP requests, as an integer
    port: 6553
  out:
    - ...
```

### Webhook

This trigger will receive HTTPs requests to start a Flow run from each request, and take data from the request body.

The `action` for this stage is `webhook`.

The `config` block accepts the following values

```yaml
- name: Receive HTTP data
  action: webhook
  config:
    # REQUIRED, URL path for expected HTTP requests
    path: /incoming/webhook
  out:
    - ...
```

## Built-in Stages

### Delay

This stage will forward the data received as input, after pausing for a specified amount of time.

The `action` for this stage is `delay`.

The `config` block accepts the following values

```yaml
- name: Wait for me
  action: delay
  config:
    # REQUIRED, number of seconds to pause, as an integer
    seconds: 55
  out:
    - ...
```

### Error

A stage that generates an error whenever it receives events. Useful for debugging.

The `action` for this stage is `error`.

No `config` block is required for this stage

### JSON transformation

This stage will maps fields from the incoming JSON object into a new outgoing JSON object.

The `action` for this stage is `field_map`.

The `config` block is optional, with the following values. If no `config` block is given, the JSON object is not modified.

```yaml
- name: Transform JSON
  action: field_map
  config:
    # OPTIONAL, list of JSON fields to copy into the new object. If not specified, all fields will be copied
    take:
      - field1
      - field2
    # OPTIONAL, list of JSON fields to exclude from the new object. If not specified, no fields will be excluded
    drop:
      - field3
      - field4
    # OPTIONAL, a map of JSON fields to rename, with the key being the input field and the value being the renamed field
    rename:
      field5: newname1
      field6: newname2
    # OPTIONAL, a map of JSON fields to convert, with the key being the input field and the value being the desired type to convert to
    convert:
      field7: int
      field8: string
  out:
    - ...
```

As an example, this configuration would take the following input:

```json
{
  "field1": "foo",
  "field2": "bar",
  "field3": "fizz",
  "field4": "buzz",
  "field5": "spooky",
  "field6": "cat",
  "field7": "01.00",
  "field8": 9.1
}
```

and produce the following output:

```json
{
  "field1": "foo",
  "field2": "bar",
  "newname1": "spooky",
  "newname2": "cat",
  "field7": 1,
  "field8": "9.1"
}
```

### Data filter

This stage will filter out data that matches a given criteria.

Whether this stage drops or passes matching events depends on the config field `reject_matches`. By default, only data that does **not** match the criteria will be passed to the next stage, i.e. matching events will be dropped.

Also note that either `match` _or_ `pattern` must be set in the config; they cannot both be empty. If both are set, the stage will apply the `match` and ignore the `pattern` (the most strict rule).

The `action` for this stage is `filter`

The `config` block accepts the following values

```yaml
- name: Continue if no match
  action: filter
  config:
    # REQUIRED, a field name or list of field names to filter on by value from the input JSON. If given multiple fields, only one of those fields must match for the run to stop.
    field:
      - field1
      - field2
    # OPTIONAL, a boolean that controls whether matching data is dropped or accepted. The default is true, i.e. drop
    reject_matches: false
    # OPTIONAL, a string that must exactly match the input JSON. If this config is not set, the "pattern" config MUST be set
    match: some string
    # OPTIONAL, a regex pattern that must match the input JSON. If this config is not set, the "match" config MUST be set
    pattern: /some regex/
  out:
    - ...
```

As an example, the following input would be accepted:

```json
{
  "field1": "spooky cat",
  "field2": "some string"
}
```

and the following input would be dropped:

```json
{
  "field1": "spooky cat",
  "field2": "foobar"
}
```

### JSON decode

This stage will take binary input and decode it into a JSON string.

The `action` for this stage is `json_decode`.

The `config` block is not necessary for this stage

### Put Kinesis record

This stage will take an input and put it onto a Kinesis stream as a new record. _NOTE_ this stage is distinct from the previous trigger stage, also called "kinesis".

The `action` for this stage is `put_kinesis`.

The `config` block accepts the following values

```yaml
- name: Put kinesis record
  action: kinesis
  config:
    # REQUIRED, the name of the Kinesis stream to put a record on
    stream: some string
    # OPTIONAL, a string that corresponds to partitioning of the record
    partition_key: some string
  out:
    - ...
```

### Merge data

Add static or secret data into the JSON input and send the combined JSON object as output.

The `action` for this stage is `merge_data`

The `config` block accepts the following values

```yaml
- name: Add secret to data
  action: merge_data
  config:
    # REQUIRED, an object of data to merge into the input JSON object
    data:
      newfield1: some string
      newfield2: ${secrets.some_secret}
  out:
    - ...
```

### Slack

A stage that sends the event to a Slack channel webhook

The `action` for this stage is `slack`

The `config` block accepts the following values

```yaml
- name: Message Slack
  action: slack
  config:
    # REQUIRED, the URL of the slack webhook
    url: https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX
    # OPTIONAL, a value to change the color of the message displayed in slack. Default is "info"
    level: error | warn | info | success
  out:
    - ...
```

### Webhook

A stage that sends its input to a specified URL in an HTTPs request.

The `action` for this stage is `webhook`

The `config` block accepts the following values

```yaml
- name: Publish to API
  action: webhook
  config:
    # REQUIRED, the url to invoke with an HTTPs request
    url: https://some.other.url
    # OPTIONAL, the HTTP method to use in the request. Default is "post"
    method: head | get |delete | trace | options | post | put | patch
    # OPTIONAL, additional HTTP headers to use with the request.
    headers:
    - Content-Type: application/json
    - Bearer: ${secrets.API_KEY}
    # OPTIONAL, include the entire input data as the body of the HTTP request. Default is true
    event_body: true | false
    # OPTIONAL, additional query params to be included in the HTTP request
    params:
      newfield1: some string
```

### Zip

A stage that collects inputs from multiple stages and combines them into a single output. The zip stage buffers inputs into a queue until it has received at least one input from each preceding stage that lists the zip stage as an output. Once an input has been received from all stages, it takes the top input from each queue, adds each to a list, and sends the list on to the `out` stages.

There is one buffer for every preceding stage that sends inputs to zip. The behavior of the queue buffer is first-in-first-out (FIFO). This means inputs are paired together in the order they are received. Each buffer has a limit set in the stage `config`. Once the buffer limit is reached for a given stage, further inputs are rejected by zip, which will trigger retry/backoff behavior in the previous stage.

The `action` for this stage is `zip`.

The `config` block accepts the following values

```yaml
- name: Zip inputs together
  action: zip
  config:
    # OPTIONAL, the number of inputs to buffer per incoming stage. Default is 100
    event_limit: 10
```

For example, see the following Flow, where the output of two stages are zipped together:

```yaml
- name: Simple Flow with a zip
- stages:
    - name: Read from kinesis
      action: kinesis
      config:
        stream: some_kinesis_string_name
        cold_start: trim_horizon
      out:
        - Zip inputs together

    - name: Receive HTTP data
      action: webhook
      config:
        host: some.host.name
      out:
        - Zip inputs together

    - name: Zip inputs together
      action: zip
      config:
        event_limit: 3
      outs:
        - Send combined webhook

    - name: Send combined webhook
```

Given the following timline of inputs:

1. The "Read from kinesis" stage receives input {Ka} and {Kb} from the kinesis stream. Both are sent to the zip stage
1. The "Receive HTTP data" stage receives input {Wa}, and sends it to the zip stage.
1. The "Zip inputs together" stage, having received one input from both the kinesis and webhook stage (which lists zip as an `out`), sends [{Ka}, {Wa}] to the stage "Send cobined webhook"
1. The "Read from kinesis" stage receives input {Kc} and {Kd} from the kinesis stream. {Kd} is rejected by the zip stage, because the buffer limit is reached for kinesis. This will trigger a retry in "Read from kinesis".
1. The "Receive HTTP data" stage receives input {Wb}, and sends it to the zip stage.
1. The "Zip inputs together" stage sends [{Kc}, {Wb}] to the stage "Send cobined webhook"
1. The "Read from kinesis" stage resends {Kd} to zip, this time successfully as there is space in the buffer

### Reduce

A stage that takes in a list of events and creates emits a single map. The reduce stage can either concatanate all of the data into a field in the map, or merge the data together.

Either `concat_field` must be set, or `merge` must be `true`. However, these are mutally exclusive and `concat_field` cannot be set when `merge` is also `true.

The `action` for this stage is `reduce`

The `config` block accepts the following values

```yaml
- name: Combine data events
  action: reduce
  config:
    # OPTIONAL, the field to combine all events under
    concat_field: event_field_name
    # OPTIONAL, whether to merge all events into a single map. Defaults to false
    merge: true | false
  out:
    - ...
```
