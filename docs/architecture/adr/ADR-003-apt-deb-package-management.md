# ADR-003: Use APT and .deb Packages for Bugema Linux

- **Status:** Accepted
- **Date:** 2026-09-24
- **Decision:** APT + dpkg + .deb
- **Scope:** Bugema Linux Alpha 0.1

## Context

Bugema Linux requires a reliable and maintainable software distribution mechanism for university laboratories, student computers, research environments, and institutional deployments.

The package management system must support:

- installation and removal of software;
- dependency management;
- security updates;
- software repositories;
- offline package distribution;
- Bugema-specific packages;
- reproducible builds;
- institutional administration; and
- long-term maintenance.

The package management system should also remain compatible with the selected Ubuntu 26.04 LTS foundation.

## Decision

Bugema Linux Alpha 0.1 will use:

- `.deb` as the primary software package format;
- `dpkg` as the underlying package installation and database system; and
- `APT` as the primary package management and repository system.

Bugema Linux will use the Ubuntu repositories as the primary upstream software source and will introduce a dedicated signed Bugema repository for Bugema-specific packages and configurations.

## Package Sources

The initial package strategy will use the following sources:

### Ubuntu Packages

Bugema Linux will use packages from the Ubuntu 26.04 LTS repositories where appropriate.

These packages provide the majority of the operating-system components and general-purpose applications.

### Bugema Packages

Bugema-specific functionality will be packaged separately where practical.

Potential packages include:

- `bugema-desktop`
- `bugema-branding`
- `bugema-wallpapers`
- `bugema-welcome`
- `bugema-system-info`
- `bugema-lab-tools`
- `bugema-student-tools`
- `bugema-developer-tools`
- `bugema-offline-tools`
- `bugema-security-tools`
- `bugema-documentation`

### Approved Third-Party Packages

Third-party software may be included when there is a clear educational, research, institutional, or technical requirement.

Third-party packages must be evaluated for:

- licensing;
- security;
- maintenance;
- compatibility;
- provenance;
- update mechanisms; and
- redistribution requirements.

## Repository Strategy

The long-term Bugema Linux package architecture will include a signed Bugema APT repository.

The repository may contain:

- Bugema packages;
- Bugema configuration packages;
- approved educational software;
- institutional utilities;
- documentation packages; and
- other project-maintained components.

Packages should be built through an automated and documented process rather than manually modified on production systems.

## Offline Package Management

Because Bugema Linux is intended for environments where Internet connectivity may be limited or unreliable, the project will support offline package distribution.

Potential mechanisms include:

- downloadable package bundles;
- local APT caches;
- local university APT mirrors;
- USB-based package repositories; and
- LAN-based package servers.

The offline architecture will be developed further in ADR-004.

## Package Profiles

Bugema Linux will use modular package profiles rather than maintaining multiple completely separate operating systems.

Initial profiles may include:

- `bugema-education`
- `bugema-research`
- `bugema-iot`
- `bugema-networking`
- `bugema-security`

These profiles will install groups of related packages while sharing the same Bugema Linux core.

## Rationale

Using APT and `.deb` provides strong alignment with the Ubuntu foundation.

It also provides:

- mature dependency management;
- extensive package availability;
- established security-update mechanisms;
- repository infrastructure;
- compatibility with Ubuntu administration practices;
- support for automated installation;
- easier maintenance by Linux administrators; and
- opportunities for students to learn real-world package management.

The project therefore avoids creating a custom package format or package manager.

## Alternatives Considered

### RPM

RPM-based systems such as Fedora and related distributions provide a mature package ecosystem.

However, Bugema Linux is based on Ubuntu 26.04 LTS, making `.deb` and APT the more direct integration choice.

### Flatpak

Flatpak is useful for distributing desktop applications in a sandboxed format.

It may be supported in future Bugema Linux releases for selected desktop applications, but it will not replace APT as the core operating-system package management system.

### Snap

Snap packages may be used where appropriate because of Ubuntu ecosystem compatibility.

However, Snap will not replace APT for the core Bugema Linux operating system.

### Custom Package Manager

Developing a Bugema-specific package manager would introduce unnecessary complexity and maintenance requirements.

The project will use established upstream tooling instead.

## Consequences

### Positive

- Strong compatibility with Ubuntu.
- Mature package-management ecosystem.
- Large software availability.
- Established dependency management.
- Easier security updates.
- Support for offline repositories.
- Enables Bugema-specific packages.
- Suitable for teaching Linux administration and DevOps.
- Reduced development and maintenance burden.

### Negative

- Dependence on Ubuntu repository structures and policies.
- Bugema must maintain its own packages carefully.
- Repository signing and infrastructure require operational management.
- Third-party software may introduce additional licensing and security considerations.

## Implementation

The Alpha 0.1 implementation will initially use Ubuntu APT repositories.

Bugema-specific packages will be introduced progressively.

The project will eventually provide:

1. package source code;
2. package build configuration;
3. automated package builds;
4. repository metadata;
5. repository signing;
6. package testing; and
7. documented installation and update procedures.

## Security Requirements

All Bugema repositories must use appropriate package-signing mechanisms.

The project should verify:

- package provenance;
- package integrity;
- repository metadata;
- build source;
- dependencies; and
- release versions.

Security updates should be prioritized for timely testing and deployment.

## Testing Requirements

Package management must be tested for:

- package installation;
- package removal;
- package upgrades;
- dependency resolution;
- repository access;
- offline installation;
- package integrity;
- repository signing;
- failed installation recovery;
- configuration preservation; and
- rollback or recovery procedures where applicable.

## Review

This decision should be reviewed if:

- Ubuntu changes its package-management strategy significantly;
- Bugema Linux introduces another distribution base;
- a new package-management requirement emerges;
- offline requirements change substantially; or
- institutional deployment introduces new package-management requirements.
