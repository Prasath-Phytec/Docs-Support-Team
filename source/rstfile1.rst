.. START:section:0:0
Learning Objectives
===================

.. END:section:0:0

.. START:list:0:0:0

.. LIST_META: {"type": "bullet", "head": "", "items": ["Explain what a PHYTEC BSP contains and why each board needs board-specific software.", "Read a PHYTEC BSP release name, Yocto machine name, image name, and distro name.", "Select a BSP release that matches your exact SoM, carrier board, and requirements, and record the choice.", "Identify which bootloader a release uses (U-Boot or Barebox) instead of assuming.", "Decide which component to work on for a given change and identify which later guide in this series covers it.", "Describe what the NXP, TI, and PHYTEC pinmux tools do and what they do not do."]}

- Explain what a PHYTEC BSP contains and why each board needs board-specific software.
- Read a PHYTEC BSP release name, Yocto machine name, image name, and distro name.
- Select a BSP release that matches your exact SoM, carrier board, and requirements, and record the choice.
- Identify which bootloader a release uses (U-Boot or Barebox) instead of assuming.
- Decide which component to work on for a given change and identify which later guide in this series covers it.
- Describe what the NXP, TI, and PHYTEC pinmux tools do and what they do not do.

.. END:list:0:0:0

.. START:section:0:1
Hardware Terminology
====================

.. END:section:0:1

.. START:main_content:0:1
One product has several names. Using the wrong one is the most common reason people download the wrong BSP.

.. END:main_content:0:1

.. START:table:0:1:0

.. csv-table:: Table 1. Hardware terms used in PHYTEC documentation
   :header: "Term", "Simple meaning", "PHYTEC example / why it matters"
   :widths: 20, 35, 45
   :align: center

   "SoC", "The processor chip: CPU cores plus peripheral controllers (UART, SPI, I²C, Ethernet and so on).", "NXP i.MX 8M Plus, TI AM62x. The SoC decides which pin functions, clocks and boot options exist."
   "SoM", "System on Module: a small board carrying the SoC, RAM, storage and power circuitry.", "phyCORE-i.MX 8M Plus, phyCORE-AM62x. Product number and PCB revision both matter."
   "Carrier board", "The board that brings SoM signals out to connectors and on-board devices.", "phyBOARD-Pollux (i.MX 8M Plus), phyBOARD-Lyra (AM62x/AM62Ax). It decides which SoM signals are really usable."
   "Development kit", "A SoM plus carrier board, power supply and accessories.", "Good for bring-up. The kit name alone may not identify the SoM and carrier revisions."
   "BSP", "The board-specific software and configuration that lets Linux run on the hardware.", "Bootloader, kernel, device tree and Yocto build metadata for one SoM family."
   "BSP release", "One versioned, tested set of the above.", "Example: BSP-Yocto-Ampliphy-AM62Ax-PD25.1.0. It ties versions together; do not mix releases."

.. END:table:0:1:0

.. START:section:0:2
PHYTEC BSP Components
=====================

.. END:section:0:2

.. START:main_content:0:2
The exact content depends on the product and release. A PHYTEC Yocto BSP normally delivers or builds the following parts.

.. END:main_content:0:2

.. START:table:0:2:0

.. csv-table:: Table 2. Parts of a PHYTEC BSP
   :header: "Part", "What it does"
   :widths: 25, 75
   :align: center

   "Bootloader", "Runs after the SoC Boot ROM, initialises what the platform needs and loads the kernel and device tree. Current PHYTEC NXP i.MX 8M and TI AM6x releases use U-Boot (see the Bootloader subsection)."
   "Linux kernel", "Manages CPU, memory, processes, networking, filesystems and drivers. PHYTEC publishes its own kernel repositories with board support and patches."
   "Device tree (DTS/DTSI/DTB)", "Describes the board hardware to Linux: peripherals, addresses, interrupts, GPIOs, clocks, regulators and pin control. Typically split into a SoM file and a carrier-board file, plus overlays for options."
   "Kernel configuration", "Selects which drivers and features are built in or modular. Recent NXP releases use kernel config fragments with a PHYTEC defconfig (for example imx8_phytec_defconfig)."
   "Yocto metadata (meta-phytec)", "Layers, recipes, machine and distro configuration that tell BitBake which sources, patches and packages make up the image."
   "Root filesystem and images", "Libraries, tools, services and applications used after Linux starts, delivered as ready-made images such as phytec-headless-image."
   "Pre-built binaries", "Ready-to-flash images and boot files on the PHYTEC download server. Use them first to confirm that your hardware works."

