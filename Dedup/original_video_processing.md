# Original Video Processing Documentation

This document describes the current implementation in `Dedup/dedup.py` for generating a reworked original-style video. It explains the main processing stages, the key code changes, and the rationale behind each step.

## 1. Overall Purpose

The code is designed to take an input video and produce a higher-quality, improved output that is suitable for original video publishing. The processing pipeline focuses on:

- forcing HD output and upscaling when enabled
- cleaning and enhancing audio
- mixing in custom background music
- removing unused subtitle-generation dependencies
- producing a final MP4 output with the processed video/audio stream

## 2. Video HD Processing

### HD always enabled

`VideoConfig` defaults to:

- `enable_hd = True`
- `hd_target = "4k"`
- `output_width = 0`
- `output_height = 0`

This means the code will automatically select a target resolution for HD output.

### HD resolution calculation

The method `VideoConfig.get_hd_resolution(width, height)` calculates the target output size:

- if `output_width` and `output_height` are manually set, those are used
- otherwise, if `enable_hd` is true, the code maps `hd_target` to one of:
  - `720p`: 1280x720
  - `1080p`: 1920x1080
  - `4k`: 3840x2160
- the source video is scaled to fit within the selected target while preserving aspect ratio

### HD processing in `VideoHandler.process_video`

During video processing, the code reads the source width and height and then computes the HD output size:

- logs `高清输出：从 {width}x{height} 调整到 {self.output_width}x{self.output_height}` when resizing is needed
- then proceeds with frame-by-frame processing using OpenCV and PIL to apply watermark and other effects

### Super-resolution models

The code retains support for super-resolution models:

- OpenCV DNN super-resolution using `EDSR_x4.pb`
- Real-ESRGAN using `RealESRGAN_x4plus.pth`

These are configured under `VideoConfig` and are available if enabled.

## 3. Audio Processing

The audio pipeline has several stages and now includes higher-quality handling for BGM and denoising.

### Silence removal

`AudioHandler.remove_silence(audio_path, config)` does the following:

1. If `config.enable_silence_check` is false, it returns the original audio unchanged
2. Otherwise, it loads audio via `pydub.AudioSegment`
3. It splits the audio on silence using:
   - `min_silence_len = config.silent_duration`
   - `silence_thresh = config.silence_threshold`
4. It stitches non-silent chunks back together with a short crossfade to avoid clicks
5. It can also re-add a small amount of retained silence between segments controlled by `config.silence_retention_ratio`
6. The output is exported as a high-quality WAV file

This stage helps make the final audio tighter and more suitable for polished publishing.

### Background music mixing

`AudioHandler.mix_bgm(original_audio_path, bgm_path, background_music_volume)` implements the current BGM pipeline:

1. Verify the BGM path and volume
2. Convert BGM to high-quality WAV using `AudioHandler.convert_to_high_quality_wav()` if necessary
   - output format: WAV, PCM S16LE, 44.1 kHz, stereo
3. Load both the source audio and BGM as WAV inputs into `ffmpeg`
4. Resample both to 44.1 kHz
5. Apply `volume` filter to the BGM track with `background_music_volume`
6. Mix with `amix` using weights `1 {background_music_volume}` and `duration='shortest'`
7. Write the mixed result to a WAV file

This ensures background music is blended cleanly with the source audio.

### Denoising the mixed audio

A new helper method was added:

- `AudioHandler.denoise_audio(input_wav_path)`

It uses FFmpeg filters to reduce noise after mixing:

- queries available FFmpeg filters
- prefers `arnndn` if available
- falls back to `afftdn` if `arnndn` fails
- if neither filter exists or both fail, it returns the original mixed audio

This step is intended to reduce the remaining `滋啦滋啦` noise from the processed track.

### Quality conversion

`AudioHandler.convert_to_high_quality_wav(input_path)` converts any input audio to a stable WAV source before mixing:

- output format: WAV
- codec: `pcm_s16le`
- channels: stereo
- sample rate: 44100 Hz

This standardization avoids mismatched sample rates or channel layouts during mixing.

## 4. Subtitle handling

The code was simplified for clarity and dependency reduction:

- Whisper-based subtitle generation has been removed
- `opencc` import has been removed
- subtitles are now loaded only from the configured file path in `VideoConfig.subtitles_file`

This removes extra dependency code and keeps the output pipeline focused on video/audio processing.

## 5. Output and final muxing

In `VideoHandler.process_video`:

- the code extracts the original audio stream if needed
- applies silence removal and/or BGM mixing if configured
- final video encoding is handled via ffmpeg with the processed audio path
- HD resizing remains part of the flow before final output

## 6. Practical impact for original-video publishing

These changes contribute to a polished, original-style output by:

- forcing high-resolution HD video output
- preserving or upscaling source video quality
- cleaning audio and removing low-level noise
- mixing higher-quality BGM in a controlled way
- removing unused subtitle generation code and dependencies

## 7. Current configuration defaults

Important defaults in `VideoConfig`:

- `enable_hd = True`
- `hd_target = "4k"`
- `enable_opencv_sr = True`
- `enable_realesrgan = True`
- `enable_silence_check = False`
- `include_background_music = True`
- `background_music_volume = 0.1`
- `include_subtitles = False`

If you want, I can also add a short changelog section at the top of this document describing the exact code modifications made during the current round of work. 
