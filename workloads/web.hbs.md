# Use web workloads

This topic tells you how to use the `web` workload type in Tanzu Application Platform
(commonly known as TAP).

## <a id="overview"></a>Overview

The `web` workload type allows you to deploy web applications on Tanzu Application Platform.
Using an application workload specification, you can turn source code into a scalable, stateless
application that runs in a container with an automatically-assigned URL.
This type of application is often called "serverless", and is deployed using Knative.

The `web` workload type is suitable for modern stateless web applications that follow
[the twelve-factor app](https://12factor.net) methodology and have the following characteristics:

- Perform all work through HTTP requests, including gRPC and WebSocket.
- Do not perform work except when processing a request.
- Start up quickly.
- Store state in external databases (do not store state locally).

Applications using the `web` workload type have the following features:

- Automatic request-based scaling, including scale-to-zero.
- Automatic URL provisioning and optional certificate provisioning.
- Automatic health check definitions, if not provided by a convention.
- Blue-green application roll outs.

When creating a workload with the `tanzu apps workload create` command, you can use the
`--type=web` argument to select the `web` workload type.
For more information, see the [Use the web Workload Type](#using) later in this topic.
You can also use the `apps.tanzu.vmware.com/workload-type:web` label in the
YAML workload description to support this deployment type.

## <a id="using"></a> Use the `web` workload type

The `tanzu-java-web-app` workload used in [Deploy an app on Tanzu Application Platform](../getting-started/deploy-first-app.hbs.md) in the Get started guide is a good match for the `web`
workload type.

This is because it serves HTTP requests and does not perform any background processing.

You can experiment with the differences between the `web` and `server` workload types
by changing the workload type:

```console
tanzu apps workload apply tanzu-java-web-app --type=server
```

After changing the workload type to `server`, the application does not autoscale or expose an
external URL. For more information about the server workload type, see [Using Server workloads](server.hbs.md).

Switch back to the `web` workload by running:

```console
tanzu apps workload apply tanzu-java-web-app --type=web
```

Use this to test which applications can function well as serverless web applications,
and which are more suited to the `server` application style.