.. END:table:0:2:0

.. START:note:0:2:1

.. note::

   **BSP and Yocto are related but not the same**. The BSP is the hardware support and configuration. Yocto is the build system PHYTEC uses to assemble it into an image.

.. END:note:0:2:1

.. START:section:0:3
Kernel, Bootloader and Yocto Overview
=====================================

.. END:section:0:3

Linux Kernel
------------

The kernel runs after the bootloader hands over control. A driver is the kernel code that controls one kind of hardware block, and the device tree tells that driver what the board actually provides.

.. START:list:0:3:1

.. LIST_META: {"type": "bullet", "head": "Work on the kernel when you need to:", "items": ["Enable or configure a driver that already exists in the kernel source.", "Add a new driver or fix an existing one.", "Change the Linux device tree, usually together with pinctrl settings."]}

.. container:: spx-list-head

    **Work on the kernel when you need to:**

- Enable or configure a driver that already exists in the kernel source.
- Add a new driver or fix an existing one.
- Change the Linux device tree, usually together with pinctrl settings.

.. END:list:0:3:1

**Example:** Linux boots but SPI is missing. Check the active device tree, the pinctrl group, the kernel configuration and the real wiring. A correct driver alone is not enough if the board description is wrong.

Bootloader
----------

The SoC first runs its built-in Boot ROM, which loads the next stage from the configured boot source. The bootloader then prepares the platform and loads the kernel, device tree and any other boot files. Stages differ by SoC and release.

.. START:list:0:3:4

.. LIST_META: {"type": "bullet", "head": "Work on the bootloader when you need to:", "items": ["Change boot order, boot source, environment variables or kernel arguments.", "Diagnose a failure that happens before Linux starts.", "Change how the device tree and overlays are located and passed to Linux."]}

.. container:: spx-list-head

    **Work on the bootloader when you need to:**

- Change boot order, boot source, environment variables or kernel arguments.
- Diagnose a failure that happens before Linux starts.
- Change how the device tree and overlays are located and passed to Linux.

.. END:list:0:3:4

.. START:warning:0:3:5

.. warning::

   | **Which bootloader does my release use? Check, do not assume.**
   | Current PHYTEC BSPs for NXP i.MX 8M and TI AM6x boards use U-Boot, not Barebox. The newer releases also changed how the board boots: kernel, device tree and overlays are stored in a FIT image, and the boot flow uses U-Boot standard boot (PHYTEC calls the AM6x version Ampliphy-Boot). Barebox appears in some older PHYTEC BSPs, so confirm in your BSP manual
   | Any procedure written for Barebox prompt commands will not work on a U-Boot release, and the other way round.

.. END:warning:0:3:5

.. START:danger:0:3:6

.. danger::

   | **Keep a recovery method before changing boot software**
   | Overwriting the bootloader or boot media without a tested recovery path (for example a serial console and an alternate boot source) can leave the board unable to boot. Confirm the recovery method in your BSP manual before you flash anything.

.. END:danger:0:3:6

Examples taken from PHYTEC release notes:

.. START:table:0:3:8

.. csv-table:: Table 3. Bootloader used in example PHYTEC BSP releases
   :header: "BSP release", "Bootloader", "Notable boot change"
   :widths: 35, 35, 30
   :align: center

   "i.MX 8M Plus PD23.1.0 (Dec 2023)", "U-Boot v2021.04_2.2.0-phy5", "Earlier boot flow"
   "i.MX 8M Plus PD24.1.0 (Nov 2024)", "U-Boot v2024.04-2.0.0-phy7", "FIT image, U-Boot standard boot, EFI boot"
   "i.MX 8M Mini PD25.1.0 (Mar 2025)", "U-Boot (standard boot)", "FIT image, imx8_phytec_defconfig"
   "AM62Ax PD25.1.0 (Sep 2025)", "U-Boot v2025.07 (tag v2025.07-phy4)", "Ampliphy-Boot"
   "AM64x PD25.2.x", "U-Boot v2025.07", "Ampliphy-Boot, watchdog started during boot"

