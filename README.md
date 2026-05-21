# ai-workstation-build    
用来存放我WSL2 + CUDA 环境搭建的完整记录，希望能给大家一些参考
# 在 Windows 中使用 WSL2 搭建 AI 开发环境
用来存放我 WSL2 + CUDA 环境搭建的完整记录，希望能给大家一些参考。
## 1. 操作的平台
### 初始状态
- 电脑型号：YOGA Pro 16 IAH10
- CPU：Intel Core Ultra 7 255H
- 内存：LPDDR5X 6400MT/s 32GB
- GPU：NVIDIA RTX 5060 Laptop
- 操作系统：Windows 11 24H2
### 最终状态
- Windows 11 专业版
- WSL 2 + Ubuntu 24.04 LTS
- g++ 13, CMake, GDB, Git
- NVIDIA CUDA Toolkit 12.9 (直通 5060 显卡)
- VS Code + Remote-WSL 连接
## 2. 安装过程
### 2.1 方案调研
查询可能的安装方法，得知有以下几种路径：
- **路径A（在线安装）**：使用 `wsl --install` 命令，适合网络条件良好的情况。
- **路径B（手动启用组件）**：通过 `dism` 命令逐项启用"虚拟机平台"和"Linux子系统"，可绕过部分卡死。
- **路径C（离线安装）**：下载 WSL 内核更新包（.msi）和 Linux 发行版（.wsl/.tar）文件，完全脱离网络限制。
试图用 PowerShell 来安装（输入 `wsl --install`），卡在了 14.6%。
### 2.2 卡在 14.6%：虚拟机冲突
查看返回的错误代码，发现是因为电脑中还有其他的虚拟机冲突占用了 Hyper-V 资源。
### 2.3 升级至 Windows 11 专业版
试图保留电脑的 MuMu 模拟器，查询得知可以共存，但需要特定版本或在重启后切换。尝试启动组件后发现自己的系统无法支持，考虑到以后的 AI 开发需要完整的虚拟化支持，遂决定升级系统。通过模拟器的官方支持将设置导出后便进行准备。
### 2.4 制作安装盘
从学校官网下载系统的官方镜像，使用 Rufus 来制作安装盘。
### 2.5 安装系统与驱动
重装系统后所有驱动都会失效，包括无线网卡和触控板。注意事项：
- 提前准备好键鼠，最好是有线且最简单的。
- 网络可以通过手机开热点，在更多设置中打开"通过 USB 共享网络"来解决。
- 对于显卡等知道具体型号的硬件，如果通过 Lenovo 官网或官方软件下载过慢，可以从显卡制造商的官网下载。
### 2.6 正式安装开发环境
#### 2.6.1 安装 WSL2 + Ubuntu
以管理员身份打开 PowerShell，输入并运行：
​```powershell
wsl --install
​```
等待完成后，重启电脑。系统会自动安装 Ubuntu，并引导设置用户名和密码。
运行完后检查版本号（在 Windows PowerShell 中执行）：
```PS C:\Users\ZKH> wsl --version
WSL 版本: 2.7.3.0
内核版本: 6.6.114.1-1
WSLg 版本: 1.0.73
MSRDC 版本: 1.2.6676
Direct3D 版本: 1.611.1-81528511
DXCore 版本: 10.0.26100.1-240331-1435.ge-release
Windows: 10.0.26200.8457
```
> 如果遇到下载慢的情况，可以用离线包方案。
#### 2.6.2 配置 C++ 开发全家桶
进入安装好的 Ubuntu 终端，逐行执行：
​```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential gdb cmake git
​```
运行完后运行并检查:（在 WSL/Ubuntu 终端中执行）
```zkh@DESKTOP-8D9JR55:~$ g++ --version
g++ (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0
Copyright (C) 2023 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
zkh@DESKTOP-8D9JR55:~$ gcc --version
gcc (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0
Copyright (C) 2023 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
zkh@DESKTOP-8D9JR55:~$ cmake --version
cmake version 3.28.3
CMake suite maintained and supported by Kitware (kitware.com/cmake).
zkh@DESKTOP-8D9JR55:~$ gdb --version
GNU gdb (Ubuntu 15.1-1ubuntu1~24.04.1) 15.1
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
zkh@DESKTOP-8D9JR55:~$ git --version
git version 2.43.0
```
#### 2.6.3 配置 VS Code 远程开发
在 Windows 端下载并安装 VS Code，安装 Remote - WSL 扩展。在 Ubuntu 终端中，进入项目目录，输入 `code .`，VS Code 就会自动连接到 WSL 环境。
#### 2.6.4 安装 CUDA Toolkit​
```# 确保 WSL 内核已更新至最新
wsl --update
# 安装 CUDA Toolkit（WSL专用版，无需装驱动）
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
sudo apt install -y cuda-toolkit-12-9
​```
运行完后运行并检查:（在 WSL/Ubuntu 终端中执行）
注：nvidia-smi 显示的 CUDA Version: 13.2 是 Windows 端驱动支持的最高 CUDA 版本，WSL2 内实际使用的是 nvcc --version 所显示的 12.9。
zkh@DESKTOP-8D9JR55:~$ nvcc --version
```nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Tue_May_27_02:21:03_PDT_2025
Cuda compilation tools, release 12.9, V12.9.86
Build cuda_12.9.r12.9/compiler.36037853_0
zkh@DESKTOP-8D9JR55:~$ nvidia-smi
Thu May 21 22:54:24 2026
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 595.71.01              Driver Version: 596.36         CUDA Version: 13.2     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 5060 ...    On  |   00000000:01:00.0  On |                  N/A |
| N/A   40C    P5             11W /  100W |    1693MiB /   8151MiB |     43%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```
### 2.7 网络与权限问题
在下载安装时还遇到了网络问题。试图更改 DNS 服务器、刷新 DNS 缓存后仍未好转，改用手机热点后可正常安装。另外，在 PowerShell 使用时遇到权限不足的问题，需以管理员身份运行 PowerShell。
## 3. 踩坑记录

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `wsl --install` 卡在 14.6% | MuMu 模拟器占用 Hyper-V | 升级至专业版，或切换 `hypervisorlaunchtype` |
| 下载 Ubuntu 超时 | 国内 DNS 解析微软服务器慢 | 更换手机热点，或使用离线安装包 |
| `g++` 安装后 PowerShell 不识别 | 在 Windows 端执行，编译器在 WSL 内 | 在 Ubuntu 终端中运行 |
| `code .` 命令失效 | VS Code Server 未正确安装 | 通过 VS Code 界面 `WSL: Connect to WSL` 触发安装 |
## 4. 参考资料
- [微软 WSL 官方文档](https://learn.microsoft.com/zh-cn/windows/wsl/)
- [NVIDIA CUDA on WSL 用户指南](https://docs.nvidia.com/cuda/wsl-user-guide/)
