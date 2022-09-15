# Riscv-Debian-on-Litex-Rocket-on-qmtech-wukong-FPGA
Running Debian on Litex/Rocket on qmtech wukong FPGA

Inspired from https://github.com/tongchen126/Boot-Debian-On-Litex-Rocket 
I managed to run a -pretty much - stable RISCV64 Debian with a Qmtech's Wukong board version 2 and Litex/rocket.
I used a Debian Buster. The steps I folllowed:

1) I Installed Linux on Litex Rocket (https://github.com/litex-hub/linux-on-litex-rocket) and followed the steps 1-4 in Prerequisites to install litex.

2) I do not really know what is the bit width of the point-to-point AXI link connecting the CPU and LiteDRAM controller. The instructions does not seem to apply, I'm not getting either "INFO:SoC:Matching AXI MEM data width (XXX)" or "INFO:SoC:Converting MEM data width: XXX to YYY via Wishbone" lines in the output. So I guessed that I need 128 bits based on the DRAM chip (MT41K128M16), that is the same as with digilent arty. Also tongchen126 mentioned linux4d and full2d. I confgured litex to include all the cpu variants that were missing:

      ```
      cd pythondata-cpu-rocket/pythondata_cpu_rocket/verilog
      vi update.sh   (see the corresponding file in releases)
      ./update.sh (will take some time)
      vi litex/litex/soc/cores/cpu/rocket/core.py  (see resulting file in releases)

3) I use Vivado. In "Building the Gateware (FPGA Bitstream)" I tried two cases: linux4d (4 cores without FPU) and fulld (1 core with FPU). For some reason I'm not able to synthesize a full2d cpu, it requires 67221 SLUTS, while I have only 63400. I found that adding L2 cache highly improves the stability of the SOC. I used 2M, which may be a bit overkill :) The clock frequency is 100MHz. I included ethernet, video terminal and an sdcard. Special attention, which board you have, mine is version 2. The comamnds that I used were :
  ```
  litex-boards/litex_boards/targets/qmtech_wukong.py --build --with-ethernet  --with-sdcard --with-video-terminal  \
      --cpu-type rocket  --cpu-variant linux4d  --l2-size 2097152 --sys-clk-freq 100e6  --board-version 2
  and
  litex-boards/litex_boards/targets/qmtech_wukong.py --build --with-ethernet  --with-sdcard --with-video-terminal  \
      --cpu-type rocket  --cpu-variant fulld  --l2-size 2097152 --sys-clk-freq 100e6  --board-version 2
```

4) I used the dts files for nexys4ddr.dts and nexys4ddr_fpu.dts as prototypes to make the corresponding files for the board. The main difference is the 256MB DRAM chip. Also I modified the linux command line to make the boot more smooth (both files in releases)

5) 