.. END:table:0:3:8

Yocto Project
-------------

Yocto/OpenEmbedded runs on a development host (a PHYTEC-tested Ubuntu version or a container). BitBake reads recipes and layers, fetches and patches sources, compiles them, creates packages and assembles the root filesystem and boot files.

.. START:list:0:3:10

.. LIST_META: {"type": "bullet", "head": "Work on Yocto when you need to:", "items": ["Build a complete image for the board.", "Add a package, service or application to every image.", "Make a runtime change (hostname, banner, config file) the permanent default.", "Turn a kernel or bootloader patch into a repeatable, rebuildable change."]}

.. container:: spx-list-head

    **Work on Yocto when you need to:**

- Build a complete image for the board.
- Add a package, service or application to every image.
- Make a runtime change (hostname, banner, config file) the permanent default.
- Turn a kernel or bootloader patch into a repeatable, rebuildable change.

.. END:list:0:3:10

| A **recipe** describes how one piece of software is fetched, patched, built and packaged.
| A **layer** groups related recipes and configuration.
| An **image** recipe selects what goes into the root filesystem. BitBake executes the tasks.

**Example:** you need curl in every production image. Add it through the image configuration and rebuild; do not install it by hand on each device.

.. START:section:0:4
Build Flow and Boot Flow
========================

.. END:section:0:4

.. START:main_content:0:4
Two separate activities are easy to confuse. Yocto **builds** software on your PC. The board **boots** the bootloader, kernel, device tree and root filesystem that the build produced.

.. END:main_content:0:4

.. START:section:0:5
Kernel, Bootloader and Yocto Comparison
=======================================

.. END:section:0:5

.. START:table:0:5:0

.. csv-table:: Table 4. Linux kernel, bootloader and Yocto compared
   :header: "Question", "Linux kernel", "Bootloader (U-Boot)", "Yocto Project"
   :widths: 20, 27, 27, 26
   :align: center

   "Main job", "Runs the OS and controls hardware through drivers.", "Prepares the platform and loads the kernel, device tree and overlays.", "Builds and assembles the complete software image."
   "Where it runs", "On the board, after hand-off.", "On the board, before Linux.", "On the development PC, at build time."
   "Typical source", "Kernel source, defconfig/fragments, DTS/DTSI and patches.", "Bootloader source, board configuration, environment and boot scripts.", "Layers, recipes, machine, distro and image configuration."
   "Typical output", "Kernel image, modules and device-tree binaries (often inside a FIT image).", "Bootloader binaries and boot files.", "Images (.wic, .partup), SDK and packages."
   "Example change", "Enable an SPI controller or edit a device tree.", "Change boot order or how the DTB is loaded.", "Add a package or a default configuration to the image."

.. END:table:0:5:0

.. START:section:0:6
Choosing the Component to Change
================================

.. END:section:0:6

.. START:main_content:0:6
Start from the result you want. Many tasks touch more than one component. The last column points to the guide in this series that covers the task in detail.

.. END:main_content:0:6

.. START:table:0:6:0

