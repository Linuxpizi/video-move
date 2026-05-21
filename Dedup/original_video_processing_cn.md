# 原创视频处理文档

本文档描述了当前 `Dedup/dedup.py` 中用于生成改造后原创风格视频的实现。它解释了主要处理阶段、关键代码改动，以及每个步骤背后的设计目的。

## 1. 总体目的

该代码旨在将输入视频处理为更高质量、适合原创视频发布的输出。处理流程重点包括：

- 强制开启高清输出并在启用时进行放大
- 清理并增强音频
- 混合自定义背景音乐
- 删除未使用的字幕生成依赖
- 生成最终带处理后音视频流的 MP4 输出

## 2. 视频高清处理

### 一定开启高清

`VideoConfig` 的默认配置为：

- `enable_hd = True`
- `hd_target = "4k"`
- `output_width = 0`
- `output_height = 0`

这意味着代码会自动选择目标分辨率进行高清输出。

### 高清分辨率计算

方法 `VideoConfig.get_hd_resolution(width, height)` 会计算目标输出大小：

- 如果手动设置了 `output_width` 和 `output_height`，则使用手动设置值
- 否则，如果 `enable_hd` 为真，代码会根据 `hd_target` 映射到：
  - `720p`：1280x720
  - `1080p`：1920x1080
  - `4k`：3840x2160
- 源视频会按比例缩放到所选目标内，并保持宽高比不变

### 在 `VideoHandler.process_video` 中的高清处理

视频处理过程中，代码读取源视频宽高，然后计算高清输出尺寸：

- 如果需要缩放，会记录日志 `高清输出：从 {width}x{height} 调整到 {self.output_width}x{self.output_height}`
- 随后继续逐帧处理，使用 OpenCV 和 PIL 应用水印及其他效果

### 超分模型支持

代码保留了超分辨率模型支持：

- 使用 `EDSR_x4.pb` 的 OpenCV DNN 超分
- 使用 `RealESRGAN_x4plus.pth` 的 Real-ESRGAN

这些模型由 `VideoConfig` 配置控制，仅在启用时有效。

## 3. 音频处理

音频流水线包含多个阶段，并增加了对 BGM 更高质量处理和降噪的支持。

### 静音消除

`AudioHandler.remove_silence(audio_path, config)` 的处理流程：

1. 如果 `config.enable_silence_check` 为 false，则直接返回原始音频
2. 否则通过 `pydub.AudioSegment` 加载音频
3. 使用如下参数按静音切割音频：
   - `min_silence_len = config.silent_duration`
   - `silence_thresh = config.silence_threshold`
4. 将非静音片段重拼接，加入短淡入淡出以避免卡顿和点击声
5. 可以根据 `config.silence_retention_ratio` 在片段间重新添加少量保留静音
6. 最后导出为高质量 WAV 文件

此阶段有助于让最终音频更紧凑、更加适合成品发布。

### 背景音乐混合

`AudioHandler.mix_bgm(original_audio_path, bgm_path, background_music_volume)` 实现当前的 BGM 混音流程：

1. 校验 BGM 路径和音量
2. 在必要时通过 `AudioHandler.convert_to_high_quality_wav()` 将 BGM 转换为高质量 WAV
   - 输出格式：WAV，PCM S16LE，44.1 kHz，立体声
3. 将原始音频和 BGM 都作为 WAV 输入交给 `ffmpeg`
4. 将两路音频都重采样到 44.1 kHz
5. 对 BGM 应用 `volume` 滤镜，调整为 `background_music_volume`
6. 使用 `amix` 混音，权重为 `1 {background_music_volume}`，`duration='shortest'`
7. 将混合结果写入 WAV 文件

这保证了背景音乐与原始音频干净融合。

### 混合后音频降噪

新增了辅助方法：

- `AudioHandler.denoise_audio(input_wav_path)`

它使用 FFmpeg 滤镜在混音后降低噪声：

- 查询可用的 FFmpeg 滤镜
- 优先使用 `arnndn`
- 如果 `arnndn` 失败，则退回使用 `afftdn`
- 如果都不可用或都失败，则返回原始混合音频

此步骤用于减少处理后音轨中剩余的“滋啦滋啦”噪声。

### 高质量格式转换

`AudioHandler.convert_to_high_quality_wav(input_path)` 将任意输入音频转换为混音前的稳定 WAV 源：

- 输出格式：WAV
- 编码：`pcm_s16le`
- 声道：立体声
- 采样率：44100 Hz

该标准化避免了混音过程中采样率或声道布局不匹配的问题。

## 4. 字幕处理

代码为清晰性和依赖精简做了简化：

- 已移除基于 Whisper 的字幕生成
- 删除了 `opencc` 引入
- 仅从 `VideoConfig.subtitles_file` 指定的文件加载字幕

这减少了额外的依赖代码，并让输出流程更专注于视频/音频处理。

## 5. 输出和最终复用

在 `VideoHandler.process_video` 中：

- 根据需要提取原始音频流
- 如配置，则执行静音去除和/或 BGM 混合
- 最终使用 ffmpeg 将处理后音频与视频一起输出
- 高清缩放仍作为输出流程的一部分

## 6. 对原创视频发布的实际影响

这些改动能让成品更符合原创视频发布要求：

- 强制高分辨率 HD 视频输出
- 保留或放大源视频质量
- 清理音频并去掉低频噪声
- 以可控方式混入更高质量 BGM
- 删除无关字幕生成代码和依赖

## 7. 当前配置默认值

`VideoConfig` 中的重要默认值：

- `enable_hd = True`
- `hd_target = "4k"`
- `enable_opencv_sr = True`
- `enable_realesrgan = True`
- `enable_silence_check = False`
- `include_background_music = True`
- `background_music_volume = 0.1`
- `include_subtitles = False`

如果你需要，我也可以继续补一个“本次改动摘要”，列出此次工作中具体修改的文件和位置。