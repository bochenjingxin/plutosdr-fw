# plutosdr-fw
PlutoSDR Firmware for the [ADALM-PLUTO](https://wiki.analog.com/university/tools/pluto "PlutoSDR Wiki Page") Active Learning Module

> **Fork Note (bochenjingxin/plutosdr-fw · branch: pluto-custom)**
>
> 本分支基于 **ADI 官方 v0.34 release** (`cbe7306 "PlutoSDR: Prepare for v0.34 release"`) 定制, 与 master 分支的 v0.40/Vivado 2025.1 版本线不互通。
>
> 配套版本如下 (与本地构建环境、Makefile、子模块指针对齐):
>
> | 项目 | 版本/说明 |
> |---|---|
> | PlutoSDR 固件版本 | **v0.34** (官方 release tag v0.34) |
> | Vivado 综合/实现工具 | **2019.1** (Makefile 默认 `VIVADO_VERSION ?= 2019.1`) |
> | VIVADO_SETTINGS 路径 | `/xilinx/Vivado/2019.1/settings64.sh` (已在 Makefile 中由默认 `/opt/Xilinx/` 修改为容器挂载路径 `/xilinx/`) |
> | 交叉编译链 (hard-float) | `arm-linux-gnueabihf-` (自 v0.30 起切换) |
> | SDK gcc 工具路径 | `/xilinx/SDK/2019.1/gnu/aarch32/lin/gcc-arm-linux-gnueabi/bin` |
> | 子模块: hdl | 指向 fork `bochenjingxin/hdl` 的 **pluto-custom** 分支 (含 6 个 HDL 改动) |
> | 子模块: linux | 指向 fork `bochenjingxin/linux` 的 **pluto-custom** 分支 (含设备树 `zynq-pluto-sdr.dtsi` 改动) |
> | 子模块: buildroot / u-boot-xlnx | 仍指向 ADI 官方 (v0.34 指针, 无改动) |
>
> 新增特性: `make` 会额外生成 **`build/unify.bin`** 一次烧写镜像 (boot.bin + uboot-env.bin + pluto.itb 拼接, 见下文说明)。

Latest binary Release : [![GitHub Release](https://img.shields.io/github/release/analogdevicesinc/plutosdr-fw.svg)](https://github.com/analogdevicesinc/plutosdr-fw/releases/latest)  [![Github Releases](https://img.shields.io/github/downloads/analogdevicesinc/plutosdr-fw/total.svg)](https://github.com/analogdevicesinc/plutosdr-fw/releases/latest)

Firmware License : [![Many Licenses](https://img.shields.io/badge/license-LGPL2+-blue.svg)](https://github.com/analogdevicesinc/plutosdr-fw/blob/master/LICENSE.md)  [![Many License](https://img.shields.io/badge/license-GPL2+-blue.svg)](https://github.com/analogdevicesinc/plutosdr-fw/blob/master/LICENSE.md)  [![Many License](https://img.shields.io/badge/license-BSD-blue.svg)](https://github.com/analogdevicesinc/plutosdr-fw/blob/master/LICENSE.md)  [![Many License](https://img.shields.io/badge/license-apache-blue.svg)](https://github.com/analogdevicesinc/plutosdr-fw/blob/master/LICENSE.md) and many others.

[Instructions from the Wiki: Building the image](https://wiki.analog.com/university/tools/pluto/building_the_image)

* Build Instructions (本 fork 版本配套)
```bash
 # 1. 系统依赖
 sudo apt-get install git build-essential fakeroot libncurses5-dev libssl-dev ccache
 sudo apt-get install dfu-util u-boot-tools device-tree-compiler libssl1.0-dev mtools
 sudo apt-get install bc python cpio zip unzip rsync file wget

 # 2. 克隆本 fork 及其子模块 (hdl/linux 会自动拉取定制分支)
 git clone --recursive -b pluto-custom https://github.com/bochenjingxin/plutosdr-fw.git
 cd plutosdr-fw

 # 3. 环境变量 (Vivado/SDK 安装路径按本 fork 默认的 /xilinx/ 挂载点)
 export CROSS_COMPILE=arm-linux-gnueabihf-
 export PATH=$PATH:/xilinx/SDK/2019.1/gnu/aarch32/lin/gcc-arm-linux-gnueabi/bin
 export VIVADO_SETTINGS=/xilinx/Vivado/2019.1/settings64.sh

 # 4. 构建 (除了官方产物外, 将额外生成 build/unify.bin 一次烧写镜像)
 make

 # 5. 仅构建一次烧写镜像 (当 boot.bin / uboot-env.bin / pluto.itb 已存在时)
 make unify
```

> 注意: 本 fork (v0.34) **仅在 Vivado 2019.1 + arm-linux-gnueabihf- hard-float 工具链下测试通过**; 官方 README 中提到的 Vivado 2018.2/2017.4 等版本未针对本定制改动做回归验证。
>
> 若 Vivado/SDK 安装在默认 `/opt/Xilinx/` 路径, 只需把上面的环境变量改回 `/opt/Xilinx/...`, 构建时可通过命令行覆盖 Makefile 默认值, 例如:
> ```bash
> make VIVADO_SETTINGS=/opt/Xilinx/Vivado/2019.1/settings64.sh
> ```


 If you receive an error similar to the following:
 ```
 Starting SDK. This could take few seconds... timeout while establishing a connection with SDK
    while executing
"error "timeout while establishing a connection with SDK""
    (procedure "getsdkchan" line 108)
    invoked from within
"getsdkchan"
    (procedure "createhw" line 26)
    invoked from within
"createhw {*}$args"
    (procedure "::sdk::create_hw_project" line 3)
    invoked from within
"sdk create_hw_project -name hw_0 -hwspec build/system_top.hdf"
    (file "scripts/create_fsbl_project.tcl" line 5)
```
you may be able to work around it by preventing eclipse from using GTK3 for the Standard Widget Toolkit (SWT). Prior to running make, also set the following environment variable: 
```bash
export SWT_GTK3=0
```
This problem seems to affect Ubuntu 16.04LTS only.

 * Updating your local repository 
 ```bash 
      git pull
      git submodule update --init --recursive
  ```
   
* Build Artifacts
 ```bash
      michael@HAL9000:~/devel/plutosdr-fw$ ls -AGhl build
      total 372M
      -rw-rw-r-- 1 michael   69 Apr 14 11:01 boot.bif
      -rw-rw-r-- 1 michael 459K Apr 14 11:01 boot.bin
      -rw-rw-r-- 1 michael 459K Apr 14 11:01 boot.dfu
      -rw-rw-r-- 1 michael 588K Apr 14 11:01 boot.frm
      -rw-rw-r-- 1 michael 254M Apr 14 11:01 legal-info-v0.33.tar.gz
      -rw-rw-r-- 1 michael 527K Apr 14 11:03 LICENSE.html
      -rw-rw-r-- 1 michael  11M Apr 14 11:01 pluto.dfu
      -rw-rw-r-- 1 michael  11M Apr 14 11:01 pluto.frm
      -rw-rw-r-- 1 michael   33 Apr 14 11:01 pluto.frm.md5
      -rw-rw-r-- 1 michael  11M Apr 14 11:01 pluto.itb
      -rw-rw-r-- 1 michael  20M Apr 14 11:01 plutosdr-fw-v0.33.zip
      -rw-rw-r-- 1 michael 571K Apr 14 11:01 plutosdr-jtag-bootstrap-v0.33.zip
      -rw-rw-r-- 1 michael 442K Apr 14 11:00 ps7_init.c
      -rw-rw-r-- 1 michael 442K Apr 14 11:00 ps7_init_gpl.c
      -rw-rw-r-- 1 michael 4,2K Apr 14 11:00 ps7_init_gpl.h
      -rw-rw-r-- 1 michael 4,8K Apr 14 11:00 ps7_init.h
      -rw-rw-r-- 1 michael 2,4M Apr 14 11:00 ps7_init.html
      -rw-rw-r-- 1 michael  31K Apr 14 11:00 ps7_init.tcl
      -rw-r--r-- 1 michael 5,4M Apr 14 11:00 rootfs.cpio.gz
      drwxrwxr-x 6 michael 4,0K Apr 14 11:01 sdk
      -rw-rw-r-- 1 michael  52M Apr 14 11:03 sysroot-v0.33.tar.gz
      -rw-rw-r-- 1 michael 943K Apr 14 11:01 system_top.bit
      -rw-rw-r-- 1 michael 476K Apr 14 11:00 system_top.hdf
      -rwxrwxr-x 1 michael 438K Apr 14 11:01 u-boot.elf
      -rw-rw---- 1 michael 128K Apr 14 11:01 uboot-env.bin
      -rw-rw---- 1 michael 129K Apr 14 11:01 uboot-env.dfu
      -rw-rw-r-- 1 michael 6,5K Apr 14 11:01 uboot-env.txt
      -rwxrwxr-x 1 michael 3,9M Apr 14 10:59 zImage
      -rw-rw-r-- 1 michael  19K Apr 14 11:00 zynq-pluto-sdr.dtb
      -rw-rw-r-- 1 michael  19K Apr 14 11:00 zynq-pluto-sdr-revb.dtb
      -rw-rw-r-- 1 michael  19K Apr 14 11:00 zynq-pluto-sdr-revc.dtb
 ```
 
 * Main targets
 
     | File  | Comment |
     | ------------- | ------------- | 
     | pluto.frm | Main PlutoSDR firmware file used with the USB Mass Storage Device |
     | pluto.dfu | Main PlutoSDR firmware file used in DFU mode |
     | boot.frm  | First and Second Stage Bootloader (u-boot + fsbl + uEnv) used with the USB Mass Storage Device |
     | boot.dfu  | First and Second Stage Bootloader (u-boot + fsbl) used in DFU mode |
     | uboot-env.dfu  | u-boot default environment used in DFU mode |
     | plutosdr-fw-vX.XX.zip  | ZIP archive containg all of the files above |  
     | plutosdr-jtag-bootstrap-vX.XX.zip  | ZIP archive containg u-boot and Vivao TCL used for JATG bootstrapping |       
     | **unify.bin** | **Single-image flash 一次烧写镜像 (boot.bin + uboot-env.bin + pluto.itb 拼接, 见下文)** |
 
  * Other intermediate targets

     | File  | Comment |
     | ------------- | ------------- |
     | boot.bif | Boot Image Format file used to generate the Boot Image |
     | boot.bin | Final Boot Image |
     | pluto.frm.md5 | md5sum of the pluto.frm file |
     | pluto.itb | u-boot Flattened Image Tree |
     | rootfs.cpio.gz | The Root Filesystem archive |
     | sdk | Vivado/XSDK Build folder including  the FSBL |
     | system_top.bit | FPGA Bitstream (from HDF) |
     | system_top.hdf | FPGA Hardware Description  File exported by Vivado |
     | u-boot.elf | u-boot ELF Binary |
     | uboot-env.bin | u-boot default environment in binary format created form uboot-env.txt |
     | uboot-env.txt | u-boot default environment in human readable text format |
     | zImage | Compressed Linux Kernel Image |
     | zynq-pluto-sdr.dtb | Device Tree Blob for Rev.A |
     | zynq-pluto-sdr-revb.dtb | Device Tree Blob for Rev.B|     
     | zynq-pluto-sdr-revc.dtb | Device Tree Blob for Rev.C|

 * unify.bin — Single-image Flash (一次烧写镜像)

    本 fork 在 `make` 完成后会额外生成 **`build/unify.bin`**,把 Bootloader、u-boot 环境变量与内核镜像树按 Zynq QSPI Flash 的启动布局拼接成单个文件,可直接通过 Flash 编程器或 JTAG 一次写入 QSPI,省去分文件烧写的步骤。

    **构建命令**:
    ```bash
    # 随完整构建自动生成
    make

    # 或单独拼接(需 boot.bin / uboot-env.bin / pluto.itb 已构建完成)
    make unify
    ```

    **Flash 布局 (单位: 128 KiB, `bs=128K`)**:

    | 偏移 (128K 扇区) | 字节偏移 | 长度 | 内容 | 来源文件 |
    |---|---|---|---|---|
    | `seek=0`  | `0x00000000` | ~460 KiB | Zynq Boot Image (FSBL + bitstream + u-boot) | `build/boot.bin` |
    | `seek=8`  | `0x00100000` | 128 KiB | u-boot default environment | `build/uboot-env.bin` |
    | `seek=16` | `0x00200000` | 剩余部分 | u-boot Flattened Image Tree (kernel + dtb + initramfs) | `build/pluto.itb` |

    **烧写方式**:

    1. **JTAG + Vivado Hardware Manager / XSCT**
       ```tcl
       # XSCT 脚本示例 (目标 QSPI 0x0, 按 128K 对齐写入)
       target 1
       configparams target.qspi_single 1
       program_flash -f build/unify.bin -offset 0x0 \
           -flash_type qspi-x4-single -fsbl build/sdk/fsbl/executable.elf \
           -verify
       ```

    2. **SD 卡启动后在 Linux 下写入 QSPI**
       ```bash
       flash_erase /dev/mtd0 0 0
       nandwrite -p /dev/mtd0 build/unify.bin
       # 或: mtd_debug write /dev/mtd0 0 $(stat -c%s build/unify.bin) build/unify.bin
       ```

    3. **SPI Flash 编程器脱机烧写 (CH341A / Flashcat 等)**
       直接选择 `unify.bin`,起始地址 `0x00000000` 烧录即可。

    > **注意**: 固件大小会随 buildroot/kernel 配置变化; unify.bin 默认从偏移 `16×128KB = 2 MiB` 开始放 pluto.itb, 意味着 boot.bin 与 uboot-env.bin 合计不能超过 2 MiB。若需要改布局, 请修改 `Makefile` 中 `unify` 目标的 `dd seek=` 参数。
 

