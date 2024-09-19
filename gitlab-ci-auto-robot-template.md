## **Project Overview**

### **Objective**

The goal of this project is to automate the process of updating the `changelog.rst` file and bumping the Python package version whenever a new Git tag is pushed to the repository. Additionally, it incorporates:

- Mentioning **robot labels** from commit messages.
- Providing a status of changed **Python** and **Robot Framework** source files.

This automation is achieved using a GitLab CI/CD pipeline that triggers on tags and runs a Python script to handle the changelog update.

---

## **How It Works**

1. **Git Tag Push**: When a new Git tag is pushed to the repository, it triggers the GitLab CI/CD pipeline.

2. **CI/CD Pipeline Execution**:
   - **Stage 1: Update Changelog**
     - The pipeline runs the `generate_changelog.py` script.
     - The script performs the following:
       - Extracts the tag name, commit message, and robot labels from the latest commit.
       - Retrieves a list of changed Python (`.py`) and Robot Framework (`.robot`) files.
       - Uses a template (`changelog_template.rst`) to generate a new changelog entry with the collected information.
       - Appends this new entry to the `changelog.rst` file.
   - **Stage 2: Commit Changes**
     - The pipeline commits the updated `changelog.rst` back to the repository under the same tag.

3. **Python Script Enhancements**:
   - **Robot Labels**: Extracted from commit messages, assuming they are enclosed in square brackets (e.g., `[label1, label2]`).
   - **Changed Files**: The script identifies and lists changed Python and Robot Framework files in the changelog.

---

## **File Contents**

Below are all the relevant files with their contents:

### **1. `generate_changelog.py`**

This Python script generates the changelog entry by filling in the placeholders in the template and appending it to `changelog.rst`.

```python
import sys
from datetime import datetime
from string import Template
import subprocess

def get_changed_files():
    """
    Retrieves a list of files changed in the last commit.
    """
    result = subprocess.run(['git', 'diff', '--name-only', 'HEAD~1'], capture_output=True, text=True)
    if result.returncode != 0:
        print(f"Error getting changed files: {result.stderr}")
        sys.exit(1)
    return result.stdout.splitlines()

def categorize_files(files):
    """
    Categorizes the list of files into Python and Robot Framework files.
    """
    python_files = [f for f in files if f.endswith('.py')]
    robot_files = [f for f in files if f.endswith('.robot')]
    return python_files, robot_files

def generate_changelog(template_file, changelog_file, tag, commit_message, robot_labels):
    """
    Generates the changelog entry and appends it to the changelog file.
    """
    # Get current date
    current_date = datetime.now().strftime('%Y-%m-%d')

    # Get the list of changed files
    changed_files = get_changed_files()
    python_files, robot_files = categorize_files(changed_files)

    # Read the template file
    with open(template_file, 'r') as file:
        template_content = file.read()

    # Create a template object and substitute values
    template = Template(template_content)
    filled_content = template.substitute(
        tag=tag,
        date=current_date,
        comments=commit_message,
        robot_labels=robot_labels if robot_labels else 'None',
        python_files='\n'.join(python_files) if python_files else 'No Python files changed.',
        robot_files='\n'.join(robot_files) if robot_files else 'No Robot Framework files changed.'
    )

    # Append the filled content to the changelog file
    with open(changelog_file, 'a') as file:
        file.write('\n' + filled_content + '\n')

    print(f"Changelog updated with tag {tag} and commit message: {commit_message}")

if __name__ == "__main__":
    if len(sys.argv) < 6:
        print("Usage: python generate_changelog.py <template_file> <changelog_file> <tag> <commit_message> <robot_labels>")
        sys.exit(1)

    template_file = sys.argv[1]
    changelog_file = sys.argv[2]
    tag = sys.argv[3]
    commit_message = sys.argv[4]
    robot_labels = sys.argv[5]

    generate_changelog(template_file, changelog_file, tag, commit_message, robot_labels)
```

**Explanation:**

