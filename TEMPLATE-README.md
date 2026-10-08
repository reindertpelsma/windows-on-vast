<!-- SPDX-License-Identifier: ISC -->
# Windows 11 on Vast.ai

Automatically replace a fresh Linux KVM rental with **Windows 11 Enterprise LTSC
2024 Evaluation**, public-key SSH, NVIDIA GeForce 580.88 and CUDA Toolkit 13.0.0.
Windows runs directly in the rented VM with its assigned GPU.

**Validated end to end (2026-10-08):** on a fresh rental (RTX 4090, 150 GB disk)
the unattended installation completed without manual repairs: three Windows
boots, public-key SSH, NVIDIA GeForce 580.88 (`nvidia-smi` lists the GPU) and CUDA
13.0.48 (`nvcc`), with `state.json` showing `complete: true`. An earlier bug that
left Windows running without SSH (an OpenSSH installer hang caused by argument
quoting) is fixed. A real CUDA workload has not been tested.

**Installation erases the entire Linux boot disk. Use a fresh rental.**

Before renting, add your SSH public key to your Vast account. Choose an x86-64
KVM offer with **150 GB disk**. Automatic driver setup targets supported
Turing-and-newer GeForce
cards; datacenter GPUs may need another driver. No purchased Windows license
is included.

**Wait for three Windows boots** in the instance log before expecting SSH.
You should see `bootx64.efi` (WinPE) once, then `Windows Boot Manager` twice:

```
BdsDxe: starting Boot0003 "bootx64.efi" from HD(1,GPT,FC58A6B9-60F8-4BC3-8F23-A95DCD6C32C9,0x800,0x200000)/\EFI\boot\bootx64.efi
BdsDxe: starting Boot0004 "Windows Boot Manager" from HD(1,GPT,5ED4A909-18F6-4DE2-A940-F21C792CA8F9,0x800,0x32000)/\EFI\Microsoft\Boot\bootmgfw.efi
BdsDxe: starting Boot0004 "Windows Boot Manager" from HD(1,GPT,5ED4A909-18F6-4DE2-A940-F21C792CA8F9,0x800,0x32000)/\EFI\Microsoft\Boot\bootmgfw.efi
```

(The disk GUIDs differ on every rental.) After the second `Windows Boot Manager`
line, Windows still needs several minutes to finish first-logon setup before SSH
answers. Two boots are not enough; do not stop the instance early.

Allow the downloads, installation and automatic reboots to finish. Connect with
your original public IP and mapped SSH port, using username **`vast`**:

```sh
ssh -p PUBLIC_SSH_PORT vast@PUBLIC_IP
```

Save the Windows SSH host key printed before reboot and verify it when connecting.
Windows uses public-key authentication only. Password SSH and RDP are disabled.

**SSH is ready before the GPU software. Do not use the GPU yet.** After SSH works,
Windows still downloads and installs the NVIDIA driver and then CUDA, which takes
several more minutes and may reboot once or twice (SSH drops briefly each time;
reconnect). `nvidia-smi` and `nvcc` do not exist until those stages finish. Wait
until `state.json` shows `"complete": true` and the log ends with `SETUP_COMPLETE`:

```powershell
Get-Content C:\ProgramData\VastWindows\state.json
Get-Content C:\ProgramData\VastWindows\setup.log -Tail 30
```

If `error` in `state.json` is not empty, setup failed and is not retried until the
next reboot; send that error and the log tail when asking for help.

Once `complete` is true, open a new SSH session and run `nvidia-smi` and
`nvcc --version`. Setup checks GPU health and driver/compiler versions; test your
CUDA workload separately.

[Source and instructions](https://github.com/reindertpelsma/windows-on-vast)
· SPDX-License-Identifier: **ISC**
· [License](https://github.com/reindertpelsma/windows-on-vast/blob/main/LICENSE)

Upstream reinstall and downloaded software retain their own licenses.
