# Local development setup guide

Want to develop your own OpenTelemetry modules? This guide explains how to get started building your own [custom instance of the OpenTelemetry collector]()https://opentelemetry.io/docs/collector/custom-collector/, run it locally and drive traffic to it with a test app allowing you to get on with develping your custom modules. 

> These instructions are intended for Mac users, adjust accordingly for other systems.


## Summary of steps

1. Make sure GO is installed
2. Install the OpenTelemetry Collector Builder tool
3. Configure, build and run a collector.
4. Install and run the test application
5. Confirm its all working


## Step 1: GO Installation
You probably already have GO installed already, if not install following [these instructions](https://go.dev/doc/install) or via [brew](https://formulae.brew.sh/formula/go). Check your version like this (anythign recent should do):

```console
$ go version
go version go1.19.2 darwin/arm64
```

## Step 2: Install the OpenTelemetry Collector Builder (ocb) tool
This [helper tool](https://github.com/open-telemetry/opentelemetry-collector/tree/main/cmd/builder) builds a custom OpenTelemetry Collector binary based on a given configuration. We'll use this to package up our collector to run.

You can install the OCB tool using the installer or by downloading a [pre-built release](https://github.com/open-telemetry/opentelemetry-collector-releases/releases).

To install using the installer run:

```
GO111MODULE=on go install go.opentelemetry.io/collector/cmd/builder@latest
```

This installs a binary `builder` in `$HOME/go/bin/`. To easily use this add it to your path:

```bash
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
export PATH=$PATH:$GOROOT/bin
```

Check it works by checking the version, in this case we have the dev version installed:

```console
$ builder version
ocb version dev
```

> For some reason the tool is sometimes named `ocb` instead of `builder`. Just use `builder` wheever you see in `ocb` in the docs. We'll use `builder` in the rest of this guide.

> If you install using a release you'll need to rename and make the file executable as well as placing it in your path. More information [here](https://github.com/open-telemetry/opentelemetry-collector/tree/main/cmd/builder)


## Step 3: Configure, build and run a collector
In this step we'll build a new version of your custom collector and get it running. 

The `builder` tool takes a yaml configuration defining what modules should be included in your collector. Here we'll configure a really basic collector with just a few modules to get things running. Create a file called `otelcol-builder.yaml` and paste in the following configuration:

*otelcol-builder.yaml*
```yaml
dist:
  name: otelcol-dev
  description: Basic OTel Collector distribution for Developers
  output_path: ./otelcol-dev
  otelcol_version: 0.75.0

exporters:
  - gomod:
      go.opentelemetry.io/collector/exporter/loggingexporter v0.75.0
  - gomod:
      github.com/open-telemetry/opentelemetry-collector-contrib/exporter/jaegerexporter v0.75.0
  - gomod:
      github.com/open-telemetry/opentelemetry-collector-contrib/exporter/prometheusexporter v0.75.0
  - gomod: 
      github.com/open-telemetry/opentelemetry-collector-contrib/exporter/zipkinexporter v0.75.0
  - gomod:
      go.opentelemetry.io/collector/exporter/otlpexporter v0.75.0

extensions:
  - gomod:
      go.opentelemetry.io/collector/extension/zpagesextension v0.75.0
  - gomod:
      github.com/open-telemetry/opentelemetry-collector-contrib/extension/healthcheckextension v0.75.0
  - gomod:
      github.com/open-telemetry/opentelemetry-collector-contrib/extension/pprofextension v0.75.0

processors:
  - gomod:
      go.opentelemetry.io/collector/processor/batchprocessor v0.75.0
  - gomod:
      github.com/open-telemetry/opentelemetry-collector-contrib/processor/transformprocessor latest
  - gomod:
      github.com/open-telemetry/opentelemetry-collector-contrib/processor/attributesprocessor latest
  - gomod:
      github.com/open-telemetry/opentelemetry-collector-contrib/processor/filterprocessor latest
  - gomod:
      github.com/open-telemetry/opentelemetry-collector-contrib/processor/logstransformprocessor latest
receivers:
  - gomod:
      go.opentelemetry.io/collector/receiver/otlpreceiver v0.75.0
```