.. csv-table:: Table 5. Which component to change for a given requirement
   :header: "Requirement / symptom", "Where to start", "Why", "Covered in"
   :widths: 30, 30, 25, 15
   :align: center

   "Change hostname or login banner on a running board", "Linux runtime configuration", "User-space settings; no rebuild needed.", "BSP-09"
   "Make that change the default in every new image", "Yocto image / recipe", "Must be built in to be repeatable.", "Yocto section"
   "Add a tool or application to the standard image", "Yocto recipe / image", "The package has to be built and included.", "Yocto section"
   "A Linux peripheral is missing or not working", "Device tree, pinctrl, kernel config, hardware documentation", "Pins, routing, driver and wiring must all agree.", "BSP-03, 04, 05"
   "Add or modify a kernel driver", "Kernel source and configuration; Yocto to package it", "The kernel owns drivers.", "BSP-02, 05"
   "Board stops before Linux starts", "Bootloader log, boot source, earlier boot stages", "Linux cannot diagnose a pre-kernel failure.", "BSP-06, 08"
   "Change boot arguments, boot order or boot media", "Bootloader environment and boot configuration", "These control how Linux is located and started.", "BSP-06, 08"
   "Change a pin function for Linux", "Pinout / schematic, pinmux tool, device tree", "The active DTS must match real wiring.", "BSP-04, 10 to 13"
   "Change DDR or other pre-Linux initialisation", "SoC vendor documentation and platform early-boot code", "Done before Linux and is SoC-specific.", "BSP-06, 07"
   "Build a complete image for a board", "Yocto / PHYTEC BSP build workflow", "The build assembles every component.", "Yocto section"
   "Compile a standalone C/C++ program for the board", "PHYTEC SDK / cross toolchain", "Creates a target executable.", "BSP-14"

.. END:table:0:6:0

.. START:section:0:7
PHYTEC Release and Image Naming
===============================

.. END:section:0:7

.. START:main_content:0:7
PHYTEC names tell you exactly what you are looking at. Copy these names exactly into your notes; a one-character difference can mean a different board variant.

.. END:main_content:0:7

.. START:table:0:7:0

.. csv-table:: Table 6. How to read PHYTEC release, machine, image and distro names
   :header: "Name type", "Example", "What it tells you"
   :widths: 20, 35, 45
   :align: center

   "BSP release", "BSP-Yocto-Ampliphy-AM62x-PD25.1.0", "Yocto-based BSP, Ampliphy distribution, AM62x SoM family and release PD25.1.0. Naming differs by family and release (for example, BSP-Yocto-NXP-i.MX8MP-PD24.1.0 and BSP-Yocto-Ampliphy-i.MX8MP-PD24.1.1)."
   "Yocto MACHINE", "phyboard-pollux-imx8mp-3; phyboard-lyra-am62axx-2", "The exact kit or board variant to build for. The number suffix is part of the name; check the BSP manual's machine table for your SoM and carrier-board revision."
   "Image", "phytec-headless-image; phytec-qt6demo-image; phytec-securiphy-image", "Defines the packages and applications included in the root filesystem, such as a console-only system, Qt6 demo or security-focused image."
   "Distro", "ampliphy; ampliphy-rt; ampliphy-rauc; ampliphy-linux-mainline", "The distribution configuration, for example real-time support, RAUC updates or tracking mainline kernel tags. Available distros vary by SoM."
   "Kernel / U-Boot tags", "v6.12.35-11.01.05-phy5; v2025.07-phy4", "Identify the base software version and PHYTEC revision (-phyN). The kernel tag may include a vendor SDK release identifier."

.. END:table:0:7:0

.. START:section:0:8
BSP Selection Procedure
=======================

.. END:section:0:8

.. START:main_content:0:8
Follow this order before you clone a repository or run a build command. BSP manuals are tied to specific releases.

.. END:main_content:0:8

.. START:list:0:8:0

.. LIST_META: {"type": "auto", "head": "", "items": ["**Identify the exact hardware.** Write down the SoM and carrier-board product names or numbers, PCB revisions, SoC, RAM and storage. A family name is not enough.", "**List the interfaces you need.** For example Ethernet, CAN, I²C, SPI, USB, display, camera, audio, Wi-Fi/Bluetooth, graphics or AI acceleration. Confirm the hardware really routes each one.", "**Open the official product page and download page.** Find the Quick Start Guide, hardware manual, BSP manual and release notes for that exact product.", "**Pick a release that explicitly supports your SoM and carrier combination.** Prefer the latest released, supported BSP unless you have a reason to stay on an older line. Record its full name and date.", "**Record the version set.** Kernel, bootloader, Yocto release, MACHINE, image, distro and repository tags. Do not mix instructions from different releases.", "**Confirm the boot and storage details.** Boot source (eMMC, SD, SPI NOR, NAND), bootloader, flash layout and the recovery method.", "**Test with the pre-built image first.** Flash the ready-made image and check that your hardware boots. Only then start customising. This separates hardware faults from your own changes.", "**Check each interface end to end.** Schematic and hardware manual, then pinmux, device tree, kernel driver and any firmware or packages.", "**Prepare the documented build environment.** Use the Yocto and development-environment guide for that release, including the tested host OS.", "**Save the selection record.** Keep it with your project so the work is reproducible."]}

