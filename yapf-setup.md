To set up **YAPF (Yet Another Python Formatter)** as a pre-commit hook for your project, follow these steps:

### 1. Install Dependencies
Ensure you have `pre-commit` and `yapf` installed. Use `pyenv` and `poetry` if you're using them as part of your workflow.

```bash
# Install pre-commit
pip install pre-commit

# Install yapf
pip install yapf
```

Run the following command to ensure dependencies are captured:
```bash
pip freeze > requirements.txt
```

### 2. Create a `.pre-commit-config.yaml` File
Add the following configuration for `yapf` in your `.pre-commit-config.yaml` file at the root of your repository:

```yaml
repos:
  - repo: https://github.com/pre-commit/mirrors-yapf
    rev: v0.40.2  # Replace with the latest version of yapf
    hooks:
      - id: yapf
        args: ["--style", "{based_on_style: pep8, indent_width: 4}"] # Optional: Specify a custom style
```

### 3. Install Pre-Commit Hook
Run the following command to install the pre-commit hooks in your repository:

```bash
pre-commit install
```

### 4. Test the Hook
Test the pre-commit hook by making a change to a Python file and committing it. `yapf` will automatically format your Python code before the commit is accepted.

```bash
git add your_python_file.py
git commit -m "Test YAPF formatting"
```

### 5. (Optional) Run Pre-Commit on All Files
To apply `yapf` formatting to all files in the repository, run:

```bash
pre-commit run --all-files
```

### 6. Verify Hook Installation
Check `.git/hooks/pre-commit` to ensure the pre-commit hook has been set up properly.

This setup ensures your Python code is automatically formatted according to YAPF's rules before every commit.
