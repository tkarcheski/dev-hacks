To create a GitLab CI pipeline with Ollama integration for processing code using an LLM such as Llama 3.2, I’ll outline the key components. This pipeline will perform the following:

- For every merge request, it will generate a patch file based on an LLM review.
- Developers can optionally apply the patch file manually.
- If the commit message includes `[ai]`, the patch will be automatically applied by the Ollama extension.

Here’s how you can set this up:

### Step 1: GitLab CI Pipeline Configuration (`.gitlab-ci.yml`)

1. **Define the stages** for the CI pipeline: `prepare`, `review`, and `patch`.
2. **Create a job** to generate a patch file for each merge request.
3. **Check for the `[ai]` flag** in the commit message to decide if the patch should be automatically applied.

```yaml
stages:
  - prepare
  - review
  - patch

variables:
  CONFIG_PROMPT: "Provide an intelligent code review and suggest improvements."
  OLLAMA_MODEL: "llama3.2"
  PATCH_DIR: "patches"
  PATCH_FILE: "${CI_COMMIT_SHA}.patch"

generate_patch:
  stage: review
  script:
    # Run a lint/check on the code changes, process through the Ollama model
    - mkdir -p $PATCH_DIR
    - echo "Generating code review with ${OLLAMA_MODEL}."
    - python scripts/generate_patch.py $CI_COMMIT_SHA $OLLAMA_MODEL $CONFIG_PROMPT > $PATCH_DIR/$PATCH_FILE
  artifacts:
    paths:
      - $PATCH_DIR/$PATCH_FILE
  only:
    - merge_requests

apply_patch:
  stage: patch
  script:
    - echo "Checking commit message for AI flag."
    - if [[ "$CI_COMMIT_MESSAGE" == *"[ai]"* ]]; then
        python scripts/apply_patch.py $PATCH_DIR/$PATCH_FILE;
      else
        echo "Patch not applied automatically. Please review manually.";
      fi
  dependencies:
    - generate_patch
  only:
    - merge_requests
```

### Step 2: Ollama Extension Setup

1. **Extension to Process Code:** The extension should interact with the Ollama API, sending relevant code sections for analysis using the `llama3.2` model.
2. **Patch Generation Script:** This script, `generate_patch.py`, will handle generating a patch file based on the code review provided by the model.
3. **Automatic Patch Application:** A second script, `apply_patch.py`, checks for the `[ai]` flag and, if present, applies the patch.

### `generate_patch.py`

This script sends the code diff to Ollama’s API and generates a patch based on suggestions.

```python
import os
import sys
import requests

def generate_patch(commit_sha, model_name, prompt):
    # Fetch the code diff for the commit
    diff = os.popen(f"git diff {commit_sha}~1 {commit_sha}").read()

    # Prepare Ollama request
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={"model": model_name, "prompt": prompt + "\n" + diff}
    )

    # Process Ollama response and generate patch
    if response.ok:
        suggestions = response.json()["output"]
        # Create patch format
        patch = f"--- {commit_sha}\n+++ {commit_sha}\n{suggestions}"
        return patch
    else:
        raise ValueError("Failed to generate suggestions from Ollama")

if __name__ == "__main__":
    commit_sha = sys.argv[1]
    model_name = sys.argv[2]
    prompt = sys.argv[3]
    patch = generate_patch(commit_sha, model_name, prompt)
    print(patch)
```

### `apply_patch.py`

This script applies the generated patch if the `[ai]` tag is present.

```python
import sys
import os

def apply_patch(patch_file):
    try:
        os.system(f"git apply {patch_file}")
        print(f"Patch {patch_file} applied successfully.")
    except Exception as e:
        print(f"Failed to apply patch: {e}")

if __name__ == "__main__":
    patch_file = sys.argv[1]
    apply_patch(patch_file)
```

### Step 3: Configuring the Ollama Extension

1. **Set up Ollama API locally** to ensure it can be accessed by the CI pipeline.
2. **Configurable System Prompt**: You can modify the `CONFIG_PROMPT` in `.gitlab-ci.yml` to customize the behavior and scope of the review.
3. **Environment Configuration**: Set up any necessary API keys or tokens as environment variables.

### Notes

- Ensure that the `ollama` server is accessible to the CI environment.
- You can adjust the `CONFIG_PROMPT` and `OLLAMA_MODEL` to fine-tune the type of review and analysis provided.
- The extension can be expanded to review and provide feedback on various types of code changes and styles, based on the models available in Ollama.

This CI pipeline and the scripts above provide a streamlined way to integrate LLM-based code reviews directly into GitLab merge requests.
