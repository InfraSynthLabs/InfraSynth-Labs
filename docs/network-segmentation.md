# Network Segmentation Design

## Overview

This project documents a segmented lab network design for separating personal devices, servers, public-facing services, guest devices, IoT devices, and management interfaces.

The goal is to practice secure infrastructure design using VLANs, firewall rules, least-privilege access, and service isolation.

This document uses example IP ranges and does not represent the exact live network configuration.

## Goals

* Separate trusted and untrusted devices.
* Isolate public-facing services from personal systems.
* Protect management interfaces from direct internet exposure.
* Reduce lateral movement risk if one system is compromised.
* Build a structure that supports future security monitoring and logging.
* Practice real-world network design concepts used in enterprise environments.

## Tools and Technologies

* pfSense firewall
* Managed switch
* VLANs
* Ubuntu Server
* Proxmox
* Docker
* Minecraft server hosting
* DNS records through Cloudflare
* Internal DNS through pfSense or AdGuard

## Example Network Zones

| VLAN    | Zone                  | Purpose                                                  | Example Subnet |
| ------- | --------------------- | -------------------------------------------------------- | -------------- |
| VLAN 10 | Main LAN              | Personal devices and trusted workstations                | 10.10.10.0/24  |
| VLAN 20 | Server Network        | Internal servers and hosted services                     | 10.10.20.0/24  |
| VLAN 30 | Public Services / DMZ | Public-facing services such as Minecraft servers         | 10.10.30.0/24  |
| VLAN 40 | IoT Network           | Smart devices and low-trust devices                      | 10.10.40.0/24  |
| VLAN 50 | Guest Network         | Guest Wi-Fi and temporary devices                        | 10.10.50.0/24  |
| VLAN 99 | Management Network    | Firewall, switch, Proxmox, and administrative interfaces | 10.10.99.0/24  |

These subnets are examples only and are used for documentation purposes.

## Design Philosophy

The network is designed around least privilege.

Each zone should only have the access it needs to function. Devices in lower-trust zones should not be able to reach sensitive systems, management interfaces, or personal devices.

The main security idea is that compromise of one system should not automatically expose the rest of the network.

## Traffic Rules

### Main LAN

The Main LAN is used for trusted personal devices.

Allowed:

* Access to the internet
* Access to selected internal services
* Administrative access to servers, when needed

Restricted:

* No unnecessary access to management interfaces unless from approved admin devices

### Server Network

The Server Network contains internal services.

Allowed:

* Access to the internet for updates
* Access from trusted admin devices
* Limited communication to other internal services when required

Restricted:

* No unrestricted access to the Main LAN
* No access to the Management Network unless explicitly required

### Public Services / DMZ

The DMZ contains public-facing services such as Minecraft servers.

Allowed:

* Public inbound access only to required service ports
* Outbound internet access for updates and downloads
* Admin access only from trusted devices or VPN

Restricted:

* No access to the Main LAN
* No access to the Management Network
* No access to firewall, switch, NAS, or Proxmox management interfaces
* No unrestricted access to internal servers

Example public service layout:

| Service                 | Public DNS Name    | Public Port | Internal Destination    |
| ----------------------- | ------------------ | ----------: | ----------------------- |
| Minecraft Server        | mc.example.com     |   TCP 25565 | Minecraft server in DMZ |
| Modded Minecraft Server | modded.example.com |   TCP 25566 | Modded server in DMZ    |

### IoT Network

The IoT Network is for smart devices and other low-trust equipment.

Allowed:

* Internet access
* Limited access to required local services, if needed

Restricted:

* No access to Main LAN devices
* No access to Management Network
* No access to internal servers unless specifically required

### Guest Network

The Guest Network is for temporary and untrusted devices.

Allowed:

* Internet access only

Restricted:

* No access to Main LAN
* No access to Server Network
* No access to DMZ systems
* No access to Management Network

### Management Network

The Management Network contains administrative interfaces.

Examples:

* pfSense
* Managed switch
* Proxmox
* Server management interfaces
* Admin dashboards

Allowed:

* Access only from trusted admin devices or VPN

Restricted:

* No direct internet exposure
* No access from Guest, IoT, or DMZ networks

## Example Firewall Rule Concepts

| Source Zone       | Destination Zone     | Action               | Reason                                            |
| ----------------- | -------------------- | -------------------- | ------------------------------------------------- |
| Guest             | Internet             | Allow                | Guest devices need internet access                |
| Guest             | Internal Networks    | Block                | Guests should not reach private systems           |
| IoT               | Internet             | Allow                | IoT devices may need cloud access                 |
| IoT               | Main LAN             | Block                | IoT devices should not access personal devices    |
| DMZ               | Internet             | Allow                | Public services may need updates                  |
| Internet          | DMZ Minecraft Server | Allow TCP 25565 only | Required for public Minecraft access              |
| DMZ               | Main LAN             | Block                | Public servers should not access personal devices |
| DMZ               | Management           | Block                | Public servers should not access admin interfaces |
| Main LAN          | Servers              | Allow selected ports | Trusted devices may use internal services         |
| Admin Device      | Management           | Allow                | Required for administration                       |
| All Other Traffic | Any                  | Deny by default      | Least-privilege security model                    |

## DNS Design

Public DNS records are used only for public services and the portfolio website.

Example public DNS:

| Hostname                                  | Destination                    |
| ----------------------------------------- | ------------------------------ |
| example.com                               | Portfolio website              |
| [www.example.com](http://www.example.com) | Portfolio website              |
| mc.example.com                            | Public Minecraft server        |
| modded.example.com                        | Public modded Minecraft server |

Private administrative services should use internal DNS only.

Example private DNS:

| Hostname                | Destination               |
| ----------------------- | ------------------------- |
| proxmox.lab.example.com | Internal Proxmox address  |
| pfsense.lab.example.com | Internal firewall address |
| adguard.lab.example.com | Internal DNS service      |
| docker.lab.example.com  | Internal Docker host      |

Private administrative hostnames should not be published in public DNS.

## Public Service Isolation

Public-facing services are placed in the DMZ so they can be reached from the internet without exposing the rest of the home network.

For example, a Minecraft server may be reachable from the internet on TCP 25565, but the server itself should not be able to access personal computers, management tools, or sensitive internal systems.

This creates a safer boundary between public services and private infrastructure.

## Security Considerations

* Public services should run in a dedicated VLAN or DMZ.
* Only required ports should be forwarded from the internet.
* Admin panels should not be exposed directly to the internet.
* Management interfaces should require VPN or trusted local access.
* Firewall rules should default to deny unless access is explicitly needed.
* Logs should be reviewed for blocked traffic and suspicious connection attempts.
* Documentation should use sanitized examples instead of real IP addresses or hostnames.

## What I Learned

* VLANs help separate systems by trust level and function.
* Firewall rules are most effective when designed around least privilege.
* Public services should be isolated from personal and management networks.
* DNS design matters because each hostname can point to a different destination.
* A DMZ reduces risk when hosting services for other users.
* Documentation should be clear enough for others to understand without exposing sensitive information.

## Future Improvements

* Add a sanitized network diagram.
* Add example pfSense rule tables.
* Add IDS/IPS monitoring.
* Add centralized logging.
* Add VPN-only administrative access.
* Add uptime monitoring for public services.
* Add separate documentation for Minecraft hosting security.
* Add backup and recovery procedures for public servers.
