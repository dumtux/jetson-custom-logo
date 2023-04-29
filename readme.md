# jetson-custom-logo

Replacing the default boot logo of Nvicia Jetson modules with a custom one.

>  Need to have a correct JetPack version installed on the host OS (a workstation, not the Jetson module itself).

## Key Concepts

* The boot logo image is saved in the bootloader.
* The boot logo is loaded and displayed by bootloader before the kernel is loaded.
* Thus, it cannot be done by building a custom kernel or a custom image.
* It can be done only by flashing the bootloader in the recovery mode.
* The logo image is compressed in LZ format. The default compressed blob is: `Linux_for_Tegra/bootloader/bmp.blob`

## HowTo

Prepare 3 custom logo images in different sizes: `640x480`, `1280x720`, `1920x1080`.
Copy the images under the `Linux_for_Tegra/tools/bmp-splash/` folder.

```
<path-to-sdk>/Linux_for_Tegra/tools/bmp-splash/
├── bmp-blob-README.txt
├── BMP_generator_L4T.py
├── config_file.example
├── genbmpblob_L4T.sh
├── custom-logo-480.bmp
├── custom-logo-720.bmp
└── custom-logo-1080.bmp
```

Edit the `config_file.example` like this.

```
custom-logo-480.bmp nvidia 480;
custom-logo-720.bmp nvidia 720;
custom-logo-1080.bmp nvidia 1080
```

Install `liblz4-tool` if it's not on the host system.
And confirm its installtion path is `/usr/bin/lz4c`.
This is used for blob compression script.
We will not use `liblz4-tool` directly.

```
$ sudo apt install liblz4-tool
$ whereis lz4c
lz4c: /usr/bin/lz4c /usr/share/man/man1/lz4c.1.gz
```

Generate `custom-bmp.blob`

```
$ cd <path-to-sdk>/Linux_for_Tegra/tools/bmp-splash/
$ OUT=$PWD ./genbmpblob_L4T.sh t194 ./config_file ./BMP_generator_L4T.py /usr/bin/lz4c custom-bmp.blob
```

Note that the general command format is like the following, and need to use correct **chip name** for the Jetson module.

```
# Jetson Nano - <chip> = t194
# Jetson Xavier NX - <chip> = t210
# ...
$ OUT=$PWD ./genbmpblob_L4T.sh <chip> <product config file> <path to blob_generator> [ <path to lz4c> ] <output file>
```

Boot the Jetson module in a **recovery** mode by plugging a jumper on the Jetson module dev boards.
Note that jumper positions may vary for each dev board types.

```
$ cd <path-to-sdk>/Linux_for_Tegra/
$ sudo ./flash.sh -r -k BMP --image ./tools/bmp-splash/custom-bmp.blob jetson-nano-emmc mmcblk0p1
```

> Note that the general command format is like the following, and need to use correct **board name** for the Jetson module.
> 
> ```
> # Jetson Nano 4GB eMMC - <board> = jetson-nano-emmc
> # Jetson Nano 4GB SD card - <board> = jetson-nano-devkit
> # Jetson Xavier NX eMMC - <board> = jetson-xavier-nx-en715
> # ...
> $ sudo ./flash.sh -r -k BMP --image <path to custom logo blob> <board> mmcblk0p1
> ```
