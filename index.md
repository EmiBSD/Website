---
title: Home
description: EmiBSD, an operating system in Rust - OpenBSD's kernel ported one-to-one, then redesigned in three layers with unsafe confined to the hardware layer. Runs in QEMU.
---
<div class="home" markdown="1">

<div class="hero">
  <picture>
    <source srcset="{{ '/assets/img/ferffy-lines-white.svg' | relative_url }}" media="(prefers-color-scheme: dark)">
    <img src="{{ '/assets/img/ferffy-lines-black.svg' | relative_url }}" alt="Ferffy, the EmiBSD mascot: a centolla, drawn in line art" width="150" height="112">
  </picture>
  <div>
    <h1>EmiBSD <span class="badge">pre-alpha</span></h1>
    <p class="tagline">A port of the OpenBSD -current kernel to Rust.</p>
    <p class="sub">OpenBSD's own programs run on it; inside, the kernel is being rebuilt subsystem by subsystem, with zero <code>unsafe</code> as the goal.</p>
    <p class="cta"><a class="button" href="#try-it-in-qemu">Try it in QEMU</a> <a class="button ghost" href="{{ site.repo }}">View on GitHub</a> <a class="button ghost" href="{{ '/docs/' | relative_url }}">Build it</a></p>
  </div>
</div>

<div class="notice" markdown="1">
**Not for production.** EmiBSD boots only in QEMU, today on arm64 alone. Its crypto, network and
storage code is unaudited. Do not use it to protect real data or real networks.
</div>

EmiBSD is an operating system written in Rust, built from OpenBSD's kernel in two steps.

**1. A faithful port.** The OpenBSD kernel is first translated to Rust one-to-one, file by file
and function by function, keeping its behaviour exactly. Everything a program can see matches
OpenBSD: the system calls, the error codes, the devices. That's why OpenBSD's own programs run
on it.

**2. A redesign in three layers.** The port is then rebuilt, piece by piece, into three parts
with a strict order between them:

| Layer | What it is | `unsafe` |
|---|---|---|
| **ABI** (*Application Binary Interface*) | the contract with programs: the system-call numbers, the data layouts, the error codes. It only translates; it decides nothing. | none, enforced by the compiler |
| **kernel** | every decision of the operating system: processes, memory, files, the network | none, enforced by the compiler |
| **HAL** (*Hardware Abstraction Layer*) | the only code that touches the hardware directly: registers, memory pages, locks. It offers the kernel a safe interface. | the only place allowed, each use justified in writing |

In Rust, `unsafe` marks the code where the compiler can't check memory safety on its own, and
those blocks are where errors like reading out of bounds or using freed memory can happen. The
aim is to shrink the `unsafe` code and confine it to the HAL, so the rest of the kernel is safe
by construction.

**A whole system, installable.** Around the kernel, [Userland]({{ site.repo_userland }}) builds
OpenBSD's base system for EmiBSD, [DevTools]({{ site.repo_devtools }}) builds the compilers (LLVM
and Rust), and [OS]({{ site.repo_os }}) assembles everything into an install ISO you can run in
QEMU (below).

<ul class="cards">
  <li><a href="{{ '/about/' | relative_url }}"><strong>About</strong><span>Why it exists, how it started, and what Rust brings.</span></a></li>
  <li><a href="{{ '/process/' | relative_url }}"><strong>Process</strong><span>From the 1:1 port to the redesign, and who does the work.</span></a></li>
  <li><a href="{{ '/status/' | relative_url }}"><strong>Status</strong><span>Each kernel subsystem: moved, in progress or pending.</span></a></li>
  <li><a href="{{ '/contributing/' | relative_url }}"><strong>Contributing</strong><span>Issues, target hardware, and a short FAQ.</span></a></li>
</ul>

## Try it in QEMU

Tested on macOS on Apple Silicon (`brew install qemu`). Each
[OS release]({{ site.repo_os }}/releases) carries `emibsd-<version>-arm64-install.iso` and its
`SHA256`.

**1. Download and check the ISO.** `V` is the release's tag, as the releases page lists it:

```sh
V=2026.11
U=https://github.com/EmiBSD/OS/releases/download/$V
curl -fLO $U/emibsd-$V-arm64-install.iso -fLO $U/SHA256 &&
  shasum -a 256 --ignore-missing -c SHA256
```

**2. Prepare a disk and the firmware:**

```sh
Q=$(brew --prefix qemu)/share/qemu
truncate -s 3g disk.img
cp $Q/edk2-arm-vars.fd vars.fd
VM="qemu-system-aarch64 -M virt,acpi=off -cpu cortex-a72 -m 1024M -smp 2
  -display none -serial mon:stdio
  -drive if=pflash,format=raw,readonly=on,file=$Q/edk2-aarch64-code.fd
  -drive if=pflash,format=raw,file=vars.fd
  -drive if=none,format=raw,file=disk.img,id=hd0 -device virtio-blk-device,drive=hd0
  -netdev user,id=n0 -device virtio-net-device,netdev=n0"
```

**3. Boot the installer:**

```sh
$=VM -device virtio-scsi-device \
  -drive if=none,format=raw,media=cdrom,readonly=on,file=emibsd-$V-arm64-install.iso,id=cd0 \
  -device scsi-cd,drive=cd0,bootindex=0
```

(`$=VM` splits the variable into words in zsh, the macOS shell; in bash write `$VM`.)

**4. Answer the installer.** Press Enter for the defaults, except:

| Question | Answer |
|---|---|
| `(I)nstall, (U)pgrade, (A)utoinstall or (S)hell?` | `i` |
| `Which disk is the root disk?` | `sd0` |
| `Location of sets?` | `cd0` |
| `Set name(s)?` | `done` |
| `Continue without verification?` | `yes` |
| `Exit to (S)hell, (H)alt or (R)eboot?` | `h` |

When it says `The operating system has halted.`, leave QEMU with `Ctrl-A X`.

**5. Boot the installed system:** run `$=VM` alone, without the ISO, and log in as `root` with
the password you gave the installer.

The USB stick image, Linux, and how to make the media yourself:
[OS's README]({{ site.repo_os }}#run-it-in-qemu).

## Latest news

{% include news-list.html limit=5 %}

</div>
