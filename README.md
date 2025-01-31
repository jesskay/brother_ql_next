## brother\_ql\_next

> **Note:**
> This version is incompatible with `brother_ql`, you must uninstall it *before* installing `brother_ql_next`  
> The *package* name is still `brother_ql` despite the new *distribution* name.

A Python package to control Brother QL label printers.
It implements the raster language of those printers and allows you to send instruction files to your printer.
In more details, the following is possible with this package:

* Create raster language files for the Brother label printers.
  They can be created from image files or programmatically in your own Python script.
* Print raster instruction files with your Brother label printer via different backends:
    * pyusb (works cross-platform)
    * network (works cross-platform for WiFi/Ethernet-enabled printers)
    * linux\_kernel (works on Linux only; uses the /dev/usb/lp0 device handles)

The following printers are claimed to be supported (✓ means verified by the author or by contributors):

* QL-500 (✓), QL-550 (✓), QL-560 (✓), QL-570 (✓), QL-580N
* QL-600, QL-650TD
* QL-700 (✓), QL-710W (✓), QL-720NW (✓)
* QL-800 (✓), QL-810W (✓), QL-820NWB (✓)
* QL-1050 (✓), QL-1060N (✓)
* QL-1100 (✓), QL-1100NWB, QL-1115NWB

The new QL-800 series can print labels with two colors (black and red) on DK-22251 labels.

Note: If your printer has an 'Editor Lite' mode, you need to disable it if you want to print via USB.
Make sure that the corresponding LED is not lit by holding the button down until it turns off.

## Why

The special feature of this package is that no printer driver is required for it to work.
This software bypasses the whole printing system including printer drivers and directly
talks to your label printer instead.
This means that even though Brother doesn't offer a driver for the Raspberry Pi (running
Linux on ARM) you can print nicely using this software.
And even if there are drivers for your operating system, many programs have difficulties to set
the page sizes and margins for the labels correctly.
If you want to print with high precision (which is important for barcodes for example),
you rather want to have control about every single pixel to be printed.
This is where brother\_ql comes into the game.

## Changelog
For notable changes between releases, see [the CHANGELOG file](CHANGELOG.md)

## Installation

To install the latest release:

    pip install --upgrade brother_ql_next

The upgrade flag makes sure, you get the latest version of brother\_ql but also
of its dependencies.

Alternatively, you can install the latest version from Github using:

    pip install --upgrade https://github.com/LunarEclipse363/brother_ql_next/archive/main.zip

This package will support Python 3.9+

In order to run the `brother_ql` command line utility, the directory it resides in
needs to be in the PATH envirnoment variable.
On some systems, the `pip install` command defaults to the `--user` flag resulting in the utility
being put in the `~/.local/bin` directory.
On those systems, extending the path variable via `export PATH="${PATH}:~/.local/bin"` is needed.

## Usage

The main user interface of this package is the command line tool `brother_ql`.

```
Usage: brother_ql [OPTIONS] COMMAND [ARGS]...

  Command line interface for the brother_ql Python package.

Options:
  -b, --backend [pyusb|network|linux_kernel]
  -m, --model [QL-500|QL-550|QL-560|QL-570|QL-580N|QL-600|QL-650TD|QL-700|QL-710W|QL-720NW|QL-800|QL-810W|QL-820NWB|QL-1050|QL-1060N|QL-1100|QL-1100NWB|QL-1115NWB]
  -p, --printer PRINTER_IDENTIFIER
                                  The identifier for the printer. This could
                                  be a string like tcp://192.168.1.21:9100 for
                                  a networked printer or
                                  usb://0x04f9:0x2015/000M6Z401370 for a
                                  printer connected via USB.
  --debug
  --version                       Show the version and exit.
  --help                          Show this message and exit.

Commands:
  analyze   interpret a binary file containing raster instructions for...
  discover  find connected label printers
  info      list available labels, models etc.
  print     print a label
  send      send an instruction file to the printer
  status    request status information from the printer
```

There are some global options available such as --model and --printer.
They can also be provided by environment variables (`BROTHER_QL_MODEL` and `BROTHER_QL_PRINTER`).

The global options are followed by a command such as `info` or `print`.
The most important command is the `print` command and here is its CLI signature:

    Usage: brother_ql print [OPTIONS] IMAGE [IMAGE] ...
    
      Print a label of the provided IMAGE.
    
    Options:
      -l, --label [12|29|38|50|54|62|102|17x54|17x87|23x23|29x42|29x90|39x90|39x48|52x29|62x29|62x100|102x51|102x152|d12|d24|d58]
                                      The label (size, type - die-cut or endless).
                                      Run `brother_ql info labels` for a full
                                      list including ideal pixel dimensions.
      -r, --rotate [auto|0|90|180|270]
                                      Rotate the image (counterclock-wise) by this
                                      amount of degrees.
      -t, --threshold FLOAT           The threshold value (in percent) to
                                      discriminate between black and white pixels.
      -d, --dither                    Enable dithering when converting the image
                                      to b/w. If set, --threshold is meaningless.
      -c, --compress                  Enable compression (if available with the
                                      model). Label creation can take slightly
                                      longer but the resulting instruction size is
                                      normally considerably smaller.
      --red                           Create a label to be printed on
                                      black/red/white tape (only with QL-8xx
                                      series on DK-22251 labels). You must use
                                      this option when printing on black/red tape,
                                      even when not printing red.
      --600dpi                        Print with 600x300 dpi available on some
                                      models. Provide your image as 600x600 dpi;
                                      perpendicular to the feeding the image will
                                      be resized to 300dpi.
      --lq                            Print with low quality (faster). Default is
                                      high quality.
      --no-cut                        Don't cut the tape after printing the label.
      --help                          Show this message and exit.

