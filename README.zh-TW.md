# Headless Ubuntu NVIDIA GPU 風扇控制工具

[English](README.md)

這是一個專為 **Headless Ubuntu / SSH 伺服器**設計的 NVIDIA GPU 風扇控制 Bash 工具。

工具透過 `nvidia-settings` 搭配獨立的 root Xorg 工作階段控制 GPU 風扇，能自動偵測 NVIDIA 驅動實際暴露的風扇控制器，並支援固定風扇轉速、查看狀態、恢復 NVIDIA 自動風扇控制，以及在需要時重新啟動 GDM 圖形介面。

> 主要解決 Headless Ubuntu 環境中，透過一般 GDM Xorg 可以讀取風扇資訊，卻無法成功寫入風扇速度的情況。

## 功能

- 專為 Headless Ubuntu / SSH Server 設計
- 自動偵測 NVIDIA 可控制的風扇數量
- 當 `/etc/X11/xorg.conf` 不存在時，自動建立最小化 Headless Xorg 設定
- 自動從 `nvidia-smi` 偵測 `gpu:0` 的 PCI Bus ID
- 自動在新建立的設定中加入 `Coolbits=4` 與 `AllowEmptyInitialConfiguration`
- 不會覆寫既有的 `/etc/X11/xorg.conf`
- 將所有偵測到的 GPU 風扇設定為固定百分比
- 恢復 NVIDIA 原廠自動風扇控制
- 查看 GPU 溫度、風扇百分比、功耗、目標風扇速度與 RPM
- 必要時自動建立獨立的 root Xorg
- 啟動 root Xorg 前自動停止 GDM
- 可透過單一指令恢復 Ubuntu 圖形登入介面
- 不需要手動指定 `fan:0`、`fan:1`、`fan:2` 的數量

目前已實機測試多個 RTX 世代的 NVIDIA GPU：

- NVIDIA GeForce RTX 2080 Ti
- NVIDIA GeForce RTX 3090
- NVIDIA GeForce RTX 4090
- NVIDIA GeForce RTX 5090

實際可控制的 `fan:X` 數量會依顯示卡型號與 NVIDIA 驅動而不同。

## 系統需求

- Ubuntu
- NVIDIA 官方驅動
- `nvidia-smi`
- `nvidia-settings`
- Xorg
- `xauth`
- `mcookie`

如缺少相關套件，可執行：

```bash
sudo apt update
sudo apt install nvidia-settings xserver-xorg xauth
```

### Xorg 設定

在新的 Headless Ubuntu 主機上，**不需要手動建立** `/etc/X11/xorg.conf`。

如果檔案不存在，`gpufan` 會自動：

1. 從 `nvidia-smi` 取得 `gpu:0` 的 PCI Bus ID
2. 自動轉換成 Xorg 使用的 `BusID` 格式
3. 建立最小化的 Headless Xorg 設定
4. 自動加入：

```text
Option "Coolbits" "4"
Option "AllowEmptyInitialConfiguration" "True"
```

如果 `/etc/X11/xorg.conf` 已經存在，`gpufan` **不會自動覆寫**。若要進行手動風扇控制，既有 NVIDIA Device 設定應啟用 `Coolbits=4`。

## 安裝

Clone 此 Repository：

```bash
git clone https://github.com/breeze0305/Nvidia-GPU-Fan-Control-Headless.git
cd Nvidia-GPU-Fan-Control-Headless
```

將 `gpufan` 安裝成系統指令：

```bash
sudo install -m 0755 gpufan /usr/local/bin/gpufan
```

確認安裝：

```bash
which gpufan
```

正常應顯示：

```text
/usr/local/bin/gpufan
```

接著就可以直接：

```bash
gpufan 99
```

首次執行時，如果系統沒有 Xorg 設定檔，腳本會自動建立，不需要額外設定。

## 使用方式

將所有偵測到的 GPU 風扇固定為 99%：

```bash
gpufan 99
```

設定為 80%：

```bash
gpufan 80
```

查看目前 GPU 與風扇狀態：

```bash
gpufan status
```

恢復 NVIDIA 自動風扇控制：

```bash
gpufan auto
```

恢復自動風扇控制並關閉 root Xorg：

```bash
gpufan stop
```

恢復 Ubuntu GUI / GDM：

```bash
gpufan gui
```

## 首次執行範例

```text
$ gpufan 99

[INFO] 找不到 /etc/X11/xorg.conf
[INFO] 自動偵測 GPU 0 PCI Bus 並建立 Headless Xorg 設定...
[OK] 已建立 /etc/X11/xorg.conf
[INFO] GPU：NVIDIA GeForce RTX 3090
[INFO] PCI Bus：00000000:01:00.0 -> PCI:1:0:0
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

如果顯示卡暴露三個獨立風扇控制器，可能會看到：

```text
[INFO] 偵測到 3 個可控制風扇：0 1 2
```

## 運作原理

部分 Ubuntu 系統中，由 GDM 啟動的 Xorg 可以正常讀取 NVIDIA 風扇資訊，但設定 `GPUTargetFanSpeed` 時可能失敗。

此工具會透過以下方式處理：

1. 如果缺少 Xorg 設定，自動建立最小化 Headless 設定
2. 自動偵測 `gpu:0` 的 PCI Bus ID
3. 停止 GDM
4. 以 root 啟動獨立的 Xorg
5. 讓 `nvidia-settings` 連線到該 X Server
6. 開啟 `GPUFanControlState`
7. 自動偵測所有可用的 `fan:X`
8. 將所有偵測到的風扇設定為指定速度

## 重要注意事項

- 目前腳本控制的是 `gpu:0`。
- 自動建立的 Xorg 設定同樣以 `gpu:0` 為目標。
- 如果系統原本已有 `/etc/X11/xorg.conf`，腳本不會自動覆寫。
- 顯示卡外觀看到的實體風扇數量，不一定等於 NVIDIA 驅動暴露的風扇控制器數量。
- 執行 `gpufan <速度>` 時會停止 GDM，因此正在使用中的 Ubuntu 圖形桌面可能會暫時關閉。
- 如需恢復圖形介面，可執行 `gpufan gui`。
- 重新開機後風扇設定可能會被重設。
- 本工具主要適合透過 SSH 管理的 Headless Ubuntu 主機。

## 免責聲明

請自行承擔使用本工具的風險。

手動風扇控制會覆蓋 NVIDIA 驅動原本的自動風扇策略。修改散熱設定時，請持續監控 GPU 溫度與硬體運作狀況。
