# Dev for RP2040 without Docker

## Setup dev tools for rp2040 on Debian *(once per PC, ... upgrading the toolchain is not yet figured out)*

Install needed packages ...

```` bash
sudo usermod -aG plugdev vscode
sudo apt install build-essential pkg-config libusb-1.0-0-dev
sudo apt  install --no-install-recommends cmake build-essential wget ca-certificates gdb-multiarch automake autoconf libtool libftdi-dev libusb-1.0-0-dev pkg-config clang-format libhidapi-dev
````

Get the SDK

```` bash
mkdir ~/opt
cd ~/opt
git clone -b master https://github.com/raspberrypi/pico-sdk.git --depth=1  
cd pico-sdk/  
git submodule update --init
````

Setup path to the SDK

```` bash
vi ~/.profile
# -> export PICO_SDK_PATH="$HOME/opt/pico-sdk"
source ~/.profile
````  

Get GCC

````bash
cd ~/opt 
TOOLCHAINURL="https://developer.arm.com/-/media/Files/downloads/gnu/12.2.rel1/binrel"
TOOLCHAINARCHIVE="arm-gnu-toolchain-12.2.rel1-x86_64-arm-none-eabi.tar.xz"
TOOLCHAINDIR="arm-gnu-toolchain-12.2.rel1-x86_64-arm-none-eabi"
wget $TOOLCHAINURL/$TOOLCHAINARCHIVE 
tar xf $TOOLCHAINARCHIVE

sudo cp -r arm-gnu-toolchain-12.2.rel1-x86_64-arm-none-eabi/* /usr/local/

````

Build custom OpenOCD for picoprobe

```` bash
cd ~/opt  
git clone https://github.com/raspberrypi/openocd.git --depth=1 --recurse-submodules
cd openocd/
./bootstrap
./configure --enable-ftdi --enable-sysfsgpio --enable-picoprobe --enable-cmsis-dap
make -j 8
sudo make install
````
  
Build picotool

````bash
cd ~/opt
git clone https://github.com/raspberrypi/picotool.git --depth=1
cd picotool/
mkdir build
cd build/
cmake ../
make -j 8
sudo cp picotool /usr/local/bin/
````

Build bootterm(bt)

````bash
cd ~/opt
git clone https://github.com/wtarreau/bootterm.git --depth=1
cd bootterm/
make -j 8    
sudo cp bin/bt /usr/local/bin/
````
  
## Create the project

Create a skeleton in a new folder(for CMake with the build directory inside the repo ... ):

````bash
mkdir helloWorld
cd helloWorld
git init .
git submodule add https://github.com/raspberrypi/pico-sdk.git
mkdir build
touch CMakeLists.txt
touch main.cpp
touch .gitignore
````

The CMakeLists.txt looks like this and text IO is via UART, not USB(can be changed later ...):

````cmake
cmake_minimum_required(VERSION 3.13)

# initialize pico-sdk from submodule
# note: this must happen before project()
include(pico-sdk/pico_sdk_init.cmake)

project(hello_world)

# initialize the Raspberry Pi Pico SDK
pico_sdk_init()

# rest of your project

add_executable(hello_world
	main.cpp
)

# Add pico_stdlib library which aggregates commonly used features
target_link_libraries(hello_world pico_stdlib)

# create map/bin/hex/uf2 file in addition to ELF.
pico_add_extra_outputs(hello_world)
````

The main.cpp looks like this:

````c++
#include <stdio.h>
#include "pico/stdlib.h"

int main() {
    setup_default_uart();
    printf("Hello, world!\n");
    return 0;
}
````

To build this, do:

````bash
cd build
cmake ..
make hello_world
````

This will create all the files needed to run the program. One of the created files, hello_world.uf2, can be copied to the PiPico if it is connected as storage medium.
This proves it works ... but if you want to debug you stuff you will need more.

## ToDo

* Explain picoprobe
* Explain vscode
    * needed plugins
    * needed (plugin) configuration
...