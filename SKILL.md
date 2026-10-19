---
name: china-dev-mirrors
description: Prefer reliable mainland China mirrors and bounded download workflows when Python or embedded/edge development dependencies are slow or unavailable, including uv, ESP-IDF/ESP32, and NVIDIA Jetson.
---

# Mainland China development mirrors

Use this skill when dependency or tool downloads are blocking development in a mainland China network. The goal is to keep the task moving without silently changing a project's dependency sources or compatibility assumptions.

## Diagnose before switching

- Identify the actual downloader and artifact type: Python index package, ESP-IDF tool/release asset, Git repository, Ubuntu package, NVIDIA JetPack package, or a direct vendor download. A PyPI mirror only helps PyPI traffic.
- Check the target OS, CPU architecture, Python version, and relevant CUDA/JetPack version before selecting binaries. On Jetson, confirm the installed JetPack/L4T release and `aarch64`; a fast index cannot make an incompatible wheel compatible.
- Inspect existing project and user package-source configuration before changing it. Prefer a trusted organization mirror if the environment already documents one. Otherwise use the official TUNA PyPI endpoint below as a practical mainland default.
- Test the selected endpoint with a short bounded request when network inspection is available. Distinguish a slow/unreachable host from a missing package, stale mirror, unsupported wheel, or source-build failure.

## Python packages with uv

For a temporary, command-scoped switch, use uv's default-index environment variable rather than writing persistent global configuration:

```sh
UV_DEFAULT_INDEX=https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple/ \
UV_HTTP_TIMEOUT=60 UV_HTTP_RETRIES=2 \
uv sync
```

Adapt the final command to the project (`uv add`, `uv pip install`, etc.). For a single `uv pip install` operation, `UV_DEFAULT_INDEX` also selects its default index. If the project explicitly defines named indexes or package-specific sources, inspect how they interact before overriding them.

Use one trusted default index at a time. Avoid casually adding public and mirror indexes together: package selection across multiple indexes can be surprising, and adding an untrusted index can create dependency-confusion risk. If the mirror lacks a package or is behind, make one bounded attempt against the project's configured or official source and report the result.

Keep retries finite. If the command produces no progress, give it a task-appropriate wall-clock deadline and stop it when that deadline expires; do not leave an agent waiting indefinitely. Preserve uv's cache and reuse completed downloads. Lower download concurrency only when evidence points to connection saturation or the device is resource constrained.

TUNA publishes current usage guidance at <https://mirror.tuna.tsinghua.edu.cn/help/pypi/>. Its PyPI simple index is `https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple/`.

If the wait is `uv python install` or uv's managed interpreter download, a PyPI index change will not help: these are standalone Python builds. Prefer an already-installed compatible interpreter when appropriate, or use `UV_PYTHON_INSTALL_MIRROR` only when the organization provides a trusted mirror that preserves the Astral release path layout. Do not point this setting at a PyPI mirror or invent a mirror URL. See uv's [Python install mirror setting](https://docs.astral.sh/uv/reference/environment/#uv_python_install_mirror).

## ESP-IDF and ESP32

Separate Python package downloads from toolchain/release assets. When ESP-IDF is downloading GitHub release assets, its documented China download host is:

```sh
export IDF_GITHUB_ASSETS=dl.espressif.cn/github_assets
export PIP_INDEX_URL=https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

Set these only for the ESP-IDF install/build command or its shell session. `IDF_GITHUB_ASSETS` rewrites supported GitHub asset URLs; it does not mirror Git clones or guarantee every upstream asset exists. If the China host lacks an asset, use the official upstream URL for one bounded retry. Do not rewrite Git remotes as a side effect.

See Espressif's [downloadable tools guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/tools/idf-tools.html) and [China setup guidance](https://docs.espressif.com/projects/esp-idf/en/v5.4.4/esp32/get-started/linux-macos-setup.html).

## NVIDIA Jetson Orin Nano / Orin Nano Super

- Use a PyPI mirror for ordinary Python packages only after checking that compatible `aarch64` wheels exist for the device's Python and system libraries. If no wheel exists, explain that a source build may be required and check its build dependencies and resource cost; do not keep retrying the index.
- Keep NVIDIA JetPack, CUDA, TensorRT, and Jetson-specific APT packages on the NVIDIA repository configured for the installed JetPack/L4T release. Do not replace NVIDIA repository URLs, signing keys, or release suites with a generic Ubuntu mirror.
- A generic Ubuntu APT mirror may be used for Ubuntu base packages only when its distribution release and ports architecture match the device. Inspect the current APT sources and signing setup before any source change.
- For SDK Manager or vendor artifacts, use vendor-documented download options or the environment's approved proxy/cache. Do not invent unofficial NVIDIA mirrors.

Use NVIDIA's [Orin Nano JetPack setup guide](https://docs.nvidia.com/jetson/orin-nano-devkit/user-guide/latest/setup_jetpack.html) to identify the supported installation path and release-specific constraints.

## Prevent a stalled task

For every slow install, make the wait bounded at both levels: configure the downloader's supported timeout/retry settings and give the overall command a deadline appropriate to the artifact size. Capture the last useful output, stop a command that exceeds its deadline, then choose one next action based on the failure (alternate trusted endpoint, official source, cached artifact, or compatibility/build diagnosis). Do not repeat the same failed download indefinitely, erase caches to troubleshoot, or change system-wide source configuration without a clear task need.
