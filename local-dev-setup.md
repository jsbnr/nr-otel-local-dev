# Local development setup guide

Want to develop your own OpenTelemetry modules? This guide explains how to get started building your own [custom instance of the OpenTelemetry collector]()https://opentelemetry.io/docs/collector/custom-collector/, run it locally and drive traffic to it with a test app allowing you to get on with develping your custom modules. 

> These instructions are intended for Mac users, adjust accordingly for other systems.


## Summary of steps

1. Make sure GO is installed
2. Install the OpenTelemetry Collector Builder tool
3. Configure and build a collector distribution
4. Configure and run a collector
5. Install and run the test application
6. Confirm its all working


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


## Step 3: Configure and build a collector distribution
In this step we'll build a new version of your custom collector (called a 'distribution'). The distribution contains all the modules you may use in your collector.

The `builder` tool takes a yaml manifest configuration defining what modules should be included in your distribution. Here we'll configure a really basic collector distribution with just a few modules to get things running. Create a file called `otelcol-builder.yaml` and paste in the following configuration:

**otelcol-builder.yaml:**
```yaml
dist:
  name: otelcol-dev
  description: Basic OTel Collector distribution for Developers
  output_path: ./otelcol-dev
  otelcol_version: 0.75.0 #set this to the version of the builder you're running

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


Next build the collector distribution defined by running the `builder` tool:

```bash
builder --config=otelcol-builder.yaml 
```

This should generate a folder called `otelcol-dev` (as thats what we called it in the `name` attribute) containing the collector distribution.

> You may get a warning *"You're building a distribution with non-aligned version of the builder"*.  Ideally you need to set the configuration `otelcol_version` attribute  on line #5 to the same version as your `builder` tool. Helpfully the info message tells you what version that is: `{"builder-version": "X.XX.XX"}`


## Step 4: Configure and run a collector
We've built a distribution, now we need to configure and run an instance of it. The configuration of a collector is again specified using yaml. 

In this example collector configuration we set up a trace, metric and log pipeline that receives the data from an `otlp` receiver and logs the data to screen using the `logging` exporter. It also exports logs *only* to New Relic via the `otlp` exporter.

Create a file called `otelcol.yaml` and paste in the following code remembering to update the `api-key` attribute with an ingest license key for your New Relic account.


**otelcol.yaml:**
```yaml
receivers:
  otlp:
    protocols:
      grpc: # 4317
      http: # 4318

processors:
  batch:
  
exporters:
  otlp:
    endpoint: "https://otlp.nr-data.net:4318"  # or for EU: eu.otlp.nr-data.net
    headers:
      api-key: .....NRAL
  logging:
    verbosity: detailed


extensions:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: []
      exporters: [logging]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp, logging]
  telemetry:
    logs:
      level: debug
```

> EU datacenter? If your New Relic account is in the EU data center be sure to change the `endpoint` attribute to `https://eu.otlp.nr-data.net:4318`


To run this configuration using your custom distribution run the following (CTRL+C to quit):
```
./otelcol-dev/otelcol-dev --config=./otelcol.yaml
```

## Step 5: Install and run the test application
We'll use a small [test application](https://github.com/dpacheconr/otel-generator-demo) to generate some traffic to the collector. You will need [docker](https://www.docker.com/products/docker-desktop/) installed and running to run the app.

#### Clone the app
In another terminal window clone the demo app and switch to the directory:

```
git clone https://github.com/dpacheconr/otel-generator-demo.git
cd otel-generator-demo
```

#### Configure the demo app
Edit the file `loggen/main.py` and change the string on the last line to something of your own you will recognise (or you can leave it as it is `PASTE_JSON_HERE`).

#### Run the app
Build and run the app using docker-compose:
```
docker-compose build
docker-compose up -d
```

> You can shutdown the app with:
> ````
> docker-compose down
> ```


## Step 6:  Confirm its all working
Once the application has started up you should see some data being logged to screen on the collector window. A short while later you should also start to see logs appearing in the Logs section of New Relic. If you're having trouble finding the logs in New Relic add the string you added to main.py to the filter or add this filter to the filter bar: `"otelcol.test":"loggen"`

> The data is flowing from the application running in docker to your custom collector, passing through the collector pipeline nd then being sent to the New Relic otlp endpoint.