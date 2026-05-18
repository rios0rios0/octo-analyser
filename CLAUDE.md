# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Docker Compose-based security scanning toolkit. Each tool lives in its own directory with a single Dockerfile. Orchestrated via `docker-compose.yml` using the `TARGET_HOST` environment variable.

## Build & Run

```bash
docker-compose build              # all images (Nmap compiles from source — slow)
docker-compose build <service>    # single service: nmap, nikto, wfuzz

TARGET_HOST=192.168.1.100 docker-compose up nmap
TARGET_HOST=192.168.1.100 docker-compose up nikto
TARGET_HOST=http://192.168.1.100 docker-compose up wfuzz
TARGET_HOST=192.168.1.100 docker-compose up           # all services
```

Override default scan arguments with `docker-compose run`:

```bash
TARGET_HOST=10.0.0.1 docker-compose run nmap -sU -sV -p 53,161,500 10.0.0.1
```

## Architecture

- **One tool per directory.** `nmap/`, `nikto/`, `wfuzz/` each contain one `Dockerfile`. Placeholder directories (`arachni/`, `dirsearch/`, `reconnoitre/`) have empty Dockerfiles reserving the structure.
- **Source compilation.** Nmap is compiled from the official tarball on Alpine for exact version pinning. Build deps are removed in the same `RUN` layer.
- **Environment-driven targets.** All services read `TARGET_HOST` from the calling shell. No secrets or targets are hardcoded.
- **Compose version 3.3.** Services define `ENTRYPOINT` in their Dockerfile; the `command` key in `docker-compose.yml` passes scan arguments.

## Dockerfile Conventions

- Chain commands with `&&` and `\` continuations. Clean caches (`rm -rf /var/cache/apk/*`) in the same layer.
- Run tools as a dedicated non-root user where practical (see `nikto/Dockerfile` as reference).
- Pin exact versions for base images and tool releases.
- `ENTRYPOINT` must invoke the tool binary directly so `docker-compose run` can pass arguments verbatim.

## Dependencies

No `package.json`, `requirements.txt`, or `go.mod` at repo root. All dependencies are managed inside individual Dockerfiles.

Runtime: Docker 20+, Docker Compose v2+.

## Conventions

Follow [commit conventions](https://github.com/rios0rios0/guide/wiki/Life-Cycle/Git-Flow) and the [Development Guide](https://github.com/rios0rios0/guide/wiki). See [CONTRIBUTING.md](CONTRIBUTING.md).
