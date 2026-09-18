# NVIDIA GPU Fan Control for Headless Ubuntu

[繁體中文](README.zh-TW.md)

A lightweight Bash utility for controlling NVIDIA GPU fan speeds on **headless Ubuntu servers** over SSH.

It uses `nvidia-settings` with a dedicated root Xorg session, automatically detects the fan controllers exposed by the NVIDIA driver, and supports fixed fan speeds, status monitoring, restoring automatic fan control, and restoring GDM when needed.

> Designed primarily for headless / SSH Ubuntu systems where NVIDIA fan control through the normal GDM Xorg session may fail.

## Features

- Designed for headless Ubuntu / SSH servers
- Automatically detects available NVIDIA fan controllers
- Automatically generates a minimal headless `/etc/X11/xorg.conf` when one does not exist
- Automatically detects the PCI Bus ID of `gpu:0`
- Enables `Coolbits=4` and `AllowEmptyInitialConfiguration` in the generated Xorg config
- Never overwrites an existing `/etc/X11/xorg.conf`
- Set all detected GPU fans to a fixed speed
- Restore NVIDIA automatic fan control
- View GPU temperature, fan percentage, power draw, target fan speed, and RPM
- Automatically starts a dedicated root Xorg session when required
- Stops GDM before starting the dedicated Xorg session
- Can restore the Ubuntu graphical login screen with one command
- No need to hard-code the number of GPU fans

Tested with:

- NVIDIA GeForce RTX 3090
- NVIDIA GeForce RTX 5090

The number of controllable `fan:X` targets depends on the GPU model and NVIDIA driver.

## Requirements

- Ubuntu
- NVIDIA proprietary driver
- `nvidia-smi`
- `nvidia-settings`
- Xorg
- `xauth`
- `mcookie`

Install the required packages if needed:

```bash
sudo apt update
sudo apt install nvidia-settings xserver-xorg xauth
```

### Xorg configuration

You do **not** need to manually create `/etc/X11/xorg.conf` on a fresh headless system.

If the file does not exist, `gpufan` automatically:

1. Reads the PCI Bus ID of `gpu:0` from `nvidia-smi`
2. Converts it to the Xorg `BusID` format
3. Creates a minimal headless Xorg configuration
4. Enables:

```text
Option "Coolbits" "4"
Option "AllowEmptyInitialConfiguration" "True"
```

If `/etc/X11/xorg.conf` already exists, `gpufan` will **not overwrite it**. The existing NVIDIA device configuration should have `Coolbits=4` enabled for manual fan control.

## Installation

Clone the repository:

```bash
git clone https://github.com/breeze0305/Nvidia-GPU-Fan-Control-Headless.git
cd Nvidia-GPU-Fan-Control-Headless
```

Install the script system-wide:

```bash
sudo install -m 0755 gpufan /usr/local/bin/gpufan
```

Verify the installation:

```bash
which gpufan
```

Expected output:

```text
/usr/local/bin/gpufan
```

You can now immediately run:

```bash
gpufan 99
```

On the first run, a headless Xorg configuration will be generated automatically if one is missing.

## Usage

Set all detected GPU fans to 99%:

```bash
gpufan 99
```

Set all detected GPU fans to 80%:

```bash
gpufan 80
```

View the current GPU and fan status:

```bash
gpufan status
```

Restore NVIDIA automatic fan control:

```bash
gpufan auto
```

Restore automatic fan control and stop the dedicated root Xorg session:

```bash
gpufan stop
```

Restore the Ubuntu GUI / GDM:

```bash
gpufan gui
```

## First-run Example

```text
$ gpufan 99

[INFO] /etc/X11/xorg.conf not found
[INFO] Detecting GPU 0 PCI Bus and creating a headless Xorg configuration...
[OK] Created /etc/X11/xorg.conf
[INFO] GPU: NVIDIA GeForce RTX 3090
[INFO] PCI Bus: 00000000:01:00.0 -> PCI:1:0:0
[INFO] Stopping GDM...
[INFO] Creating Xauthority...
[INFO] Starting root Xorg :0...
[OK] Xorg started
[INFO] Detected 2 controllable fans: 0 1
[INFO] Enabling NVIDIA manual fan control...
[INFO] Fan 0 -> 99%
[INFO] Fan 1 -> 99%

[OK] 2 NVIDIA fans have been set to 99%
```

A GPU exposing three independent fan targets may instead show:

```text
Detected 3 controllable fans: 0 1 2
```

## How It Works

On some Ubuntu systems, the Xorg session started by GDM can read NVIDIA fan information but fails when writing `GPUTargetFanSpeed`.

This utility works around that behavior by:

1. Creating a minimal headless Xorg configuration when one is missing
2. Detecting the PCI Bus ID of `gpu:0`
3. Stopping GDM
4. Starting a dedicated Xorg session as root
5. Connecting `nvidia-settings` to that X server
6. Enabling `GPUFanControlState`
7. Detecting available `fan:X` targets
8. Setting every detected fan target to the requested speed

## Important Notes

- The script currently controls `gpu:0`.
- Automatic Xorg config generation also targets `gpu:0`.
- Existing `/etc/X11/xorg.conf` files are never overwritten automatically.
- The physical number of fans on a graphics card may differ from the number of fan controllers exposed by the NVIDIA driver.
- Running `gpufan <speed>` may stop the current Ubuntu graphical desktop because GDM is stopped before the dedicated root Xorg session starts.
- Use `gpufan gui` to stop the dedicated Xorg session and restore GDM.
- Fan settings may reset after reboot.
- This tool is intended primarily for machines managed through SSH / headless environments.

## Disclaimer

Use this utility at your own risk.

Manual fan control overrides the NVIDIA driver's normal fan policy. Monitor GPU temperature and hardware behavior when changing cooling settings.
