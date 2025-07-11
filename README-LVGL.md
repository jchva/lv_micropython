# Micropython + lvgl

**Micropython bindings to LVGL for Embedded devices, Unix and JavaScript**

[![Build lv_micropython unix port](https://github.com/lvgl/lv_micropython/actions/workflows/unix_port.yml/badge.svg)](https://github.com/lvgl/lv_micropython/actions/workflows/unix_port.yml)
[![Build lv_micropython stm32 port](https://github.com/lvgl/lv_micropython/actions/workflows/stm32_port.yml/badge.svg)](https://github.com/lvgl/lv_micropython/actions/workflows/stm32_port.yml)
[![esp32 port](https://github.com/lvgl/lv_micropython/actions/workflows/ports_esp32.yml/badge.svg)](https://github.com/lvgl/lv_micropython/actions/workflows/ports_esp32.yml) [![Build lv_micropython rp2 port](https://github.com/lvgl/lv_micropython/actions/workflows/rp2_port.yml/badge.svg)](https://github.com/lvgl/lv_micropython/actions/workflows/rp2_port.yml)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/lvgl/lv_micropython)  

To quickly run Micropython + LVGL from your web browser you can also use the [Online Simulator](https://sim.lvgl.io/).

**For information about Micropython lvgl bindings please refer to [lv_binding_micropython/README.md](https://github.com/lvgl/lv_binding_micropython/blob/master/README.md)**

See also [Micropython + LittlevGL](https://blog.lvgl.io/2019-02-20/micropython-bindings) blog post. (LittlevGL is LVGL's previous name.)  
For questions and discussions - please use the forum: https://forum.lvgl.io/c/micropython

Original micropython README: https://github.com/micropython/micropython/blob/master/README.md

## Relationship between `lv_micropython` and `lv_binding_micropython`

Originally, `lv_micropython` was created as an example of how to use [lv_binding_micropython](https://github.com/lvgl/lv_binding_micropython) on a Micropython fork.  

As such, we try to keep changes here as minimal as possible and we try to keep it in sync with Micropython upstream releases. We also try to add changes to `lv_binding_micropython` instead of to `lv_micropython`, when possible. (for example we keep all drivers in `lv_binding_micropython`,  etc.)

Eventually it turned out that many people prefer using `lv_micropython` directly and only a few use it as a reference to support LVGL on their own Micropython fork.

If you are only starting with Micropython+LVGL, it's recommended that you use `lv_micropython`, while porting a Micropython fork to LVGL is for advanced users.

Actual `lv_micropython` repo is using [LVGL binding](https://github.com/lvgl/lv_binding_micropython) as MicroPython C module. 

More details: https://docs.micropython.org/en/latest/develop/cmodules.html

## Build Instructions

First step is always to clone `lv_micropython` and update its submodules recursively:

```
git clone https://github.com/lvgl/lv_micropython.git
cd lv_micropython
git submodule update --init --recursive user_modules/lv_binding_micropython
```

Next step is to build the port you want to use.

Some basic build and deploy scripts are added to `scripts` folder, to easily build and deploy firmware to your device (or use unix port).

You can of course build firmwares manually with `make` commands, if build script is missing for the port or you want to override some build parameters.

### Unix (Linux) port

Using build script:

```
cd scripts
./build-unix.sh
cd ..
./ports/unix/build-lvgl/micropython
```

Manual build:

1. `sudo apt-get install build-essential libreadline-dev libffi-dev git pkg-config libsdl2-2.0-0 libsdl2-dev python3.8 parallel`
Python 3 is required, but you can install some other version of python3 instead of 3.8, if needed.
2. `git clone https://github.com/lvgl/lv_micropython.git`
3. `cd lv_micropython`
4. `git submodule update --init --recursive user_modules/lv_binding_micropython`
5. `make -C mpy-cross`
6. `make -C ports/unix submodules`
7. `make -C ports/unix`
8. `./ports/unix/build-lvgl/micropython`

## Unix (MAC OS) port

1. `brew install sdl2 pkg-config`
2. `sdl2-config --version` Get the sdl2 version
3. `git clone https://github.com/lvgl/lv_micropython.git`
4. `cd lv_micropython`
5. `git submodule update --init --recursive user_modules/lv_binding_micropython`
6. `sudo mkdir -p /usr/local/lib/`
7. `sudo cp /opt/homebrew/Cellar/sdl2/<YOUR_SDL_VERSION>/lib/libSDL2.dylib /usr/local/lib/`
8. `sudo cp -r /opt/homebrew/Cellar/sdl2/<YOUR_SDL_VERSION>/include /usr/local/`
9. `sed -i '' 's/ -Werror//' ports/unix/Makefile mpy-cross/Makefile` Remove -Werror from compiler parameters as Mac fails compilation otherwise
10. `make -C mpy-cross`
11. `make -C ports/unix submodules`
12. `make -C ports/unix VARIANT=lvgl`
13. `./ports/unix/build-lvgl/micropython`

### ESP32 port

Install ESP-IDF v5.x: https://docs.espressif.com/projects/esp-idf/en/v5.2.3/esp32/get-started/index.html#manual-installation

(you can configure ESP-IDF path in [scripts/env-variables-esp32.sh](./scripts/env-variables-esp32.sh) file)

Build and deploy with scripts:

```
cd scripts
./build-esp32.sh
./deploy-esp32.sh
```

Manual build:

Please run `esp-idf/export.sh` from your ESP-IDF installation directory as explained in the [Micropython ESP32 Getting Started documentation](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/get-started/#get-started-export)  
ESP-IDF version needs to match Micropython expected esp-idf, otherwise a warning will be displayed (and build will probably fail)
For more details refer to [Setting up the toolchain and ESP-IDF](https://github.com/lvgl/lv_micropython/blob/master/ports/esp32/README.md#setting-up-the-toolchain-and-esp-idf)

When using IL9341 driver, the color depth need to be set to match ILI9341. This can be done from the command line.
Here is the command to build ESP32 + LVGL which is compatible with ILI9341 driver:

```
make -C mpy-cross
make -C ports/esp32 LV_CFLAGS="-DLV_COLOR_DEPTH=16" BOARD=ESP32_GENERIC BOARD_VARIANT=SPIRAM deploy
```

Explanation about the parameters:
- `LV_CFLAGS` are used to override color depth, for ILI9341 compatibility.
  - `LV_COLOR_DEPTH=16` is needed if you plan to use the ILI9341 driver.
- `BOARD` - I use WROVER board with SPIRAM. You can choose other boards from `ports/esp32/boards/` directory.
- `deploy` - make command will create ESP32 port of Micropython, and will try to deploy it through USB-UART bridge.

For more details please refer to [Micropython ESP32 README](https://github.com/micropython/micropython/blob/master/ports/esp32/README.md).

### JavaScript port

Refer to the README of the `lvgl_javascript` branch: https://github.com/lvgl/lv_micropython/tree/lvgl_javascript_v8#javascript-port

### Raspberry Pi Pico port

This port uses [Micropython infrastructure for C modules](https://docs.micropython.org/en/latest/develop/cmodules.html#compiling-the-cmodule-into-micropython) and `USER_C_MODULES` must be given:

1. `git clone https://github.com/lvgl/lv_micropython.git`
2. `cd lv_micropython`
3. `git submodule update --init --recursive user_modules/lv_binding_micropython`
4. `make -C ports/rp2 BOARD=PICO submodules`
5. `make -j -C mpy-cross`
6. `make -j -C ports/rp2 BOARD=PICO USER_C_MODULES=../../user_modules/lv_binding_micropython/bindings.cmake`

#### Troubleshooting

If you experience unstable behaviour, it is worth checking the value of *MICROPY_HW_FLASH_STORAGE_BASE* against the value of *__flash_binary_end* from the firmware.elf.map file.
If the storage base is lower than the binary end, parts of the firmware will be overwritten when the micropython filesystem is initialised.

## Super Simple Example

First, LVGL needs to be imported and initialized

```python
import lvgl as lv
lv.init()
```

Then event loop, display driver and input driver needs to be registered.
Refer to [Porting the library](https://docs.lvgl.io/8.0/porting/index.html) for more information.
Here is an example of registering SDL drivers on Micropython unix port:

```python
# Create an event loop and Register SDL display/mouse/keyboard drivers.
from lv_utils import event_loop

WIDTH = 480
HEIGHT = 320

event_loop = event_loop()
disp_drv = lv.sdl_window_create(WIDTH, HEIGHT)
disp_drv.set_default()
display = lv.display_get_default()

group = lv.group_create()
group.set_default()

mouse = lv.sdl_mouse_create()
mouse.set_display(display)
mouse.set_group(group)

keyboard = lv.sdl_keyboard_create()
keyboard.set_display(display)
keyboard.set_group(group)
```

Here is an alternative example, for registering ILI9341 drivers on Micropython ESP32 port:

```python
import lvgl as lv

# Import ILI9341 driver and initialized it

from ili9341 import ili9341
disp = ili9341()

# Import XPT2046 driver and initialize it

from xpt2046 import xpt2046
touch = xpt2046()
```

By default, both ILI9341 and XPT2046 are initialized on the same SPI bus with the following parameters:

- ILI9341: `miso=5, mosi=18, clk=19, cs=13, dc=12, rst=4, power=14, backlight=15, spihost=esp.HSPI_HOST, mhz=40, factor=4, hybrid=True`
- XPT2046: `cs=25, spihost=esp.HSPI_HOST, mhz=5, max_cmds=16, cal_x0 = 3783, cal_y0 = 3948, cal_x1 = 242, cal_y1 = 423, transpose = True, samples = 3`

You can change any of these parameters on ili9341/xpt2046 constructor.
You can also initialize them on different SPI buses if you want, by providing miso/mosi/clk parameters. Set them to -1 to use existing (initialized) spihost bus.

Now you can create the GUI itself:

```python

# Create a screen with a button and a label

scr = lv.obj()
btn = lv.button(scr)
btn.align_to(lv.scr_act(), lv.ALIGN.CENTER, 0, 0)
label = lv.label(btn)
label.set_text("Hello World!")

# Load the screen

lv.screen_load(scr)

```

## More information

More info about LVGL:
- Website https://lvgl.io
- GitHub: https://github.com/lvgl/lvgl
- Documentation: https://docs.lvgl.io/master/get-started/bindings/micropython.html
- Examples: https://docs.lvgl.io/master/examples.html
- More examples: https://github.com/lvgl/lv_binding_micropython/tree/master/examples

More info about lvgl Micropython bindings:
- https://github.com/lvgl/lv_binding_micropython/blob/master/README.md

Discussions about the Micropython binding: https://github.com/lvgl/lvgl/issues/557

More info about the unix port: https://github.com/micropython/micropython/wiki/Getting-Started#debian-ubuntu-mint-and-variants
