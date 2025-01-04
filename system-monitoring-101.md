# **System Monitoring Commands for macOS, Windows, and Linux**

This guide provides commands and tools for monitoring system resources on macOS, Windows, and Linux. Each section includes commands for CPU, memory, disk, network, and GPU usage.

---

## **1. CPU Usage**

### **macOS**
```bash
# Monitor CPU usage
top -l 1 | grep "CPU usage" | awk '{print $3, $5, $7}'
```

### **Windows (PowerShell)**
```powershell
# Monitor CPU usage
Get-Counter "\Processor(_Total)\% Processor Time"
```

### **Linux**
```bash
# Monitor CPU usage
top -b -n1 | grep "Cpu(s)" | awk '{print $2 + $4}'
```

---

## **2. Memory Usage**

### **macOS**
```bash
# Monitor memory usage
vm_stat | awk '
BEGIN { print "Active Memory:", "Inactive Memory:", "Free Memory:"; }
$1 == "Pages active:" { print $2 * 4096 / (1024*1024), "MB"; }
$1 == "Pages inactive:" { print $2 * 4096 / (1024*1024), "MB"; }
$1 == "Pages free:" { print $2 * 4096 / (1024*1024), "MB"; }'
```

### **Windows (PowerShell)**
```powershell
# Monitor memory usage
Get-Counter "\Memory\Available MBytes"
```

### **Linux**
```bash
# Monitor memory usage
free -h
```

---

## **3. Disk Usage**

### **macOS**
```bash
# Monitor disk usage
df -h
```

### **Windows (PowerShell)**
```powershell
# Monitor disk usage
Get-PSDrive -PSProvider FileSystem
```

### **Linux**
```bash
# Monitor disk usage
df -h
```

---

## **4. Network Usage**

### **macOS**
```bash
# Monitor network usage
netstat -ib | awk '
BEGIN { print "Interface BytesIn BytesOut"; }
NR > 1 { print $1, $7, $10; }'
```

### **Windows (PowerShell)**
```powershell
# Monitor network usage
Get-Counter -Counter "\Network Interface(*)\Bytes Total/sec"
```

### **Linux**
```bash
# Monitor network usage
ifconfig | grep "RX bytes" -A1
```

---

## **5. GPU Usage**

### **macOS**
```bash
# Monitor GPU usage (integrated GPUs)
sudo powermetrics --samplers gpu_power -n 1 | grep -E "GPU Power|GPU Utilization"
```

### **Windows (Command Prompt or PowerShell)**
#### For NVIDIA GPUs:
```powershell
# Monitor GPU usage
nvidia-smi --query-gpu=timestamp,utilization.gpu,utilization.memory,temperature.gpu --format=csv
```

### **Linux**
#### For NVIDIA GPUs:
```bash
# Monitor GPU usage
nvidia-smi --query-gpu=timestamp,utilization.gpu,utilization.memory,temperature.gpu --format=csv
```

#### For AMD GPUs:
```bash
# Monitor GPU usage
rocm-smi --showtemp --showuse
```

---

## **Automating Data Collection**

### macOS
Use `cron` or a shell script to collect data over time:
```bash
*/1 * * * * /path/to/command >> /path/to/output.csv
```

### Windows
Use Task Scheduler or a batch file with:
```powershell
powershell -Command "& {command} >> output.csv"
```

### Linux
Use `cron` to automate commands:
```bash
*/1 * * * * /path/to/command >> /path/to/output.csv
```

---

## **Visualizing Data**

To visualize the collected data, use tools like **Python** with libraries such as `matplotlib` and `pandas`, or export the data to a spreadsheet for manual plotting.

Example Python visualization script:
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load CSV data
data = pd.read_csv("output.csv")

# Plot data
plt.figure(figsize=(10, 5))
plt.plot(data['Time'], data['Metric'], label="Metric")
plt.xlabel("Time")
plt.ylabel("Value")
plt.title("System Resource Usage Over Time")
plt.legend()
plt.show()
```
