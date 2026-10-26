# Cloud Infrastructure Hardening & DevSecOps Case Study

An architectural blueprint and security assessment based on a live, cloud-hosted Linux infrastructure deployment I designed, provisioned, and maintain. This repository demonstrates containerized microservice isolation, edge reverse proxy configuration, and the host-level defensive security controls enforced to minimize an environment's public attack surface.

**Core Infrastructure Stack:** Ubuntu Linux (LTS Baseline) · Cloud VPS (DigitalOcean) · Docker & Isolated Bridge Networks · Nginx Proxy · UFW · OpenSSH · Certbot (Automated TLS) · systemd

## Network Architecture & Traffic Isolation Boundary

```text
External Clients / Secure Webhook Senders
               |
         HTTP / HTTPS (TLS Enforced)
               |
      [ UFW Stateful Firewall ]
               |
     Nginx Edge Ingress Proxy
               |
  Localhost-Bound Published Ports (127.0.0.1)
               |
  +------------+------------+

  |            |            |
API Ingestion Workflow    Data Worker    Admin / Trade
   Service     Engine       Process        Dashboard

  |            |            |                  |
  +------ Docker Containers & Isolated Bridge Networks ------+
```

### Architectural Design Framework
*   **Ingress Segmentation:** Inbound network traffic is intercepted at the perimeter by Nginx acting as a reverse proxy. 
*   **Interface Isolation:** All application container ports (8000, 8001, 8005, 8006) are published strictly on the local loopback interface (`127.0.0.1`), entirely cutting off direct network exposure to the public internet.
*   **Intentional Dashboard De-exposure:** Verified reverse-proxy routes are intentionally provisioned only for active data ingestion services. The administration dashboard container is purposefully decoupled from public HTTP routing rules to establish an air-gapped security posture against external brute-force vectors.

## Verified Security Posture & Baseline Controls

*   **Network Perimeter Hardening:** Enforced a strict default-deny inbound network policy via UFW, exposing only essential web ports (80/443) and an obfuscated administrative port.
*   **Cryptographic Access Control:** Secured remote administration via OpenSSH by enforcing high-entropy cryptographic public-key authentication while completely disabling password-based handshakes.
*   **Automated Edge Cryptography:** Deployed automated Certbot routines for edge TLS certificate management, enforcing automated HTTP-to-HTTPS redirection.
*   **Daemon & Lifecycle Management:** Microservices are abstracted into unprivileged Docker container runtimes managed natively by host-level systemd service units with strict failover restart policies.

## Active Threat Modeling & Hardening Roadmap

In alignment with CompTIA Security+ infrastructure standards, this deployment undergoes continuous vulnerability auditing. Rather than treating security as a static state, the following items represent prioritized findings tracked on the environment's internal risk mitigation roadmap:

*   **Privileged Containers Audit:** Remediating application processes running inside container layers under root privileges (UID 0) by migrating to non-privileged container users.
*   **Host Access Hardening:** Transitioning from permitted root SSH access to dedicated administrative accounts managed via tracked `sudo` access lists.
*   **Immutability Enforcement:** Evaluating the deployment of read-only root filesystems (`--read-only`) for stateless worker nodes to mitigate post-exploitation persistence vectors.
*   **Webhook Authentication Ingestion:** Auditing a separate HTTP webhook ingress point to enforce transport-layer security and request-signature verification.

*For full deep-dives, risk rankings, and explicit remediation steps, see the supporting documentation below.*

## Supporting Documentation

*   [Infrastructure Architecture & Verified Boundaries](infrastructure.md)
*   [Security Configuration Assessment & Remediation Matrix](security-assessment.md)
*   [Sanitized Nginx Production Configuration Example](nginx-reverse-proxy.conf)

## Scope, Boundary & Secure Disclosure

This repository serves as a sanitized architectural documentation project and configuration baseline; it is not a direct clone or deployable mirror of a live production environment. It contains absolutely zero proprietary business logic, active hostnames, public IP addresses, live API tokens, or credential hashes. All configuration templates illustrate standard defensive design patterns rather than an exact production environment export.

