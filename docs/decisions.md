# Decisions

Why things are set up the way they are, including the calls I'd make differently.
Most of these came from getting it wrong first.

---

## mergerfs instead of RAID

**Chose:** pool two disks with mergerfs into one mount point. No parity, no
striping.

**Why:** my disks are different sizes and different ages, and they were bought at
different times. RAID wants matched disks and a rebuild when you add one.
mergerfs just presents whatever disks you give it as a single directory tree, and
adding another one later is trivial. Each disk also keeps a normal filesystem, so
if a drive dies the files on the *other* drive are still readable. With striped
RAID a failure can take everything.

**The trade:** no redundancy at all. A dead disk means restoring from backup, not
rebuilding live. I covered that with backups instead of parity, which is a
deliberate choice about where to spend money. RAID buys you uptime, not safety,
and for a home media server I'd rather have a second copy than a faster recovery.

**Would I change it:** no, but I'd add parity eventually. It's the right call for
mixed hardware.

## No forwarded ports, ever

**Chose:** a WireGuard mesh VPN for all private access, and nothing forwarded on
the router.

**Why:** forwarding a port means something on my home connection is listening to
the entire internet and getting scanned constantly. A mesh VPN puts my devices
and the server on the same private network without exposing anything. Access is
tied to device identity rather than to whoever finds the port.

**The trade:** anything I want to reach has to be on the VPN, which means the VPN
has to be working. In practice this has never been the problem I expected.

## One service public, through an outbound tunnel

**Chose:** publish exactly one service to the internet, using an outbound-only
tunnel rather than an open port.

**Why:** the tunnel connects outward from my server to the provider, so there's
still nothing inbound on my home IP. The routing is scoped to that single
hostname, and every other hostname gets a 404. I verified that rather than
assuming it.

**The trade:** I depend on a third party for that one service. Acceptable, given
it's the only thing exposed and losing it breaks nothing important.

## Backup drives deliberately outside the pool

**Chose:** the two backup disks are not mergerfs members.

**Why:** if the backup target is part of the pool, it isn't a backup. A mistake
that deletes from the pool deletes from the backup at the same time. Keeping them
separate means a bad command has to be run twice to destroy both copies.

This is written in large letters in my own notes because it's exactly the kind of
thing that looks tidy to "fix" later.

## Photo library on a single disk, not the pool

**Chose:** the photo server's library lives on one physical disk rather than in
the pooled mount.

**Why:** it's a database-backed application doing a lot of small writes. I wanted
its storage predictable and on the SSD rather than spread across a pool where a
given file could land on the slower disk.

**The catch I hit:** that disk is also a pool member, so my pool-mirroring backup
job was copying the photo library twice. The backup script now excludes it and
handles it separately. Worth knowing if you do the same thing.

## Downloads and library under one mount

**Chose:** the media automation containers get a single volume containing both
the download folder and the library, rather than two separate volumes.

**Why:** hardlinks only work within a single mount point. Mounting the two
directories separately means the applications can't hardlink between them and
silently fall back to copying, which wastes disk and breaks seeding. One mount,
with both directories as paths underneath it, fixes it.

A full write-up of how I found this is coming, since it went unnoticed for three
months.

## IoT devices mapped but not integrated

**Chose:** the smart-home and camera devices get accounted for in the network
plan, but are deliberately not pulled into monitoring or automation.

**Why:** they're cloud-first products that are genuinely difficult to self-host
or integrate, and the effort-to-value ratio is terrible. I'd rather spend that
time on segmentation, which actually reduces risk, than on making a doorbell
report to a dashboard.

Knowing what not to build is a decision too.

## SSH key-only

**Chose:** password authentication disabled on the server.

**Why:** it removes brute-force as a threat entirely rather than mitigating it.

**The trade, which bit me:** a device that didn't already have a key couldn't
connect, and I didn't notice until I tried to SSH in from my phone two weeks
later. The fix was to use the VPN's own identity-based SSH, which authenticates
by device rather than by key and works from anywhere without managing keys per
device.

Lesson: when you close a door, check what you were still using it for.

## Abandoned: upgrading to the spare rig

**Chose:** stopped planning around a spare desktop I'd been treating as the
future server.

**Why:** it wouldn't display anything. I reseated the GPU and RAM, reset CMOS,
found dried thermal paste, and eventually took it to a repair shop, who
diagnosed the motherboard as dead. The CPU could never be tested because the
board won't POST.

I'd been building plans on top of that machine for months, including moving the
SIEM onto it. All of those plans were resting on an assumption I hadn't
verified. Now it's a parts donor and the plans have been rewritten.

Worth writing down because the lesson isn't about hardware. It's that I had
several months of roadmap depending on something I had never confirmed worked.