1. **Identify the exact hardware.** Write down the SoM and carrier-board product names or numbers, PCB revisions, SoC, RAM and storage. A family name is not enough.
2. **List the interfaces you need.** For example Ethernet, CAN, I²C, SPI, USB, display, camera, audio, Wi-Fi/Bluetooth, graphics or AI acceleration. Confirm the hardware really routes each one.
3. **Open the official product page and download page.** Find the Quick Start Guide, hardware manual, BSP manual and release notes for that exact product.
4. **Pick a release that explicitly supports your SoM and carrier combination.** Prefer the latest released, supported BSP unless you have a reason to stay on an older line. Record its full name and date.
5. **Record the version set.** Kernel, bootloader, Yocto release, MACHINE, image, distro and repository tags. Do not mix instructions from different releases.
6. **Confirm the boot and storage details.** Boot source (eMMC, SD, SPI NOR, NAND), bootloader, flash layout and the recovery method.
7. **Test with the pre-built image first.** Flash the ready-made image and check that your hardware boots. Only then start customising. This separates hardware faults from your own changes.
8. **Check each interface end to end.** Schematic and hardware manual, then pinmux, device tree, kernel driver and any firmware or packages.
9. **Prepare the documented build environment.** Use the Yocto and development-environment guide for that release, including the tested host OS.
10. **Save the selection record.** Keep it with your project so the work is reproducible.

.. END:list:0:8:0

.. START:tip:0:8:1

.. tip::

   | **Test the pre-built image first**
   | A working pre-built image proves that the hardware, power and boot path are fine. If your own build then fails, you know the cause is in your changes and not in the board.

.. END:tip:0:8:1

Example of a complete version record, using values from PHYTEC release notes:

.. START:table:0:8:3

.. csv-table:: Table 7. Example BSP version record
   :header: "BSP release", "Linux kernel", "Bootloader", "Yocto"
   :widths: 30, 25, 25, 20
   :align: center

   "AM62Ax PD25.1.0 (2025-09-26)", "v6.12.35-11.01.05 (tag -phy5)", "U-Boot v2025.07 (tag v2025.07-phy4)", "5.0.12 Scarthgap"
   "AM64x PD25.2.1", "v6.12.79 (tag v6.12.79-phy1)", "U-Boot v2025.07", "5.0.16 Scarthgap"
   "i.MX 8M Plus PD24.1.0 (2024-11-06)", "v6.6.23-2.0.0-phy10", "U-Boot v2024.04-2.0.0-phy7", "See release notes"

.. END:table:0:8:3

.. START:warning:0:8:4

.. warning::

   | **Do not mix releases**
   | A kernel, bootloader and Yocto layer taken from different BSP releases may fail to boot or may break peripherals. Use the tags that are published together in one release.

.. END:warning:0:8:4

PHYTEC states that its AM62x PD25.1.0 release is largely based on TI Processor SDK Linux v11.01.05.

.. START:section:0:9
NXP and TI Platform Comparison
==============================

.. END:section:0:9

.. START:main_content:0:9
NXP and TI are SoC vendors. PHYTEC builds modules and boards around their processors, so Linux concepts are shared while the vendor details differ.

.. END:main_content:0:9

.. START:table:0:9:0

.. csv-table:: Table 8. Common ideas and differences between NXP and TI platforms
   :header: "Area", "Common idea", "What differs between NXP and TI"
   :widths: 20, 35, 45
   :align: center

   "Kernel", "Same Linux architecture: drivers, subsystems, configuration and logs.", "Different driver sets and vendor patches. Kernel tags show the base: NXP-style (6.6.23-2.0.0) versus TI SDK-style (6.12.35-11.01.05)."
   "Device tree", "DTS/DTSI describes the board; the DTB is used at boot.", "Pinctrl syntax, node names, clock/reset/power bindings differ. For example, TI AM62 uses k3-am62-phycore-som.dtsi."
   "Bootloader", "Loads the OS and passes boot data to Linux.", "Boot ROM behaviour, early stages, boot media and configuration are SoC-specific. Both use U-Boot in current PHYTEC releases."
   "BSP build", "Yocto/BitBake builds the image.", "Machine configuration, layers, and kernel/bootloader versions are release-specific."
   "Pinmux tool", "Tools help select and check pin functions.", "Different tools are used by each vendor, and their output formats differ (see the Pinmux Tools Overview section)."

