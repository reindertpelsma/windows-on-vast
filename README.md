<!-- SPDX-License-Identifier: ISC -->
# Windows on Vast.ai

Turn a fresh Vast.ai KVM rental into Windows with public-key SSH, NVIDIA drivers
and the CUDA toolkit. Windows runs directly in the rented VM with its assigned GPU.

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

**Experimental:** this exact standalone script still needs an unattended
fresh-template Windows/SSH/CUDA pass. Offline checks have passed. Its automatic
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

## License

SPDX-License-Identifier: **ISC**. See [LICENSE](LICENSE).
Upstream reinstall and downloaded software retain their own licenses.