So, printing an image file onto 62mm endless tape on a QL-710W label printer can be as easy as:

    export BROTHER_QL_PRINTER=tcp://192.168.1.21
    export BROTHER_QL_MODEL=QL-710W
    brother_ql print -l 62 my_image.png

The available label names can be listed with `brother_ql info labels`:

     Name      Printable px   Description
     12         106           12mm endless
     29         306           29mm endless
     38         413           38mm endless
     50         554           50mm endless
     54         590           54mm endless
     62         696           62mm endless
     102       1164           102mm endless
     103       1200           103mm endless
     17x54      165 x  566    17mm x 54mm die-cut
     17x87      165 x  956    17mm x 87mm die-cut
     23x23      202 x  202    23mm x 23mm die-cut
     29x42      306 x  425    29mm x 42mm die-cut
     29x90      306 x  991    29mm x 90mm die-cut
     39x90      413 x  991    38mm x 90mm die-cut
     39x48      425 x  495    39mm x 48mm die-cut
     52x29      578 x  271    52mm x 29mm die-cut
     54x29      598 x  271    54mm x 29mm die-cut
     62x29      696 x  271    62mm x 29mm die-cut
     62x100     696 x 1109    62mm x 100mm die-cut
     102x51    1164 x  526    102mm x 51mm die-cut
     102x152   1164 x 1660    102mm x 153mm die-cut
     103x164   1200 x 1822    103mm x 164mm die-cut
     d12         94 x   94    12mm round die-cut
     d24        236 x  236    24mm round die-cut
     d58        618 x  618    58mm round die-cut

**Pro Tip™**:
For the best results, use image files with the matching pixel dimensions.
Die-cut labels have to be in the exact pixel dimensions stated above.
For endless label rolls, you can provide image files with a pixel width as stated above.
If you provide a file with different dimensions when creating an endless label file,
it will be scaled to fit the width.

### Backends

There are multiple backends for connecting to the printer available (✔: supported, ✘: not supported):

 Backend      | Kind | Linux   | Mac OS  | Windows
--------------|------|---------|---------|--------
network (1)   | TCP  | ✔       | ✔       | ✔
linux\_kernel | USB  | ✔ (2)   | ✘       | ✘
pyusb (3)     | USB  | ✔ (3.1) | ✔ (3.2) | ✔ (3.3)

Notes:

1. The network backend doesn't support reading back the printer state, currently.
   Failure such as *wrong label type* or *end of label roll reached* won't be detected by this software.
2. The label printer should show up automatically as `/dev/usb/lp0` when connected.
   Please check the ownership (user, group) of this file to be able to print as a regular user.
   Consider setting up a udev .rules file.
3. PyUSB is a Python wrapper allowing to implement USB communication in userspace.
   1. On Linux: install libusb1 as offered by your distribution: `sudo apt-get install libusb-1.0-0` (Ubuntu, Debian), `sudo zyppe in libusb-1_0-0` (OpenSUSE), `sudo pacman -S libusb` (Arch).
   2. On Mac OS: Install [Homebrew](https://brew.sh/) and then install libusb1 using: `brew install libusb`.
   3. On Windows: download [libusb-win32-devel-filter-1.2.6.0.exe](https://sourceforge.net/projects/libusb-win32/files/libusb-win32-releases/1.2.6.0/)
      from sourceforge and install it.
      After installing, you have to use the "Filter Wizard" to setup a "device filter" for the label printer.

### Legacy command line tools

For a long time, this project provided multiple command line tools, such as
`brother_ql_create`, `brother_ql_print`, `brother_ql_analyze`, and more.
The overview of those tools can still be found in the [LEGACY](./LEGACY.md) documentation.
The use of these tools is now considered deprecated and they will be
removed in a future release.

## Authors

This software package was written based on Brother's documentation
of its raster language and based on additional reverse engineering efforts.

**Current Maintainers:**

* LunarEclipse  
  <luna@lunareclipse.zone>

**Special thanks to:**

* Philipp Klaus  
  <philipp.l.klaus@web.de>

Many more have contributed by raising issues, helping to solve them,
improving the code and helping out financially.

## Contributing

There are many ways to support the development of brother\_ql:

* **File an issue** on Github, if you encounter problems, have a proposal, etc.
* **Submit a pull request** on Github if you improved the code and know how to use git.

Thanks to everyone helping to improve brother\_ql.

## Links

* The source code and issue tracker of this package is to be found on **Github**: [LunarEclipse363/brother\_ql\_next](https://github.com/LunarEclipse363/brother_ql_next).
* The **PyPi** release is avaiable at: [https://pypi.org/project/brother-ql-next/](https://pypi.org/project/brother-ql-next/)
* Original unmaintained version of `brother_ql`: [pklaus/brother\_ql](https://github.com/pklaus/brother_ql/)
