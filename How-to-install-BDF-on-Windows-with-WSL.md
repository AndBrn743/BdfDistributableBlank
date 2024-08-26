# How to install BDF on Windows with WSL

## Summary
The Windows Subsystem for Linux (WSL) enables users to install the Beijing Density Functional (BDF) program on a Windows machine. This document provides instructions on how to create and deploy BDF distributable images.

## Prerequisites
The present document assumes that your machine
* Have WSL enabled, and
* Have WSL updated to a version that supports WSL 2 (Optional, but highly recommended.)

> * Please reference to [Install WSL | Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/install) for WSL installation instruction.
> * Virtualization technology must be enabled in BIOS or UEFI to install WSL. To check if virtualization technology was enabled, see Windows Task Manager > Performance > CPU > Virtualization.
> * We recommend WSL 2 over WSL 1. If you want to use WSL 1, there are a few extra steps you'll need to take. These aren't included in the current version of the document, but you can find out more in the [ArchWSL Documentation](https://wsldl-pg.github.io/ArchW-docs/Known-issues/). For the comparison between WSL 1 and WSL 2, see [Comparing WSL Versions | Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/compare-versions). 

## Instructions for Distributable Creation
### 1. Import BDF Distributable Blank as a new WSL distro

Execute the following command in Windows PowerShell
 ``` PowerShell
 wsl --import BdfServer <InstallLocation> BdfDistributableBlank.vhdx --version 2 --vhd
 ```
or
 ``` PowerShell
 wsl --import BdfServer <InstallLocation> BdfDistributableBlank.tar.gz --version 2
 ```

To make the new distro has been successfully imported one may run
`wsl -l -v`.
One should expect info like the following to be printed (notice the very last line)
 ``` console
 PS C:\Users\UserName> wsl -l -v 
   NAME                   STATE           VERSION
 * Arch                   Running         2
   CentOS8                Stopped         2
   openSUSE               Stopped         2
   BdfServer              Stopped         2
 ```

> * BdfDistributableBlank can be obtained from [here](https://github.com/AndBrn743/BdfDistributableBlank/releases).
> * Please replace `<InstallLocation>` with the desired installation location.
> * Instead of `BdfServer` you can any name you like.

### 2. Download, Compile, and Install BDF

#### 2.1. Enter the BdfServer distro

In Windows PowerShell, run the following command
``` PowerShell
wsl -d BdfServer
```
and you should enter a Linux Bash shell instance of the `BdfServer` distro

#### 2.2. Perform full system upgrade (Optional)

In Linux Bash shell, execute
``` bash
pacman -Syyu
```
and follow the instructions shown on the screen
> * For more information about `pacman`, please reference to [ArchLinux wiki](https://wiki.archlinux.org/title/Pacman)

#### 2.3. Download or copy the BDF source files to BdfServer

In Linux Bash shell, run following command
``` bash
git clone user_name@server:/path/to/bdf-pkg
```
e.g.,
``` bash
git clone user_name@182.92.69.169:/opt/gitroot/bdf-pkg 
```
or
``` bash
cp /mnt/windows/path/to/bdf-pkg.tar.gz .
```
e.g.,
``` bash
cp /mnt/d/data/bdf-pkg.tar.gz . 
```
if `bdf-pkg.tar.gz` can be found in Windows directory of `D:\data\bdf-pkg.tar.gz`

#### 2.4. Compile and install BDF

BDF Distributable Blank comes with `compile_and_install_bdf` script. By executing the `compile_and_install_bdf` script in Linux Bash shell, BDF will be automatically compiled with the bundled linear algebra library, and installed BDF in a suitable directory. If the BDF source folder is named `bdf-pkg` and located under present working directory, or the home directory of current user, simply run the following command:
``` bash
compile_and_install_bdf
```
One may also pass a custom path to the BDF source folder to `compile_and_install_bdf` as its first argument, e.g.
``` bash
compile_and_install_bdf /custom/path/to/bdf/source/files
```
Custom build flags can be passed to `compile_and_install_bdf` directly, e.g.
``` bash
compile_and_install_bdf -DENABLE_LICENSE=YES -DONLY_BDFPRO=YES
```

> * Please execute `compile_and_install_bdf` script with the root account
> * `compile_and_install_bdf` script does not support user defined install-directory. Install BDF to a non-default installation path will cause the bundled BDF run script `bdf` to break
> * After compilation and installation was completed, `compile_and_install_bdf` ask one if the deletion of the build and source directories, as well as the pacman caches. Choose `Y` for all of them unless there is reason not to do so 

#### 2.5. Cleanup
* Delete the BDF build folder if did do so already
* Delete the BDF source folder if did do so already
* Clear pacman caches if did do so already (execute `pacman -Scc` and choose `Y` for all)
* Delete IDE and code editor (e.g., Visual Studio, Visual Studio Code, CLion, Rider, and PyCharm) caches if one has connected them to `BdfServer`
* Delete other temporary files and folders

### 3. Generate the Distributable

#### 3.1. Close all Windows program connected to `BdfServer`, including but not limited to `conhost.exe`, Windows Terminal, Windows File Explorer, Visual Studio, and Visual Studio Code

#### 3.2. Shut WSL Down
    
Execute the following command in Windows PowerShell

``` PowerShell
wsl --shutdown
```

> * Run `wsl -t BdfServer` won't work by our experience, your milage may very
> * `BdfServer` can restarted automatically there is a Windows program that was connected to it was still running, which will interfere with the next step

#### 3.3. Export BDF Distributable
    
##### 3.3.1. To export the Distributable as a tarball, run the following command in Windows PowerShell
``` PowerShell
wsl --export BdfServer BdfServer.tar.gz
```
The output file `BdfServer.tar.gz` is the distributable
   
##### 3.3.2. To export the Distributable as a virtual disk, run the following command in Windows PowerShell
``` PowerShell
wsl --export BdfServer BdfServer.vhdx --vhd
```
The output file `BdfServer.vhdx` is the distributable

## Instructions for Distributable Deployment

To deploy the Distributable on user machines, run the following command in Windows PowerShell
``` PowerShell
wsl --import BdfServer <InstallLocation> BdfServer.tar.gz --version 2 
```
or
``` PowerShell
wsl --import BdfServer <InstallLocation> BdfServer.vhdx --version 2 --hvd
```

> * Please replace `<InstallLocation>` with the desired installation location.
> * Instead of `BdfServer` you can any name you like.
> * One may BDF as norm as this point. However, it's highly recommend to add a non-root account and set it as a default login account. The instructions can be found [here](https://wsldl-pg.github.io/ArchW-docs/How-to-Setup/#setup-after-install). 

## Frequently Used Commands
* To run `BdfServer` commands through PowerShell, e.g., `htop`, use the following command
``` PowerShell
wsl -d BdfServer htop
```

* To run `BdfServer` commands through PowerShell, e.g., `ls`, in current Windows directory, use the following command
``` PowerShell
wsl -d BdfServer ls
```

* To run `BdfServer` commands through PowerShell, e.g., `ls`, in a given Linux directory (e.g., `~/tasks`) use the following command
``` PowerShell
wsl -d BdfServer --cd ~/tasks ls
```

* To copy or move a file from the current Windows directory to a `BdfServer` directory (e.g., `~/tasks`) use the following command in Windows PowerShell
``` PowerShell
wsl -d BdfServer cp MyWindowsFile.txt ~/tasks/
```
or
``` PowerShell
wsl -d BdfServer mv MyWindowsFile.txt ~/tasks/
```

* To copy or move a file from a `BdfServer` directory, e.g., ~/tasks, to current Windows directory to use the following command in Windows PowerShell
``` PowerShell
wsl -d BdfServer cp ~/tasks/MyLinuxFile.txt .
```
or
``` PowerShell
wsl -d BdfServer mv ~/tasks/MyLinuxFile.txt .
```

* To run a BDF calculation task in a Windows directory through PowerShell, use the following command
``` PowerShell
wsl -d BdfServer bdf BdfCalculationInputFile.inp
```

* To open a `BdfServer` directory, e.g., ~/tasks, with Windows File Explorer execute the following command in Windows PowerShell
``` PowerShell
wsl -d BdfServer --cd ~/tasks/ explorer.exe .
```

> * Depending on WSL version, executing BDF directory in Windows directory may or may not be a good idea. For WSL1, it's perfectly fine to do so. In WSL2, however, the IO operation between Windows directories and WSL directories is very slow, which made this approach undesirable. Therefore, we'd recommend copy the BDF input file inside a BdfServer directory and perform the BDF calculation inside the BdfServer directory for WSL2. The PowerShell commend for copy and move files between Windows and WSL directories is listed above.

## Notes
* BDF Distributable Blank (BDB) is a distributable WSL mirror like the BDF Distributable (BD). The difference is that BDB does not have BDF installed, what it has installed are the dependent libraries and compilation toolchains for compile and install BDF. Therefore, BDB is much smaller than BD and can be reused for years (update it yearly is still recommended), and BD should be replaced by a newer version every time BDF updates.
* Do *not* set password for `root` account. Do *not* add any additional account.
* Since the Distributable contains not only compiled BDF program but also dependence, its file may be as large as 10 GB. Distributor should choose the distribution media accordingly.



# 如何使用 WSL 在 Windows 上安装 BDF 

## 总结
通过适用于 Linux 的 Windows 子系统 (WSL), BDF (北京密度泛函程序) 可被安装至运行 Windows 操作系统的计算机上. 本文档记录了可分发的 BDF 镜像的构造及部署步骤. 

## 前提条件
本文档假设您的计算机已
* 启用 WSL, 并已
* 更新 WSL 至一支持 WSL 2 的版本 (可选, 但强烈建议.)

> * WSL 的安装步骤可参见 [Install WSL | Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/install).
> * 若要安装 WSL 虚拟化技术必须被 (从 BIOS 或 UEFI 里) 启用. 要检查您的计算机是否已经启用虚拟化技术可查看 Windows 任务管理器 > 性能 > CPU > 虚拟化.
> * 我们推荐使用 WSL 2. 如果您想使用 WSL 1, 则需要执行一些未包含在当前版本的文档中的额外步骤. 您可以在 [ArchWSL Documentation](https://wsldl-pg.github.io/ArchW-docs/Known-issues/) 中找到更多信息. WSL 1 同 WSL 2 的比较见 [Comparing WSL Versions | Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/compare-versions).

## 创建 BDF 可分发镜像的步骤
### 1. 将 BDF Distributable Blank 注册为一新的 WSL distro

在 Windows PowerShell 中执行下面的命令
 ``` PowerShell
 wsl --import BdfServer <InstallLocation> BdfDistributableBlank.vhdx --version 2 --vhd
 ```
或
 ``` PowerShell
 wsl --import BdfServer <InstallLocation> BdfDistributableBlank.tar.gz --version 2
 ```

要确认新的distro 以被成功注册可以运行下面的命令
`wsl -l -v`.
您应期待类似下面的内容被打印至屏幕 (注意最后一行)
 ``` console
 PS C:\Users\UserName> wsl -l -v 
   NAME                   STATE           VERSION
 * Arch                   Running         2
   CentOS8                Stopped         2
   openSUSE               Stopped         2
   BdfServer              Stopped         2
 ```

> * BdfDistributableBlank 可从[此处](https://github.com/AndBrn743/BdfDistributableBlank/releases)获取.
> * 请将 `<InstallLocation>` 替换为真正的安装路径.
> * 您不一定需要将其命名为 `BdfServer`.

### 2. 下载, 编译, 并安装 BDF

#### 2.1. 进入 BdfServer distro

在 Windows PowerShell 运行下面的命令
``` PowerShell
wsl -d BdfServer
```
现在您应进入BdfServer distro 的一个Bash shell 实例

#### 2.2. 进行系统更新 (可选)

在 Linux Bash shell 中执行
``` bash
pacman -Syyu
```
并按照屏幕上的指示操作

> * 您可在 [ArchLinux wiki](https://wiki.archlinux.org/title/Pacman) 上阅读关于 `pacman` 的更多信息

#### 2.3. 下载或复制 BDF 源文件至 BdfServer

在 Linux 的Bash shell 运行下面的命令
``` bash
git clone user_name@server:/path/to/bdf-pkg
```
如,
``` bash
git clone user_name@182.92.69.169:/opt/gitroot/bdf-pkg 
```
或
``` bash
cp /mnt/windows/path/to/bdf-pkg.tar.gz .
```
如,
``` bash
cp /mnt/d/data/bdf-pkg.tar.gz . 
```
如果 `bdf-pkg.tar.gz` 可以在 Windows 下的路径 `D:\data\bdf-pkg.tar.gz` 中被找到

#### 2.4. 编译并安装 BDF

BDF Distributable Blank (BDF) 自带一 `compile_and_install_bdf` 脚本. 在 Linux Bash shell 中运行 `compile_and_install_bdf` 便可自动使用 BDB 捆绑的线性代数库编译 BDF 并将 BDF 安装至一合适的路径下. 若 BDF 的源代码所在文件夹为 `bdf-pkg` 且位于当前工作目录或当前用户的 home 路径下, 则您仅需运行下面的命令:
``` bash
compile_and_install_bdf
```
您也可以将 BDF 源代码所在的自定义路径作为 `compile_and_install_bdf` 的第一个命令行参数传给它, 如
``` bash
compile_and_install_bdf /custom/path/to/bdf/source/files
```
额外的 build flags 也可被直接传给 `compile_and_install_bdf`, 如
``` bash
compile_and_install_bdf -DENABLE_LICENSE=YES -DONLY_BDFPRO=YES
```

> * 请用管理员权限运行 `compile_and_install_bdf` 脚本
> * `compile_and_install_bdf` 脚本不支持自定义安装路径. 将 BDF 安装至非默认路径将导致捆绑的 BDF 运行脚本 `bdf` 不可用
> * 在编译和安装完成后 `compile_and_install_bdf` 将询问您是否希望删除编译路径, 源代码所在文件夹, 及 pacman 缓存. 除非您有特殊原因, 否则请全部选择 `Y`

#### 2.5. 清理
* 删除 BDF 编译文件夹, 若尚未删除
* 删除 BDF 源代码文件夹, 若尚未删除
* 删除 pacman 的全部缓存文件, 若尚未删除 (执行 `pacman -Scc` 并对所有选择都选择 `Y`)
* 删除 IDE 及代码编辑器 (如, Visual Studio, Visual Studio Code, CLion, Rider, 及 PyCharm) 的缓存文件及文件夹, 若您曾将它们连接至 `BdfServer`
* 删除其它零时文件及文件夹

### 3. 产生可分发镜像

#### 3.1. 关闭所有连接至 `BdfServer` 了的所有 Windows 程序, 其中包括但不限于 `conhost.exe`, Windows Terminal, Windows File Explorer, Visual Studio, 及 Visual Studio Code

#### 3.2. 将 WSL 置于关机状态

在 Windows PowerShell 运行下面的命令

``` PowerShell
wsl --shutdown
```

> * 仅运行 `wsl -t BdfServer` 据我们的经验无法达到相同的效果, 您的结果可能与我们不同
> * `BdfServer` 可以被连接至其的 Windows 程序重新启动, 若如此将影响下一步

#### 3.3. 导出BDF 可发布镜像

##### 3.3.1. 如需 tar 格式的可分发镜像请在 Windows PowerShell 中运行下面的命令
``` PowerShell
wsl --export BdfServer BdfServer.tar.gz
```
输出文件 `BdfServer.tar.gz` 就是可分发镜像

##### 3.3.2. 如需 virtual disk (vhdx) 格式的可分发镜像请在 Windows PowerShell 中运行下面的命令
``` PowerShell
wsl --export BdfServer BdfServer.vhdx --vhd
```
输出文件 `BdfServer.vhdx` 就是可分发镜像

## 部署可分发镜像的步骤

在将可分发镜像部署在用户计算机上时需运行下面的 Windows PowerShell 命令
``` PowerShell
wsl --import BdfServer <InstallLocation> BdfServer.tar.gz --version 2 
```
或
``` PowerShell
wsl --import BdfServer <InstallLocation> BdfServer.vhdx --version 2 --hvd
```

> * 请将 `<InstallLocation>` 替换为真正的安装路径.
> * 您不一定需要将其命名为 BdfServer.
> * 由此, 用户便可正常使用 BDF. 然而, 我们强烈建议用户在 `BdfServer` 添加一非 root 账户并将该账户设为默认登录账户. 该步骤的指南可参见[此处](https://wsldl-pg.github.io/ArchW-docs/locale/zh-CN/How-to-Setup/#%E5%AE%8C%E6%88%90%E5%AE%89%E8%A3%85%E5%90%8E%E7%9A%84%E6%93%8D%E4%BD%9C).

## 常用命令
* 若需通过 PowerShell 来间接运行一个 BdfServer 上的命令, 如, `htop`, 可以使用下面的命令
``` PowerShell
wsl -d BdfServer htop
```

* 若需通过 PowerShell 来在当前 Windows 路径下运行一个 BdfServer上的命令, 如 `ls`, 可以使用下面的命令
``` PowerShell
wsl -d BdfServer ls
```

* 若需通过 PowerShell 来在一给定 Linux 路径 (如 `~/tasks`) 下运行一个 BdfServer 上的命令可使用下面的命令
``` PowerShell
wsl -d BdfServer --cd ~/tasks ls
```

* 如需将当前 Windows 路径下的文件复制或剪切至 BdfServer上的某一路径 (如 `~/tasks`)  可使用下面的 Windows PowerShell 命令
``` PowerShell
wsl -d BdfServer cp MyWindowsFile.txt ~/tasks/
```
或
``` PowerShell
wsl -d BdfServer mv MyWindowsFile.txt ~/tasks/
```

* 如需将 BdfServer 路径, 如 `~/tasks`, 下的某一文件复制或剪切至当前 Windows 路径可使用下面的 Windows PowerShell 命令
``` PowerShell
wsl -d BdfServer cp ~/tasks/MyLinuxFile.txt .
```
or
``` PowerShell
wsl -d BdfServer mv ~/tasks/MyLinuxFile.txt .
```

* 欲在一 Windows 路径下运行 BDF 计算任务可以使用下面的命令
``` PowerShell
wsl -d BdfServer bdf BdfCalculationInputFile.inp
```

* 如需使用 Windows File Explorer 来打开某一 BdfServer 路径, 如 `~/tasks`, 可执行下面的 Windows PowerShell 命令
``` PowerShell
wsl -d BdfServer --cd ~/tasks/ explorer.exe .
```

> * 根据 WSL 的版本不同直接在 Windows 路径下直接执行 BDF 计算可能是, 也可能不是一个好的方案. 对 WSL 1 来说, 如此行没有任何问题. 对 WSL2 来说, 由于 Windows 和 WSL 文件系统间的 IO 操作很慢, 使得此举不优. 因此, 对 WSL 2 来说, 我们建议 将BDF 输入文件复制至 BdfServer 的文件系统内并在 BdfServer 的文件系统内执行计算. 用于在 Windows 和 WSL 文件系统间进行文件复制和剪切的命令在上文已给出.

## 备注
* BDF Distributable Blank (BDB) 是一类似于可分发的BDF WSL 镜像 (BD). 它们的区别在于 BDB 上没有安装 BDF, 其上装有的是编译和安装 BDF 所需的依赖库及软件. 因此 BDB 的文件大小远远小于 BD 且可被重用数年 (我们仍然建议每年更新 BDB 一次), 而 BD 文件很大且每次 BDF 更新时都应被完全替换.
* 请 *勿* 为 `root` 账号设置密码. 请 *勿* 添加任何额外的账号.
* 由于可分发镜像中不仅包含 BDF 且包含它的依赖项, 可分发镜像文件的的大小可达 10 GB. 分发者应合理选择分发介质.
