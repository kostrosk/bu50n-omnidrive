# OmniDrive on the LG BU50N CC04

This guide records one successful Windows installation on a BU50N with the MediaTek MT1959 platform.
The drive reported BU40N 1.00 and OmniDrive v1.0.4 after complete power removal.
The owner also confirmed tray eject.
Game-disc reading was not tested for this guide.

## Why

SDFTool reported successful writes but left this drive on CC04.
Redumper b767 changed the firmware identity and passed the final power-removal test.
The exact cause of the earlier SDFTool failures remains unknown.
No custom firmware code was necessary.

> **CAUTION:** A firmware write can make the drive unusable. This setup has no proven recovery procedure.

## Tested hardware

| Part | Tested item |
|---|---|
| Drive | [JOYIN-BURN BU50N, Amazon B0FK5DBJKG](https://www.amazon.com/dp/B0FK5DBJKG) |
| USB cable and supply | [BENFEI SATA-to-USB 3.0, Amazon B07JGT17B8](https://www.amazon.com/dp/B07JGT17B8) |
| Drive connector | [YARCHONN slimline SATA adapter, Amazon B0GXD6S88P](https://www.amazon.com/dp/B0GXD6S88P) |

The adapter changes the 13-pin slimline connector to a 22-pin SATA connector.
The BENFEI cable includes a 12 V, 2 A supply.

The tested drive label and software showed:

| Item | Value |
|---|---|
| Physical model | HL-DT-ST BD-RE BU50N |
| Label code | ANCK1N0 |
| Original firmware | CC04, date 2120-05-29 |
| Drive platform | MT1959 |
| Boot mark | `MT1959 Boot BU51` |
| Hardware / mechanism | A / A |
| Factory | GH |
| Manufacture date | January 2022 |
| USB bridge identifier | `152D:0583` |

An Amazon listing can contain different hardware revisions.
Match the label, platform and original firmware before you use this procedure.
This result does not prove compatibility with every BU50N.

## Steps

### 1. Prepare the drive and tools

Remove the disc.
Connect the drive through the slimline adapter and BENFEI cable.
Connect both BENFEI external power and USB.

Get [MakeMKV](https://www.makemkv.com/download/).
The tested version was 2.0.0.

Get [redumper b767 for Windows x64](https://github.com/superg/redumper/releases/download/b767/redumper-b767-windows-x64.zip).
Extract the complete archive under `C:\BU50N\redumper`.
Keep its files together.

The archive SHA-256 must be:

```text
90c195fdc95323ff49c40c9902d90fb00b3ec63345d03a75b1bbc9dbac79b20f
```

On your Windows computer, in PowerShell, from `C:\BU50N`:

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath '.\redumper-b767-windows-x64.zip'
$redumper = (Get-ChildItem '.\redumper' -Filter redumper.exe -Recurse).FullName
if (@($redumper).Count -ne 1) { throw 'Keep exactly one redumper.exe in this folder.' }
& $redumper --help
& 'C:\Program Files (x86)\MakeMKV\makemkvcon64.exe' f -l
```

Make sure that the archive hash matches.
Make sure that the drive reports BU50N CC04.
Set `$drive` to the drive letter from the output.
Record your serial number locally so that you can identify the same unit later.

On your Windows computer, in the same PowerShell window, from `C:\BU50N`:

```powershell
$drive = 'E:' # Replace E: with your actual drive letter.
& 'C:\Program Files (x86)\MakeMKV\sdftool.exe' -d $drive --info
New-Item -ItemType Directory -Force '.\backup' | Out-Null
& 'C:\Program Files (x86)\MakeMKV\sdftool.exe' -d $drive dump auto -o 'C:\BU50N\backup'
```

The information must show MT1959.
On this CC04 drive, a full firmware dump was disabled.
The saved dump contained user data only.
That dump is not a complete recovery image.

### 2. Make the OmniDrive image

Firmware files are not included here.
Use your own stock files and the [OmniDrive v1.0.4 build instructions](https://github.com/RibShark/OmniDrive/tree/v1.0.4#building).

Install Git, CMake 3.26 or newer, Ninja, the `arm-none-eabi-gcc` toolchain, armips, Python 3 and pycryptodome.
Put the command-line tools on your PATH.
PATH is the list of folders that Windows searches for programs.

On your Windows computer, in PowerShell, from `C:\BU50N`:

```powershell
git clone --branch v1.0.4 --depth 1 https://github.com/RibShark/OmniDrive.git
```

Put these exact stock files in `C:\BU50N\OmniDrive\firmware`:

| File | Required SHA-1 |
|---|---|
| `HL-DT-ST_BD-RE_BU40N_1.00.bin` | `9c48677b155154d24a3b95a32b4a29ca02ff40b3` |
| `ASUS_BW-16D1HT_3.02.bin` | `f8df5b579f25da8d4e5aa5ef79f3005dac5eb8c7` |

On your Windows computer, in PowerShell, from `C:\BU50N`:

```powershell
Get-FileHash -Algorithm SHA1 -LiteralPath '.\OmniDrive\firmware\HL-DT-ST_BD-RE_BU40N_1.00.bin'
Get-FileHash -Algorithm SHA1 -LiteralPath '.\OmniDrive\firmware\ASUS_BW-16D1HT_3.02.bin'
```

Make sure that both hashes match before you continue.

On your Windows computer, in PowerShell, from `C:\BU50N`:

```powershell
cmake -S '.\OmniDrive' -B '.\OmniDrive\build' -G Ninja
cmake --build '.\OmniDrive\build'
$firmware = 'C:\BU50N\OmniDrive\patched_firmware\HL-DT-ST_BD-RE_BU40N_1.00_OmniDrive-1.0.4.bin'
$expected = '0BD77FED4EC9DF86AAB9D8550F3182A8A95F4C27EE8342119B1B1F9F2C8303E0'
if ((Get-Item -LiteralPath $firmware).Length -ne 2097152) { throw 'Wrong firmware size.' }
if ((Get-FileHash -Algorithm SHA256 -LiteralPath $firmware).Hash -ne $expected) { throw 'Wrong firmware hash. Stop.' }
```

The tested image was 2,097,152 bytes with the SHA-256 shown in the command.
A hash match identifies the tested file; it does not remove the write risk.
If your output differs, stop and examine the build inputs.

### 3. Write the firmware once

Close MakeMKV and other drive programs.
Keep both USB and external power connected.
Make sure that `$drive` still identifies your empty BU50N.

> **CAUTION:** The next command writes immediately after its internal checks. It has no confirmation prompt.

On your Windows computer, in the same PowerShell window, from `C:\BU50N`:

```powershell
& $redumper flash::mt1959 "--drive=$drive" "--firmware=$firmware"
```

Do not add `--force-flash` or a boot-mark override.
The tested command needed neither option.
Redumper selected the current-drive boot mark and transmission encryption.

The successful run showed programming progress, then:

```text
flashing success
```

If the command fails, save its output and stop.
Do not repeat writes with guessed options.

### 4. Remove all drive power

Wait for the write command to finish.
Disconnect USB.
Disconnect BENFEI external power.
Wait 30 seconds.
Reconnect external power.
Reconnect USB.

Windows eject only opens the tray.
It does not remove drive power.

## Verify

On your Windows computer, in PowerShell, from `C:\BU50N`:

```powershell
& 'C:\Program Files (x86)\MakeMKV\makemkvcon64.exe' f -l
```

Find the drive again.
Its letter can change after reconnection.
Set `$drive` to its current letter.

On your Windows computer, in the same PowerShell window, from `C:\BU50N`:

```powershell
$drive = 'E:' # Replace E: with the current drive letter.
& 'C:\Program Files (x86)\MakeMKV\sdftool.exe' -d $drive --info
& $redumper eject "--drive=$drive" --image-path='C:\BU50N' --image-name=after-power-cycle
```

The last command also opens the tray.
Do not use `--force-omnidrive` for this test.

The expected results are:

```text
MakeMKV drive: BD-RE BU40N, firmware 1.00
Drive-specific LibreDrive status: Enabled
Redumper configuration: MTK8B
Redumper firmware: OmniDrive v1.0.4
```

Make sure that the serial number still matches your unit.
The drive-specific SDF status controls the LibreDrive result.
The generic identification section can still say `Possible, not yet enabled`.
MakeMKV can also show `Original (unpatched)` while redumper reads the genuine OmniDrive marker.

These results prove firmware persistence for the tested unit.
They do not prove a successful Xbox dump.
Do a separate read test with your own game disc before you rely on the drive.

## Rollback

No complete CC04 recovery image or proven restore procedure was available for this unit.
The original partial dump cannot restore arbitrary damaged firmware.
Keep the backup, image hashes and command output locally.
If the drive stops responding, stop before another write.

## Sources and credit

- [OmniDrive v1.0.4](https://github.com/RibShark/OmniDrive/tree/v1.0.4): firmware modification and build instructions.
- [Redumper b767](https://github.com/superg/redumper/releases/tag/b767): the tool used for the successful write.
- [Pinned redumper MT1959 source](https://github.com/superg/redumper/blob/642bdec05c83c52bbfb94f09a81e710118f96370/drive/flash_mt1959.ixx): compatible boot groups, encryption and transfer.
- [ibizara's packed-firmware tool](https://github.com/ibizara/OmniDrive_BU40N_1.03/blob/b4271133f427fe7986dca50a7a07d871dd1c76cd/bu40n_mtk_packed_tool.py): used for earlier offline inspection.

The packed-firmware tool was not needed for the successful redumper command.
This repository contains instructions only, not firmware or a new flasher.

## GitHub mirror

Push changes to your Forgejo repository.
Use Forgejo's push-mirror settings when you choose to mirror it to GitHub.
This guide contains no private network addresses, drive serial numbers or personal file paths.
