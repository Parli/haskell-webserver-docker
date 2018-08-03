# Haskell Webserver Base Image

This image contains common webserver dependencies pre-compiled for web applications written in Haskell.
The goal of this is simply to speed up builds of your actual project, since the precompiled packages you'd normally have locally won't exist in subsequent Docker builds.

## Sample usage

This is an example `Dockerfile` based around our internal usage of this.
It uses multistage builds to maximize caching in CI as well as local builds, while reducing the final image size.

```Dockerfile
# Build the runtime environment first since there's currently no way to specficy a starting point in a docker build, only an ending point
FROM debian:jessie as runtimeenv
# Install any additional libraries as needed - we use Postgres so libpq gets added to the runtime
RUN apt-get update && apt-get install -y \
    libpq-dev \
  && rm -rf /var/lib/apt/lists/*

# Use the same resolver from your stack.yaml
FROM parli/haskell-webserver:lts-11.18 as dependencies
# If you have compile-time system dependencies, add them here before copying manifests:
# RUN apt-get update && apt-get -y install make xz-utils libpq-dev
# Copy dependency manifests and build dependencies only first to help layer caching
COPY stack.yaml .
COPY package.yaml .
# This fixes a potential deadlock - see https://github.com/commercialhaskell/stack/issues/3830
RUN rm -rf ~/.stack/indices/
RUN stack build --dependencies-only

# Copy actual source code and build it - layer caching should result in typically starting here
FROM dependencies as build
COPY . .
RUN stack build

# Copy the built binary into a standalone server - this cuts the final image size down tremendously (for us, about 1.2GB to about 75MB)
FROM runtimeenv
WORKDIR /app
# Update this path if using a different LTS version
COPY --from-build /app/.stack-work/install/x84_64-linux/lts-11.18/8.2.2/bin/your-application-binary .
CMD ./your-application-binary
```

## Supported tags and respective `Dockerfile` links

- [`lts-11.18`](lts-11.18/Dockerfile)
- [`lts-11.15`](lts-11.15/Dockerfile)
