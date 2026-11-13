# How to unlock an RTX 3060 for AI with nv-uefi-unlocker

This is the procedure used for the **"unlocked"** setups in the [benchmark results](https://shisa84.github.io/rtx3060-unlocked-vs-rtx5050-comfyui/). On those tests, unlocking cut generation time by 19 to 38% on the stock profile.

It relies on [nv-uefi-unlocker](https://github.com/physicsdolphin/nv-uefi-unlocker), a third-party open-source project. Read its [official documentation](https://github.com/physicsdolphin/nv-uefi-unlocker/blob/main/README.en.md) before starting.

> [!WARNING]
> This is an unofficial mod, not supported by NVIDIA. It requires turning off Secure Boot. Use it at your own risk.

## What to know first

- **Only AI workloads benefit**: local LLMs, image and video generation. Gaming FPS do not change.
- **Nothing is written to the card.** The unlock is applied at boot from a USB stick and is lost at every full power-off. You repeat Part B at each startup.
- **Supported cards**: the RTX 30 series (Ampere). Other cards in the same PC are ignored.
- **Tested on**: RTX 3060 driving the monitor, with an RTX 5050 alongside (not supported, ignored). Asus TUF Gaming X570-Plus motherboard, Windows, nv-uefi-unlocker v2.1.11.
- **If your Windows drive is encrypted with BitLocker, read the next section first.** Otherwise you may be locked out of your PC.

## Before you start: BitLocker

> [!CAUTION]
> Without your BitLocker recovery key, these steps can lock you out of Windows and your data.

### The problem

BitLocker keeps the key of your Windows drive in the motherboard's security chip (TPM). The TPM only releases it if the PC boots exactly as it did when BitLocker was set up: same Secure Boot state, same boot program.

This guide changes both:

- **Part A, step 5** turns off Secure Boot.
- **Part B** starts the PC from a USB stick instead of starting Windows directly.

BitLocker sees these changes as a possible attack. At the next startup, instead of Windows, you get a blue **BitLocker recovery** screen that asks for a **48-digit recovery key**. Without that key, the drive cannot be opened: reinstalling Windows erases everything on it.

Because the unlock is redone from the USB stick at every startup, the recovery screen can come back often, for example each time you switch between a boot with the stick and a normal boot.

BitLocker is often turned on without you knowing it. Many Windows 11 PCs enable "Device encryption" automatically when you sign in with a Microsoft account.

### Check whether BitLocker is on

Start menu → type `powershell` → right-click → **Run as administrator**, then run:

```powershell
manage-bde -status C:
```

- `Protection Status: Protection Off` and `Percentage Encrypted: 0.0%`: BitLocker is off. Nothing to do, go to Part A.
- `Protection Status: Protection On`: BitLocker is on. Follow the steps below **before** touching the BIOS.

### If BitLocker is on

1. **Save your recovery key.** Do this first, whatever you decide next.
   - If you use a Microsoft account, the key is usually stored online at https://aka.ms/myrecoverykey.
   - You can also display it with the command below. Copy the 48-digit **Numerical Password**, and keep it outside this PC: on paper or on your phone.

     ```powershell
     manage-bde -protectors -get C:
     ```

2. **Suspend BitLocker before changing the BIOS.** The drive stays encrypted, but Windows stops checking the boot until you resume protection:

   ```powershell
   Suspend-BitLocker -MountPoint "C:" -RebootCount 0
   ```

   `-RebootCount 0` keeps it suspended until you resume it yourself.

3. Do Part A and Part B, and check that Windows starts normally with the unlock.

4. **Choose how to live with it**:
   - **Keep BitLocker suspended** while you use the unlock. No more recovery screens, but the drive key is stored unprotected on the disk. Anyone who takes the PC can read the data.
   - **Resume protection** with `Resume-BitLocker -MountPoint "C:"`. Your data is protected again, but expect the recovery screen again after boots with the stick. Keep the key at hand.
   - **Turn BitLocker off completely** (Settings → Privacy & security → Device encryption, or `manage-bde -off C:`). Same security trade-off as suspending, without having to manage it.

To go back to the original setup: stop booting from the stick, turn Secure Boot back on, then run `Resume-BitLocker -MountPoint "C:"`.

## Part A: Preparation (one time only)

### 1. Download

On the [releases page](https://github.com/physicsdolphin/nv-uefi-unlocker/releases), take the release marked **Latest**. Avoid versions ending in `-rc` (pre-releases).

Under **Assets**, download:
- `nv-uefi-unlocker-v2.1.11.zip`: the package
- `ssdiag-x86_64-windows-0.7.1.exe`: the verification tool

Do not download "Source code" or the Linux files. Version numbers may be newer than these; adapt the file names in the commands below.

Put both files in a folder, for example `Desktop\unlock`. Right-click the zip → **Extract All** → **Extract**.

### 2. Generate the USB files

1. Open the `nv-uefi-unlocker-v2.1.11` folder, the one that contains `make-usbroot.ps1`.
2. Click the File Explorer address bar, clear it, type `powershell` and press Enter.
3. Run this command (an internet connection is required):

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\make-usbroot.ps1 -Family 30
   ```

4. Wait for the **DONE** line. A `usbroot` folder is created.

The files are named `ga102`: that is normal, the name is shared by the whole 30 series.

### 3. Check that the card is supported (optional)

In the same PowerShell window:

```powershell
..\ssdiag-x86_64-windows-0.7.1.exe --inventory
```

The RTX 3060 must show **IN-SCOPE** in green. Unsupported cards, such as an RTX 5050, show **OUT-OF-SCOPE** in red.

### 4. Prepare the USB stick

1. The stick must be formatted as **FAT32**: right-click the stick → **Properties** → **File system**. If it is not, format it as FAT32. Formatting erases everything on it.
2. Open the `usbroot` folder, press **Ctrl+A**, copy, and paste **at the root of the stick**. Copy the contents, not the `usbroot` folder itself.

The stick should look like this:

```
USB stick (FAT32)
├── EFI\BOOT\BOOTX64.EFI   ← do not move it
└── the .bin files
```

### 5. Turn off Secure Boot

If BitLocker is on, save your recovery key and suspend BitLocker first (see [Before you start: BitLocker](#before-you-start-bitlocker)).

Steps for an Asus motherboard. Menu names differ on other brands.

1. Restart and press **Del** repeatedly at power-on to enter the BIOS.
2. Press **F7** for Advanced Mode.
3. Go to the **Boot** tab → **Secure Boot**.
4. Change **OS Type** from "Windows UEFI mode" to **Other OS**.
5. Press **F10** → **OK** to save and restart.

## Part B: At every PC startup

1. Plug in the USB stick.
2. Power on or restart, and press **F8** repeatedly at power-on to open the boot menu (Asus).
3. Select **UEFI: <your USB stick name>**, for example "UEFI: Kingston DataTraveler", and press Enter.
4. On the tool's version screen, press **Space** or **Enter** **within 30 seconds**. Otherwise it aborts and Windows boots normally, without the unlock.
5. **Do not touch anything.** A black or frozen screen for a few moments is normal.
6. Windows starts on its own. You can then remove the stick.

## Part C: Check the unlock

1. Start menu → type `powershell` → right-click → **Run as administrator**.
2. Run:

   ```powershell
   cd $env:USERPROFILE\Desktop\unlock
   .\ssdiag-x86_64-windows-0.7.1.exe
   ```

3. Scroll up to the RTX 3060 block and look for:
   - `VERDICT [ampere]: UNLOCKED` in green: the unlock is active.
   - `CANARY: intact`: nothing has cleared the unlock since boot.

Unsupported cards show `NOT_UNLOCKED`. That is normal, ignore them.

## Troubleshooting

- **Blue "BitLocker recovery" screen**: type the 48-digit recovery key you saved. Windows then starts normally. See [Before you start: BitLocker](#before-you-start-bitlocker) to avoid it next time.
- **Black screen that stays, STOP message or error**: do not retry right away. Power off completely, remove the stick and boot normally.
- **Going back to normal**: just stop booting from the stick. To turn Secure Boot back on: BIOS → **Boot** → **Secure Boot** → **OS Type** = "Windows UEFI mode".
- **Stops working after a driver update**: a future NVIDIA driver could block the method. Check the [project's GitHub page](https://github.com/physicsdolphin/nv-uefi-unlocker) for news.