.. END:table:0:9:0

.. START:section:0:10
Pinmux Tools Overview
=====================

.. END:section:0:10

.. START:main_content:0:10
Pin multiplexing (pinmux) decides which internal SoC function is connected to a physical pin. One pin can offer UART, SPI, I²C or GPIO, but normally only one at a time. Electrical settings such as pull-ups and drive strength are related but separate from choosing the function.

.. END:main_content:0:10

.. START:table:0:10:0

.. csv-table:: Table 9. Pinmux tools for NXP, TI and PHYTEC boards
   :header: "Tool", "Main purpose", "How to use it in a PHYTEC Linux BSP"
   :widths: 25, 35, 40
   :align: center

   "NXP Config Tools for i.MX (Pins tool)", "Pin routing and electrical settings for supported i.MX processors. Other tools in the suite cover areas such as clocks and DDR. [11]", "Select the exact SoC and package, assign signals, check conflicts and export. Compare the result with the existing board device tree and merge only the required pinctrl settings. The export is not a complete device tree. (Guide: BSP-10)"
   "TI SysConfig", "Graphical pinmux and peripheral configuration for supported TI devices and SDKs. [12]", "Select the exact device and context. Outputs depend on the SDK and may not be Linux device-tree source, so follow the TI and PHYTEC instructions for your SoC. (Guide: BSP-11)"
   "PHYTEC Pinmuxing Tool (pinmux.phytec.com)", "Web tool for PHYTEC products. Select a SoM and carrier board, choose interfaces, and view the result as a connector pinout or physical pinout. Selections can be saved and loaded. [13]", "Use it to identify which SoM and connector pins an interface uses on your board. Cross-check with the hardware manual and schematic, then make the matching change in the device tree. (Guide: BSP-12)"

.. END:table:0:10:0

.. START:tip:0:10:1

.. tip::

   | **Suggested order of use**
   | Use the PHYTEC Pinmuxing Tool first to see what is on your real SoM and carrier board. Then use the NXP or TI tool for SoC-level details such as electrical settings.

.. END:tip:0:10:1

.. START:important:0:10:2

.. important::

   | **Practical rule**
   | A pinmux tool helps you find or generate a configuration. It does not prove that the signal is wired to the connector, that the driver is enabled, or that your change is in the device tree the board actually boots. Verify all three on the real board.

.. END:important:0:10:2

.. START:section:0:11
PHYTEC Documentation Map
========================

.. END:section:0:11

.. START:main_content:0:11
PHYTEC publishes manuals in more than one place. Start from the page of your exact product and use the manual that matches your BSP release.

.. END:main_content:0:11

.. START:table:0:11:0

.. csv-table:: Table 10. PHYTEC documents and where to use them
   :header: "Document / site", "What it answers", "When to use it"
   :widths: 25, 40, 35
   :align: center

   "Quick Start Guide", "How to connect, power, boot and try the board.", "First use of the kit, or checking that the default image boots."
   "Hardware manual", "SoM and carrier-board design, connectors, interfaces, electrical details and pinout tables.", "Before changing pinmux, device tree, power or a hardware interface."
   "BSP manual", "Release-specific workflow for booting, flashing, device tree, peripherals and building. TI-based manuals at docs.phytec.com are organised into Quickstart, Interface Guides, Booting Essentials, Secure Boot, Application Development, Building the BSP and Pre-Built Binaries.", "Daily BSP work and platform-specific commands."
   "Yocto reference manual", "Yocto and BitBake setup, layers, recipes and image customisation for PHYTEC.", "Building and customising images."
   "Release notes", "New features, kernel and bootloader versions, supported boot sources and machines.", "Choosing a release or before upgrading."
   "Download server", "Pre-built images, boot tools and release notes for each BSP release.", "First hardware test and flashing known-good images."

