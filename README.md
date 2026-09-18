# NVIDIA GPU Fan Control for Headless Ubuntu

[繁體中文](README.zh-TW.md)

A lightweight Bash utility for controlling NVIDIA GPU fan speeds on **headless Ubuntu servers** over SSH.

It uses `nvidia-settings` with a dedicated root Xorg session, automatically detects the fan controllers exposed by the NVIDIA driver, and supports fixed fan speeds, status monitoring, restoring automatic fan control, and restoring GDM when needed.

> Designed primarily for headless / SSH Ubuntu systems where NVIDIA fan control through the normal GDM Xorg session may fail.

## Features

- Designed for headless Ubuntu / SSH servers
- Automatically detects available NVIDIA fan controllers
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
- NVIDIA Xorg configuration with fan control enabled through `Coolbits`

Install the required packages if needed:

```bash
sudo apt update
sudo apt install nvidia-settings xserver-xorg xauth
```

Your NVIDIA device section in `/etc/X11/xorg.conf` should include:

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

## Example

```text
$ gpufan 99

[INFO] 停止 GDM...
[INFO] 建立 Xauthority...
[INFO] 啟動 root Xorg :0...
[OK] Xorg 已啟動
[INFO] 偵測到 2 個可控制風扇：0 1
[INFO] 啟用 NVIDIA 手動風扇控制...
[INFO] Fan 0 -> 99%
[INFO] Fan 1 -> 99%

[OK] 2 個 NVIDIA 風扇已設定為 99%
```

A GPU exposing three independent fan targets may instead show:

```text
[INFO] 偵測到 3 個可控制風扇：0 1 2
```

## How It Works

On some Ubuntu systems, the Xorg session started by GDM can read NVIDIA fan information but fails when writing `GPUTargetFanSpeed`.

This utility works around that behavior by:

1. Stopping GDM
2. Starting a dedicated Xorg session as root
3. Connecting `nvidia-settings` to that X server
4. Enabling `GPUFanControlState`
5. Detecting available `fan:X` targets
6. Setting every detected fan target to the requested speed

## Important Notes

- The script currently controls `gpu:0`.
- The physical number of fans on a graphics card may differ from the number of fan controllers exposed by the NVIDIA driver.
- Running `gpufan <speed>` may stop the current Ubuntu graphical desktop because GDM is stopped before the dedicated root Xorg session starts.
- Use `gpufan gui` to stop the dedicated Xorg session and restore GDM.
- Fan settings may reset after reboot.
- This tool is intended primarily for machines managed through SSH / headless environments.

## Disclaimer

Use this utility at your own risk.

Manual fan control overrides the NVIDIA driver's normal fan policy. Monitor GPU temperature and hardware behavior when changing cooling settings.
