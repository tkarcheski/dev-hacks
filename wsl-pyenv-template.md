Here are the step-by-step instructions to set up WSL (Windows Subsystem for Linux), Python, and `pyenv` for managing Python 3.12 environments in WSL, specifically for a beginner:

### 1. **Install WSL**
   - **Step 1:** Open **PowerShell** - if needed (sometimes not), run as an Administrator.
     - You can do this by right-clicking the Start menu and selecting **Windows PowerShell (Admin)** or **Terminal (Admin)**.
   
   - **Step 2:** Install WSL with this command:
     ```bash
     wsl --install
     ```
     - This will install the latest version of WSL (WSL2) and the default Linux distribution (Ubuntu).
   
   - **Step 3:** Restart your computer when prompted.

   - **Step 4:** After the restart, **Ubuntu** (or your chosen Linux distribution) will automatically open. It will ask you to create a new user and password for the Linux system. Make sure to remember these, as you will use them regularly in the WSL terminal.

### 2. **Update Your WSL Distribution**
   - Once you are inside the WSL terminal (Ubuntu), update the system to make sure everything is up-to-date:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

### 3. **Install Build Dependencies for `pyenv`**
   - `pyenv` requires some dependencies to build Python versions. Install them by running the following command in WSL:
     ```bash
     sudo apt install -y make build-essential libssl-dev zlib1g-dev \
     libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
     libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev \
     liblzma-dev python-openssl git
     ```

### 4. **Install `pyenv`**
   - **Step 1:** Install `pyenv` by running the following command in the WSL terminal:
     ```bash
     curl https://pyenv.run | bash
     ```
   
   - **Step 2:** Add `pyenv` to your shell startup file to enable it whenever you open a terminal. Run these commands:
     ```bash
     echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
     echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
     echo 'eval "$(pyenv init --path)"' >> ~/.bashrc
     echo 'eval "$(pyenv init -)"' >> ~/.bashrc
     source ~/.bashrc
     ```

### 5. **Install Python 3.12 Using `pyenv`**
   - **Step 1:** Install Python 3.12 using `pyenv`. First, list all the available versions:
     ```bash
     pyenv install --list
     ```
     - This will show all available Python versions.
   
   - **Step 2:** Install Python 3.12 (replace `x.x` with the exact version number, like `3.12.0`):
     ```bash
     pyenv install 3.12.x
     ```

   - **Step 3:** Set Python 3.12 as the global default:
     ```bash
     pyenv global 3.12.x
     ```

   - **Step 4:** Verify that Python 3.12 is installed correctly:
     ```bash
     python --version
     ```

### 6. **Ensure `pip` and Virtual Environment Support**
   - **Step 1:** Install `pip`, the Python package manager, if it's not already included:
     ```bash
     sudo apt install python3-pip
     ```
   - **Step 2:** Install `virtualenv` to create isolated environments for your Python projects:
     ```bash
     pip install virtualenv
     ```

### 7. **Create a Virtual Environment (Optional)**
   - **Step 1:** Navigate to your project folder or create a new one:
     ```bash
     mkdir myproject && cd myproject
     ```
   
   - **Step 2:** Create a virtual environment:
     ```bash
     python -m venv venv
     ```

   - **Step 3:** Activate the virtual environment:
     ```bash
     source venv/bin/activate
     ```

   - Now, your Python environment is isolated, and you can start installing packages using `pip`.

### 8. **Run `pip freeze > requirements.txt` After Installing Any Packages**
   - This will generate a `requirements.txt` file to track your Python dependencies:
     ```bash
     pip freeze > requirements.txt
     ```

### Summary:
- **Install WSL** via PowerShell.
- **Update your system** in WSL (Ubuntu).
- **Install necessary build tools** for Python and `pyenv`.
- **Install and configure `pyenv`** to manage Python versions.
- **Install Python 3.12** and set it as the global version.
- Optionally, **create and activate virtual environments** for Python projects.

This setup will give you a flexible Python development environment inside WSL with version control via `pyenv`.
