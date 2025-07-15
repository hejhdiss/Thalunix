# Thalunix: A Conceptual QEMU Manager

**Concept and Design by Muhammed Shafin P**  
GitHub: [@hejhdiss](https://github.com/hejhdiss)

---

## Overview

Thalunix is a conceptual design for an open-source graphical user interface (GUI) built on top of QEMU. The aim is to create a lightweight, user-friendly virtualization tool primarily for Windows users, simplifying VM management by providing an intuitive graphical alternative to QEMU's command-line interface.

The name **Thalunix** combines *Thalassa* (Greek for "sea") and *Unix*, representing depth, stability, and flow in design and function.

This is a **conceptual project**, meaning there is currently no implementation. If you are interested in building this idea or deriving from it, please read the licensing and attribution section below.

---

## Project Goals

- Offer a clear and intuitive interface to manage QEMU virtual machines
- Provide lightweight installation and portable usability
- Focus initially on Windows, with plans to expand cross-platform
- Ensure simplicity while allowing access to advanced QEMU features
- Encourage modular and community-driven growth

---

## Key Design Components (Conceptual)

```
+------------------------------------------------+
Thalunix GUI
[PySide6/Qt UI] [Console Output Panel]
[VM Config Editor] [ISO/Image Manager]
[Snapshot Control] [Network Settings]
+------------------------------+-----------------+
QEMU Command Builder
+------------------------------+-----------------+
QEMU Engine (CLI or Libvirt)
+------------------------------------------------+
```

---

## Core Conceptual Features

- Virtual machine creation, editing, and management from GUI
- Bootable ISO mounting via file picker or drag-and-drop
- Live QEMU console output window embedded in UI
- Disk image creation in QCOW2 or RAW format
- VM snapshot manager for saving and reverting states
- VM cloning (configuration and image duplication)
- Integration with WHPX (Windows Hypervisor Platform) for acceleration
- Networking support (User-mode NAT, Bridged modes)
- UEFI firmware selection (standard or Secure Boot-enabled OVMF)

---

## Planned and Proposed Features

- Persistent UEFI variable support (OVMF_VARS.fd)
- swtpm (software TPM) integration for Secure Boot and Windows 11 support
- Guest agent features (file sharing, shutdown signals)
- Virtual USB hotplugging
- Download center for popular ISOs (Ubuntu, Fedora, etc.)
- Portable mode (zero-install binary)
- Advanced QEMU hardware passthrough options
- Built-in diagnostics and QEMU error logs
- Custom network definition interface
- Libvirt integration as an optional backend
- Multiple theme support including dark mode

---
## TPM Integration via Lightweight Micro-VM (Windows)

To support Secure Boot and Windows 11 compatibility, Thalunix plans to integrate TPM 2.0 using `swtpm`. Since native support for `swtpm.exe` on Windows is limited or complex to maintain, we propose an alternative approach using a minimal Linux micro-VM that runs `swtpm` as a TCP server.

This helper VM runs transparently in the background and communicates with QEMU over the loopback network interface. This strategy ensures performance and portability while avoiding heavyweight dependencies or complex setup.

### Architecture Diagram

```
           +------------------------------------+
           |    Lightweight Linux Micro-VM      |
           |  (Alpine or Debian with swtpm)     |
           +------------------------------------+
           | swtpm --tpm2                       |
           |   --server type=tcp,port=2321      |
           |   --ctrl   type=tcp,port=2322      |
           |   --tpmstate dir=/var/swtpm        |
           +------------------------------------+
                      |  Loopback (127.0.0.1)
                      v
+--------------------------------------------------------------+
| QEMU Host |
| |
| -chardev socket,id=chrtpm,host=127.0.0.1,port=2321 |
| -tpmdev emulator,id=tpm0,chardev=chrtpm |
| -device tpm-tis,tpmdev=tpm0 |
| |
| Guest VM (e.g., Windows 11) |
+--------------------------------------------------------------+
```

### How It Works

1. **A minimal Linux VM** (such as Alpine) is launched automatically in the background using QEMU. It has `swtpm` pre-installed and runs a system service that listens on TCP ports `2321` (TPM communication) and `2322` (control channel).

2. **The Thalunix-managed VM** connects to this micro-VM via QEMU's `-chardev socket` and `-tpmdev emulator` options.

3. **Persistent TPM state** is stored inside the micro-VM image, allowing consistent TPM behavior across VM reboots.

4. **All communication** is securely routed through the loopback interface (`127.0.0.1`), ensuring no external exposure.

### Benefits of This Approach

- **Cross-Platform Compatibility**: sidesteps the need for native swtpm binaries on Windows.
- **Lightweight**: the helper VM can run with less than 64 MB RAM and start in under 2 seconds.
- **Isolation and Security**: no TPM access is exposed beyond the host.
- **Persistence**: supports TPM state retention required by Windows 11 setup and Secure Boot.
- **Modular**: this micro-VM can be reused or replaced independently from the main VM.

### Optional Enhancements (Future Work)

- Provide multiple helper VM images (Alpine, Debian, TinyCore)
- Built-in toggle in GUI to enable or disable swtpm integration
- Background management of the helper VM (auto-shutdown when not in use)
- swtpm log monitoring and error reporting in the Thalunix console

> This design allows full TPM emulation on Windows with minimal performance impact, without relying on complex or native builds of `swtpm` for Windows.

---

## Supported Virtualization Backends (Planned)

- **WHPX** – Windows Hypervisor Platform (Windows 10/11)
- **KVM** – Kernel-based Virtual Machine (Linux)
- **HVF** – Hypervisor.framework (macOS, planned)
- **HAXM** – Intel HAXM (deprecated, optional legacy support)
- **TCG** – Tiny Code Generator (software fallback, slow)
- **libvirt** – Planned optional integration for advanced users

---

## Known Limitations in Early Stages

- UEFI variable persistence is not implemented initially
- Secure Boot and TPM are unsupported in early designs
- Certain OS installations (e.g., Windows 11) may not function without full UEFI + TPM stack
- This is a design document, and no stable version exists yet

---

## Illustrative UI Mockup (Conceptual, Text-Based)

```
+----------------------------------------------+
| [ Thalunix - Virtual Machine ] |
+----------------------------------------------+
| Name: [UbuntuTestVM ] |
| ISO: [ubuntu-24.04.iso] [Browse...] |
| Disk: [Create New] [Load Existing] |
| RAM: [2048 MB] CPU Cores: [2] |
| Firmware: [Standard OVMF] [Secure OVMF] |
| TPM: [Disabled] |

Network: [User-NAT] [Bridged]
[Start VM] [Pause] [Stop] [Snapshot]
+----------------------------------------------+
Console Output:
>> Booting Ubuntu...
>> Systemd initialized
+----------------------------------------------+
```

---

## Contributing and Collaboration

Although Thalunix is not yet implemented, contributions are welcome in the form of:

- Feature suggestions and architectural feedback
- UI mockups and design proposals
- Python/PySide6 prototype development
- Documentation and translation
- Research on QEMU and libvirt integration

To start contributing:

1. Fork this repository
2. Explore the conceptual structure and open issues
3. Propose your own ideas or improvements
4. Open a pull request to share your work

---

## Attribution and Licensing

This is a **conceptual design project** created by **Muhammed Shafin P**. It is **not released under a permissive software license for source code reuse**, but is licensed under:

**Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)**  
Details: [https://creativecommons.org/licenses/by-sa/4.0/](https://creativecommons.org/licenses/by-sa/4.0/)

### You are free to:
- Share — copy and redistribute this material in any medium or format
- Adapt — remix, transform, and build upon the material for any purpose, even commercially

### Under the following terms:
- **Attribution** — You must give appropriate credit to *Muhammed Shafin P* and link to the original concept or GitHub repository.
- **ShareAlike** — If you remix, transform, or build upon this concept, you must distribute your contributions under the same license as the original.

> **Note:** This license covers the *concept, text, diagrams, and structure* provided in this README, not executable code.

---

## Final Notes

Thalunix is not a finished application but a well-defined design direction. It is intended to spark development and collaboration toward a modern, simple virtual machine manager built on QEMU.

If you are developing a QEMU GUI or similar virtualization tool and this concept inspired you, please consider crediting:

**Concept by: Muhammed Shafin P**  
**GitHub:** [https://github.com/hejhdiss](https://github.com/hejhdiss)

---

Let Thalunix serve as a stable and modern foundation for accessible virtualization design.
