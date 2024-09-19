If the Dediprog software on GitHub does not currently support libusb, and it's necessary for your setup, you'll need to handle USB communication in another way or wait for libusb support to be added by the developers. However, if the software provides a way to communicate with the device using alternative libraries or methods, you should follow those instructions. Here's what you can generally do in such a scenario:

### 1. Check for Alternative Dependencies

Review the README or any documentation included with the GitHub repository to identify if alternative USB libraries or methods are required (e.g., `libftdi`).

### 2. Install General Dependencies

Make sure your system has the necessary build tools:

```bash
sudo apt update
sudo apt install build-essential git
```

Install any specified USB communication libraries that might be mentioned:

```bash
sudo apt install libftdi1-dev  # Example for libftdi
```

### 3. Clone and Review the Repository

Clone the repository to your local machine:

```bash
git clone https://github.com/DediProgSW/SF100Linux.git
cd SF100Linux
```

Again, adjust the URL to match the actual repository.

### 4. Follow Specific Build Instructions

Projects often require specific commands for building. If it's a C or C++ project without libusb, they might use another library like `libftdi`. The commands might look like this:

```bash
make
sudo make install
```

### 5. Configure Device Permissions

Regardless of the USB library used, you will still likely need to configure udev rules to allow device access:

```bash
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="0483", ATTR{idProduct}=="a140", MODE="0666"' | sudo tee /etc/udev/rules.d/99-dediprog.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Make sure to replace the vendor and product IDs with those appropriate for your device.

### 6. Execute the Software

Run the software according to the provided documentation. This might be a command-line tool or a GUI-based application:

```bash
./dpinst  # As an example
```

### Summary

- Install necessary build tools and any alternative USB libraries specified.
- Clone the GitHub repository and follow the build instructions provided there.
- Set up udev rules to facilitate device access.
- Run the software using the provided executables or scripts.

If there are still issues with USB communication due to the lack of libusb support and no alternative methods are provided, you may need to contact the Dediprog support or the repository maintainers for further guidance or to request the addition of libusb support.
