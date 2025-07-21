# Base PHP Docker Image [![Docker Build](https://github.com/ems-project/base-php-docker/actions/workflows/docker-build.yml/badge.svg?branch=8.4)](https://github.com/ems-project/base-php-docker/actions/workflows/docker-build.yml)

Base Docker image to build robust, layered PHP environments for production and development use.

## Features

- Based on [Official PHP Docker image](https://hub.docker.com/_/php)
- Managed by [Supervisor](http://supervisord.org/) (PID 1)
- Read-only, non-privileged container support
- Entrypoint hooks support
- Optional components:
  - [Nginx](https://pkgs.alpinelinux.org/package/v3.21/main/x86_64/nginx)
  - [Apache 2.4](https://pkgs.alpinelinux.org/package/v3.21/main/x86_64/apache2)
  - [Varnish](https://pkgs.alpinelinux.org/package/v3.21/main/x86_64/varnish)

## Build

### Requirements

- Docker + Buildx
- Optional: `.build.env` file to override default values

### Build examples

```sh
# Build fpm variant (production)
docker buildx bake --set target.args.PHP_VERSION=8.4.7 --set target.args.variant=fpm --set target.args.tgt=prd

# Build nginx variant (development)
docker buildx bake --set target.args.PHP_VERSION=8.4.7 --set target.args.variant=nginx --set target.args.tgt=dev

# Build all variants
docker buildx bake

--set prd.platform="linux/amd64"

## Lint

```bash
docker run -it --rm \
-e "DEFAULT_BRANCH=main" \
-e "VALIDATE_ALL_CODEBASE=true" \
-e "LINTER_RULES_PATH=/" \
-e "VALIDATE_BASH=true" \
-e RUN_LOCAL=true \
-e "FILTER_REGEX_EXCLUDE=.*test.*" \
-v $(pwd):/tmp/lint \
ghcr.io/super-linter/super-linter:slim-v7.4.0
```

```bash
docker run -it --rm \
-e "DEFAULT_BRANCH=main" \
-e "VALIDATE_ALL_CODEBASE=true" \
-e "IGNORE_GITIGNORED_FILES=true" \
-e "LINTER_RULES_PATH=/" \
-e "VALIDATE_MARKDOWN_PRETTIER=false" \
-e "VALIDATE_DOCKERFILE_HADOLINT=false" \
-e "VALIDATE_JSCPD=false" \
-e "RUN_LOCAL=true" \
-e "FILTER_REGEX_EXCLUDE=.*test.*" \
-v $(pwd):/tmp/lint \
ghcr.io/super-linter/super-linter:slim-v7.4.0
```