- **Functions:**
  - `get_changed_files()`: Uses `git diff` to get the list of files changed in the last commit.
  - `categorize_files(files)`: Splits the files into Python and Robot Framework files based on their extensions.
  - `generate_changelog(...)`: Fills in the template with the provided data and appends it to `changelog.rst`.

- **Usage:**
  - The script expects five arguments:
    1. `template_file`: Path to `changelog_template.rst`.
    2. `changelog_file`: Path to `changelog.rst`.
    3. `tag`: The Git tag name.
    4. `commit_message`: The commit message associated with the tag.
    5. `robot_labels`: Extracted robot labels from the commit message.

---

### **2. `changelog_template.rst`**

This is the template used by the Python script to generate the changelog entry.

```rst
Version ${tag} - Released on ${date}
------------------------------------

${comments}

**Robot Labels:** ${robot_labels}

**Changed Python Files:**
${python_files}

**Changed Robot Framework Files:**
${robot_files}
```

**Explanation:**

- **Placeholders:**
  - `${tag}`: The Git tag name.
  - `${date}`: The current date.
  - `${comments}`: The commit message.
  - `${robot_labels}`: Robot labels extracted from the commit message.
  - `${python_files}`: List of changed Python files.
  - `${robot_files}`: List of changed Robot Framework files.

- **Structure:**
  - The template uses reStructuredText (RST) formatting.
  - Each changelog entry includes all the placeholders, which will be replaced with actual values by the Python script.

---

### **3. `.gitlab-ci.yml`**

The GitLab CI/CD configuration file that defines the pipeline.

```yaml
stages:
  - update_changelog
  - commit_changes

variables:
  GIT_STRATEGY: fetch  # Ensure that git history is available

update_changelog:
  stage: update_changelog
  image: python:3.8  # Use a Python Docker image
  before_script:
    - pip install --upgrade pip
  script:
    - echo "Updating changelog with GitLab tag, commit message, robot labels, and today's date"
    - TAG_NAME=${CI_COMMIT_TAG}
    - COMMIT_MESSAGE=$(git log -1 --pretty=%B ${CI_COMMIT_SHA})  # Get the commit message
    - ROBOT_LABELS=$(echo "${COMMIT_MESSAGE}" | grep -o '\[.*\]' | tr -d '[]')  # Extract robot labels
    - python generate_changelog.py changelog_template.rst changelog.rst "$TAG_NAME" "$COMMIT_MESSAGE" "$ROBOT_LABELS"
  rules:
    - if: '$CI_COMMIT_TAG'  # Only run this job if there is a Git tag
  artifacts:
    paths:
      - changelog.rst

commit_changes:
  stage: commit_changes
  image: alpine/git  # Lightweight image with Git
  script:
    - echo "Committing updated changelog.rst"
    - git config --global user.email "ci-bot@example.com"
    - git config --global user.name "GitLab CI"
    - git add changelog.rst
    - git commit -m "Update changelog for tag $CI_COMMIT_TAG"
    - git push https://gitlab-ci-token:${CI_JOB_TOKEN}@${CI_SERVER_HOST}/${CI_PROJECT_PATH}.git HEAD:$CI_COMMIT_REF_NAME
  rules:
    - if: '$CI_COMMIT_TAG'  # Only run this job if there is a Git tag
  dependencies:
    - update_changelog
```

**Explanation:**

- **Stages:**
  - `update_changelog`: Runs the Python script to update the changelog.
  - `commit_changes`: Commits and pushes the updated `changelog.rst` back to the repository.

- **Variables:**
  - `GIT_STRATEGY: fetch`: Ensures that the Git history is available, which is necessary for `git diff` and `git log` commands.

- **Job: `update_changelog`**
  - **Image**: Uses `python:3.8` Docker image to run Python scripts.
  - **Before Script**:
    - Upgrades `pip` to the latest version.
  - **Script**:
    - Retrieves the tag name, commit message, and robot labels.
    - Runs `generate_changelog.py` with the appropriate arguments.
  - **Rules**:
    - Executes only when a Git tag is present.
  - **Artifacts**:
    - Saves `changelog.rst` as an artifact for the next stage.

