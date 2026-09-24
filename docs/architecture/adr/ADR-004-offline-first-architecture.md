# ADR-004: Adopt an Offline-First Architecture for Bugema Linux

- **Status:** Accepted
- **Date:** 2026-09-24
- **Decision:** Offline-first architecture
- **Scope:** Bugema Linux Alpha 0.1 and future releases

## Context

Bugema Linux is intended for use in university laboratories, student computers, research environments, community learning environments, and institutional computing facilities.

These environments may experience:

- unreliable Internet connectivity;
- expensive or limited bandwidth;
- temporary network outages;
- overloaded institutional networks;
- limited access to external software repositories;
- interruptions in cloud-based services; and
- differences in network availability between campuses and laboratories.

A conventional Internet-dependent desktop distribution would therefore create challenges for teaching, learning, software development, research, and system administration.

Bugema Linux must remain useful when Internet connectivity is unavailable.

## Decision

Bugema Linux will adopt an **offline-first architecture**.

The operating system must provide a useful baseline experience without requiring continuous Internet access.

Internet connectivity will be treated as an enhancement rather than an absolute requirement for core learning and computing activities.

The offline-first architecture will include:

- an installable offline ISO;
- local package availability;
- offline documentation;
- offline educational resources;
- local development tools;
- local configuration and administration tools; and
- mechanisms for synchronizing updates when connectivity becomes available.

## Offline Operating-System Installation

The Bugema Linux installation media should contain the core operating-system packages required for installation.

The installation process should support installation in environments where Internet connectivity is unavailable.

Where practical, users should be able to:

1. boot the Bugema Linux installation media;
2. install the operating system;
3. configure a local user;
4. configure networking;
5. install the core Bugema software environment; and
6. begin using the system without immediately connecting to the Internet.

## Local Package Availability

Bugema Linux will progressively support local package distribution.

Possible mechanisms include:

- APT package caches;
- local APT mirrors;
- LAN-based package servers;
- USB package repositories;
- offline package bundles; and
- campus package repositories.

A university laboratory may therefore maintain a local package server that can serve multiple Bugema Linux computers.

This approach reduces repeated downloads and improves resilience during Internet outages.

## Offline Educational Resources

Bugema Linux will provide mechanisms for accessing educational content without continuous Internet connectivity.

Potential components include:

- Kiwix;
- offline documentation;
- locally hosted manuals;
- open educational resources;
- programming documentation;
- Linux administration guides;
- course materials;
- research documentation; and
- Bugema Linux documentation.

A future Bugema Learning Repository may provide educational content through the university local network.

## Local Development Environment

The base Bugema Linux installation should provide or support installation of essential development tools without requiring continuous Internet access.

These may include:

- Git;
- Python;
- Java;
- C/C++;
- build tools;
- shell scripting;
- SQL databases;
- Node.js;
- text editors;
- documentation tools; and
- debugging tools.

Where practical, commonly required development packages should be included in offline installation media or available through a local repository.

## Local Network Architecture

The long-term institutional architecture may use:

```text
                    Internet
                       |
                       |
              Bugema Update Server
                       |
                       |
             Local APT Mirror/Cache
                       |
              -------------------
              |        |        |
            Lab 1    Lab 2    Lab 3
              |        |        |
          Bugema    Bugema    Bugema
           PCs       PCs       PCs
