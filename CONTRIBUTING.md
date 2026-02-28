# Contributing

Contributions are welcome. By participating, you agree to maintain a respectful and constructive environment.

For coding standards, testing patterns, architecture guidelines, commit conventions, and all
development practices, refer to the **[Development Guide](https://github.com/rios0rios0/guide/wiki)**.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) 20.10+
- [Docker Compose](https://docs.docker.com/compose/install/) v2+ (Compose file version 3.3)

## Development Workflow

1. Fork and clone the repository
2. Create a branch: `git checkout -b feat/my-change`
3. Build all Docker images (Nmap 7.80, Nikto 2.1.6, Wfuzz 2.4.6):
   ```bash
   docker-compose build
   ```
4. Run a scan against a target host (e.g. Nmap full port SYN scan):
   ```bash
   TARGET_HOST=192.168.1.100 docker-compose up nmap
   ```
5. Run Nikto web server vulnerability scan:
   ```bash
   TARGET_HOST=192.168.1.100 docker-compose up nikto
   ```
6. Run Wfuzz directory/parameter fuzzing:
   ```bash
   TARGET_HOST=http://192.168.1.100 docker-compose up wfuzz
   ```
7. Run all scanning services at once:
   ```bash
   TARGET_HOST=192.168.1.100 docker-compose up
   ```
8. Commit following the [commit conventions](https://github.com/rios0rios0/guide/wiki/Life-Cycle/Git-Flow)
9. Open a pull request against `main`
