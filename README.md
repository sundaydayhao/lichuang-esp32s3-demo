# 立创实战派 ESP32-S3 开发板例程

本仓库是“立创实战派 ESP32-S3 开发板”的 ESP-IDF 5.4 示例工程集合，按外设和综合应用由浅到深组织。每个编号目录都是一个独立的 ESP-IDF 工程，可以单独进入目录后编译、烧录和调试。

仓库内容覆盖 GPIO 中断、I2C 姿态传感器、TF/SD 卡、音频录放、LCD 显示、摄像头、LVGL 图形界面、Wi-Fi 扫描连接、BLE HID、MP3 播放、语音识别、人脸检测，以及一个整合多应用的手持终端示例。

## 适用环境

- 开发板：立创实战派 ESP32-S3 开发板
- 主控：ESP32-S3
- 框架：ESP-IDF 5.4
- Flash：示例配置以 16 MB Flash 为主
- PSRAM：LCD、摄像头、LVGL、音频、AI 等示例需要启用 PSRAM
- 常用外设：QMI8658 姿态传感器、PCA9557 IO 扩展、ST7789 LCD、FT5x06 触摸、DVP 摄像头、ES7210 麦克风 ADC、ES8311 音频 Codec、TF/SD 卡座

## 仓库结构

| 目录 | 示例名称 | 主要功能 |
| --- | --- | --- |
| `01-boot_key` | BOOT 按键 | 使用 GPIO0 下降沿中断，通过 FreeRTOS 队列把按键事件打印到串口。 |
| `02-attitude` | 姿态传感器 | 通过 I2C 驱动 QMI8658，周期读取加速度解算出的 X/Y/Z 倾角。 |
| `03-micro_sd` | TF/SD 卡 | 使用 SDMMC 1 线模式挂载 FAT 文件系统，创建、写入、读取文件并卸载 SD 卡。 |
| `04-audio_es7210` | ES7210 录音 | 配置 ES7210 和 I2S TDM，从麦克风录制音频并保存为 `/sdcard/RECORD.WAV`。 |
| `05-audio_es8311` | ES8311 播放 | 初始化 ES8311 和 I2S，将内置 `canon.pcm` 音频通过扬声器循环播放。 |
| `06-lcd` | LCD 显示 | 初始化 320x240 ST7789 LCD，绘制内置图片资源。 |
| `07-lcd_camera` | LCD + 摄像头 | 初始化 LCD 和 DVP 摄像头，将实时摄像头画面显示到屏幕。 |
| `08-lcd_lvgl` | LVGL 显示 | 初始化 LCD、触摸和 LVGL，默认运行 `lv_demo_benchmark()`，可切换其他 LVGL demo。 |
| `09-wifi_scan_connect` | Wi-Fi 扫描连接 | 使用 LVGL 做 Wi-Fi 扫描列表、密码输入和连接状态界面。 |
| `10-ble_hid_device` | BLE HID | 将开发板作为 BLE HID 设备广播，通过屏幕按钮发送音量加减等 Consumer Control 指令。 |
| `11-mp3_player` | MP3 播放器 | 挂载 SPIFFS 音乐资源，使用 LVGL 做播放/暂停、上一首/下一首、音量和歌曲列表界面。 |
| `12-speech_recognition` | 语音识别播放器 | 在 MP3 播放器基础上加入 ESP-SR 唤醒和中文命令词识别，支持语音控制播放、暂停、切歌和音量。 |
| `13-human_face_detection` | 人脸检测 | 使用摄像头采集图像，结合 ESP-DL 人脸检测模型在 LCD 上显示 AI 检测结果。 |
| `14-handheld` | 综合手持终端 | 集成开机动画/音乐、姿态监测、音乐播放器、SD 文件浏览、摄像头、Wi-Fi 设置和 BLE 控制器。 |

## 快速开始

先确保已经安装并导出 ESP-IDF 5.4 环境：

```bash
. $IDF_PATH/export.sh
```

进入任意一个示例目录进行构建。比如运行 LCD + 摄像头示例：

```bash
cd 07-lcd_camera
idf.py set-target esp32s3
idf.py build
idf.py -p /dev/ttyUSB0 flash monitor
```

如果你的串口不是 `/dev/ttyUSB0`，请改成实际端口，例如 `/dev/ttyACM0`、`COM3` 等。退出串口监视器可按 `Ctrl+]`。

多数工程已经提供 `sdkconfig.defaults` 和已生成的 `sdkconfig`，一般可以直接 `idf.py build`。如果切换 ESP-IDF 版本或配置混乱，建议在对应示例目录中执行：

```bash
idf.py fullclean
idf.py set-target esp32s3
idf.py build
```

## 工程组织说明

每个示例目录通常包含：

