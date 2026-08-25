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

<!-- chlog:start -->
## Changelog (chlog) — MANDATORY

If the repository you are working in uses chlog (a `.chlog.yaml` or `.chlog.yml`
config file, or a `.changes/` directory, exists at the project root), the
following is binding and ALWAYS applies: whenever you make ANY change, you MUST
create a changelog fragment as part of the same change — automatically, without
being asked, before committing.

- Do NOT edit CHANGELOG.md directly; it is generated from fragments.
- Create the fragment with:
  `chlog new --kind <Kind> --body "<imperative description>"`
- Valid kinds: Added, Changed, Deprecated, Removed, Fixed, Security
- Choose the kind that best matches the change (e.g., new feature → Added,
  bug fix → Fixed, behavior change → Changed, removal → Removed, security fix → Security).
- If the change is backward-INCOMPATIBLE with the public API (a breaking
  change), you MUST add the `--breaking` flag:
  `chlog new --kind <Kind> --breaking --body "<description>"`.
  This is the ONLY thing that triggers a major version bump — the kind alone
  never does (per SemVer, major = incompatible change). When unsure whether a
  change breaks compatibility, ask the user instead of guessing.
- Fragments are YAML files in `.changes/unreleased/`; stage them with your commit.
- `chlog check` fails the build when a fragment is missing — never skip it.
<!-- chlog:end -->
