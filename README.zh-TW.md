# Ubuntu NVIDIA GPU 風扇控制工具

[English](README.md)

這是一個簡單的 Bash 工具，透過 `nvidia-settings` 在 Ubuntu 上控制 NVIDIA GPU 風扇速度。

腳本會自動偵測 NVIDIA 驅動實際暴露的風扇控制器，必要時自動啟動 root Xorg，並支援固定風扇轉速、恢復 NVIDIA 自動風扇控制以及查看目前 GPU / 風扇狀態。

## 功能

- 自動偵測 NVIDIA 可控制的風扇數量
- 將所有偵測到的 GPU 風扇設定為固定百分比
- 恢復 NVIDIA 原廠自動風扇控制
- 查看 GPU 溫度、風扇百分比、功耗、目標風扇速度與 RPM
- 必要時自動啟動 root Xorg
- 啟動專用 Xorg 前自動停止 GDM
- 可重新恢復 Ubuntu 圖形登入介面
- 適合 Headless / SSH Ubuntu 主機

已測試過的顯示卡包含：

- NVIDIA GeForce RTX 3090
- NVIDIA GeForce RTX 5090

實際可控制的 `fan:X` 數量會依顯示卡與 NVIDIA 驅動而不同。

## 系統需求

- Ubuntu
- NVIDIA 官方驅動
- `nvidia-smi`
- `nvidia-settings`
- Xorg
- `xauth`
- `mcookie`
- NVIDIA Xorg 設定需開啟 `Coolbits` 風扇控制

如缺少相關套件，可安裝：

```bash
sudo apt update
sudo apt install nvidia-settings xserver-xorg xauth
```

`/etc/X11/xorg.conf` 中 NVIDIA Device 區段需要包含：

```text
Option "Coolbits" "4"
```

例如：

```text
Section "Device"
    Identifier "Device0"
    Driver "nvidia"
    Option "Coolbits" "4"
EndSection
```

## 安裝

將 `gpufan` 腳本下載或複製到：

```bash
/usr/local/bin/gpufan
```

加入執行權限：

```bash
sudo chmod +x /usr/local/bin/gpufan
```

確認安裝：

```bash
which gpufan
```

正常應顯示：

```text
/usr/local/bin/gpufan
```

## 使用方式

將所有偵測到的 GPU 風扇固定為 99%：

```bash
gpufan 99
```

設定為 80%：

```bash
gpufan 80
```

查看目前狀態：

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

## 執行範例

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

如果顯示卡暴露三個風扇控制器，可能會看到：

```text
偵測到 3 個可控制風扇：0 1 2
```

## 運作原理

部分 Ubuntu 系統中，透過 GDM 啟動的 Xorg 可以正常讀取 NVIDIA 風扇資訊，但設定 `GPUTargetFanSpeed` 時可能出現錯誤。

這個工具會透過以下方式處理：

1. 停止 GDM
2. 以 root 啟動 Xorg
3. 讓 `nvidia-settings` 連線至該 X Server
4. 開啟 `GPUFanControlState`
5. 自動偵測所有可用的 `fan:X`
6. 將偵測到的風扇設定為指定速度

## 注意事項

- 目前腳本控制的是 `gpu:0`。
- 顯示卡外觀看到的實體風扇數量，不一定等於 NVIDIA 驅動暴露的風扇控制器數量。
- 執行 `gpufan` 時會停止 GDM，因此 Ubuntu 圖形桌面可能會暫時關閉。
- 如需恢復圖形介面，可執行 `gpufan gui`。
- 重新開機後風扇設定可能會被重設。
- 一般工作負載不一定需要長時間維持 100% 風扇速度。

## 免責聲明

請自行承擔使用本工具的風險。

手動風扇控制會覆蓋 NVIDIA 驅動原本的自動風扇策略。修改散熱設定時，請持續監控 GPU 溫度與硬體運作狀況。

## License

可依專案需求選擇授權，例如 MIT License。
