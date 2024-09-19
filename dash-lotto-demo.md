This project is a **Dash web application** that simulates lottery number draws and tracks how frequently each number appears, as well as how often certain combinations of lottery numbers are repeated. It provides visual feedback through a graph and continuously updates over time using an **interval-based callback**.

Here's a breakdown of the project's key components and functionality:

### 1. **Dash Framework**
   - **Dash** is a Python framework for building web applications with a focus on data visualization. It integrates well with libraries like Plotly for interactive graphs.
   - The application is served via `app.run_server()` and can be accessed through a web browser.

### 2. **Application Layout**
   - The layout is defined in the `app.layout` variable, which uses a combination of Dash components:
     - **`dcc.Interval`**: Triggers the callback function every 100 milliseconds (configurable via the `interval` property) to simulate a new lottery roll.
     - **`dcc.Graph`**: Displays a bar graph showing the frequency of the lottery numbers.
     - **`dcc.Markdown`**: Displays additional information, such as:
       - The total number of winning combinations.
       - Time-tracking information (e.g., how long each lottery roll takes and the total time since the app started).

### 3. **Lottery Generation**
   - The `generate_lottery()` function randomly generates 6 unique numbers between 1 and 45 to simulate a lottery draw.
   - This list of 6 numbers is sorted and returned as the lottery result for each interval.

### 4. **Tracking Winning Combinations**
   - Every time a new set of lottery numbers is generated, it's added to the global `winning_combinations` list.
   - The project also tracks the frequency of each number in the lottery results using Python's `collections.Counter` to count how often each number is drawn.

### 5. **Visualization**
   - The frequency of the lottery numbers is displayed in a bar chart (using **Plotly** via `go.Bar`), with colors representing the relative frequency of each number.
   - The title of the graph updates dynamically, showing:
     - The count of repeated combinations (if any).
     - The **most frequent combination** of lottery numbers that has appeared the most times.

### 6. **Repeated Combinations**
   - The `combination_counter` tracks how many times each set of lottery numbers has been drawn. This allows the app to detect repeated combinations and display the total number of such repeated combinations.

### 7. **Performance Tracking**
   - The app tracks:
     - **Time taken for each lottery roll**: Measured using the `time.time()` function to compute the duration of each interval's callback execution.
     - **Total time since the app started**: The time elapsed since the app began running.

### 8. **Callback Function**
   - The `@app.callback` decorator ties the UI (inputs and outputs) to a Python function. This callback:
     - Updates the lottery number frequency graph.
     - Shows the count of total winning combinations.
     - Tracks and displays time-related data.
     - Displays the most frequent lottery combination in the graph's title.

### Workflow of the Application:
1. **Start**: When the app runs, it continuously generates new sets of lottery numbers every 100 milliseconds.
2. **Data Collection**: Each new set of numbers is added to a list, and the frequency of individual numbers and combinations is updated.
3. **Visualization**: The frequency of numbers is shown in a bar chart, with the title updating to show repeated combinations and the most frequent set of numbers.
4. **Performance Tracking**: The app keeps track of how long each lottery generation takes and the total time since the application started.
5. **Updates**: The graph, counts, and time data are updated every 100 milliseconds in real time.

### Key Technologies:
- **Dash**: Provides the web framework and user interface components.
- **Plotly**: Handles interactive charting and data visualization.
- **NumPy**: Generates random numbers efficiently.
- **collections.Counter**: Tracks the frequency of numbers and combinations.
- **time**: Measures time intervals to track performance and total runtime.

### Possible Uses:
- **Simulation Tool**: Could be used for simulating or analyzing random events like lottery draws.
- **Real-time Data Visualization**: Useful for learning about real-time graphing and data update mechanisms in Dash.
- **Performance Tracking**: Demonstrates how to measure execution time for operations and track cumulative performance over time.

This project can be expanded with additional features like user input to customize the lottery number range, frequency of updates, or even saving and exporting data.

### 1. **Main Python File (app.py)**

