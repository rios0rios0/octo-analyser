# Copilot Instructions for Octo Analyser

## Project Overview

**Octo Analyser** is a Docker Compose-based security scanning toolkit that orchestrates multiple
reconnaissance and vulnerability assessment tools against target hosts. Each tool is containerized
with its own Dockerfile, compiled or installed from source for maximum version control, enabling
reproducible penetration testing workflows.

## Repository Structure

```
octo-analyser/
├── .github/
│   └── copilot-instructions.md   # This file
├── nmap/
│   └── Dockerfile                # Nmap 7.80 compiled from source on Alpine 3.12
├── nikto/
│   └── Dockerfile                # Nikto 2.1.6 with hardened non-root setup on Alpine 3.10
├── wfuzz/
│   └── Dockerfile                # Wfuzz 2.4.6 on Python 3.6 Alpine
├── arachni/
│   └── Dockerfile                # Arachni scanner (placeholder – not yet implemented)
├── dirsearch/
│   └── Dockerfile                # Dirsearch brute-forcer (placeholder – not yet implemented)
├── reconnoitre/
│   └── Dockerfile                # Reconnoitre recon tool (placeholder – not yet implemented)
├── docker-compose.yml            # Service orchestration driven by TARGET_HOST variable
├── .dockerignore
├── .editorconfig
├── CHANGELOG.md
├── CLAUDE.md                     # Guidance for Claude Code sessions
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## Technology Stack

| Component | Technology |
|-----------|-----------|
| Orchestration | Docker Compose v3.3 |
| Nmap container | Alpine 3.12, Nmap 7.80 compiled from source |
| Nikto container | Alpine 3.10, Nikto 2.1.6 cloned from GitHub |
| Wfuzz container | Python 3.6 Alpine, Wfuzz 2.4.6 cloned from GitHub |
| Arachni / Dirsearch / Reconnoitre | Placeholder Dockerfiles (not yet implemented) |

## Build Commands

```bash
# Build all Docker images
docker-compose build

# Build a specific service image
docker-compose build nmap
docker-compose build nikto
docker-compose build wfuzz
```

Builds compile or install tools from source, so initial builds may take **several minutes** per
service (5–15 minutes for Nmap due to source compilation).

## Run Commands

All scans are driven by the `TARGET_HOST` environment variable:

```bash
# Nmap – full port SYN scan with service detection and vulnerability scripts
TARGET_HOST=192.168.1.100 docker-compose up nmap
# Executes: nmap -sS -sV --script=default,vuln -p- -T5 192.168.1.100

# Nikto – web server vulnerability scan
TARGET_HOST=192.168.1.100 docker-compose up nikto
# Executes: nikto.pl -h 192.168.1.100

# Wfuzz – SQL injection parameter fuzzing
TARGET_HOST=http://192.168.1.100 docker-compose up wfuzz
# Executes: wfuzz -t 50 -w SQL.txt http://192.168.1.100/FUZZ

# Run all services simultaneously
TARGET_HOST=192.168.1.100 docker-compose up

