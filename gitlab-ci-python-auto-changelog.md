To set up a GitLab CI/CD pipeline that triggers on a tag, runs Python scripts to update the `changelog.rst` and bump the Python version, you can follow these steps:

### 1. Python Scripts

First, create Python scripts that handle updating the version and changelog:

#### `update_version.py`

```python
import re
import sys

def bump_version(version):
    version_parts = version.split('.')
    version_parts[-1] = str(int(version_parts[-1]) + 1)
    return '.'.join(version_parts)

def update_version_file(file_path):
    with open(file_path, 'r') as file:
        content = file.read()

    version_match = re.search(r"version=['\"](\d+\.\d+\.\d+)['\"]", content)
    if not version_match:
        print("Version not found!")
        sys.exit(1)

    current_version = version_match.group(1)
    new_version = bump_version(current_version)

    new_content = re.sub(r"version=['\"]\d+\.\d+\.\d+['\"]", f"version='{new_version}'", content)

    with open(file_path, 'w') as file:
        file.write(new_content)

    print(f"Updated version from {current_version} to {new_version}")
    return new_version

if __name__ == "__main__":
    file_path = sys.argv[1]  # Pass the file path as an argument
    update_version_file(file_path)
```

#### `update_changelog.py`

```python
import sys
from datetime import datetime

def update_changelog(changelog_path, new_version):
    with open(changelog_path, 'a') as changelog_file:
        changelog_file.write(f"\n## Version {new_version} - {datetime.now().strftime('%Y-%m-%d')}\n")
        changelog_file.write("- Auto-generated update.\n")

if __name__ == "__main__":
    changelog_path = sys.argv[1]  # Pass the changelog file path as an argument
    new_version = sys.argv[2]  # Pass the new version as an argument
    update_changelog(changelog_path, new_version)
```

### 2. GitLab CI/CD Configuration

Next, configure your `.gitlab-ci.yml` to trigger on tags, run these Python scripts, and push the changes back to the repository.

#### `.gitlab-ci.yml`

```yaml
stages:
  - update_version
  - update_changelog
  - commit_changes

variables:
  VERSION_FILE: "setup.py"  # or pyproject.toml if you're using it
  CHANGELOG_FILE: "changelog.rst"

before_script:
  - git config --global user.email "ci@example.com"
  - git config --global user.name "GitLab CI"
  - python -m pip install --upgrade pip  # Ensure the latest pip
  - pip install -r requirements.txt  # Install any dependencies

update_version:
  stage: update_version
  script:
    - echo "Running version update script"
    - NEW_VERSION=$(python update_version.py $VERSION_FILE)
    - echo "New version is $NEW_VERSION"
    - git add $VERSION_FILE

update_changelog:
  stage: update_changelog
  script:
    - echo "Running changelog update script"
    - python update_changelog.py $CHANGELOG_FILE $NEW_VERSION
    - git add $CHANGELOG_FILE

commit_changes:
  stage: commit_changes
  script:
    - git commit -m "Bump version to $NEW_VERSION and update changelog"
    - git push origin $CI_COMMIT_REF_NAME
  only:
    - tags
```

### 3. Explanation:

- **Stages**: The pipeline is divided into three stages: `update_version`, `update_changelog`, and `commit_changes`.

- **Python Scripts**: 
  - `update_version.py`: This script reads the version from the `setup.py` (or another file), increments it, and writes the new version back to the file.
  - `update_changelog.py`: This script appends a new entry in the `changelog.rst` with the new version.

- **Commit and Push**: The changes are committed and pushed back to the same tag.

- **Trigger on Tag**: The `only: - tags` directive ensures that this pipeline runs only when a tag is pushed.