- `CMakeLists.txt`：该示例的 ESP-IDF 工程入口。
- `main/`：应用源码、BSP 驱动封装、图片/字体/音频等资源。
- `main/idf_component.yml`：需要从 Espressif 组件注册表拉取的依赖。
- `sdkconfig.defaults`：推荐默认配置。
- `sdkconfig`：当前工程配置。
- `partitions.csv`：需要 SPIFFS、模型或较大 App 分区的示例会提供自定义分区表。
- `spiffs/`：MP3、提示音等会随固件烧录到 SPIFFS 的文件资源。
- `managed_components/`：ESP-IDF 组件管理器下载的依赖组件。
- `build/`：本地构建产物，可按需清理后重新生成。

根目录本身不是一个可直接 `idf.py build` 的工程；请进入具体编号目录构建。

## 常用硬件连接

以下引脚来自各示例中的 BSP 定义，适用于本开发板资料配套硬件：

| 外设 | 关键引脚/参数 |
| --- | --- |
| BOOT 按键 | `GPIO0`，下降沿中断，内部上拉。 |
| I2C 总线 | SDA `GPIO1`，SCL `GPIO2`，100 kHz。 |
| QMI8658 | I2C 地址 `0x6A`。 |
| PCA9557 | I2C 地址 `0x19`，控制 LCD CS、PA_EN、DVP_PWDN 等扩展 IO。 |
| LCD | ST7789，320x240，SPI3，MOSI `GPIO40`，CLK `GPIO41`，DC `GPIO39`，背光 `GPIO42`。 |
| 触摸 | FT5x06，挂在 I2C 总线上。 |
| SDMMC | 1 线模式，CLK `GPIO47`，CMD `GPIO48`，D0 `GPIO21`。 |
| 摄像头 | XCLK `GPIO5`，PCLK `GPIO7`，VSYNC `GPIO3`，HREF `GPIO46`，D0-D7 分别为 `GPIO16/18/8/17/15/6/4/9`。 |
| I2S 音频 | MCLK `GPIO38`，BCLK `GPIO14`，WS/LRCK `GPIO13`，录音 SDIN `GPIO12`，播放 DOUT `GPIO45`。 |

## 资源与分区

- `11-mp3_player/spiffs/`、`12-speech_recognition/spiffs/`、`14-handheld/spiffs/` 中的 MP3 文件会通过 `spiffs_create_partition_image(... FLASH_IN_PROJECT)` 自动打包并随 `idf.py flash` 烧录。
- `05-audio_es8311/main/canon.pcm` 和 `14-handheld/main/sword.pcm` 使用 `EMBED_FILES` 编译进固件。
- `12-speech_recognition` 需要额外的 `model` 分区供 ESP-SR 语音模型使用。
- `13-human_face_detection` 自带 `components/esp-dl/`，用于人脸检测相关的模型、库和工具。
- 涉及 SPIFFS、大模型或综合应用的示例使用自定义 `partitions.csv`，不要随意改小 App、`storage` 或 `model` 分区。

## 主要依赖组件

部分示例使用 ESP-IDF 组件管理器下载依赖，常见组件包括：

- `espressif/es7210`：ES7210 麦克风 ADC 驱动。
- `espressif/es8311`：ES8311 音频 Codec 驱动。
- `espressif/esp32-camera`：ESP32 摄像头驱动。
- `lvgl/lvgl`：LVGL 图形库。
- `espressif/esp_lvgl_port`：ESP-IDF 与 LVGL 适配层。
- `espressif/esp_lcd_touch_ft5x06`：FT5x06 触摸驱动。
- `espressif/esp_codec_dev`：音频 Codec 抽象层。
- `chmorgan/esp-audio-player`：MP3 等音频播放组件。
- `espressif/esp-sr`：语音唤醒和命令词识别。
- `esp-dl`：深度学习推理与人脸检测相关库。

首次构建相关示例时，ESP-IDF 会根据 `idf_component.yml` 和 `dependencies.lock` 解析依赖；如果网络不可用，需要提前准备好 `managed_components/`。

## 运行提示

- 音频、摄像头、LVGL、AI 示例对 PSRAM 和分区大小比较敏感，建议保留示例自带配置。
- 使用 SD 卡示例前，请插入可用 TF/SD 卡；示例里部分挂载配置允许在挂载失败时格式化，请注意备份卡内数据。
- Wi-Fi 示例会初始化 NVS，连接信息和系统网络状态依赖当前运行时环境。
- BLE HID 示例烧录后可在手机或电脑蓝牙列表中搜索名为 `HID` 的设备。
- 人脸检测和综合终端示例占用资源较多，构建、烧录和启动时间会比基础外设示例更长。
- 如果遇到组件版本或缓存问题，可在对应示例目录删除 `managed_components/` 和 `dependencies.lock` 后重新构建，但建议先备份当前可用版本。

## 学习路线建议

建议按编号顺序阅读和运行：先从 `01-boot_key`、`02-attitude`、`03-micro_sd` 熟悉基础外设，再进入 `06-lcd`、`08-lcd_lvgl` 建立显示和 UI 基础；随后学习 Wi-Fi、BLE、音频、摄像头和 AI 示例。最后阅读 `14-handheld`，它把多个模块组合成一个完整的触摸屏应用，可以作为二次开发的参考框架。