- **Job: `commit_changes`**
  - **Image**: Uses `alpine/git` for lightweight Git operations.
  - **Script**:
    - Configures Git user details.
    - Adds and commits the updated `changelog.rst`.
    - Pushes the commit back to the repository on the same branch/tag.
  - **Dependencies**:
    - Depends on the `update_changelog` stage to ensure it has the updated `changelog.rst`.

- **Security Note**:
  - Uses the `CI_JOB_TOKEN` for authentication to push changes securely.

---

## **Workflow Summary**

1. **Trigger**: A new Git tag is pushed to the repository.

2. **Pipeline Execution**:
   - **Update Changelog**:
     - The pipeline checks out the code and ensures the Git history is available.
     - Retrieves the latest commit message and extracts robot labels.
     - Identifies changed Python and Robot Framework files.
     - Runs `generate_changelog.py` to update `changelog.rst`.
   - **Commit Changes**:
     - Commits the updated `changelog.rst` with a message like "Update changelog for tag v1.2.3".
     - Pushes the commit back to the repository under the same tag.

3. **Changelog Update**:
   - The `changelog.rst` now includes a new entry with:
     - The version tag and release date.
     - Commit message.
     - Robot labels.
     - Lists of changed Python and Robot Framework files.

---

## **Example Output**

Assuming the latest commit has the following:

- **Tag**: `v1.2.3`
- **Commit Message**: "Added new feature [robot-label1, robot-label2]"
- **Changed Files**:
  - `src/new_feature.py`
  - `tests/test_new_feature.robot`

After running the pipeline, the `changelog.rst` would have a new entry:

```rst
Version v1.2.3 - Released on 2023-10-05
------------------------------------

Added new feature [robot-label1, robot-label2]

**Robot Labels:** robot-label1, robot-label2

**Changed Python Files:**
src/new_feature.py

**Changed Robot Framework Files:**
tests/test_new_feature.robot
```

---

## **Additional Notes**

- **Robot Labels Extraction**:
  - The pipeline assumes that robot labels are included in the commit message enclosed in square brackets.
  - For example, a commit message like "Fixed issue with login [critical, login]" would extract `critical, login` as robot labels.

- **Error Handling**:
  - The Python script includes basic error handling to exit if it cannot retrieve changed files.

- **Flexibility**:
  - The script and pipeline can be adjusted to accommodate different file extensions or additional metadata as needed.

- **Security**:
  - The use of `CI_JOB_TOKEN` ensures secure authentication when the pipeline pushes changes back to the repository.

---

## **Setting Up the Project**

1. **Place the Files**:
   - Add `generate_changelog.py` to your repository.
   - Add `changelog_template.rst` to your repository.
   - Ensure your `changelog.rst` exists or is created by the script.

2. **Configure GitLab CI/CD**:
   - Place the provided `.gitlab-ci.yml` in the root of your repository.
   - Ensure that GitLab CI/CD is enabled for your project.

3. **Ensure Git History Availability**:
   - The `GIT_STRATEGY: fetch` variable in `.gitlab-ci.yml` ensures that the Git history is available for commands like `git log` and `git diff`.

4. **Push a Tag**:
   - When you push a new tag to the repository, the pipeline will automatically run and update the changelog.

---

## **Testing the Pipeline**

To test the pipeline before using it in production:

1. **Create a Test Branch**:
   - Create a new branch in your repository for testing purposes.

2. **Push Test Commits**:
   - Make some commits with different changes and include robot labels in the commit messages.

3. **Push a Test Tag**:
   - Push a new tag on the test branch to trigger the pipeline.

4. **Check Pipeline Output**:
   - Monitor the pipeline in GitLab to ensure it runs successfully.
   - Verify that the `changelog.rst` is updated correctly in the repository.

5. **Review the Changes**:
   - Check the updated `changelog.rst` in your repository to confirm that it includes all the expected information.

---

## **Conclusion**

This setup automates the process of maintaining your changelog and versioning, ensuring consistency and saving time. By integrating the robot labels and tracking changed files, it provides valuable context for each release, which is beneficial for both developers and stakeholders.

Feel free to adjust the scripts and configurations to better suit your project's specific needs.
