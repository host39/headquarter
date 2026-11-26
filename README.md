# Headquarter

**Headquarter (HQ)** is the central control plane for the home lab. Think of it as an office.com style portal for everything running at home: one place to launch your apps, manage your Docker containers, watch the health of every service, and get alerted when something goes wrong.

> **Status:** Early planning. This README describes the vision and intended scope. Nothing below is implemented yet.

---

## Why

A home lab grows fast. Services end up scattered across ports, hosts, and bookmarks, and the only way to know something broke is when it stops working. Headquarter brings all of it under one roof:

- **One front door** to every self-hosted app.
- **One console** to manage containers without SSH-ing into a box.
- **One dashboard** to see whether everything is healthy.
- **One alerting pipeline** so problems find you, not the other way around.

## Core Features

### 1. App Launcher (the "portal")
- Tile-based home page linking to every self-hosted service.
- Groups and categories (media, networking, storage, dev tools, and so on).
- Live status badge on each tile (up, degraded, down).
- Search across all registered apps.

### 2. Docker Management
- List containers across one or more Docker hosts.
- Start, stop, restart, and remove containers.
- View logs (live tail and history).
- Inspect container details: image, ports, volumes, environment, resource usage.
- Pull and update images.
- Basic Docker Compose stack awareness.

### 3. Service and Health Monitoring
- Health checks for services (HTTP, TCP, ping, container health status).
- Host metrics: CPU, memory, disk, network, temperatures.
- Uptime history and response time graphs.
- At-a-glance overview of the whole lab.

### 4. Alerting
- Configurable rules (service down, disk above threshold, container restarting in a loop, certificate expiring, and similar).
- Notification channels such as email, push, or chat webhooks.
- Alert history, acknowledgement, and silencing.

## Planned Architecture (draft)

```
                 ┌──────────────────────────┐
                 │      Headquarter UI       │
                 │ (portal, dashboards, ops) │
                 └────────────┬─────────────┘
                              │
                 ┌────────────▼─────────────┐
                 │     Headquarter API       │
                 │ auth, config, scheduling  │
                 └──┬──────────┬─────────┬──┘
                    │          │         │
          ┌─────────▼───┐ ┌────▼─────┐ ┌─▼──────────┐
          │ Docker      │ │ Health   │ │ Alerting   │
          │ connector   │ │ checker  │ │ engine     │
          └─────────────┘ └──────────┘ └────────────┘
                    │          │         │
             Docker hosts   Services   Notification
                                       channels
```

Technology choices (frontend framework, backend language, datastore) are still open and will be documented here once decided.

## Security Considerations

Headquarter will have control over the Docker socket, which is effectively root access to the host. The design should account for this from day one:

- Authentication required for every page and API call.
- Role based permissions (for example, read-only viewers versus operators).
- Prefer a Docker socket proxy over mounting the raw socket.
- Keep the portal on the internal network or behind a VPN or reverse proxy with SSO.
- Audit log of every container action.

## Roadmap

- [ ] Decide on tech stack
- [ ] Project scaffolding and local dev setup
- [ ] App launcher with static configuration
- [ ] Docker host connection and container listing
- [ ] Container actions (start, stop, restart, logs)
- [ ] Health checks and status badges
- [ ] Host metrics dashboard
- [ ] Alert rules and notification channels
- [ ] Authentication and roles
- [ ] Deployment via Docker Compose

## Getting Started

Setup instructions will be added once the initial code lands.

## Contributing

This is a personal home lab project. Ideas and issues are welcome in the issue tracker.
