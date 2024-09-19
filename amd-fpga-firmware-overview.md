Notice: current information as of October 2023.

**Versal Prime FPGA Firmware** 

---

### Updated Guide: Versal Prime FPGA Firmware Development

---

#### Overview

The Versal Prime FPGA, part of AMD Xilinx's Versal ACAP (Adaptive Compute Acceleration Platform) family, integrates scalar processing, adaptable hardware, and intelligent engines. Developing firmware for this platform involves understanding its boot process, setting up the appropriate development environment, and leveraging advanced tools for firmware customization and optimization.

### Detailed Boot Process

The boot process of the Versal Prime FPGA is intricate, involving several stages to initialize and configure the system properly:

1. **Power-On Reset (POR):**
   - **Platform Management Controller (PMC):** The PMC is a dedicated subsystem responsible for managing system power, boot, and configuration. It initializes the hardware platform, configures power supplies, and resets the system components, including the processing system (PS) and programmable logic (PL).
   - **Reset and Configuration Unit (RCU):** Part of the PMC, the RCU handles system resets and boot mode selection, determining the source from which the FPGA will boot (e.g., QSPI flash, SD card, eMMC).

2. **BootROM Execution:**
   - The **BootROM** is embedded in the Versal device and executes immediately after reset. It performs initial system checks and reads the Boot Header from the primary boot device.
   - **Key Functions:**
     - Validates the boot image.
     - Loads and authenticates the **Platform Loader and Manager (PLM)**.

3. **Platform Loader and Manager (PLM):**
   - The PLM is a critical firmware component that runs on the PMC's MicroBlaze™ processor. It manages the system's boot and configuration processes.
   - **Responsibilities:**
     - Initializes the PL and PS subsystems.
     - Loads and authenticates additional firmware and user applications.
     - Manages system resources, security, and error handling.

4. **Loading Secondary Boot Components:**
   - The PLM may load secondary boot components such as the **ARM Trusted Firmware (ATF)**, **U-Boot bootloader**, and user applications.
   - **ARM Trusted Firmware (ATF):** Provides a secure environment for the execution of trusted code and manages the transition to non-secure world software.

5. **Operating System Boot:**
   - **U-Boot Bootloader:** Initializes system peripherals and loads the operating system kernel (e.g., Linux) and device tree blobs into memory.
   - **Kernel Initialization:** The OS kernel takes over, initializes drivers, mounts the root filesystem, and starts user-space applications.

#### Updated Boot Flow Diagram
```plaintext
[PMC & RCU] → [BootROM] → [PLM] → [ATF] → [U-Boot] → [Operating System]
```

### Setting Up the Development Environment

To effectively develop firmware for the Versal Prime FPGA, you need to set up a robust development environment using the latest tools provided by AMD Xilinx:

#### 1. Vivado Design Suite

**Vivado** is the primary tool for hardware design on Xilinx FPGAs.

- **Key Features:**
  - **Synthesis and Implementation:** Transforms HDL code into a bitstream for configuring the PL.
  - **Hardware Debugging:** Provides tools like the Integrated Logic Analyzer (ILA) for debugging.

- **Installation:**
  - Download the latest version from the [AMD Xilinx Downloads](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools.html).
  - Ensure that system requirements are met and necessary dependencies are installed.

  ```bash
  sudo apt-get update
  sudo apt-get install build-essential libssl-dev libncurses5-dev flex bison libselinux1 gnupg
  ```

- **Basic Usage:**
  - Launch Vivado:
    ```bash
    vivado &
    ```
  - Common Tcl Commands:
    - **Create a project:**
      ```tcl
      create_project <project_name> <path> -part <device_part_number>
      ```
    - **Synthesize Design:**
      ```tcl
      synth_design
      ```
    - **Implement Design:**
      ```tcl
      implement_design
      ```
    - **Generate Bitstream:**
      ```tcl
      write_bitstream
      ```

#### 2. Vitis Unified Software Platform

**Vitis** is a unified software platform that replaces the SDK and provides development tools for embedded software and accelerated applications.

- **Key Features:**
  - **Embedded Software Development:** Supports application development for the ARM Cortex-A72 and Cortex-R5F processors.
  - **Acceleration Development:** Enables the creation of hardware-accelerated applications using high-level programming languages.

- **Installation:**
  - Download from the [AMD Xilinx Vitis Downloads](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vitis.html).
  - Source the settings script:
    ```bash
    source /opt/Xilinx/Vitis/<version>/settings64.sh
    ```

- **Basic Usage:**
  - Launch Vitis:
    ```bash
    vitis &
    ```
  - **Create a Platform Project:**
    - Import the hardware design (XSA file) exported from Vivado.
  - **Develop Applications:**
    - Create application projects targeting the processors in the Versal device.

#### 3. PetaLinux Tools

**PetaLinux** tools simplify the development and deployment of Linux-based systems on Xilinx hardware.

- **Key Features:**
  - **Customizable Linux Kernel and Root Filesystem:**
    - Configure and build a tailored Linux system.
  - **Integration with Vitis and Vivado:**
    - Utilize hardware descriptions and software platforms from other tools.

