Here’s a step-by-step guide to set up a Python DevOps project using **pyenv**, **Poetry**, and **pre-commit**. The example includes setting up YAPF for formatting, a pre-commit hook, and a basic project structure.

---

### 1. Set Up Python with `pyenv`
Install and set a Python version using `pyenv`:

```bash
# Install pyenv
curl https://pyenv.run | bash

# Add pyenv to your shell configuration
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init --path)"' >> ~/.bashrc
source ~/.bashrc

# Install a Python version (e.g., 3.12.0)
pyenv install 3.12.0

# Set the Python version globally or for the project
pyenv global 3.12.0
# or
pyenv local 3.12.0
```

Verify Python version:

```bash
python --version
```

---

### 2. Create and Set Up the Project
Create a new project directory and initialize it:

```bash
mkdir my-devops-project
cd my-devops-project
```

#### Initialize Git
```bash
git init
```

---

### 3. Install and Configure Poetry
Install Poetry:

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

Configure Poetry to use the `pyenv`-managed Python version:

```bash
poetry env use $(pyenv which python)
```

#### Initialize the Project
```bash
poetry init
```

Follow the prompts to configure the project (e.g., project name, version, etc.).

---

### 4. Install Dev Tools
Add `pre-commit` and `yapf` as development dependencies:

```bash
poetry add --group dev pre-commit yapf
```

---

### 5. Set Up Pre-Commit
Create a `.pre-commit-config.yaml` file for formatting and linting:

```yaml
repos:
  - repo: https://github.com/pre-commit/mirrors-yapf
    rev: v0.40.2  # Replace with the latest stable version
    hooks:
      - id: yapf
        args: ["--style", "{based_on_style: pep8, indent_width: 4}"]  # Optional: Customize formatting
```

Install the pre-commit hooks:

```bash
poetry run pre-commit install
```

---

### 6. Project Structure
Organize the project as follows:

```plaintext
my-devops-project/
├── .git/
├── .pre-commit-config.yaml
├── pyproject.toml
├── README.md
├── requirements.txt  # Exported dependencies
├── src/
│   └── my_devops_project/
│       ├── __init__.py
│       └── main.py  # Main script
└── tests/
    └── test_main.py
```

#### Create Source Directory
```bash
mkdir -p src/my_devops_project
touch src/my_devops_project/__init__.py
touch src/my_devops_project/main.py
```

Example `main.py`:

```python
def main():
    print("Welcome to My DevOps Project!")

if __name__ == "__main__":
    main()
```

#### Create Test Directory
```bash
mkdir tests
touch tests/test_main.py
```

Example `test_main.py`:

```python
from my_devops_project.main import main

def test_main():
    assert main() is None
```

---

### 7. Automate Formatting and Testing
Add `format` and `test` scripts in `pyproject.toml`:

```toml
[tool.poetry.scripts]
format = "yapf:main"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"
```

---

### 8. Export Dependencies
Export `requirements.txt` for environments that need it:

```bash
poetry export -f requirements.txt --without-hashes -o requirements.txt
```

---

### 9. Run Your Workflow
- **Run Pre-Commit Hooks:**

  ```bash
  poetry run pre-commit run --all-files
  ```

- **Format Code:**

  ```bash
  poetry run yapf --recursive --in-place src/
  ```

- **Run Tests:**

  ```bash
  poetry run pytest
  ```

---

### Full Example Commands

```bash
# Set up Python environment
pyenv install 3.12.0
pyenv local 3.12.0

# Initialize project
mkdir my-devops-project
cd my-devops-project
git init

# Install Poetry and configure environment
curl -sSL https://install.python-poetry.org | python3 -
poetry env use $(pyenv which python)
poetry init

# Add development dependencies
poetry add --group dev pre-commit yapf pytest

# Set up pre-commit
echo "repos:
  - repo: https://github.com/pre-commit/mirrors-yapf
    rev: v0.40.2
    hooks:
      - id: yapf
        args: [\"--style\", \"{based_on_style: pep8, indent_width: 4}\"]
" > .pre-commit-config.yaml
poetry run pre-commit install

# Set up project structure
mkdir -p src/my_devops_project
touch src/my_devops_project/{__init__.py,main.py}
mkdir tests
touch tests/test_main.py

# Run checks and tests
poetry run pre-commit run --all-files
poetry run yapf --recursive --in-place src/
poetry run pytest
```

This setup ensures a robust environment for a Python-based DevOps project!
