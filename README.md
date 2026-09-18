# NVIDIA GPU Fan Controller for Ubuntu

[繁體中文](README.zh-TW.md)

A simple Bash utility for controlling NVIDIA GPU fan speeds on Ubuntu using `nvidia-settings`.

It automatically detects the fan controllers exposed by the NVIDIA driver, starts a root Xorg session when needed, and supports both manual fan speed control and restoring NVIDIA automatic fan control.

## Features

- Automatically detects available NVIDIA fan controllers
- Set all detected GPU fans to a fixed speed
- Restore NVIDIA automatic fan control
- View GPU temperature, fan speed, power draw, target fan speed, and RPM
- Starts a root Xorg session automatically when required
- Stops GDM before starting the dedicated Xorg session
- Can restore the Ubuntu graphical login screen
- Works well for headless / SSH Ubuntu machines

Tested with GPUs such as:

- NVIDIA GeForce RTX 3090
- NVIDIA GeForce RTX 5090

The number of controllable fan channels depends on the GPU and NVIDIA driver.

## Requirements

- Ubuntu
- NVIDIA proprietary driver
- `nvidia-smi`
- `nvidia-settings`
- Xorg
- `xauth`
- `mcookie`
- NVIDIA Xorg configuration with `Coolbits` fan control enabled

Install required packages if needed:

```bash
sudo apt update
sudo apt install nvidia-settings xserver-xorg xauth
```

Your NVIDIA device section in `/etc/X11/xorg.conf` should contain:

```text
Option "Coolbits" "4"
```

Example:

```text
Section "Device"
    Identifier "Device0"
    Driver "nvidia"
    Option "Coolbits" "4"
EndSection
```

## Installation

Download or copy the `gpufan` script to:

```bash
/usr/local/bin/gpufan
```

Then make it executable:

```bash
sudo chmod +x /usr/local/bin/gpufan
```

Verify:

```bash
which gpufan
```

Expected output:

```text
/usr/local/bin/gpufan
```

## Usage

Set all detected GPU fans to 99%:

```bash
gpufan 99
```

Set all detected GPU fans to 80%:

```bash
gpufan 80
```

View current status:

```bash
gpufan status
```

Restore NVIDIA automatic fan control:

```bash
gpufan auto
```

Restore automatic fan control and stop the root Xorg session:

```bash
gpufan stop
```

Restore the Ubuntu GUI / GDM:

```bash
gpufan gui
```

## Example

```text
$ gpufan 99

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

A GPU with three exposed fan controllers may instead show:

```text
Detected 3 controllable fans: 0 1 2
```

## How It Works

On some Ubuntu systems, `nvidia-settings` can read fan information through the GDM Xorg session but fails to write `GPUTargetFanSpeed`.

This utility works around that situation by:

1. Stopping GDM
2. Starting Xorg as root
3. Connecting `nvidia-settings` to that X server
4. Enabling `GPUFanControlState`
5. Detecting available `fan:X` targets
6. Setting each detected fan to the requested speed

## Notes

- This script currently controls `gpu:0`.
- The physical number of fans on a graphics card may differ from the number of fan controllers exposed by the NVIDIA driver.
- Running `gpufan` may stop the Ubuntu graphical desktop because GDM is stopped before the dedicated Xorg session starts.
- Use `gpufan gui` to restore GDM.
- Fan settings may be reset after reboot.
- Do not assume that maximum fan speed is necessary for normal workloads.

## Disclaimer

Use this utility at your own risk.

Manual fan control overrides the NVIDIA driver's normal fan policy. Monitor GPU temperature and hardware behavior when changing cooling settings.

## License

Choose a license appropriate for your project, such as MIT.