- **Installation:**
  - Download the latest PetaLinux tools from the [AMD Xilinx PetaLinux Downloads](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html).
  - Ensure that the host system meets all prerequisites.

- **Basic Usage:**
  - Set up the environment:
    ```bash
    source /opt/pkg/petalinux/<version>/settings.sh
    ```
  - Create a new project:
    ```bash
    petalinux-create --type project --template versal --name <project_name>
    ```
  - Import hardware description:
    ```bash
    petalinux-config --get-hw-description=<path_to_xsa_file>
    ```
  - Build the project:
    ```bash
    petalinux-build
    ```
  - Package the boot images:
    ```bash
    petalinux-package --boot --format BIN --fsbl <path_to_fsbl> --u-boot
    ```

#### 4. U-Boot Bootloader and ARM Trusted Firmware (ATF)

- **U-Boot:**
  - **Configuration and Building:**
    ```bash
    cd <project_dir>/build/tmp/work/<machine>/u-boot-xlnx
    make distclean
    make xilinx_versal_virt_defconfig
    make
    ```
- **ARM Trusted Firmware:**
  - Provides a reference implementation of secure world software for ARM Cortex-A processors.
  - **Building ATF:**
    ```bash
    cd <project_dir>/build/tmp/work/<machine>/trusted-firmware-a
    make PLAT=versal RESET_TO_BL31=1 bl31
    ```

### Advanced Boot Customizations

1. **Boot Mode Configuration:**
   - Boot modes can be configured to determine the primary boot source (e.g., JTAG, QSPI, SD, eMMC).
   - **Boot Mode Pins:** The Versal hardware uses boot mode pins to select the boot source at power-on.

2. **Customizing the PLM:**
   - The PLM can be customized using the **Versal Custom Boot and Configuration Design Flow**.
   - **Xilinx GitHub Repository:**
     - AMD Xilinx provides reference designs and PLM customization examples on their [GitHub](https://github.com/Xilinx).
   - **Building Custom PLM:**
     - Use the Vitis software platform to modify and build the PLM code.

3. **Secure Boot Implementation:**
   - Implementing Secure Boot involves:
     - **Boot Image Signing:** Using RSA authentication to sign boot images.
     - **Encryption:** AES encryption for boot image confidentiality.
   - **Tools:**
     - **Bootgen:** A utility to create boot images with authentication and encryption.

### Firmware Security and Management

- **Secure Boot Process:**
  - **BootROM and PLM Authentication:** The BootROM authenticates the PLM, and the PLM authenticates subsequent boot components.
  - **TrustZone Technology:** Utilizes ARM TrustZone to separate secure and non-secure resources.

- **Dynamic Function eXchange (DFX):**
  - Allows partial reconfiguration of the PL at runtime.
  - **Benefits:**
    - Modify FPGA functionality without halting system operations.
    - Optimize resource utilization.

- **System Monitoring and Management:**
  - **System Monitor (SYSMON):** Monitors on-chip temperatures, voltages, and other parameters.
  - **Power Management Framework:** PLM manages power states of various subsystems to optimize power consumption.

### Best Practices

- **Version Control:**
  - Use Git or another version control system to manage your source code and project files.

- **Continuous Integration/Continuous Deployment (CI/CD):**
  - Automate builds and testing using tools like Jenkins or GitLab CI.

- **Documentation:**
  - Keep thorough documentation of your system architecture, configurations, and customizations.

- **Stay Updated:**
  - Regularly check for updates to tools and libraries.
  - Subscribe to AMD Xilinx newsletters and follow their forums for the latest information.

### Additional Resources

- **AMD Xilinx Documentation:**
  - [Versal ACAP Technical Reference Manual](https://docs.xilinx.com/r/en-US/am011-versal-acap-trm)
  - [PetaLinux Tools Documentation](https://docs.xilinx.com/r/en-US/ug1144-petalinux-tools-reference-guide)
  - [Vitis Unified Software Platform Documentation](https://docs.xilinx.com/r/en-US/ug1416-vitis-documentation)

- **Online Communities and Support:**
  - [AMD Xilinx Community Forums](https://support.xilinx.com/s/)
  - [GitHub Repositories](https://github.com/Xilinx) for example projects and reference designs.
  - [Stack Overflow](https://stackoverflow.com/questions/tagged/xilinx) for community-driven Q&A.

- **Training and Tutorials:**
  - [AMD Xilinx Developer Site](https://developer.xilinx.com/) for tutorials and workshops.
  - [Embedded System Design Tutorials](https://docs.xilinx.com/r/en-US/ug1209-embedded-system-design-tutorials)

---

This updated guide provides a comprehensive roadmap for developing firmware on the Versal Prime FPGA using the latest tools and methodologies. By following these guidelines and leveraging the provided resources, you can effectively design, implement, and manage complex firmware projects on the Versal platform.

Please ensure to consult the official AMD Xilinx documentation for the most accurate and detailed information, as tools and processes may evolve over time.
