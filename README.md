<!-- SPDX-License-Identifier: ISC -->
# Windows on Vast.ai

Turn a fresh Vast.ai KVM rental into Windows with public-key SSH, NVIDIA drivers
and the CUDA toolkit. Windows runs directly in the rented VM with its assigned GPU.

**Experimental — no unattended end-to-end pass yet (2026-10-05).** The successful
development boot using `reinstall` required manual repairs. A later fresh-template
attempt became unreachable after Windows Boot Manager. Treat the automation as
broken until it reproduces the repaired `reinstall` run without intervention;
the first investigation is which manual step the script omits or performs differently.

**[Open the Windows template on Vast.ai](https://cloud.vast.ai/?ref_id=527355&creator_id=527355&name=Windows%2011%20VM)**

## Use the template

1. Add your SSH public key to your Vast account before renting.
2. Choose a fresh x86-64 KVM rental. Request **150 GB disk**.
3. Let installation and its automatic reboots finish. **The entire Linux boot
   disk, including all partitions, is erased.**
4. Connect using your key, the original public IP/mapped SSH port, and user `vast`:

```sh
ssh -p PUBLIC_SSH_PORT vast@PUBLIC_IP
```

Save the Windows SSH host key printed before reboot and verify it when connecting.
The intermediate installer has a different host key. SSH can become available
before the NVIDIA/CUDA installation finishes.

| Component | Included |
| --- | --- |
| Windows | Windows 11 Enterprise LTSC 2024 Evaluation |
| Access | Public-key SSH; password SSH and RDP disabled |
| GPU software | GeForce 580.88 and CUDA Toolkit 13.0.0 |

The driver targets supported Turing-and-newer GeForce cards. Datacenter GPUs may
need a different driver. A purchased Windows license is not included.

Offline checks have passed. Its automatic
checks cover device health and driver/compiler versions; a real CUDA workload
must be tested separately.

## Template configuration

- Image: `docker.io/vastai/kvm`.
- On-start script: paste the **entire [install.sh](install.sh)**.
- ReadMe: paste [TEMPLATE-README.md](TEMPLATE-README.md).

The script is readable text under Vast's 16,384-byte limit. It embeds our setup
code and does not download from this repository. It uses pinned upstream
[bin456789/reinstall](https://github.com/bin456789/reinstall/tree/80c3d5e175f39c2d2bbd267cd583842140154140)
for WinPE/Windows Setup, networking and VirtIO drivers. OS and package downloads
remain necessary. No nested Windows preparation VM is used.

Inspect the embedded files without installing anything:

```sh
bash install.sh --render-only /tmp/vast-windows-review
```

Follow progress with `journalctl -fu vast-reinstall` in original Linux,
`/reinstall.log` in the intermediate installer, and
`C:\ProgramData\VastWindows\setup.log` / `state.json` in Windows. The Windows
scheduled task is `VastWindowsSetup`; `complete: true` means its setup checks passed.

## Bootstrap audit — 2026-10-05

The baseline is our manually repaired `reinstall` development run, not the older
nested-QEMU workflow. The two recorded repairs are already in the failed template:

| Check | Finding |
| --- | --- |
| Exact ISO image name | [Line 93](install.sh#L93) includes `LTSC 2024 Evaluation`. |
| WIM payload copy | [Line 131](install.sh#L131) uses `cp -r`, retaining the WIM/FUSE repair. The separate `cp -a` copies into the Linux initramfs. |
| Generated installer hooks | Comparing the preserved repaired wrapper with this script produced byte-identical `reinstall.sh`; `trans.sh` differed by one blank line. Payload preservation and SetupComplete insertion remain present. |
| Failed template contents | The saved 16,359-byte template matches this 16,360-byte script except for its final newline. No truncation or stale pre-repair template was found. |

The Windows bootstrap was substantially rewritten after the development run:
native-process helpers, scheduled-task identity/retry settings, and state handling
changed; GPU setup moved into the same task. That rewrite has not reproduced the
successful boot unattended. It remains the next comparison to validate, not an
established explanation of the failure. GPU/compiler version checks also do not
replace a real CUDA workload.

The failed run saved no Windows setup, task or network logs. Task registration
occurs [before transcript logging](install.sh#L192), so a registration failure can
leave no bootstrap transcript. The successful run likewise lacks a complete
manual-command transcript; an additional unrecorded intervention cannot be ruled
out. No further omitted manual step or exact failure cause has been established,
and the evidence does not justify blaming the provider.

This audit changed documentation only. Before another success claim, reproduce
the repaired baseline from a fresh install and preserve Windows setup and task/SSH
evidence even if networking fails. No speculative installer fix was published.

## License

SPDX-License-Identifier: **ISC**. See [LICENSE](LICENSE).
Upstream reinstall and downloaded software retain their own licenses.
