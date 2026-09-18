# Homelab

A self-hosted server I've been running for about a year. It started as something
to do with an old office PC and turned into the reason I'm moving into
cybersecurity.

This repo documents how it's put together, the decisions behind it, and the parts
I got wrong.

---

## How it started

I pulled a dusty office PC out of storage. A 2009 quad-core with 8 GB of RAM,
sitting unused, and I went looking for something practical to do with it.

I wiped Windows, installed Debian, and SSH'd in from my desktop. That was the
first time I'd used Linux that way. It worked, and I immediately wanted to know
what else I could do with it.

A year later it runs about thirteen containers, serves my household, and has
taught me more about Linux, Docker, storage and networking than anything else
I've done. It's also what pushed me toward security work, because once you're
running something people depend on, you start caring about who can reach it and
what happens when it breaks.

## What it runs

| | |
|---|---|
| **Host** | Dell OptiPlex 5050 SFF (i7-7700, 32 GB RAM), Debian, headless, managed over SSH. Moved off the original 2009 PC in September 2026. |
| **Containers** | ~13, managed through CasaOS |
| **Media** | Jellyfin for video, plus the automation stack that manages the library |
| **Music** | Self-hosted music server, ~106 GB and around 18,000 tracks. Built to get off streaming subscriptions. |
| **Photos** | Immich, single user, replacing cloud photo storage |
| **DNS** | Pi-hole filtering for the whole house |
| **Monitoring** | Uptime checks with chat alerting, plus resource telemetry |
| **Security** | A single-node Wazuh SIEM on a separate VM, with agents on Linux, macOS and Windows |

My girlfriend uses it. My dad watches things on it on the TV in the living room.
That part matters more than it sounds, because it means downtime is a real
problem and not just an inconvenience for me.

## Storage

Two disks pooled with **mergerfs** into a single ~8 TB mount point, plus two
dedicated backup drives kept deliberately outside the pool.

mergerfs presents separate disks as one directory tree without striping. It is
not RAID and there is no parity. That was a deliberate choice, and the reasoning
is in [docs/decisions.md](docs/decisions.md).

Backups are two nightly cron jobs. One tars the application config with
versioned retention, the other mirrors the media with rsync. Both report a
heartbeat to the uptime monitor when they finish, so a job that silently stops
running raises an alert instead of quietly doing nothing.

## Access

**No ports are forwarded on my router.** Nothing is listening on my home IP.

Everything private goes over a WireGuard mesh VPN, which puts my phone and
laptops on the same private network as the server from anywhere. One service is
published to the internet through an outbound-only tunnel, scoped to that single
hostname with a catch-all 404 for everything else.

SSH is key-only. Password authentication is disabled.

## Documentation

- **[docs/network.md](docs/network.md)**: network topology, how the containers
  are mounted, and the two paths in from outside. Diagrams included.
- **[docs/decisions.md](docs/decisions.md)**: the "why this and not that"
  choices, including the ones I'd make differently.

Related repo: **[soc-home-lab-wazuh](https://github.com/mzarem01/soc-home-lab-wazuh)**,
the SIEM build with a validated SSH brute-force detection.

---

## What I got wrong

**Start with better hardware.** This is the big one. A 2009 office PC was free
and I learned a lot fighting it, but the board had no USB 3.0 at all, so every
external drive was capped at USB 2.0 speeds. The CPU couldn't transcode video. You
can find a much faster used machine for very little money, and you will not spend
your evenings working around a sixteen-year-old motherboard. If I were starting
again I'd buy a cheap used small-form-factor PC and skip the headache. In September 2026 I finally did exactly that.

**I built monitoring and backups after the first time something failed quietly,
not before.** Both should have gone in on day one. Every problem I've had was
found because I happened to look, not because anything told me.

**My network is flat.** Everything sits on one segment, including IoT devices,
which is not where you want a camera and a server sharing a broadcast domain. I
didn't think about segmentation until well after everything was running, and now
the fix needs hardware my router can't provide.

**I assumed my documentation was accurate.** It wasn't. Addresses drifted, notes
went stale, and I found things in my own inventory that hadn't been true for
weeks. Verify against the system, not against your notes.

## What's next

- Network segmentation, which needs a VLAN-capable router since mine has no VLAN
  support at all
- An offsite backup copy, so the setup is actually 3-2-1 and not just two copies
  in the same house
- Alerting on services that are running but unhealthy, not just services that are
  down
- fail2ban on the server, then re-running an attack against it to show detection
  and remediation end to end

---

## A note on sanitization

**All IP addresses, hostnames, domains and identifiers in this repo are
illustrative.** They're internally consistent so the diagrams make sense, but
they aren't my real values. Nothing here exposes a routable address, a tunnel
hostname, an API key or a serial number.

If you're documenting your own setup publicly: **git history is permanent.**
Committing a real secret and removing it in a later commit does not delete it.
Sanitize before the first commit.