# Override the default command for a specific service
TARGET_HOST=10.0.0.1 docker-compose run nmap -sU -sV -p 53,161,500 10.0.0.1
TARGET_HOST=http://target.htb docker-compose run wfuzz -t 50 -w web-app.hms.txt -H 'HOST: FUZZ.htb' --hh 8193 target.htb
```

## Architecture and Design Patterns

- **One tool per directory**: Each security tool lives in its own subdirectory containing exactly
  one `Dockerfile`. The `docker-compose.yml` at the root references each subdirectory as the build
  context.
- **Source compilation for security-critical tools**: Nmap is compiled from the official tarball
  to allow precise version pinning and to strip build dependencies post-install (multi-stage
  style without a second stage, using `apk del .build-deps`).
- **Security hardening in containers**: Nikto runs as a dedicated non-root user (`nikto`), the
  root account is locked, and a random root password is set and immediately discarded.
- **Environment-variable-driven scanning**: All services expect `TARGET_HOST` to be set in the
  calling shell. No secrets or target information are hardcoded.
- **Placeholder pattern**: Tools that are not yet implemented (Arachni, Dirsearch, Reconnoitre)
  contain an empty `Dockerfile` to reserve the directory structure and signal intent.

## Dependencies

### Runtime prerequisites

- [Docker](https://docs.docker.com/get-docker/) 20+
- [Docker Compose](https://docs.docker.com/compose/install/) v2+ (`docker compose` or
  `docker-compose`)

### No application-level package manager is used

There is no `package.json`, `requirements.txt`, `go.mod`, or equivalent at the repository root.
All dependencies are managed inside the individual Dockerfiles.

## CI/CD Pipeline

There is currently **no build or deployment pipeline** in this repository. The only workflows are `.github/workflows/claude-review.yaml` and `.github/workflows/claude-mention.yaml`, which call the shared Claude reusable workflows in `rios0rios0/pipelines` and need the `CLAUDE_CODE_OAUTH_TOKEN` secret. Build and scan validation is performed locally via `docker-compose build`.

## Development Workflow

1. Fork and clone the repository.
2. Create a feature branch: `git checkout -b feat/my-change`
3. Make changes to the relevant `Dockerfile` or `docker-compose.yml`.
4. Build and test the affected service:
   ```bash
   docker-compose build <service>
   TARGET_HOST=<test-ip> docker-compose up <service>
   ```
5. Commit following the [commit conventions](https://github.com/rios0rios0/guide/wiki/Life-Cycle/Git-Flow).
6. Open a pull request against `main`.

Refer to [CONTRIBUTING.md](../CONTRIBUTING.md) and the
[Development Guide](https://github.com/rios0rios0/guide/wiki) for full coding standards, commit
conventions, and architecture guidelines.

## Coding Conventions

- **Dockerfile style**: Use `RUN` layers that chain `&&` commands with `\` line continuations;
  clean up package caches (`rm -rf /var/cache/apk/*`) in the same layer to minimise image size.
- **Non-root users**: Containerised tools should run as a dedicated non-root user where practical
  (see Nikto's `Dockerfile` as the reference implementation).
- **Version pinning**: Pin exact versions for base images (e.g., `alpine:3.12`, `python:3.6-alpine`)
  and tool releases in every `Dockerfile`.
- **Entrypoint**: Each `Dockerfile` must declare an `ENTRYPOINT` that invokes the tool binary
  directly, allowing the `docker-compose.yml` `command` key to pass arguments verbatim.
- **Compose file version**: Keep `docker-compose.yml` at version `"3.3"`.

## Common Tasks

### Add a new security tool

1. Create a new directory at the repository root: `mkdir <toolname>`.
2. Add a `Dockerfile` that installs/compiles the tool, creates a non-root user, and sets the
   correct `ENTRYPOINT`.
3. Add a corresponding service entry in `docker-compose.yml` referencing
   `dockerfile: <toolname>/Dockerfile` and setting the default `command` with `${TARGET_HOST}`.

### Implement a placeholder tool

The three placeholder tools (Arachni, Dirsearch, Reconnoitre) follow the same pattern as the
implemented tools. Fill in the empty `Dockerfile` in the respective directory and add the service
entry to `docker-compose.yml`.

### Troubleshooting

| Problem | Solution |
|---------|----------|
| Build fails on source compilation | Check internet access from the build host; source tarballs and `git clone` calls require outbound HTTPS |
| `TARGET_HOST` not set error | Export the variable before running: `export TARGET_HOST=<ip>` |
| Wfuzz `pycurl` build error | Ensure `PYCURL_SSL_LIBRARY=openssl` is set (already in the `Dockerfile`) |
| Nikto permission denied | Confirm the container runs as the `nikto` user and `/nikto` ownership is correct |

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
