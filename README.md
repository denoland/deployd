deployd: Self-Hosted Edge Runtime

Self-hosted version of Deno Deploy. Runs serverless functions and full
Javascript/Typescript applications with production-grade sandboxing and
performance using Firecracker VMs.

## Quick Start

```bash
docker run -it -p 8080:8080 --privileged ghcr.io/denoland/deployd
docker run -it -p 8080:8080 --privileged ghcr.io/denoland/deployd --help
```

## Operation

- JavaScript/TypeScript code runs in isolated Firecracker VMs
- Deployments managed via app.deno.com or `deno deploy` command
- S3 integration for deployment artifact storage

## Deploying applications

Deploy applications by:
- Linking GitHub repositories via app.deno.com
- Creating playgrounds on app.deno.com
- Using `deno deploy` command in your project (requires Deno 2.4.1+)

On first run, deployd displays registration steps for your Deno Deploy account.

## Storage

Deployments can be stored in three ways:

- S3 Storage (recommended for production): Set AWS credentials and specify
  bucket with --s3-bucket flag
- Local Storage: Mount `/data` directory to persist deployments locally
- Ephemeral Storage (default): Deployments stored in memory, lost when container
  stops

## Configuration

Storage configuration:
```
AWS_ACCESS_KEY_ID                 # S3 access key
AWS_SECRET_ACCESS_KEY             # S3 secret key  
S3_ENDPOINT                       # Optional S3-compatible endpoint
--s3-bucket=<bucket>              # S3 bucket name
```

Performance tuning:
```
--isolate-threads=<n>             # Number of isolate threads
                                  # (auto-detected if unspecified, min 8)
```

## Usage

Basic usage
```bash
docker run -it -p 8080:8080 --privileged ghcr.io/denoland/deployd
```

Alternative without --privileged:
```bash
docker run -it -p 8080:8080 --cap-add CAP_SYS_ADMIN --device /dev/net/tun ghcr.io/denoland/deployd
```

With S3 storage:
```bash
docker run -it -p 8080:8080 --privileged \
  -e AWS_ACCESS_KEY_ID=your_key \
  -e AWS_SECRET_ACCESS_KEY=your_secret \
  ghcr.io/denoland/deployd --s3-bucket=your-bucket
```

With local storage:
```bash
docker run -it -p 8080:8080 -v /path/to/data:/data --privileged ghcr.io/denoland/deployd
```

With custom thread count:
```bash
docker run -it -p 8080:8080 --privileged ghcr.io/denoland/deployd --isolate-threads=16
```

## Requirements

- Docker with privileged mode, or `CAP_SYS_ADMIN` capability and tap device
- Port 8080 available for web interface
- For S3 storage: valid AWS credentials or S3-compatible storage access