```python
import dash
from dash import dcc, html, Input, Output
import plotly.graph_objs as go
import numpy as np
from collections import Counter
import time

# Global list to track winning combinations (initialized empty)
winning_combinations = []
combination_counter = Counter()
start_time = time.time()

app = dash.Dash(__name__)

def generate_lottery():
    """
    Simulates generating 6 unique random numbers between 1 and 45 for a lottery game.
    
    Returns:
        A sorted list of six distinct integers.
    """
    return sorted(np.random.choice(range(1, 46), size=6, replace=False).tolist())

app.layout = html.Div([
    dcc.Interval(id='interval-component', interval=1*100),  # Runs every 100 milliseconds
    dcc.Graph(id='lottery-graph'),
    dcc.Markdown(id='winning-combinations-count'),
    dcc.Markdown(id='time-tracking')
], style={'padding': '10px'})

@app.callback(
    [Output('lottery-graph', 'figure'),
     Output('winning-combinations-count', 'children'),
     Output('time-tracking', 'children')],
    [Input('interval-component', 'n_intervals')]
)
def update_lottery_graph(n):
    """
    Updates the graph of lottery numbers and updates the winning combinations count.
    
    Args:
        n (int): The number of intervals that have passed since last callback.
        
    Returns:
        A dictionary with 'data' for plotting, and 'layout' to define chart structure.
        A string with the count of winning combinations.
        A string with time tracking information.
    """
    # Ensure `n` is an integer and greater than 0
    if n is not None and isinstance(n, int) and n % 1 == 0:
        interval_start_time = time.time()
        
        new_combination = generate_lottery()
        combination_tuple = tuple(new_combination)
        
        # Track the combination count
        combination_counter[combination_tuple] += 1
        winning_combinations.extend(new_combination)
        
        # Calculate frequency of each number in all winning combinations
        counter = Counter(winning_combinations)
        
        # Prepare data for plotting the frequency of each number
        x_data = list(counter.keys())
        y_data = list(counter.values())
        
        # Create a color scale based on frequency
        colors = ['rgba(0, 204, 150, {:.2f})'.format(val / max(y_data)) for val in y_data]
        
        trace = go.Bar(x=x_data, y=y_data, marker=dict(color=colors), name='Number Frequency')
        
        # Check for repeated combinations
        repeated = [combo for combo, count in combination_counter.items() if count > 1]
        
        # Find the most frequent hitting combination
        most_frequent_combo = combination_counter.most_common(1)
        if most_frequent_combo:
            most_frequent_combo_str = "Most Frequent Combo: {}".format(most_frequent_combo[0][0])
        else:
            most_frequent_combo_str = "No combos yet."
        
        title = "Lottery Number Frequency Over Time"
        if repeated:
            title += " (Repeated Combos: {})".format(len(repeated))
        
        title += f" | {most_frequent_combo_str}"
        
        # Construct figure with the latest data and layout settings (e.g., title).
        fig = {
            'data': [trace],
            'layout': go.Layout(title=title,
                                xaxis={'title': 'Lottery Numbers'},
                                yaxis={'title': 'Frequency'})
        }
        
        winning_combinations_count = "Winning Combinations Count: {}".format(len(winning_combinations) // 6)
        
        interval_end_time = time.time()
        interval_duration = interval_end_time - interval_start_time
        total_duration = interval_end_time - start_time
        
        time_tracking_info = (
            f"Time for last lottery roll: {interval_duration:.4f} seconds\n"
            f"Total time since start: {total_duration:.4f} seconds"
        )
        
        return fig, winning_combinations_count, time_tracking_info
    else:
        raise ValueError("Invalid interval value provided to the update_lottery_graph callback.")

if __name__ == '__main__':
    app.run_server(debug=True, dev_tools_hot_reload=True)
```

### 2. **Requirements File (requirements.txt)**

To ensure all necessary dependencies are installed, create a `requirements.txt` file:

```txt
dash==2.11.0
plotly==5.15.0
numpy==1.24.0
```

### Running the Application

1. **Install dependencies**:
   First, create a virtual environment and install the necessary libraries using:

   ```bash
   pip install -r requirements.txt
   ```

2. **Run the Dash application**:
   After installing the dependencies, run the app by executing the following command:

   ```bash
   python app.py
   ```

### Directory Structure

The project directory should look like this:

```
/your_project_directory
    ├── app.py
    ├── requirements.txt
```

This setup will allow you to run the Dash application, visualize the lottery frequency, and track time and combinations.
