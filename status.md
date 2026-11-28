---
title: Status
description: Where the EmiBSD redesign stands, subsystem by subsystem - moved into the safe kernel crate, in progress, or still in the ported LandingZone tree.
---
{%- assign s = site.data.status -%}
{%- assign hal = s.crates | where: "name", "hal" | first -%}
{%- assign bsd = s.crates | where: "name", "bsd" | first %}
# Status

**Pre-alpha.** EmiBSD boots in QEMU on arm64 and runs OpenBSD's userland, but most of the
kernel is still the 1:1 port inherited from LandingZone. amd64 builds are frozen until milestone
N10. There is no release.

The goal is a kernel with no `unsafe`: today the `kernel` and `abi` crates hold **0**, the
hardware layer `hal` holds {{ hal.unsafe }} (each with a written proof), and the ported LandingZone tree,
`bsd`, still holds {{ bsd.unsafe }}, falling as each subsystem moves out. Figures from the
[Kernel]({{ site.repo }}) repository at commit `{{ s.commit }}` ({{ s.date }}), {{ s.at }}; the
[numbers per crate](#the-crates-today) are at the end of the page.

## What already works

Everything the port could do when Kernel split from it still works, and the redesign may not
change it. On arm64 in QEMU, with OpenBSD's own binaries:

- **System:** boot to `login:`, rc(8), ksh(1), ddb(4), multiprocessor boots on two and four CPUs.
- **Network:** ifconfig(8) and ping(8), pfctl(8), ipsecctl(8), TLS with ftp(1) and nc(1).
- **Storage:** FFS, tmpfs, FAT, ISO 9660, ext2fs and FUSE; softraid(4) RAID 0/1/5/6 and CRYPTO;
  NVMe and AHCI disks, USB storage.
- **Devices:** USB keyboards, audio, the frame buffer console.

Each line is checked by a scripted QEMU boot, the smoke tests, at every milestone close.

## Milestones

| Milestone | Scope | State |
|---|---|---|
{%- for m in s.milestones %}
| {{ m.name }} {{ m.title }} | {{ m.scope }} | {% if m.date %}<span class="state state-done">done</span> {{ m.date | slice: 0, 10 }}{% elsif m.state %}<span class="state state-wip">{{ m.state }}</span>{% else %}<span class="state state-todo">pending</span>{% endif %} |
{%- endfor %}

## {{ s.next.name }}, the milestone in progress

N7 moves every device driver QEMU's arm64 machine uses out of the LandingZone tree into the safe kernel, about 418,000 lines, the random device last.

The slices below are N7's plan of 2026-11-26; a slice may be split or renumbered when it starts.

| Slice | What moves | State |
|---|---|---|
{%- for sl in s.next.slices %}
| {{ sl.slice }} | {{ sl.what }} | {% if sl.state == "done" %}<span class="state state-done">done</span>{% elsif sl.state == "in progress" %}<span class="state state-wip">in progress</span>{% else %}<span class="state state-todo">pending</span>{% endif %} |
{%- endfor %}

N12.1, a performance slice brought forward, made the walks over a process's memory map 36-80 % faster.

N6, the milestone before, closed on 2026-11-26; its results are in
[its news entry]({{ '/news/2026/11/26/n6-met/' | relative_url }}). N5's are in
[its own]({{ '/news/2026/11/07/n5-met/' | relative_url }}).

## The crates today

Files and `unsafe` counts printed by `cargo xtask unsafe-report` at `{{ s.commit }}`.

| Crate | Files | `unsafe` | Note |
|---|---|---|---|
{%- for c in s.crates %}
| `{{ c.name }}`{% if c.label %} ({{ c.label }}){% endif %} | {{ c.files }} | {{ c.unsafe }} | {{ c.note }} |
{%- endfor %}
{: .numbers}

The detailed plan, with each milestone's exit criterion, is
[ROADMAP.md]({{ site.repo }}/blob/main/docs/ROADMAP.md) in the Kernel repository; the day-to-day state
is [STATUS.md]({{ site.repo }}/blob/main/docs/STATUS.md).
