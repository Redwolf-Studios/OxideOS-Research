# OxideOS Research

Research, experiments, and architectural analysis behind **OxideOS**.

This repository documents the process of investigating Linux distributions, technologies, and system architecture to understand how they work and determine what should—or should not—be incorporated into OxideOS.

## Research Method

OxideOS research follows:

**Analyze → Learn → Adapt → Evaluate → Incorporate / Adapt / Pass / Investigate Further**

The goal is not to copy another distribution.

Instead, each technology and design decision is examined on its own merits:

- What problem does it solve?
- How does it work?
- What are its dependencies?
- Is it reliable?
- Is it maintainable?
- Does OxideOS actually need it?
- Is there a simpler or lighter alternative?
- What are the tradeoffs?

A technology being used by another distribution does not automatically make it a requirement for OxideOS.

## Current Research

### Ubuntu

Ubuntu is currently being dissected to understand:

- Linux desktop architecture
- systemd and systemd targets
- system and user services
- display managers
- GNOME architecture
- D-Bus
- XDG desktop portals
- package and dependency relationships
- desktop session startup
- PipeWire and Linux audio architecture

### Fedora

Previous research investigated:

- Fedora live image construction
- Kickstart
- Anaconda
- Lorax
- livemedia-creator
- LiveOS and squashfs
- bootloader configuration
- installation environments
- runtime image construction

The Fedora research is being retained as part of the project's historical and architectural investigation.

## Why This Repository Exists

Building a Linux distribution requires understanding more than how to install packages or assemble an ISO.

OxideOS is intended to be:

- User-friendly
- Lightweight
- Modern
- Reliable
- Maintainable
- Developer-friendly
- Transparent
- Reproducible

The research in this repository is used to make those goals concrete technical decisions rather than assumptions.

## OxideOS Philosophy

OxideOS will not be designed by simply choosing a distribution and copying its architecture.

The project will study multiple approaches, compare them, test them, and determine what fits the goals of OxideOS.

Some technologies may be incorporated.

Some may be adapted.

Some may be rejected.

Some may require further investigation.

The research process is part of the project.

## Repository Structure

```text
oxideos-research/
├── README.md
├── Ubuntu/
│   ├── README.md
│   ├── Desktop-Architecture.md
│   ├── Experiments/
│   └── Handoffs/
├── Fedora/
├── Comparisons/
└── OxideOS-Decisions/
```

The structure will evolve as the research expands.

## Status

**Active research**

Current focus: **Ubuntu architecture and PipeWire**

---

**OxideOS — RedWolf Studios**
