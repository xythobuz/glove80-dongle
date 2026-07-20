# Glove80 nRF52840 Dongle (PCA10059)

This is my attempt at getting an [nRF52840 Dongle (PCA10059)](https://www.nordicsemi.com/Products/Development-hardware/nRF52840-Dongle) to work as a BLE central for my Glove80 keyboard.

I started out with ["zmk-dongle-config" by stammy](https://github.com/stammy/zmk-dongle-config), which didn't need much adjustmends to build, but I wasn't getting any keystrokes.
Looking at the USB logs, the key presses were transmitted and detected, but it didn't know what to do with them.
So I simply added the physical-layout and keymap related stuff that was missing, as explained in the [documentation for dongles](https://zmk.dev/docs/hardware-integration/dongle).
This got me a working upstream ZMK Glove80 build with Dongle support.

## Details

There are two options, the [upstream ZMK](https://github.com/zmkfirmware/zmk) and the [MoErgo fork](https://github.com/moergo-sc/zmk).

I got a working version built with the [upstream commit `a23b84a` or so](https://github.com/zmkfirmware/zmk/commit/a23b84ae290883cddc2a43bde3f41649f0a3e3b6).
Unfortunately this does not have the same support for RGB key lighting and indicators as the original Glove80 firmware.
It works for typing with the keyboard, but I'm not sure how to check eg. battery status.
Pre-built binaries from this upstream build are available [as a Release](https://github.com/xythobuz/glove80-dongle/releases/tag/upstream-v1).

I haven't gotten the Moergo fork to build, even with their [zephyr-4-1 branch](https://github.com/moergo-sc/zmk/tree/zephyr-4-1).
On my attempts to build with this, I reverted to an older [build-user-config.yml script](https://github.com/zmkfirmware/zmk/commit/8266433d6b85cfecf57a4ba48e27e1eebb3de7b2) to avoid a broken check for ZMK board stuff.
Otherwise only the board name for the dongle changes, and of course the indicator-stuff with the keymap.
As the build fails with missing functions, I guess the problem is more fundamental.
The required data is only available on the Central, which is now the dongle, so without a mechanism to transfer this state, the left-hand side can't know or show them.

## Dongle

### Preparing nrfutil

Nordic has changed the way nrfutil works, and it's now closed-source and binary-only.
This is how to get it and install the required dependencies to do what we need.

    wget -O nrfutil https://files.nordicsemi.com/ui/api/v1/download?repoKey=swtools&path=external/nrfutil/executables/x86_64-unknown-linux-gnu/nrfutil&isNativeBrowsing=false
    chmod a+x nrfutil

    ./nrfutil install nrf5sdk-tools
    ./nrfutil install device

### Flashing Firmware

You need to convert the generated `.bin` file to a `.zip` for the DFU bootloader in the dongle.
Then you can flash via the same DFU bootloader.
To enter the bootloader, stick the dongle into an USB port, then press the `RESET` button.
    
    ./nrfutil pkg generate --hw-version 52 --sd-req=0x00  --application glove80_dongle-nrf52840dongle__zmk-zmk.bin --application-version 1 dongle.zip

    ./nrfutil device program --firmware dongle.zip --traits nordicDfu

## Keyboard

Flash the two halves [as usual](https://docs.moergo.com/glove80-user-guide/customizing-key-layout/#putting-glove80-into-bootloader-for-firmware-loading).
When switching to the dongle or back, remember to flash the settings-reset firmware in between, to clear cached Bluetooth pairing info.
