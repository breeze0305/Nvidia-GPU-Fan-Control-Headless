# Headless Ubuntu NVIDIA GPU 風扇控制工具

[English](README.md)

這是一個專為 **Headless Ubuntu / SSH 伺服器**設計的 NVIDIA GPU 風扇控制 Bash 工具。

工具透過 `nvidia-settings` 搭配獨立的 root Xorg 工作階段控制 GPU 風扇，能自動偵測風扇控制器，並且支援使用 **`--gpu N` 指定多張 NVIDIA GPU**。

> 主要解決 Headless Ubuntu 環境中，透過一般 GDM Xorg 可以讀取 NVIDIA 風扇資訊，卻無法成功寫入風扇速度的情況。

## 功能

- 專為 Headless Ubuntu / SSH Server 設計
- 支援**多張 NVIDIA GPU**
- 使用與 `nvidia-smi` 相同的 GPU index 選卡
- 未指定 `--gpu` 時預設控制 **GPU 0**
- 透過 PCI Bus ID，自動將 `nvidia-smi` GPU 對應到正確的 `nvidia-settings` GPU target
- 自動找出真正連接到該 GPU 的 fan targets
- 不會假設 `fan:0` 一定屬於 `gpu:0`
- 自動在 `/run` 建立多 GPU Headless Xorg 暫存設定
- **不會覆寫** `/etc/X11/xorg.conf`
- 暫存 Xorg 設定自動加入 `Coolbits=4` 與 `AllowEmptyInitialConfiguration`
- 可將指定 GPU 的所有可控制風扇固定為指定百分比
- 可單獨恢復指定 GPU 的 NVIDIA 自動風扇控制
- 查看 GPU 溫度、風扇百分比、功耗、目標風扇速度與 RPM
- 可停止專用 Xorg，並將已載入的 GPU 全部恢復自動風扇控制
- 可用單一指令恢復 Ubuntu GDM 圖形介面

## 已測試 GPU

目前已實機測試多個 RTX 世代的 NVIDIA GPU：

- NVIDIA GeForce RTX 2080 Ti
- NVIDIA GeForce RTX 3090
- NVIDIA GeForce RTX 4090
- NVIDIA GeForce RTX 5090

實際可控制的 `fan:X` 數量與編號會依顯示卡型號及 NVIDIA 驅動而不同。

## 支援 Ubuntu 版本

- Ubuntu 20.04
- Ubuntu 22.04
- Ubuntu 24.04

## 系統需求

- NVIDIA 官方驅動
- `nvidia-smi`
- `nvidia-settings`
- Xorg
- `xauth`
- `mcookie`
- sudo/root 權限

如缺少相關套件，可執行：

```bash
sudo apt update
sudo apt install nvidia-settings xserver-xorg xauth
```

## 安裝

Clone 此 Repository：

```bash
git clone https://github.com/breeze0305/Nvidia-GPU-Fan-Control-Headless.git
cd Nvidia-GPU-Fan-Control-Headless
```

安裝成系統指令：

```bash
sudo install -m 0755 gpufan /usr/local/bin/gpufan
```

確認：

```bash
which gpufan
```

正常應顯示：

```text
/usr/local/bin/gpufan
```

## 快速開始

列出所有 NVIDIA GPU：

```bash
gpufan list
```

例如：

```text
========================================
 NVIDIA GPUs
========================================
0, NVIDIA GeForce RTX 4090, 00000000:01:00.0
1, NVIDIA GeForce RTX 3090, 00000000:41:00.0
```

將 GPU 0 設成 99%：

```bash
gpufan 99
```

等同：

```bash
gpufan --gpu 0 99
```

將 GPU 1 設成 80%：

```bash
gpufan --gpu 1 80
```

也可以使用短參數：

```bash
gpufan -g 1 80
```

## 使用方式

### 設定風扇速度

```bash
gpufan 99
gpufan --gpu 1 80
gpufan --gpu=2 70
```

### 查看狀態

GPU 0：

```bash
gpufan status
```

GPU 1：

```bash
gpufan --gpu 1 status
```

### 恢復自動風扇控制

GPU 0：

```bash
gpufan auto
```

只恢復 GPU 1：

```bash
gpufan --gpu 1 auto
```

### 停止 gpufan Xorg

```bash
gpufan stop
```

這會先將目前 gpufan Xorg 載入的所有 GPU 恢復 NVIDIA 自動風扇控制，再停止專用 Xorg。

### 恢復 Ubuntu GUI / GDM

```bash
gpufan gui
```

## 多 GPU 的運作方式

`--gpu` 後面的編號指的是 **`nvidia-smi` 顯示的 GPU index**。

腳本不會假設這個 index 和 `nvidia-settings` 內部的 GPU target 編號相同。

例如：

```bash
gpufan --gpu 1 99
```

大致會執行：

1. 從 `nvidia-smi` 取得 GPU 1
2. 取得該 GPU 的 PCI Bus ID
3. 建立包含所有 NVIDIA GPU 的專用 Xorg
4. 查詢 Xorg 中各 GPU target 的 PCI domain / bus / device / function
5. 透過 PCI 位址找出真正對應的 GPU target
6. 執行 `nvidia-settings -q fans --verbose`
7. 找出真正連接到該 GPU 的 fan targets
8. 只開啟該 GPU 的手動風扇控制
9. 只調整該 GPU 對應的風扇

這一點在多 GPU 主機非常重要，因為 Xorg 中的 fan target 編號可能是全域編號，因此：

```text
fan:0
```

**不保證一定屬於：**

```text
gpu:0
```

## Xorg 設定

gpufan 現在會建立自己的暫存 Xorg 設定：

```text
/run/gpufan-xorg.conf
```

設定檔會包含 `nvidia-smi` 偵測到的所有 NVIDIA GPU，並依照每張卡的 PCI Bus ID 自動生成設定。

每張 GPU 都會加入：

```text
Option "Coolbits" "4"
Option "AllowEmptyInitialConfiguration" "True"
```

gpufan **不會修改或覆寫**：

```text
/etc/X11/xorg.conf
```

執行：

```bash
gpufan stop
```

時會刪除暫存設定；由於檔案位於 `/run`，重新開機後也會自然清除。

## 重要注意事項

- `--gpu N` 使用的是 `nvidia-smi` 中的 GPU index。
- 沒有指定 `--gpu` 時預設為 GPU 0。
- fan target 的編號不一定和 GPU 編號相同。
- 顯示卡外觀看到的實體風扇數量，不一定等於 NVIDIA 驅動暴露的風扇控制器數量。
- 多 GPU 環境下，如果 NVIDIA 沒有提供足夠資訊讓腳本安全判斷風扇歸屬，gpufan 會停止操作，而不是猜測。
- 啟動 gpufan 時會停止 GDM，讓專用 root Xorg 接管。
- 如需恢復圖形介面，可執行 `gpufan gui`。
- 重新開機後風扇設定可能會被重設。
- 本工具主要適合桌上型 NVIDIA GPU 與透過 SSH 管理的 Headless Ubuntu 主機。

## 免責聲明

請自行承擔使用本工具的風險。

手動風扇控制會覆蓋 NVIDIA 驅動原本的自動風扇策略。修改散熱設定時，請持續監控 GPU 溫度與硬體運作狀況。
