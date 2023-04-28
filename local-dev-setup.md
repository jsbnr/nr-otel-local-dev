# Local development setup guide

Want to develop your own OpenTelemetry components? This guide explains how to get started building your own [custom instance of the OpenTelemetry collector]()https://opentelemetry.io/docs/collector/custom-collector/, run it locally and drive traffic to it with a test app allowing you to get on with develping your custom components. 

> These instructions are intended for Mac users, adjust accordingly for other systems.


## Summary of steps

1. Make sure GO is installed
2. Install the OpenTelemetry Collector Builder tool
3. Configure, build and run a collector.
4. Install and run the test application
5. Confirm its all working


## Step 1: GO Installation
You probably already have GO installed already, if not install following [these instructions](https://go.dev/doc/install) or via [brew](https://formulae.brew.sh/formula/go).

```console
$ go version
go version go1.19.2 darwin/arm64
```


