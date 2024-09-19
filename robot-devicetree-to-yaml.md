**Integrating the Device Tree Project with a Parser to Input Variables into Robot Framework**

---

**Introduction**

Integrating your device tree project with Robot Framework involves creating a parser that can read the Device Tree Source (DTS) files, extract relevant hardware configuration variables, and make them available for use in your Robot Framework test cases. This allows your tests to dynamically adapt based on the hardware descriptions, leading to more robust and flexible testing.

---

### **Overview**

1. **Understand the Device Tree Files**
2. **Create a Parser for DTS Files**
3. **Extract Variables from the Device Tree**
4. **Make Variables Available to Robot Framework**
5. **Integrate the Parser into the Build Process**
6. **Use the Variables in Robot Framework Tests**
7. **Example Implementation**

---

### **1. Understand the Device Tree Files**

**Device Tree Source (DTS) Format**

- **Hierarchical Structure**: DTS files represent hardware components in a tree-like hierarchy.
- **Nodes and Properties**: Each node can have properties (key-value pairs) and child nodes.
- **Syntax**: The syntax resembles C language structure declarations.

**Variables to Extract**

- **Hardware Specifications**: Memory addresses, CPU types, peripheral configurations.
- **Custom Properties**: Any additional properties specific to your hardware that are relevant for testing.

---

### **2. Create a Parser for DTS Files**

**Options for Parsing DTS Files**

- **Direct Parsing of DTS Files**: Write a custom parser to read and interpret the DTS syntax.
- **Convert DTS to an Easier-to-Parse Format**:
  - **Using `dtc` (Device Tree Compiler)**:
    - Convert DTS to Device Tree Blob (DTB).
    - Decompile DTB to readable formats like JSON or YAML if supported.
  - **Using Third-Party Tools**:
    - Tools like `dtxdiff` or `pyfdtdump` can parse DTBs into dictionaries.

**Recommended Approach**

- **Use `dtc` to Convert DTS to YAML**:
  - Newer versions of `dtc` support outputting to YAML, which is easier to parse.
  - Command:
    ```bash
    dtc -I dts -O yaml -o zephyr.yaml zephyr.dts
    ```

---

### **3. Extract Variables from the Device Tree**

**Parsing the YAML File**

- **Use a YAML Parser in Python**:
  - Libraries: `PyYAML` or `ruamel.yaml`.
- **Extract Relevant Data**:
  - Load the YAML file into a Python dictionary.
  - Traverse the dictionary to find the nodes and properties you need.
  
**Example of Extracted Variables**

- `MODEL`
- `COMPATIBLE`
- `MEMORY_REG`
- `CPU_COMPATIBLE`
- Any other hardware-specific variables.

---

### **4. Make Variables Available to Robot Framework**

**Methods to Provide Variables**

- **Python Variable Files**: Robot Framework can import variables from Python files.
- **Dynamic Variable Libraries**: Create a custom library that generates variables at runtime.
- **Resource Files**: Use `.robot` files to define variables, but this is less dynamic.

**Creating a Variable File**

- **Generate a `variables.py` File**:
  - The parser script writes extracted variables into this file.
  - Variables are defined as uppercase Python variables.

---

### **5. Integrate the Parser into the Build Process**

**Automation**

- **Include Parsing in the Build Script**:
  - Modify your `build.sh` script to call the parser after compiling the device trees.
- **Ensure Up-to-Date Variables**:
  - Each time the device tree is built, the variables are regenerated.

**Example Modification to `build.sh`**

```bash
# After compiling the DTBs
python parse_dts.py zephyr.yaml variables.py
```

---

### **6. Use the Variables in Robot Framework Tests**

**Importing Variables**

- **In Your Test Suite**:
  ```robot
  *** Settings ***
  Variables    variables.py
  ```

**Using Variables**

- Access variables using `${VARIABLE_NAME}` syntax.
- Use them in test cases, keywords, and for making decisions.

**Example Test Case**

```robot
*** Test Cases ***
Validate Hardware Configuration
    Log    Model: ${MODEL}
    Should Be Equal    ${CPU_COMPATIBLE}    arm,cortex-a53
    Should Be Equal    ${MEMORY_REG[0]}     0x80000000
```

---

### **7. Example Implementation**

#### **7.1 Parser Script: `parse_dts.py`**

```python
import sys
import yaml

def parse_device_tree(yaml_file, output_file):
    with open(yaml_file, 'r') as f:
        data = yaml.safe_load(f)

    variables = {}
    root = data.get('/', {})

    # Extract model and compatible strings
    variables['MODEL'] = root.get('model', '')
    variables['COMPATIBLE'] = root.get('compatible', '')

    # Extract memory information
    memory_node = root.get('memory@80000000', {})
    variables['MEMORY_DEVICE_TYPE'] = memory_node.get('device_type', '')
    variables['MEMORY_REG'] = memory_node.get('reg', [])

    # Extract CPU information
    cpus = root.get('cpus', {}).get('cpu@0', {})
    variables['CPU_DEVICE_TYPE'] = cpus.get('device_type', '')
    variables['CPU_COMPATIBLE'] = cpus.get('compatible', '')
    variables['CPU_REG'] = cpus.get('reg', [])

    # Write variables to output file
    with open(output_file, 'w') as f:
        for key, value in variables.items():
            f.write(f"{key} = {repr(value)}\n")

if __name__ == '__main__':
    if len(sys.argv) != 3:
        print("Usage: python parse_dts.py <input_yaml> <output_variables.py>")
        sys.exit(1)
    parse_device_tree(sys.argv[1], sys.argv[2])
```

#### **7.2 Modified `build.sh` Script**

Add the following lines after the device trees are compiled:

```bash
# Parse Zephyr device tree and generate variables.py
dtc -I dtb -O yaml -o ${OUTPUT_DIR}/zephyr.yaml ${OUTPUT_DIR}/zephyr.dtb
python ../../tools/scripts/parse_dts.py ${OUTPUT_DIR}/zephyr.yaml variables.py
```

#### **7.3 Robot Framework Test Suite: `test.robot`**

```robot
*** Settings ***
Variables    variables.py

*** Test Cases ***
Validate Hardware Configuration
    Log    Model: ${MODEL}
    Should Be Equal    ${CPU_COMPATIBLE}    arm,cortex-a53
    Should Be Equal As Integers    ${MEMORY_REG[0]}    ${0x80000000}
```

---

### **8. Run the Process**

1. **Build the Device Trees and Parse Variables**

   ```bash
   cd tools/scripts
   ./build.sh
   ```

2. **Run Robot Framework Tests**

   ```bash
   robot test.robot
   ```

---

### **9. Additional Considerations**

- **Error Handling**: Enhance the parser to handle cases where nodes or properties might be missing.
- **Complex Structures**: If your device trees have more nested structures, adjust the parser to navigate through them.
- **Data Types**: Ensure that numerical values are correctly interpreted (e.g., hex values).
- **Multiple Platforms**: If you have multiple platforms, consider generating separate variable files for each.

---

### **10. Conclusion**

By integrating a parser into your build process, you can automatically extract hardware configuration variables from your device trees and make them available to your Robot Framework tests. This approach ensures that your tests are always in sync with the hardware descriptions, reducing manual errors and increasing test reliability.

---

**Feel free to customize and extend this solution to fit the specific needs of your project. If you have any further questions or need assistance with any step, don't hesitate to ask!**
