<!-- SPDX-License-Identifier: ISC -->
# Windows 11 on Vast.ai

Automatically replace a fresh Linux KVM rental with **Windows 11 Enterprise LTSC
2024 Evaluation**, public-key SSH, NVIDIA GeForce 580.88 and CUDA Toolkit 13.0.0.
Windows runs directly in the rented VM with its assigned GPU.

**Installation erases the entire Linux boot disk. Use a fresh rental.**

Before renting, add your SSH public key to your Vast account. Choose an x86-64
KVM offer with **150 GB disk**. Automatic driver setup targets supported
Turing-and-newer GeForce
cards; datacenter GPUs may need another driver. No purchased Windows license
is included.

Allow the downloads, installation and automatic reboots to finish. Connect with
your original public IP and mapped SSH port, using username **`vast`**:

```sh
ssh -p PUBLIC_SSH_PORT vast@PUBLIC_IP
```

Save the Windows SSH host key printed before reboot and verify it when connecting.
Windows uses public-key authentication only. Password SSH and RDP are disabled.

**SSH may be ready before the GPU software.** In Windows, check progress with:

```powershell
Get-Content C:\ProgramData\VastWindows\state.json
Get-Content C:\ProgramData\VastWindows\setup.log -Tail 30
```

After setup completes, open a new SSH session and run `nvidia-smi` and `nvcc --version`. Setup checks GPU health and driver/compiler versions; test your CUDA
workload separately.

Experimental: this exact standalone script is awaiting a complete fresh-template
Windows/SSH/CUDA validation.

[Source and instructions](https://github.com/reindertpelsma/windows-on-vast)
· SPDX-License-Identifier: **ISC**
· [License](https://github.com/reindertpelsma/windows-on-vast/blob/main/LICENSE)

Upstream reinstall and downloaded software retain their own licenses.
