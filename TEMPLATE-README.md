<!-- SPDX-License-Identifier: ISC -->
# Windows 11 on Vast.ai

Renting this template gives you **Windows 11 Enterprise LTSC 2024 Evaluation**
with public-key SSH, NVIDIA GeForce driver 580.88 and CUDA Toolkit 13.0.0. Windows
runs directly in the rented KVM VM (150 GB disk) with its assigned GPU.

## How to use

1. **Add your SSH public key to your Vast account before you rent.** The keys
   registered at rental time are the only ones Windows gets; a key added later is
   **not** installed.
2. Rent the template. It starts a KVM instance with a 150 GB disk and installs
   Windows automatically; nothing else is required.
3. **Wait for three Windows boots** in the instance log. You should see
   `bootx64.efi` (WinPE) once, then `Windows Boot Manager` twice (the disk GUIDs
   differ on every rental):

   ```
   BdsDxe: starting Boot0003 "bootx64.efi" from HD(1,GPT,FC58A6B9-60F8-4BC3-8F23-A95DCD6C32C9,0x800,0x200000)/\EFI\boot\bootx64.efi
   BdsDxe: starting Boot0004 "Windows Boot Manager" from HD(1,GPT,5ED4A909-18F6-4DE2-A940-F21C792CA8F9,0x800,0x32000)/\EFI\Microsoft\Boot\bootmgfw.efi
   BdsDxe: starting Boot0004 "Windows Boot Manager" from HD(1,GPT,5ED4A909-18F6-4DE2-A940-F21C792CA8F9,0x800,0x32000)/\EFI\Microsoft\Boot\bootmgfw.efi
   ```

   Two boots are not enough. After the second `Windows Boot Manager` line Windows
   still needs several minutes before SSH answers.
4. Connect with your key, the rental's public IP and mapped SSH port, user `vast`:

   ```sh
   ssh -p PUBLIC_SSH_PORT vast@PUBLIC_IP
   ```

   Password SSH and RDP are disabled. The Windows SSH host key is printed in the
   log before the first reboot; verify it on first connect (the intermediate
   installer uses a different key).
5. **Do not use the GPU yet.** SSH works before the NVIDIA driver and CUDA are
   installed, which takes several more minutes and may reboot once or twice (SSH
   drops briefly; reconnect). Wait until `state.json` shows `"complete": true` and
   the log ends with `SETUP_COMPLETE`:

   ```powershell
   Get-Content C:\ProgramData\VastWindows\state.json
   Get-Content C:\ProgramData\VastWindows\setup.log -Tail 30
   ```

   Then open a new SSH session and run `nvidia-smi` and `nvcc --version`. Setup
   checks GPU health and driver/compiler versions only; test your own CUDA
   workload.

## Good to know

- **The Linux disk is replaced.** The on-start script erases the rental's boot
  disk and installs Windows over it; that is the purpose of the template, and a
  new rental has nothing on it to lose. If SSH shows Linux with a warning banner
  instead of Windows, the install is still running or failed: do not install or
  store anything there; destroy the instance and rent again. Never run the script
  on a machine that holds data you care about.
- **Evaluation Windows, no license included.** The image is Microsoft's 90-day
  evaluation build; after it expires Windows shuts down every hour. A purchased
  license is not included.
- **GPUs:** the driver targets supported Turing-and-newer GeForce cards.
  Datacenter GPUs may need a different driver.
- **Failures are not retried.** If `error` in `state.json` is not empty, setup
  failed and will not run again until the next reboot. Report the error and the
  `setup.log` tail.
- **Status:** the unattended install, SSH, driver and CUDA setup were validated end
  to end on 2026-10-08 on a fresh RTX 4090 rental. A real CUDA workload has not
  been tested.
- **Where to look:** `journalctl -fu vast-reinstall` in the original Linux,
  `/reinstall.log` in the intermediate installer, and the files above in Windows.

[Source and instructions](https://github.com/reindertpelsma/windows-on-vast)
· SPDX-License-Identifier: **ISC**
· [License](https://github.com/reindertpelsma/windows-on-vast/blob/main/LICENSE)

Upstream reinstall and downloaded software retain their own licenses.
