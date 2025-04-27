# Create a working Rocket Chip bitstream for Xilinx boards on Ubuntu 24.04 LTS by Eugene Tarassov's GitHub repo

Assuming your Linux box is an Ubuntu 24.04 LTS

Assuming your FPGA developer board can be programmed by JTAG via UART and you're going to install Vitis 2024.2

## Table of Contents
- [Install git make and libtinfo5 from deb package](#install-git-make-and-libtinfo5-from-deb-package)
- [Clone the Tarassov's GitHub repo](#clone-the-tarassovs-github-repo)
- [Patch the Makefile](#patch-the-makefile)
- [Install apt packages and check default JDK](#install-apt-packages-and-check-default-jdk)
- [Install Tarassov's repo submodules](#install-tarassovs-repo-submodules)
- [Install Vitis 2024.2](#install-vitis-20242)
- [MAKE YOUR FIST BITSTREAM](#make-your-fist-bitstream)
- [Launch Minicom](#launch-minicom)
- [Upload the bitstream](#upload-the-bitstream)

## Install git make and libtinfo5 from deb package
```bash
cd ~
sudo apt update
sudo apt install git make
wget http://security.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.3-2ubuntu0.1_amd64.deb
sudo apt install ./libtinfo5_6.3-2ubuntu0.1_amd64.deb
```

## Clone the Tarassov's GitHub repo
```bash
git clone https://github.com/eugene-tarassov/vivado-risc-v
cd vivado-risc-v
```

## Patch the Makefile
In Ubuntu 24.04 LTS you have to patch Makefile in root dir because ```libtinfo5``` is not available in apt repo and SBT requires OpenJDK 11.x instead of 21.x which is the ```default-jdk``` in Ubuntu 24.04.

```bash
curl -OL https://raw.githubusercontent.com/dgiovanardi/rocket-chip-on-ubuntu-24/refs/heads/main/Makefile.diff
patch Makefile Makefile.diff
```

Content of Makefile.diff:
```bash
--- Makefile.orig	2025-04-27 12:38:01.466581298 +0200
+++ Makefile	2025-04-27 12:38:01.466581298 +0200
@@ -30,8 +30,10 @@
 apt-install:
 	sudo apt update
 	sudo apt upgrade
-	sudo apt install default-jdk device-tree-compiler curl gawk \
-	 libtinfo5 libmpc-dev libssl-dev gcc gcc-riscv64-linux-gnu flex bison bc parted udev dosfstools
+	sudo apt install openjdk-11-jdk device-tree-compiler curl gawk \
+	 libmpc-dev libssl-dev gcc gcc-riscv64-linux-gnu flex bison bc parted udev dosfstools
+	wget http://security.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.3-2ubuntu0.1_amd64.deb
+	sudo apt install libtinfo5_6.3-2ubuntu0.1_amd64.deb
 ifeq ($(shell test -r /etc/os-release && . /etc/os-release && echo $$VERSION_CODENAME),jammy)
 	sudo apt install python-is-python3
 else
```

## Install apt packages and check default JDK
```bash
make apt-install
```

> **<ins>WARNING!</ins> Before continue check your default JDK is 11.x and not 21.x!**

```bash
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

## Install Tarassov's repo submodules
```bash
make update-submodules
```

## Install Vitis 2024.2
Run xsetup
```bash
sudo ./xsetup
```

And after Vitis setup completed type the following:
```bash
sudo /tools/Xilinx/Vitis/2024.2/scripts/installLibs.sh
sudo /tools/Xilinx/Vivado/2024.2/data/xicom/cable_drivers/lin64/install_script/install_drivers/install_drivers
sudo adduser $USER dialout
cd ~
git clone https://github.com/Digilent/vivado-boards
sudo mv vivado-boards/new/board_files /tools/Xilinx/Vivado/2024.2/data/boards/
sudo chown -R root:root /tools/Xilinx/Vivado/2024.2/data/boards
```

Finally copy Vitis icons from root desktop to yours: remember to right click on each icon and choose "Allow Launching" option\
(Nella versione italiana di Ubuntu la directory di destinazione è Scrivania)
```bash
sudo cp /root/Desktop/*.desktop ~/Desktop
sudo chown $USER:$USER Desktop/*.desktop
```

## MAKE YOUR FIST BITSTREAM

Add Vivado to your path:
```bash
source /tools/Xilinx/Vivado/2024.2/settings64.sh
```

Available configs and board models are listed at\
[https://github.com/eugene-tarassov/vivado-risc-v?tab=readme-ov-file#build-fpga-bitstream](https://github.com/eugene-tarassov/vivado-risc-v?tab=readme-ov-file#build-fpga-bitstream)
```bash
make CONFIG=rocket32s1 BOARD=nexys-a7-100t bitstream
```

## Launch Minicom
Turn on your FPGA board and launch minicom in another Terminal window\
(usually my Nexys A7 UART is mapped to /dev/ttyUSB1)
```bash
sudo apt install minicom
minicom -D /dev/ttyUSB1
```

## Upload the bitstream
1. Return to your main Terminal window
2. Launch Vivado
```bash
vivado
```

3. In the Tasks pane click on "Open Hardware Manager"
4. Click "Open target" on the right of "No hardware target is open" label
5. Select "Auto Connect" option from popup menu
6. Inside Hardware pane right click on your board chip model (should be at the third level)
7. Select "Program Device..." option
8. Select the following bistream and leave empty the Debug probes file
```
~/vivado-risc-v/workspace/rocket32s1/vivado-nexys-a7-100t-riscv/nexys-a7-100t-riscv.runs/impl_1/riscv_wrapper.bit
```
9. Click the Program button

> **Now boot.elf stored in SD card will be loaded and I/O can by controlled by minicom window**