.. END:table:0:11:0

.. START:section:0:12
Example: Enabling an SPI Interface
==================================

.. END:section:0:12

.. START:main_content:0:12
**Requirement:** an application needs an SPI device on a connector of a PHYTEC board. This is the high-level workflow; exact files and commands come from your BSP manual.

.. END:main_content:0:12

.. START:danger:0:12:0

.. danger::

   | **Check the voltage before connecting hardware**
   | Check the I/O voltage of the pins in the hardware manual before you connect an external device. A higher voltage than the pin supports can permanently damage the SoM or carrier board. Power the board off before changing any wiring.

.. END:danger:0:12:0

.. START:list:0:12:1

.. LIST_META: {"type": "auto", "head": "", "items": ["**Confirm the physical connection.** Use the hardware manual, schematic or the PHYTEC Pinmuxing Tool to find the SoM pins, connector pins and voltage level.", "**Check the SoC pin options.** Use the SoC reference manual and the NXP, TI or PHYTEC pinmux reference to find an SPI function on the routed pins.", "**Check for conflicts.** The pins must not be needed by another interface or reserved by the board.", "**Update the device tree.** In the board DTS/DTSI (kernel path usually arch/arm64/boot/dts/freescale/ for NXP or .../ti/ for TI), add the pinctrl group and set the SPI controller node to enabled. Add the child device if its binding requires one.", "**Check driver support.** Confirm the controller driver and the device driver are enabled in the kernel configuration. A device tree describes hardware; it does not create a missing driver.", "**Build with the matching BSP workflow.** Rebuild only what changed (kernel, device tree or FIT image) using the documented Yocto steps.", "**Deploy and test.** Update the correct artifact, check boot and kernel logs, confirm the device node, then test real communication with the connected device."]}

1. **Confirm the physical connection.** Use the hardware manual, schematic or the PHYTEC Pinmuxing Tool to find the SoM pins, connector pins and voltage level.
2. **Check the SoC pin options.** Use the SoC reference manual and the NXP, TI or PHYTEC pinmux reference to find an SPI function on the routed pins.
3. **Check for conflicts.** The pins must not be needed by another interface or reserved by the board.
4. **Update the device tree.** In the board DTS/DTSI (kernel path usually arch/arm64/boot/dts/freescale/ for NXP or .../ti/ for TI), add the pinctrl group and set the SPI controller node to enabled. Add the child device if its binding requires one.
5. **Check driver support.** Confirm the controller driver and the device driver are enabled in the kernel configuration. A device tree describes hardware; it does not create a missing driver.
6. **Build with the matching BSP workflow.** Rebuild only what changed (kernel, device tree or FIT image) using the documented Yocto steps.
7. **Deploy and test.** Update the correct artifact, check boot and kernel logs, confirm the device node, then test real communication with the connected device.

.. END:list:0:12:1

.. START:section:0:13
Common Mistakes
===============

.. END:section:0:13

.. START:table:0:13:0

.. csv-table:: Table 11. Common mistakes when selecting and using a BSP
   :header: "Mistake", "What to do instead"
   :widths: 45, 55
   :align: center

   "Following a Barebox guide on a U-Boot release (or the reverse).", "Check the bootloader in your BSP release notes and manual first."
   "Mixing a kernel, bootloader or Yocto layer from different BSP releases.", "Use the tags listed together in one release."
   "Choosing a BSP by kit name only.", "Record SoM and carrier-board product numbers and PCB revisions."
   "Starting changes before testing the pre-built image.", "Flash and boot the pre-built image first to rule out hardware problems."
   "Pasting pinmux tool output straight into a device tree.", "Merge only the required pinctrl entries and compare them with the existing board files."
   "Editing the wrong device tree file or the wrong DTB.", "Identify which DTB the boot flow loads (BSP-03, BSP-07)."
   "Fixing the running filesystem by hand for a product.", "Prove the change at runtime, then make it permanent through Yocto."

.. END:table:0:13:0

.. START:section:0:14
BSP Selection Checklist
=======================

.. END:section:0:14

.. START:list:0:14:0

