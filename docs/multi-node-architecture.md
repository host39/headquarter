# Multi-Node Architecture (idea capture)

> **Status:** Idea / design notes. Nothing here is implemented. This captures the direction so we can build it piece by piece later.

## The shift

The original idea was that Headquarter manages **one machine**. The new idea is that Headquarter manages **the whole network**: many machines of different kinds, each reporting into one central place.

That splits the project into two parts:

1. **Headquarter (the hub).** The main machine where the web UI and API live and where all user interaction happens.
2. **The agent (working name, final name TBD).** A small monitoring and management service installed on every device we want to manage. It connects back to Headquarter, reports data, and carries out actions on that device.

```
                    ┌───────────────────────────────┐
                    │        Headquarter (hub)       │
                    │  Web UI · API · auth · storage │
                    │  alerting · service catalog    │
                    └───────────────▲───────────────┘
                                    │  agents connect in
        ┌───────────────┬───────────┼───────────────┬───────────────┐
        │               │           │               │               │
 ┌──────┴──────┐ ┌──────┴──────┐ ┌──┴──────────┐ ┌──┴──────────┐ ┌──┴──────────┐
 │ Agent       │ │ Agent       │ │ Agent       │ │ Agent       │ │ Agent       │
 │ Linux server│ │ Raspberry Pi│ │ OpenWrt     │ │ NAS         │ │ HQ host     │
 │ docker, sys │ │ docker, sys │ │ router      │ │ storage,sys │ │ itself      │
 └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
```

The hub machine also runs an agent, so it is managed the same way as everything else.

## Two kinds of users, two views

| Role | What they see |
|------|---------------|
| **Administrator** | Every machine, every agent, every container and service, device health, and management actions (start/stop containers, manage router hosts, and so on). |
| **Normal user** | Only the services available to them, as a portal of app tiles. They do not need to know which machine a service runs on. |

The user view is an abstraction over the infrastructure: a "service" is what users see, and where it runs is an admin concern only.

## The agent

A small, lightweight program or script installed on each managed device. It:

- Registers with Headquarter and stays connected.
- Collects data about the device (metrics, running services, containers where relevant).
- Exposes a set of **capabilities** that Headquarter can call to manage the device.
- Replaces the need to SSH into the box for routine tasks. (A direct remote shell route is possible as an optional capability, but the main goal is structured actions, not raw SSH.)

### Capabilities depend on the device

Not every device can do everything. The agent reports which capabilities it supports, and the UI only shows actions that make sense for that device.

| Capability | Linux server | Raspberry Pi | OpenWrt router | Notes |
|------------|:-:|:-:|:-:|-------|
| System metrics (CPU, memory, disk, network, temps) | ✓ | ✓ | ✓ | Baseline for every agent |
| System services (systemd / init.d) | ✓ | ✓ | ✓ | Router uses init.d / procd |
| Docker containers | ✓ | ✓ | ✗ | Routers usually do not run Docker |
| Router hosts / DHCP leases / DNS entries | ✗ | ✗ | ✓ | Router-only |
| Firewall / port forwards | optional | optional | ✓ | |
| Package updates | ✓ | ✓ | ✓ (opkg) | |
| Remote shell | optional | optional | optional | Off by default, admin only |

Example: installing the agent on an OpenWrt router enables "manage hosts" but not "manage containers". Installing it on a Raspberry Pi enables containers and services.

A clean way to model this is a **core agent + capability modules** (sometimes called plugins or collectors). The core handles connection, auth, and heartbeat. Modules like `docker`, `systemd`, `openwrt` are loaded based on what the device supports. This keeps the router build tiny and the server build full-featured.

## Design considerations to resolve later

- **Connection direction.** Agents dialing out to Headquarter (outbound connection, for example a persistent WebSocket or gRPC stream) is easier than Headquarter dialing in: it works behind NAT, needs no open ports on devices, and fits routers well.
- **Enrollment and trust.** How a new agent joins: one-time enrollment token generated in the admin UI, then a per-agent identity (for example mTLS certificate or signed key). Admin can revoke an agent.
- **Footprint.** The router agent must be very small (OpenWrt devices have limited flash and RAM). This affects language choice. A single static binary (for example Go or Rust) or a shell/Lua script for OpenWrt are options.
- **Security.** An agent with Docker or router control is effectively root on that device. All actions should be authenticated, permission-checked at the hub, and written to an audit log. Remote shell, if built, should be opt-in per device.
- **Offline behaviour.** What the UI shows when an agent disconnects, and whether agents buffer data while the hub is unreachable.
- **Service catalog.** How services discovered on agents (containers, ports) become user-facing tiles: auto-discovery plus admin curation, and per-user or per-group visibility.
- **Naming.** The agent needs a real name. Placeholder: "agent".

## Suggested build order (piecemeal)

1. Hub skeleton: API, auth, admin vs user roles.
2. Minimal agent: register, heartbeat, report basic system metrics.
3. Admin view: list of machines with online/offline status and metrics.
4. Docker capability module on the agent, container list and actions in the admin view.
5. Service catalog and the user-facing portal (services without host details).
6. Systemd / services capability module.
7. OpenWrt agent build with the router hosts capability.
8. Alerting across all agents.
9. Optional remote shell capability.
