### Project Overview

This project is a **Rocket Launch Simulation** built using **Dash** and **Plotly** for visualizing the rocket's trajectory in 3D. The app simulates the launch of a rocket from Earth, with parameters like thrust, mass, drag coefficient, and flight angle adjustable through a web interface. It also simulates rocket stages, where the rocket sheds mass and adjusts thrust at specific points to increase efficiency.

The rocket's trajectory is visualized in a 3D environment, showing Earth, the Moon, and the rocket's path in real time. The project handles the physics of a rocket launch, including atmospheric drag and the effect of multiple rocket stages, and caps velocity to prevent overflow issues.

### Core Features:
- Adjustable rocket parameters: thrust, mass, drag coefficient, reference area, flight angle, and more.
- Simulates multiple rocket stages, each with different thrust and mass configurations.
- 3D visualization of Earth's and the Moon's positions, along with the rocket's trajectory.
- Escape velocity capping to avoid numerical overflow.

### Project Files

Here is the list of all files needed to rebuild the project:

1. **`app.py`**: The main Dash application file. It handles the layout of the app and the callback logic for updating the graph based on user input.

2. **`plots.py`**: Contains the logic to calculate the rocket's trajectory and generate the 3D Plotly graph, including Earth, the Moon, and the rocket's path.

---

### File 1: `app.py`

This file sets up the web interface using **Dash** and defines sliders for the rocket's parameters. It also includes the callback that triggers the `update_graph` function from `plots.py` to update the simulation when parameters change.

```python
import dash
import dash_bootstrap_components as dbc
from dash import dcc, html
from dash.dependencies import Input, Output
from plots import update_graph

# Initialize the Dash app
app = dash.Dash(__name__, external_stylesheets=[dbc.themes.BOOTSTRAP])

# Layout of the app
app.layout = dbc.Container([
    dbc.Row(dbc.Col(html.H1("Rocket Launch Simulation"), width={'size': 6, 'offset': 3})),
    
    # Input controls
    dbc.Row([
        dbc.Col([
            dbc.Label("Thrust (N):"),
            dcc.Slider(id='thrust-slider', min=0, max=5e7, step=1e5, value=2e7,
                       marks={i: f'{i/1e6:.1f}M' for i in range(0, int(5e7) + 1, int(1e7))}),
            dbc.Label("Mass (kg):"),
            dcc.Slider(id='mass-slider', min=10000, max=3e6, step=10000, value=2.8e6,
                       marks={i: f'{i/1e6:.1f}M' for i in range(10000, int(3e6) + 1, int(5e5))}),
            dbc.Label("Drag Coefficient:"),
            dcc.Slider(id='cd-slider', min=0, max=1, step=0.01, value=0.5,
                       marks={i / 10: str(i / 10) for i in range(0, 11)}),
            dbc.Label("Reference Area (m^2):"),
            dcc.Slider(id='area-slider', min=1, max=100, step=1, value=10,
                       marks={i: str(i) for i in range(1, 101, 10)}),
            dbc.Label("Scale Height (m):"),
            dcc.Slider(id='scale-height-slider', min=1000, max=20000, step=100, value=8500,
                       marks={i: str(i) for i in range(1000, 20001, 2000)}),
            dbc.Label("Flight Angle (degrees):"),
            dcc.Slider(id='angle-slider', min=0, max=90, step=1, value=90,
                       marks={i: str(i) for i in range(0, 91, 10)}),
            dbc.Label("Launch Longitude (degrees):"),
            dcc.Slider(id='longitude-slider', min=-180, max=180, step=1, value=0,
                       marks={i: str(i) for i in range(-180, 181, 30)}),
            dbc.Label("Rocket Stages:"),
            dcc.Slider(id='stages-slider', min=1, max=5, step=1, value=1,
                       marks={i: str(i) for i in range(1, 6)}),
        ], width=6),
    ], style={'padding': 20}),
    
    # Graphs
    dbc.Row(dbc.Col(dcc.Graph(id='simulation-graph', style={'height': '80vh'}))),
], fluid=True)

# Callback to update the graph based on input controls
@app.callback(
    Output('simulation-graph', 'figure'),
    Input('thrust-slider', 'value'),
    Input('mass-slider', 'value'),
    Input('cd-slider', 'value'),
    Input('area-slider', 'value'),
    Input('scale-height-slider', 'value'),
    Input('angle-slider', 'value'),
    Input('longitude-slider', 'value'),
    Input('stages-slider', 'value')
)
def update_graph_callback(thrust, mass, Cd, A, scale_height, angle, longitude, stages):
    return update_graph(thrust, mass, Cd, A, scale_height, angle, longitude, stages)

# Run the app
if __name__ == '__main__':
    app.run_server(debug=True)
```

---

### File 2: `plots.py`

This file handles the actual rocket simulation calculations. It calculates the rocket's velocity, altitude, and trajectory considering drag, gravity, and multiple rocket stages. It also generates a 3D Plotly plot of Earth, the Moon, and the rocket's path.