.. LIST_META: {"type": "bullet", "head": "", "items": ["SoM product number and PCB revision, and carrier product number and PCB revision, recorded.", "SoC family identified; required interfaces confirmed in the hardware manual and schematic.", "BSP release chosen; kernel, bootloader, Yocto, MACHINE, image and distro recorded.", "Bootloader type confirmed (U-Boot or Barebox) from the release documentation.", "Boot source, storage type, flash layout and recovery method confirmed.", "Pre-built image flashed and the board boots.", "Pinmux, device tree, driver and firmware support checked for each required interface.", "Documented build host and matching PHYTEC SDK/toolchain identified.", "Repository names, branches, tags and local patches recorded."]}

- SoM product number and PCB revision, and carrier product number and PCB revision, recorded.
- SoC family identified; required interfaces confirmed in the hardware manual and schematic.
- BSP release chosen; kernel, bootloader, Yocto, MACHINE, image and distro recorded.
- Bootloader type confirmed (U-Boot or Barebox) from the release documentation.
- Boot source, storage type, flash layout and recovery method confirmed.
- Pre-built image flashed and the board boots.
- Pinmux, device tree, driver and firmware support checked for each required interface.
- Documented build host and matching PHYTEC SDK/toolchain identified.
- Repository names, branches, tags and local patches recorded.

.. END:list:0:14:0

.. START:section:0:15
Glossary
========

.. END:section:0:15

.. START:table:0:15:0

.. csv-table:: Table 12. Glossary
   :header: "Term", "Meaning"
   :widths: 25, 75
   :align: center

   "BSP", "Board Support Package: board-specific software and configuration for running an OS on hardware."
   "DTS / DTSI / DTB", "Device Tree Source, Device Tree Source Include, and compiled Device Tree Binary."
   "FIT image", "A container that holds the kernel, device tree and overlays in one signed or verifiable file; used by newer PHYTEC releases."
   "Standard boot", "U-Boot's common mechanism for finding and starting an OS; the basis of PHYTEC's Ampliphy-Boot."
   "Driver", "Kernel software that controls a hardware device."
   "Kernel configuration", "Options that decide which features and drivers are built in or available as modules."
   "BitBake / Recipe / Layer", "Task executor / build description for one component / group of related build metadata."
   "MACHINE / Distro / Image", "Hardware variant to build for / distribution policy / contents of the root filesystem."
   "Root filesystem", "Programs, libraries, configuration files and services Linux uses after it starts."
   "SDK / toolchain", "Cross-compiler and target libraries for building software for the board."
   "Pinmux / pinctrl", "Choosing the function of a pad / the Linux framework that applies pin settings."
   "Bootargs", "Parameters the bootloader passes to the kernel, such as the console and root device."

.. END:table:0:15:0

.. START:section:0:16
Conclusion
==========

.. END:section:0:16

.. START:main_content:0:16
| Choosing the right PHYTEC BSP starts with your exact hardware: the SoM, the carrier board and the interfaces you need. Pick a release that supports that combination, and record the kernel, bootloader, Yocto, MACHINE, image and distro versions together. Never mix components from different releases.
| Always check which bootloader your release uses. Current PHYTEC BSPs for NXP i.MX 8M and TI AM6x use U-Boot, and Barebox appears only in some older BSPs. Before you change anything, flash the pre-built image to confirm the board and boot path work.
| Before you make a change, decide where it belongs:

.. END:main_content:0:16

.. START:list:0:16:0

.. LIST_META: {"type": "bullet", "head": "", "items": ["**Kernel or device tree:** drivers, pinctrl and hardware description.", "**Bootloader:** boot order, boot media and boot arguments.", "**Yocto:** permanent, repeatable image content.", "**Running system:** quick runtime changes that don’t need a rebuild."]}

- **Kernel or device tree:** drivers, pinctrl and hardware description. 
- **Bootloader:** boot order, boot media and boot arguments. 
- **Yocto:** permanent, repeatable image content. 
- **Running system:** quick runtime changes that don’t need a rebuild.

.. END:list:0:16:0

The NXP, TI and PHYTEC pinmux tools help you find or generate pin settings. They do not prove the wiring, the driver or the active device tree, so verify those on the real board.
