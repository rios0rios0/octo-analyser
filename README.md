<h1 align="center">Octo Analyser</h1>
<p align="center">
    <a href="https://github.com/rios0rios0/octo-analyser/releases/latest">
        <img src="https://img.shields.io/github/release/rios0rios0/octo-analyser.svg?style=for-the-badge&logo=github" alt="Latest Release"/></a>
    <a href="https://github.com/rios0rios0/octo-analyser/blob/main/LICENSE">
        <img src="https://img.shields.io/github/license/rios0rios0/octo-analyser.svg?style=for-the-badge&logo=github" alt="License"/></a>
</p>

A Docker Compose-based security scanning toolkit that orchestrates multiple reconnaissance and vulnerability assessment tools against target hosts. Each tool is containerized with its own Dockerfile, compiled or installed from source for maximum control over versions and configurations, enabling reproducible penetration testing workflows.

## Features

| Tool | Version | Image Base | Purpose |
|------|---------|------------|---------|
| **Nmap** | 7.80 | Alpine 3.12 | TCP SYN scan (`-sS`), service version detection (`-sV`), NSE default + vuln scripts, full port range (`-p-`), aggressive timing (`-T5`) |
| **Nikto** | 2.1.6 | Alpine 3.10 | Web server vulnerability scanning with Perl, runs as a non-root `nikto` user with a locked root account |
| **Wfuzz** | 2.4.6 | Python 3.6 Alpine | Web application fuzzing for directory/file discovery and SQL injection parameter testing with 50 threads |
| **Arachni** | -- | Custom | Web application security scanner framework (Dockerfile placeholder) |
| **Dirsearch** | -- | Custom | Web directory and file brute-forcing (Dockerfile placeholder) |
| **Reconnoitre** | -- | Custom | Automated information gathering and service enumeration (Dockerfile placeholder) |

### Container highlights

- **Nmap** is compiled from source tarball on Alpine with Lua scripting support and OpenSSL, then build dependencies are removed to minimize image size
- **Nikto** clones the latest version from GitHub, creates a dedicated non-root user, sets a random root password, and locks the root account for security hardening
- **Wfuzz** installs from the GitHub repository with `pycurl` compiled against OpenSSL

## Technologies

- [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/) (Compose file version 3.3)
- [Nmap](https://nmap.org/) -- network scanner with NSE scripting engine
- [Nikto](https://cirt.net/Nikto2) -- web server vulnerability scanner
- [Wfuzz](https://github.com/xmendez/wfuzz) -- web application fuzzer
- [Arachni](https://www.arachni-scanner.com/) -- web application security scanner
- [Dirsearch](https://github.com/maurosoria/dirsearch) -- directory brute-forcer
- [Reconnoitre](https://github.com/codingo/Reconnoitre) -- automated reconnaissance

## Project Structure

```
octo-analyser/
├── nmap/
│   └── Dockerfile             # Nmap 7.80 compiled from source on Alpine 3.12
├── nikto/
│   └── Dockerfile             # Nikto 2.1.6 with hardened non-root setup
├── wfuzz/
│   └── Dockerfile             # Wfuzz 2.4.6 on Python 3.6 Alpine
├── arachni/
│   └── Dockerfile             # Arachni scanner (placeholder)
├── dirsearch/
│   └── Dockerfile             # Dirsearch brute-forcer (placeholder)
├── reconnoitre/
│   └── Dockerfile             # Reconnoitre recon tool (placeholder)
├── docker-compose.yml         # Service orchestration with TARGET_HOST variable
├── .dockerignore
├── .editorconfig
├── LICENSE
└── README.md
```

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/rios0rios0/octo-analyser.git
   cd octo-analyser
   ```

2. Build all Docker images:
   ```bash
   docker-compose build
   ```

## Usage

All scans are driven by the `TARGET_HOST` environment variable. Run individual tools or combine them as needed:

### Nmap -- full port SYN scan with service detection and vulnerability scripts

```bash
TARGET_HOST=192.168.1.100 docker-compose up nmap
```

This executes: `nmap -sS -sV --script=default,vuln -p- -T5 192.168.1.100`

### Nikto -- web server vulnerability scan

```bash
TARGET_HOST=192.168.1.100 docker-compose up nikto
```

This executes: `nikto.pl -h 192.168.1.100`

### Wfuzz -- SQL injection parameter fuzzing

```bash
TARGET_HOST=http://192.168.1.100 docker-compose up wfuzz
```

This executes: `wfuzz -t 50 -w SQL.txt http://192.168.1.100/FUZZ`

### Running all services at once

```bash
TARGET_HOST=192.168.1.100 docker-compose up
```

### Custom scan parameters

Override the default command for any service:

```bash
TARGET_HOST=10.0.0.1 docker-compose run nmap -sU -sV -p 53,161,500 10.0.0.1
TARGET_HOST=http://target.htb docker-compose run wfuzz -t 50 -w web-app.hms.txt -H 'HOST: FUZZ.htb' --hh 8193 target.htb
```

## Contributing

Contributions are welcome. Please open an issue or submit a pull request.

## License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.
