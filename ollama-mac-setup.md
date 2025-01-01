# Configuring Ollama on Mac: GPU Memory Allocation

This guide explains how to configure Ollama to optimize GPU memory usage on a macOS system with Apple Silicon (e.g., 48GB of memory). You can achieve this by temporarily or permanently modifying kernel parameters.

---

## **Temporary Configuration**

1. **Set GPU Memory Limit:**
   Open Terminal and run the following command:
   ```bash
   sudo sysctl -w iogpu.wired_limit_mb=32768
   ```
   - This sets the GPU memory allocation to 32GB.
   - Adjust the value (`32768` MB) as needed.

2. **Revert Changes:**
   The setting will reset automatically after a reboot. Alternatively, you can revert it manually:
   ```bash
   sudo sysctl -w iogpu.wired_limit_mb=default_value
   ```

---

## **Permanent Configuration**

To make the GPU memory allocation persistent across reboots, modify the system's kernel configuration file.

1. **Create or Edit `/etc/sysctl.conf`:**
   Open the configuration file with a text editor:
   ```bash
   sudo nano /etc/sysctl.conf
   ```

2. **Add the GPU Memory Parameter:**
   Add the following line to allocate 32GB to the GPU:
   ```plaintext
   iogpu.wired_limit_mb = 32768
   ```

3. **Save and Exit:**
   - In Nano, press `CTRL + O` to save and `CTRL + X` to exit.

4. **Apply Changes Immediately (Optional):**
   Run the following command to apply changes without rebooting:
   ```bash
   sudo sysctl -p /etc/sysctl.conf
   ```

5. **Revert Changes:**
   To revert, delete or comment out the line in `/etc/sysctl.conf` and restart your system.

---

## **Ollama-Specific Configuration**

While macOS dynamically manages GPU memory, you can optimize Ollama’s performance by setting environment variables:

1. **Set Environment Variables Temporarily:**
   Run these commands in the terminal:
   ```bash
   export OLLAMA_NUM_THREADS=8
   export OLLAMA_CUDA=1
   export OLLAMA_MAX_LOADED=2
   ```

2. **Make Variables Persistent:**
   Add the environment variables to your shell configuration file (e.g., `~/.zshrc`):
   ```bash
   echo "export OLLAMA_NUM_THREADS=8" >> ~/.zshrc
   echo "export OLLAMA_CUDA=1" >> ~/.zshrc
   echo "export OLLAMA_MAX_LOADED=2" >> ~/.zshrc
   source ~/.zshrc
   ```

---

## **Important Considerations**

- **Testing Stability:**
  Gradually increase GPU memory limits and monitor system behavior.
  
- **System Compatibility:**
  Ensure the parameter values do not exceed your system's memory capacity.

- **macOS Versions:**
  Some macOS versions may not fully support `/etc/sysctl.conf` without additional steps.

---

By following these steps, you can optimize Ollama's performance on macOS, leveraging your system's memory resources effectively.
