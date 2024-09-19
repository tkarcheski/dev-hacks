### Device Trees in Zephyr RTOS

#### Overview
A device tree in Zephyr RTOS is a data structure that describes the hardware components of a system, including processors, memory, buses, and peripherals. It is used by the OS to understand the hardware layout and configure drivers accordingly.

#### Key Concepts

1. **Device Tree Source (DTS):**
   - A human-readable file that describes the hardware layout. This is typically found in files with `.dts` or `.dtsi` extensions.
   - For example:
     ```dts
     / {
         model = "My Custom Board";
         compatible = "my,board";

         memory@20000000 {
             device_type = "memory";
             reg = <0x20000000 0x40000>;
         };

         gpio@40020000 {
             compatible = "nordic,nrf-gpio";
             reg = <0x40020000 0x300>;
             label = "GPIO_0";
         };
     };
     ```

2. **Device Tree Binding:**
   - Defines how a particular hardware component is represented in the device tree. Bindings describe the properties that can be associated with nodes in the DTS.

3. **Macros for Device Tree Access:**
   - Zephyr provides macros to access properties in the device tree. These macros are usually defined in `dts.h` and related files.

   **Common Macros:**
   - `DT_NODELABEL(label)`: Gets a node by its label.
   - `DT_ALIAS(alias)`: Gets a node by its alias.
   - `DT_INST(inst, comp)`: Gets an instance of a device.
   - `DT_PROP(node_id, prop)`: Gets a property from a node.

   **Example Usage:**
   ```c
   #include <zephyr.h>
   #include <device.h>
   #include <drivers/gpio.h>

   #define LED0_NODE DT_ALIAS(led0)

   static const struct gpio_dt_spec led = GPIO_DT_SPEC_GET(LED0_NODE, gpios);

   void main(void) {
       gpio_pin_configure_dt(&led, GPIO_OUTPUT_ACTIVE);
       gpio_pin_set_dt(&led, 1); // Turn on the LED
   }
   ```

### Debug Prints in Zephyr RTOS

#### Overview
Debug prints in Zephyr are handled primarily through the logging subsystem, which provides flexible and efficient logging at various levels.

#### Configuring Debug Prints

1. **Enabling Logging:**
   - Logging is configured using Kconfig options. Make sure the `CONFIG_LOG` and `CONFIG_LOG_PRINTK` options are enabled in your `prj.conf` file.
     ```plaintext
     CONFIG_LOG=y
     CONFIG_LOG_PRINTK=y
     CONFIG_LOG_DEFAULT_LEVEL=4  # Set the default log level (0-4)
     ```

2. **Using the Logging API:**
   - Zephyr provides macros to log messages at different levels. These are:
     - `LOG_ERR(...)`: Error messages.
     - `LOG_WRN(...)`: Warnings.
     - `LOG_INF(...)`: Informational messages.
     - `LOG_DBG(...)`: Debug messages.

   **Example:**
   ```c
   #include <logging/log.h>
   LOG_MODULE_REGISTER(main_module, LOG_LEVEL_DBG);

   void main(void) {
       LOG_INF("Starting application");
       LOG_DBG("This is a debug message");
   }
   ```

3. **Adding Debug Prints to Other Modules:**
   - To add debug prints to other modules, follow these steps:
     1. **Identify the Module:**
        - Determine where you want to add debug prints. This could be a driver, a custom application module, or any other part of the Zephyr codebase.
     2. **Register the Module with Logging:**
        - Use `LOG_MODULE_REGISTER` at the top of your file:
          ```c
          LOG_MODULE_REGISTER(my_module, LOG_LEVEL_DBG);
          ```
     3. **Add Log Messages:**
        - Insert log macros (`LOG_INF`, `LOG_DBG`, etc.) where needed in the code.
     4. **Ensure Logging is Enabled:**
        - Confirm that the appropriate log level is set in `prj.conf` or via Kconfig for your module.
        - If you’re adding logs to a driver, ensure that `CONFIG_<DRIVER>_LOG_LEVEL_DBG=y` is set, where `<DRIVER>` corresponds to the name of the driver in Kconfig.

4. **Log Output:**
   - By default, log messages are printed to the console. You can redirect them to other outputs like UART or over a network, depending on your system’s configuration.

5. **Filtering Logs:**
   - Zephyr allows you to filter logs at runtime using shell commands or by setting the log level for specific modules.
     ```plaintext
     shell> log level set <module> <level>
     ```

### Example: Adding Debug Prints to a GPIO Driver

Suppose you want to add additional debug prints to the GPIO driver:

1. **Identify the Source File:**
   - Locate the source file for the GPIO driver. For instance, it might be in `drivers/gpio/gpio_nrf.c`.

2. **Register a Logging Module:**
   - At the top of the file, register the module:
     ```c
     LOG_MODULE_REGISTER(gpio_nrf, LOG_LEVEL_DBG);
     ```

3. **Add Debug Prints:**
   - Add debug prints in the functions where you want to trace execution:
     ```c
     static int gpio_nrf_configure(const struct device *dev,
                                   gpio_pin_t pin, gpio_flags_t flags) {
         LOG_DBG("Configuring pin %d with flags 0x%x", pin, flags);
         // Existing code...
     }
     ```

4. **Build and Flash:**
   - Rebuild your application with `west build` and flash it to the device. The new debug prints should appear in the console.

### Conclusion

- **Device Trees:** Central to configuring hardware in Zephyr, accessed via macros.
- **Debug Prints:** Handled by the logging subsystem, configurable via Kconfig and the logging API.
- **Adding Debug Prints:** Involves registering a logging module and using logging macros at the desired log level.

With this understanding, you can effectively debug and monitor your Zephyr applications by leveraging device trees and the robust logging infrastructure.
