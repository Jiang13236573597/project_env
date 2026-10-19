# Project Env
为了解决国内网络环境下 uv 软件、Python 包、ESP32 固件/工具链、以及 NVIDIA Jetson Orin Nano 等开发环境下载卡死的问题，可以封装一个自动化换源与环境配置 Skill。

🚀 核心逻辑：网络优化 Skill 脚本

该 Skill 优先使用中国科学技术大学（USTC）、清华大学（TUNA）和阿里云的镜像源。请在 AI Agent 的初始化阶段或环境配置步骤中执行以下命令/脚本：

```bash
#!/bin/bash
# 自动优化国内开发环境下载速度的配置脚本

echo "=== 开始配置国内加速镜像源 ==="

# 1. 优化 uv (Python 包管理工具) 的全局配置
# uv 兼容 pip 环境变量，同时支持自身的并发下载
export UV_INDEX_URL="https://tsinghua.edu.cn"
export UV_DEFAULT_INDEX="https://tsinghua.edu.cn"
# 写入环境变量以便持久化
echo 'export UV_INDEX_URL="https://tsinghua.edu.cn"' >> ~/.bashrc
echo 'export UV_DEFAULT_INDEX="https://tsinghua.edu.cn"' >> ~/.bashrc

# 2. 优化 ESP32 (ESP-IDF / PlatformIO) 下载
# 针对 ESP-IDF 工具链下载设置乐鑫官方国内镜像源
export IDF_GITHUB_ASSETS="://espressif.com"
echo 'export IDF_GITHUB_ASSETS="://espressif.com"' >> ~/.bashrc

# 3. 优化 NVIDIA Jetson Orin Nano (Ubuntu APT 源)
# 备份并替换为清华/阿里 Ubuntu 镜像源 (以 Ubuntu 20.04/22.04 LTS 为例)
if [ -f /etc/apt/sources.list ]; then
    sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
    # 使用中科大或清华源替换官方源
    sudo sed -i 's/://ubuntu.com' /etc/apt/sources.list
fi

# 4. 优化 GitHub 资源下载 (Agent 经常需要从 GitHub 拉取代码/Release)
# 使用 ghproxy.com 代理（如失效可更换为其他可用镜像）
git config --global url."https://ghfast.top".insteadOf "https://github.com"

# 刷新环境变量
source ~/.bashrc
echo "=== 国内加速源配置完成 ==="
```

🛠️ 针对具体环境的专项优化指南

若在 Agent 执行中仍有卡死，可让 Agent 针对性地执行以下指令：

开发环境	核心优化手段 / 替代命令	预期效果
uv / pip	uv pip install <包名> --index-url https://tsinghua.edu.cn	解决 Python 依赖库下载卡死
ESP32	在执行 install.sh 前，运行 export ESP_IDF_GME_URL="https://gitee.com"	解决工具链与 Gitee 托管仓库下载失败
Orin Nano Super	检查 /etc/apt/sources.list.d/nvidia-l4t-apt-source.list，将其中的 NVIDIA 官方源替换为国内镜像（如有必要）。	解决 JetPack 组件及 CUDA 环境刷新慢


💡 Agent 防卡死防御机制建议

1. 设置显式超时时间：在 Agent 调用 uv 或 apt 命令时，强制加上超时参数。例如：uv pip install --timeout 60 <包名>。
2. 失败自动降级：如果默认源在 30 秒内无响应，自动切换到下一个备用源（如阿里云 https://aliyun.com）。
如果你需要，我可以为你：
• 编写一个Python 版本的 Agent 插件函数，自动检测网络延迟并切换最优源
• 提供特定版本 JetPack (Orin Nano) 的 sources.list 完整文本
• 解决特定 Python 包（如 PyTorch/Torchvision）在嵌入式设备上的换源报错问题
