# Infrastructure architecture

## Verified deployment inventory

| Layer | Observed implementation |
|---|---|
| Hosting | DigitalOcean virtual server |
| OS | Ubuntu 22.04.5 LTS |
| Containers | Four Docker application containers |
| Reverse proxy | Nginx listening on public ports 80 and 443 |
| Application bindings | Loopback-only published ports 8000, 8001, 8005, 8006 |
| Networking | Default bridge and three application-specific Docker bridge networks |
| Host firewall | UFW active; inbound denied by default |
| Remote access | SSH public-key authentication enabled; password authentication disabled |
| Operations | Container restart policies, system logging, cloud monitoring agent |

## Request path

1. An external client reaches the host through a firewall-permitted web port.
2. Nginx selects a virtual host or route and proxies traffic to a localhost-bound application port.
3. Docker forwards traffic to the appropriate container.
4. Application processing happens within the container. Internal application logic is outside the scope of this repository.

Verified Nginx upstreams point to three distinct application ports. A fourth dashboard container listens on localhost but no corresponding Nginx route was identified in the inspected configuration.

## Security boundaries and limitations

- Loopback-only published application ports reduce direct exposure from external interfaces, but do not establish that the applications are unreachable through the reverse proxy.
- Separate Docker network assignments were observed; full isolation and additional attachments were not audited.
- SSL-enabled Nginx virtual hosts were observed; certificate validity and redirect behavior were not tested.
- Firewall rules and listening sockets were inspected; external reachability was not tested.
- Container restart policies were verified; application-level health checks and recovery behavior were not verified.
