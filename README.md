# Cloud Infrastructure & Security Assessment

A sanitized, generalized case study based on a Linux cloud deployment I built and maintain. It demonstrates how containerized application services connect through a reverse proxy, along with the operational and security controls used to manage the environment.

**Technologies:** Ubuntu 22.04 LTS · DigitalOcean · Docker · Nginx · UFW · OpenSSH · Certbot · systemd

## Architecture at a glance

```text
External clients / webhook senders
               |
        HTTP / HTTPS
               |
      UFW + Nginx proxy
               |
     localhost-bound ports
               |
  +------------+------------+
  |            |            |
API service  Workflow    Data worker     Dashboard
  |          service         |              |
  +------ Docker containers and bridge networks ------+
```

Four application containers are deployed with localhost-bound published ports. Nginx has verified reverse-proxy routes for three services; a public route for the dashboard was not identified. The diagram is conceptual, not a claim that all four services are reachable through Nginx.

## Verified implementation

- Ubuntu Linux server with Docker and Nginx running as system services.
- Four non-privileged Docker containers with automatic restart policies.
- Application ports published on `127.0.0.1`, not public interfaces.
- Nginx proxy routes to three local services, with SSL-configured virtual hosts using Certbot-managed configuration.
- UFW enabled with default-deny inbound policy.
- SSH public-key authentication enabled; SSH password authentication disabled.
- System logging and a cloud monitoring agent present.

## Security assessment

The configuration also has documented improvement opportunities: application processes currently run as root inside containers, root SSH login is permitted, container root filesystems are writable, and an inbound firewall rule for an application port should be reviewed. A separate HTTP webhook entry point requires a transport-security and authentication review.


These are **findings, not completed remediations**. See [security assessment](security-assessment.md).




## Supporting documentation

- [Architecture and verified boundaries](infrastructure.md)
- [Security assessment and potential improvements](security-assessment.md)
- [Sanitized Nginx example](nginx-reverse-proxy.conf)

## Scope and disclosure

This is a generalized documentation and configuration-example project, not a deployable copy of the production environment. It contains no proprietary business logic, live hostnames, public IP addresses, credentials, tokens, or secret configuration values. Example configuration illustrates a pattern; it is not presented as an exact production export.