```python
import numpy as np
import plotly.graph_objs as go

def update_graph(thrust, mass, Cd, A, scale_height, angle, longitude, stages):
    # Constants
    rho0 = 1.225  # kg/m^3, sea level air density
    g = 9.81  # m/s^2, gravity
    angle_rad = np.radians(angle)  # convert angle to radians
    earth_radius = 6371000  # meters
    moon_radius = 1737000  # meters
    moon_distance = 384400000  # meters

    # Time parameters
    time_end = 86400  # seconds, total simulation time (24 hours)
    dt = 10  # time step in seconds
    time = np.arange(0, time_end, dt)

    # Initialize arrays
    altitude = np.zeros_like(time, dtype=np.float64)
    velocity = np.zeros_like(time, dtype=np.float64)
    drag = np.zeros_like(time, dtype=np.float64)
    acceleration = np.zeros_like(time, dtype=np.float64)
    x_position = np.zeros_like(time, dtype=np.float64)
    y_position = np.zeros_like(time, dtype=np.float64)
    stage_thrust = thrust / stages
    stage_mass = mass / stages

    current_mass = mass

    # Simulation loop
    for i in range(1, len(time)):
        if i % (len(time) // stages) == 0:
            # Change stage
            current_mass -= stage_mass
            thrust -= stage_thrust

        rho = rho0 * np.exp(-altitude[i-1] / scale_height)
        drag[i] = 0.5 * rho * velocity[i-1]**2 * Cd * A
        net_force = thrust - drag[i] - current_mass * g
        acceleration[i] = net_force / current_mass
        velocity[i] = velocity[i-1] + acceleration[i] * dt
        altitude[i] = altitude[i-1] + velocity[i] * np.sin(angle_rad) * dt
        x_position[i] = x_position[i-1] + velocity[i] * np.cos(angle_rad) * dt
        y_position[i] = altitude[i]

        # Cap velocity to avoid overflow
        if velocity[i] > 11000:  # Approximate escape velocity
            velocity[i] = 11000

    # Create the 3D figure with Earth and Moon
    fig = go.Figure()

    # Earth sphere
    u = np.linspace(0, 2 * np.pi, 100)
    v = np.linspace(0, np.pi, 100)
    x = earth_radius * np.outer(np.cos(u), np.sin(v))
    y = earth_radius * np.outer(np.sin(u), np.sin(v))
    z = earth_radius * np.outer(np.ones(np.size(u)), np.cos(v))

    fig.add_trace(go.Surface(x=x, y=y, z=z, colorscale='Blues', showscale=False, opacity=0.5, name='Earth'))

    # Moon sphere
    x_moon = moon_radius * np.outer(np.cos(u), np.sin(v)) + moon_distance
    y_moon = moon_radius * np.outer(np.sin(u), np.sin(v))
    z_moon = moon_radius * np.outer(np.ones(np.size(u)), np.cos(v))

    fig.add_trace(go.Surface(x=x_moon, y=y_moon,```python
    z=z_moon, colorscale='Greys', showscale=False, opacity=0.5, name='Moon'))

    # Rocket trajectory
    fig.add_trace(go.Scatter3d(
        x=x_position + earth_radius * np.cos(np.radians(longitude)),
        y=y_position,
        z=altitude + earth_radius * np.sin(np.radians(longitude)),
        mode='lines', name='Rocket Trajectory', line=dict(color='red')
    ))

    fig.update_layout(
        title='Rocket Trajectory and Drag Force vs. Time',
        scene=dict(
            xaxis_title='X Position (m)',
            yaxis_title='Y Position (m)',
            zaxis_title='Altitude (m)'
        )
    )

    return fig
```

---

### Instructions for Rebuilding the Project

1. **Set up a Python environment**:
   - Create a virtual environment:
     ```sh
     python3 -m venv venv
     source venv/bin/activate  # For Windows: venv\Scripts\activate
     ```

   - Install the necessary packages:
     ```sh
     pip install dash dash-bootstrap-components plotly numpy
     ```

2. **Create the project structure**:
   - Create a folder for your project.
   - Inside that folder, create two files: `app.py` and `plots.py`.

3. **Add code to `app.py` and `plots.py`**:
   - Copy the code for `app.py` and `plots.py` provided above into their respective files.

4. **Run the application**:
   - Run the app by executing the following command in the project directory:
     ```sh
     python app.py
     ```

5. **Access the application**:
   - Open your browser and go to `http://127.0.0.1:8050/` to see the running Dash application. You can now adjust the rocket's parameters and stages, and see its trajectory relative to Earth and the Moon in real time.

### Summary

- The project is a 3D rocket launch simulation built using Dash and Plotly.
- The simulation includes adjustable parameters like thrust, mass, drag coefficient, and flight angle, as well as a slider for simulating multiple rocket stages.
- The project contains two main files:
  - `app.py`: The main Dash application, containing the layout and callback logic.
  - `plots.py`: The logic for calculating the rocket's trajectory and generating the 3D plot.
- By adjusting the inputs in the web interface, you can see how the rocket's trajectory changes, including the impact of multiple stages.
