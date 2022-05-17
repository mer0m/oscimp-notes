# Getting started with plutosdr

We assume the final folder tree will be as following:

```
~
└── git
    ├── analogdevicesinc
    ├── buildroot-2019.02
    ├── oscimpDigitala
    └── PlutoSDR
```

```
mkdir ~/git
cd ~/git
```

## Prepare a buildroot

Get the lastest version of buildroot compatible:
```
wget https://buildroot.org/downloads/buildroot-2019.02.tar.gz
tar -xf buildroot-2019.02.tar.gz
```

## Prepare plutosdr support for buildroot

```
git clone -b bma_pluto https://github.com/oscimp/PlutoSDR.git
```

## Get ADI HDL project

```
mkdir analogdevicesinc
git clone https://github.com/analogdevicesinc/hdl.git
```

## Get oscimpDigital

```
git clone --recursive https://github.com/oscimp/oscimpDigital.git
```

## Prepare bash environment

Add plutosdr board to buildroot.
```
source PlutoSDR/sourceme.ggm
```

Create your `oscimpDigital/settings.sh` to link oscimpDigital to buildroot.
```
cp oscimpDigital/settings.sh.sample oscimpDigital/settings.sh
```

Specify your setup with following variables:

`BOARD_NAME='plutosdr'`

`BR_DIR=~/git/buildroot-2019.02/`

### Example of `settings.sh`
```bash
# define the board being used. Must be adapted to either:
# plutosdr for the Xilinx Zynq-based ADI PlutoSDR board
# redpitaya for the Xilinx Zynq-based 14-bit legacy Red Pitaya board
# redpitaya16 for the new Xilinx Zynq-based 16-bit Red Pitaya board
# de0nansoc for the Altera/Intel Terrasic DE0Nano SoC
export BOARD_NAME='plutosdr'

#define Buildroot location
export BR2_DL_DIR='~/git/buildroot/dl'
export BR_DIR='~/git/buildroot-2019.02/'

# define target IP
# 192.168.0.10 for RedPitaya
# 192.168.2.1 for PlutoSDR
export IP=192.168.2.1

#only mandatory for plutosdr
export ADI_HDL_DIR=~/git/analogdevicesinc/hdl

# define where to install apps, drivers, etc
#export OSCIMP_DIGITAL_NFS='/nfs'

OSCIMP_DIGITAL=$(cd `dirname "${BASH_SOURCE[0]}"` && pwd)
source $OSCIMP_DIGITAL/app/setenv.sh        # defines $OSCIMP_DIGITAL_APP
source $OSCIMP_DIGITAL/fpga_ip/setenv.sh    # defines OSCIMP_DIGITAL_IP
source $OSCIMP_DIGITAL/lib/setenv.sh        # OSCIMP_DIGITAL_LIB
source $OSCIMP_DIGITAL/linux_driver/setenv.sh

export PATH=$PATH:$OSCIMP_DIGITAL_APP/tools/module_generator:$OSCIMP_DIGITAL_IP/tools/

# /!\ locale settings for Vivado (uses '.' as separator, as opposed to the French ',')
export LANG=en_US.UTF-8
# /!\ check /etc/locale.gen: en_US.UTF-8 UTF-8 must be UNcommented. If it was commented:
#     remove comment and execute as root locale-gen
```

Finnaly
```
source oscimpDigital/settings.sh
```

## Prepare your buildroot

### Configuration

Go to buildroot dir.
```
cd buildroot-2019.02
```

Set the default configuration.
```
make zynq_pluto_defconfig
```

To add optional packages for the specific application.
```
make menuconfig
```

### Make your buildroot base

**WARNING** this command can take several hours according to your internet connection

```
make
```

An image of the sdcard is created in `output/images/sdcard.img`.

## Build oscimpDigital tools, drivers and lib

Make module_generator:
```
cd ~/git/oscimpDigital/app/tools/module_genetator
make
sudo make install
```

Make libraries:
```
cd ~/git/oscimpDigital/lib
make
make install
```

Make drivers:
```
cd ~/git/oscimpDigital/linux_driver
make
```

## Make your buildroot with lib

```
cd ~/git/buildroot-2019.02
make
```

## [Install Vivado HL WebPack 2021.1](../vivado_install/2021.1.md)

## Create your first application

Here, we will use the [OscIMP project2](https://github.com/oscimp/oscimpDigital/tree/master/doc/tutorials/plutosdr/1-adalmPluto_within_OscimpDigital/project2) application.

```
cd ~/git/oscimpDigital/doc/tutorials/plutosdr/1-adalmPluto_within_OscimpDigital/project2

```

### Bitstream

```
cd design
make
make image
make dfu_frm
```

This last command could fail. To solve it:
```
/home/bma/git/PlutoSDR/board/pluto/post_image.sh image/
```

### Flash the plutosdr

There are 3 ways to flash the plutosdr:

1. Mount the PLUTOSDR volume, copy the `image/pluto.frm` and eject. Wait until the led stops blinking.
2. Use the dfu-tools: `make flash_dfu_frm`
3. Use the embedded flashing tool (useful for remote plutosdr): `scp image/pluto.frm root@192.168.0.2:`, then `ssh root@192.168.0.2` and `update_frm.sh pluto.frm && reboot`

### Modules

```
cd ..
module_generator module_generator.xml
webserver_generator module_generator.xml
```

### Linux application

We will use a webserver instead of the C app.
```
cd app
rm main.c
make
make install SUPP_FILE='*_webserver.py
```

### Run it on plutosdr

```
ssh root@192.168.2.1
cd /tmp/project2/bin
./project2_us.sh
./module_generator_webserver.py &
```

Start a web browser and connect to http://192.168.2.1:8080/

You can move the sliders or set values in spinbox, save your current config, or reload it after reboot.

Enjoy.
