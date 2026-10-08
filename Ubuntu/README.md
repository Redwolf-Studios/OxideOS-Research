# OxideOS Research: Excavating Ubuntu's Desktop Architecture

**Project:** OxideOS  
**Research Branch:** Ubuntu 26.04  
**Date:** October 7, 2026  
**Status:** Research / Investigation

---

## Why We Are Doing This

OxideOS is not being built by simply taking another Linux distribution and changing its wallpaper, packages, or name.

The goal is to understand how Linux distributions actually work, determine which ideas are useful, and then decide what belongs in OxideOS.

Our research process is:

> **Analyze → Learn → Adapt → Ask whether it suits the build → Incorporate / Adapt / Pass / Investigate further**

Ubuntu 26.04 is currently being used as one of our research subjects.

This investigation started with a simple question:

> **What actually happens between Linux booting and the user seeing a graphical desktop?**

The answer turned out to be considerably more interesting than expected.

---

# 1. The First Discovery: The Desktop Is a Stack

One of the first assumptions we had to discard was the idea that a Linux desktop is essentially one large program.

It isn't.

Our investigation produced a model roughly like this:

```text
Hardware
   ↓
Linux Kernel
   ↓
System systemd
   ↓
graphical.target
   ↓
Display Manager
   ↓
Login
   ↓
User systemd
   ↓
graphical-session.target
   ↓
Desktop Session
   ↓
Desktop Environment
   ↓
Desktop Services
   ↓
Executables
   ↓
Packages
```

Ubuntu's graphical environment is therefore not one monolithic system.

It is a collection of layers that cooperate with one another.

That distinction is important to OxideOS because it gives us a better way to reason about the operating system.

Instead of asking:

> "What desktop should OxideOS use?"

we can also ask:

> "What responsibilities belong to the operating system, the user session, the desktop environment, and the applications?"

---

# 2. System systemd vs User systemd

One of the most useful discoveries was that there are effectively two systemd environments involved.

At the system level, systemd manages things such as:

```text
NetworkManager
GDM
graphical.target
multi-user.target
system services
```

After the user logs in, a user-level systemd instance manages much of the user's graphical session.

The simplified model becomes:

```text
SYSTEM SYSTEMD
      ↓
     GDM
      ↓
    LOGIN
      ↓
USER SYSTEMD
      ↓
DESKTOP SESSION
```

This provides an important separation between the operating system itself and the user's desktop environment.

That separation is something worth considering when designing OxideOS.

---

# 3. What Is a systemd Target?

During the investigation we encountered several targets:

```text
graphical.target
basic.target
graphical-session.target
gnome-session.target
```

At first glance, these names can make targets look like programs.

They aren't.

A useful mental model is:

> **A systemd target is a checkpoint or grouping mechanism used to organize other units.**

For example:

```text
graphical.target
```

represents the system entering its graphical operating state.

It can pull in or coordinate other components needed to reach that state.

Likewise:

```text
graphical-session.target
```

represents the user's graphical session.

The target itself isn't drawing windows or running a desktop.

It is helping organize the components that make the session possible.

---

# 4. The Display Manager

Ubuntu's graphical boot path led us to GDM.

The simplified chain is:

```text
systemd
   ↓
graphical.target
   ↓
display-manager.service
   ↓
GDM
   ↓
login screen
```

One interesting detail is that the higher-level system refers to:

```text
display-manager.service
```

rather than hard-coding every component as "GDM."

That provides an abstraction layer.

The system needs a display manager.

The specific implementation can be GDM, SDDM, LightDM, or another compatible display manager.

This is another architectural pattern worth studying for OxideOS:

> **Use generic interfaces where practical, while allowing specific implementations underneath them.**

---

# 5. GNOME Is a Collection of Components

GNOME turned out to be considerably more modular than the simple mental model of "GNOME = desktop."

We found separate components responsible for things such as:

```text
Session management
Desktop shell
Power management
Keyboard handling
Media keys
Accessibility
Sharing
RFKill
Smartcard support
USB protection
```

For example, GNOME's power functionality follows a chain similar to:

```text
GNOME session
      ↓
GNOME basic services
      ↓
Power target
      ↓
Power service
      ↓
/usr/libexec/gsd-power
      ↓
gnome-settings-daemon package
```

This demonstrates that a desktop environment is really a collection of coordinated subsystems.

---

# 6. D-Bus

D-Bus appeared repeatedly throughout the investigation.

The simplest explanation is:

> **D-Bus is a communication system that lets programs and services talk to one another.**

Instead of every application needing to directly understand every other application, they can communicate through defined interfaces.

Conceptually:

```text
Program A
    ↓
  D-Bus
    ↓
Program B
```

This is an important part of modern Linux desktop architecture.

It is also something OxideOS will need to understand before deciding whether to build any replacement infrastructure.

---

# 7. XDG Desktop Portals

The graphical session also exposed another interesting abstraction:

```text
xdg-desktop-portal.service
xdg-desktop-portal-gnome.service
xdg-desktop-portal-gtk.service
```

The basic idea is that applications can use standardized desktop-facing interfaces rather than being tightly coupled to one particular desktop environment.

Conceptually:

```text
Application
    ↓
XDG Portal
    ↓
Desktop-specific implementation
    ↓
System functionality
```

This is particularly interesting for OxideOS because we eventually want the Oxide application suite to work as a coherent ecosystem without unnecessarily tying every application to one specific desktop environment.

---

# 8. We Learned How to Trace a Feature Back to Its Package

One of the most useful techniques discovered during the investigation was the ability to work backward from a running service to the package that provides it.

For example:

```text
systemd service
      ↓
ExecStart
      ↓
executable
      ↓
dpkg -S
      ↓
owning package
```

We used:

```bash
dpkg -S /usr/libexec/gsd-power
```

to determine which package provided the executable.

This gives us a practical method for dissecting Linux distributions.

If we encounter a feature we want to understand, we can ask:

1. What behavior am I seeing?
2. Which service provides it?
3. What executable does that service run?
4. Which package owns that executable?
5. What dependencies does that package have?
6. Is the feature actually necessary for OxideOS?

This is becoming one of our core research techniques.

---

# 9. "Minimal" Does Not Necessarily Mean Minimal

Ubuntu also provided an important lesson about terminology.

The package/task:

```text
ubuntu-desktop-minimal
```

does not mean:

> "The smallest possible graphical Linux installation."

It means something closer to:

> "A smaller version of Ubuntu's intended desktop environment."

It still involves major GNOME components.

This is an important distinction for OxideOS.

If OxideOS eventually uses the term **minimal**, we need to define exactly what that means.

For us, minimal should probably be based on:

- Required functionality
- Hardware support
- Maintainability
- Reliability
- Security
- User experience
- Development requirements

rather than simply copying another distribution's package selection.

---

# 10. Enabled Does Not Mean Running

Another useful systemd lesson came from GDM.

A service can be:

```text
disabled
```

while also being:

```text
active (running)
```

These states describe different things.

**Enabled** means the service has been configured to start automatically through the normal boot dependency mechanism.

**Running** means the service is currently active.

Something else can start a service even when that service itself isn't enabled.

Therefore:

```text
disabled + running
```

is perfectly valid.

This is a small detail, but understanding it prevents a lot of confusion when debugging Linux systems.

---

# 11. A More Complete Picture

After the investigation, our working model looks like this:

```text
                    HARDWARE
                       ↓
                  LINUX KERNEL
                       ↓
                 SYSTEM SYSTEMD
                       ↓
                graphical.target
                       ↓
             display-manager.service
                       ↓
                      GDM
                       ↓
                     LOGIN
                       ↓
                 USER SYSTEMD
                       ↓
             graphical-session.target
                       ↓
                 GNOME SESSION
                       ↓
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      GNOME Shell   Settings     Other Services
                     Daemon
          ↓            ↓            ↓
       D-Bus        D-Bus        D-Bus
          └────────────┼────────────┘
                       ↓
                  Applications
                       ↓
                    Packages
```

This is not intended to be a literal representation of every dependency.

It is a conceptual map of what we discovered.

---

# 12. What This Means for OxideOS

We have **not** decided to copy Ubuntu.

We have learned several architectural patterns worth investigating.

### Generic infrastructure

Use established Linux infrastructure where it makes sense instead of reinventing everything.

### Modular services

Keep independent responsibilities separated when doing so improves maintainability.

### Clear boundaries

Distinguish between:

```text
System services
```

and:

```text
User/session services
```

when practical.

### Standard interfaces

Investigate established technologies such as:

```text
systemd
D-Bus
XDG portals
PipeWire
```

before designing replacements.

### Package awareness

Understand exactly why a package exists and what depends on it.

---

# 13. What We Have Not Decided

Nothing becomes part of OxideOS merely because Ubuntu uses it.

Every candidate technology must eventually answer questions such as:

```text
Does OxideOS need it?

Is it reliable?

Is it maintainable?

Does it improve compatibility?

Does it add unnecessary complexity?

Is there a lighter alternative?

Does it fit our design goals?

Would we still want it if Ubuntu didn't use it?
```

That last question is particularly important.

The goal is not:

> Build Ubuntu again.

The goal is:

> **Build OxideOS.**

---

# 14. The Next Investigation: PipeWire

The next major area of investigation is PipeWire.

The initial question is simple:

```bash
apt policy pipewire
```

From there we intend to trace:

```text
PipeWire package
      ↓
Installed files
      ↓
Executables
      ↓
User services
      ↓
Sockets / D-Bus
      ↓
Desktop session
      ↓
Audio / Video / Media
```

PipeWire is particularly important to OxideOS because the future Oxide software ecosystem includes:

```text
Oxide Record
Oxide DAW
Oxide Editor
Oxide Studios
```

Audio and video infrastructure will therefore be a major part of the eventual operating system design.

---

# 15. Why This Research Is Public

OxideOS is intended to be an open-source project.

That means the reasoning behind the system should ideally be open as well.

This research is not presented as authoritative Linux documentation.

It is an engineering notebook.

We're documenting:

- Questions
- Experiments
- Commands
- Observations
- Discoveries
- Mistakes
- Hypotheses
- Decisions
- Things we rejected
- Things we still need to investigate

The goal is to make the development process understandable and reproducible.

---

# 16. Current Status

At this point we have a working understanding of the Ubuntu graphical stack:

```text
Ubuntu
  ↓
systemd
  ↓
graphical.target
  ↓
GDM
  ↓
login
  ↓
user systemd
  ↓
graphical-session.target
  ↓
GNOME session
  ↓
GNOME services
  ↓
D-Bus / portals
  ↓
executables
  ↓
packages
```

The next major investigation is:

```text
PIPEWIRE
```

From there, the research will eventually move toward comparing the architecture we've discovered across multiple Linux distributions.

---

# OxideOS Research Philosophy

> **Analyze → Learn → Adapt → Ask whether it suits the build → Incorporate / Adapt / Pass / Investigate further.**

We're not building Ubuntu.

We're not building Fedora.

We're studying Linux.

Then we're going to build **OxideOS**.
