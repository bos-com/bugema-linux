# ADR-002: Use Xfce 4.20 as the Default Bugema Linux Desktop

- **Status:** Accepted
- **Date:** 2026-09-24
- **Decision:** Xfce 4.20
- **Scope:** Bugema Linux Alpha 0.1

## Context

Bugema Linux is intended for use in university laboratories, student computers, research environments, virtual machines, and institutional computing environments.

Some target computers may have limited:

- RAM;
- CPU performance;
- storage capacity; and
- graphics resources.

The project therefore requires a desktop environment that provides a complete graphical Linux experience while remaining relatively lightweight and configurable.

The desktop must also support:

- teaching and learning;
- software development;
- office productivity;
- networking;
- research;
- system administration;
- accessibility;
- institutional branding; and
- student customization projects.

## Decision

Bugema Linux Alpha 0.1 will use **Xfce 4.20** as its default desktop environment.

The project will use Ubuntu 26.04 LTS as the operating-system foundation and integrate Xfce as the Bugema desktop layer.

Bugema Linux will not maintain a separate fork of the Xfce desktop environment.

The Bugema desktop layer will primarily provide:

- configuration;
- branding;
- default applications;
- desktop layout;
- wallpapers;
- menus;
- shortcuts;
- system defaults; and
- Bugema-specific utilities.

## Rationale

Xfce provides a mature graphical desktop environment while requiring fewer system resources than some full-featured desktop environments.

This is important because Bugema Linux is intended to support a mixture of:

- modern university computers;
- older laboratory machines;
- student laptops;
- virtual machines; and
- resource-constrained environments.

The decision also supports the project's objective of extending the useful life of existing university computer hardware.

## Alternatives Considered

### GNOME

GNOME provides a modern and highly integrated desktop experience and is the default desktop of Ubuntu Desktop.

However, its hardware resource requirements and desktop design make Xfce more aligned with the initial Bugema objective of supporting a broad range of university hardware.

GNOME may still be evaluated for a future optional Bugema desktop profile.

### KDE Plasma

KDE Plasma provides extensive customization and a powerful desktop environment.

However, the initial project requires a simpler baseline focused on resource efficiency and maintainability.

KDE may be evaluated as an optional environment in future releases.

### LXQt

LXQt is lightweight and could support very low-resource computers.

However, Xfce provides a more mature balance between resource efficiency, functionality, configurability, and institutional desktop usability for the initial target environment.

### Developing a Custom Desktop

Developing a Bugema-specific desktop environment would create substantial development and maintenance requirements.

The project will instead customize an established open-source desktop.

## Consequences

### Positive

- Lower resource requirements.
- Good suitability for university laboratories.
- Support for older hardware.
- Mature open-source project.
- Extensive configuration options.
- Suitable environment for student customization projects.
- Reduced desktop development and maintenance burden.

### Negative

- Bugema Linux depends on the upstream Xfce project.
- Some users may prefer GNOME or KDE Plasma.
- Certain modern desktop features may require additional configuration.
- The Bugema team must monitor compatibility between Xfce and Ubuntu 26.04 LTS.

## Implementation

The Alpha 0.1 desktop will include the required Xfce components and Bugema-specific configuration.

Bugema-specific desktop customizations should be packaged where practical rather than maintained as undocumented manual changes.

Potential Bugema packages include:

```text
bugema-desktop
bugema-branding
bugema-wallpapers
bugema-welcome
