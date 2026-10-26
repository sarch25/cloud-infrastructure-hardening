# Cloud Infrastructure Engineering & Security Assessment

**Designing, deploying, operating, and assessing a multi-service Linux cloud environment.**

An engineering case study documenting a cloud-hosted application environment I independently designed, provisioned, and maintain.

The environment supports four containerized application services with distinct responsibilities, including API ingestion, workflow processing, background data operations, and application dashboards.

This project demonstrates how I integrated application deployment, container networking, reverse-proxy routing, Linux administration, and infrastructure security into a functioning cloud environment—and how I assessed the resulting architecture for security risks and operational improvements.

**Technology Stack:** Ubuntu Linux · DigitalOcean · Docker · Nginx · UFW · OpenSSH · Certbot · systemd

## 1. System Architecture

```text
                 External Clients
                        |
                  HTTP / HTTPS
                        |
                Cloud VPS Firewall
                        |
                 Host UFW Rules
                        |
                  Nginx Proxy
                        |
           Localhost-Bound Services
                 (127.0.0.1)
                        |
         +--------------+--------------+
         |              |              |
    API Ingestion    Workflow      Background
      Service        Service       Data Service
         |              |              |
         +--------------+--------------+
                        |
                 Docker Runtime

              Additional Container
                        |
                Application Dashboard
             (No verified public route)
```

**Deployment characteristics:**

- Four application containers deployed on a Linux cloud server.
- Application ports published on the host loopback interface rather than public network interfaces.
- Nginx reverse-proxy routes verified for three services.
- Separate Docker networking configurations supporting different application responsibilities.
- Persistent application storage where required.
- Container restart policies supporting service recovery following process or host interruptions.

The diagram is a conceptual representation of the deployment. It does not imply that all services share one Docker network or that every service is publicly accessible.

## 2. Engineering Implementation

### Multi-Service Application Deployment

Deployed and configured four containerized application services within a single cloud-hosted Linux environment.

The deployment required coordinating application processes, container networking, port assignments, runtime configuration, and service availability while keeping application interfaces separated from direct public network access.

**Engineering considerations:**

- Configuring multiple application services without conflicting host ports.
- Separating externally routed endpoints from internal application interfaces.
- Managing container lifecycles and restart behavior.
- Supporting application-specific networking and persistence requirements.
- Maintaining visibility into running services and their host-level exposure.

### Reverse Proxy & Network Architecture

Configured Nginx as the primary reverse-proxy layer for externally routed application endpoints.

Application containers publish their service ports to `127.0.0.1`, allowing Nginx to forward requests locally without requiring those application ports to listen directly on public interfaces.

This creates a clearer boundary between public ingress and application services.

**Verified implementation:**

- Localhost-bound application ports.
- Nginx routing to three application services.
- UFW default-deny inbound policy.
- TLS-configured Nginx virtual hosts with Certbot-managed configuration.

A public reverse-proxy route for the dashboard was not identified during the assessment.

### Linux Operations & Service Management

Configured and maintained the supporting Linux environment, including Docker, Nginx, SSH access, firewall rules, logging, and monitoring components.

Operational controls include:

- Docker container restart policies.
- Linux service management through systemd.
- SSH public-key authentication.
- Disabled SSH password authentication.
- System logging and a cloud monitoring agent.

These controls support routine operation and troubleshooting, although container restart policies do not constitute a high-availability or automatic failover architecture.

## 3. Infrastructure Security Assessment

Beyond deployment, I conducted a configuration-level security review of the running environment.

The assessment focused on network exposure, authentication, container privileges, service configuration, and opportunities to reduce attack surface.

| Security area | Verified condition | Assessment |
|---|---|---|
| Network exposure | Application ports bound to `127.0.0.1` | Reduces direct external exposure |
| Host firewall | UFW enabled with default-deny inbound policy | Provides host-level network restrictions |
| SSH authentication | Public-key authentication enabled; password authentication disabled | Reduces password-based attack exposure |
| Reverse proxy | Nginx routes verified for three services | Establishes a controlled application ingress layer |
| TLS | Certbot-managed TLS configuration present | Supports encrypted access on configured HTTPS endpoints |
| Container privileges | Application processes running as root inside containers | Hardening opportunity |
| Root SSH access | Root SSH login permitted | Hardening opportunity |
| Container filesystem | Writable container root filesystems | Hardening opportunity |
| Webhook ingress | Separate HTTP entry point identified | Requires transport-security and authentication review |

The assessment distinguishes existing security controls from risks that still require remediation.

## 4. Security Findings & Engineering Tradeoffs

### Container Runtime Privileges

The assessed application containers run processes with root privileges inside their respective containers.

**Risk:** A compromised application process may have greater privileges within its container than required.

**Improvement:** Evaluate non-root application users, reduced Linux capabilities, and filesystem permissions compatible with each service.

### Administrative Access

SSH password authentication is disabled, but root SSH login remains permitted.

**Risk:** Direct administrative access increases the importance of protecting privileged authentication credentials and monitoring access.

**Improvement:** Evaluate dedicated administrative accounts with controlled `sudo` privileges and restricted root SSH access.

### Container Filesystem Protection

Container root filesystems are currently writable.

**Risk:** Writable filesystems can increase opportunities for unwanted modification or persistence following application compromise.

**Improvement:** Evaluate read-only container filesystems for compatible services, with explicit writable mounts for required application data.

### Webhook Transport & Authentication

A separate HTTP webhook entry point was identified during the assessment.

**Risk:** Its transport-security and request-authentication properties require additional verification.

**Improvement:** Review HTTPS enforcement, authentication mechanisms, request validation, and applicable network restrictions.

**These are documented findings and proposed improvements—not claims of completed remediation.**

## 5. Supporting Technical Evidence

The repository includes supporting documentation describing the architecture, verified configurations, and identified security risks.

- [Infrastructure Architecture & Verified Boundaries](infrastructure.md)
- [Security Assessment & Improvement Opportunities](security-assessment.md)
- [Sanitized Nginx Configuration Example](nginx-reverse-proxy.conf)

The documentation is intended to show not only which technologies were used, but how the components interact, what was verified, and which security tradeoffs were identified.

## 6. Scope & Disclosure

This repository documents a real cloud infrastructure deployment through sanitized architecture descriptions, configuration examples, and security assessment findings.

It is not a deployable copy of the live environment.

Proprietary application logic, credentials, access tokens, private hostnames, public IP addresses, and sensitive operational configuration are excluded.

Configuration examples illustrate relevant implementation patterns and are not represented as exact production exports.

