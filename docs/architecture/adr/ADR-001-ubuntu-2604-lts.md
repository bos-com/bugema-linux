# ADR-001: Use Ubuntu 26.04 LTS as the Bugema Linux Base Distribution

- **Status:** Accepted
- **Date:** 2026-09-24
- **Decision:** Ubuntu 26.04 LTS
- **Scope:** Bugema Linux Alpha 0.1

## Context

Bugema Linux requires a stable and maintainable operating-system foundation for teaching, learning, research, and institutional computing.

Building an operating system completely from the Linux kernel and independently maintaining the entire userspace would create a significant maintenance burden for the Bugema Open Source Community (BOSC).

The project therefore requires an established upstream distribution with:

- long-term support;
- a large software ecosystem;
- broad hardware support;
- established security processes;
- extensive documentation;
- a mature package-management system;
- support for educational and development workloads; and
- an active international open-source community.

## Decision

Bugema Linux will use **Ubuntu 26.04 LTS** as its upstream operating-system foundation.

The initial Bugema Linux target architecture will be:

- Ubuntu 26.04 LTS;
- amd64/x86_64;
- Ubuntu-supported Linux kernel;
- Ubuntu userspace;
- Ubuntu/Debian package ecosystem.

Bugema Linux will add Bugema-specific components above this foundation rather than creating an independent operating-system ecosystem.

## Alternatives Considered

### Debian

Debian provides a strong technical foundation and is widely used for stable Linux deployments.

However, Ubuntu provides a broader institutional ecosystem and established desktop and education-oriented derivatives that align well with the intended Bugema Linux use cases.

### Fedora

Fedora provides a modern Linux platform and strong development tooling.

However, its shorter release lifecycle is less aligned with the initial institutional long-term-support requirement.

### Linux From Scratch

Linux From Scratch provides maximum control over the operating system.

However, it would create a significantly larger maintenance and security burden and is therefore unsuitable as the initial production foundation.

### Building an Independent Distribution

Maintaining an independent kernel, package ecosystem, repositories, installer, and security infrastructure would require substantially more resources than are currently justified.

## Consequences

### Positive

- Access to a mature package ecosystem.
- Strong hardware compatibility.
- Established security infrastructure.
- Long-term support foundation.
- Large developer community.
- Reduced maintenance burden.
- Easier onboarding of students and lecturers already familiar with Ubuntu.

### Negative

- Bugema Linux remains dependent on upstream Ubuntu.
- Some architectural decisions will be constrained by Ubuntu.
- Ubuntu-specific changes may require adaptation when upstream changes.
- Bugema cannot independently control every component of the operating system.

## Implementation

The build system will use Ubuntu 26.04 LTS as the base environment.

Bugema-specific packages and configurations will be maintained in the Bugema Linux repository.

## Review

This decision should be reviewed before a future major release if:

- Ubuntu's support model changes;
- a different upstream provides a substantial technical advantage;
- Bugema develops sufficient resources to maintain a different base;
- hardware requirements change significantly; or
- institutional requirements change.
