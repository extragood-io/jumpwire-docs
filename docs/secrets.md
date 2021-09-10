---
layout: default
title: Secrets
nav_order: 3
---

## Secrets

A Secret is a variable in the JumpWire system. It can be referenced in Flow stage configurations, and gets inserted into the Flow definition before execution.

Use Secrets to store any values that should be externalized from individual Flow definitions. Common examples include host names, authentication parameters, or values that are repeatedly used. It is especially recommended to use Secrets to store authentication information such as password or bearer tokens.

### Secret syntax

Secrets are referenced by their name, which must be unique across an installation. To reference a secret, use the syntax `${secret.SECRET_NAME}` in place of the value you wish to substitute with the secret.

For example, the following Flow stage uses an externalized host name for sending data to an external API:

```yaml
- name: send webhook
  action: webhook
  config:
    url: ${secret.my-api-host}
    method: post
    event_body: true
```

with the value of the secret named `my-api-host`:

```
https://stage.jumpwire.ai/demo/webhook
```

### Secret requirements

Secret names must be unique, non-empty, and contain only characters, underscores or dashes. Secret values can contain any arbitrary text data.

### Secret backend

Coming soon
{: .label .label-yellow }

Secrets are stored by default in JumpWire's database. This can be overridden by configuration to use other storage engines, such as AWS Secrets Manager, AWS System Manager Parameter Store, or Hashicorp Vault.
