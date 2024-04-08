# Environment configuration

RoadRunner offers the capability to set and manage environment variables for workers. This capability allows you to set
specific environment variables when workers are initialized, providing a flexible and organized method for managing
application configurations.

## Setting Env Variables

You can set environment variables for PHP workers by defining them in the `server.env` section of the RoadRunner
configuration file. These variables will be applied to all workers when they are started by the server.

**Here's an example:**

{% code title=".rr.yaml" %}

```yaml
server:
  command: "php worker.php"
  env:
     APP_RUNTIME: prod
```

{% endcode %}

In this example, when RoadRunner starts a PHP worker, it will set the `APP_RUNTIME` environment variable to `prod`.

{% hint style="warning" %}
All environment variable keys will be automatically converted to uppercase.
{% endhint %}

## Default Env Values in PHP Workers

RoadRunner comes with a set of default environment (ENV) values that facilitate proper communication between the PHP
process and the server. These values are automatically available to workers and can be used to configure and manage
various aspects of the worker's operation.

**Here's a list of the default ENV values provided by RoadRunner:**

| Key            | Description                                                                                            |
|----------------|--------------------------------------------------------------------------------------------------------|
| **RR_MODE**    | Identifies what mode worker should work with (`http`, `temporal`, `grpc`, `jobs`, `tcp`, `centrifuge`) |
| **RR_RPC**     | Contains RPC connection address when enabled.                                                          |
| **RR_RELAY**   | `pipes` or `tcp://...`, depends on server relay configuration.                                         |
| **RR_VERSION** | RoadRunner version started the PHP worker (minimum `2023.1.0`)                                         |

These default environment values can be used within your PHP worker to configure various settings and adapt the worker's
behavior according to the specific requirements of your application.

## What's Next?

1. [Environment variables](../intro/config.md) - Learn how to use environment variables in your RoadRunner
   configuration.
