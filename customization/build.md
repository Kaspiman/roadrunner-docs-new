# Building a Server

Developers can take advantage of the customization options available with RoadRunner to create a server optimized
for their particular project.

**This can include:**

- adding custom plugins,
- forking existing ones to make changes,
- or building a lightweight server with only the necessary plugins.

We created a tool called **Velox** that lets developers build a RoadRunner server binary. It uses a configuration file
to determine which plugins and repositories are required for building a RoadRunner server binary.

## Configuration

The configuration file is written in TOML format and contains a list of repositories to add to the build. For each
repository, you can specify the owner and version. You can also add private repositories from GitHub or Gitlab, and
authenticate with access tokens.

{% hint style="info" %}
To download all the required plugins for RoadRunner, you need a GitHub token. If you try to download plugins without a
token, anonymous access is limited to 50 requests per hour. You can read more about these limits on
the [Rate limits for GitHub Apps](https://docs.github.com/en/apps/creating-github-apps/setting-up-a-github-app/rate-limits-for-github-apps)
page.
{% endhint %}

**Here is an example of a configuration file:**

{% code title="velox_rr_2024.toml" %}

```toml
[velox]
build_args = [
  '-trimpath',
  '-ldflags',
  '-s -X github.com/roadrunner-server/roadrunner/v2024/internal/meta.version=${VERSION} -X github.com/roadrunner-server/roadrunner/v2024/internal/meta.buildTime=${TIME}'
]

[roadrunner]
ref = "v2024.1.0"

[github]
[github.token]
token = "${RT_TOKEN}"

# ref -> master, commit or tag
[github.plugins]
# LOGS
appLogger = { ref = "v4.4.0", owner = "roadrunner-server", repository = "app-logger" }
logger = { ref = "v4.4.0", owner = "roadrunner-server", repository = "logger" }
lock = { ref = "v4.7.0", owner = "roadrunner-server", repository = "lock" }
rpc = { ref = "v4.4.0", owner = "roadrunner-server", repository = "rpc" }

# CENTRIFUGE BROADCASTING PLATFORM
centrifuge = { ref = "v4.9.0", owner = "roadrunner-server", repository = "centrifuge" }

# WORKFLOWS ENGINE
temporal = { ref = "v4.7.0", owner = "temporalio", repository = "roadrunner-temporal" }

# METRICS
metrics = { ref = "v4.3.0", owner = "roadrunner-server", repository = "metrics" }

# HTTP + MIDDLEWARE
http = { ref = "v4.7.0", owner = "roadrunner-server", repository = "http" }
gzip = { ref = "v4.3.0", owner = "roadrunner-server", repository = "gzip" }
prometheus = { ref = "v4.3.0", owner = "roadrunner-server", repository = "prometheus" }
headers = { ref = "v4.4.0", owner = "roadrunner-server", repository = "headers" }
static = { ref = "v4.3.0", owner = "roadrunner-server", repository = "static" }
proxy = { ref = "v4.3.0", owner = "roadrunner-server", repository = "proxy_ip_parser" }

# OpenTelemetry
otel = { ref = "v4.5.0", owner = "roadrunner-server", repository = "otel" }

# SERVER
server = { ref = "v4.8.0", owner = "roadrunner-server", repository = "server" }

# SERVICE aka lightweit systemd
service = { ref = "v4.7.0", owner = "roadrunner-server", repository = "service" }

# JOBS
jobs = { ref = "v4.9.0", owner = "roadrunner-server", repository = "jobs" }
amqp = { ref = "v4.11.0", owner = "roadrunner-server", repository = "amqp" }
sqs = { ref = "v4.8.0", owner = "roadrunner-server", repository = "sqs" }
beanstalk = { ref = "v4.8.0", owner = "roadrunner-server", repository = "beanstalk" }
nats = { ref = "v4.8.0", owner = "roadrunner-server", repository = "nats" }
kafka = { ref = "v4.6.0", owner = "roadrunner-server", repository = "kafka" }

# KV
kv = { ref = "v4.6.0", owner = "roadrunner-server", repository = "kv" }
boltdb = { ref = "v4.9.0", owner = "roadrunner-server", repository = "boltdb" }
memory = { ref = "v4.8.0", owner = "roadrunner-server", repository = "memory" }
redis = { ref = "v4.4.0", owner = "roadrunner-server", repository = "redis" }
memcached = { ref = "v4.5.0", owner = "roadrunner-server", repository = "memcached" }

# FILESERVER (static files)
fileserver = { ref = "v4.3.0", owner = "roadrunner-server", repository = "fileserver" }

# gRPC plugin
grpc = { ref = "v4.8.0", owner = "roadrunner-server", repository = "grpc" }

# HEALTHCHECKS + READINESS CHECKS
status = { ref = "v4.6.0", owner = "roadrunner-server", repository = "status" }

# TCP for the RAW TCP PAYLOADS
tcp = { ref = "v4.5.0", owner = "roadrunner-server", repository = "tcp" }

[gitlab]
[gitlab.token]
# api, read-api, read-repo
token = "${GL_TOKEN}"

[gitlab.endpoint]
endpoint = "https://gitlab.com"

[gitlab.plugins]
# ref -> master, commit or tag
test_plugin_1 = { ref = "main", owner = "rustatian", repository = "36405203" }
test_plugin_2 = { ref = "main", owner = "rustatian", repository = "36405235" }

[log]
level = "debug"
mode = "development"
```

{% endcode %}

{% hint style="info" %}
You can find the latest version of the example configuration file in
the [official repository](https://github.com/roadrunner-server/velox/blob/master/velox_rr_v2023.toml).
{% endhint %}

{% hint style="warning" %}
When using official plugins for RoadRunner, it is recommended avoid using the `master` branch as it may contain
unstable code. Instead, use tags with the same major version (e.g., `logger:v4.x.x` + `amqp:v4.x.x`, but
not `logger:v4.0.0` + `amqp:v3.0.5`). Please note that the currently supported plugin version is `v4.x.x`, and the
supported RoadRunner version is `v2024.x.x`.

Failure to follow these guidelines may result in compatibility issues and
other problems. Please pay close attention to your configuration file to ensure proper use of plugins.
{% endhint %}

You can use environment variables in the configuration file. This is useful when you want to keep the configuration file
in the repository, but you don't want to expose your tokens or just want to pass them as arguments to the `vx` command.

Here are the list of environment variables from the example above:

| Variable      | Description                                                                |
|---------------|----------------------------------------------------------------------------|
| `${GL_TOKEN}` | GitLab token.                                                              |
| `${RT_TOKEN}` | GitHub token.                                                              |
| `${VERSION}`  | RR version to write into the binary (will be shown with `./rr --version`). |
| `${TIME}`     | Build time (will be shown with `./rr --version`).                          |

{% hint style="info" %}
Keep in mind to set the latest stable version in the `${VERSION}` env variable. You may also use `${TIME}` env
variable to write the build time in the output binary.
{% endhint %}

### Options

| Option         | Description                                                                                                                                                                                                                  |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **ref**        | Tag, commit hash or branch name.                                                                                                                                                                                             |
| **owner**      | Repository owner (might be the user or organization).                                                                                                                                                                        |
| **repository** | Repository name.                                                                                                                                                                                                             |
| **folder**     | If the plugin is in some folder in your repository, you may specify it via this configuration option. <br/>For example: `cache = { ref = "v1.6.18", owner = "darkweak", repository = "souin", folder="plugins/roadrunner" }` |
| **replace**    | Go.mod [replace directive](https://go.dev/ref/mod#go-mod-file-replace).                                                                                                                                                      |

### Private repositories

- Make sure the `ssh-agent` is running and the ssh key has been
  added: [link](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- Exclude your organization package prefix from the Go environment variables:

{% code %}

```bash
go env -w GOPRIVATE="gitlab/github.com/<company_name>/*"
go env -w GONOSUMDB="gitlab/github.com/<company_name>/*"
```

{% endcode %}

## Building

{% tabs %}

{% tab title="Docker" %}

Using the Docker image simplifies the build process by automatically building the RoadRunner binary and storing it in
the `/usr/bin/` folder. This eliminates the need to install Golang or other dependencies on your computer. Once the
build is complete, Docker will automatically start the RoadRunner server.

**Here is an example of Dockerfile:**

{% code title="Dockerfile" %}

```dockerfile
# https://docs.docker.com/buildx/working-with-buildx/
# TARGETPLATFORM if not empty OR linux/amd64 by default
FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/roadrunner-server/velox:latest as velox

# app version and build date must be passed during image building (version without any prefix).
# e.g.: `docker build --build-arg "APP_VERSION=1.2.3" --build-arg "BUILD_TIME=$(date +%FT%T%z)" .`
ARG APP_VERSION="undefined"
ARG BUILD_TIME="undefined"

# copy your configuration into the docker
COPY velox_rr_2024.toml .

# we don't need CGO
ENV CGO_ENABLED=0

# RUN build
RUN vx build -c velox_rr_2024.toml -o /usr/bin/

FROM --platform=${TARGETPLATFORM:-linux/amd64} php:8.3-cli

# copy required files from builder image
COPY --from=velox /usr/bin/rr /usr/bin/rr

# use roadrunner binary as image entrypoint
CMD ["/usr/bin/rr"]
```

{% endcode %}

{% endtab %}

{% tab title="Go" %}

You can use the `go install` command to download Velox.

{% hint style="warning" %}
To download Velox and build an application server you need [Golang 1.22+](https://golang.org/dl/) on you local server.
{% endhint %}

{% code title="go install" %}

```bash
go install github.com/roadrunner-server/velox/v2024/cmd/vx@latest
```

{% endcode %}

After the binary has been downloaded, you can build the application server:

{% code title="vx build" %}

```bash
vx build -c velox_rr_2024.toml -o ~/Downloads
```

{% endcode %}

| Option | Description                     |
|--------|---------------------------------|
| `-c`   | path to the configuration       |
| `-o`   | path where to put the RR binary |

{% endtab %}

{% tab title="Downloading Binary" %}

To build the application server, you need to download Velox binary
the [GitHub releases page](https://github.com/roadrunner-server/velox/releases) and unpack it to your `PATH`.

{% hint style="warning" %}
To build an application server you need [Golang 1.22+](https://golang.org/dl/) on you local server.
{% endhint %}

After the binary has been downloaded, you can build the application server:

{% code title="vx build" %}

```bash
vx build -c velox_rr_2024.toml -o ~/Downloads
```

{% endcode %}

| Option | Description                     |
|--------|---------------------------------|
| `-c`   | path to the configuration       |
| `-o`   | path where to put the RR binary |

{% endtab %}

{% endtabs %}

## Video tutorials

### How to write a plugin

{% embed url="https://www.youtube.com/watch?v=h5PPvc_YOtg" %}

### `v2023.x.x` update

{% embed url="https://www.youtube.com/watch?v=w_uxFhdinvU" %}

### Velox configuration

{% embed url="https://www.youtube.com/watch?v=sddi_lh7ePo" %}

## Third-party and deprecated plugins

- [souin, third-party](https://github.com/darkweak/souin/tree/master/plugins/roadrunner)
- [reload, deprecated](https://github.com/roadrunner-server/reload)

## Known limitation

- At the moment only GitHub and GitLab repositories are supported.
