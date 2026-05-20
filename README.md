# Video Move: 全自动视频搬运、去重与发布工作流

[简体中文](./README.md) | [English](./README_en.md)

[![GitHub stars](https://img.shields.io/github/stars/Linuxpizi/video-move?style=social)](https://github.com/Linuxpizi/video-move/stargazers)
[![GitHub forks](https://im.shields.io/github/forks/Linuxpizi/video-move?style=social)](https://github.com/Linuxpizi/video-move/network/members)
[![MIT License](https://imgshields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/Linuxpizi/video-move/pulls)

**ideo Move 是一款强大的、全自动化的内容创作流水线工具，旨在实现从视频源监控下载、深度二次创作到多平台自动发布的无人值守工作流。**

本项目为需要大规模、高效率进行视频内容分发和二次创作的团队及个人设计，通过模块化的设计，将复杂的视频处理流程集成为一套完整的自动化解决方案。


---

## ✨ 核心功能

-   **📥 自动下载 (Auto-Download)**
    -   **实时监控**：7x24小时自动监听指定TikTok博主的发布状态。
    -   **即时下载**：一旦发布新视频，立即无水印下载到本地，为后续处理做准备。

-   **✂️ 智能去重 (Intelligent Deduplication)**
    -   提供一套强大的视频二次创作工具箱，所有功能均可配置和组合，以达到理想的去重效果。
    -   **性能优化**: **🚀 GPU加速**，利用NVIDIA显卡大幅提升处理速度。
    -   **内容增强**: 自动字幕、自定义标题、背景音乐 (BGM)、画中画 (PIP)。
    -   **视频处理**: 静音剪辑、镜像、旋转、裁剪、淡入淡出、画质调整。
    -   **高级特效**: 背景模糊、帧交换、颜色偏移、频域扰乱、纹理噪声等数十种视觉特效。

-   **🚀 AI 驱动上传 (AI-Powered Upload)**
    -   **AI标题生成**：调用阿里云百炼AI大模型，分析视频内容，自动生成爆款标题和标签。
    -   **自动化发布**：模拟浏览器操作，登录视频号后台，自动填写所有信息并发布视频。


## 🚀 快速开始

请严格按照以下步骤进行环境配置和安装。

### 系统要求

1.  **操作系统**: Windows。
2.  **软件/工具**:
    | 软件/工具              | 下载链接                                                     | 备注                                                     |
    | :--------------------- | :----------------------------------------------------------- | :------------------------------------------------------- |
    | **.NET Framework 4.8** | [官方下载](https://dotnet.microsoft.com/en-us/download/dotnet-framework/thank-you/net48-web-installer) | Windows 系统组件。                                       |
    | **Python 3.12+**       | [官方下载](https://www.python.org/ftp/python/3.12.9/python-3.12.9-amd64.exe) | 安装时请务必勾选 `Add Python to PATH`。                  |
    | **Node.js 22.x**       | [官方下载](https://nodejs.org/dist/v22.14.0/node-v22.14.0-x64.msi) | 建议选择 LTS 版本。                                      |
    | **Git**                | [官方下载](https://git-scm.com/downloads/win)                | 版本控制工具。                                           |
    | **FFmpeg**             | [Gyan.dev Builds](https://github.com/GyanD/codexffmpeg/releases/download/7.1.1/ffmpeg-7.1.1-full_build.7z) | **必须**解压并将其 `bin` 目录添加到系统环境变量 `PATH`。 |
    | **Chrome 浏览器**      | [官方下载](https://www.google.com/)                          | 用于自动化上传。                                         |
    | **v2rayN** (可选)      | [GitHub Releases](https://github.com/2dust/v2rayN/releases/download/5.39/v2rayN-Core.zip) | 如果你需要网络代理来访问TikTok。                         |

### 安装与配置

1.  **克隆本仓库：**
    ```bash
    git clone https://github.com/Linuxpizi/video-move.git
    cd video-move
    ```

---

## 🤝 参与贡献

欢迎任何形式的贡献！如果你有新的功能点子、发现了Bug，或者有任何改进建议，请：
-   提交一个 [Issue](https://github.com/Linuxpizi/video-move/issues) 进行讨论。
-   Fork 本仓库提交 [Pull Request](https://github.com/Linuxpizi/video-move/pulls)。

如果个项目对你有帮助，请不吝点亮一颗 ⭐！

## 📜 开源协议

本项目基于 MIT 协议开源。详情请见 [LICENSE](LICENSE) 文件。
