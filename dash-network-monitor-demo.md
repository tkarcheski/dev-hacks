### Step 1: Update and Upgrade Your WSL Environment

1. **Open WSL Terminal**.
2. **Update and Upgrade Packages**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

### Step 2: Install Python and Pip

1. **Install Python and Pip**:
   ```bash
   sudo apt install python3 python3-pip python3-venv -y
   ```

### Step 3: Set Up a Python Virtual Environment

1. **Create a Virtual Environment**:
   ```bash
   python3 -m venv network_monitor_env
   ```
2. **Activate the Virtual Environment**:
   ```bash
   source network_monitor_env/bin/activate
   ```

### Step 4: Install Required Python Packages

1. **Install Dash, Dash Bootstrap Components, Pandas, and Psutil**:
   ```bash
   pip install dash dash-bootstrap-components pandas psutil
   ```

### Step 5: Create the Python Script

1. **Create a Script File**:
   - Save the following script as `network_monitor_dash.py`:

```python
import dash
import dash_bootstrap_components as dbc
from dash import dcc, html, Input, Output, State
import pandas as pd
import subprocess
import re
import logging
import platform
import psutil

# Configure logging with color formatting
class ColorFormatter(logging.Formatter):
    FORMAT = "%(levelname)s - %(message)s"

    BLACK, RED, GREEN, YELLOW, BLUE, MAGENTA, CYAN, WHITE = range(30, 38)

    COLORS = {
        'DEBUG': BLUE,
        'INFO': GREEN,
        'WARNING': YELLOW,
        'ERROR': RED,
        'CRITICAL': MAGENTA
    }

    def format(self, record):
        color = self.COLORS.get(record.levelname, self.WHITE)
        message = self.FORMAT % record.__dict__
        return f"\033[{color}m{message}\033[0m"

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger()
handler = logging.StreamHandler()
handler.setFormatter(ColorFormatter())
logger.addHandler(handler)

def detect_system():
    system_info = {
        'system': platform.system(),
        'node': platform.node(),
        'release': platform.release(),
        'version': platform.version(),
        'machine': platform.machine(),
        'processor': platform.processor()
    }
    logger.info(f"System information: {system_info}")
    return system_info

def get_system_diagnostics():
    cpu_usage = psutil.cpu_percent(interval=1)
    memory_info = psutil.virtual_memory()
    diagnostics = {
        'cpu_usage': cpu_usage,
        'memory_usage': memory_info.percent,
        'total_memory': memory_info.total,
        'used_memory': memory_info.used,
        'available_memory': memory_info.available
    }
    logger.info(f"System diagnostics: {diagnostics}")
    return diagnostics

def ping_host(host, count=1):
    logger.debug(f"Pinging host {host} with count {count}")
    command = ['ping', '-c', str(count), host]
    result = subprocess.run(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
    output = result.stdout
    logger.debug(f"Ping output: {output}")

    packet_loss, latency = parse_ping_output(output)
    timestamp = pd.Timestamp.now()
    
    if packet_loss is not None and latency is not None:
        ping_result = {
            'timestamp': timestamp.isoformat(),
            'packet_loss': packet_loss,
            'latency_min': latency['min'],
            'latency_avg': latency['avg'],
            'latency_max': latency['max'],
            'latency_mdev': latency['mdev']
        }
        logger.debug(f"Ping result: {ping_result}")
        return ping_result
    else:
        logger.debug("Ping result is None")
        return None

def parse_ping_output(output):
    logger.debug("Parsing ping output")
    packet_loss_pattern = re.compile(r'(\d+)% packet loss')
    latency_pattern = re.compile(r'(\d+\.\d+)/(\d+\.\d+)/(\d+\.\d+)/(\d+\.\d+) ms')

    packet_loss_match = packet_loss_pattern.search(output)
    latency_match = latency_pattern.search(output)

    packet_loss = int(packet_loss_match.group(1)) if packet_loss_match else None
    latency = {
        'min': float(latency_match.group(1)),
        'avg': float(latency_match.group(2)),
        'max': float(latency_match.group(3)),
        'mdev': float(latency_match.group(4))
    } if latency_match else None

    logger.debug(f"Packet loss: {packet_loss}, Latency: {latency}")
    return packet_loss, latency

app = dash.Dash(__name__, external_stylesheets=[dbc.themes.BOOTSTRAP])

app.layout = dbc.Container([
    dbc.Row([
        dbc.Col(html.H1("Network Monitoring Dashboard"), className="mb-2")
    ]),
    dbc.Row([
        dbc.Col(dbc.Input(id='host-input', type='text', placeholder='Enter host to monitor', debounce=True), width=6),
        dbc.Col(dbc.Button('Set Host', id='set-host-button', n_clicks=0), width=2)
    ], className="mb-2"),
    dbc.Row([
        dbc.Col(dcc.Graph(id='packet-loss-graph'), width=6),
        dbc.Col(dcc.Graph(id='latency-graph'), width=6),
    ]),
    dbc.Row([
        dbc.Col(html.Div(id='system-info'), width=12)
    ]),
    dcc.Store(id='ping-data', data=[]),
    dcc.Interval(id='interval-component', interval=1*1000, n_intervals=0)
], fluid=True)

@app.callback(
    Output('ping-data', 'data'),
    Input('interval-component', 'n_intervals'),
    State('ping-data', 'data'),
    State('host-input', 'value')
)
def update_ping_data(n_intervals, existing_data, host):
    logger.debug(f"update_ping_data called with n_intervals={n_intervals}, host={host}")
    if not host:
        logger.debug("No host set, returning existing data")
        return existing_data

    new_ping_result = ping_host(host)
    if new_ping_result:
        existing_data.append(new_ping_result)
        logger.info(f"Ping results: {new_ping_result}")

    logger.debug(f"Returning updated data with {len(existing_data)} entries")
    return existing_data

@app.callback(
    [Output('packet-loss-graph', 'figure'),
     Output('latency-graph', 'figure')],
    Input('ping-data', 'data')
)
def update_graphs(ping_data):
    logger.debug(f"update_graphs called with {len(ping_data)} ping_data entries")
    if not ping_data:
        logger.debug("No ping data available")
        return {}, {}

    df = pd.DataFrame(ping_data)

    packet_loss_fig = {
        'data': [{
            'x': df['timestamp'],
            'y': df['packet_loss'],
            'type': 'line',
            'name': 'Packet Loss (%)'
        }],
        'layout': {
            'title': 'Packet Loss Over Time',
            'xaxis': {'title': 'Time'},
            'yaxis': {'title': 'Packet Loss (%)'}
        }
    }

    latency_fig = {
        'data': [
            {'x': df['timestamp'], 'y': df['latency_min'], 'type': 'line', 'name': 'Min Latency (ms)'},
            {'x': df['timestamp'], 'y': df['latency_avg'], 'type': 'line', 'name': 'Avg Latency (ms)'},
            {'x': df['timestamp'], 'y': df['latency_max'], 'type': 'line', 'name': 'Max Latency (ms)'},
            {'x': df['timestamp'], 'y': df['latency_mdev'], 'type': 'line', 'name': 'Mdev Latency (ms)'}
        ],
        'layout': {
            'title': 'Latency Over Time',
            'xaxis': {'title': 'Time'},
            'yaxis': {'title': 'Latency (ms)'}
        }
    }

    logger.debug("Returning updated figures")
    return packet_loss_fig, latency_fig

@app.callback(
    Output('system-info', 'children'),
    Input('interval-component', 'n_intervals')
)
def update_system_info(n_intervals):
    system_info = detect_system()
    diagnostics = get_system_diagnostics()
    system_info_str = f"System: {system_info['system']} {system_info['release']} {system_info['version']} {system_info['machine']} {system_info['processor']}"
    diagnostics_str = f"CPU Usage: {diagnostics['cpu_usage']}%, Memory Usage: {diagnostics['memory_usage']}%, Total Memory: {diagnostics['total_memory']} bytes, Used Memory: {diagnostics['used_memory']} bytes, Available Memory: {diagnostics['available_memory']} bytes"
    return html.Div([
        html.P(system_info_str),
        html.P(diagnostics_str)
    ])

if __name__ == "__main__":
    logger.debug("Starting Dash app")
    app.run_server(debug=True)
```

### Step 6: Run the Script

1. **Open WSL Terminal**.
2. **Activate the Virtual Environment**:
  

 ```bash
   source network_monitor_env/bin/activate
   ```
3. **Run the Script**:
   ```bash
   python network_monitor_dash.py
   ```
4. **Open Your Web Browser**:
   - Go to `http://127.0.0.1:8050` to view the dashboard.
   - Enter the host to monitor in the input field and click "Set Host" to start monitoring. The log messages will provide detailed debug information about the script's execution.
