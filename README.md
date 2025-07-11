# `deployd`: Self-Hosted Deno Deploy

Self-hosted version of Deno Deploy. `deployd` is the same edge-compute runtime that powers Deno Deploy, but running entirely in your infrastructure. It runs serverless functions and full JavaScript/TypeScript applications with production-grade sandboxing and performance using Firecracker VMs.

**Status:** Beta release, running production traffic.

## Quick Start

```bash
docker run -it -p 8080:8080 --privileged ghcr.io/denoland/deployd
docker run -it -p 8080:8080 --privileged ghcr.io/denoland/deployd --help
```

## Key Features

- Serverless JavaScript/TypeScript applications that run in isolated Firecracker VMs
- Deployments via Deno Deploy's managed control plane ([app.deno.com](http://app.deno.com/)) or directly to your own instances.
- Horizontal scaling with automatic node discovery.
- Built-in observability with OpenTelemetry.

## Requirements

- Docker with privileged mode, or `CAP_SYS_ADMIN` capability and tun device.
- Requires KVM to be available for full performance
- S3-compatible object storage

## Usage

Basic usage:

```bash
docker run -it -p 8080:8080 --privileged ghcr.io/denoland/deployd
```

With S3 storage:

```bash
docker run -it -p 8080:8080 --privileged \
  -e AWS_ACCESS_KEY_ID=your_key \
  -e AWS_SECRET_ACCESS_KEY=your_secret \
  ghcr.io/denoland/deployd --s3-bucket=your-bucket
```

Or if you don’t want to use S3 or just running a single instance you can use a persistent volume:

```bash
docker run -it -p 8080:8080 -v /path/to/data:/data --privileged ghcr.io/denoland/deployd
```

## Deploying Applications

Deploy applications by:

- Linking GitHub repositories via [app.deno.com](http://app.deno.com/), or
- Creating playgrounds on [app.deno.com](http://app.deno.com/), or
- Using the `deno deploy` command in your project, or
- Directly uploading code, avoiding Deno Deploy's control plane entirely
(documentation available upon request)

## Storage Options

Deployments can be stored in three ways:

- S3 Storage (recommended for production): Set AWS credentials and specify
bucket with `--s3-bucket` flag.
- Local Storage: Mount `/data` directory to persist deployments locally.
- Ephemeral Storage (default): Deployments stored in memory, lost when
container stops.

## Configuration

Configuration options:

| **Flag** | **Environment Variable** | **Description** |
| --- | --- | --- |
| `--s3-bucket=<bucket>` | `S3_BUCKET` | S3 bucket name |
| `--aws-access-key-id` | `AWS_ACCESS_KEY_ID` | S3 access key |
| `--aws-secret-access-key` | `AWS_SECRET_ACCESS_KEY` | S3 secret key |
| `--aws-region` | `AWS_REGION` | S3 region |
| `--s3-endpoint` | `S3_ENDPOINT` | S3-compatible endpoint |
| `--isolate-threads` | `ISOLATE_THREADS` | Number of isolate threads (auto-detected if unspecified, min 8) |

## Internal Redis API

Cluster state is exposed on port 3911 (`-p 3911:3911`) using the RESP2 protocol:

```bash
# list all nodes
$ redis-cli -p 3911 KEYS 'node/*'
 1) "node/aws-eu-west-1.demo.deno-cluster.net/b193727c9ee2a868c5ae2563aa58b84f/ip"
 2) "node/aws-eu-west-1.demo.deno-cluster.net/b193727c9ee2a868c5ae2563aa58b84f/num_active_isolates"
 3) "node/aws-eu-west-1.demo.deno-cluster.net/b193727c9ee2a868c5ae2563aa58b84f/settings"
 4) "node/aws-eu-west-1.demo.deno-cluster.net/b193727c9ee2a868c5ae2563aa58b84f/token"
 5) "node/aws-eu-west-1.demo.deno-cluster.net/b193727c9ee2a868c5ae2563aa58b84f/update_time"
 6) "node/aws-eu-west-1.demo.deno-cluster.net/b193727c9ee2a868c5ae2563aa58b84f/used_balloon_budget_mib"
 7) "node/aws-us-east-2.demo.deno-cluster.net/c21a915f54091d23fa8d4c88052685d4/ip"
 8) "node/aws-us-east-2.demo.deno-cluster.net/c21a915f54091d23fa8d4c88052685d4/num_active_isolates"
 9) "node/aws-us-east-2.demo.deno-cluster.net/c21a915f54091d23fa8d4c88052685d4/settings"
10) "node/aws-us-east-2.demo.deno-cluster.net/c21a915f54091d23fa8d4c88052685d4/token"
11) "node/aws-us-east-2.demo.deno-cluster.net/c21a915f54091d23fa8d4c88052685d4/update_time"
12) "node/aws-us-east-2.demo.deno-cluster.net/c21a915f54091d23fa8d4c88052685d4/used_balloon_budget_mib"
```

Then, for example, inspect the active isolates on a `deployd` instance:

```bash
$ redis-cli -p 3911 GET "node/aws-eu-west-1.demo.deno-cluster.net/b193727c9ee2a868c5ae2563aa58b84f/num_active_isolates"
"19"
```

There’s a lot of stuff here, documentation provided upon request.

## Observability

`deployd` emits traces, metrics, and logs through OpenTelemetry out of the box. Telemetry from your applications can optionally be viewed through Deno Deploy's managed control plane ([app.deno.com](http://app.deno.com/)), or sent to your own collector by setting standard OTEL environment variables: `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_PROTOCOL`, `OTEL_RESOURCE_ATTRIBUTES`,`OTEL_SERVICE_NAME`.

See https://opentelemetry.io/docs/languages/sdk-configuration/otlp-exporter/

## Who is `deployd` for?

- Compliance-sensitive organizations that must keep the data plane inside
their own public cloud account or on-prem servers
- Platform builders who want to run serverless JavaScript code as a service
- AI platforms who need a place to run LLM-generated code
- Any team executing untrusted user code and requiring VM-grade isolation

## Support

`deployd` is in active development. For support and feedback: [deploy@deno.com](mailto:deployd@deno.com)

Commercial terms will be available closer to the public release.
