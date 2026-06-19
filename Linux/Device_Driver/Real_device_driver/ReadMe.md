# Linux Device Driver Tutorial – Programming

This repository is a collection of example Linux kernel modules and character device drivers from an introductory device driver tutorial series. It focuses on practical programming: building, loading, and testing real drivers on Linux.

## Overview

This tutorial series covers:

- Basics of Linux device drivers and kernel modules
- Character device drivers (read/write/open/release)
- Major and minor numbers for character devices
- Dynamic device file creation under `/dev`
- Using the `cdev` structure and `file_operations`
- Passing arguments (parameters) to kernel modules
- Build systems and module development workflow

Each part in the series has its own directory or file with example code and a small README explaining its goals.

## Goals of This Repository

The goals are:

- To provide working example code for learning Linux device driver programming
- To keep all examples in a single place for easy reference and experimentation
- To demonstrate modern kernel APIs for character drivers (`cdev`, `alloc_chrdev_region`, etc.)
- To show how to build modules with a `Makefile`, load them with `insmod`/`modprobe`, and test them from user space

## Repository Structure

Example structure (adapt to your actual layout):

```text
linux-device-driver-programming/
  part1-introduction/
    README.md
    hello_module.c
    Makefile
  part2-first-device-driver/
    README.md
    first_driver.c
    Makefile
  part3-passing-arguments/
    README.md
    param_driver.c
    Makefile
  major-minor-numbers/
    README.md
    char_dev_major_minor.c
    Makefile
  device-file-creation/
    README.md
    dev_file_creation.c
    Makefile
  cdev-file-operations/
    README.md
    cdev_fileops.c
    Makefile
  README.md          # This file
```

You can reorganize or rename directories/files to match your actual source.

## Common Build and Test Commands

For each part:

Build:

```bash
cd partX-...
make
```

Load:

```bash
sudo insmod <module_name>.ko
dmesg | tail
```

Check devices:

```bash
cat /proc/devices
ls /dev/your_device_name
```

Test:

```bash
echo "test" | sudo tee /dev/your_device_name
sudo cat /dev/your_device_name
```

Unload:

```bash
sudo rmmod <module_name>
dmesg | tail
```

Clean:

```bash
make clean
```

## Prerequisites

- Linux system (e.g., Ubuntu) with kernel headers and build tools installed
- Basic C programming knowledge
- Familiarity with:
  - `make`, `gcc`
  - `insmod`, `rmmod`, `modprobe`
  - `dmesg`, `/proc/devices`, `/sys`
- Root access (or `sudo`) for loading modules and creating device nodes

## Minimal Build Environment Setup (Ubuntu Example)

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

Then clone or copy your source files and run `make` in each part directory.

## How to Use This Repository

1. Pick a part that matches what you want to learn (e.g., “first device driver” or “cdev + file_operations”).
2. Read that part’s `README.md`.
3. Build the module with `make`.
4. Load it with `insmod` and inspect kernel logs with `dmesg`.
5. Test using shell commands (`echo`, `cat`) on the device file.
6. Unload with `rmmod` and observe cleanup messages.

Repeat for each part, gradually building your understanding.

## Conclusion
This is just a basic linux device driver which explains about the real read and write of the device file.

Please update your Beaglebone board's kernel directory in the Makefile.

Build for Beaglebone:
	sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi-

Build for Raspberry Pi or Virtualbox Ubuntu:
	sudo make

Build the application using the below command for Ubuntu and Raspberry Pi.
		gcc -o test_app test_app.c
Build the application using the below command for BeagleBone.	
		arm-linux-gnueabihf-gcc -o test_app test_app.c


Please refer this URL for the complete tutorial of this example source code.
https://embetronicx.com/tutorials/linux/device-drivers/linux-device-driver-tutorial-programming/

You can check the video tutorial of this project - https://youtu.be/xp9HTR6a98I

The Linux Device Driver Video Playlist - https://www.youtube.com/watch?v=BRVGchs9UUQ&list=PLArwqFvBIlwHq8WMKgsXSQdqIvymrEz9k

How to Setup Ubuntu and Raspberry PI - https://www.youtube.com/watch?v=e6gNeje3ljA
How to Setup BeagleBone and Cross compile the kernel - https://www.youtube.com/watch?v=am-dgmrMgYY&t 
