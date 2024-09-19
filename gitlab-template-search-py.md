Here's a Python script that uses the GitLab API to search for GitLab CI/CD templates. The script allows you to search for templates using keywords.

Before running the script, make sure you have the `requests` library installed. You can install it using pip:

```bash
pip install requests
```

Here is the Python script:

```python
import requests
import sys

def search_gitlab_templates(keyword, private_token):
    base_url = "https://gitlab.com/api/v4/templates"
    headers = {
        "PRIVATE-TOKEN": private_token
    }

    # Search in CI/CD templates
    response = requests.get(f"{base_url}/gitlab_ci_ymls", headers=headers)

    if response.status_code == 200:
        templates = response.json()
        matched_templates = [template for template in templates if keyword.lower() in template['name'].lower()]

        if matched_templates:
            print(f"Found {len(matched_templates)} templates matching '{keyword}':")
            for template in matched_templates:
                print(f"- {template['name']}: {template['content'][:100]}...")  # Truncate content for preview
        else:
            print(f"No templates found matching '{keyword}'.")
    else:
        print(f"Failed to fetch templates. Status code: {response.status_code}")
        sys.exit(1)

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: python search_gitlab_templates.py <keyword> <private_token>")
        sys.exit(1)

    keyword = sys.argv[1]
    private_token = sys.argv[2]

    search_gitlab_templates(keyword, private_token)
```

### Usage

1. Save the script to a file, for example, `search_gitlab_templates.py`.
2. Run the script using Python, passing in the search keyword and your GitLab private token:

```bash
python search_gitlab_templates.py <keyword> <your_private_token>
```

### Notes

- Replace `<your_private_token>` with your GitLab private token. This is required for authenticating with the GitLab API.
- The script searches for GitLab CI/CD templates (`gitlab_ci_ymls`). You can modify the script to search in other template types by changing the API endpoint.

This script should help you quickly search for GitLab CI/CD templates by keyword.
