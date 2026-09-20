# NVIDIA GPU Fan Control for Headless Ubuntu

[繁體中文](README.zh-TW.md)

A lightweight Bash utility for controlling NVIDIA GPU fan speeds on **headless Ubuntu servers** over SSH.

It uses `nvidia-settings` with a dedicated root Xorg session, automatically detects NVIDIA fan controllers, supports **multiple GPUs with `--gpu N`**, and can restore NVIDIA automatic fan control or the Ubuntu GDM graphical login when needed.

> Designed primarily for headless / SSH Ubuntu systems where NVIDIA fan control through the normal GDM Xorg session may fail.

## Features

- Designed for headless Ubuntu / SSH servers
- Supports **multiple NVIDIA GPUs**
- Select a GPU using the same index shown by `nvidia-smi`
- Defaults to **GPU 0** when `--gpu` is omitted
- Automatically maps the selected `nvidia-smi` GPU to the correct `nvidia-settings` GPU target using its PCI Bus ID
- Automatically discovers the fan targets connected to the selected GPU
- Avoids assuming that `fan:0` belongs to `gpu:0`
- Generates a temporary multi-GPU headless Xorg configuration under `/run`
- Does **not** overwrite `/etc/X11/xorg.conf`
- Enables `Coolbits=4` and `AllowEmptyInitialConfiguration` in the temporary Xorg configuration
- Set all detected fans of one GPU to a fixed speed
- Restore automatic fan control for one selected GPU
- View GPU temperature, fan percentage, power draw, target fan speed, and RPM
- Stop the dedicated Xorg session and restore all loaded GPUs to automatic fan control
- Restore Ubuntu GDM with one command

## Tested GPUs

Tested with NVIDIA GPUs across multiple RTX generations:

- NVIDIA GeForce RTX 2080 Ti
- NVIDIA GeForce RTX 3090
- NVIDIA GeForce RTX 4090
- NVIDIA GeForce RTX 5090

The number and numbering of controllable `fan:X` targets depend on the GPU model and NVIDIA driver.

## Supported Ubuntu Versions

- Ubuntu 20.04
- Ubuntu 22.04
- Ubuntu 24.04

## Requirements

- NVIDIA proprietary driver
- `nvidia-smi`
- `nvidia-settings`
- Xorg
- `xauth`
- `mcookie`
- sudo/root access

Install the required packages if needed:

```bash
sudo apt update
sudo apt install nvidia-settings xserver-xorg xauth
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

Verify:

```bash
which gpufan
```

Expected output:

```text
/usr/local/bin/gpufan
```

## Quick Start

List all NVIDIA GPUs:

```bash
gpufan list
```

Example:

```text
========================================
 NVIDIA GPUs
========================================
0, NVIDIA GeForce RTX 4090, 00000000:01:00.0
1, NVIDIA GeForce RTX 3090, 00000000:41:00.0
```

Set GPU 0 to 99% fan speed:

```bash
gpufan 99
```

This is equivalent to:

```bash
gpufan --gpu 0 99
```

Set GPU 1 to 80%:

```bash
gpufan --gpu 1 80
```

Short form:

```bash
gpufan -g 1 80
```

## Usage

### Set fan speed

```bash
gpufan 99
gpufan --gpu 1 80
gpufan --gpu=2 70
```

### Show status

GPU 0:

```bash
gpufan status
```

GPU 1:

```bash
gpufan --gpu 1 status
```

### Restore automatic fan control

GPU 0:

```bash
gpufan auto
```

GPU 1 only:

```bash
gpufan --gpu 1 auto
```

### Stop gpufan Xorg

```bash
gpufan stop
```

This restores automatic fan control for all GPUs currently loaded by the gpufan Xorg session, then stops the dedicated Xorg server.

### Restore Ubuntu GUI / GDM

```bash
gpufan gui
```

## Multi-GPU Design

The GPU index supplied to `--gpu` is the **`nvidia-smi` GPU index**.

It is intentionally not assumed to be the same as the internal `nvidia-settings` target number.

For example:

```bash
gpufan --gpu 1 99
```

works approximately as follows:

1. Read GPU 1 from `nvidia-smi`
2. Read its PCI Bus ID
3. Start a dedicated Xorg session containing all detected NVIDIA GPUs
4. Query the PCI domain, bus, device, and function of each `nvidia-settings` GPU target
5. Match the correct Xorg GPU target by PCI address
6. Query `nvidia-settings -q fans --verbose`
7. Find only the fan targets connected to that GPU
8. Enable manual fan control for that GPU
9. Set only those fan targets to the requested speed

This is important because fan target numbering can be global across the X server. On a multi-GPU machine, `fan:0` is not guaranteed to belong to `gpu:0`.

## Xorg Configuration

gpufan creates its own runtime Xorg configuration:

```text
/run/gpufan-xorg.conf
```

The generated configuration contains all NVIDIA GPUs reported by `nvidia-smi` and automatically includes:

```text
Option "Coolbits" "4"
Option "AllowEmptyInitialConfiguration" "True"
```

The runtime configuration is built from each GPU's PCI Bus ID.

gpufan does **not** overwrite:

```text
/etc/X11/xorg.conf
```

The temporary configuration is removed when `gpufan stop` is used and is naturally cleared after reboot because it is stored under `/run`.

## Important Notes

- `--gpu N` uses the GPU index shown by `nvidia-smi`.
- If `--gpu` is omitted, GPU 0 is used.
- Fan target numbers do not necessarily match GPU numbers.
- The physical number of fans on a graphics card may differ from the number of fan controllers exposed by the NVIDIA driver.
- gpufan refuses to guess fan ownership on a multi-GPU system when NVIDIA does not provide a safe fan-to-GPU mapping.
- Starting gpufan stops GDM so the dedicated root Xorg server can take control.
- Use `gpufan gui` to stop gpufan Xorg and restore GDM.
- Fan settings may reset after reboot.
- This tool is intended primarily for desktop NVIDIA GPUs in headless / SSH-managed Ubuntu systems.

## Disclaimer

Use this utility at your own risk.

Manual fan control overrides the NVIDIA driver's normal fan policy. Monitor GPU temperature and hardware behavior when changing cooling settings.
