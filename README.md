# Create a working Rocket Chip bitstream on Xilinx devel boards by Eugene Tarassov's GitHub repo

Assuming your FPGA developer board can be programmed by JTAG via UART and you're going to install Vitis 2024.2

## Table of Contents
- [Install git make and libtinfo5 from deb package](#building-ball-c-and-run-the-simulation)
- [Clone the Tarassov's GitHub repo](#building-the-visualizer)
- [Patch the Makefile](#binary-download)
- [Install apt packages and check default JDK](#dependencies-and-credits)
- [Install Tarassov's repo submodules](#license)
- [Install Vitis 2024.2](#sdada)
- [MAKE YOUR FIST BITSTREAM](#dasdasda)
- [Launch Minicom](#ffff)
- [Upload the bitstream](#gggg)

## Install git make and libtinfo5 from deb package
cd ~
sudo apt update
sudo apt install git make
wget http://security.ubuntu.com/ubuntu/pool/universe/n/ncurses/libtinfo5_6.3-2ubuntu0.1_amd64.deb
sudo apt install ./libtinfo5_6.3-2ubuntu0.1_amd64.deb

## Clone the Tarassov's GitHub repo
git clone https://github.com/eugene-tarassov/vivado-risc-v
cd vivado-risc-v

## Patch the Makefile
wget Makefile.diff
patch Makefile Makefile.diff

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

## Install apt packages and check default JDK
make apt-install

WARNING! Before continue check your default JDK is 11.x and not 21.x!

sudo update-alternatives --config java
sudo update-alternatives --config javac

## Install Tarassov's repo submodules
make update-submodules

## Install Vitis 2024.2
sudo ./xsetup

And after Vitis setup completed type the following:

sudo /tools/Xilinx/Vitis/2024.2/scripts/installLibs.sh
sudo /tools/Xilinx/Vivado/2024.2/data/xicom/cable_drivers/lin64/install_script/install_drivers/install_drivers
sudo adduser $USER dialout
cd ~
git clone https://github.com/Digilent/vivado-boards
sudo mv vivado-boards/new/board_files /tools/Xilinx/Vivado/2024.2/data/boards/
sudo chown -R root:root /tools/Xilinx/Vivado/2024.2/data/boards

Finally copy Vitis icons from root desktop to yours: remember to right click on each icon and choose "Allow Launching" option
(Nella versione italiana di Ubuntu la directory di destinazione è Scrivania)

sudo cp /root/Desktop/*.desktop ~/Desktop
sudo chown $USER:$USER Desktop/*.desktop

## MAKE YOUR FIST BITSTREAM

Add Vivado to your path:
source /tools/Xilinx/Vivado/2024.2/settings64.sh

Available configs and board models are listed at
https://github.com/eugene-tarassov/vivado-risc-v?tab=readme-ov-file#build-fpga-bitstream 
make CONFIG=rocket32s1 BOARD=nexys-a7-50t bitstream

## Launch Minicom
Turn on your FPGA board and launch minicom in another Terminal window (usually my Nexys A7 UART is mapped to /dev/ttyUSB1)
sudo apt install minicom
minicom -D /dev/ttyUSB1

## Upload the bitstream
# 1. Return to your main Terminal window
# 2. Launch Vivado
vivado

# 3. In the Tasks pane click on "Open Hardware Manager"
# 4. Click "Open target" on the right of "No hardware target is open" label
# 5. Select "Auto Connect" option from popup menu
# 6. Inside Hardware pane right click on your board chip model (should be at the third level)
# 7. Select "Program Device..." option
# 8. Select the following bistream and leave empty the Debug probes file
~/vivado-risc-v/workspace/rocket32s1/vivado-nexys-a7-100t-riscv/nexys-a7-100t-riscv.runs/impl_1/riscv_wrapper.bit
# 9. Click the Program button


Now boot.elf stored in SD card will be loaded and I/O can by controlled by minicom window
